# Java I/O Fundamentals: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Java I/O (Input/Output)** is the mechanism by which Java programs communicate with the outside world—reading data from sources (keyboard, files, network, other programs) and writing data to destinations (screen, files, network, other programs). It is built on the concept of **streams**: unidirectional sequences of data that flow into or out of a program .

### Technical Definition

The `java.io` package provides two independent class hierarchies for I/O: **byte streams** (`InputStream`/`OutputStream`) that operate on 8-bit bytes, and **character streams** (`Reader`/`Writer`) that operate on 16-bit Unicode characters . Streams are FIFO queues with sequential access; they abstract away the details of the underlying data source or destination, presenting a uniform programming interface . The `try-with-resources` statement (Java 7+) ensures that resources implementing `AutoCloseable` are properly closed, preventing resource leaks . Standard streams (`System.in`, `System.out`, `System.err`) are automatically available for command-line interaction .

### Beginner-Friendly Explanation

Think of Java I/O as a system of pipes. When you want to bring data into your program (like reading a file or typing on a keyboard), you attach an **input pipe** to the data source. When you want to send data out (like writing to a file or displaying text), you attach an **output pipe** to the destination. Each pipe carries data in one direction only. If you need to read and write, you use two pipes. Java provides different pipe sizes: some carry tiny 8-bit pieces (bytes), others carry 16-bit characters for text.

### Key Characteristics

- **Stream-based abstraction**: All I/O operations use a uniform stream model .
- **Unidirectional**: Input streams read; output streams write; separate streams are needed for bidirectional communication .
- **Two hierarchies**: Byte-oriented (`InputStream`/`OutputStream`) and character-oriented (`Reader`/`Writer`) .
- **FIFO ordering**: Data is read/written in the order it was produced .
- **Decorator pattern**: Streams can be layered to add functionality (buffering, data conversion, compression) .
- **Automatic resource management**: `try-with-resources` ensures `close()` is called .

### Prerequisites

- Basic Java syntax (classes, methods, exceptions).
- Understanding of the `try-catch-finally` mechanism.
- Familiarity with file paths (using `File` or `Path`).

### Related Programming Areas

- **File Systems**: Reading and writing files, traversing directories.
- **Networking**: Reading from and writing to sockets.
- **Serialization**: Converting objects to byte streams.
- **Internationalization**: Character encoding and Unicode handling.
- **Modern I/O**: The `java.nio` (New I/O) API for scalable, non-blocking operations.

### Core Concepts / Features

1. Input Streams (`InputStream`) and Output Streams (`OutputStream`)
2. Reader (`Reader`) and Writer (`Writer`)
3. Byte-Oriented vs. Character-Oriented I/O (Encoding/Charsets)
4. Resource Lifecycle Management and `try-with-resources` (`AutoCloseable`)
5. Standard I/O Streams (`System.in`, `System.out`, `System.err`)


## Core Concept 1: Input Streams (`InputStream`) and Output Streams (`OutputStream`)

### Definitions

**Core Definition**: `InputStream` and `OutputStream` are the abstract base classes for reading and writing **binary data** (8-bit bytes) in Java.

**Technical Definition**: `InputStream` is the abstract superclass for all byte-input streams. It defines methods for reading bytes and byte arrays, skipping input, and marking positions . `OutputStream` is the abstract superclass for all byte-output streams, defining methods for writing bytes and byte arrays . These classes are intended for binary data (images, audio, serialized objects) where byte-level manipulation is appropriate .

**Beginner-Friendly Explanation**: Byte streams are like pipes that carry tiny Lego bricks. They don't know or care what the bricks represent—they just move them one at a time. If you're dealing with images, music files, or any non-text data, you use byte streams.

### Purposes

- To read and write raw binary data (images, audio, serialized objects).
- To provide the foundation for all other byte-oriented streams through the decorator pattern.
- To enable low-level, byte-at-a-time processing when character encoding is irrelevant.
- To serve as the underlying transport for character streams via bridge classes.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Reading bytes
InputStream in = new FileInputStream("data.bin");
int byteValue = in.read();  // -1 at end of stream
in.close();

// Writing bytes
OutputStream out = new FileOutputStream("output.bin");
out.write(byteValue);
out.close();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `InputStream` | Abstract base for byte input streams. |
| `OutputStream` | Abstract base for byte output streams. |
| `read()` | Returns the next byte as an `int` (0–255), or -1 at EOF . |
| `write(int)` | Writes the low 8 bits of the `int` argument . |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `int read()` | Reads one byte, returns -1 at EOF. |
| `int read(byte[] b)` | Reads into a byte array, returns bytes read. |
| `void write(int b)` | Writes one byte. |
| `void write(byte[] b)` | Writes a byte array. |
| `void close()` | Closes the stream and releases resources. |

