# Java File I/O: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** File I/O in Java refers to the reading from and writing to files on a computer's file system. Java provides two main APIs for this: the legacy `java.io.File` class (Java 1.0) and the modern `java.nio.file` package (NIO.2, introduced in Java 7), which includes the `Path` interface and the `Files` utility class.

**Technical Definition:** The `java.io.File` class is "an abstract representation of file and directory pathnames" [6†L5-L6]. Instances of the `File` class are immutable and "may or may not denote an actual file-system object such as a file or a directory" [6†L43-L44]. The `java.nio.file` package and its related `java.nio.file.attribute` package "provide comprehensive support for file I/O and for accessing the default file system" [7†L18-L20]. The `Path` interface is "the primary entry point for the package" [7†L26-L27], and the `Files` class "contains methods that deal with file operations" [7†L30-L31].

**Beginner-Friendly Explanation:** File I/O is how your Java program reads from and writes to files on your computer. The old way uses the `File` class, which is like a simple address book entry for a file — it tells you where the file is but has limited abilities. The modern way uses `Path` and `Files`, which are like a full-featured file manager — they can create, delete, copy, move, and inspect files with detailed error messages. As of Java 18, UTF-8 is the default encoding for text files, making international text handling much simpler [8†L11-L12].

### Key Characteristics

