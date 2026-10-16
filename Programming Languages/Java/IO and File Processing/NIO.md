# Java NIO (New I/O): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Java NIO (New I/O), introduced in Java 1.4, is a collection of Java programming language APIs that offer features for intensive I/O operations. It provides a scalable, high-performance I/O model based on buffers, channels, and selectors.

**Technical Definition:** "The `java.nio` package provides the New I/O (NIO) APIs introduced in Java 1.4, offering scalable, high-performance I/O operations. The package includes buffers for efficient data handling, channels for I/O operations, and selectors for multiplexed, non-blocking I/O". NIO is built around three core abstractions: **Buffers** (containers for data of specific primitive types), **Channels** (connections to I/O devices such as files and sockets), and **Selectors** (multiplexing of non-blocking channels). Unlike classic I/O, which is stream-oriented and blocking, NIO is buffer-oriented and can operate in non-blocking mode.

**Beginner-Friendly Explanation:** Classic Java I/O works like a water pipe — you read or write one byte at a time, and the thread waits (blocks) until data is available. NIO is more like a conveyor belt system: data is loaded into containers (buffers), moved through channels, and a single supervisor (selector) can watch many conveyor belts at once. This makes NIO much more efficient when your program needs to handle thousands of connections or process large files. Think of classic I/O as a single-lane road and NIO as a multi-lane highway with a traffic controller.

### Key Characteristics

- **Buffer-Oriented:** Data is read into and written from buffers, not directly from streams.
- **Non-Blocking:** Channels can be configured to operate in non-blocking mode, allowing a thread to continue working while waiting for I/O.
- **Multiplexed:** A single thread can monitor multiple channels using a `Selector`.
- **Channel-Based:** Channels represent connections to I/O sources and sinks (files, sockets, datagrams).
- **Direct Buffers:** Byte buffers can be allocated in native memory for faster I/O with the operating system.
- **Memory-Mapped Files:** Files can be mapped directly into memory via `MappedByteBuffer`.
- **Zero-Copy Transfers:** Data can be transferred directly between channels and files without passing through user space.
- **NIO.2 (Java 7+):** The `java.nio.file` package adds the `Path` interface, the `Files` utility class, asynchronous channels, and watch services.

### Prerequisites

- Solid understanding of Java classes, interfaces, and methods
- Familiarity with exceptions and exception handling
- Basic knowledge of file I/O using `java.io`
- Understanding of threads and concurrency concepts

### Related Programming Areas

- **High-Performance Networking:** NIO is the foundation for frameworks like Netty, which power high-throughput servers.
- **File Processing:** Memory-mapped files and `FileChannel` enable efficient large-file operations.
- **Async I/O:** `AsynchronousFileChannel` and `AsynchronousSocketChannel` provide callback-based I/O.
- **NIO.2:** The `java.nio.file` package offers a modern file API with `Path`, `Files`, and directory traversal.
- **Zero-Copy:** `FileChannel.transferTo()` and `transferFrom()` enable direct data transfer.

### Core Concepts / Features

---

## 1. NIO (New I/O)

### Definitions

**Core Definition:** NIO is Java's scalable, non-blocking I/O API introduced in Java 1.4, providing an alternative to the classic stream-based I/O model.

**Technical Definition:** "Java NIO (New I/O) introduced a lower-level, more scalable model for I/O. Instead of read/write stream calls that block a thread until data arrives, NIO centers on channels, buffers, and selectors". The `java.nio` package includes buffers for efficient data handling, channels for I/O operations, and selectors for multiplexed, non-blocking I/O.

**Beginner-Friendly Explanation:** NIO is a more modern way to do I/O in Java. While classic I/O makes your program wait for each read or write to finish, NIO lets your program do other work while waiting, making it much faster for applications that handle many connections or large amounts of data.

### Purposes

- To provide scalable I/O for high-concurrency applications
- To enable non-blocking I/O operations
- To support multiplexed I/O through selectors
- To enable memory-mapped file access for high-performance file processing
- To support zero-copy data transfers between channels and files

### Syntax Rules and Structure

**NIO Architecture:**

```
Java NIO
├── Channels (FileChannel, SocketChannel, ServerSocketChannel, DatagramChannel)
├── Buffers (ByteBuffer, CharBuffer, IntBuffer, LongBuffer, FloatBuffer, DoubleBuffer, ShortBuffer, MappedByteBuffer)
├── Selectors (Selector, SelectionKey)
└── Pipe (SinkChannel, SourceChannel)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Channels | Transport mechanisms for data between buffers and sources/destinations |
| Buffers | Containers for data that is being processed |
| Selectors | Allow a single thread to monitor multiple channels for events |
| Pipe | A pair of channels implementing a unidirectional pipe |

### Annotated Complete Step-by-Step Code Examples

**Example 1: NIO File Read and Write**

```java
// File: NioBasicDemo.java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