**Syntax Rules:**

- `read()` returns an `int` (0–255) or -1 at end of stream .
- `write(int)` writes only the low 8 bits of the argument.
- Always close streams to free system resources.
- Streams are automatically opened when created; they must be explicitly closed .

**Constraints and Limitations:**

- Byte streams do not handle character encoding; text read as bytes may be garbled.
- `read()` is blocking; it waits until data is available.
- Basic byte streams are oriented around byte-at-a-time operations, which can be inefficient .

### Annotated Complete Code Examples

**Example 1: Reading and Writing Binary Data**

```java
import java.io.*;

public class ByteStreamDemo {
    public static void main(String[] args) {
        // Writing bytes to a file
        try (OutputStream out = new FileOutputStream("bytes.bin")) {
            out.write(65);  // ASCII 'A'
            out.write(66);  // ASCII 'B'
            out.write(67);  // ASCII 'C'
            System.out.println("Bytes written.");
        } catch (IOException e) {
            System.err.println("Write error: " + e.getMessage());
        }

        // Reading bytes from the file
        try (InputStream in = new FileInputStream("bytes.bin")) {
            int byteValue;
            System.out.print("Bytes read: ");
            while ((byteValue = in.read()) != -1) {
                System.out.print((char) byteValue + " ");
            }
            System.out.println();
        } catch (IOException e) {
            System.err.println("Read error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Bytes written.
Bytes read: A B C
```

**Why This Output Occurs:**
- `write(65)`, `write(66)`, `write(67)` write the bytes for 'A', 'B', 'C'.
- `read()` returns each byte as an `int` until -1 at EOF.
- Casting to `char` displays the characters.

### Real-World Cases

- **File copying**: Reading bytes from one file and writing to another.
- **Image processing**: Reading raw image bytes for manipulation.
- **Network communication**: Sending and receiving binary protocols.
- **Object serialization**: `ObjectOutputStream` extends `OutputStream` .

### References

- IIT Bombay – Overview of I/O Streams - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/essential/io/overview.html
- MIT OpenCourseWare – File I/O - https://ocw.mit.edu/courses/1-124j-foundations-of-software-engineering-fall-2000/pages/lecture-notes/file_io/


## Core Concept 2: Reader (`Reader`) and Writer (`Writer`)

### Definitions

**Core Definition**: `Reader` and `Writer` are the abstract base classes for reading and writing **character data** (16-bit Unicode characters) in Java.

**Technical Definition**: `Reader` is the abstract superclass for character-input streams, and `Writer` is the abstract superclass for character-output streams. They were introduced in JDK 1.1 to support internationalization. Where byte-stream methods operate on bytes, character-stream methods operate on characters, character arrays, or strings . The primary advantage of character streams is that they hide the complexity of character encoding conversion .

**Beginner-Friendly Explanation**: Character streams are like pipes that carry letters and symbols instead of raw bytes. They automatically translate between the bytes on disk and the characters your program sees, so you don't have to worry about encoding—you just read and write text.

### Purposes

- To read and write textual data with automatic character encoding conversion.
- To support internationalization by handling Unicode and various encodings.
- To provide more efficient buffered character I/O .
- To enable line-by-line reading through `BufferedReader.readLine()` .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Reading characters
Reader reader = new FileReader("text.txt");
int charValue = reader.read();  // -1 at end
reader.close();

// Writing characters
Writer writer = new FileWriter("output.txt");
writer.write("Hello");
writer.close();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Reader` | Abstract base for character input streams . |
| `Writer` | Abstract base for character output streams . |
| `read()` | Returns the next character as an `int` (0–65535), or -1 at EOF. |
| `write(int)` | Writes a single character. |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `int read()` | Reads one character, returns -1 at EOF. |
| `int read(char[] cbuf)` | Reads into a char array. |
| `void write(int c)` | Writes one character. |
| `void write(String str)` | Writes a string. |
| `void flush()` | Flushes the buffer. |

**Syntax Rules:**

- Character streams operate on 16-bit Unicode characters .
- Most character-stream class names parallel byte-stream names (e.g., `FileReader` ↔ `FileInputStream`) .
- `BufferedReader` adds `readLine()` for line-by-line reading .
- `PrintWriter` adds `print()` and `println()` for formatted output.

**Constraints and Limitations:**

- Character streams are slightly less efficient than byte streams for raw binary data due to encoding conversion.
- `FileReader` uses the platform's default encoding unless a `Charset` is specified.
- `BufferedReader` and `BufferedWriter` are needed for efficient line-by-line processing.

### Annotated Complete Code Examples

**Example 1: Reading and Writing Text with Character Streams**

```java
import java.io.*;

