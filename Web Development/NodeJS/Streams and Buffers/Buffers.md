# Node.js Buffers and Binary Data — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Buffer` is a fixed-length sequence of bytes used to represent binary data in Node.js, providing a way to read, write, and manipulate raw memory outside of JavaScript's standard string and number types.

**Technical Definition:** Prior to the introduction of `TypedArray`, the JavaScript language had no mechanism for reading or manipulating streams of binary data. The `Buffer` class was introduced as part of the Node.js API to enable interaction with octet streams in TCP streams, file system operations, and other contexts. Instances of the `Buffer` class are similar to arrays of integers from 0 to 255 but correspond to fixed-sized, raw memory allocations **outside the V8 heap**. The size of a `Buffer` is established when it is created and cannot be changed. With `TypedArray` now available, the `Buffer` class implements the `Uint8Array` API in a manner that is more optimized and suitable for Node.js.

**Beginner-Friendly Explanation:** Imagine you're sending a photo over the internet. The photo isn't text — it's a stream of bytes (numbers from 0 to 255). JavaScript strings can't handle that efficiently. A `Buffer` is like a box of numbered slots, where each slot holds one byte. You can fill the slots with data, read them, copy them, or combine them. Node.js uses Buffers whenever it deals with binary data like files, network packets, or images.

### Key Characteristics

- **Fixed-size, raw memory:** Buffers are allocated outside the V8 heap and cannot be resized after creation.
- **Uint8Array-based:** Since Node.js 4.0.0, Buffer instances are also Uint8Array instances, inheriting the full TypedArray API.
- **Global availability:** The `Buffer` class is within the global scope; `require('buffer').Buffer` is rarely needed.
- **Multiple creation paths:** `Buffer.alloc()`, `Buffer.allocUnsafe()`, and `Buffer.from()` each serve different performance and safety needs.
- **Encoding-aware:** Buffers convert between binary data and strings using encodings like `utf8`, `base64`, `hex`, and `latin1`.
- **Internal pooling:** Small buffers are sliced from a shared pre-allocated pool for performance.

### Prerequisites

- **Node.js runtime:** The `buffer` module is built into Node.js; no external installation is required. It has been stable since v0.1.90.
- **Basic JavaScript knowledge:** Understanding of arrays, strings, and typed arrays.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Basic understanding of binary concepts:** Bits, bytes, and character encodings.

### Related Programming Areas

- **File System (`fs`):** All file reads and writes produce or consume Buffers.
- **Streams:** Streams emit Buffers as `'data'` events.
- **Network (`net`, `http`):** TCP sockets and HTTP bodies are streamed as Buffers.
- **Cryptography (`crypto`):** Hash digests, cipher output, and key material are Buffers.
- **Compression (`zlib`):** Compressed data is processed as Buffers.
- **Worker Threads:** `SharedArrayBuffer` enables zero-copy data sharing between threads.

### Core Concepts

1. **Fundamentals of Binary Data** — bits, bytes, octets, and V8 vs. Buffer memory.
2. **Buffer Creation & Memory Management** — `alloc`, `allocUnsafe`, `from`, and pooling.
3. **Encoding & Decoding** — supported encodings and `StringDecoder`.
4. **Buffer Manipulation** — reading/writing numbers, copying, slicing, concatenating, and shared memory.
5. **Binary Protocols** — parsing network packets and binary file headers.

---

## Core Concept 1: Fundamentals of Binary Data

### Sub-Feature 1.1: Bits, Bytes, and Octets

#### Definitions

**Core Definition:** A **bit** is a single binary digit (0 or 1); a **byte** is a group of 8 bits; an **octet** is a byte in the context of networking and file formats, representing exactly 8 bits.

**Technical Definition:** A bit is the smallest unit of digital information, having exactly one of two possible values: 0 or 1. A byte is a sequence of 8 bits, capable of representing 256 distinct values (0–255). The term "octet" is used in protocol specifications (e.g., TCP/IP RFCs) to mean a byte, to avoid ambiguity on platforms where "byte" might not be exactly 8 bits. A Buffer is an "octet stream" — a sequence of these 8-bit values.

**Beginner-Friendly Explanation:** Think of a bit as a single light switch (on or off). A byte is a row of 8 light switches, which together can represent 256 different combinations. When Node.js talks about an "octet stream," it means a continuous flow of these 8-switch groups — like a binary alphabet with 256 different letters.

#### Purposes

- To provide the foundational unit of measurement for binary data representation.
- To enable precise specification of data sizes in network protocols and file formats.
- To clarify the distinction between JavaScript strings (Unicode) and binary data.

#### Syntax Rules and Structure

| Unit | Size | Range | Usage |
|------|------|-------|-------|
| Bit | 1 binary digit | `0` or `1` | Boolean logic, flags |
| Byte | 8 bits | `0`–`255` | Buffer element, file I/O |
| Octet | 8 bits (network term) | `0`–`255` | Protocol specifications |

**Constraints and Limitations:**
- JavaScript numbers can represent integers up to 2^53 safely, but Buffer elements are always 8-bit.
- A Buffer's `length` property is the number of bytes, not the number of characters (for multi-byte encodings).

#### Annotated Code Example

```js
// bytes-basics.js
// A single byte can hold 256 values
const buf = Buffer.alloc(3);

buf[0] = 0;      // minimum byte value
buf[1] = 127;    // middle byte value
buf[2] = 255;    // maximum byte value

console.log('Bytes:', buf);
// → <Buffer 00 7f ff>

// Bitwise representation of a byte
const byte = 0b10101010;
console.log('Binary:', byte.toString(2).padStart(8, '0'));
// → '10101010'
console.log('Decimal:', byte);
// → 170
```

**Expected Output:**
```
Bytes: <Buffer 00 7f ff>
Binary: 10101010
Decimal: 170
```

**Why this output:** Each slot in the Buffer holds one byte. The values 0, 127, and 255 display in hexadecimal as `00`, `7f`, and `ff`. The binary representation shows the 8 individual bits of the decimal number 170.

### Sub-Feature 1.2: How V8 Handles Memory vs. Node.js Buffers

#### Definitions

**Core Definition:** V8 manages JavaScript objects and primitive values in a garbage-collected heap, while Buffer memory is allocated outside the V8 heap as raw C++ memory managed by Node.js.

