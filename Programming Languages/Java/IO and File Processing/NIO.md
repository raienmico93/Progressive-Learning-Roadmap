# Java NIO (New I/O): A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Java NIO** (New I/O, also called Non-blocking I/O) is a collection of Java programming language APIs that offer features for intensive I/O operations. It was introduced in Java 1.4 and provides an alternative to the standard `java.io` package, with a focus on **buffer-oriented**, **channel-based** I/O, and **non-blocking** operations.

### Technical Definition

The NIO API is centered on four core abstractions: **Buffers** (containers for data), **Channels** (connections to entities capable of performing I/O), **Selectors** (multiplexors for non-blocking I/O), and the **Path/Files** API for filesystem access . Unlike traditional stream I/O which is byte-oriented and blocking, NIO is **block-oriented** and supports both **non-blocking** and **asynchronous** modes . The `java.nio` package defines the buffer classes, while `java.nio.channels` defines channels and selectors, and `java.nio.file` provides the modern filesystem API .

### Beginner-Friendly Explanation

Traditional I/O is like a single-lane road where only one car (thread) can drive at a time. If the car stops (blocks waiting for data), everything behind it stops too. NIO is like a multi-lane highway with a traffic controller (Selector). One controller can manage many lanes (channels) at once, directing traffic only when cars are ready to move. This makes NIO far more scalable for handling thousands of connections simultaneously.

### Key Characteristics

- **Buffer-oriented**: Data is read into and written from buffers, not processed byte-by-byte .
- **Channel-based**: I/O operations go through channels, which can be bidirectional .
- **Non-blocking mode**: Channels can be put into non-blocking mode, allowing a thread to perform other tasks while waiting for I/O .
- **Selector-based multiplexing**: A single thread can monitor multiple channels via a Selector .
- **Asynchronous I/O**: Operations can complete in the background via Future or CompletionHandler .
- **Memory-mapped files**: Files can be mapped directly into memory for high-performance access .

### Prerequisites

- Basic understanding of Java I/O streams (`InputStream`, `OutputStream`).
- Familiarity with the `try-with-resources` statement.
- Knowledge of the `java.nio.file` package (Path, Files).

### Related Programming Areas

- **High-Performance Servers**: Web servers, proxies, and game servers use NIO for scalability .
- **Asynchronous File I/O**: Non-blocking file operations with `AsynchronousFileChannel` .
- **Memory-Mapped Files**: Ultra-fast large file processing .
- **Reactive Programming**: NIO is the foundation for reactive streams and backpressure .

### Core Concepts / Features

1. `java.nio` Core Architecture
2. Channels (FileChannel, SocketChannel)
3. Buffers (ByteBuffer, CharBuffer) and Block-Oriented Operations
4. Filesystem APIs (`java.nio.file`)
5. Non-Blocking I/O and Asynchronous Channel Operations
6. Memory-Mapped Files (`MappedByteBuffer`)


## Core Concept 1: `java.nio` Core Architecture

### Definitions

**Core Definition**: The `java.nio` package architecture is built around four central abstractions: Buffers (data containers), Charsets (byte-to-character translation), Channels (I/O connections), and Selectors (multiplexed non-blocking I/O) .

**Technical Definition**: The `java.nio` package defines buffer classes used throughout the NIO APIs. The charset API is in `java.nio.charset`, the channel and selector APIs in `java.nio.channels`, and the filesystem APIs in `java.nio.file` . Each subpackage has its own SPI (Service Provider Interface) subpackage for extending or replacing default implementations .

**Beginner-Friendly Explanation**: Think of NIO as a toolbox with four main compartments: one for containers (Buffers), one for translators (Charsets), one for pipes (Channels), and one for traffic controllers (Selectors). Each compartment has its own set of tools.

### Purposes