- **Legacy vs. Modern:** `java.io.File` is legacy (obsolete); `java.nio.file.Path` and `Files` are the modern standard [8†L17-L20].
- **Two Components of a Path:** An optional system-dependent prefix (like `C:\` on Windows or `/` on UNIX) and a sequence of zero or more string names [6†L8-L11].
- **Absolute vs. Relative:** An absolute pathname is complete; a relative pathname is resolved against the current user directory (`user.dir`) [6†L20-L27].
- **Immutable:** `File` instances are immutable; `Path` instances are also immutable.
- **Metadata-Rich:** NIO.2 provides extensive support for file attributes, permissions, and metadata.
- **Exception-Aware:** NIO.2 methods throw informative `IOException`s; legacy `File` methods often return booleans with no error detail [9†L16-L19].

### Prerequisites

- Basic understanding of Java classes, methods, and objects
- Familiarity with exceptions and exception handling (try-catch-finally)
- Knowledge of the `java.io` and `java.nio.file` packages
- Understanding of file paths and directories

### Related Programming Areas

- **Streams I/O:** Reading/writing file content using byte and character streams.
- **NIO.2:** The `java.nio.file` package provides the modern file API.
- **File Attributes:** Metadata such as permissions, owner, and timestamps.
- **Directory Operations:** Listing, walking, and filtering directory contents.
- **Concurrency:** Asynchronous file I/O with `AsynchronousFileChannel`.

### Core Concepts / Features

---

## 1. `File` Class

### Definitions

**Core Definition:** `java.io.File` is an abstract representation of file and directory pathnames, providing methods to inspect and manipulate files and directories.

**Technical Definition:** "An abstract representation of file and directory pathnames. User interfaces and operating systems use system-dependent pathname strings to name files and directories. This class presents an abstract, system-independent view of hierarchical pathnames" [6†L5-L8]. A `File` object "may or may not denote an actual file-system object such as a file or a directory" [6†L43-L44].

**Beginner-Friendly Explanation:** The `File` class is like a business card for a file. The card tells you the file's address (path), name, and some basic information, but it doesn't contain the file itself. You can use the card to ask questions about the file (Does it exist? Is it a directory?) and to perform basic operations (create, delete, rename).

### Purposes

- To represent file and directory pathnames in a system-independent way
- To inspect file metadata (name, path, size, existence)
- To perform basic file operations (create, delete, rename)
- To list the contents of directories
- To provide a bridge to the modern NIO.2 API via `toPath()`

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Constructors
File file = new File("path/to/file.txt");
File file = new File("parentDir", "childFile.txt");
File file = new File(parentFile, "childFile.txt");

// Common methods
boolean exists = file.exists();
boolean isFile = file.isFile();
boolean isDir = file.isDirectory();
String name = file.getName();
String path = file.getPath();
String absPath = file.getAbsolutePath();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `File(String pathname)` | Creates a File from a path string. |
| `File(String parent, String child)` | Creates a File from parent and child path strings. |
| `File(File parent, String child)` | Creates a File from a parent File and child string. |
| `exists()` | Returns `true` if the file/directory exists. |
| `isFile()` / `isDirectory()` | Tests whether the path denotes a file or directory. |
| `getName()` | Returns the name of the file or directory. |
| `getPath()` | Returns the path string. |
| `getAbsolutePath()` | Returns the absolute path string. |
| `toPath()` | Converts to a `java.nio.file.Path`. |

**Syntax Rules:**

- Path separators are system-dependent; use `File.separator` for portability.
- Relative paths are resolved against the current user directory (`user.dir`).
- The `File` class does not throw exceptions for failed operations (e.g., `delete()` returns `false`).

**Constraints and Limitations:**

- "Many methods didn't throw exceptions when they failed, so it was impossible to obtain a useful error message" [9†L16-L17].
- "The rename method didn't work consistently across platforms" [9†L20].
- "There was no real support for symbolic links" [9†L21].
- "Accessing file metadata was inefficient" [9†L23].
- "Many of the File methods didn't scale" [9†L23-L24].

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic File Inspection**

```java
// File: FileInspectionDemo.java
import java.io.File;

public class FileInspectionDemo {
    public static void main(String[] args) {
        // Create a File object for a path
        File file = new File("example.txt");

        // Inspect file metadata
        System.out.println("Name: " + file.getName());
        System.out.println("Path: " + file.getPath());
        System.out.println("Absolute Path: " + file.getAbsolutePath());
        System.out.println("Exists: " + file.exists());
        System.out.println("Is File: " + file.isFile());
        System.out.println("Is Directory: " + file.isDirectory());
        System.out.println("Can Read: " + file.canRead());
        System.out.println("Can Write: " + file.canWrite());
        System.out.println("Length: " + file.length() + " bytes");
    }
}
```

**Expected Output (if file does not exist):**
```
Name: example.txt
Path: example.txt
Absolute Path: /current/dir/example.txt
Exists: false
Is File: false
Is Directory: false
Can Read: false
Can Write: false
Length: 0 bytes
```

**Step-by-Step Explanation:**

1. A `File` object is created for the relative path `"example.txt"`.
2. `getName()` returns just the file name.
3. `getPath()` returns the path as provided (relative).
4. `getAbsolutePath()` resolves the path against the current working directory.
5. `exists()` returns `false` because the file does not exist.
6. The `isFile()`, `isDirectory()`, and `canRead()`/`canWrite()` methods return `false` for non-existent files.
7. `length()` returns `0` for non-existent files.

### Real-World Cases

- **Legacy Code Maintenance:** Existing codebases often use `File`; NIO.2 provides `toPath()` for gradual migration [9†L30-L33].
- **Simple Existence Checks:** `File.exists()` is a quick way to check if a file is present.
- **Directory Listing:** `File.listFiles()` returns an array of `File` objects for directory contents.

---

## 2. File Operations

### Definitions

**Core Definition:** File operations are the actions performed on files and directories: creation, deletion, renaming, copying, moving, and inspection.

**Technical Definition:** The `Files` class in `java.nio.file` "consists exclusively of static methods that operate on files, directories, or other types of files" [0†L18-L19]. It provides methods for creating (`createFile`, `createDirectory`), deleting (`delete`), copying (`copy`), moving (`move`), and inspecting (`exists`, `size`, `readAttributes`) files.

**Beginner-Friendly Explanation:** File operations are the everyday tasks you perform on files: creating new ones, deleting old ones, renaming, copying, and moving. Java's `Files` class provides simple, reliable methods for all of these.

### Purposes

- To create new files and directories
- To delete files and directories
- To rename or move files
- To copy files and directories
- To inspect file metadata (size, existence, attributes)

### Syntax Rules and Structure

**Common NIO.2 File Operations:**

| Operation | Method | Description |
|-----------|--------|-------------|
| Create file | `Files.createFile(path)` | Creates a new empty file. |
| Create directory | `Files.createDirectory(path)` | Creates a new directory. |
| Create directories | `Files.createDirectories(path)` | Creates directories including parents. |
| Delete | `Files.delete(path)` | Deletes a file or empty directory; throws on failure. |
| Delete if exists | `Files.deleteIfExists(path)` | Deletes if exists; returns boolean. |
| Copy | `Files.copy(source, target)` | Copies a file or directory. |
| Move | `Files.move(source, target)` | Moves or renames a file or directory. |
| Read attributes | `Files.readAttributes(path, BasicFileAttributes.class)` | Reads file metadata. |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Create, Write, Copy, and Delete a File**

```java
// File: FileOperationsDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.charset.StandardCharsets;

public class FileOperationsDemo {
    public static void main(String[] args) {
        Path source = Path.of("source.txt");
        Path copy = Path.of("copy.txt");

        try {
            // 1. Create a file and write content
            Files.writeString(source, "Hello, NIO.2!\n", StandardCharsets.UTF_8);
            System.out.println("File created: " + source);

            // 2. Copy the file
            Files.copy(source, copy, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("File copied to: " + copy);

            // 3. Read the copy
            String content = Files.readString(copy, StandardCharsets.UTF_8);
            System.out.println("Content: " + content.trim());

            // 4. Move/rename the copy
            Path renamed = Path.of("renamed.txt");
            Files.move(copy, renamed, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("File renamed to: " + renamed);

            // 5. Delete both files
            Files.delete(renamed);
            Files.delete(source);
            System.out.println("Files deleted successfully");

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
File created: source.txt
File copied to: copy.txt
Content: Hello, NIO.2!
File renamed to: renamed.txt
Files deleted successfully
```

**Step-by-Step Explanation:**

1. `Files.writeString()` creates `source.txt` and writes text using UTF-8 encoding.
2. `Files.copy()` copies `source.txt` to `copy.txt`, replacing if it exists.
3. `Files.readString()` reads the content of the copy.
4. `Files.move()` renames `copy.txt` to `renamed.txt`.
5. `Files.delete()` removes both files.
6. Each operation throws `IOException` on failure, providing a clear error message.

### Real-World Cases

- **Web Applications:** Creating temporary files for uploads and deleting them after processing.
- **Data Pipelines:** Copying files from one directory to another for staging.
- **Backup Utilities:** Moving files to backup locations.
- **Log Rotation:** Renaming log files and creating new ones.

---

## 3. File Paths

### Definitions

**Core Definition:** A file path is a hierarchical location that identifies a file or directory on a file system.

**Technical Definition:** "A `Path` represents a hierarchical location of a file or directory on a file system" [2†L19-L20]. The `java.nio.file.Path` interface "is the main entry point for working with this new API" and "acts as a direct replacement for `java.io.File` class" [5†L33-L35]. A path can be absolute (complete, no other information needed) or relative (must be resolved against another path) [6†L20-L24].

**Beginner-Friendly Explanation:** A file path is like an address for a file. An absolute path is a full address (e.g., `/home/user/documents/report.txt`); a relative path is a partial address (e.g., `documents/report.txt`) that is interpreted relative to where your program is running.

### Purposes

- To locate files and directories on the file system
- To construct and manipulate paths programmatically
- To resolve relative paths against a base directory
- To normalise and compare paths

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Creating a Path
Path path = Path.of("dir", "file.txt");
Path path = Paths.get("dir/file.txt");

// Path operations
Path absolute = path.toAbsolutePath();
Path normalized = path.normalize();
Path parent = path.getParent();
Path fileName = path.getFileName();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Path.of(String first, String... more)` | Creates a Path from path strings (Java 11+). |
| `Paths.get(String first, String... more)` | Creates a Path (Java 7+). |
| `toAbsolutePath()` | Returns the absolute path. |
| `normalize()` | Removes redundant elements (`.` and `..`). |
| `getParent()` | Returns the parent path. |
| `getFileName()` | Returns the file name component. |
| `resolve(Path other)` | Resolves another path against this path. |

**Syntax Rules:**

- Use `Path.of()` (Java 11+) or `Paths.get()` (Java 7+) to create paths.
- Paths are platform-independent; use `File.separator` or `Path` methods for portability.
- Use `resolve()` to combine paths.

**Constraints and Limitations:**

- Paths are not validated against the file system at creation time.
- Absolute paths require a file system root (e.g., `/` on UNIX, `C:\` on Windows).

### Annotated Complete Step-by-Step Code Examples

```java
// File: PathDemo.java
import java.nio.file.*;

public class PathDemo {
    public static void main(String[] args) {
        // Create a path
        Path path = Path.of("home", "user", "documents", "report.txt");
        System.out.println("Path: " + path);

        // Absolute path
        Path absolute = path.toAbsolutePath();
        System.out.println("Absolute: " + absolute);

        // Parent and file name
        System.out.println("Parent: " + path.getParent());
        System.out.println("File name: " + path.getFileName());

        // Normalize a path with redundant elements
        Path messy = Path.of("home", "user", ".", "docs", "..", "documents", "report.txt");
        System.out.println("Normalized: " + messy.normalize());

        // Resolve a path against a base
        Path base = Path.of("home", "user");
        Path resolved = base.resolve("documents/report.txt");
        System.out.println("Resolved: " + resolved);
    }
}
```

**Expected Output:**
```
Path: home/user/documents/report.txt
Absolute: /current/dir/home/user/documents/report.txt
Parent: home/user/documents
File name: report.txt
Normalized: home/user/documents/report.txt
Resolved: home/user/documents/report.txt
```

**Step-by-Step Explanation:**

1. `Path.of("home", "user", "documents", "report.txt")` creates a path with the correct separator for the platform.
2. `toAbsolutePath()` prepends the current working directory.
3. `getParent()` returns the parent path.
4. `getFileName()` returns the final component.
5. `normalize()` removes `.` and resolves `..` to produce a clean path.
6. `resolve()` combines a base path with a relative path.

### Real-World Cases

- **Configuration Files:** Resolving paths relative to the application's home directory.
- **File Uploads:** Constructing safe paths for user-uploaded files.
- **Resource Loading:** Locating resources in the classpath or file system.

---

## 4. File Creation

### Definitions

**Core Definition:** File creation is the process of creating a new, empty file on the file system.

**Technical Definition:** `Files.createFile(Path)` creates a new empty file. The operation is atomic with respect to other file-system operations. It throws `FileAlreadyExistsException` if the file already exists.

**Beginner-Friendly Explanation:** Creating a file is like creating a blank document. You give it a name and location, and Java creates an empty file ready for writing.

### Purposes

- To create new files for writing data
- To create temporary files for intermediate processing
- To create lock files for coordination
- To create placeholder files

### Syntax Rules and Structure

```java
Path file = Path.of("newfile.txt");
Files.createFile(file);  // Throws FileAlreadyExistsException if exists
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Files.createFile(path)` | Creates a new empty file. |
| `Files.createDirectories(path)` | Creates directories including parents. |
| `Files.createTempFile(prefix, suffix)` | Creates a temporary file. |

**Syntax Rules:**

- `createFile` throws `FileAlreadyExistsException` if the file exists.
- Parent directories must exist; use `createDirectories` first.
- `createTempFile` creates a file in the default temporary directory.

**Constraints and Limitations:**

- The parent directory must exist.
- File creation may fail due to permissions or disk space.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileCreationDemo.java
import java.io.IOException;
import java.nio.file.*;

public class FileCreationDemo {
    public static void main(String[] args) {
        Path dir = Path.of("testdir");
        Path file = dir.resolve("newfile.txt");

        try {
            // Create parent directory first
            Files.createDirectories(dir);
            System.out.println("Directory created: " + dir);

            // Create the file
            Files.createFile(file);
            System.out.println("File created: " + file);

            // Create a temporary file
            Path temp = Files.createTempFile("prefix", ".tmp");
            System.out.println("Temp file: " + temp);

            // Clean up
            Files.delete(file);
            Files.delete(temp);
            Files.delete(dir);
            System.out.println("Cleaned up");

        } catch (FileAlreadyExistsException e) {
            System.out.println("File already exists: " + e.getFile());
        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Directory created: testdir
File created: testdir/newfile.txt
Temp file: /tmp/prefix123456.tmp
Cleaned up
```

**Step-by-Step Explanation:**

1. `Files.createDirectories(dir)` creates the directory and any missing parents.
2. `Files.createFile(file)` creates an empty file inside the directory.
3. `Files.createTempFile("prefix", ".tmp")` creates a temporary file with a unique name.
4. All resources are cleaned up with `Files.delete()`.

### Real-World Cases

- **Application Startup:** Creating lock files to prevent multiple instances.
- **Data Processing:** Creating temporary files for intermediate results.
- **Testing:** Creating temporary test fixtures.

---

## 5. File Deletion

### Definitions

**Core Definition:** File deletion is the process of removing a file or directory from the file system.

**Technical Definition:** `Files.delete(Path)` deletes a file or empty directory. `Files.deleteIfExists(Path)` deletes if present and returns a boolean. Both throw `IOException` on failure (except `deleteIfExists` which returns `false` if the file does not exist).

**Beginner-Friendly Explanation:** Deleting a file is like throwing away a document. Java provides methods that either succeed or fail with a clear error message.

### Purposes

- To remove unwanted or temporary files
- To clean up after processing
- To enforce storage quotas
- To implement file lifecycle management

### Syntax Rules and Structure

```java
Files.delete(path);           // Throws if file doesn't exist or on failure
Files.deleteIfExists(path);   // Returns false if file doesn't exist
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `delete(path)` | Deletes a file or empty directory; throws `NoSuchFileException` if not found. |
| `deleteIfExists(path)` | Deletes if present; returns `true` if deleted, `false` if not found. |

**Syntax Rules:**

- `delete` throws `NoSuchFileException` if the file does not exist.
- `deleteIfExists` returns `false` if the file does not exist, avoiding exceptions.
- Directories must be empty before deletion (unless using `walkFileTree` with custom deletion).

**Constraints and Limitations:**

- Deleting a non-empty directory throws `DirectoryNotEmptyException`.
- File locks or permissions may prevent deletion.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileDeletionDemo.java
import java.io.IOException;
import java.nio.file.*;

public class FileDeletionDemo {
    public static void main(String[] args) {
        Path file = Path.of("tobedeleted.txt");

        try {
            // Create a file
            Files.writeString(file, "Temporary data");
            System.out.println("File created: " + file);

            // Delete with exception on failure
            Files.delete(file);
            System.out.println("File deleted with delete()");

            // deleteIfExists returns false for non-existent files
            boolean deleted = Files.deleteIfExists(file);
            System.out.println("deleteIfExists returned: " + deleted);

        } catch (NoSuchFileException e) {
            System.out.println("File not found: " + e.getFile());
        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
File created: tobedeleted.txt
File deleted with delete()
deleteIfExists returned: false
```

**Step-by-Step Explanation:**

1. `Files.writeString()` creates the file with content.
2. `Files.delete()` removes the file.
3. `Files.deleteIfExists()` is called on the already-deleted file and returns `false`.
4. No exceptions are thrown for the second call because `deleteIfExists` handles the missing file gracefully.

### Real-World Cases

- **Temporary File Cleanup:** Deleting temporary files after processing.
- **Log Rotation:** Deleting old log files.
- **User-Initiated Deletion:** Removing user files in a file manager application.

---

## 6. File Renaming

### Definitions

**Core Definition:** File renaming is the process of changing the name of a file or directory, or moving it to a different location.

**Technical Definition:** `Files.move(Path source, Path target)` moves or renames a file. "The rename method didn't work consistently across platforms" in the legacy `File` API [9†L20]; NIO.2 provides a consistent cross-platform implementation.

**Beginner-Friendly Explanation:** Renaming a file is like changing the label on a folder. You tell Java the old name and the new name, and it makes the change.

### Purposes

- To change a file's name
- To move a file to a different directory
- To reorganise file storage
- To implement file versioning

### Syntax Rules and Structure

```java
Files.move(source, target);  // Rename or move
Files.move(source, target, StandardCopyOption.REPLACE_EXISTING);
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Files.move(source, target)` | Moves or renames; throws `FileAlreadyExistsException` if target exists. |
| `StandardCopyOption.REPLACE_EXISTING` | Replaces the target if it exists. |
| `StandardCopyOption.ATOMIC_MOVE` | Performs the move atomically. |

**Syntax Rules:**

- `move` can rename within the same directory or move to a different directory.
- By default, `move` fails if the target exists.
- Use `REPLACE_EXISTING` to overwrite the target.

**Constraints and Limitations:**

- Cross-file-system moves may not be atomic.
- Moving a directory moves all its contents.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileRenameDemo.java
import java.io.IOException;
import java.nio.file.*;

public class FileRenameDemo {
    public static void main(String[] args) {
        Path original = Path.of("original.txt");
        Path renamed = Path.of("renamed.txt");
        Path moved = Path.of("subdir", "moved.txt");

        try {
            // Create the original file
            Files.writeString(original, "Content");
            System.out.println("Created: " + original);

            // Rename within the same directory
            Files.move(original, renamed);
            System.out.println("Renamed to: " + renamed);

            // Create directory and move the file
            Files.createDirectories(moved.getParent());
            Files.move(renamed, moved);
            System.out.println("Moved to: " + moved);

            // Clean up
            Files.delete(moved);
            Files.delete(moved.getParent());
            System.out.println("Cleaned up");

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Created: original.txt
Renamed to: renamed.txt
Moved to: subdir/moved.txt
Cleaned up
```

**Step-by-Step Explanation:**

1. `Files.writeString()` creates the original file.
2. `Files.move(original, renamed)` renames the file in place.
3. `Files.createDirectories()` creates the target directory.
4. `Files.move(renamed, moved)` moves the file to the subdirectory.
5. Cleanup removes the file and directory.

### Real-World Cases

- **File Uploads:** Renaming uploaded files to unique names.
- **Batch Processing:** Renaming processed files with a timestamp suffix.
- **File Organisation:** Moving files into category-based directories.

---

## 7. File Listing

### Definitions

**Core Definition:** File listing is the process of enumerating the contents of a directory.

**Technical Definition:** The `Files` class provides `Files.list(Path)` which returns a lazy `Stream<Path>` of directory entries, and `Files.walk(Path)` which returns a `Stream<Path>` of all files and directories recursively. The legacy `File.listFiles()` returns an array of `File` objects.

**Beginner-Friendly Explanation:** Listing files is like reading the table of contents of a folder. You get a list of all files and subdirectories inside a directory.

### Purposes

- To enumerate directory contents
- To filter files by criteria (extension, size, date)
- To recursively process directory trees
- To implement file browsers and search utilities

### Syntax Rules and Structure

```java
try (Stream<Path> entries = Files.list(dir)) {
    entries.forEach(System.out::println);
}

try (Stream<Path> tree = Files.walk(dir)) {
    tree.filter(Files::isRegularFile).forEach(System.out::println);
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Files.list(dir)` | Returns a lazy `Stream<Path>` of direct entries. |
| `Files.walk(dir)` | Returns a lazy `Stream<Path>` of all entries recursively. |
| `Files.walk(dir, depth)` | Limits recursion depth. |
| `Files.newDirectoryStream(dir)` | Returns a `DirectoryStream` for iteration. |

**Syntax Rules:**

- `Files.list()` and `Files.walk()` return streams that must be closed (use try-with-resources) [8†L40-L42].
- `Files.walk()` follows symbolic links by default; use `FileVisitOption` to control.
- Filter using stream operations like `filter()`, `map()`, `sorted()`.

**Constraints and Limitations:**

- Large directories can consume significant memory if not processed lazily.
- `Files.walk()` on huge trees may be slow; consider `walkFileTree` with a visitor for more control.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileListingDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.util.stream.Stream;

public class FileListingDemo {
    public static void main(String[] args) {
        Path dir = Path.of("testdir");

        try {
            // Create sample directory structure
            Files.createDirectories(dir.resolve("subdir"));
            Files.writeString(dir.resolve("file1.txt"), "A");
            Files.writeString(dir.resolve("file2.txt"), "B");
            Files.writeString(dir.resolve("subdir/file3.txt"), "C");

            // List direct entries
            System.out.println("Direct entries:");
            try (Stream<Path> entries = Files.list(dir)) {
                entries.forEach(p -> System.out.println("  " + p.getFileName()));
            }

            // Walk the tree recursively
            System.out.println("All files (recursive):");
            try (Stream<Path> tree = Files.walk(dir)) {
                tree.filter(Files::isRegularFile)
                    .forEach(p -> System.out.println("  " + p));
            }

            // Clean up
            try (Stream<Path> cleanup = Files.walk(dir)) {
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
  subdir
All files (recursive):
  testdir/file1.txt
  testdir/file2.txt
  testdir/subdir/file3.txt
```

**Step-by-Step Explanation:**

1. `Files.createDirectories()` and `Files.writeString()` create a sample directory structure.
2. `Files.list(dir)` returns a stream of direct entries; `forEach` prints each.
3. `Files.walk(dir)` returns a recursive stream; `filter(Files::isRegularFile)` keeps only files.
4. Cleanup uses `Files.walk(dir)` with reverse sorting to delete children before parents.

### Real-World Cases

- **File Search Utilities:** Finding all files with a specific extension.
- **Build Tools:** Scanning source directories for compilation.
- **Backup Systems:** Enumerating files to back up.
- **Log Analysis:** Walking a log directory and processing recent files.

---

## 8. File Attributes

### Definitions

**Core Definition:** File attributes are metadata about a file, such as its size, creation time, last modified time, and type.

**Technical Definition:** The `Files.readAttributes(Path, Class)` method reads file attributes as an object of the specified type. `BasicFileAttributes` provides common attributes: creation time, last access time, last modified time, size, and type (regular file, directory, symbolic link, or other) [7†L40-L41].

**Beginner-Friendly Explanation:** File attributes are the "details" about a file — when it was created, when it was last changed, how big it is, and whether it's a file or a directory.

### Purposes

- To inspect file metadata for display or processing
- To determine when a file was last modified for cache validation
- To check file size before processing
- To identify file types

### Syntax Rules and Structure

```java
BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);

attrs.creationTime();      // FileTime
attrs.lastModifiedTime();  // FileTime
attrs.size();              // long
attrs.isRegularFile();     // boolean
attrs.isDirectory();       // boolean
attrs.isSymbolicLink();    // boolean
```

**Component Breakdown:**

| Attribute | Description |
|-----------|-------------|
| `creationTime()` | When the file was created. |
| `lastModifiedTime()` | When the file was last modified. |
| `lastAccessTime()` | When the file was last accessed. |
| `size()` | File size in bytes. |
| `isRegularFile()` | Whether the entry is a regular file. |
| `isDirectory()` | Whether the entry is a directory. |
| `isSymbolicLink()` | Whether the entry is a symbolic link. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileAttributesDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

public class FileAttributesDemo {
    public static void main(String[] args) {
        Path file = Path.of("attributes.txt");

        try {
            Files.writeString(file, "Hello, attributes!");
            BasicFileAttributes attrs = Files.readAttributes(file, BasicFileAttributes.class);

            System.out.println("Created: " + attrs.creationTime());
            System.out.println("Modified: " + attrs.lastModifiedTime());
            System.out.println("Accessed: " + attrs.lastAccessTime());
            System.out.println("Size: " + attrs.size() + " bytes");
            System.out.println("Is file: " + attrs.isRegularFile());
            System.out.println("Is directory: " + attrs.isDirectory());
            System.out.println("Is symlink: " + attrs.isSymbolicLink());

            Files.delete(file);

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output (times vary):**
```
Created: 2026-10-16T10:30:00Z
Modified: 2026-10-16T10:30:00Z
Accessed: 2026-10-16T10:30:00Z
Size: 19 bytes
Is file: true
Is directory: false
Is symlink: false
```

**Step-by-Step Explanation:**

1. `Files.writeString()` creates the file with content.
2. `Files.readAttributes()` reads the basic attributes.
3. Each attribute is printed: creation, modification, and access times, size, and type flags.
4. The file is deleted.

### Real-World Cases

- **Cache Validation:** Checking `lastModifiedTime()` to decide if a cached file is stale.
- **File Managers:** Displaying file size and modification date.
- **Backup Systems:** Comparing modification times to determine what to back up.
- **Build Tools:** Checking if source files are newer than compiled output.

---

## 9. File Permissions

### Definitions

**Core Definition:** File permissions are the access rights that determine who can read, write, or execute a file.

**Technical Definition:** On POSIX systems, the `PosixFileAttributeView` provides access to "the nine permission bits defined by the `PosixFilePermission` class. These nine permission bits determine the read, write, and execute access for the file owner, group, and others" [10†L31-L36]. The `PosixFilePermissions` class provides methods for reading and setting permissions.

**Beginner-Friendly Explanation:** File permissions are like the rules for who can open a door. The owner might have full access, a group might have limited access, and others might have no access. Java's NIO.2 API lets you read and change these rules.

### Purposes

- To inspect who can read, write, or execute a file
- To restrict access to sensitive files
- To make scripts executable
- To enforce security policies

### Syntax Rules and Structure

```java
// Reading permissions
PosixFileAttributes attrs = Files.readAttributes(path, PosixFileAttributes.class);
Set<PosixFilePermission> perms = attrs.permissions();
System.out.println(PosixFilePermissions.toString(perms));

// Setting permissions
Set<PosixFilePermission> perms = PosixFilePermissions.fromString("rw-r--r--");
Files.setPosixFilePermissions(path, perms);
```

**Component Breakdown:**

| Permission | Symbol | Description |
|------------|--------|-------------|
| Owner read | `r` | Owner can read. |
| Owner write | `w` | Owner can write. |
| Owner execute | `x` | Owner can execute. |
| Group read | `r` | Group can read. |
| Group write | `w` | Group can write. |
| Group execute | `x` | Group can execute. |
| Others read | `r` | Others can read. |
| Others write | `w` | Others can write. |
| Others execute | `x` | Others can execute. |

**Syntax Rules:**

- POSIX permissions are only available on POSIX-compliant file systems (Linux, macOS).
- On Windows, use ACL-based permissions via `AclFileAttributeView`.
- Use `PosixFilePermissions.fromString()` to create permission sets from symbolic strings.

**Constraints and Limitations:**

- POSIX permissions are not available on all platforms.
- Changing permissions may require appropriate privileges.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FilePermissionsDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.*;
import java.util.Set;

public class FilePermissionsDemo {
    public static void main(String[] args) {
        Path file = Path.of("permtest.txt");

        try {
            Files.writeString(file, "Permission test");

            // Check if POSIX permissions are supported
            if (Files.getFileStore(file).supportsFileAttributeView(PosixFileAttributeView.class)) {

                // Read permissions
                PosixFileAttributes attrs = Files.readAttributes(file, PosixFileAttributes.class);
                System.out.println("Owner: " + attrs.owner().getName());
                System.out.println("Group: " + attrs.group().getName());
                System.out.println("Permissions: " + PosixFilePermissions.toString(attrs.permissions()));

                // Set new permissions: owner rwx, group r--, others ---
                Set<PosixFilePermission> perms = PosixFilePermissions.fromString("rwxr-----");
                Files.setPosixFilePermissions(file, perms);
                System.out.println("New permissions: " + PosixFilePermissions.toString(perms));

            } else {
                System.out.println("POSIX permissions not supported on this file system");
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
Owner: user
Group: user
Permissions: rw-r--r--
New permissions: rwxr-----
```

**Step-by-Step Explanation:**

1. `Files.writeString()` creates the file.
2. `Files.getFileStore().supportsFileAttributeView()` checks if POSIX permissions are supported.
3. `Files.readAttributes()` reads the POSIX attributes including owner, group, and permissions.
4. `PosixFilePermissions.fromString("rwxr-----")` creates a permission set.
5. `Files.setPosixFilePermissions()` applies the new permissions.
6. The file is deleted.

### Real-World Cases

- **Security-Sensitive Files:** Setting `rw-------` for private keys.
- **Executable Scripts:** Setting `rwxr-xr-x` for shell scripts.
- **Shared Directories:** Setting `rwxrwxr-x` for team collaboration.
- **Configuration Files:** Setting `rw-r--r--` for read-only configuration.

---

## 10. File I/O Best Practices

### Definitions

**Core Definition:** File I/O best practices are guidelines for writing efficient, safe, and maintainable file handling code in Java.

**Technical Definition:** Best practices derive from authoritative sources including Oracle's Java Tutorials and Dev.java. Key principles include using NIO.2 APIs, leveraging try-with-resources, preferring `Files` utility methods over legacy `File` methods, and using UTF-8 encoding.

**Beginner-Friendly Explanation:** Best practices help you write file handling code that is fast, safe, and easy to maintain. They prevent common mistakes like forgetting to close streams, using outdated APIs, or writing inefficient code.

### Purposes

- To prevent resource leaks
- To improve I/O performance
- To ensure data integrity
- To write portable and maintainable code
- To handle errors gracefully

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use NIO.2 (`Path`/`Files`) | "Stay away from the legacy `File` class" [8†L17-L20]. | Dev.java |
| Use Try-with-Resources | "The stream returned by `Files.lines` needs to be closed. To ensure that this happens, use a try-with-resources statement" [8†L40-L42]. | Oracle |
| Use UTF-8 by Default | "UTF-8 is the default for I/O since Java 18" [8†L11-L12]. | Dev.java |
| Prefer `Files.writeString`/`readString` | "Use `Files.writeString()` and `Files.readString()` for concise text I/O" [2†L21-L22]. | Dev.java |
| Use `Files.lines` for Large Files | "If the file is large, process the lines lazily as a `Stream<String>`" [8†L32-L33]. | Dev.java |
| Use `Files.walk` for Traversal | "Use `Files.list` or `Files.walk` to traverse directory entries" [4†L6-L7]. | Dev.java |
| Specify Charset for Portability | "Always specify the charset for text" [4†L29]. | Best practice |
| Handle Exceptions with `IOException` | NIO.2 methods throw `IOException` with useful messages. | Oracle |
| Validate Paths Before Operations | Check existence and permissions before acting. | Best practice |
| Use `Path.of` (Java 11+) | Modern factory method for creating paths. | Dev.java |
| Avoid `File` for New Code | "The `java.io.File` class is now thoroughly obsolete" [8†L17-L20]. | Dev.java |
| Use `Files.createTempFile` | For temporary files with unique names. | Oracle |
| Use `StandardCharsets.UTF_8` | Explicitly specify UTF-8 for compatibility. | Dev.java |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileIOBestPractices.java
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.stream.Stream;

public class FileIOBestPractices {
    public static void main(String[] args) {
        Path path = Path.of("best_practices.txt");

        // BEST PRACTICE: Use Files.writeString for simple writes (Java 11+)
        try {
            Files.writeString(path, "Line 1\nLine 2\n", StandardCharsets.UTF_8);
            System.out.println("File written with UTF-8");
        } catch (IOException e) {
            System.out.println("Write error: " + e.getMessage());
            return;
        }

        // BEST PRACTICE: Use Files.readString for simple reads (Java 11+)
        try {
            String content = Files.readString(path, StandardCharsets.UTF_8);
            System.out.println("Content: " + content.trim());
        } catch (IOException e) {
            System.out.println("Read error: " + e.getMessage());
        }

        // BEST PRACTICE: Use Files.lines for lazy stream processing
        try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8)) {
            lines.forEach(line -> System.out.println("Line: " + line));
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }

        // BEST PRACTICE: Use Files.walk for directory traversal
        try (Stream<Path> tree = Files.walk(Path.of("."))) {
            tree.filter(Files::isRegularFile)
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
Found: ./best_practices.txt
Cleaned up
```

**Step-by-Step Explanation:**

1. `Files.writeString()` writes text using UTF-8 encoding.
2. `Files.readString()` reads the entire file into a string.
3. `Files.lines()` returns a lazy `Stream<String>` processed line by line.
4. `Files.walk()` traverses the directory tree; `filter(Files::isRegularFile)` keeps only files.
5. `Files.deleteIfExists()` cleans up the temporary file.

### Real-World Cases

- **Web Applications:** Reading configuration files at startup with UTF-8.
- **Data Processing:** Processing large log files with `Files.lines` for memory efficiency.
- **File Search:** Using `Files.walk` to find files matching criteria.
- **Backup Utilities:** Copying files with `Files.copy` and `StandardCopyOption`.

---

## References Links

- Class File — Java® Platform, Standard Edition & Java Development Kit Version 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html
- File I/O (Featuring NIO.2) — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/io/fileio.html
- Common I/O Tasks in Modern Java — Dev.java - https://dev.java/learn/modernio/
- Legacy File I/O Code — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/io/legacy.html
- Class Files — Java® Platform, Standard Edition & Java Development Kit Version 17 API Specification - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/file/Files.html
- Interface Path — Java® Platform, Standard Edition & Java Development Kit Version 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html
- Interface PosixFileAttributeView — Java® Platform, Standard Edition & Java Development Kit Version 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/attribute/PosixFileAttributeView.html
- Class PosixFilePermissions — Java® Platform, Standard Edition & Java Development Kit Version 23 API Specification - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/nio/file/attribute/PosixFilePermissions.html
- JEP 400: UTF-8 by Default — OpenJDK - https://openjdk.org/jeps/400
- Package java.nio.file — Java® Platform, Standard Edition & Java Development Kit Version 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/package-summary.html