**Technical Definition:** V8 allocates memory for JavaScript objects, strings, and numbers within its own heap, which is subject to generational garbage collection. Buffer instances, by contrast, correspond to fixed-sized, raw memory allocations outside the V8 heap. This memory is allocated at the C++ layer and is eventually reclaimed by V8's garbage collector, but the allocation itself bypasses the V8 heap entirely. This design allows Buffers to handle large binary data without placing pressure on V8's garbage collector.

**Beginner-Friendly Explanation:** V8's heap is like a library with a librarian (garbage collector) who periodically cleans up unused books. Buffers are like a separate warehouse outside the library — you can store large items there without the librarian having to manage them. This keeps the library running smoothly.

#### Purposes

- To avoid placing large binary data under V8's garbage collection pressure.
- To enable efficient memory operations on raw byte sequences.
- To provide predictable memory behaviour for I/O-intensive applications.

#### Syntax Rules and Structure

| Aspect | V8 Heap | Buffer Memory |
|--------|---------|---------------|
| Location | Inside V8's managed heap | Outside V8 heap (C++ level) |
| Collection | Generational GC | Deferred deallocation |
| Size limit | Heap limit (e.g., ~4GB default) | `buffer.constants.MAX_LENGTH` |
| Access | JavaScript objects | Uint8Array-like interface |

**Constraints and Limitations:**
- Buffer memory is not immediately freed when garbage collected; deallocation may be deferred.
- Buffer memory is not zeroed by default when using `allocUnsafe`; sensitive data may remain.
- Large Buffers may cause memory fragmentation outside V8.

#### Annotated Code Example

```js
// memory-location.js
const v8 = require('node:v8');

// V8 heap statistics
const heapStats = v8.getHeapStatistics();
console.log('V8 heap size limit:', (heapStats.heap_size_limit / 1024 / 1024).toFixed(0), 'MB');

// Buffer is allocated outside this heap
const buf = Buffer.alloc(1024 * 1024); // 1 MB
console.log('Buffer length:', buf.length, 'bytes');
console.log('Buffer is Uint8Array:', buf instanceof Uint8Array);
// → true

// Buffer memory is NOT reflected in V8 heap usage
console.log('Heap used (MB):', (process.memoryUsage().heapUsed / 1024 / 1024).toFixed(1));
// → e.g., 5.2 MB (does not include the 1 MB Buffer)
```

**Expected Output:**
```
V8 heap size limit: 4144 MB
Buffer length: 1048576 bytes
Buffer is Uint8Array: true
Heap used (MB): 5.2
```

**Why this output:** The V8 heap limit is typically several gigabytes. The 1 MB Buffer is not counted in `heapUsed` because it lives outside the V8 heap. The `instanceof Uint8Array` check confirms that Buffer inherits from the TypedArray API.

#### Real-World Cases

- **File I/O:** Reading a 500 MB file into a Buffer does not trigger V8 heap pressure.
- **Network servers:** Streaming data through Buffers keeps the garbage collector idle.
- **Cryptography:** Hash computations operate on raw Buffers outside the V8 heap.

---

## Core Concept 2: Buffer Creation & Memory Management

### Sub-Feature 2.1: `Buffer.alloc()` vs. `Buffer.allocUnsafe()` (and Security Risks)

#### Definitions

**Core Definition:** `Buffer.alloc(size)` creates a zero-filled Buffer safely, while `Buffer.allocUnsafe(size)` creates an uninitialized Buffer that may contain leftover data from previous operations — a potential security risk.

**Technical Definition:** `Buffer.alloc(size[, fill[, encoding]])` allocates a new Buffer of `size` bytes. If `fill` is `undefined`, the Buffer will be zero-filled. This method is slower than `Buffer.allocUnsafe(size)` but guarantees that newly created Buffer instances never contain old data that is potentially sensitive. `Buffer.allocUnsafe(size)` allocates a new Buffer of `size` bytes. The underlying memory for Buffer instances created in this way is not initialized; the contents are unknown and may contain sensitive data. Use `buf.fill(0)` to initialize such Buffer instances to zeroes.

**Beginner-Friendly Explanation:** `Buffer.alloc` is like getting a brand-new, blank piece of paper. `Buffer.allocUnsafe` is like getting a piece of paper from a recycling bin — it might have someone else's notes on it. It's faster because nobody had to erase it first, but you might accidentally see something you shouldn't. Always use `allocUnsafe` only when you're going to overwrite every byte immediately.

#### Purposes

- To allocate safe, zero-initialized memory for predictable behaviour.
- To allocate fast, uninitialized memory for performance-critical paths where every byte will be overwritten.
- To understand and mitigate the security risks of uninitialized memory exposure.

#### Syntax Rules and Structure

**`Buffer.alloc`:**
```js
const buf = Buffer.alloc(size, fill?, encoding?);
```
| Component | Breakdown |
|-----------|-----------|
| `size` | Desired length in bytes. |
| `fill` | Optional value to pre-fill (default: `0`). |
| `encoding` | Optional encoding for `fill` if it's a string. |

**`Buffer.allocUnsafe`:**
```js
const buf = Buffer.allocUnsafe(size);
```
| Component | Breakdown |
|-----------|-----------|
| `size` | Desired length in bytes. |
| Returns | Buffer with uninitialized memory. |
| Pool | Uses the internal pool if `size <= Buffer.poolSize >>> 1`. |

**Constraints and Limitations:**
- `Buffer.alloc(size, fill)` never uses the internal Buffer pool.
- `Buffer.allocUnsafe(size).fill(fill)` uses the internal pool if `size <= Buffer.poolSize >>> 1`.
- Uninitialized memory may contain sensitive data (passwords, tokens, cryptographic keys).
- CVE-2025-55131: A flaw in Node.js's buffer allocation logic can expose uninitialized memory when allocations are interrupted.

#### Annotated Code Example

```js
// alloc-vs-allocunsafe.js
// Safe allocation — zero-filled
const safe = Buffer.alloc(5);
console.log('alloc:', safe);
// → <Buffer 00 00 00 00 00>

// Unsafe allocation — contents unknown
const unsafe = Buffer.allocUnsafe(5);
console.log('allocUnsafe:', unsafe);
// → <Buffer ?? ?? ?? ?? ??> (contents vary)

// Demonstrate the risk: reusing memory from a "secret"
const secret = Buffer.from('password123');
secret.fill(0); // Simulate cleanup, but memory may not be erased
const risky = Buffer.allocUnsafe(11);
// risky MIGHT contain remnants of 'password123'

// Safe pattern: always overwrite allocUnsafe buffers
const safeAfterWrite = Buffer.allocUnsafe(5);
safeAfterWrite.fill(0); // Or write every byte before use
console.log('After fill:', safeAfterWrite);
// → <Buffer 00 00 00 00 00>
```

