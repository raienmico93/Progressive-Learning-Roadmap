# Java Buffered I/O: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Buffered I/O** in Java is a technique that improves the efficiency of input and output operations by reducing the number of native system calls. Instead of reading or writing data one byte or character at a time directly to the disk or network, buffered streams read from and write to an in-memory **buffer**, invoking the expensive native API only when the buffer is empty (for reading) or full (for writing) .

### Technical Definition

The Java platform implements buffered I/O through four wrapper classes in `java.io`: `BufferedInputStream` and `BufferedOutputStream` for byte streams, and `BufferedReader` and `BufferedWriter` for character streams. These classes wrap an existing unbuffered stream and add an internal buffer array (typically 8 KB by default, though configurable) . The `BufferedReader` class also provides the `readLine()` method for efficient line-by-line text reading . The `Scanner` utility class, part of `java.util`, provides an alternative approach to reading and tokenizing text input, with built-in parsing for primitive types .

### Beginner-Friendly Explanation

Imagine you need to move 1,000 books from one room to another. You could carry one book at a time, walking back and forth 1,000 times—this is unbuffered I/O. Or you could use a cart to carry 50 books at once, making only 20 trips—this is buffered I/O. The cart (buffer) reduces the number of expensive trips (system calls) you make.

### Key Characteristics

- **Reduced system calls**: Buffering minimizes the number of expensive native I/O operations .
- **Wrapper pattern**: Buffered streams wrap existing unbuffered streams .
- **Configurable buffer size**: The default is usually sufficient, but you can specify a custom size .
- **Line-oriented reading**: `BufferedReader.readLine()` enables convenient line-by-line text processing .
- **Flushing support**: Buffered output streams can be manually or automatically flushed .
- **Tokenizing with Scanner**: `Scanner` provides parsing and tokenization as an alternative to raw buffered reading .

### Prerequisites

- Basic understanding of `InputStream`, `OutputStream`, `Reader`, and `Writer`.
- Familiarity with the decorator/wrapper pattern.
- Knowledge of `try-with-resources` for resource management.

### Related Programming Areas

- **File Processing**: Reading and writing files efficiently.
- **Network I/O**: Buffering socket communication.
- **Text Processing**: Line-by-line reading with `BufferedReader`.
- **Input Parsing**: Using `Scanner` for structured input.

### Core Concepts / Features

1. Buffered Input (`BufferedReader`, `BufferedInputStream`)
2. Buffered Output (`BufferedWriter`, `BufferedOutputStream`)
3. Efficient File Processing and Buffering Mechanics
4. Tokenizing Text Inputs with `Scanner`


## Core Concept 1: Buffered Input (`BufferedReader`, `BufferedInputStream`)

### Definitions

**Core Definition**: `BufferedReader` and `BufferedInputStream` are wrapper classes that add buffering to character and byte input streams respectively, reducing the frequency of native read operations.

**Technical Definition**: `BufferedInputStream` adds functionality to another input stream—namely, the ability to buffer the input and to support the `mark` and `reset` methods. When created, an internal buffer array is allocated. As bytes are read or skipped, the buffer is refilled as necessary from the contained input stream, many bytes at a time . `BufferedReader` reads text from a character-input stream, buffering characters to provide efficient reading of characters, arrays, and lines. The `readLine()` method reads a line of text, returning `null` at end of stream .

**Beginner-Friendly Explanation**: A buffered input stream is like having a pantry next to your kitchen. Instead of going to the grocery store (disk) for every single ingredient (byte), you stock up a bunch at once (buffer) and take what you need from the pantry.

### Purposes

- To reduce the number of expensive native read operations .
- To provide efficient line-by-line reading through `readLine()` .
- To support mark and reset operations for lookahead parsing .
- To enable efficient reading of characters, arrays, and lines .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Byte input
BufferedInputStream bis = new BufferedInputStream(
    new FileInputStream("data.bin")
);

// Character input
BufferedReader br = new BufferedReader(
    new FileReader("text.txt")
);