- To provide a unified, scalable I/O framework for modern applications.
- To enable high-performance, non-blocking I/O without thread-per-connection overhead.
- To support both byte-oriented and character-oriented data processing.
- To offer a modern, cross-platform filesystem API.

### Syntax Rules and Structure

**Package Structure:**

| Package | Purpose |
|---------|---------|
| `java.nio` | Buffers (Buffer, ByteBuffer, CharBuffer, etc.)  |
| `java.nio.channels` | Channels and Selectors  |
| `java.nio.charset` | Character encoding/decoding  |
| `java.nio.file` | Filesystem API (Path, Files)  |

**Core Classes:**

| Class | Description |
|-------|-------------|
| `Buffer` | Container for data of a specific primitive type  |
| `ByteBuffer` | Byte buffer, source/target of I/O operations  |
| `MappedByteBuffer` | Byte buffer mapped to a file  |
| `Channel` | Nexus for I/O operations  |
| `Selector` | Multiplexor of selectable channels  |

**Syntax Rules:**

- Buffers have position, limit, and capacity; operations include clear, flip, rewind, mark/reset .
- Byte buffers can be direct (JVM performs native I/O directly upon them) or non-direct .
- Channels are either open or closed, and are asynchronously closeable and interruptible .
- Selectors work with selectable channels that can be put into non-blocking mode .

**Constraints and Limitations:**

- Buffers are not thread-safe; external synchronization is required for concurrent access.
- Direct buffers have higher allocation cost but lower I/O overhead.
- Selector-based multiplexing is a hint, not a guarantee—operations may still block .

### Annotated Complete Code Examples

**Example 1: NIO Architecture in Action**

```java
import java.nio.*;
import java.nio.channels.*;
import java.nio.file.*;
import java.nio.charset.StandardCharsets;

public class NioArchitectureDemo {
    public static void main(String[] args) throws Exception {
        // Buffer: container for data
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        // Channel: connection for I/O
        Path path = Path.of("nio_demo.txt");
        Files.writeString(path, "Hello, NIO Architecture!", StandardCharsets.UTF_8);

        try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
            // Read from channel into buffer
            int bytesRead = channel.read(buffer);
            System.out.println("Bytes read: " + bytesRead);

            // Flip buffer: prepare for reading (limit = position, position = 0)
            buffer.flip();

            // Decode bytes to characters
            String content = StandardCharsets.UTF_8.decode(buffer).toString();
            System.out.println("Content: " + content);
        }

        Files.delete(path);
    }
}
```

**Expected Output:**

```
Bytes read: 24
Content: Hello, NIO Architecture!
```

**Why This Output Occurs:**
- `ByteBuffer.allocate(1024)` creates a buffer with capacity 1024, position 0, limit 1024 .
- `channel.read(buffer)` reads bytes into the buffer and advances position to 24.
- `buffer.flip()` sets limit to 24 and position to 0, preparing for reading .
- `StandardCharsets.UTF_8.decode(buffer)` converts bytes to a character string.

### Real-World Cases

- **Web servers**: Tomcat, Undertow, and Netty use NIO for scalable request handling .
- **Message brokers**: Kafka uses NIO for high-throughput network communication.
- **File servers**: NIO enables efficient file serving with memory-mapped files.

### References