**Expected Output:**
```
alloc: <Buffer 00 00 00 00 00>
allocUnsafe: <Buffer f8 3a 00 00 00>
After fill: <Buffer 00 00 00 00 00>
```

**Why this output:** `Buffer.alloc(5)` returns five zero bytes. `Buffer.allocUnsafe(5)` returns five bytes with whatever values happened to be in that memory location (shown here as `f8 3a 00 00 00`, but this varies per run). The `fill(0)` call zeroes the buffer, making it safe.

### Sub-Feature 2.2: `Buffer.from()` for Strings, Arrays, and ArrayBuffers

#### Definitions

**Core Definition:** `Buffer.from()` creates a new Buffer from an existing data source — a string, an array of bytes, or an ArrayBuffer.

**Technical Definition:** `Buffer.from(string[, encoding])` creates a new Buffer containing the UTF-8 encoded bytes of the string (or another specified encoding). `Buffer.from(array)` allocates a new Buffer using an array of bytes in the range 0–255; array entries outside that range are truncated. `Buffer.from(arrayBuffer[, byteOffset[, length]])` creates a Buffer that shares the same memory as the given ArrayBuffer, rather than copying it. `Buffer.from(buffer)` copies the data from the given Buffer into a new Buffer.

**Beginner-Friendly Explanation:** `Buffer.from` is like a copy machine with different input options. You can feed it a string and it converts it to bytes. You can feed it an array of numbers and it turns them into bytes. You can feed it an ArrayBuffer and it creates a Buffer that shares the same memory.

#### Purposes

- To convert strings into their binary byte representation.
- To create Buffers from existing byte arrays.
- To create Buffers that share memory with ArrayBuffers for zero-copy operations.
- To copy data from one Buffer to another.

#### Syntax Rules and Structure

```js
Buffer.from(string, encoding?);
Buffer.from(array);
Buffer.from(arrayBuffer, byteOffset?, length?);
Buffer.from(buffer);
```
| Overload | Breakdown |
|----------|-----------|
| `string` | String to encode. Default encoding: `'utf8'`. |
| `array` | Array of integers 0–255. Values outside range are truncated. |
| `arrayBuffer` | ArrayBuffer or SharedArrayBuffer to share memory with. |
| `buffer` | Existing Buffer to copy. |

**Constraints and Limitations:**
- `Buffer.from(array)` and `Buffer.from(string)` may use the internal Buffer pool like `Buffer.allocUnsafe()` does.
- `Buffer.from(arrayBuffer)` shares memory; modifications to one affect the other.
- `Buffer.from(buffer)` copies data; modifications do not affect the original.

#### Annotated Code Example

```js
// buffer-from.js
// From string (UTF-8)
const fromString = Buffer.from('Hello');
console.log('From string:', fromString);
// → <Buffer 48 65 6c 6c 6f>

// From string with encoding
const fromBase64 = Buffer.from('SGVsbG8=', 'base64');
console.log('From base64:', fromBase64);
// → <Buffer 48 65 6c 6c 6f>

// From array
const fromArray = Buffer.from([72, 101, 108, 108, 111]);
console.log('From array:', fromArray);
// → <Buffer 48 65 6c 6c 6f>

// From ArrayBuffer (shares memory)
const ab = new ArrayBuffer(4);
const fromAB = Buffer.from(ab);
fromAB[0] = 255;
console.log('ArrayBuffer byte 0:', new Uint8Array(ab)[0]);
// → 255 (shared memory)

// From existing Buffer (copies)
const original = Buffer.from('ABC');
const copy = Buffer.from(original);
original[0] = 88; // 'X'
console.log('Original:', original.toString());
// → 'XBC'
console.log('Copy:', copy.toString());
// → 'ABC' (unchanged)
```

**Expected Output:**
```
From string: <Buffer 48 65 6c 6c 6f>
From base64: <Buffer 48 65 6c 6c 6f>
From array: <Buffer 48 65 6c 6c 6f>
ArrayBuffer byte 0: 255
Original: XBC
Copy: ABC
```

**Why this output:** `Buffer.from('Hello')` produces the UTF-8 bytes 48 65 6c 6c 6f. The base64 string `'SGVsbG8='` decodes to the same bytes. `Buffer.from(arrayBuffer)` shares memory, so writing `255` to `fromAB[0]` is visible through the original `Uint8Array`. `Buffer.from(buffer)` copies, so modifying the original does not affect the copy.

### Sub-Feature 2.3: Buffer Pooling and `Buffer.poolSize` Internal Architecture

#### Definitions

**Core Definition:** The Buffer module pre-allocates an internal Buffer of size `Buffer.poolSize` (default 8192 bytes) and uses it as a pool for fast allocation of small Buffers.

**Technical Definition:** The Buffer module pre-allocates an internal Buffer instance of size `Buffer.poolSize` that is used as a pool for the fast allocation of new Buffer instances created using `Buffer.allocUnsafe()`, `Buffer.from(array)`, and `Buffer.concat()` only when size is less than `Buffer.poolSize >>> 1` (floor of `Buffer.poolSize` divided by two). This allows applications to avoid the garbage collection overhead of creating many individually allocated Buffer instances, improving both performance and memory usage.

**Beginner-Friendly Explanation:** Instead of giving every small Buffer its own piece of memory, Node.js grabs one big chunk (8 KB by default) and hands out slices of it. This is like a pizza — instead of baking a tiny pizza for each person, you bake one large pizza and cut slices. It's much faster and wastes less.

#### Purposes

- To reduce garbage collection overhead from many small Buffer allocations.
- To improve memory allocation performance for small Buffers.
- To provide a tunable pool size for different workload characteristics.

#### Syntax Rules and Structure

```js
Buffer.poolSize = 8192; // default, modifiable
```
| Component | Breakdown |
|-----------|-----------|
| `Buffer.poolSize` | Size in bytes of the internal pool (default: 8192). |
| Pool threshold | Buffers ≤ `Buffer.poolSize >>> 1` (4096) use the pool. |
| Methods using pool | `allocUnsafe()`, `from(array)`, `from(string)`, `concat()`. |