public class NioBasicDemo {
    public static void main(String[] args) {
        Path path = Path.of("nio_example.txt");

        // Write data using NIO
        try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.CREATE,
                StandardOpenOption.WRITE)) {

            // Create a buffer and write data into it
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put("Hello, NIO!".getBytes());

            // Flip the buffer from writing mode to reading mode
            buffer.flip();

            // Write buffer contents to the channel
            channel.write(buffer);
            System.out.println("Data written to file");

        } catch (IOException e) {
            System.out.println("Write error: " + e.getMessage());
        }

        // Read data using NIO
        try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            // Read data from channel into buffer
            int bytesRead = channel.read(buffer);
            System.out.println("Bytes read: " + bytesRead);

            // Flip and read the data
            buffer.flip();
            byte[] data = new byte[buffer.remaining()];
            buffer.get(data);
            System.out.println("Content: " + new String(data));

        } catch (IOException e) {
            System.out.println("Read error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Data written to file
Bytes read: 11
Content: Hello, NIO!
```

**Step-by-Step Explanation:**

1. `FileChannel.open()` opens a channel to the file with create and write options.
2. `ByteBuffer.allocate(1024)` creates a heap buffer with 1024 bytes of capacity.
3. `buffer.put(bytes)` writes data into the buffer in writing mode.
4. `buffer.flip()` switches the buffer from writing mode to reading mode, setting the limit to the current position and position to 0.
5. `channel.write(buffer)` writes the buffer contents to the file.
6. For reading, `channel.read(buffer)` reads from the file into the buffer.
7. `buffer.flip()` prepares the buffer for reading.
8. `buffer.get(data)` extracts the data into a byte array.

### Real-World Cases

- **High-Performance Web Servers:** NIO enables servers to handle thousands of concurrent connections with fewer threads.
- **File Processing:** Memory-mapped files allow fast random access to large files.
- **Message Brokers:** NIO is used in systems like Apache Kafka for high-throughput data transfer.
- **Proxies and Gateways:** NIO's event-driven model is ideal for network proxies.

---

## 2. Channels

### Definitions

**Core Definition:** A channel is an abstraction representing an open connection to an entity (such as a file or socket) that is capable of performing I/O operations.

**Technical Definition:** "Channels represent connections to entities capable of I/O operations (files, sockets, etc.). Unlike streams, channels can be non-blocking and work directly with buffers". The `java.nio.channels` package "defines channels, which represent connections to entities that are capable of performing I/O operations, such as files and sockets; defines selectors, for multiplexed, non-blocking I/O operations".

**Beginner-Friendly Explanation:** A channel is like a pipe that connects your program to a file or a network socket. You read data from a channel into a buffer, or write data from a buffer into a channel. Unlike streams, channels can be non-blocking, meaning they don't make your program wait.

### Purposes

- To provide a uniform abstraction for I/O operations on different types of sources and destinations
- To enable non-blocking I/O operations
- To support reading and writing data through buffers
- To serve as the foundation for selectors and multiplexed I/O

### Syntax Rules and Structure

**Channel Hierarchy:**

```
Channel (interface)
├── ReadableByteChannel
├── WritableByteChannel
├── ByteChannel
├── ScatteringByteChannel
├── GatheringByteChannel
├── SeekableByteChannel
├── InterruptibleChannel
├── NetworkChannel
└── MulticastChannel
```

**Common Channel Implementations:**

| Channel | Description |
|---------|-------------|
| `FileChannel` | Reads, writes, maps, and manipulates files |
| `SocketChannel` | A selectable channel for stream-oriented connecting sockets |
| `ServerSocketChannel` | A selectable channel for stream-oriented listening sockets |
| `DatagramChannel` | A selectable channel for datagram-oriented sockets |
| `AsynchronousFileChannel` | An asynchronous channel for reading, writing, and manipulating files |

**Syntax Rules:**

- Channels implement `AutoCloseable` and should be closed with try-with-resources.
- Channels work with buffers: `channel.read(buffer)` and `channel.write(buffer)`.
- Network channels can be configured as non-blocking with `configureBlocking(false)`.

### Annotated Complete Step-by-Step Code Examples

**Example 1: FileChannel Read and Write**

```java
// File: FileChannelDemo.java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

public class FileChannelDemo {
    public static void main(String[] args) {
        Path source = Path.of("source.txt");
        Path dest = Path.of("dest.txt");

        try (FileChannel in = FileChannel.open(source,
                    StandardOpenOption.CREATE,
                    StandardOpenOption.WRITE,
                    StandardOpenOption.READ);
             FileChannel out = FileChannel.open(dest,
                    StandardOpenOption.CREATE,
                    StandardOpenOption.WRITE)) {

            // Write some data to source
            ByteBuffer writeBuffer = ByteBuffer.wrap("FileChannel test data".getBytes());
            in.write(writeBuffer);

            // Reset position for reading
            in.position(0);

            // Transfer data directly using zero-copy
            long transferred = in.transferTo(0, in.size(), out);
            System.out.println("Transferred " + transferred + " bytes");

            // Read back from dest to verify
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            out.position(0);
            out.read(readBuffer);
            readBuffer.flip();
            byte[] data = new byte[readBuffer.remaining()];
            readBuffer.get(data);
            System.out.println("Content: " + new String(data));

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Transferred 22 bytes
Content: FileChannel test data
```

**Step-by-Step Explanation:**

1. `FileChannel.open()` opens channels to source and destination files.
2. `ByteBuffer.wrap()` creates a buffer from a byte array.
3. `in.write(writeBuffer)` writes the buffer contents to the source file.
4. `in.position(0)` resets the channel position to the beginning.
5. `in.transferTo(0, in.size(), out)` uses zero-copy transfer to move data directly from the source channel to the destination channel.
6. The destination file is read back to verify the transfer.

### Real-World Cases

- **File Copying:** Using `transferTo()` for efficient file copies.
- **Network Servers:** `ServerSocketChannel` accepts connections; `SocketChannel` communicates with clients.
- **UDP Communication:** `DatagramChannel` for connectionless datagram protocols.
- **Memory-Mapped Files:** `FileChannel.map()` for high-performance file access.

---

## 3. Buffers

### Definitions

**Core Definition:** A buffer is a container for a fixed amount of data of a specific primitive type, used for reading from and writing to channels.

**Technical Definition:** "The abstract `Buffer` class is the foundation for all buffer types. A buffer is a linear, finite sequence of elements of a specific primitive type". Every buffer has four essential properties: **Capacity** (the number of elements it contains, never changes), **Limit** (the index of the first element that should not be read or written), **Position** (the index of the next element to be read or written), and **Mark** (the index to which position will be reset when `reset()` is called). The invariant is: `0 <= mark <= position <= limit <= capacity`.

**Beginner-Friendly Explanation:** A buffer is like a cup that holds data. It has a fixed size (capacity). You fill it with data (writing mode), then flip it to pour out the data (reading mode). The position tells you where you are in the cup, and the limit tells you how much data is actually in the cup.

### Purposes

- To provide a fixed-size container for reading from and writing to channels
- To manage the flow of data between channels and the application
- To enable efficient bulk data transfer
- To support different primitive data types through specialised buffer classes

### Syntax Rules and Structure

**Buffer Types:**

| Buffer Class | Data Type |
|--------------|-----------|
| `ByteBuffer` | `byte` |
| `CharBuffer` | `char` |
| `ShortBuffer` | `short` |
| `IntBuffer` | `int` |
| `LongBuffer` | `long` |
| `FloatBuffer` | `float` |
| `DoubleBuffer` | `double` |
| `MappedByteBuffer` | `byte` (memory-mapped) |

**Buffer Operations:**

| Method | Description |
|--------|-------------|
| `put(data)` | Writes data into the buffer (writing mode). |
| `get()` | Reads data from the buffer (reading mode). |
| `flip()` | Prepares for reading: `limit = position, position = 0`. |
| `clear()` | Prepares for writing: `position = 0, limit = capacity`. |
| `rewind()` | Prepares for re-reading: `position = 0, limit unchanged`. |
| `compact()` | Prepares for writing after partial read. |
| `mark()` | Sets mark at current position. |
| `reset()` | Resets position to previously marked position. |

**Syntax Rules:**

- Buffers are created via `allocate()` (heap) or `allocateDirect()` (direct/native memory).
- The `flip()` method is essential when switching from writing to reading.
- The `clear()` method is used when reusing a buffer for writing.

### Annotated Complete Step-by-Step Code Examples

**Example 1: ByteBuffer Lifecycle**

```java
// File: BufferLifecycleDemo.java
import java.nio.ByteBuffer;

public class BufferLifecycleDemo {
    public static void main(String[] args) {
        // Create a heap buffer with 10 bytes capacity
        ByteBuffer buffer = ByteBuffer.allocate(10);

        System.out.println("After allocate: position=" + buffer.position() +
            ", limit=" + buffer.limit() + ", capacity=" + buffer.capacity());

        // Write data into the buffer
        buffer.put((byte) 65);  // 'A'
        buffer.put((byte) 66);  // 'B'
        buffer.put((byte) 67);  // 'C'

        System.out.println("After put: position=" + buffer.position() +
            ", limit=" + buffer.limit());

        // Flip to reading mode
        buffer.flip();
        System.out.println("After flip: position=" + buffer.position() +
            ", limit=" + buffer.limit());

        // Read data
        while (buffer.hasRemaining()) {
            System.out.println("Read: " + (char) buffer.get());
        }

        // Clear to reuse
        buffer.clear();
        System.out.println("After clear: position=" + buffer.position() +
            ", limit=" + buffer.limit());
    }
}
```

**Expected Output:**
```
After allocate: position=0, limit=10, capacity=10
After put: position=3, limit=10
After flip: position=0, limit=3
Read: A
Read: B
Read: C
After clear: position=0, limit=10
```

**Step-by-Step Explanation:**

1. `allocate(10)` creates a buffer with capacity 10, position 0, limit 10.
2. Each `put()` writes a byte and increments the position.
3. `flip()` sets the limit to the current position (3) and the position to 0.
4. The `while` loop reads all remaining bytes (`hasRemaining()` checks if position < limit).
5. `clear()` resets position to 0 and limit to capacity for reuse.

### Real-World Cases

- **File Reading:** Reading file contents into a `ByteBuffer` for processing.
- **Network Communication:** Sending and receiving data over sockets via buffers.
- **Text Processing:** Using `CharBuffer` for character data.
- **Binary Protocols:** Using `IntBuffer`, `LongBuffer`, etc. for structured binary data.

---

## 4. Selectors

### Definitions

**Core Definition:** A selector is a multiplexor that allows a single thread to monitor multiple channels for I/O events, enabling non-blocking I/O.

**Technical Definition:** "Selectors allow a single thread to monitor multiple channels for events". "Selectors multiplex many channels so a single thread can coordinate I/O across thousands of connections. This paradigm enables highly concurrent servers without tying each socket to its own thread". Channels register with a selector, and the selector blocks until one or more channels are ready for I/O operations.

**Beginner-Friendly Explanation:** A selector is like a receptionist at a busy office. Instead of having one person wait at each door (one thread per connection), the receptionist watches all the doors and notifies you when someone is at any of them. This lets one thread handle many connections efficiently.

### Purposes

- To enable a single thread to manage multiple channels
- To implement non-blocking, event-driven I/O
- To scale to thousands of concurrent connections with fewer threads
- To reduce context switching and memory overhead

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Create a selector
Selector selector = Selector.open();

// Register a channel with the selector
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

// Wait for events
int readyCount = selector.select();

// Process ready keys
Set<SelectionKey> keys = selector.selectedKeys();
for (SelectionKey key : keys) {
    if (key.isReadable()) { /* handle read */ }
    if (key.isWritable()) { /* handle write */ }
    if (key.isAcceptable()) { /* handle accept */ }
    if (key.isConnectable()) { /* handle connect */ }
}
keys.clear();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Selector.open()` | Creates a new selector. |
| `channel.register(selector, ops)` | Registers a channel with the selector for specific operations. |
| `selector.select()` | Blocks until at least one channel is ready. |
| `SelectionKey` | Represents a channel's registration with a selector. |
| `interestOps()` | The set of operations the channel is interested in. |
| `readyOps()` | The set of operations that are ready. |

**Operation Flags:**

| Flag | Description |
|------|-------------|
| `SelectionKey.OP_READ` | Read operation ready. |
| `SelectionKey.OP_WRITE` | Write operation ready. |
| `SelectionKey.OP_ACCEPT` | Accept operation ready (server sockets). |
| `SelectionKey.OP_CONNECT` | Connect operation ready (client sockets). |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Non-Blocking Echo Server**

```java
// File: NioEchoServer.java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class NioEchoServer {
    public static void main(String[] args) throws IOException {
        // Create and configure the server socket channel
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);
        serverChannel.bind(new InetSocketAddress(8080));
        System.out.println("Server listening on port 8080");

        // Create a selector and register the server channel
        Selector selector = Selector.open();
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);

        ByteBuffer buffer = ByteBuffer.allocate(256);

        while (true) {
            // Block until at least one channel is ready
            int readyCount = selector.select();
            if (readyCount == 0) continue;

            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iter = selectedKeys.iterator();

            while (iter.hasNext()) {
                SelectionKey key = iter.next();

                if (key.isAcceptable()) {
                    // Accept a new client connection
                    ServerSocketChannel server = (ServerSocketChannel) key.channel();
                    SocketChannel client = server.accept();
                    client.configureBlocking(false);
                    client.register(selector, SelectionKey.OP_READ);
                    System.out.println("Accepted connection from: " + client.getRemoteAddress());
                }

                if (key.isReadable()) {
                    // Read data from the client and echo it back
                    SocketChannel client = (SocketChannel) key.channel();
                    buffer.clear();
                    int bytesRead = client.read(buffer);

                    if (bytesRead == -1) {
                        client.close();
                        continue;
                    }

                    buffer.flip();
                    client.write(buffer);
                    buffer.flip();
                    byte[] data = new byte[buffer.remaining()];
                    buffer.get(data);
                    System.out.println("Echoed: " + new String(data));
                }

                iter.remove();
            }
        }
    }
}
```

**Expected Output (when a client sends "Hello"):**
```
Server listening on port 8080
Accepted connection from: /127.0.0.1:52345
Echoed: Hello
```

**Step-by-Step Explanation:**

1. `ServerSocketChannel.open()` creates a server channel.
2. `configureBlocking(false)` sets non-blocking mode.
3. `bind()` binds the server to port 8080.
4. `Selector.open()` creates a selector.
5. `serverChannel.register(selector, SelectionKey.OP_ACCEPT)` registers the server channel for accept events.
6. The infinite loop calls `selector.select()`, which blocks until an event occurs.
7. When a client connects, `isAcceptable()` is true, and the server accepts the connection, registering the client channel for read events.
8. When data is received, `isReadable()` is true, and the server reads the data and echoes it back.
9. `iter.remove()` removes the processed key from the selected set.

### Real-World Cases

- **High-Performance Web Servers:** Netty, Apache MINA, and other frameworks use selectors.
- **Chat Servers:** A single thread can manage thousands of client connections.
- **Proxy Servers:** Event-driven architecture for forwarding data between clients and backends.
- **Game Servers:** Low-latency multiplayer communication.

---

## 5. `ByteBuffer`, `CharBuffer`, `IntBuffer`, `LongBuffer`, `FloatBuffer`, `DoubleBuffer`, `ShortBuffer`

### Definitions

**Core Definition:** These are buffer classes for different primitive data types, providing typed access to buffer data.

**Technical Definition:** NIO provides specialised buffer classes for all primitive types except `boolean`: `ByteBuffer` (most commonly used), `CharBuffer`, `ShortBuffer`, `IntBuffer`, `LongBuffer`, `FloatBuffer`, and `DoubleBuffer`. A `ByteBuffer` can also create typed views of its contents using methods like `asCharBuffer()`, `asIntBuffer()`, etc..

**Beginner-Friendly Explanation:** Each buffer type is designed to hold a specific kind of data. `ByteBuffer` holds bytes (for binary data), `CharBuffer` holds characters (for text), `IntBuffer` holds integers, and so on. This makes it easy to work with data in its natural type without manual conversions.

### Purposes

- To provide type-specific data containers for different primitive types
- To enable efficient bulk reading and writing of typed data
- To support type-safe operations on buffer contents
- To allow views of `ByteBuffer` as other types

### Syntax Rules and Structure

```java
// Creating typed buffers
ByteBuffer byteBuf = ByteBuffer.allocate(1024);
CharBuffer charBuf = CharBuffer.allocate(100);
IntBuffer intBuf = IntBuffer.allocate(50);
LongBuffer longBuf = LongBuffer.allocate(25);
FloatBuffer floatBuf = FloatBuffer.allocate(50);
DoubleBuffer doubleBuf = DoubleBuffer.allocate(25);
ShortBuffer shortBuf = ShortBuffer.allocate(100);

// Typed views of ByteBuffer
ByteBuffer bb = ByteBuffer.allocate(1024);
CharBuffer cb = bb.asCharBuffer();
IntBuffer ib = bb.asIntBuffer();
LongBuffer lb = bb.asLongBuffer();
FloatBuffer fb = bb.asFloatBuffer();
DoubleBuffer db = bb.asDoubleBuffer();
ShortBuffer sb = bb.asShortBuffer();
```

**Syntax Rules:**

- Typed buffers are created with `allocate()` or via typed views of `ByteBuffer`.
- All typed buffers support the same core operations: `put()`, `get()`, `flip()`, `clear()`, etc.
- Typed views share the underlying data with the original `ByteBuffer`.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Using Typed Buffers**

```java
// File: TypedBufferDemo.java
import java.nio.*;

public class TypedBufferDemo {
    public static void main(String[] args) {
        // CharBuffer for text
        CharBuffer charBuf = CharBuffer.allocate(20);
        charBuf.put("Hello, NIO!");
        charBuf.flip();
        System.out.println("CharBuffer: " + charBuf);

        // IntBuffer for integers
        IntBuffer intBuf = IntBuffer.allocate(5);
        for (int i = 0; i < 5; i++) {
            intBuf.put(i * 10);
        }
        intBuf.flip();
        System.out.print("IntBuffer: ");
        while (intBuf.hasRemaining()) {
            System.out.print(intBuf.get() + " ");
        }
        System.out.println();

        // DoubleBuffer for doubles
        DoubleBuffer doubleBuf = DoubleBuffer.allocate(3);
        doubleBuf.put(3.14);
        doubleBuf.put(2.71);
        doubleBuf.put(1.41);
        doubleBuf.flip();
        System.out.print("DoubleBuffer: ");
        while (doubleBuf.hasRemaining()) {
            System.out.print(doubleBuf.get() + " ");
        }
        System.out.println();

        // Typed view of ByteBuffer
        ByteBuffer bb = ByteBuffer.allocate(16);
        IntBuffer view = bb.asIntBuffer();
        view.put(100);
        view.put(200);
        view.put(300);
        view.flip();
        System.out.print("IntBuffer view: ");
        while (view.hasRemaining()) {
            System.out.print(view.get() + " ");
        }
        System.out.println();
    }
}
```

**Expected Output:**
```
CharBuffer: Hello, NIO!
IntBuffer: 0 10 20 30 40
DoubleBuffer: 3.14 2.71 1.41
IntBuffer view: 100 200 300
```

**Step-by-Step Explanation:**

1. `CharBuffer.allocate(20)` creates a character buffer.
2. `charBuf.put("Hello, NIO!")` writes a string into the buffer.
3. `flip()` prepares for reading.
4. `IntBuffer.allocate(5)` creates an integer buffer; values are written and read back.
5. `DoubleBuffer` works similarly for `double` values.
6. `bb.asIntBuffer()` creates an `IntBuffer` view of a `ByteBuffer`, sharing the underlying memory.

### Real-World Cases

- **Text Processing:** `CharBuffer` for reading and writing text.
- **Binary Protocols:** `IntBuffer`, `LongBuffer` for structured binary data.
- **Scientific Computing:** `DoubleBuffer`, `FloatBuffer` for numerical data.
- **Image Processing:** `ByteBuffer` with typed views for pixel manipulation.

---

## 6. `MappedByteBuffer`

### Definitions

**Core Definition:** `MappedByteBuffer` is a direct byte buffer whose content is a memory-mapped region of a file.

**Technical Definition:** "`MappedByteBuffer` extends `ByteBuffer` to represent a buffer whose content is a memory-mapped region of a file". Memory mapping allows a file to be mapped directly into memory, enabling fast random access to file contents without explicit read/write calls. The buffer is created via `FileChannel.map(MapMode, position, size)`.

**Beginner-Friendly Explanation:** `MappedByteBuffer` is like projecting a file directly onto your computer's memory. Instead of reading the file piece by piece, you can access any part of it as if it were an array in memory. This is extremely fast for large files.

### Purposes

- To enable high-performance file access through memory mapping
- To provide fast random access to file contents
- To avoid the overhead of explicit read/write system calls
- To share file data between processes

### Syntax Rules and Structure

```java
// Map a file into memory
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
    long size = channel.size();
    MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_ONLY, 0, size);

    // Access bytes directly
    byte b = buffer.get(0);
    buffer.put(0, (byte) 65);  // If writable
}
```

**Map Modes:**

| Mode | Description |
|------|-------------|
| `READ_ONLY` | Buffer is read-only. |
| `READ_WRITE` | Buffer is readable and writable; changes are written back to the file. |
| `PRIVATE` | Buffer is copy-on-write; changes are not written back. |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Memory-Mapped File Processing**

```java
// File: MappedBufferDemo.java
import java.io.IOException;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

public class MappedBufferDemo {
    public static void main(String[] args) {
        Path path = Path.of("mapped_example.txt");

        // Write a file first
        try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.CREATE,
                StandardOpenOption.WRITE)) {
            channel.write(java.nio.ByteBuffer.wrap("Memory-mapped file content!".getBytes()));
        } catch (IOException e) {
            System.out.println("Write error: " + e.getMessage());
            return;
        }

        // Memory-map the file and process it
        try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
            long size = channel.size();
            MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_ONLY, 0, size);

            System.out.println("File size: " + size + " bytes");
            System.out.println("Mapped buffer capacity: " + buffer.capacity());

            // Read bytes directly from memory
            StringBuilder content = new StringBuilder();
            for (int i = 0; i < buffer.capacity(); i++) {
                content.append((char) buffer.get(i));
            }
            System.out.println("Content: " + content);

            // Random access: read byte at position 7
            System.out.println("Byte at position 7: " + (char) buffer.get(7));

        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
File size: 26 bytes
Mapped buffer capacity: 26
Content: Memory-mapped file content!
Byte at position 7: m
```

**Step-by-Step Explanation:**

1. A file is created and written with content.
2. `FileChannel.open()` opens the file for reading.
3. `channel.map(READ_ONLY, 0, size)` maps the entire file into memory as a `MappedByteBuffer`.
4. `buffer.get(i)` reads bytes directly from memory without explicit I/O calls.
5. `buffer.get(7)` demonstrates random access to any position in the file.

### Real-World Cases

- **Large File Processing:** Reading and searching large log files without loading them entirely.
- **Database Systems:** Memory-mapped files for fast data access.
- **Image Processing:** Mapping large image files into memory for pixel manipulation.
- **Inter-Process Communication:** Multiple processes can map the same file for shared memory.

---

## 7. `FileChannel`

### Definitions

**Core Definition:** `FileChannel` is a channel for reading, writing, mapping, and manipulating files.

**Technical Definition:** "A channel for reading, writing, mapping, and manipulating a file". "`FileChannel` is a selectable channel for file I/O. It supports reading and writing at specific positions, transferring data between channels, and mapping files into memory".

**Beginner-Friendly Explanation:** `FileChannel` is the main way to read and write files in NIO. It's more powerful than the classic file streams because it supports random access, memory mapping, and zero-copy transfers.

### Purposes

- To read and write files efficiently
- To support random access to file contents
- To enable memory-mapped file access
- To perform zero-copy data transfers between channels
- To lock file regions for exclusive access

### Syntax Rules and Structure

```java
// Open a FileChannel
FileChannel channel = FileChannel.open(path, StandardOpenOption.READ,
    StandardOpenOption.WRITE, StandardOpenOption.CREATE);

// Read into a buffer
ByteBuffer buffer = ByteBuffer.allocate(1024);
int bytesRead = channel.read(buffer);

// Write from a buffer
buffer.flip();
int bytesWritten = channel.write(buffer);

// Random access
channel.position(100);  // Move to byte 100

// Transfer between channels (zero-copy)
channel.transferTo(0, channel.size(), destChannel);
channel.transferFrom(srcChannel, 0, srcChannel.size());

// Close
channel.close();
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `read(ByteBuffer)` | Reads bytes into a buffer. |
| `write(ByteBuffer)` | Writes bytes from a buffer. |
| `position(long)` | Sets the file position for the next read/write. |
| `size()` | Returns the file size. |
| `transferTo(long, long, WritableByteChannel)` | Transfers bytes to another channel. |
| `transferFrom(ReadableByteChannel, long, long)` | Transfers bytes from another channel. |
| `map(MapMode, long, long)` | Maps a region of the file into memory. |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Random Access with FileChannel**

```java
// File: FileChannelRandomAccess.java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

public class FileChannelRandomAccess {
    public static void main(String[] args) {
        Path path = Path.of("random_access.dat");

        try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.CREATE,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE)) {

            // Write data at the beginning
            ByteBuffer buffer = ByteBuffer.wrap("0123456789".getBytes());
            channel.write(buffer);

            // Seek to position 5 and overwrite
            channel.position(5);
            ByteBuffer overwrite = ByteBuffer.wrap("ABCDE".getBytes());
            channel.write(overwrite);

            // Read the entire file back
            channel.position(0);
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            channel.read(readBuffer);
            readBuffer.flip();

            byte[] data = new byte[readBuffer.remaining()];
            readBuffer.get(data);
            System.out.println("Content: " + new String(data));

        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Content: 01234ABCDE
```

**Step-by-Step Explanation:**

1. The file is opened for reading, writing, and creating.
2. `channel.write(buffer)` writes `"0123456789"` to the file.
3. `channel.position(5)` moves the file position to byte 5.
4. `channel.write(overwrite)` writes `"ABCDE"` starting at position 5, overwriting bytes 5–9.
5. `channel.position(0)` resets to the beginning.
6. The file is read back, producing `"01234ABCDE"`.

### Real-World Cases

- **Database Files:** Random access to records by offset.
- **File Copying:** Using `transferTo()` for efficient copies.
- **Log File Processing:** Reading specific sections of large log files.
- **File Locking:** Preventing concurrent modification.

---

## 8. `SocketChannel`, `ServerSocketChannel`, `DatagramChannel`

### Definitions

**Core Definition:** These are network channel implementations for TCP (`SocketChannel`, `ServerSocketChannel`) and UDP (`DatagramChannel`) communication.

**Technical Definition:** `SocketChannel` is "a selectable channel for stream-oriented connecting sockets"; `ServerSocketChannel` is "a selectable channel for stream-oriented listening sockets"; `DatagramChannel` is "a selectable channel for datagram-oriented sockets".

**Beginner-Friendly Explanation:** These channels are for network communication. `ServerSocketChannel` listens for incoming connections, `SocketChannel` connects to a server and sends/receives data, and `DatagramChannel` sends and receives UDP packets.

### Purposes

- To enable non-blocking network communication
- To support multiplexed networking with selectors
- To implement TCP servers and clients
- To handle UDP datagram communication

### Syntax Rules and Structure

```java
// ServerSocketChannel
ServerSocketChannel server = ServerSocketChannel.open();
server.configureBlocking(false);
server.bind(new InetSocketAddress(8080));

// SocketChannel (client)
SocketChannel client = SocketChannel.open();
client.configureBlocking(false);
client.connect(new InetSocketAddress("example.com", 80));

// DatagramChannel
DatagramChannel udp = DatagramChannel.open();
udp.configureBlocking(false);
udp.bind(new InetSocketAddress(9999));
```

### Annotated Complete Step-by-Step Code Examples

**Example 1: Blocking TCP Client-Server**

```java
// File: TcpChannelDemo.java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;

public class TcpChannelDemo {
    public static void main(String[] args) throws IOException, InterruptedException {
        int port = 9090;

        // Start server in a separate thread
        Thread serverThread = new Thread(() -> {
            try (ServerSocketChannel server = ServerSocketChannel.open()) {
                server.bind(new InetSocketAddress(port));
                System.out.println("Server ready on port " + port);

                try (SocketChannel client = server.accept()) {
                    ByteBuffer buffer = ByteBuffer.allocate(256);
                    client.read(buffer);
                    buffer.flip();
                    byte[] data = new byte[buffer.remaining()];
                    buffer.get(data);
                    System.out.println("Server received: " + new String(data));

                    // Echo back
                    client.write(ByteBuffer.wrap("Echo: ".getBytes()));
                    client.write(ByteBuffer.wrap(data));
                }
            } catch (IOException e) {
                System.out.println("Server error: " + e.getMessage());
            }
        });
        serverThread.start();
        Thread.sleep(500);

        // Client
        try (SocketChannel client = SocketChannel.open()) {
            client.connect(new InetSocketAddress("localhost", port));
            client.write(ByteBuffer.wrap("Hello, Server!".getBytes()));

            ByteBuffer buffer = ByteBuffer.allocate(256);
            client.read(buffer);
            buffer.flip();
            byte[] data = new byte[buffer.remaining()];
            buffer.get(data);
            System.out.println("Client received: " + new String(data));
        }
        serverThread.join();
    }
}
```

**Expected Output:**
```
Server ready on port 9090
Server received: Hello, Server!
Client received: Echo: Hello, Server!
```

**Step-by-Step Explanation:**

1. A `ServerSocketChannel` is created and bound to port 9090.
2. The server accepts a connection from a client.
3. The server reads data from the client into a `ByteBuffer`.
4. The server echoes the data back to the client.
5. The client connects, sends data, and reads the response.

### Real-World Cases

- **Web Servers:** HTTP servers use `ServerSocketChannel` and `SocketChannel`.
- **Chat Applications:** Real-time messaging over TCP channels.
- **DNS Servers:** `DatagramChannel` for UDP-based DNS queries.
- **Game Servers:** Low-latency UDP communication for multiplayer games.

---

## 9. `Pipe`

### Definitions

**Core Definition:** A `Pipe` is a pair of channels that implements a unidirectional pipe for communication between threads within the same JVM.

**Technical Definition:** "A pipe consists of a pair of channels: A writable `SinkChannel` and a readable `SourceChannel`. Once some bytes are written to the sink channel they can be read from the source channel in exactly the order in which they were written". "Whether or not a thread writing bytes to a pipe will block until another thread reads those bytes is system-dependent and therefore unspecified".

**Beginner-Friendly Explanation:** A `Pipe` is like a one-way tunnel between two threads. One thread writes data into one end (`SinkChannel`), and another thread reads it from the other end (`SourceChannel`). It's useful when threads need to communicate without using shared memory.

### Purposes

- To enable communication between threads within the same JVM
- To provide a unidirectional data channel
- To avoid shared memory and its synchronization issues
- To implement producer-consumer patterns

### Syntax Rules and Structure

```java
// Create a pipe
Pipe pipe = Pipe.open();

// Get the channels
Pipe.SinkChannel sink = pipe.sink();      // Write end
Pipe.SourceChannel source = pipe.source(); // Read end

// Write to the pipe
ByteBuffer buffer = ByteBuffer.allocate(1024);
buffer.put("Message".getBytes());
buffer.flip();
sink.write(buffer);

// Read from the pipe
ByteBuffer readBuffer = ByteBuffer.allocate(1024);
source.read(readBuffer);
readBuffer.flip();
```

**Key Classes:**

| Class | Description |
|-------|-------------|
| `Pipe` | The pipe itself; created via `Pipe.open()`. |
| `Pipe.SinkChannel` | The writable end of the pipe. |
| `Pipe.SourceChannel` | The readable end of the pipe. |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Thread Communication via Pipe**

```java
// File: PipeDemo.java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.Pipe;

public class PipeDemo {
    public static void main(String[] args) throws IOException, InterruptedException {
        Pipe pipe = Pipe.open();

        // Producer thread writes to the pipe
        Thread producer = new Thread(() -> {
            try (Pipe.SinkChannel sink = pipe.sink()) {
                ByteBuffer buffer = ByteBuffer.allocate(256);
                buffer.put("Hello from producer!".getBytes());
                buffer.flip();
                sink.write(buffer);
                System.out.println("Producer: message sent");
            } catch (IOException e) {
                System.out.println("Producer error: " + e.getMessage());
            }
        });

        // Consumer thread reads from the pipe
        Thread consumer = new Thread(() -> {
            try (Pipe.SourceChannel source = pipe.source()) {
                ByteBuffer buffer = ByteBuffer.allocate(256);
                source.read(buffer);
                buffer.flip();
                byte[] data = new byte[buffer.remaining()];
                buffer.get(data);
                System.out.println("Consumer: received - " + new String(data));
            } catch (IOException e) {
                System.out.println("Consumer error: " + e.getMessage());
            }
        });

        producer.start();
        Thread.sleep(100); // Ensure producer writes first
        consumer.start();

        producer.join();
        consumer.join();
    }
}
```

**Expected Output:**
```
Producer: message sent
Consumer: received - Hello from producer!
```

**Step-by-Step Explanation:**

1. `Pipe.open()` creates a new pipe.
2. The producer thread gets the `SinkChannel` and writes a message into a `ByteBuffer`.
3. The consumer thread gets the `SourceChannel` and reads from the pipe into a `ByteBuffer`.
4. The consumer prints the received message.
5. Both channels are closed automatically with try-with-resources.

### Real-World Cases

- **Producer-Consumer Patterns:** Threads communicate without shared memory.
- **Pipeline Processing:** Data flows from one thread to another for multi-stage processing.
- **Testing:** Simulating I/O streams between threads.

---

## 10. `Selector` and `SelectionKey`

### Definitions

**Core Definition:** A `Selector` is a multiplexor for non-blocking channels, and a `SelectionKey` represents a channel's registration with a selector.

**Technical Definition:** A `Selector` allows a single thread to monitor multiple channels for I/O events. A `SelectionKey` encapsulates the registration of a channel with a selector, including the channel's interest set (operations it is interested in) and the ready set (operations that are ready). The `interestOps()` method sets the interest set, and `readyOps()` returns the ready set.

**Beginner-Friendly Explanation:** A `Selector` is like a switchboard operator. It watches many phone lines (channels) and tells you when someone is calling (an event is ready). A `SelectionKey` is like a note attached to each phone line, saying what kind of call you're interested in (read, write, accept, connect).

### Purposes

- To enable a single thread to manage multiple channels
- To implement non-blocking, event-driven I/O
- To scale to thousands of concurrent connections
- To monitor multiple channels for readiness

### Syntax Rules and Structure

```java
// Create selector and register channel
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

// Wait for events
selector.select();

// Process ready keys
Set<SelectionKey> keys = selector.selectedKeys();
Iterator<SelectionKey> iter = keys.iterator();
while (iter.hasNext()) {
    SelectionKey k = iter.next();
    if (k.isReadable()) { /* handle read */ }
    if (k.isWritable()) { /* handle write */ }
    iter.remove();
}
```

**SelectionKey Operations:**

| Method | Description |
|--------|-------------|
| `isReadable()` | Returns `true` if the channel is ready for reading. |
| `isWritable()` | Returns `true` if the channel is ready for writing. |
| `isAcceptable()` | Returns `true` if the channel is ready to accept a new connection. |
| `isConnectable()` | Returns `true` if the channel is ready to complete a connection. |
| `interestOps()` | Returns the interest set. |
| `interestOps(int)` | Sets the interest set. |
| `readyOps()` | Returns the ready set. |
| `channel()` | Returns the channel associated with the key. |
| `attachment()` | Returns the attachment object. |
| `attach(Object)` | Attaches an object to the key. |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Selector-Based Chat Server**

```java
// File: SelectorChatServer.java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class SelectorChatServer {
    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();
        ServerSocketChannel server = ServerSocketChannel.open();
        server.bind(new InetSocketAddress(7070));
        server.configureBlocking(false);
        server.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("Chat server started on port 7070");
        ByteBuffer buffer = ByteBuffer.allocate(1024);

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
                    System.out.println("New client: " + client.getRemoteAddress());
                } else if (key.isReadable()) {
                    SocketChannel client = (SocketChannel) key.channel();
                    buffer.clear();
                    int bytesRead = client.read(buffer);

                    if (bytesRead == -1) {
                        client.close();
                        continue;
                    }

                    buffer.flip();
                    byte[] data = new byte[buffer.remaining()];
                    buffer.get(data);
                    String message = new String(data).trim();
                    System.out.println("Received: " + message);

                    // Broadcast to all other clients
                    for (SelectionKey k : selector.keys()) {
                        if (k != key && k.channel() instanceof SocketChannel) {
                            SocketChannel other = (SocketChannel) k.channel();
                            other.write(ByteBuffer.wrap((message + "\n").getBytes()));
                        }
                    }
                }
            }
        }
    }
}
```

**Expected Output (when clients connect and send messages):**
```
Chat server started on port 7070
New client: /127.0.0.1:52345
Received: Hello everyone!
New client: /127.0.0.1:52346
Received: Hi there!
```

**Step-by-Step Explanation:**

1. A `Selector` and `ServerSocketChannel` are created; the server binds to port 7070.
2. The server channel is registered for `OP_ACCEPT`.
3. `selector.select()` blocks until an event occurs.
4. When a client connects, `isAcceptable()` is true; the client channel is registered for `OP_READ`.
5. When data arrives, `isReadable()` is true; the message is read and broadcast to all other connected clients.
6. The `iter.remove()` removes the processed key from the selected set.

### Real-World Cases

- **Chat Servers:** Broadcasting messages to multiple clients.
- **HTTP Servers:** Handling multiple requests concurrently with a single thread.
- **Proxy Servers:** Forwarding data between clients and backends.
- **Game Servers:** Real-time multiplayer communication.

---

## 11. NIO Best Practices

### Definitions

**Core Definition:** NIO best practices are guidelines for using Java NIO effectively, avoiding common pitfalls, and achieving optimal performance.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java and Oracle's API documentation. Key principles include using direct buffers judiciously, sizing buffers appropriately, using try-with-resources for channel management, and choosing between blocking and non-blocking modes based on the use case.

**Beginner-Friendly Explanation:** Best practices help you use NIO correctly without falling into common traps. They're like rules of the road for high-performance I/O programming.

### Purposes

- To achieve optimal I/O performance
- To avoid memory leaks and resource exhaustion
- To write maintainable and correct NIO code
- To choose the right NIO features for the problem

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use Direct Buffers Judiciously | "Direct buffers should be allocated primarily for large, long-lived buffers that are subject to the underlying system's native I/O operations". | Oracle |
| Avoid Direct Buffers for Short-Lived Objects | "Direct buffers are beyond the scope of Java's garbage collector, which can cause memory leaks if they are used injudiciously". | SEI CERT |
| Size Buffers Appropriately | "Start with 4KB-64KB buffers and adjust based on profiling results". | Repost |
| Use Try-with-Resources | Channels implement `AutoCloseable`; always close them with try-with-resources. | Oracle |
| Prefer Non-Blocking for High Concurrency | "Non-blocking sockets, multiplexed with a selector, allow you to serve many clients using fewer threads". | Cleverence |
| Use Zero-Copy for File Transfers | "Zero-copy techniques can reduce the number of copies between kernel space and user space". | Yisu |
| Choose Blocking for Simple Tasks | "Blocking calls can be easier to test and reason about when concurrency needs are minimal". | Cleverence |
| Handle Buffer State Correctly | Always use `flip()` before reading, `clear()` before reusing. | Oracle |
| Use `MappedByteBuffer` for Large Files | Memory mapping provides fast random access to large files. | Oracle |
| Validate Buffer Operations | Check return values from `read()` and `write()`; handle `-1` for EOF. | Best practice |
| Use `Selector` for Multiplexing | "A single thread can coordinate I/O across thousands of connections". | Cleverence |
| Avoid Memory Leaks with Direct Buffers | "Frequent allocation of large direct buffers can cause an `OutOfMemoryError`". | SEI CERT |

### Annotated Complete Step-by-Step Code Examples

```java
// File: NioBestPracticesDemo.java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

public class NioBestPracticesDemo {
    public static void main(String[] args) {
        Path path = Path.of("best_practices.dat");

        // BEST PRACTICE: Use try-with-resources for channels
        try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.CREATE,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE)) {

            // BEST PRACTICE: Use heap buffer for short-lived operations
            ByteBuffer writeBuffer = ByteBuffer.allocate(4096);
            writeBuffer.put("Best practices for NIO".getBytes());
            writeBuffer.flip();
            channel.write(writeBuffer);

            // BEST PRACTICE: Use direct buffer for repeated I/O
            ByteBuffer directBuffer = ByteBuffer.allocateDirect(4096);
            channel.position(0);
            int bytesRead = channel.read(directBuffer);
            System.out.println("Read " + bytesRead + " bytes using direct buffer");

            // BEST PRACTICE: Use zero-copy for file transfers
            Path destPath = Path.of("best_practices_copy.dat");
            try (FileChannel destChannel = FileChannel.open(destPath,
                    StandardOpenOption.CREATE,
                    StandardOpenOption.WRITE)) {
                long transferred = channel.transferTo(0, channel.size(), destChannel);
                System.out.println("Zero-copy transferred " + transferred + " bytes");
            }

            // Clean up
            java.nio.file.Files.deleteIfExists(destPath);
            java.nio.file.Files.deleteIfExists(path);

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Read 21 bytes using direct buffer
Zero-copy transferred 21 bytes
```

**Step-by-Step Explanation:**

1. `FileChannel.open()` is used with try-with-resources for automatic resource management.
2. A heap `ByteBuffer` is used for the short-lived write operation.
3. A direct `ByteBuffer` is used for the read operation, which may benefit from native I/O.
4. `transferTo()` performs a zero-copy transfer from the source channel to the destination channel.
5. Both files are cleaned up with `Files.deleteIfExists()`.

### Real-World Cases

- **High-Throughput Servers:** Using selectors and non-blocking channels for scalability.
- **Large File Processing:** Memory-mapped buffers and zero-copy transfers.
- **File Servers:** Direct buffers for efficient file serving.
- **Proxy Servers:** Non-blocking multiplexed I/O for forwarding data.

---

## References Links

- java.nio — OpenJDK API Documentation - https://mintlify.wiki/openjdk/jdk/api/java-nio
- Java NIO Deep Dive: Channels, Buffers, Selectors, Async I/O and Zero-Copy — Cleverence - https://www.cleverence.com/articles/oracle-documentation/9-java-nio-4827/
- Package java.nio.channels — Java™ Platform Standard Ed. 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/nio/channels/compact3-package-summary.html
- Java NIO Package — Compile-N-Run (GitHub) - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/language/java/6-java-file-handling/7-java-nio-package.mdx
- Non-Blocking Time Server NIO Example — Oracle Core Libraries - https://docs.oracle.com/en/java/javase/27/core/non-blocking-time-server-nio-example.html
- Grep NIO Example — Oracle Core Libraries - https://docs.oracle.com/en/java/javase/27/core/grep-nio-example.html
- OBJ53-J. Do not use direct buffers for short-lived, infrequently used objects — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/pages/diffpagesbyversion.action?pageId=88487617
- SelectionKey (Java SE 21 API Specification) — Oracle - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/channels/SelectionKey.html
- Class Pipe — Java™ Platform Standard Ed. 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Pipe.html
- Java NIO Buffer Guide — Cleverence - https://www.cleverence.com/articles/oracle-documentation/9-java-nio-4827/
- Java Async NIO for File Processing — Repost (AWS) - https://www.repost.aws
- Java NIO如何优化I/O性能 — Yisu - https://m.yisu.com