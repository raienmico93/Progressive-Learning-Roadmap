# Node.js Core Stream Architectures — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A stream is an abstract interface for working with streaming data in Node.js, allowing data to be read from or written to a source or destination incrementally rather than all at once.

**Technical Definition:** A stream is an abstract interface for working with streaming data in Node.js. The `stream` module provides a base API that makes it easy to build objects that implement the stream interface. There are four fundamental stream types: `Readable`, `Writable`, `Duplex`, and `Transform`. All streams are instances of `EventEmitter`. Streams can be readable, writable, or both. Streams operate exclusively on strings and `Buffer` (or `Uint8Array`) objects, though "object mode" allows other JavaScript values (except `null`, which serves a special purpose within streams).

**Beginner-Friendly Explanation:** Imagine a water pipe system. A **Readable** stream is a faucet — data flows out of it. A **Writable** stream is a drain — data flows into it. A **Duplex** stream is a two-way pipe — data flows both ways (like a telephone conversation). A **Transform** stream is a water filter installed in the pipe — data goes in one way and comes out changed. Streams let your program handle data piece by piece instead of waiting for the whole thing, which is essential for large files, network connections, and real-time data.

### Key Characteristics

- **Four fundamental types:** `Readable`, `Writable`, `Duplex`, and `Transform`, each serving a distinct data flow role.
- **Event-driven:** All streams extend `EventEmitter` and emit lifecycle events such as `'data'`, `'end'`, `'error'`, `'finish'`, `'close'`, and `'drain'`.
- **Buffer-based:** Both readable and writable streams maintain internal buffers controlled by the `highWaterMark` option.
- **Backpressure-aware:** Streams signal when their buffers are full, allowing producers to slow down and prevent memory exhaustion.
- **Modern consumption:** Readable streams support `async` iteration (`for await...of`), and Node.js streams interoperate with the WHATWG Web Streams API via `toWeb()` and `fromWeb()`.
- **Composable:** Streams can be piped together using `.pipe()`, `stream.pipeline()`, or async iteration.

### Prerequisites

- **Node.js runtime:** The `stream` module is built into Node.js. It has been stable since v0.10.0.
- **Basic JavaScript knowledge:** Understanding of functions, callbacks, and the `EventEmitter` pattern.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Asynchronous programming concepts:** The event loop, callbacks, Promises, and `async/await`.
- **Buffer basics:** Understanding of `Buffer` and binary data.

### Related Programming Areas

- **File System (`fs`):** `fs.createReadStream()` and `fs.createWriteStream()` are stream implementations.
- **Network (`net`, `http`):** TCP sockets and HTTP request/response objects are streams.
- **Compression (`zlib`):** `zlib.createGzip()` and `zlib.createDeflate()` are Transform streams.
- **Cryptography (`crypto`):** Cipher and hash objects are Transform streams.
- **Web Streams API:** WHATWG-standard streams interoperable with Node.js streams.
- **Worker Threads:** Streams can be transferred between threads via `MessagePort`.

### Core Concepts

1. **The 4 Fundamental Stream Types** — `Readable`, `Writable`, `Duplex`, `Transform`.
2. **Stream Events & State Management** — lifecycle events and error propagation.
3. **Modern Stream Consuming** — `for await...of` and Web Streams API.
4. **Backpressure Mechanics** — `highWaterMark`, internal buffers, and the `'drain'` event.

---

## Core Concept 1: The 4 Fundamental Stream Types

### Sub-Feature 1.1: Readable Streams

#### Definitions

**Core Definition:** A Readable stream is a stream from which data can be read, representing a source of streaming data.

**Technical Definition:** `Readable` streams are streams from which data can be read (for example, `fs.createReadStream()`). Readable streams operate in one of two modes: **paused mode** or **flowing mode**. In flowing mode, data is read from the underlying system automatically and provided to an application as quickly as possible using events via the `EventEmitter` interface. In paused mode, the `stream.read()` method must be called explicitly to read chunks of data from the stream. All Readable streams begin in paused mode but can be switched to flowing mode by adding a `'data'` event handler, calling `stream.resume()`, or calling `stream.pipe()`.