**Constraints and Limitations:**
- `Buffer.alloc(size, fill)` never uses the internal pool.
- `Buffer.allocUnsafeSlow(size)` explicitly bypasses the pool.
- A Buffer from the pool may keep the entire pool alive if a reference to the pool is retained.

#### Annotated Code Example

```js
// buffer-pooling.js
console.log('Pool size:', Buffer.poolSize);
// → 8192

// Small buffers use the pool
const small1 = Buffer.allocUnsafe(100);
const small2 = Buffer.allocUnsafe(100);
console.log('Small buffer byteOffset:', small1.byteOffset);
console.log('Small buffer byteOffset:', small2.byteOffset);
// → Different offsets within the same pool

// Large buffer bypasses the pool
const large = Buffer.allocUnsafe(5000);
console.log('Large buffer byteOffset:', large.byteOffset);
// → 0 (own ArrayBuffer)

// alloc with fill never uses the pool
const filled = Buffer.alloc(100, 0xFF);
console.log('Filled byteOffset:', filled.byteOffset);
// → 0 (own ArrayBuffer)

// allocUnsafeSlow bypasses the pool
const slow = Buffer.allocUnsafeSlow(100);
console.log('Slow byteOffset:', slow.byteOffset);
// → 0 (own ArrayBuffer)
```

**Expected Output:**
```
Pool size: 8192
Small buffer byteOffset: 8
Small buffer byteOffset: 108
Large buffer byteOffset: 0
Filled byteOffset: 0
Slow byteOffset: 0
```

**Why this output:** `small1` and `small2` have different `byteOffset` values (8 and 108), indicating they are slices of the same pooled ArrayBuffer. `large` (5000 bytes > 4096 threshold) has `byteOffset` 0 because it received its own dedicated ArrayBuffer. `filled` uses `alloc` with a fill value, which never uses the pool. `slow` explicitly bypasses the pool.

### Real-World Cases

- **High-throughput servers:** Using `allocUnsafe` with pooling for per-request buffers.
- **Security-sensitive applications:** Preferring `alloc` to avoid exposing uninitialized memory.
- **Memory-constrained environments:** Tuning `Buffer.poolSize` to balance performance and memory.
- **Long-lived small buffers:** Using `allocUnsafeSlow` when retaining small chunks from a pool for an indeterminate time.

---

## Core Concept 3: Encoding & Decoding

### Sub-Feature 3.1: Supported Encodings

#### Definitions

**Core Definition:** Node.js supports several character encodings for converting between Buffers and strings, including `utf8`, `ascii`, `base64`, `hex`, and `latin1`.

**Technical Definition:** When creating a Buffer from a string, or converting a Buffer to a string, an encoding may be specified. The following encodings are supported: `'utf8'` (aliases: `'utf-8'`) — multibyte encoded Unicode characters, used by most web pages; `'utf16le'` (aliases: `'utf-16le'`, `'ucs2'`, `'ucs-2'`) — 2 or 4 bytes, little-endian encoded Unicode characters, supporting surrogate pairs; `'latin1'` (alias: `'binary'`) — a way of encoding the Buffer into a one-byte encoded string as defined by IANA in RFC 1345, page 63; `'base64'` — Base64 string encoding; `'base64url'` — URL-safe Base64 encoding; `'hex'` — encode each byte as two hexadecimal characters; `'ascii'` — for 7-bit ASCII data only, stripping the high bit if set.

**Beginner-Friendly Explanation:** Encodings are like different languages for writing down bytes as text. `utf8` is the universal language (handles all characters). `base64` is like writing binary in a safe alphabet (A–Z, a–z, 0–9, +, /) so it can be sent through email or stored in JSON. `hex` writes each byte as two characters (0–9, a–f). `latin1` is an old Western European encoding that maps each byte to one character.

#### Purposes

- To convert strings to Buffers using the correct character representation.
- To convert Buffers to strings for display, storage, or transmission.
- To encode binary data as text for JSON, URLs, or email.
- To decode text data received from files or networks.

#### Syntax Rules and Structure

**Creating a Buffer from a string:**
```js
Buffer.from(string, encoding?); // default: 'utf8'
```

**Converting a Buffer to a string:**
```js
buf.toString(encoding?, start?, end?);
```

| Encoding | Description | Bytes per char |
|----------|-------------|---------------|
| `utf8` | Unicode UTF-8 | 1–4 |
| `utf16le` | UTF-16 little-endian | 2–4 |
| `latin1` | ISO-8859-1 | 1 |
| `base64` | Base64 | 4 chars per 3 bytes |
| `base64url` | URL-safe Base64 | 4 chars per 3 bytes |
| `hex` | Hexadecimal | 2 chars per byte |
| `ascii` | 7-bit ASCII | 1 |

**Constraints and Limitations:**
- `'binary'` is deprecated; use `'latin1'` instead.
- `'latin-1'` (with hyphen) is **not** supported; use `'latin1'`.
- `'ascii'` strips the high bit; do not use for binary data.
- `utf8` is not suitable for arbitrary binary data (e.g., images); use `base64` or `hex` instead.

#### Annotated Code Example

```js
// encodings.js
const original = 'Hello, 世界! 🌍';

// UTF-8 (default)
const utf8Buf = Buffer.from(original, 'utf8');
console.log('UTF-8 bytes:', utf8Buf.length);
// → 24

// Base64
const base64 = utf8Buf.toString('base64');
console.log('Base64:', base64);
// → 'SGVsbG8sIOS4lueVjCEg8J+MjQ=='

// Hex
const hex = utf8Buf.toString('hex');
console.log('Hex:', hex);
// → '48656c6c6f2c20e4b896e7958c2120f09f8c8d'

// Latin1 (cannot represent non-Latin1 characters)
const latin1 = Buffer.from('café', 'latin1');
console.log('Latin1 bytes:', latin1);
// → <Buffer 63 61 66 e9>

// Round-trip: hex → Buffer → string
const restored = Buffer.from(hex, 'hex').toString('utf8');
console.log('Restored:', restored);
// → 'Hello, 世界! 🌍'
```

**Expected Output:**
```
UTF-8 bytes: 24
Base64: SGVsbG8sIOS4lueVjCEg8J+MjQ==
Hex: 48656c6c6f2c20e4b896e7958c2120f09f8c8d
Latin1 bytes: <Buffer 63 61 66 e9>
Restored: Hello, 世界! 🌍
```