- OpenJDK – Package java.nio (Java 20 draft) - https://cr.openjdk.org/~iris/se/20/spec/latest/apidiffs/java.base/java/nio/package-summary-report.html 
- Oracle Help Center – Package java.nio (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/nio/package-summary.html 


## Core Concept 2: Channels (FileChannel, SocketChannel)

### Definitions

**Core Definition**: A **Channel** is an abstraction representing a connection to an entity capable of performing I/O operations, such as a file or socket. Unlike streams, channels can be bidirectional and support non-blocking operations .

**Technical Definition**: Channels are defined in `java.nio.channels`. They are either open or closed, and are both asynchronously closeable and interruptible . Key channel types include `FileChannel` (file I/O), `SocketChannel` (stream-oriented connecting sockets), `ServerSocketChannel` (listening sockets), and `DatagramChannel` (UDP) . Channels are created via static `open()` methods .

**Beginner-Friendly Explanation**: A channel is like a two-way pipe that connects your program to a data source or destination. Unlike traditional streams (which are one-way), a channel can both read and write. You can also put a channel in "non-blocking" mode, meaning it won't make your program wait if data isn't ready.

### Purposes

- To provide a bidirectional I/O abstraction for files and network sockets.
- To enable non-blocking and asynchronous I/O operations.
- To support memory-mapped file access via `FileChannel.map()` .
- To facilitate multiplexed I/O through Selectors.

### Syntax Rules and Structure

**Complete General Syntax (FileChannel):**

```java
try (FileChannel channel = FileChannel.open(path, options)) {
    // read/write/map operations
}
```

**Complete General Syntax (SocketChannel):**

```java
try (SocketChannel channel = SocketChannel.open()) {
    channel.connect(new InetSocketAddress("host", port));
    // read/write operations
}
```

**Channel Types:**

| Channel | Description |
|---------|-------------|
| `FileChannel` | Reading, writing, mapping, and manipulating a file  |
| `SocketChannel` | Stream-oriented connecting socket  |
| `ServerSocketChannel` | Stream-oriented listening socket  |
| `DatagramChannel` | UDP packet send/receive  |
| `AsynchronousFileChannel` | Asynchronous file I/O  |

**Syntax Rules:**

- Channels are created via `open()` static methods .
- Channels must be closed to release resources (use `try-with-resources`).
- Selectable channels can be registered with a Selector for non-blocking I/O .
- `FileChannel.map()` creates a `MappedByteBuffer` for memory-mapped access .

**Constraints and Limitations:**

- `FileChannel` is not selectable (cannot be multiplexed with a Selector) .
- Asynchronous channels are non-blocking but do not support memory-mapped I/O .
- Channels are not thread-safe by default; synchronization is needed for concurrent access.

### Annotated Complete Code Examples

**Example 1: FileChannel Read/Write**

```java
import java.nio.*;
import java.nio.channels.*;
import java.nio.file.*;
import java.nio.charset.StandardCharsets;

public class FileChannelDemo {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("channel_demo.txt");

        // Write using FileChannel
        try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {
            ByteBuffer buffer = ByteBuffer.wrap("Hello, Channel!".getBytes(StandardCharsets.UTF_8));
            channel.write(buffer);
            System.out.println("Written: " + buffer.position() + " bytes");
        }

        // Read using FileChannel
        try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int bytesRead = channel.read(buffer);
            buffer.flip();
            String content = StandardCharsets.UTF_8.decode(buffer).toString();
            System.out.println("Read: " + content);
        }

        Files.delete(path);
    }
}
```

**Expected Output:**

```
Written: 15 bytes
Read: Hello, Channel!
```

**Why This Output Occurs:**
- `ByteBuffer.wrap()` creates a buffer backed by the byte array .
- `channel.write(buffer)` writes the buffer's content to the file and advances position.
- `channel.read(buffer)` reads file content into the buffer.
- `buffer.flip()` prepares the buffer for reading .

### Real-World Cases

- **File copying**: `FileChannel.transferTo()` and `transferFrom()` for zero-copy file transfers .
- **Network servers**: `ServerSocketChannel` and `SocketChannel` for scalable network I/O .
- **Memory-mapped databases**: `FileChannel.map()` for high-performance data access .

### References

- Oracle Help Center – Package java.nio.channels (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/package-summary.html 
- Oracle Help Center – Package java.nio.channels (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/channels/compact3-package-summary.html 
- Oracle – Java Core Libraries Developer Guide (Channels) - https://docs.oracle.com/javase/jp/21/core/java-core-libraries-developer-guide.pdf 


## Core Concept 3: Buffers (ByteBuffer, CharBuffer) and Block-Oriented Operations

### Definitions

**Core Definition**: A **Buffer** is a container for a fixed amount of data of a specific primitive type. Buffers are the foundation of NIO's block-oriented I/O model .

**Technical Definition**: Each buffer class (except `Boolean`) extends the abstract `Buffer` class, which defines position, limit, and capacity. Operations include `clear()`, `flip()`, `rewind()`, `mark()`, and `reset()` . `ByteBuffer` is special: it can be the source or target of I/O operations, can be direct (native I/O) or non-direct, and can be memory-mapped . `CharBuffer` provides character-level access .

**Beginner-Friendly Explanation**: A buffer is like a bucket. You fill it with data (write), then flip it over to pour out the data (read). The bucket has a capacity (how much it can hold), a position (where you're filling or pouring), and a limit (how much data is actually in it).

### Purposes

- To provide a fixed-size container for data transfer between channels and applications.
- To enable block-oriented I/O, reducing system call overhead.
- To support direct buffers for high-performance native I/O.
- To provide views for interpreting bytes as other primitive types.

### Syntax Rules and Structure

**Buffer Properties:**

| Property | Description |
|----------|-------------|
| `capacity` | The total number of elements the buffer can hold  |
| `position` | The index of the next element to read or write  |
| `limit` | The index of the first element that should not be read or written  |
| `mark` | A remembered position for later `reset()`  |

**Core Operations:**

| Operation | Description |
|-----------|-------------|
| `clear()` | Sets position to 0, limit to capacity (for writing) |
| `flip()` | Sets limit to position, position to 0 (for reading) |
| `rewind()` | Sets position to 0 (for re-reading) |
| `mark()` | Sets mark at current position |
| `reset()` | Sets position to mark |

**Syntax Rules:**

- `ByteBuffer.allocate(int)` creates a non-direct buffer.
- `ByteBuffer.allocateDirect(int)` creates a direct buffer .
- `ByteBuffer.wrap(byte[])` wraps an existing array .
- `ByteBuffer.asCharBuffer()`, `asIntBuffer()`, etc., create views .
- `buffer.order(ByteOrder.BIG_ENDIAN)` or `LITTLE_ENDIAN` sets byte order .

**Constraints and Limitations:**

- Buffers are not thread-safe.
- Direct buffers have higher allocation/deallocation cost.
- `BufferOverflowException` on put when limit reached; `BufferUnderflowException` on get .

### Annotated Complete Code Examples

**Example 1: Buffer Lifecycle**

```java
import java.nio.ByteBuffer;

public class BufferDemo {
    public static void main(String[] args) {
        // Allocate a buffer with capacity 10
        ByteBuffer buffer = ByteBuffer.allocate(10);
        System.out.println("Initial: pos=" + buffer.position() + ", limit=" + buffer.limit());

        // Write data
        buffer.put((byte) 10);
        buffer.put((byte) 20);
        buffer.put((byte) 30);
        System.out.println("After put: pos=" + buffer.position() + ", limit=" + buffer.limit());

        // Flip: prepare for reading
        buffer.flip();
        System.out.println("After flip: pos=" + buffer.position() + ", limit=" + buffer.limit());

        // Read data
        System.out.println("Read: " + buffer.get());
        System.out.println("Read: " + buffer.get());
        System.out.println("After read: pos=" + buffer.position() + ", limit=" + buffer.limit());

        // Clear: reset for writing
        buffer.clear();
        System.out.println("After clear: pos=" + buffer.position() + ", limit=" + buffer.limit());
    }
}
```

**Expected Output:**

```
Initial: pos=0, limit=10
After put: pos=3, limit=10
After flip: pos=0, limit=3
Read: 10
Read: 20
After read: pos=2, limit=3
After clear: pos=0, limit=10
```

**Why This Output Occurs:**
- `allocate(10)` creates a buffer with capacity 10, position 0, limit 10 .
- `put()` writes bytes and advances position (0 → 1 → 2 → 3).
- `flip()` sets limit to position (3) and position to 0 .
- `get()` reads bytes and advances position (0 → 1 → 2).
- `clear()` resets position to 0 and limit to capacity .

### Real-World Cases

- **Network protocols**: Buffers hold incoming/outgoing packets.
- **File I/O**: Buffers mediate between channels and application data.
- **Serialization**: Buffers hold serialized object bytes.

### References

- Oracle Help Center – Package java.nio (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/nio/package-summary.html 
- OpenJDK – Package java.nio (Java 20 draft) - https://cr.openjdk.org/~iris/se/20/spec/latest/apidiffs/java.base/java/nio/package-summary-report.html 


## Core Concept 4: Filesystem APIs (`java.nio.file`)

### Definitions

**Core Definition**: The `java.nio.file` package provides a modern, feature-rich filesystem API that replaces the legacy `java.io.File` class. It centers on `Path` (location representation) and `Files` (utility operations) .

**Technical Definition**: `FileSystem` is the interface to a filesystem, obtained via `FileSystems.getDefault()`. It provides factory methods for `Path`, `PathMatcher`, `WatchService`, and `FileStore` . `Path` is an immutable, OS-independent representation of a file location. `Files` contains static methods for file operations .

**Beginner-Friendly Explanation**: `Path` is like an address; `Files` is like a moving company that can create, copy, move, and delete things at that address. `FileSystem` is the city—it knows all the addresses and services available.

### Purposes

- To provide a reliable, exception-rich filesystem API.
- To support cross-platform path manipulation.
- To enable efficient directory traversal with Streams.
- To access file metadata and attributes.

### Syntax Rules and Structure

**Core Classes:**

| Class | Description |
|-------|-------------|
| `Path` | Immutable location representation  |
| `Files` | Static utility methods for file operations |
| `FileSystem` | Interface to a filesystem  |
| `FileSystems` | Factory for obtaining `FileSystem` instances  |
| `FileStore` | Storage for files (disk, partition)  |

**Syntax Rules:**

- `Path.of("path/to/file")` creates a Path (Java 11+).
- `Files.createDirectories(path)` creates directory hierarchies.
- `Files.list(path)` returns `Stream<Path>` of direct children.
- `Files.walk(path)` returns `Stream<Path>` recursively.
- `Files.find(path, depth, predicate)` returns filtered `Stream<Path>`.

**Constraints and Limitations:**

- `Files.list()` and `Files.walk()` streams must be closed.
- Symbolic links require explicit handling with `LinkOption.NOFOLLOW_LINKS`.
- Filesystem operations may throw `AccessDeniedException`, `NoSuchFileException`.

### Annotated Complete Code Examples

**Example 1: Filesystem Operations**

```java
import java.nio.file.*;
import java.util.stream.Stream;

public class FileSystemDemo {
    public static void main(String[] args) throws Exception {
        Path dir = Path.of("fs_demo");
        Files.createDirectories(dir);

        Files.writeString(dir.resolve("a.txt"), "Content A");
        Files.writeString(dir.resolve("b.txt"), "Content B");

        System.out.println("Files in directory:");
        try (Stream<Path> entries = Files.list(dir)) {
            entries.forEach(p -> System.out.println("  " + p.getFileName()));
        }

        // Read file content
        String content = Files.readString(dir.resolve("a.txt"));
        System.out.println("a.txt content: " + content);

        // Cleanup
        Files.delete(dir.resolve("a.txt"));
        Files.delete(dir.resolve("b.txt"));
        Files.delete(dir);
    }
}
```

**Expected Output:**

```
Files in directory:
  a.txt
  b.txt
a.txt content: Content A
```

**Why This Output Occurs:**
- `Files.createDirectories()` creates the directory .
- `Files.list(dir)` streams the direct children .
- `Files.readString()` reads the entire file content.

### Real-World Cases

- **Configuration management**: Reading and writing property files.
- **Log processing**: Walking directories to find log files.
- **File synchronization**: Comparing directories with `Files.walk()`.

### References

- Oracle Help Center – FileSystem (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html 
- Oracle Help Center – Package java.nio (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/nio/package-summary.html 


## Core Concept 5: Non-Blocking I/O and Asynchronous Channel Operations

### Definitions

**Core Definition**: **Non-blocking I/O** allows a thread to initiate an I/O operation and immediately continue without waiting for it to complete. **Asynchronous I/O** completes operations in the background and notifies the application via `Future` or `CompletionHandler` .

**Technical Definition**: Non-blocking I/O is achieved through `SelectableChannel` and `Selector`. Channels registered with a Selector can be tested for readiness without blocking . Asynchronous channels (`AsynchronousFileChannel`, `AsynchronousSocketChannel`) return a `Future` or invoke a `CompletionHandler` when the operation completes . Asynchronous channels are bound to an `AsynchronousChannelGroup` with an `ExecutorService` .

**Beginner-Friendly Explanation**: Non-blocking I/O is like ordering food at a counter—you place your order and immediately go sit down. When the food is ready, you get notified. Asynchronous I/O is like having a waiter bring the food to your table when it's ready—you don't even have to check the counter.

### Purposes

- To handle thousands of connections with a small number of threads.
- To improve scalability and resource utilization.
- To enable responsive applications that don't block on I/O.
- To support high-throughput event-driven architectures.

### Syntax Rules and Structure

**Non-Blocking with Selector:**

```java
Selector selector = Selector.open();
channel.configureBlocking(false);
channel.register(selector, SelectionKey.OP_READ);

while (true) {
    selector.select(); // Blocks until a channel is ready
    for (SelectionKey key : selector.selectedKeys()) {
        if (key.isReadable()) {
            // Read from key.channel()
        }
    }
}
```

**Asynchronous Channel:**

```java
AsynchronousFileChannel channel = AsynchronousFileChannel.open(path);
channel.read(buffer, position, attachment, new CompletionHandler<Integer, Object>() {
    public void completed(Integer result, Object attachment) { }
    public void failed(Throwable exc, Object attachment) { }
});
```

**Syntax Rules:**

- `channel.configureBlocking(false)` puts a channel in non-blocking mode .
- `selector.select()` blocks until at least one channel is ready .
- Asynchronous operations return `Future` or accept `CompletionHandler` .
- `AsynchronousChannelGroup` manages thread pools for async channels .

**Constraints and Limitations:**

- Selector readiness is a hint, not a guarantee .
- Asynchronous channels do not support memory-mapped I/O .
- Non-blocking operations may transfer fewer bytes than requested.

### Annotated Complete Code Examples

**Example 1: Non-Blocking Echo Server**

```java
import java.net.*;
import java.nio.*;
import java.nio.channels.*;
import java.util.*;

public class NonBlockingEchoServer {
    public static void main(String[] args) throws Exception {
        ServerSocketChannel server = ServerSocketChannel.open();
        server.bind(new InetSocketAddress(8080));
        server.configureBlocking(false);

        Selector selector = Selector.open();
        server.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("Server listening on port 8080...");

        ByteBuffer buffer = ByteBuffer.allocate(256);

        while (true) {
            selector.select();
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> iter = keys.iterator();

            while (iter.hasNext()) {
                SelectionKey key = iter.next();
                iter.remove();

                if (key.isAcceptable()) {
                    SocketChannel client = server.accept();
                    client.configureBlocking(false);
                    client.register(selector, SelectionKey.OP_READ);
                    System.out.println("Client connected: " + client.getRemoteAddress());
                } else if (key.isReadable()) {
                    SocketChannel client = (SocketChannel) key.channel();
                    buffer.clear();
                    int bytesRead = client.read(buffer);
                    if (bytesRead == -1) {
                        client.close();
                        continue;
                    }
                    buffer.flip();
                    client.write(buffer);
                    buffer.clear();
                }
            }
        }
    }
}
```

**Expected Output (server):**

```
Server listening on port 8080...
Client connected: /127.0.0.1:54321
```

**Why This Output Occurs:**
- `server.configureBlocking(false)` enables non-blocking mode .
- `selector.select()` blocks until a channel is ready .
- `key.isAcceptable()` indicates a new connection; `key.isReadable()` indicates data available.

### Real-World Cases

- **High-performance web servers**: Netty, Undertow, Vert.x .
- **Chat servers**: Handling thousands of concurrent connections.
- **Proxy servers**: Forwarding data between clients and backends.

### References

- Oracle Help Center – Package java.nio.channels (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/package-summary.html 
- Oracle Help Center – Package java.nio.channels (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/package-summary.html 
- Cleverence – Java NIO Deep Dive - https://www.cleverence.com/articles/oracle-documentation/9-java-nio-4827/ 


## Core Concept 6: Memory-Mapped Files (`MappedByteBuffer`)

### Definitions

**Core Definition**: A **`MappedByteBuffer`** is a direct byte buffer whose content is a memory-mapped region of a file. It allows file contents to be accessed as if they were in memory, providing ultra-fast I/O for large files .

**Technical Definition**: `MappedByteBuffer` is created via `FileChannel.map(MapMode, position, size)`. It extends `ByteBuffer` with operations specific to memory-mapped file regions: `isLoaded()`, `load()`, and `force()` . The mapping remains valid until the buffer is garbage-collected. Changes to the mapped file may be visible through the buffer at any time, depending on the OS .

**Beginner-Friendly Explanation**: Imagine a book so large you can't hold it all in your hands. Instead, you get a magic window that shows any page you want, instantly. The window is the `MappedByteBuffer`—it maps a portion of the file into memory, so you can read and write it as if it were a simple array, without loading the entire file.

### Purposes

- To provide ultra-fast access to large files.
- To avoid the overhead of traditional read/write system calls.
- To enable multiple processes to share file data efficiently.
- To support high-performance file processing (search, checksum, indexing) .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
try (FileChannel channel = FileChannel.open(path,
        StandardOpenOption.READ, StandardOpenOption.WRITE)) {
    MappedByteBuffer buffer = channel.map(
        FileChannel.MapMode.READ_WRITE,  // Mode
        0,                                // Position
        channel.size()                    // Size
    );
    // Access buffer as if it were an array
    byte b = buffer.get(0);
    buffer.put(0, (byte) 65);
    buffer.force(); // Write changes to disk
}
```

**Map Modes:**

| Mode | Description |
|------|-------------|
| `READ_ONLY` | Read-only mapping  |
| `READ_WRITE` | Read-write mapping  |
| `PRIVATE` | Copy-on-write mapping |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `isLoaded()` | Checks if content is in physical memory  |
| `load()` | Loads content into physical memory  |
| `force()` | Forces changes to be written to storage  |

**Syntax Rules:**

- `FileChannel.map()` creates the mapping .
- The mapped buffer behaves like a direct `ByteBuffer`.
- `force()` is required to guarantee changes are written to disk .
- The mapping is released when the buffer is garbage-collected.

**Constraints and Limitations:**

- Mapping a file larger than 2 GB requires multiple mappings (int size limit).
- The behavior of concurrent access by other programs is OS-dependent and unspecified .
- If the mapped file is truncated, accessing the buffer may throw an exception .
- `force()` has no effect for read-only mappings .

### Annotated Complete Code Examples

**Example 1: Memory-Mapped File Search**

```java
import java.nio.*;
import java.nio.channels.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;

public class MappedFileDemo {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("mapped.txt");
        Files.writeString(path, "Hello, Memory-Mapped World!", StandardCharsets.UTF_8);

        try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
            long size = channel.size();
            MappedByteBuffer buffer = channel.map(
                FileChannel.MapMode.READ_ONLY, 0, size);

            System.out.println("Mapped size: " + size + " bytes");
            System.out.println("Is loaded: " + buffer.isLoaded());

            // Decode the entire mapped region
            String content = StandardCharsets.UTF_8.decode(buffer).toString();
            System.out.println("Content: " + content);

            // Access individual bytes
            System.out.println("First byte: " + (char) buffer.get(0));
        }

        Files.delete(path);
    }
}
```

**Expected Output:**

```
Mapped size: 28 bytes
Is loaded: true
Content: Hello, Memory-Mapped World!
First byte: H
```

**Why This Output Occurs:**
- `channel.map(READ_ONLY, 0, size)` maps the entire file into memory .
- `isLoaded()` returns `true` if the content is in physical memory .
- `StandardCharsets.UTF_8.decode(buffer)` decodes the mapped bytes.
- `buffer.get(0)` accesses the first byte directly.

### Real-World Cases

- **Large file search**: Searching multi-gigabyte files without loading them .
- **Checksum computation**: Computing file checksums with memory-mapped buffers .
- **Database storage**: Memory-mapped files for fast data access.
- **IPC**: Shared memory between processes via mapped files.

### References

- Oracle Help Center – MappedByteBuffer (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/MappedByteBuffer.html 
- Oracle Help Center – MappedByteBuffer (Java 25) - https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/MappedByteBuffer.html 
- Oracle – Java Core Libraries Developer Guide (Grep NIO Example) - https://docs.oracle.com/javase/jp/21/core/java-core-libraries-developer-guide.pdf 


## References

- OpenJDK – Package java.nio (Java 20 draft) - https://cr.openjdk.org/~iris/se/20/spec/latest/apidiffs/java.base/java/nio/package-summary-report.html 
- Oracle Help Center – Package java.nio.channels (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/package-summary.html 
- Oracle Help Center – Package java.nio (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/nio/package-summary.html 
- Oracle Help Center – FileSystem (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html 
- Oracle Help Center – Package java.nio.channels (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/package-summary.html 
- Oracle Help Center – MappedByteBuffer (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/MappedByteBuffer.html 
- Cleverence – Java NIO Deep Dive - https://www.cleverence.com/articles/oracle-documentation/9-java-nio-4827/ 
- Oracle Help Center – Channels (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Channels.html 
- Oracle Help Center – Package java.nio (Java 8) - https://docs.oracle.com/javase/8/docs/api/java/nio/compact2-package-summary.html 
- Oracle Help Center – FileSystem (Japanese) - https://docs.oracle.com/javase/jp/12/docs/api/java.base/java/nio/file/class-use/FileSystem.html 
- Oracle Help Center – Package java.nio.channels (Java 8 compact3) - https://docs.oracle.com/javase/8/docs/api/java/nio/channels/compact3-package-summary.html 
- IBM Developer – Getting started with new I/O (NIO) - https://developer.ibm.com/tutorials/j-nio/ 
- OpenJDK – java.nio Package Summary - https://cr.openjdk.org/~yan/JDK-8029451/webrev.00/raw_files/old/src/share/classes/java/nio/package.html 
- Oracle Help Center – FileSystem (Japanese) - https://docs.oracle.com/javase/jp/8/docs/api/java/nio/file/FileSystem.html 
- Oracle Help Center – Java I/O, NIO, and NIO.2 - https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html 
- Oracle Help Center – MappedByteBuffer (Java 25) - https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/MappedByteBuffer.html 
- Oracle – Java Core Libraries Developer Guide - https://docs.oracle.com/javase/jp/21/core/java-core-libraries-developer-guide.pdf 