**Beginner-Friendly Explanation:** A Readable stream is like a book you can read page by page. You can either read it at your own pace (paused mode — you call `read()` whenever you want the next chunk), or you can set it to auto-read where every page is handed to you as soon as it's available (flowing mode — you listen for `'data'` events).

#### Purposes

- To provide a source of data that can be consumed incrementally.
- To enable processing of data that is too large to fit in memory at once.
- To support both pull-based (`read()`) and push-based (`'data'` event) consumption models.
- To serve as the data source in pipeline compositions.

#### Syntax Rules and Structure

**Creating a Readable stream:**
```js
const { Readable } = require('node:stream');

const readable = new Readable({
  read(size) {
    // Push data using this.push(chunk)
    // Call this.push(null) to signal end of stream
  }
});
```

**Two reading modes:**

| Mode | How to Enter | How to Read | Use Case |
|------|-------------|-------------|----------|
| Paused | Default | `stream.read()` | Manual control, backpressure-aware |
| Flowing | Add `'data'` listener, call `.resume()` or `.pipe()` | `'data'` event | Real-time processing, piping |

**Key methods:**
| Method | Description |
|--------|-------------|
| `readable.read([size])` | Reads and returns data from the internal buffer. |
| `readable.pause()` | Stops emitting `'data'` events, switching out of flowing mode. |
| `readable.resume()` | Resumes emitting `'data'` events. |
| `readable.pipe(destination)` | Pipes data to a Writable stream. |
| `readable.unpipe([destination])` | Detaches a Writable stream. |

**Constraints and Limitations:**
- Data is buffered internally when the consumer does not call `read()`. Once the buffer reaches `highWaterMark`, the stream stops calling the underlying `_read()` method.
- Adding a `'data'` listener switches the stream to flowing mode, which may cause data loss if the stream is not ready.
- `readable.read()` returns `null` when there is no data available at the moment.

#### Annotated Code Example

```js
// readable-example.js
const { Readable } = require('node:stream');

// Create a Readable stream that emits numbers 0-4
const readable = new Readable({
  objectMode: true, // Allow non-Buffer values
  read() {
    // Push data one item at a time
    if (this._count === undefined) this._count = 0;
    if (this._count < 5) {
      this.push(this._count++);
    } else {
      this.push(null); // Signal end of stream
    }
  }
});

// --- Paused mode: using read() ---
console.log('Paused mode:');
readable.on('readable', () => {
  let chunk;
  while ((chunk = readable.read()) !== null) {
    console.log('  Read:', chunk);
  }
});

// --- Flowing mode: using 'data' event ---
// (Uncomment to see flowing mode instead)
// readable.on('data', (chunk) => console.log('  Data:', chunk));
```

**Expected Output:**
```
Paused mode:
  Read: 0
  Read: 1
  Read: 2
  Read: 3
  Read: 4
```

**Why this output:** In paused mode, the `'readable'` event fires when data is available. The `while` loop calls `read()` repeatedly until it returns `null`, consuming all available chunks. The stream pushes values 0 through 4, then `null` to signal end.

### Sub-Feature 1.2: Writable Streams

#### Definitions

**Core Definition:** A Writable stream is a stream to which data can be written, representing a destination for streaming data.

**Technical Definition:** `Writable` streams are streams to which data can be written (for example, `fs.createWriteStream()`). Data is buffered in Writable streams when the `writable.write(chunk)` method is called repeatedly. While the total size of the internal write buffer is below the threshold set by `highWaterMark`, calls to `writable.write()` will return `true`. Once the size of the internal buffer reaches or exceeds the `highWaterMark`, `false` will be returned. The return value is strictly advisory: you MAY continue to write even if it returns `false`, but writes will be buffered in memory, and it is recommended to wait for the `'drain'` event before writing more data.