**Why this output:** The string contains multi-byte UTF-8 characters, resulting in 24 bytes. Base64 encodes those 24 bytes as 32 characters. Hex encodes each byte as two characters (48 hex characters). `latin1` can only represent characters in the ISO-8859-1 range; `é` is `0xe9`. The round-trip via hex restores the original string exactly.

### Sub-Feature 3.2: Handling Multi-Byte Characters with `StringDecoder`

#### Definitions

**Core Definition:** The `StringDecoder` class decodes Buffer objects into strings while preserving encoded multi-byte UTF-8 and UTF-16 characters that may be split across multiple Buffer chunks.

**Technical Definition:** The `node:string_decoder` module provides an API for decoding Buffer objects into strings in a manner that preserves encoded multi-byte UTF-8 and UTF-16 characters. When a Buffer instance is written to the StringDecoder instance, an internal buffer is used to ensure that the decoded string does not contain any incomplete multibyte characters. These are held in the buffer until the next call to `stringDecoder.write()` or until `stringDecoder.end()` is called.

**Beginner-Friendly Explanation:** Imagine a multi-byte character (like `€`) is split across two data packets — the first packet has the first byte, and the second packet has the remaining two bytes. If you decoded each packet separately, you'd get garbage. `StringDecoder` holds onto the incomplete bytes until the rest arrives, then decodes the complete character.

#### Purposes

- To correctly decode multi-byte characters split across Buffer chunks.
- To prevent garbled text when processing streamed data.
- To handle UTF-8 and UTF-16 encodings safely in chunked I/O.

#### Syntax Rules and Structure

```js
const { StringDecoder } = require('node:string_decoder');
const decoder = new StringDecoder('utf8');
const str = decoder.write(buffer);
const final = decoder.end(buffer?);
```
| Component | Breakdown |
|-----------|-----------|
| `new StringDecoder([encoding])` | Creates a decoder; default encoding `'utf8'`. |
| `decoder.write(buffer)` | Decodes as much as possible, holding incomplete bytes. |
| `decoder.end(buffer?)` | Flushes any remaining bytes. |

**Constraints and Limitations:**
- If an incomplete character is written and `end()` is called, the incomplete bytes are replaced with the Unicode replacement character (`U+FFFD`).
- `StringDecoder` only handles UTF-8, UTF-16LE, and latin1 encodings.
- Without `StringDecoder`, naive `buffer.toString()` on each chunk may produce incorrect characters.

#### Annotated Code Example

```js
// string-decoder.js
const { StringDecoder } = require('node:string_decoder');

// The Euro symbol (€) is 3 bytes in UTF-8: E2 82 AC
const decoder = new StringDecoder('utf8');

// Simulate receiving the bytes in three separate chunks
const chunk1 = Buffer.from([0xE2]);
const chunk2 = Buffer.from([0x82]);
const chunk3 = Buffer.from([0xAC]);

console.log('Without decoder:');
console.log(chunk1.toString('utf8')); // → '' (incomplete)
console.log(chunk2.toString('utf8')); // → '' (incomplete)
console.log(chunk3.toString('utf8')); // → '¬' (wrong character)

console.log('With StringDecoder:');
const decoder2 = new StringDecoder('utf8');
process.stdout.write(decoder2.write(chunk1)); // → '' (held)
process.stdout.write(decoder2.write(chunk2)); // → '' (held)
process.stdout.write(decoder2.end(chunk3));   // → '€' (correct)
console.log();
```

**Expected Output:**
```
Without decoder:

¬
With StringDecoder:
€
```

**Why this output:** The three bytes of `€` (`E2 82 AC`) are split across three Buffers. Without the decoder, each byte is decoded independently, producing empty strings and then the wrong character `¬`. With `StringDecoder`, the first two bytes are held in an internal buffer until the third byte arrives, at which point the complete `€` character is decoded correctly.

### Real-World Cases

- **Streaming text files:** Decoding UTF-8 text from `fs.createReadStream` without corrupting multi-byte characters.
- **Network protocols:** Handling text-based protocols (HTTP headers, WebSocket text frames) where characters may split across TCP packets.
- **Terminal output:** Decoding UTF-8 output from child processes.

---

## Core Concept 4: Buffer Manipulation

### Sub-Feature 4.1: Reading/Writing Specific Byte Types

#### Definitions

**Core Definition:** Buffer provides methods to read and write specific numeric types (unsigned/signed integers of various sizes, floats) at specific byte offsets, with big-endian or little-endian byte order.

**Technical Definition:** Buffer methods such as `buf.readUInt32BE(offset)`, `buf.writeFloatLE(value, offset)`, `buf.readInt16LE(offset)`, and `buf.writeBigUInt64BE(value, offset)` allow precise reading and writing of numeric values at specified offsets. Endianness is indicated by the suffix: `BE` for big-endian (most significant byte first), `LE` for little-endian (least significant byte first). Offsets must satisfy `0 <= offset <= buf.length - byteLength`.

**Beginner-Friendly Explanation:** A Buffer is a row of bytes. These methods let you read or write numbers of different sizes at specific positions. For example, `readUInt32BE(0)` reads 4 bytes starting at position 0 and interprets them as an unsigned 32-bit integer in big-endian order. This is essential for parsing binary file formats and network protocols.

#### Purposes

- To parse binary file headers and network protocol packets.
- To write binary data with a specific numeric format.
- To interoperate with systems that use specific byte orders.
- To read/write floating-point numbers for scientific data.

#### Syntax Rules and Structure

**Reading methods:**
```js
buf.readUInt32BE(offset);  // unsigned 32-bit big-endian
buf.readInt16LE(offset);   // signed 16-bit little-endian
buf.readFloatLE(offset);   // 32-bit float little-endian
buf.readDoubleBE(offset);  // 64-bit double big-endian
buf.readBigUInt64LE(offset); // BigInt 64-bit little-endian
```

**Writing methods:**
```js
buf.writeUInt32BE(value, offset);
buf.writeFloatLE(value, offset);
buf.writeInt16LE(value, offset);
buf.writeBigUInt64BE(value, offset);
```

| Suffix | Meaning |
|--------|---------|
| `BE` | Big-endian (network byte order) |
| `LE` | Little-endian (x86 byte order) |