// With custom buffer size
BufferedReader br2 = new BufferedReader(
    new FileReader("text.txt"), 16384  // 16 KB buffer
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BufferedInputStream(InputStream)` | Wraps a byte input stream . |
| `BufferedReader(Reader)` | Wraps a character input stream . |
| `BufferedReader(Reader, int)` | Specifies custom buffer size . |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `int read()` | Reads a single byte/character, returns -1 at EOF . |
| `int read(char[] cbuf)` | Reads into a char array . |
| `String readLine()` | Reads a line of text (BufferedReader only) . |
| `void mark(int)` | Marks a position for later reset . |
| `void reset()` | Returns to the marked position . |
| `void close()` | Closes the stream and releases resources . |

**Syntax Rules:**

- Always wrap a `BufferedReader` around any `Reader` whose `read()` operations may be costly, such as `FileReader` and `InputStreamReader` .
- `readLine()` returns `null` at end of stream .
- `BufferedReader` supports `mark()` and `reset()` .
- The default buffer size is large enough for most purposes .

**Constraints and Limitations:**

- `readLine()` strips the line terminator; it does not return it.
- The `mark()` read-ahead limit may cause buffer reallocation if exceeded .
- Buffering adds a small memory overhead for the buffer array.

### Annotated Complete Code Examples

**Example 1: Reading a File Line by Line**

```java
import java.io.*;

public class BufferedReadDemo {
    public static void main(String[] args) {
        // Create a file for demonstration
        try (Writer writer = new FileWriter("sample.txt")) {
            writer.write("Line 1: Hello\n");
            writer.write("Line 2: Buffered I/O\n");
            writer.write("Line 3: Efficient reading\n");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Read line by line with BufferedReader
        try (BufferedReader br = new BufferedReader(new FileReader("sample.txt"))) {
            String line;
            int lineNumber = 1;
            while ((line = br.readLine()) != null) {
                System.out.println("Line " + lineNumber + ": " + line);
                lineNumber++;
            }
        } catch (IOException e) {
            System.err.println("Error reading: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Line 1: Line 1: Hello
Line 2: Line 2: Buffered I/O
Line 3: Line 3: Efficient reading
```

**Why This Output Occurs:**
- `BufferedReader` wraps the `FileReader` and buffers the input .
- `readLine()` reads one line at a time, returning `null` at EOF .
- The buffer reduces the number of underlying read operations .

### Real-World Cases

- **Log file analysis**: Reading and processing log files line by line.
- **CSV parsing**: Reading comma-separated values row by row.
- **Configuration files**: Reading properties and INI files.
- **Network protocols**: Buffering socket input for efficient parsing.

### References

- Oracle Help Center – Buffered Streams - https://docs.oracle.com/javase/tutorial/essential/io/buffers.html 
- Oracle Help Center – BufferedReader (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html 
- Oracle Help Center – BufferedInputStream (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/BufferedInputStream.html 


## Core Concept 2: Buffered Output (`BufferedWriter`, `BufferedOutputStream`)

### Definitions

**Core Definition**: `BufferedWriter` and `BufferedOutputStream` are wrapper classes that add buffering to character and byte output streams, accumulating data in memory before writing to the underlying stream.

**Technical Definition**: `BufferedWriter` writes text to a character-output stream, buffering characters to provide efficient writing of single characters, arrays, and strings . `BufferedOutputStream` implements a buffered output stream, where data is written to a buffer and the native output API is called only when the buffer is full . Both classes support a `flush()` method that forces the buffer's contents to be written immediately .

**Beginner-Friendly Explanation**: A buffered output stream is like a shopping cart. Instead of making a trip to the store (disk) for every item you buy, you fill your cart and make one trip when it's full (or when you're ready to check out with `flush()`).

### Purposes

- To reduce the number of expensive native write operations .
- To batch small writes into larger, more efficient operations .
- To provide efficient writing of characters, strings, and arrays .
- To support manual flushing for timely output .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Byte output
BufferedOutputStream bos = new BufferedOutputStream(
    new FileOutputStream("output.bin")
);

// Character output
BufferedWriter bw = new BufferedWriter(
    new FileWriter("output.txt")
);

// With custom buffer size
BufferedWriter bw2 = new BufferedWriter(
    new FileWriter("output.txt"), 16384
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BufferedOutputStream(OutputStream)` | Wraps a byte output stream . |
| `BufferedWriter(Writer)` | Wraps a character output stream . |
| `flush()` | Forces buffered data to be written . |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `void write(int b)` | Writes a single byte/character. |
| `void write(char[] cbuf)` | Writes a character array. |
| `void write(String str)` | Writes a string (BufferedWriter) . |
| `void newLine()` | Writes a line separator (BufferedWriter). |
| `void flush()` | Flushes the buffer . |
| `void close()` | Flushes and closes the stream. |

**Syntax Rules:**

- Buffered output streams write to a buffer and call the native API only when the buffer is full .
- `flush()` forces buffered data to be written immediately .
- Some buffered output classes support autoflush, specified by an optional constructor argument .
- Always call `flush()` or `close()` to ensure data is written before the program ends.

**Constraints and Limitations:**

- Data may be lost if the program exits without flushing or closing.
- Autoflush can reduce the benefits of buffering if used excessively.
- `flush()` on an unbuffered stream has no effect .

### Annotated Complete Code Examples

**Example 1: Writing with BufferedWriter**

```java
import java.io.*;

public class BufferedWriteDemo {
    public static void main(String[] args) {
        // Write with BufferedWriter
        try (BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"))) {
            bw.write("First line");
            bw.newLine();
            bw.write("Second line");
            bw.newLine();
            bw.write("Third line");
            // Buffer is automatically flushed and closed by try-with-resources
            System.out.println("Writing complete.");
        } catch (IOException e) {
            System.err.println("Error writing: " + e.getMessage());
        }

        // Demonstrate flush()
        try (BufferedWriter bw = new BufferedWriter(new FileWriter("flush_demo.txt"))) {
            bw.write("This is buffered");
            bw.flush(); // Force write to disk
            System.out.println("Flushed to disk.");
            bw.write(" - More text");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**Expected Output:**

```
Writing complete.
Flushed to disk.
```

**Why This Output Occurs:**
- `BufferedWriter` accumulates data in memory .
- `flush()` forces the buffer contents to be written to the file immediately .
- `try-with-resources` ensures `close()` is called, which flushes and closes.

### Real-World Cases

- **Log writing**: Efficiently appending log entries.
- **Report generation**: Writing large text reports.
- **Data export**: Writing CSV or JSON files.
- **Network communication**: Buffering socket output.

### References

- Oracle Help Center – Buffered Streams (Flushing) - https://docs.oracle.com/javase/tutorial/essential/io/buffers.html 
- Max-Planck-Gesellschaft – java.io Package Summary (BufferedWriter) - https://resources.mpi-inf.mpg.de/departments/d5/teaching/ss05/is05/java/IRDMSearch/javadoc/java/io/package-summary.html 


## Core Concept 3: Efficient File Processing and Buffering Mechanics

### Definitions

**Core Definition**: **Buffering mechanics** refer to how buffered streams reduce native I/O calls by batching data into larger chunks, and the performance implications of this approach.

**Technical Definition**: Most examples of unbuffered I/O handle each read or write request directly through the underlying OS. This can make a program much less efficient, since each such request often triggers disk access, network activity, or some other operation that is relatively expensive . Buffered input streams read data from a memory area known as a buffer; the native input API is called only when the buffer is empty. Similarly, buffered output streams write data to a buffer, and the native output API is called only when the buffer is full . Performance tests show dramatic improvements: raw file streams took 10,800 ms to copy a test file, while buffered streams took only 130 ms .

**Beginner-Friendly Explanation**: Without buffering, every time you want a sip of water, you walk to the kitchen, fill a glass, drink, and walk back. With buffering, you fill a pitcher once and pour from it as needed. The pitcher is your buffer—it reduces the number of trips to the kitchen (disk access).

### Purposes

- To dramatically improve I/O performance by reducing system calls .
- To batch small reads/writes into larger, more efficient operations .
- To reduce disk access, network activity, and other expensive operations .
- To provide predictable performance for file processing.

### Syntax Rules and Structure

**Buffering Mechanics:**

| Aspect | Unbuffered | Buffered |
|--------|-----------|----------|
| Read operation | Native API call per byte | Native API call when buffer empty  |
| Write operation | Native API call per byte | Native API call when buffer full  |
| Typical buffer size | N/A | 8 KB default  |
| Performance (copy test) | 10,800 ms | 130 ms  |

**Syntax Rules:**

- Wrap unbuffered streams with buffered wrappers for efficiency .
- For file copying, use `BufferedInputStream` and `BufferedOutputStream` .
- For character data, use `BufferedReader` and `BufferedWriter` .
- Consider custom buffer sizes for specialized workloads .

**Constraints and Limitations:**

- Buffering adds memory overhead for the buffer array.
- The performance benefit diminishes for very large reads/writes (already batched).
- Custom buffering may outperform built-in buffering in some cases .

### Annotated Complete Code Examples

**Example 1: Efficient File Copy**

```java
import java.io.*;

public class EfficientCopyDemo {
    public static void main(String[] args) throws IOException {
        // Create a test file
        byte[] data = new byte[1024 * 1024]; // 1 MB
        for (int i = 0; i < data.length; i++) {
            data[i] = (byte) (i % 256);
        }
        try (FileOutputStream fos = new FileOutputStream("source.bin")) {
            fos.write(data);
        }

        // Copy with buffered streams
        long start = System.nanoTime();
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("source.bin"));
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("dest.bin"))) {
            int b;
            while ((b = bis.read()) != -1) {
                bos.write(b);
            }
        }
        long bufferedTime = System.nanoTime() - start;

        System.out.println("Buffered copy completed in " + (bufferedTime / 1_000_000) + " ms");
        System.out.println("Files match: " + (new File("source.bin").length() == new File("dest.bin").length()));
    }
}
```

**Expected Output:**

```
Buffered copy completed in X ms
Files match: true
```

**Why This Output Occurs:**
- `BufferedInputStream` reads a chunk of data into its buffer, then serves `read()` calls from memory .
- `BufferedOutputStream` accumulates written bytes and flushes to disk when the buffer is full .
- The number of native disk operations is dramatically reduced compared to unbuffered copying .

### Real-World Cases

- **Large file copying**: Using buffered streams for efficient bulk data transfer.
- **Database export/import**: Writing/reading large data files.
- **Media processing**: Reading and writing binary media files.
- **Backup tools**: Efficiently copying directory trees.

### References

- Oracle Help Center – Buffered Streams - https://docs.oracle.com/javase/tutorial/essential/io/buffers.html 
- Oracle – Java Performance Chapter 4 (Buffering Performance) - http://java.sun.com/developer/Books/performance/ch04.pdf 
- Oracle Help Center – BufferedReader (Default Buffer Size) - https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html 


## Core Concept 4: Tokenizing Text Inputs with `Scanner`

### Definitions

**Core Definition**: `Scanner` is a utility class in `java.util` that parses primitive types and strings from input sources using delimiter-based tokenization.

**Technical Definition**: `Scanner` breaks its input into tokens using a delimiter pattern, which by default matches whitespace. The resulting tokens may then be converted into values of different types using the various `next` methods . `Scanner` can read from `InputStream`, `File`, `Path`, `Readable`, `String`, or `ReadableByteChannel` . It provides `hasNext` methods for checking token availability and type compatibility before reading .

**Beginner-Friendly Explanation**: `Scanner` is like a smart reader that automatically separates words and numbers for you. If you give it the text "John 25 3.14", it can give you "John" as a String, 25 as an int, and 3.14 as a double—all without you writing parsing code.

### Purposes

- To read and parse primitive types directly from input .
- To tokenize text using custom delimiter patterns .
- To provide convenient input parsing for command-line and file-based applications .
- To offer `hasNext` methods for safe, type-aware reading .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// From System.in
Scanner sc = new Scanner(System.in);

// From a file
Scanner fileSc = new Scanner(new File("data.txt"));

// From a string
Scanner strSc = new Scanner("10 20 30");

// With custom delimiter
Scanner csvSc = new Scanner("a,b,c").useDelimiter(",");
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Scanner(InputStream)` | Reads from input stream . |
| `Scanner(File)` | Reads from file . |
| `Scanner(String)` | Reads from string . |
| `useDelimiter(String)` | Sets custom delimiter pattern . |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `boolean hasNext()` | Checks if another token exists . |
| `boolean hasNextInt()` | Checks if next token can be read as int . |
| `String next()` | Returns the next token as String . |
| `int nextInt()` | Reads next token as int . |
| `double nextDouble()` | Reads next token as double . |
| `String nextLine()` | Advances past current line and returns it . |

**Syntax Rules:**

- The default delimiter is whitespace .
- Use `hasNextX()` before `nextX()` to avoid `InputMismatchException`.
- `Scanner` implements `Closeable` and should be closed when done .
- `nextLine()` returns the input skipped by the current line, which may be empty if the cursor is already at a line boundary .

**Constraints and Limitations:**

- `Scanner` is not thread-safe.
- The default delimiter (whitespace) may not be suitable for all input formats.
- `nextInt()` followed by `nextLine()` can cause issues because `nextInt()` leaves the newline character unconsumed.

### Annotated Complete Code Examples

**Example 1: Parsing Mixed Input with Scanner**

```java
import java.util.Scanner;
import java.io.File;
import java.io.FileWriter;

public class ScannerDemo {
    public static void main(String[] args) throws Exception {
        // Create a data file
        try (FileWriter fw = new FileWriter("data.txt")) {
            fw.write("Alice 30 3.14\n");
            fw.write("Bob 25 2.71\n");
            fw.write("Charlie 35 1.41\n");
        }

        // Read and parse with Scanner
        try (Scanner sc = new Scanner(new File("data.txt"))) {
            while (sc.hasNext()) {
                String name = sc.next();       // String token
                int age = sc.nextInt();        // int token
                double value = sc.nextDouble(); // double token
                System.out.printf("%s is %d years old, value=%.2f%n", name, age, value);
            }
        }
    }
}
```

**Expected Output:**

```
Alice is 30 years old, value=3.14
Bob is 25 years old, value=2.71
Charlie is 35 years old, value=1.41
```

**Why This Output Occurs:**
- `Scanner` tokenizes the input by whitespace .
- `next()`, `nextInt()`, and `nextDouble()` read and convert tokens to the appropriate types .
- `hasNext()` checks for more tokens before reading .

### Real-World Cases

- **Command-line applications**: Reading user input with type validation.
- **CSV/TSV parsing**: Reading delimited data files.
- **Configuration parsing**: Reading settings from files.
- **Interactive programs**: Reading mixed input types from users.

### References

- Oracle Help Center – Scanner (Java 23) - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/util/Scanner.html 
- Oracle Help Center – Scanner (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html 
- Oracle Help Center – Scanner (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/util/Scanner.html 
- Android Developers – Scanner - https://android-dot-devsite-v2-prod.appspot.com/reference/java/util/Scanner.html 


## References

- Oracle Help Center – Buffered Streams - https://docs.oracle.com/javase/tutorial/essential/io/buffers.html 
- Oracle – Java Performance Chapter 4 (Buffering Performance) - http://java.sun.com/developer/Books/performance/ch04.pdf 
- Oracle Help Center – BufferedReader (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html 
- Oracle Help Center – BufferedInputStream (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/BufferedInputStream.html 
- Oracle Help Center – Scanner (Java 23) - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/util/Scanner.html 
- Oracle Help Center – Scanner (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html 
- Oracle Help Center – Scanner (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/util/Scanner.html 
- Max-Planck-Gesellschaft – java.io Package Summary - https://resources.mpi-inf.mpg.de/departments/d5/teaching/ss05/is05/java/IRDMSearch/javadoc/java/io/package-summary.html 
- Android Developers – Scanner - https://android-dot-devsite-v2-prod.appspot.com/reference/java/util/Scanner.html 
- Oracle Help Center – BufferedInputStream (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/BufferedInputStream.html 