**Beginner-Friendly Explanation:** A Writable stream is like a drain or a mailbox. You put things into it using `write()`. If the drain is getting full, `write()` returns `false` — a polite way of saying "slow down, I'm getting backed up." When it's ready for more, it emits a `'drain'` event to tell you to continue.

#### Purposes

- To provide a destination for data that can be written incrementally.
- To enable writing data that is produced over time.
- To support backpressure so that fast producers do not overwhelm slow consumers.
- To serve as the destination in pipeline compositions.

#### Syntax Rules and Structure

```js
const { Writable } = require('node:stream');

const writable = new Writable({
  write(chunk, encoding, callback) {
    // Process the chunk
    callback(); // Signal completion (or callback(error))
  }
});
```

**Key methods:**
| Method | Description |
|--------|-------------|
| `writable.write(chunk[, encoding][, callback])` | Writes data to the stream. Returns `false` if the internal buffer is full. |
| `writable.end([chunk][, encoding][, callback])` | Signals that no more data will be written. |
| `writable.cork()` | Forces all writes to be buffered in memory. |
| `writable.uncork()` | Flushes all data buffered by `cork()`. |

**Constraints and Limitations:**
- Once `writable.write()` returns `false`, do not write more chunks until the `'drain'` event is emitted.
- `writable.end()` must be called to signal completion; otherwise the stream may hang.
- The `callback` passed to `write()` is called when the chunk is flushed, not when it is written.

#### Annotated Code Example

```js
// writable-example.js
const { Writable } = require('node:stream');

// Create a Writable stream that logs each chunk
const writable = new Writable({
  write(chunk, encoding, callback) {
    console.log('Writing:', chunk.toString());
    // Simulate async processing
    setTimeout(callback, 10);
  }
});

// Write data
writable.write('Hello, ');
writable.write('World!');

// Signal end
writable.end('Done.', () => {
  console.log('All writes completed.');
});
```

**Expected Output:**
```
Writing: Hello, 
Writing: World!
Writing: Done.
All writes completed.
```

**Why this output:** Each `write()` call passes a chunk to the `_write` implementation. The `callback` is invoked after the `setTimeout` completes, allowing the stream to process the next chunk. The `end()` method writes the final chunk and invokes its callback when all data has been flushed.

### Sub-Feature 1.3: Duplex Streams

#### Definitions

**Core Definition:** A Duplex stream is a stream that is both Readable and Writable, with independent read and write channels.

**Technical Definition:** `Duplex` streams are streams that are both `Readable` and `Writable` (for example, `net.Socket`). Because Duplex and Transform streams are both Readable and Writable, each maintains two separate internal buffers used for reading and writing, allowing each side to operate independently of the other. This means that a Duplex stream can have data flowing in both directions simultaneously.

**Beginner-Friendly Explanation:** A Duplex stream is like a telephone line. You can talk (write) and listen (read) at the same time, and the two directions are independent — what you say doesn't interfere with what you hear.

#### Purposes

- To model full-duplex communication channels like TCP sockets.
- To allow simultaneous reading and writing on the same stream object.
- To serve as the base for Transform streams.

#### Syntax Rules and Structure

```js
const { Duplex } = require('node:stream');

const duplex = new Duplex({
  read(size) { /* implement readable side */ },
  write(chunk, encoding, callback) { /* implement writable side */ }
});
```

**Key characteristic:** Independent read and write buffers. The `readableHighWaterMark` and `writableHighWaterMark` options can be set independently.

**Constraints and Limitations:**
- Duplex streams do not automatically connect their read and write sides (unlike Transform streams).
- Data written to a Duplex stream is not automatically readable from the same stream unless the implementation explicitly does so.
- TCP sockets are the canonical Duplex stream example.

#### Annotated Code Example