**Constraints and Limitations:**
- The `noAssert` parameter was **removed** in Node.js v10.0.0; offsets are now always validated.
- Out-of-range offsets throw `ERR_OUT_OF_RANGE`.
- `readBigUInt64BE`/`LE` returns a `BigInt`, not a Number.
- `readFloatLE` and `readDoubleLE` follow IEEE 754 floating-point format.

#### Annotated Code Example

```js
// read-write-numbers.js
const buf = Buffer.alloc(16);

// Write a 32-bit unsigned integer in big-endian
buf.writeUInt32BE(0x12345678, 0);
console.log('Bytes 0-3:', buf.subarray(0, 4));
// → <Buffer 12 34 56 78>

// Write a 16-bit signed integer in little-endian
buf.writeInt16LE(-1000, 4);
console.log('Bytes 4-5:', buf.subarray(4, 6));
// → <Buffer 18 fc> (-1000 = 0xFC18 in LE)

// Write a 32-bit float in little-endian
buf.writeFloatLE(3.14, 6);
console.log('Bytes 6-9:', buf.subarray(6, 10));
// → <Buffer c3 f5 48 40>

// Read them back
console.log('UInt32BE:', buf.readUInt32BE(0));
// → 305419896 (0x12345678)
console.log('Int16LE:', buf.readInt16LE(4));
// → -1000
console.log('FloatLE:', buf.readFloatLE(6));
// → 3.140000104904175 (float precision)
```

**Expected Output:**
```
Bytes 0-3: <Buffer 12 34 56 78>
Bytes 4-5: <Buffer 18 fc>
Bytes 6-9: <Buffer c3 f5 48 40>
UInt32BE: 305419896
Int16LE: -1000
FloatLE: 3.140000104904175
```

**Why this output:** `writeUInt32BE` writes the bytes in big-endian order (most significant first). `writeInt16LE` writes -1000 as two's complement in little-endian (`18 fc`). `writeFloatLE` stores the IEEE 754 representation of 3.14. Reading back produces the expected values, with the float showing slight precision loss (a known characteristic of 32-bit floats).

### Sub-Feature 4.2: Copying, Slicing, and Concatenating

#### Definitions

**Core Definition:** `buf.copy()` copies bytes between Buffers, `buf.subarray()` returns a new Buffer that shares memory with the original (a view), and `Buffer.concat()` combines multiple Buffers into one.

**Technical Definition:** `buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])` copies data from a region of `buf` to a region in `target`, even if the `target` memory region overlaps with `buf`. `buf.subarray([start[, end]])` returns a new Buffer that references the same memory as the original, offset and cropped by `start` and `end`. This is a **view**, not a copy — modifications to the subarray affect the original Buffer. `Buffer.concat(list[, totalLength])` returns a new Buffer which is the result of concatenating all the Buffer instances in the `list` together.

**Beginner-Friendly Explanation:** `copy` is like photocopying pages from one notebook to another — the original and copy are independent. `subarray` is like putting a window over part of a page — you're looking at the same content, just a smaller view. `concat` is like stapling multiple pages together into one long scroll.

#### Purposes

- To duplicate data between Buffers without manual byte-by-byte copying.
- To create lightweight views of Buffer segments without copying memory.
- To combine multiple chunks of data into a single Buffer for processing.

#### Syntax Rules and Structure

**Copy:**
```js
buf.copy(target, targetStart?, sourceStart?, sourceEnd?);
```
| Component | Breakdown |
|-----------|-----------|
| `target` | Buffer to copy into. |
| `targetStart` | Offset in target to begin writing (default: 0). |
| `sourceStart` | Offset in source to begin reading (default: 0). |
| `sourceEnd` | Offset in source to stop reading (default: `buf.length`). |

**Subarray:**
```js
const view = buf.subarray(start?, end?);
```
| Component | Breakdown |
|-----------|-----------|
| `start` | Start index (inclusive); default: 0. |
| `end` | End index (exclusive); default: `buf.length`. |
| Returns | A view sharing memory with the original. |

**Concat:**
```js
Buffer.concat(list, totalLength?);
```
| Component | Breakdown |
|-----------|-----------|
| `list` | Array of Buffers to concatenate. |
| `totalLength` | Optional total length; if omitted, calculated from `list`. |

**Constraints and Limitations:**
- `buf.subarray()` shares memory; modifications affect the original Buffer.
- `buf.slice()` is deprecated; use `buf.subarray()` instead.
- `Buffer.concat()` with a `totalLength` larger than the sum of Buffers fills remaining bytes with zeros.
- Overlapping `buf.copy(buf, ...)` is supported and handles overlap correctly.

#### Annotated Code Example

```js
// copy-slice-concat.js
// --- Copy ---
const source = Buffer.from('Hello World');
const target = Buffer.alloc(5);
source.copy(target, 0, 0, 5);
console.log('Copy:', target.toString());
// → 'Hello'

// --- Subarray (view) ---
const original = Buffer.from('Hello World');
const view = original.subarray(6); // "World"
view[0] = 0x58; // 'X'
console.log('View:', view.toString());
// → 'Xorld'
console.log('Original after view modification:', original.toString());
// → 'Hello Xorld' (original was modified!)

// --- Concat ---
const buf1 = Buffer.from('Hello, ');
const buf2 = Buffer.from('World');
const buf3 = Buffer.from('!');
const combined = Buffer.concat([buf1, buf2, buf3]);
console.log('Concat:', combined.toString());
// → 'Hello, World!'
console.log('Total length:', combined.length);
// → 13
```

**Expected Output:**
```
Copy: Hello
View: Xorld
Original after view modification: Hello Xorld
Concat: Hello, World!
Total length: 13
```

**Why this output:** `copy` creates independent copies. `subarray` creates a view — modifying `view[0]` to `0x58` ('X') changes the original Buffer because they share the same underlying memory. `Buffer.concat` combines the three Buffers into a new Buffer of length 13.

### Sub-Feature 4.3: Shared Memory with TypedArray and SharedArrayBuffer

#### Definitions

**Core Definition:** `SharedArrayBuffer` is a fixed-length binary data buffer that can be shared across multiple worker threads, allowing zero-copy data sharing between threads.