public class CharStreamDemo {
    public static void main(String[] args) {
        // Writing text to a file
        try (Writer writer = new FileWriter("text.txt")) {
            writer.write("Hello, Character Streams!");
            writer.write("\nSecond line");
            System.out.println("Text written.");
        } catch (IOException e) {
            System.err.println("Write error: " + e.getMessage());
        }

        // Reading text with BufferedReader for line-by-line access
        try (BufferedReader reader = new BufferedReader(new FileReader("text.txt"))) {
            String line;
            System.out.println("File contents:");
            while ((line = reader.readLine()) != null) {
                System.out.println("  " + line);
            }
        } catch (IOException e) {
            System.err.println("Read error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Text written.
File contents:
  Hello, Character Streams!
  Second line
```

**Why This Output Occurs:**
- `FileWriter` writes characters to the file, handling encoding automatically.
- `BufferedReader.readLine()` reads lines efficiently, returning `null` at EOF .
- The `try-with-resources` ensures both streams are closed.

### Real-World Cases

- **Reading configuration files**: Properties files, JSON, XML.
- **Log processing**: Reading and analyzing log lines.
- **CSV parsing**: Reading comma-separated values line by line.
- **Text editing**: Character-by-character processing of documents.

### References

- Oracle Help Center – Character Streams - https://docs.oracle.com/javase/8/docs/technotes/guides/io/io.html
- MIT OpenCourseWare – File I/O - https://ocw.mit.edu/courses/1-124j-foundations-of-software-engineering-fall-2000/pages/lecture-notes/file_io/


## Core Concept 3: Byte-Oriented vs. Character-Oriented I/O (Encoding/Charsets)

### Definitions

**Core Definition**: **Byte-oriented I/O** operates on raw 8-bit bytes without interpretation; **character-oriented I/O** operates on 16-bit Unicode characters and handles conversion between bytes and characters using a specified **charset** (encoding).

**Technical Definition**: Java stores strings in Unicode internally, but typical text files use encodings like UTF-8, ISO-8859-1, or Windows-1252 . Character streams provide **bridge classes**—`InputStreamReader` and `OutputStreamWriter`—that convert between byte streams and character streams using a specified or default charset . The `InputStreamReader` reads bytes from an `InputStream` and decodes them into characters; `OutputStreamWriter` encodes characters into bytes and writes them to an `OutputStream`.

**Beginner-Friendly Explanation**: Byte streams are like raw ingredients; character streams are like a recipe that tells you what those ingredients mean. The bridge classes are translators: they take the raw bytes and translate them into readable characters using a codebook (charset). If you use the wrong codebook, the translation comes out garbled.

### Purposes

- To correctly handle text data across different languages and platforms.
- To make programs independent of platform-specific encoding .
- To read and write text files with explicit control over encoding.
- To bridge between byte-oriented and character-oriented APIs.

### Syntax Rules and Structure

**Bridge Classes:**

```java
// Byte stream → Character stream (decoding)
Reader reader = new InputStreamReader(
    new FileInputStream("data.txt"),
    StandardCharsets.UTF_8  // Explicit charset
);

// Character stream → Byte stream (encoding)
Writer writer = new OutputStreamWriter(
    new FileOutputStream("output.txt"),
    StandardCharsets.UTF_8
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `InputStreamReader` | Decodes bytes into characters . |
| `OutputStreamWriter` | Encodes characters into bytes . |
| `Charset` | The encoding scheme (UTF-8, ISO-8859-1, etc.). |

**Common Charsets:**

| Charset | Description |
|---------|-------------|
| `UTF-8` | Variable-width Unicode encoding (default since Java 18) . |
| `ISO-8859-1` | 8-bit Latin-1 encoding. |
| `US-ASCII` | 7-bit ASCII. |
| `UTF-16` | 16-bit Unicode encoding. |

**Syntax Rules:**

- If no charset is specified, the platform's default encoding is used (historically a portability issue) .
- Since Java 18, UTF-8 is the default for I/O (JEP 400) .
- Always specify a charset explicitly for portable code.
- `Charset.forName("UTF-8")` or `StandardCharsets.UTF_8` for the charset object.

**Constraints and Limitations:**

- Using the wrong charset produces garbled text (mojibake).
- Charset conversion adds CPU overhead.
- Not all byte sequences are valid in all charsets; invalid sequences may be replaced or cause exceptions.

### Annotated Complete Code Examples

**Example 1: Reading with Explicit Encoding**

```java
import java.io.*;
import java.nio.charset.StandardCharsets;

public class EncodingDemo {
    public static void main(String[] args) {
        // Write UTF-8 text using OutputStreamWriter
        try (Writer writer = new OutputStreamWriter(
                new FileOutputStream("utf8.txt"),
                StandardCharsets.UTF_8)) {
            writer.write("Hello, 世界! ¡Hola!");
            System.out.println("UTF-8 file written.");
        } catch (IOException e) {
            System.err.println("Write error: " + e.getMessage());
        }

        // Read with explicit UTF-8 decoding
        try (Reader reader = new InputStreamReader(
                new FileInputStream("utf8.txt"),
                StandardCharsets.UTF_8)) {
            int ch;
            System.out.print("Read: ");
            while ((ch = reader.read()) != -1) {
                System.out.print((char) ch);
            }
            System.out.println();
        } catch (IOException e) {
            System.err.println("Read error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
UTF-8 file written.
Read: Hello, 世界! ¡Hola!
```

**Why This Output Occurs:**
- `OutputStreamWriter` encodes the string as UTF-8 bytes.
- `InputStreamReader` decodes the UTF-8 bytes back into characters .
- The explicit charset ensures correct handling of non-ASCII characters.

### Real-World Cases

- **Web applications**: Reading form data with specified encoding.
- **Data exchange**: Reading CSV/JSON files from different systems.
- **Internationalization**: Supporting multiple languages in a single application.
- **Legacy system integration**: Reading files encoded in ISO-8859-1 or Windows-1252.

### References

- Oracle Help Center – Character Streams (Why use character streams?) - https://docs.oracle.com/javase/8/docs/technotes/guides/io/io.html
- Dev.java – Common I/O Tasks in Modern Java (UTF-8 default) - https://dev.java/learn/modernio/
- Dev.java – Decorating I/O Streams (Bridge classes) - https://dev.java/learn/java-io/reading-writing/decorating/


## Core Concept 4: Resource Lifecycle Management and `try-with-resources` (`AutoCloseable`)

### Definitions

**Core Definition**: **`try-with-resources`** is a `try` statement that declares one or more resources, ensuring each resource is **automatically closed** at the end of the statement, regardless of whether the block completes normally or abruptly.

**Technical Definition**: A **resource** is an object that must be closed after the program is finished with it. Any object implementing `java.lang.AutoCloseable`—which includes all objects implementing `java.io.Closeable`—can be used as a resource . The `try-with-resources` statement guarantees that `close()` is called on each resource in the **reverse order** of their creation . If exceptions are thrown from both the `try` block and the resource closing, the exception from the `try` block is propagated, and the closing exception is **suppressed** .

**Beginner-Friendly Explanation**: Resources like files and network connections are like borrowed tools. You must return them when you're done, even if something goes wrong. `try-with-resources` is like having a responsible assistant who always returns the tools—no matter what happens—so you never accidentally keep a tool (leak a resource).

### Purposes

- To prevent **resource leaks** by guaranteeing `close()` is called .
- To eliminate the verbose and error-prone `finally` block pattern .
- To handle **suppressed exceptions** correctly when both the body and closing fail .
- To simplify code by removing explicit `close()` calls.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
try (ResourceType resource1 = createResource1();
     ResourceType resource2 = createResource2()) {
    // use resources
} catch (ExceptionType e) {
    // handle exceptions
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `try (...)` | Resources declared in parentheses after `try`. |
| `ResourceType` | Must implement `AutoCloseable` . |
| Semicolon | Separates multiple resources. |
| `close()` | Called automatically in reverse order . |

**Syntax Rules:**

- Resources must implement `AutoCloseable` or `Closeable` .
- Multiple resources are separated by semicolons .
- Resources are closed in **reverse order** of declaration .
- If the body and closing both throw exceptions, the body's exception is propagated and the closing exception is suppressed .
- Suppressed exceptions can be retrieved via `Throwable.getSuppressed()`.

**Constraints and Limitations:**

- The resource variable is implicitly `final` (cannot be reassigned).
- Resources must be declared and initialized in the `try` header (or use the Java 9+ effectively final resource syntax).
- `AutoCloseable.close()` throws `Exception`; `Closeable.close()` throws `IOException` .

### Annotated Complete Code Examples

**Example 1: try-with-resources with Multiple Resources**

```java
import java.io.*;

public class TryWithResourcesDemo {
    public static void main(String[] args) {
        // Multiple resources: closed in reverse order
        try (FileInputStream fis = new FileInputStream("input.txt");
             BufferedInputStream bis = new BufferedInputStream(fis);
             FileOutputStream fos = new FileOutputStream("output.txt");
             BufferedOutputStream bos = new BufferedOutputStream(fos)) {

            int data;
            while ((data = bis.read()) != -1) {
                bos.write(data);
            }
            System.out.println("File copied successfully.");
        } catch (IOException e) {
            System.err.println("I/O error: " + e.getMessage());
            // Suppressed exceptions can be inspected:
            for (Throwable t : e.getSuppressed()) {
                System.err.println("Suppressed: " + t.getMessage());
            }
        }
    }
}
```

**Expected Output:**

```
File copied successfully.
```

**Why This Output Occurs:**
- All four streams are declared in the `try-with-resources` header.
- `bis` reads from `fis`; `bos` writes to `fos`.
- When the block exits, `bos`, `fos`, `bis`, and `fis` are closed in that order (reverse of declaration) .
- If `input.txt` doesn't exist, `FileNotFoundException` is caught.

### Real-World Cases

- **Database connections**: `try (Connection conn = ...; Statement stmt = ...)` .
- **File processing**: Reading and writing files with automatic cleanup.
- **Network sockets**: Ensuring sockets are closed after use.
- **Zip file processing**: `try (ZipFile zf = ...)` .

### References

- OpenJDK – The try-with-resources Statement - https://cr.openjdk.org/~naoto/idna2008/docs_www/technotes/guides/language/try-with-resources.html
- Oracle Help Center – The try-with-resources Statement - https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
- Dev.java – Releasing Resources and Catching Exceptions - https://dev.java/learn/java-io/reading-writing/common-operations/


## Core Concept 5: Standard I/O Streams (`System.in`, `System.out`, `System.err`)

### Definitions

**Core Definition**: **Standard I/O streams** are three pre-opened streams provided by the Java platform for command-line interaction: `System.in` (standard input), `System.out` (standard output), and `System.err` (standard error).

**Technical Definition**: The `System` class contains three static fields: `in` (an `InputStream` connected to keyboard input), `out` (a `PrintStream` connected to the display), and `err` (a `PrintStream` connected to error output) . These streams are automatically opened and do not need to be closed (though they can be redirected). `System.out` and `System.err` are `PrintStream` objects that use the platform's default encoding and line separator . `System.in` is a raw byte stream with no character-stream features; to read text, it must be wrapped in an `InputStreamReader` .

**Beginner-Friendly Explanation**: Standard streams are like the default doors of a program. `System.in` is the front door where input comes in (keyboard). `System.out` is the main door where output goes out (screen). `System.err` is a side door specifically for error messages, so you can separate regular output from errors—like having a separate mailbox for bills.

### Purposes

- To provide immediate, pre-configured I/O for command-line programs .
- To allow users to interact with the program via keyboard and screen.
- To separate normal output from error output for logging and redirection .
- To serve as the foundation for console-based applications.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Output
System.out.println("Normal output");  // PrintStream
System.err.println("Error output");   // PrintStream

// Input
int byteValue = System.in.read();     // InputStream, returns -1 at EOF

// Wrapping System.in for character input
BufferedReader reader = new BufferedReader(
    new InputStreamReader(System.in)
);
String line = reader.readLine();
```

**Component Breakdown:**

| Stream | Type | Purpose |
|--------|------|---------|
| `System.in` | `InputStream` | Standard input (keyboard) . |
| `System.out` | `PrintStream` | Standard output (display) . |
| `System.err` | `PrintStream` | Standard error (display) . |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `System.out.print()` | Prints without newline. |
| `System.out.println()` | Prints with newline. |
| `System.err.println()` | Prints error with newline. |
| `System.in.read()` | Reads one byte. |

**Syntax Rules:**

- Standard streams are automatically available; no need to open them .
- `System.out` and `System.err` are `PrintStream` objects .
- `System.in` is a byte stream; wrap in `InputStreamReader` for character input .
- Do not close standard streams (they are managed by the JVM).

**Constraints and Limitations:**

- `System.in.read()` is byte-oriented; for text input, use a `BufferedReader` wrapper.
- `System.out` and `System.err` may interleave unpredictably if used concurrently.
- Standard streams can be redirected using `System.setIn()`, `System.setOut()`, `System.setErr()`.

### Annotated Complete Code Examples

**Example 1: Reading from System.in and Writing to System.out/err**

```java
import java.io.*;

public class StandardIODemo {
    public static void main(String[] args) {
        // Output to standard output
        System.out.println("Enter your name:");

        // Read from standard input (wrap System.in for character input)
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(System.in))) {
            String name = reader.readLine();

            if (name == null || name.trim().isEmpty()) {
                // Output to standard error
                System.err.println("Error: No name provided.");
            } else {
                System.out.println("Hello, " + name + "!");
            }
        } catch (IOException e) {
            System.err.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output (interactive):**

```
Enter your name:
Alice
Hello, Alice!
```

**Why This Output Occurs:**
- `System.out.println()` writes to standard output .
- `System.in` is wrapped in `InputStreamReader` and `BufferedReader` for line reading .
- `System.err.println()` writes to standard error, which can be separately redirected .

### Real-World Cases

- **Command-line tools**: Reading arguments and printing results.
- **Debugging**: Using `System.err` for diagnostic messages.
- **Logging**: Redirecting `System.err` to a log file.
- **Interactive programs**: Reading user input from the console.

### References

- Oracle Help Center – I/O from the Command Line - https://docs.oracle.com/javase/tutorial/essential/io/cl.html
- IIT Bombay – The Standard I/O Streams - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/essential/system/iostreams.html
- Oracle Help Center – コマンドラインを使用したI/O - https://docs.oracle.com/cd/E26537_01/tutorial/essential/io/cl.html


## References

- MIT OpenCourseWare – I/O Lecture Notes - http://ocw.mit.edu/NR/rdonlyres/Civil-and-Environmental-Engineering/1-00Spring-2005/6D9F0160-E067-4D66-91CC-657A8C3BF2BE/0/lec30.pdf
- Oracle Help Center – Character Streams (Java SE 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/io/io.html
- OpenJDK – The try-with-resources Statement - https://cr.openjdk.org/~naoto/idna2008/docs_www/technotes/guides/language/try-with-resources.html
- OpenJDK – AutoCloseable Interface (Java SE 8) - https://cr.openjdk.org/~mr/se/8/java-se-8-pr-spec/apidiffs/java/lang/AutoCloseable-report.html
- Oracle Help Center – I/O from the Command Line - https://docs.oracle.com/javase/tutorial/essential/io/cl.html
- Oracle Help Center – I/O Streams - https://docs.oracle.com/javase/tutorial/essential/io/streams.html
- Dev.java – Common I/O Tasks in Modern Java - https://dev.java/learn/modernio/
- MIT OpenCourseWare – File I/O - https://ocw.mit.edu/courses/1-124j-foundations-of-software-engineering-fall-2000/pages/lecture-notes/file_io/
- IIT Bombay – Overview of I/O Streams - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/essential/io/overview.html
- Oracle Help Center – The try-with-resources Statement - https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
- OpenJDK – AutoCloseable (Java SE 8 PFD) - https://cr.openjdk.org/~mr/se/8/java-se-8-pfd-spec-01/apidiffs/java/lang/AutoCloseable-report.html
- Oracle Help Center – コマンドラインを使用したI/O - https://docs.oracle.com/cd/E26537_01/tutorial/essential/io/cl.html
- Oracle Help Center – Lesson: Basic I/O - https://docs.oracle.com/javase/tutorial/essential/io/
- Dev.java – In Memory IO Streams - https://dev.java/learn/java-io/reading-writing/in-memory/
- MIT OpenCourseWare – What is I/O? - https://dspace.mit.edu/bitstream/handle/1721.1/77247/1-00-fall-2005/contents/study-materials/tutorial12.pdf
- Oracle Help Center – 文字ストリーム - https://docs.oracle.com/javase/jp/6/technotes/guides/io/io.html
- Dev.java – Releasing Resources and Catching Exceptions - https://dev.java/learn/java-io/reading-writing/common-operations/
- OpenJDK – AutoCloseable (JDK 8) - https://cr.openjdk.org/~henryjen/ccc/8025909/0/specdiff/java/lang/AutoCloseable-report.html
- IIT Bombay – The Standard I/O Streams - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/essential/system/iostreams.html
- Dev.java – Decorating I/O Streams - https://dev.java/learn/java-io/reading-writing/decorating/