```js
// duplex-example.js
const { Duplex } = require('node:stream');

// A Duplex stream that echoes written data back on the readable side
const echo = new Duplex({
  read(size) {
    // No-op: data is pushed in _write
  },
  write(chunk, encoding, callback) {
    // Echo the chunk back to the readable side
    this.push(`Echo: ${chunk.toString()}`);
    callback();
  },
  final(callback) {
    this.push(null); // End readable side
    callback();
  }
});

// Write to the Duplex stream
echo.write('Hello');
echo.write('World');
echo.end();

// Read from the Duplex stream
echo.on('data', (chunk) => {
  console.log('Read:', chunk.toString());
});
```

**Expected Output:**
```
Read: Echo: Hello
Read: Echo: World
```

**Why this output:** The `_write` implementation pushes the echoed data onto the readable side of the same stream. The `_final` method pushes `null` to signal the end of the readable side. The `'data'` event then reads the echoed chunks.

### Sub-Feature 1.4: Transform Streams

#### Definitions

**Core Definition:** A Transform stream is a Duplex stream where the readable side is computed from the writable side, modifying or transforming data as it passes through.

**Technical Definition:** `Transform` streams are `Duplex` streams that can modify or transform the data as it is written and read (for example, `zlib.createDeflate()`). The `_transform()` method is called for each chunk written, and the transformed output is pushed to the readable side. Unlike Duplex streams, Transform streams connect their read and write sides, making them ideal for data processing pipelines.

**Beginner-Friendly Explanation:** A Transform stream is like a coffee filter. You pour water in (write), and coffee comes out (read) — the input is transformed into something different. Compression, encryption, and text decoding are all Transform operations.

#### Purposes

- To modify, transform, or compute data as it passes through a stream.
- To implement compression, encryption, encoding conversion, and other data processing.
- To serve as processing stages in a stream pipeline.
- To simplify the implementation of data transformations without managing separate readable and writable streams.

#### Syntax Rules and Structure

```js
const { Transform } = require('node:stream');

const transform = new Transform({
  transform(chunk, encoding, callback) {
    // Process chunk and push result
    this.push(processedChunk);
    callback();
  }
});
```

**Key method:**
| Method | Description |
|--------|-------------|
| `transform._transform(chunk, encoding, callback)` | Called for each chunk written. Must call `this.push()` with the transformed data and `callback()` when done. |
| `transform._flush(callback)` | Called at the end to flush any remaining data. |

**Constraints and Limitations:**
- `_transform()` must call `callback()` when processing is complete; failing to do so hangs the stream.
- The `_flush()` method is optional but useful for finalisation.
- Transform streams maintain separate read and write buffers, but the flow is coupled.

#### Annotated Code Example

```js
// transform-example.js
const { Transform } = require('node:stream');

// A Transform stream that converts text to uppercase
const upperCase = new Transform({
  transform(chunk, encoding, callback) {
    const upper = chunk.toString().toUpperCase();
    this.push(upper);
    callback();
  }
});

// Pipe data through the transform
process.stdin.pipe(upperCase).pipe(process.stdout);
```

**Expected Output (when typing "hello" and pressing Enter):**
```
HELLO
```

**Why this output:** Each chunk written to the Transform stream is converted to uppercase in `_transform()`. The transformed data is pushed to the readable side, which is piped to `process.stdout`. This demonstrates a complete pipeline: stdin → transform → stdout.

### Real-World Cases

- **File compression:** `fs.createReadStream('file.txt').pipe(zlib.createGzip()).pipe(fs.createWriteStream('file.txt.gz'))`.
- **HTTP file serving:** `fs.createReadStream('video.mp4').pipe(res)` — streaming a large video to an HTTP response.
- **Encryption:** `crypto.createCipheriv().pipe(transform).pipe(output)` — encrypting data on the fly.
- **TCP chat server:** `net.Socket` is a Duplex stream, allowing simultaneous send and receive.

---

## Core Concept 2: Stream Events & State Management

### Definitions