**Technical Definition:** SharedArrayBuffer is used to represent fixed-length binary data buffers that can be shared across multiple workers. The `SharedArrayBuffer` allocated will have an underlying byte buffer whose size is determined by the `byte_length` parameter. The underlying buffer is optionally returned back to the caller in case the caller wants to directly manipulate the buffer. This buffer can only be written to directly from native code. To write to this buffer from JavaScript, a typed array or DataView object would need to be created. All TypedArray and Buffer instances are views over an underlying ArrayBuffer. That is, the ArrayBuffer stores the raw data, while the TypedArray and Buffer objects provide a way to view and manipulate the data. It is also possible to create multiple views over the same ArrayBuffer instance.

**Beginner-Friendly Explanation:** Normally, when you pass data between threads, it gets copied — which is slow for large data. `SharedArrayBuffer` is like a whiteboard that multiple people can see and write on simultaneously. They all look at the same whiteboard, so there's no need to copy anything. `TypedArray` and `Buffer` are like different pairs of glasses that interpret the same whiteboard differently.

#### Purposes

- To share memory between worker threads without copying.
- To enable high-performance parallel data processing.
- To provide a zero-copy path for large binary data.
- To allow multiple views (TypedArray, Buffer) over the same memory.

#### Syntax Rules and Structure

**Creating a SharedArrayBuffer:**
```js
const sab = new SharedArrayBuffer(1024); // 1024 bytes
```

**Creating views:**
```js
const uint8 = new Uint8Array(sab);
const int32 = new Int32Array(sab);
const buffer = Buffer.from(sab);
```

**Sending to a worker:**
```js
const worker = new Worker('worker.js');
worker.postMessage({ sab });
```

| Component | Breakdown |
|-----------|-----------|
| `SharedArrayBuffer` | Raw memory shared between threads. |
| `TypedArray` | View over the buffer with a specific element type. |
| `Buffer.from(sab)` | Creates a Buffer view over the SharedArrayBuffer. |
| `Atomics` | Provides atomic operations for thread-safe access. |

**Constraints and Limitations:**
- `SharedArrayBuffer` is not available in all environments due to security restrictions (Spectre mitigations).
- Access from multiple threads requires synchronization primitives (e.g., `Atomics.wait`, `Atomics.notify`).
- Buffer views over SharedArrayBuffer are not automatically synchronized.
- Node.js supports SharedArrayBuffer but with some web API incompatibilities.

#### Annotated Code Example

```js
// shared-memory.js
const { Worker, isMainThread, workerData } = require('node:worker_threads');

if (isMainThread) {
  // Main thread: create shared buffer
  const sab = new SharedArrayBuffer(16);
  const view = new Uint8Array(sab);
  view[0] = 42;

  // Spawn worker with shared buffer
  const worker = new Worker(__filename, { workerData: sab });

  worker.on('message', (msg) => {
    console.log('Main thread sees:', msg);
    // → 142
    console.log('View[0] is now:', view[0]);
    // → 142
  });
} else {
  // Worker thread: modify shared buffer
  const view = new Uint8Array(workerData);
  view[0] += 100; // 42 + 100 = 142
  parentPort.postMessage(view[0]);
}
```

**Expected Output:**
```
Main thread sees: 142
View[0] is now: 142
```

**Why this output:** The `SharedArrayBuffer` is created on the main thread and passed to the worker via `workerData`. Both threads have views over the same memory. The worker modifies `view[0]` from 42 to 142, and the main thread sees the change immediately because the memory is shared — no copying occurred.

### Real-World Cases

- **Image processing:** Worker threads share pixel data via `SharedArrayBuffer`.
- **Real-time audio:** Multiple threads share audio sample buffers.
- **Scientific computing:** Parallel numerical computations share large matrices.
- **Game engines:** Physics and rendering threads share world state.

---

## Core Concept 5: Binary Protocols

### Definitions

**Core Definition:** Binary protocol parsing involves reading a stream of bytes according to a predefined format to extract structured data such as message types, lengths, and payloads.

**Technical Definition:** Binary protocols typically define a frame structure consisting of fixed-size header fields (e.g., message type, payload length) followed by a variable-length payload. When data arrives over TCP, it may be split across multiple `data` events. A robust parser must buffer incoming chunks, detect complete packets, and extract them for processing. Buffers are the primary tool for this: `Buffer.concat` accumulates partial data, and methods like `readUInt32BE` extract header fields.

**Beginner-Friendly Explanation:** Imagine you're receiving a series of postcards, but the postal service delivers them in random-sized bundles. Sometimes a postcard is split across two bundles. A binary protocol parser is like a postal worker who collects all the pieces, figures out where each postcard starts and ends, and delivers complete postcards to the right desk.

### Purposes

- To parse custom network protocols that use binary framing.
- To extract structured data from binary file formats (e.g., PNG, ZIP, Protocol Buffers).
- To handle TCP's stream nature where packets do not align with message boundaries.
- To validate and process binary data safely.

### Syntax Rules and Structure

**Typical binary packet structure:**
```
[Header: type (4 bytes BE)] [Header: length (4 bytes BE)] [Payload: length bytes]
```

**Parsing pattern:**
```js
let pending = Buffer.alloc(0);

socket.on('data', (chunk) => {
  pending = Buffer.concat([pending, chunk]);
  while (pending.length >= HEADER_SIZE) {
    const type = pending.readUInt32BE(0);
    const length = pending.readUInt32BE(4);
    const totalSize = HEADER_SIZE + length;
    if (pending.length < totalSize) break; // incomplete
    const payload = pending.subarray(HEADER_SIZE, totalSize);
    // Process packet
    pending = pending.subarray(totalSize);
  }
});
```

**Constraints and Limitations:**
- TCP is a stream protocol; message boundaries are not preserved.
- A single `data` event may contain multiple packets, partial packets, or a combination.
- `Buffer.concat` creates a new Buffer each time, which can be inefficient for high-throughput streams.
- Large packets may require memory limits to prevent denial-of-service attacks.

### Multiple Annotated Code Examples

#### Example 1: Parsing a Simple Length-Prefixed Protocol

```js
// packet-parser.js
const net = require('node:net');

// Protocol: [4-byte BE length][UTF-8 payload]
const server = net.createServer((socket) => {
  let pending = Buffer.alloc(0);

  socket.on('data', (chunk) => {
    pending = Buffer.concat([pending, chunk]);

    while (pending.length >= 4) {
      const length = pending.readUInt32BE(0);
      const totalSize = 4 + length;

      if (pending.length < totalSize) break; // Wait for more data

      const payload = pending.subarray(4, totalSize);
      console.log('Received packet:', payload.toString('utf8'));

      pending = pending.subarray(totalSize);
    }
  });
});

server.listen(3000, () => console.log('Server listening on port 3000'));
```

**Client (to send a packet):**
```js
// client.js
const net = require('node:net');

const client = net.connect(3000, () => {
  const message = Buffer.from('Hello, World!', 'utf8');
  const header = Buffer.alloc(4);
  header.writeUInt32BE(message.length, 0);
  client.write(Buffer.concat([header, message]));
});
```

**Expected Output (server):**
```
Server listening on port 3000
Received packet: Hello, World!
```

**Why this output:** The client prepends a 4-byte big-endian length to the UTF-8 payload. The server accumulates incoming data in `pending`, reads the length, and extracts the payload once enough bytes have arrived. The `while` loop handles multiple packets in a single `data` event.

#### Example 2: Parsing a PNG File Header

```js
// png-header.js
const fs = require('node:fs');

const png = fs.readFileSync('image.png');

// PNG signature: 137 80 78 71 13 10 26 10
const signature = png.subarray(0, 8);
console.log('Signature:', signature);
// → <Buffer 89 50 4e 47 0d 0a 1a 0a>

// First chunk: IHDR (Image Header)
const chunkLength = png.readUInt32BE(8);
const chunkType = png.subarray(12, 16).toString('ascii');
console.log('First chunk length:', chunkLength);
// → 13
console.log('First chunk type:', chunkType);
// → 'IHDR'

// IHDR data (13 bytes)
const width = png.readUInt32BE(16);
const height = png.readUInt32BE(20);
const bitDepth = png.readUInt8(24);
const colorType = png.readUInt8(25);

console.log(`Dimensions: ${width}x${height}`);
console.log('Bit depth:', bitDepth);
console.log('Color type:', colorType);
```

**Expected Output (for a 800x600 PNG):**
```
Signature: <Buffer 89 50 4e 47 0d 0a 1a 0a>
First chunk length: 13
First chunk type: IHDR
Dimensions: 800x600
Bit depth: 8
Color type: 6
```

**Why this output:** The PNG signature is the first 8 bytes. The IHDR chunk follows: 4 bytes for length (13), 4 bytes for type (`'IHDR'`), then 13 bytes of data containing width, height, bit depth, and color type. `readUInt32BE` and `readUInt8` extract these values at the correct offsets.

#### Example 3: Using a Streaming Transform for Protocol Parsing

```js
// transform-parser.js
const { Transform } = require('node:stream');
const net = require('node:net');

class PacketParser extends Transform {
  constructor() {
    super();
    this.pending = Buffer.alloc(0);
  }

  _transform(chunk, encoding, callback) {
    this.pending = Buffer.concat([this.pending, chunk]);

    while (this.pending.length >= 4) {
      const length = this.pending.readUInt32BE(0);
      const totalSize = 4 + length;

      if (this.pending.length < totalSize) break;

      const payload = this.pending.subarray(4, totalSize);
      this.push(payload);
      this.pending = this.pending.subarray(totalSize);
    }

    callback();
  }
}

const server = net.createServer((socket) => {
  const parser = new PacketParser();
  parser.on('data', (payload) => {
    console.log('Packet:', payload.toString('utf8'));
  });
  socket.pipe(parser);
});

server.listen(3000, () => console.log('Server listening on port 3000'));
```

**Expected Output:**
```
Server listening on port 3000
Packet: Hello, World!
```

**Why this output:** The `PacketParser` class extends `Transform` and implements `_transform` to accumulate and parse packets. Incoming socket data is piped through the parser, which emits complete payloads as readable data. This is a more idiomatic streaming approach than manual event handling.

### Real-World Cases

- **Database protocols:** Parsing PostgreSQL, MySQL, or Redis wire protocols.
- **Game networking:** Parsing custom UDP/TCP game packet formats.
- **File format parsers:** Extracting metadata from PNG, JPEG, ZIP, or PDF files.
- **IoT protocols:** Parsing MQTT, CoAP, or custom sensor data formats.
- **RPC frameworks:** Parsing gRPC or Thrift binary framing.

---

## References

- Node.js Documentation — Buffer — https://nodejs.org/api/buffer.html
- Node.js Documentation — `Buffer.alloc()` — https://nodejs.org/api/buffer.html#static-method-bufferallocsize-fill-encoding
- Node.js Documentation — `Buffer.allocUnsafe()` — https://nodejs.org/api/buffer.html#static-method-bufferallocunsafesize
- Node.js Documentation — `Buffer.from()` — https://nodejs.org/api/buffer.html#static-method-bufferfromarray
- Node.js Documentation — `Buffer.poolSize` — https://nodejs.org/api/buffer.html#bufferpoolsize
- Node.js Documentation — `buf.subarray()` — https://nodejs.org/api/buffer.html#bufsubarraystart-end
- Node.js Documentation — `Buffer.concat()` — https://nodejs.org/api/buffer.html#static-method-bufferconcatlist-totallength
- Node.js Documentation — `buf.readUInt32BE()` — https://nodejs.org/api/buffer.html#bufreaduint32beoffset
- Node.js Documentation — Buffer and Character Encodings — https://nodejs.org/api/buffer.html#buffers-and-character-encodings
- Node.js Documentation — String Decoder — https://nodejs.org/api/string_decoder.html
- Node.js Documentation — `SharedArrayBuffer` — https://nodejs.org/api/n-api.html#node_api_create_sharedarraybuffer
- MDN Web Docs — SharedArrayBuffer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer
- MDN Web Docs — TypedArray — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray
- CVE-2025-55131 — Node.js vm Timeout Race Uninitialized Memory Exposure — https://nvd.nist.gov/vuln/detail/CVE-2025-55131
- Node.js Security Releases — January 2026 — https://nodejs.org/en/blog/vulnerability/january-2026-security-releases
- IANA — Character Sets — https://www.iana.org/assignments/character-sets/character-sets.xhtml
- RFC 1345 — Character Mnemonics and Character Sets — https://www.rfc-editor.org/rfc/rfc1345
- node-packet-reader — Length-prefixed binary packet reader — https://github.com/brianc/node-packet-reader
- binary-parser — Binary parser for Node.js — https://github.com/Keichi/binary-parser