**Core Definition:** Streams emit lifecycle events that signal state changes, data availability, completion, errors, and resource cleanup.

**Technical Definition:** All streams are instances of `EventEmitter`. Key events include `'data'` (emitted when data is available on a Readable stream), `'end'` (emitted when a Readable stream has no more data), `'error'` (emitted when an error occurs), `'finish'` (emitted when a Writable stream has flushed all data), `'close'` (emitted when the stream and its underlying resources have been closed), and `'drain'` (emitted when a Writable stream's internal buffer has emptied).

**Beginner-Friendly Explanation:** Streams are like a live concert with different announcements. "Data" means new content is available. "End" means the show is over. "Error" means something went wrong. "Finish" means all your songs have been played. "Close" means the venue is shut down. "Drain" means the stage is clear and ready for more.

### Purposes

- To provide a standardised mechanism for reacting to stream lifecycle changes.
- To enable proper error handling and resource cleanup.
- To coordinate data flow between producers and consumers.
- To signal completion and readiness states.

### Syntax Rules and Structure

| Event | Stream Type | When Emitted |
|-------|-------------|--------------|
| `'data'` | Readable | Data is available (flowing mode). |
| `'end'` | Readable | No more data will be emitted. |
| `'error'` | Both | An error occurred. |
| `'finish'` | Writable | All data has been flushed to the underlying system. |
| `'close'` | Both | The stream and its resources have been closed. |
| `'drain'` | Writable | The internal buffer has emptied; safe to write again. |
| `'readable'` | Readable | Data is available or the end of the stream has been reached. |
| `'pipe'` | Writable | `.pipe()` was called on a Readable stream. |
| `'unpipe'` | Writable | `.unpipe()` was called. |

**Constraints and Limitations:**
- The `'error'` event is special: if a stream emits `'error'` without a listener, the error is thrown, potentially crashing the process.
- `'close'` is not always emitted if the stream is destroyed with an error.
- The `'finish'` event fires after `end()` is called and all data is flushed, but before `'close'`.

### Annotated Code Example

```js
// stream-events.js
const fs = require('node:fs');
const { pipeline } = require('node:stream');

const readable = fs.createReadStream('input.txt');
const writable = fs.createWriteStream('output.txt');

// Lifecycle events
readable.on('data', (chunk) => {
  console.log('Data chunk received:', chunk.length, 'bytes');
});

readable.on('end', () => {
  console.log('Readable ended.');
});

writable.on('finish', () => {
  console.log('Writable finished.');
});

writable.on('close', () => {
  console.log('Writable closed.');
});

// Error handling
readable.on('error', (err) => {
  console.error('Read error:', err.message);
});

writable.on('error', (err) => {
  console.error('Write error:', err.message);
});

// Pipe with proper cleanup
pipeline(readable, writable, (err) => {
  if (err) {
    console.error('Pipeline failed:', err);
  } else {
    console.log('Pipeline succeeded.');
  }
});
```

**Expected Output:**
```
Data chunk received: 12 bytes
Readable ended.
Writable finished.
Writable closed.
Pipeline succeeded.
```

**Why this output:** The `'data'` event fires for each chunk. The `'end'` event fires when the readable side is exhausted. The `'finish'` event fires after the writable flushes all data. The `'close'` event fires when resources are released. The `pipeline` callback confirms completion.

### Real-World Cases

- **Graceful shutdown:** Listening for `'close'` to release resources.
- **Progress tracking:** Counting `'data'` events to report progress.
- **Error recovery:** Attaching `'error'` listeners to prevent crashes.

---

## Core Concept 3: Modern Stream Consuming

### Sub-Feature 3.1: Async Iteration (`for await...of`)

#### Definitions

**Core Definition:** Readable streams implement the async iterator protocol, allowing them to be consumed with `for await...of` loops.

**Technical Definition:** Readable streams are async iterable and can be consumed using `for await...of` syntax. When a `for await...of` loop is exited via `return`, `break`, or `throw`, the stream is destroyed by default, unless the `destroyOnReturn` option is set to `false`. This provides a clean, readable alternative to `'data'` event handling and automatically handles backpressure.

**Beginner-Friendly Explanation:** Instead of setting up event listeners and managing callbacks, you can simply write `for await (const chunk of stream)` and process each chunk as it arrives. It's like reading a book page by page in a simple loop.

#### Purposes

- To provide a clean, synchronous-looking syntax for consuming stream data.
- To automatically handle backpressure.
- To simplify error handling with `try/catch`.
- To enable early termination with `break`.

#### Syntax Rules and Structure

```js
for await (const chunk of readableStream) {
  // Process chunk
}
```

| Option | Description |
|--------|-------------|
| `destroyOnReturn` | If `true` (default), stream is destroyed when the loop exits early. |

**Constraints and Limitations:**
- The stream is locked to a single consumer during iteration.
- Early termination destroys the stream by default.
- Not all streams support async iteration; check for `Symbol.asyncIterator`.

#### Annotated Code Example

```js
// async-iteration.js
const fs = require('node:fs');

(async () => {
  const stream = fs.createReadStream('large-file.txt', { encoding: 'utf8' });

  let lineCount = 0;
  for await (const chunk of stream) {
    lineCount += chunk.split('\n').length;
    if (lineCount > 100) {
      console.log('Stopping early at 100 lines.');
      break; // Stream is destroyed automatically
    }
  }

  console.log('Processed', lineCount, 'lines.');
})();
```

**Expected Output:**
```
Stopping early at 100 lines.
Processed 101 lines.
```

**Why this output:** The `for await...of` loop reads chunks sequentially. When `break` is called, the stream is destroyed automatically, preventing resource leaks. The `lineCount` tracks processed lines.

### Sub-Feature 3.2: Web Streams API Interoperability

#### Definitions

**Core Definition:** Node.js streams can be converted to and from the WHATWG Web Streams API (`ReadableStream`, `WritableStream`, `TransformStream`) using `toWeb()` and `fromWeb()` methods.

**Technical Definition:** The WHATWG Streams Standard (or "web streams") defines an API for handling streaming data that has become the standard across JavaScript environments. Node.js streams can be converted to web streams and vice versa via the `toWeb` and `fromWeb` methods: `stream.Readable.toWeb()` converts a Node.js Readable to a `ReadableStream`; `stream.Readable.fromWeb()` converts a `ReadableStream` to a Node.js Readable; and similar methods exist for Writable and Duplex streams. This interoperability is stable since Node.js v21.0.0.

**Beginner-Friendly Explanation:** Web streams are the browser's version of Node.js streams. If you're writing code that needs to work in both Node.js and the browser, or you're using a library that expects web streams, you can convert between the two formats.

#### Purposes

- To enable code sharing between Node.js and browser environments.
- To use libraries and frameworks built around the WHATWG Streams Standard.
- To leverage modern web APIs in Node.js applications.
- To provide a standard interface for streaming data across platforms.

#### Syntax Rules and Structure

| Conversion | Method |
|------------|--------|
| Node.js Readable → Web ReadableStream | `stream.Readable.toWeb(nodeReadable)` |
| Web ReadableStream → Node.js Readable | `stream.Readable.fromWeb(webStream)` |
| Node.js Writable → Web WritableStream | `stream.Writable.toWeb(nodeWritable)` |
| Web WritableStream → Node.js Writable | `stream.Writable.fromWeb(webStream)` |
| Node.js Duplex → Web streams | `stream.Duplex.toWeb(nodeDuplex)` |
| Web streams → Node.js Duplex | `stream.Duplex.fromWeb(webStream)` |

**Constraints and Limitations:**
- Web streams have different backpressure semantics than Node.js streams.
- Not all Node.js stream features map directly to web streams.
- Web streams are stable since Node.js v21.0.0.

#### Annotated Code Example

```js
// web-streams-interop.js
const { Readable } = require('node:stream');

// Create a Node.js Readable stream
const nodeReadable = Readable.from(['Hello', ' ', 'World']);

// Convert to a Web ReadableStream
const webStream = Readable.toWeb(nodeReadable);

// Consume the Web stream using async iteration
(async () => {
  for await (const chunk of webStream) {
    console.log('Web chunk:', chunk.toString());
  }
})();
```

**Expected Output:**
```
Web chunk: Hello
Web chunk:  
Web chunk: World
```

**Why this output:** `Readable.toWeb()` wraps the Node.js stream in a Web `ReadableStream`. The `for await...of` loop consumes the Web stream, which internally pulls data from the Node.js stream. The chunks are passed through unchanged.

### Real-World Cases

- **Framework integration:** Using web stream libraries (e.g., Fetch API `Response.body`) in Node.js.
- **Isomorphic code:** Sharing streaming logic between server and browser.
- **Modern APIs:** Interfacing with `fetch()` responses, which return web streams.

---

## Core Concept 4: Backpressure Mechanics

### Definitions

**Core Definition:** Backpressure is the mechanism by which a stream signals that its internal buffer is full, allowing producers to slow down and prevent memory exhaustion.

**Technical Definition:** Backpressure is a key goal of the `stream` API. The `highWaterMark` option specifies the maximum amount of data (in bytes for normal streams, or number of objects for object mode) that can be buffered before the stream stops reading from the underlying resource or returns `false` from `write()`. For `Readable` streams, when the internal buffer reaches `highWaterMark`, the stream temporarily stops calling `_read()`. For `Writable` streams, when the internal buffer reaches `highWaterMark`, `write()` returns `false`, and the caller should wait for the `'drain'` event before writing more.

**Beginner-Friendly Explanation:** Imagine a funnel. If you pour water in faster than it drains out, the funnel fills up. Backpressure is the funnel saying "stop pouring, I'm full!" — and you wait until it drains (the `'drain'` event) before pouring more. Without backpressure, the funnel would overflow (memory exhaustion).

### Purposes

- To prevent memory exhaustion from fast producers overwhelming slow consumers.
- To provide a standardised mechanism for flow control between stream stages.
- To enable efficient resource utilisation without unbounded buffering.
- To coordinate data flow in pipeline compositions.

### Syntax Rules and Structure

**`highWaterMark` option:**
```js
const readable = new Readable({ highWaterMark: 16384 }); // 16 KB
const writable = new Writable({ highWaterMark: 16384 });
```
| Stream Type | Default `highWaterMark` | Unit |
|-------------|------------------------|------|
| Normal (byte) | 16 KB (16,384) | Bytes |
| Object mode | 16 | Objects |
| `Readable` (Node.js v20+) | 64 KB (65,536) | Bytes |

**`write()` return value:**
```js
const canContinue = writable.write(chunk);
// true → buffer below highWaterMark
// false → buffer at or above highWaterMark; wait for 'drain'
```

**`drain` event:**
```js
writable.on('drain', () => {
  // Safe to write more data
});
```

**Constraints and Limitations:**
- `write()` returning `false` is advisory; you MAY continue writing, but data will be buffered in memory.
- The `'drain'` event is only emitted after `write()` has returned `false` and the buffer has emptied.
- `highWaterMark` does not limit the total memory usage of a stream; it only controls the threshold for backpressure signalling.
- Transform streams have separate read and write buffers, each with its own `highWaterMark`.

### Multiple Annotated Code Examples

#### Example 1: Respecting Backpressure in a Writable Stream

```js
// backpressure-writable.js
const { Writable } = require('node:stream');

// Slow consumer with a small buffer
const slowWritable = new Writable({
  highWaterMark: 3, // Small buffer for demonstration
  write(chunk, encoding, callback) {
    console.log('Processing:', chunk.toString());
    setTimeout(callback, 100); // Simulate slow processing
  }
});

// Fast producer
let i = 0;
function writeMore() {
  let canWrite = true;
  while (canWrite && i < 10) {
    canWrite = slowWritable.write(`chunk-${i}\n`);
    i++;
  }

  if (i < 10) {
    console.log('Buffer full, waiting for drain...');
    slowWritable.once('drain', () => {
      console.log('Drain received, resuming...');
      writeMore();
    });
  } else {
    slowWritable.end();
  }
}

writeMore();
```

**Expected Output:**
```
Processing: chunk-0
Processing: chunk-1
Processing: chunk-2
Processing: chunk-3
Buffer full, waiting for drain...
Processing: chunk-4
...
Drain received, resuming...
```

**Why this output:** The `write()` method returns `false` once the internal buffer reaches `highWaterMark` (3 chunks). The loop stops writing and waits for the `'drain'` event. When the slow consumer finishes processing, the buffer empties and `'drain'` is emitted, allowing the producer to resume.

#### Example 2: Using `pipeline()` for Automatic Backpressure

```js
// pipeline-backpressure.js
const fs = require('node:fs');
const zlib = require('node:zlib');
const { pipeline } = require('node:stream');

// pipeline() handles backpressure automatically
pipeline(
  fs.createReadStream('large-file.txt'),
  zlib.createGzip(),
  fs.createWriteStream('large-file.txt.gz'),
  (err) => {
    if (err) {
      console.error('Pipeline failed:', err);
    } else {
      console.log('Pipeline succeeded.');
    }
  }
);
```

**Expected Output:**
```
Pipeline succeeded.
```

**Why this output:** `stream.pipeline()` automatically manages backpressure between all stages. If the gzip transform is slower than the file read, the read stream is paused. If the write stream is slower than gzip, the gzip stream is paused. The `pipeline` callback is invoked when all data has been processed or an error occurs. `stream.pipeline()` will call `stream.destroy(err)` on all streams when an error is raised, ensuring proper cleanup.

### Real-World Cases

- **File compression:** Piping a large file through gzip to a compressed output file without loading it all into memory.
- **HTTP proxying:** Forwarding request data to a backend service while respecting backpressure.
- **Database exports:** Streaming query results to a file or network socket.
- **Video streaming:** Serving video files to clients while managing slow connections.

---

## References

- Node.js Documentation — Stream — https://nodejs.org/api/stream.html
- Node.js Documentation — Types of Streams — https://nodejs.org/api/stream.html#types-of-streams
- Node.js Documentation — Readable Streams — https://nodejs.org/api/stream.html#class-streamreadable
- Node.js Documentation — Writable Streams — https://nodejs.org/api/stream.html#class-streamwritable
- Node.js Documentation — Duplex and Transform Streams — https://nodejs.org/api/stream.html#class-streamduplex
- Node.js Documentation — `stream.pipeline()` — https://nodejs.org/api/stream.html#streampipelinestreams-callback
- Node.js Documentation — `stream.finished()` — https://nodejs.org/api/stream.html#streamfinishedstream-options-callback
- Node.js Documentation — `writable.write()` — https://nodejs.org/api/stream.html#writablewritechunk-encoding-callback
- Node.js Documentation — `readable.read()` — https://nodejs.org/api/stream.html#readablereadsize
- Node.js Documentation — Backpressuring in Streams — https://nodejs.org/en/learn/modules/backpressuring-in-streams
- Node.js Documentation — Web Streams API — https://nodejs.org/api/webstreams.html
- Node.js Documentation — `Readable.toWeb()` — https://nodejs.org/api/stream.html#streamreadabletowebstream
- Node.js Documentation — Async Iteration — https://nodejs.org/api/stream.html#readablesymbolasynciterator
- WHATWG Streams Standard — https://streams.spec.whatwg.org/
- MDN Web Docs — Streams API — https://developer.mozilla.org/en-US/docs/Web/API/Streams_API