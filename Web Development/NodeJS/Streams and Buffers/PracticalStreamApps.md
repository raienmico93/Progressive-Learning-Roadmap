# Node.js High-Performance & Practical Stream Applications — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** High-performance stream applications leverage Node.js's stream architecture to process large volumes of data — files, network traffic, logs — incrementally and with bounded memory usage, enabling production-grade data pipelines for ETL, uploads, compression, encryption, and real-time observability.

**Technical Definition:** Node.js streams provide a composable, backpressure-aware abstraction over chunked data sources and sinks. The `stream.pipeline()` utility connects multiple stream stages (source, transforms, destination) with automatic error propagation and resource cleanup. The `zlib`, `crypto`, and `readline` modules expose Transform and Readable stream implementations that integrate into pipelines for compression, encryption, and line-by-line parsing respectively. Multiplexing protocols (e.g., `mux-demux`) allow multiple logical streams to share a single physical connection.

**Beginner-Friendly Explanation:** Imagine you need to process a 10 GB log file on a laptop with 8 GB of RAM. Reading the whole file at once would crash your program. Streams solve this by processing the file piece by piece — like sipping through a straw instead of trying to drink the entire pool. This cheat sheet covers the practical patterns that make this work at scale: reading huge files line by line, uploading to cloud storage without buffering, compressing and encrypting data on the fly, and running multiple data streams over a single connection.

### Key Characteristics

- **Bounded memory:** Streams process data in chunks, keeping memory usage flat regardless of input size.
- **Backpressure-aware:** Pipelines automatically slow down fast producers when consumers are overwhelmed.
- **Composable:** Readable, Transform, and Writable stages can be chained via `stream.pipeline()`.
- **Error-safe:** `pipeline()` destroys all streams on failure, preventing resource leaks.
- **Protocol-agnostic:** Works with files, HTTP, TCP sockets, cloud storage, and custom binary protocols.
- **Observable:** Streams emit lifecycle events that enable real-time monitoring and log scraping.

### Prerequisites

- **Node.js runtime:** Node.js 18+ is recommended for `stream/promises` and modern APIs.
- **Basic JavaScript knowledge:** Understanding of functions, callbacks, Promises, and `async/await`.
- **Stream fundamentals:** Familiarity with Readable, Writable, Duplex, and Transform stream types.
- **Buffer basics:** Understanding of `Buffer`, encodings, and binary data.
- **Event loop concepts:** How asynchronous operations and backpressure work.

### Related Programming Areas

- **File System (`fs`):** `fs.createReadStream()` and `fs.createWriteStream()` are the foundational file streaming utilities.
- **HTTP (`http`):** Request and response objects are streams, enabling streaming uploads and downloads.
- **Compression (`zlib`):** Gzip, Brotli, and Deflate Transform streams for real-time compression.
- **Cryptography (`crypto`):** Cipher and Hash objects are Transform streams for streaming encryption and hashing.
- **Readline (`readline`):** Line-by-line parsing of text streams for CSV/JSON processing.
- **Cloud Storage (AWS SDK):** Multipart uploads to S3 via `@aws-sdk/lib-storage`.
- **Multiplexing libraries:** `mux-demux`, `bpmux`, `stream-demux` for multiple streams over one connection.

### Core Concepts

1. **Large File & Network Processing** — CSV/JSON line-by-line processing, chunked uploads, and multipart parsing.
2. **Data Modification & Security** — real-time compression and streaming encryption/hashing.
3. **Log Architecture & Observability** — real-time log pipelines and stream multiplexing.

---

## Core Concept 1: Large File & Network Processing

### Sub-Feature 1.1: Processing Multi-Gigabyte CSV/JSON Files Line-by-Line

#### Definitions

**Core Definition:** Line-by-line processing reads a large text file one line at a time using `readline.createInterface()` over a `fs.createReadStream()`, keeping memory usage flat regardless of file size.

**Technical Definition:** The `node:readline` module provides an interface for reading data from a Readable stream one line at a time. `readline.createInterface({ input: stream })` returns an interface that emits a `'line'` event for each line and supports async iteration via `for await...of`. This is the standard approach for processing multi-gigabyte CSV or JSONL files without loading them into memory. A 10 GB file can be processed with less than 50 MB of memory using this pattern.

**Beginner-Friendly Explanation:** Instead of reading the entire file into memory (which would crash on a 2 GB file), you read it line by line — like reading a book page by page instead of trying to memorise the whole thing at once. Each line is processed and discarded before the next one is read.

#### Purposes

- To process files larger than available memory without crashing.
- To parse CSV or JSONL files row by row for ETL, analysis, or import.
- To maintain flat memory usage regardless of input size.
- To integrate line-by-line parsing into larger stream pipelines.

#### Syntax Rules and Structure

```js
const { createReadStream } = require('node:fs');
const { createInterface } = require('node:readline');

const rl = createInterface({
  input: createReadStream('large-file.csv'),
  crlfDelay: Infinity, // Handle \r\n and \n consistently
});

for await (const line of rl) {
  // Process one line at a time
}
```

| Component | Breakdown |
|-----------|-----------|
| `createReadStream(path)` | Creates a Readable stream from the file. |
| `createInterface({ input })` | Wraps the stream in a line-reading interface. |
| `crlfDelay: Infinity` | Treats `\r\n` as a single line break (Windows compatibility). |
| `for await...of` | Async iteration over lines (memory-efficient). |

**Constraints and Limitations:**
- The default `crlfDelay` may split lines incorrectly on Windows files; set to `Infinity`.
- Parsing CSV with quoted fields containing newlines requires a proper CSV parser (e.g., `csv-parse`).
- Each line is a string; for binary data, use raw stream chunks instead.

#### Annotated Code Example

```js
// csv-stream.js
const { createReadStream } = require('node:fs');
const { createInterface } = require('node:readline');

async function processCSV(filePath) {
  const fileStream = createReadStream(filePath, { encoding: 'utf8' });
  const rl = createInterface({
    input: fileStream,
    crlfDelay: Infinity,
  });

  let lineCount = 0;
  for await (const line of rl) {
    // Skip empty lines
    if (!line.trim()) continue;

    // Parse CSV line (simple comma split)
    const [id, name, email] = line.split(',');

    // Process the record
    console.log(`Record ${lineCount}: ${name} (${email})`);
    lineCount++;

    // Memory stays flat — each line is discarded after processing
  }

  return lineCount;
}

// A 10 GB file uses < 50 MB of memory
processCSV('./million-rows.csv')
  .then((count) => console.log(`Processed ${count} lines.`))
  .catch((err) => console.error('Error:', err.message));
```

**Expected Output:**
```
Record 0: Alice (alice@example.com)
Record 1: Bob (bob@example.com)
...
Processed 1000000 lines.
```

**Why this output:** `createReadStream` reads the file in chunks (default 64 KB). `createInterface` buffers chunks and emits complete lines. The `for await...of` loop processes each line, and memory usage remains constant because each line is discarded after processing. This pattern handles files far larger than available RAM.

#### Real-World Cases

- **Database imports:** Streaming a 50 GB CSV into PostgreSQL via `COPY` or batch inserts.
- **Log analysis:** Counting error lines in a multi-gigabyte log file.
- **Data migration:** Transforming records from one format to another line by line.

---

### Sub-Feature 1.2: Chunked File Uploads and Direct-to-Cloud Streaming (S3, Cloudinary)

#### Definitions

**Core Definition:** Chunked uploads stream file data directly from an incoming HTTP request to cloud storage (S3, Cloudinary) without buffering the entire file in memory, using the cloud provider's multipart upload API.

**Technical Definition:** The `@aws-sdk/lib-storage` `Upload` class provides a managed multipart upload that reads from a Node.js Readable stream, automatically splitting data into parts (default 5 MB), uploading them concurrently, and completing the multipart upload. Combined with `busboy` for parsing `multipart/form-data`, this enables streaming uploads where the file never touches local disk or the Node.js heap in full. The `PassThrough` stream is commonly used to connect the multipart parser to the S3 upload manager.

**Beginner-Friendly Explanation:** Instead of saving an uploaded file to your server's disk first and then uploading it to S3 (which uses double the time and disk space), you stream the file directly from the user's browser to S3 — the data flows through your server like water through a pipe, never stopping to pool in one place.

#### Purposes

- To upload large files to cloud storage without buffering them in memory.
- To reduce disk I/O and latency by streaming directly to the destination.
- To handle concurrent uploads efficiently with bounded memory.
- To support resumable and multipart uploads for very large files.

#### Syntax Rules and Structure

```js
const { S3Client } = require('@aws-sdk/client-s3');
const { Upload } = require('@aws-sdk/lib-storage');
const { PassThrough } = require('stream');

const s3 = new S3Client({ region: 'us-east-1' });
const pass = new PassThrough();

const upload = new Upload({
  client: s3,
  params: {
    Bucket: 'my-bucket',
    Key: 'uploads/file.zip',
    Body: pass,
  },
});

upload.done();
// Pipe incoming data to pass
req.pipe(pass);
```

| Component | Breakdown |
|-----------|-----------|
| `S3Client` | AWS SDK S3 client. |
| `Upload` | Managed multipart upload class. |
| `PassThrough` | A Duplex stream that passes data through. |
| `Body: pass` | The Readable stream to upload from. |
| `upload.done()` | Promise that resolves when upload completes. |

**Constraints and Limitations:**
- S3 multipart uploads require each part (except the last) to be at least 5 MB.
- The `Upload` class buffers parts before sending; configure `partSize` for your workload.
- Error handling must destroy the stream on failure to prevent leaks.
- Cloudinary has a similar streaming upload API (`uploader.upload_stream`).

#### Annotated Code Example

```js
// s3-stream-upload.js
const express = require('express');
const busboy = require('busboy');
const { S3Client } = require('@aws-sdk/client-s3');
const { Upload } = require('@aws-sdk/lib-storage');
const { PassThrough } = require('stream');
const crypto = require('crypto');

const app = express();
const s3 = new S3Client({ region: process.env.AWS_REGION });

app.post('/upload', (req, res) => {
  const bb = busboy({ headers: req.headers });

  bb.on('file', (name, file, info) => {
    const { filename, mimeType } = info;
    const key = `uploads/${Date.now()}-${crypto.randomBytes(8).toString('hex')}-${filename}`;

    // Create a PassThrough to connect busboy to S3
    const pass = new PassThrough();

    const upload = new Upload({
      client: s3,
      params: {
        Bucket: process.env.S3_BUCKET,
        Key: key,
        Body: pass,
        ContentType: mimeType,
      },
    });

    // Pipe the file stream through PassThrough to S3
    file.pipe(pass);

    upload.done()
      .then(() => {
        console.log(`Uploaded ${key} to S3.`);
        res.json({ key, filename });
      })
      .catch((err) => {
        console.error('S3 upload failed:', err);
        res.status(500).json({ error: 'Upload failed' });
      });

    // Handle file size limit
    file.on('limit', () => {
      pass.destroy(new Error('File too large'));
      res.status(413).json({ error: 'File too large' });
    });
  });

  bb.on('close', () => {
    if (!res.headersSent) {
      res.status(400).json({ error: 'No file uploaded' });
    }
  });

  req.pipe(bb);
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Expected Output:**
```
Uploaded uploads/1712345678-abcd1234-video.mp4 to S3.
```

**Why this output:** `busboy` parses the multipart form data and emits a `'file'` event with a Readable stream. The `PassThrough` connects this stream to the S3 `Upload` manager, which reads chunks and uploads them as multipart parts. The file never accumulates in memory; it flows from the HTTP request to S3 in chunks. The `upload.done()` Promise resolves when the multipart upload is complete.

#### Real-World Cases

- **User avatar uploads:** Streaming profile pictures directly to S3 or Cloudinary.
- **Video platforms:** Uploading multi-gigabyte videos without server-side buffering.
- **Document management:** Streaming PDFs and Office documents to cloud storage.
- **Backup systems:** Streaming database dumps directly to S3.

---

### Sub-Feature 1.3: Multipart Form-Data Parsing

#### Definitions

**Core Definition:** Multipart form-data parsing reads `multipart/form-data` request bodies — used for file uploads — without buffering the entire request in memory, using a streaming parser like `busboy`.

**Technical Definition:** `busboy` is a streaming parser for `multipart/form-data` request bodies. It emits `'file'` events with Readable streams for each file part and `'field'` events for non-file form fields. Unlike `multer` with memory storage (which buffers entire files in memory), `busboy` allows each file stream to be piped directly to a destination (disk, S3, etc.), keeping memory usage bounded. `busboy` is implemented by receiving a Node.js `ReadableStream` and emitting events via an `EventEmitter` interface.

**Beginner-Friendly Explanation:** When a browser uploads a file using a form, it sends the data in a special format called `multipart/form-data`. `busboy` reads this format piece by piece, telling you "here's a file field" or "here's a text field" as it encounters them. You can then pipe each file stream wherever you want — to disk, to S3, or through a transform — without ever holding the whole upload in memory.

#### Purposes

- To parse file uploads without buffering entire files in memory.
- To extract text fields and file fields from a multipart form.
- To stream uploaded files directly to their destination.
- To enforce file size limits and other constraints during upload.

#### Syntax Rules and Structure

```js
const busboy = require('busboy');

const bb = busboy({ headers: req.headers, limits: { fileSize: 100 * 1024 * 1024 } });

bb.on('file', (name, file, info) => {
  const { filename, mimeType } = info;
  file.pipe(destinationStream);
});

bb.on('field', (name, value) => {
  // Handle non-file fields
});

bb.on('close', () => { /* all parts parsed */ });

req.pipe(bb);
```

| Component | Breakdown |
|-----------|-----------|
| `busboy({ headers })` | Creates a parser for the request headers. |
| `limits.fileSize` | Maximum file size in bytes. |
| `'file'` event | Emitted for each file part with a Readable stream. |
| `'field'` event | Emitted for each non-file field. |
| `'close'` event | Emitted when parsing is complete. |

**Constraints and Limitations:**
- `busboy` does not save files to disk; the caller must consume or discard each file stream.
- File streams must be fully consumed or explicitly destroyed to prevent leaks.
- The `limits` option can also limit the number of files and fields.

#### Annotated Code Example

```js
// multipart-parser.js
const http = require('node:http');
const busboy = require('busboy');
const fs = require('node:fs');
const path = require('node:path');
const crypto = require('node:crypto');

const server = http.createServer((req, res) => {
  if (req.method !== 'POST') {
    res.statusCode = 405;
    return res.end('Method Not Allowed');
  }

  const bb = busboy({
    headers: req.headers,
    limits: {
      fileSize: 50 * 1024 * 1024, // 50 MB
      files: 5,
    },
  });

  const fields = {};
  const files = [];

  bb.on('field', (name, value) => {
    fields[name] = value;
    console.log(`Field: ${name} = ${value}`);
  });

  bb.on('file', (name, file, info) => {
    const { filename, mimeType } = info;
    const uniqueName = `${crypto.randomBytes(8).toString('hex')}${path.extname(filename)}`;
    const savePath = path.join('/tmp/uploads', uniqueName);

    // Stream directly to disk
    const writeStream = fs.createWriteStream(savePath);
    file.pipe(writeStream);

    let size = 0;
    file.on('data', (chunk) => { size += chunk.length; });

    file.on('limit', () => {
      writeStream.destroy();
      fs.unlink(savePath, () => {});
      console.error(`File ${filename} exceeded size limit.`);
    });

    writeStream.on('finish', () => {
      files.push({ field: name, original: filename, saved: uniqueName, size, mimeType });
      console.log(`Saved ${filename} (${size} bytes) as ${uniqueName}`);
    });
  });

  bb.on('close', () => {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ fields, files }));
  });

  bb.on('error', (err) => {
    console.error('Busboy error:', err);
    res.statusCode = 500;
    res.end('Upload failed');
  });

  req.pipe(bb);
});

server.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output:**
```
Field: username = alice
Saved photo.jpg (245760 bytes) as a1b2c3d4e5f6.jpg
```
```
{"fields":{"username":"alice"},"files":[{"field":"photo","original":"photo.jpg","saved":"a1b2c3d4e5f6.jpg","size":245760,"mimeType":"image/jpeg"}]}
```

**Why this output:** `busboy` parses the multipart body. The `'field'` event fires for text fields (`username`). The `'file'` event fires for the uploaded file, providing a Readable stream that is piped directly to a `WriteStream` on disk. The file never accumulates in memory; it streams from the request to disk in chunks. The `'close'` event signals that all parts have been parsed, and the response is sent.

#### Real-World Cases

- **File upload endpoints:** Handling avatar, document, and media uploads in Express/Koa applications.
- **Form processing:** Extracting text fields and files from HTML forms.
- **API gateways:** Parsing multipart requests before forwarding to backend services.

---

## Core Concept 2: Data Modification & Security

### Sub-Feature 2.1: Real-Time Compression/Decompression (`zlib.createGzip`, Brotli)

#### Definitions

**Core Definition:** Real-time compression uses `zlib` Transform streams to compress or decompress data as it flows through a pipeline, without buffering the entire dataset in memory.

**Technical Definition:** The `node:zlib` module provides compression functionality implemented using Gzip, Deflate/Inflate, Brotli, and Zstd. Compression and decompression are built around the Node.js Streams API. Compressing or decompressing a stream (such as a file) can be accomplished by piping the source stream through a `zlib` Transform stream into a destination stream. `zlib.createGzip()` creates a Gzip compression Transform stream; `zlib.createBrotliCompress()` creates a Brotli compression Transform stream with configurable quality (0–11).

**Beginner-Friendly Explanation:** Compression is like vacuum-sealing clothes to fit more in a suitcase. Instead of compressing the whole file at once (which requires the whole file in memory), you compress it piece by piece as it flows through the pipeline — like vacuum-sealing each item as it comes off the conveyor belt.

#### Purposes

- To reduce file sizes for storage or transmission.
- To compress HTTP responses on the fly (Content-Encoding: gzip/br).
- To decompress compressed data streams without buffering.
- To support multiple compression algorithms (Gzip, Brotli, Deflate, Zstd) in a single pipeline.

#### Syntax Rules and Structure

```js
const zlib = require('node:zlib');
const { pipeline } = require('node:stream/promises');

await pipeline(
  sourceStream,
  zlib.createGzip({ level: 9 }), // or createBrotliCompress({ params: { [zlib.constants.BROTLI_PARAM_QUALITY]: 11 } })
  destinationStream
);
```

| Component | Breakdown |
|-----------|-----------|
| `createGzip(options)` | Gzip compression Transform stream. |
| `createBrotliCompress(options)` | Brotli compression Transform stream. |
| `createGunzip()` | Gzip decompression. |
| `createBrotliDecompress()` | Brotli decompression. |
| `level` | Compression level (0–9 for Gzip, 0–11 for Brotli). |

**Constraints and Limitations:**
- Higher compression levels use more CPU and memory.
- Brotli is generally more efficient than Gzip for text but slower.
- `pipeline()` automatically handles backpressure between the source, compressor, and destination.

#### Annotated Code Example

```js
// compression-pipeline.js
const { createReadStream, createWriteStream } = require('node:fs');
const zlib = require('node:zlib');
const { pipeline } = require('node:stream/promises');

async function compressFile(input, output) {
  try {
    await pipeline(
      createReadStream(input),
      zlib.createGzip({ level: 6 }),
      createWriteStream(output)
    );
    console.log(`Compressed ${input} -> ${output}`);
  } catch (err) {
    console.error('Compression failed:', err.message);
  }
}

async function brotliCompress(input, output) {
  try {
    await pipeline(
      createReadStream(input),
      zlib.createBrotliCompress({
        params: {
          [zlib.constants.BROTLI_PARAM_QUALITY]: 11,
        },
      }),
      createWriteStream(output)
    );
    console.log(`Brotli compressed ${input} -> ${output}`);
  } catch (err) {
    console.error('Brotli failed:', err.message);
  }
}

compressFile('large.log', 'large.log.gz');
brotliCompress('large.log', 'large.log.br');
```

**Expected Output:**
```
Compressed large.log -> large.log.gz
Brotli compressed large.log -> large.log.br
```

**Why this output:** `pipeline()` connects the file read stream to the Gzip Transform stream and then to the file write stream. Data flows through the compressor in chunks, and `pipeline()` manages backpressure — if the compressor is slower than the disk read, the read stream is paused. The result is a compressed file written incrementally without buffering the entire input in memory.

#### Real-World Cases

- **HTTP compression:** Compressing API responses with `Content-Encoding: gzip` or `br`.
- **Log archival:** Compressing old log files to save disk space.
- **Data transfer:** Compressing data before sending over a network.
- **Static asset serving:** Pre-compressing assets and serving the compressed version.

---

### Sub-Feature 2.2: Streaming Encryption and Hashing (`crypto.createCipheriv`, `crypto.createHash`)

#### Definitions

**Core Definition:** Streaming encryption uses `crypto.createCipheriv()` Transform streams to encrypt data as it flows through a pipeline, while streaming hashing uses `crypto.createHash()` to compute a hash digest incrementally.

**Technical Definition:** The `node:crypto` module provides cryptographic functionality. `crypto.createCipheriv(algorithm, key, iv)` creates and returns a `Cipher` object that can be used as a Transform stream — data written to it is encrypted, and encrypted data is readable from it. `crypto.createHash(algorithm)` creates and returns a `Hash` object that can be used to generate hash digests; it supports incremental updates via `hash.update(data)` and finalisation via `hash.digest()`. For streaming, `hash` can be used as a Transform stream, emitting the digest at the end. `crypto.createCipher` is deprecated; use `crypto.createCipheriv` instead.

**Beginner-Friendly Explanation:** Encryption is like putting a letter in a locked box. Streaming encryption means you lock each page of the letter as it's written, rather than waiting for the whole letter and then locking it. Hashing is like creating a unique fingerprint for data — streaming hashing lets you compute the fingerprint piece by piece as data arrives.

#### Purposes

- To encrypt files or data streams without buffering the entire content.
- To decrypt encrypted streams on the fly.
- To compute hash digests (SHA-256, MD5, etc.) of large files incrementally.
- To verify data integrity during transmission.

#### Syntax Rules and Structure

```js
const crypto = require('node:crypto');

// Streaming encryption
const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
await pipeline(
  createReadStream('plain.txt'),
  cipher,
  createWriteStream('encrypted.bin')
);

// Streaming hashing
const hash = crypto.createHash('sha256');
await pipeline(
  createReadStream('large-file.bin'),
  hash,
  // hash emits the digest at the end
);
```

| Component | Breakdown |
|-----------|-----------|
| `createCipheriv(algorithm, key, iv)` | Creates a Cipher Transform stream. |
| `algorithm` | e.g., `'aes-256-gcm'`, `'aes-192-cbc'`. |
| `key` | Encryption key (Buffer). |
| `iv` | Initialisation vector (Buffer). |
| `createHash(algorithm)` | Creates a Hash Transform stream. |
| `hash.digest([encoding])` | Returns the final hash digest. |

**Constraints and Limitations:**
- `createCipher` is deprecated; always use `createCipheriv`.
- The initialisation vector (IV) must be unique per encryption operation for security.
- GCM mode provides authentication; CBC mode does not.
- Hash algorithms like MD5 and SHA-1 are considered insecure for cryptographic purposes.

#### Annotated Code Example

```js
// encryption-pipeline.js
const crypto = require('node:crypto');
const { createReadStream, createWriteStream } = require('node:fs');
const { pipeline } = require('node:stream/promises');

async function encryptFile(input, output, password) {
  const algorithm = 'aes-256-gcm';
  const key = crypto.scryptSync(password, 'salt', 32);
  const iv = crypto.randomBytes(16);

  const cipher = crypto.createCipheriv(algorithm, key, iv);

  try {
    await pipeline(
      createReadStream(input),
      cipher,
      createWriteStream(output)
    );

    // Get the authentication tag (for GCM)
    const authTag = cipher.getAuthTag();
    console.log('Encryption complete.');
    console.log('IV:', iv.toString('hex'));
    console.log('Auth tag:', authTag.toString('hex'));
  } catch (err) {
    console.error('Encryption failed:', err.message);
  }
}

async function hashFile(input) {
  const hash = crypto.createHash('sha256');

  await pipeline(
    createReadStream(input),
    hash
  );

  const digest = hash.digest('hex');
  console.log('SHA-256:', digest);
  return digest;
}

encryptFile('secret.txt', 'secret.enc', 'my-password');
hashFile('secret.txt');
```

**Expected Output:**
```
Encryption complete.
IV: a1b2c3d4e5f6...
Auth tag: 1a2b3c4d5e6f...
SHA-256: 9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08
```

**Why this output:** The `cipher` Transform stream encrypts data as it flows from the read stream to the write stream. The IV and authentication tag are required for decryption (GCM mode). The `hash` Transform stream computes the SHA-256 digest incrementally; `hash.digest('hex')` returns the final hash after all data has been processed.

#### Real-World Cases

- **File encryption:** Encrypting sensitive documents before storing in cloud storage.
- **Data integrity:** Computing SHA-256 checksums for downloaded files.
- **Password hashing:** Using `crypto.scrypt` for password storage (not streaming, but related).
- **Secure backups:** Encrypting database dumps before transferring to remote storage.

---

## Core Concept 3: Log Architecture & Observability

### Sub-Feature 3.1: Real-Time Log Scraping and Transformation Pipelines

#### Definitions

**Core Definition:** Real-time log scraping uses streams to read log data as it is produced (from files, sockets, or stdout) and transform it on the fly for filtering, aggregation, or forwarding.

**Technical Definition:** A log processing pipeline typically consists of a Readable source (a log file being tailed, a child process stdout, or a network socket), one or more Transform streams for parsing and filtering, and a Writable destination (a file, database, or another socket). The `stream.pipeline()` utility orchestrates these stages with backpressure and error handling. The pattern is used in production for real-time log analysis, error detection, and ETL pipelines processing millions of rows.

**Beginner-Friendly Explanation:** Imagine a security camera monitoring a building. Instead of recording everything and reviewing it later, you want to watch the feed in real time and alert on suspicious activity. A log pipeline does the same for your application's logs: it reads them as they are written, filters for errors or patterns, and forwards the important ones immediately.

#### Purposes

- To process log files as they grow in real time.
- To filter, parse, and transform log lines into structured data.
- To forward logs to monitoring systems, databases, or alerting services.
- To handle high-volume log streams with bounded memory.

#### Syntax Rules and Structure

```js
const { createReadStream } = require('node:fs');
const { createInterface } = require('node:readline');
const { pipeline } = require('node:stream/promises');
const { Transform } = require('node:stream');

const filter = new Transform({
  objectMode: true,
  transform(line, encoding, callback) {
    if (line.includes('ERROR')) {
      this.push(line);
    }
    callback();
  },
});
```

| Component | Breakdown |
|-----------|-----------|
| `createReadStream(logPath)` | Reads the log file. |
| `createInterface({ input })` | Line-by-line parsing. |
| Custom `Transform` | Filters/parses lines. |
| `pipeline(...)` | Orchestrates the chain. |

**Constraints and Limitations:**
- Tailing a file requires watching for file changes (e.g., `fs.watch` or `tail -f` behaviour).
- Log rotation can break stream continuity; consider using a proper log shipper.
- Backpressure must be managed if the destination is slower than the log source.

#### Annotated Code Example

```js
// log-pipeline.js
const { createReadStream } = require('node:fs');
const { createInterface } = require('node:readline');
const { Transform, pipeline } = require('node:stream');
const { promisify } = require('node:util');

const pipe = promisify(pipeline);

// Transform: filter ERROR lines and parse JSON
const errorFilter = new Transform({
  objectMode: true,
  transform(line, encoding, callback) {
    try {
      const entry = JSON.parse(line);
      if (entry.level === 'ERROR') {
        this.push(entry);
      }
    } catch {
      // Skip non-JSON lines
    }
    callback();
  },
});

// Transform: format for output
const formatter = new Transform({
  objectMode: true,
  transform(entry, encoding, callback) {
    this.push(`[${entry.timestamp}] ${entry.message}\n`);
    callback();
  },
});

async function processLogs(logPath) {
  try {
    const fileStream = createReadStream(logPath);
    const rl = createInterface({ input: fileStream, crlfDelay: Infinity });

    // Convert readline interface to an object-mode Readable
    const lineStream = require('node:stream').Readable.from(rl);

    await pipe(
      lineStream,
      errorFilter,
      formatter,
      process.stdout
    );

    console.log('Log processing complete.');
  } catch (err) {
    console.error('Pipeline failed:', err);
  }
}

processLogs('app.log');
```

**Expected Output:**
```
[2026-01-15T10:30:00.000Z] Database connection failed
[2026-01-15T10:31:00.000Z] Timeout on request /api/users
Log processing complete.
```

**Why this output:** The `readline` interface reads the log file line by line. The `errorFilter` Transform parses each line as JSON and only pushes entries with `level: 'ERROR'`. The `formatter` Transform formats the filtered entries for output. `pipeline()` connects all stages with backpressure and automatic cleanup. Only error-level logs are printed to stdout.

#### Real-World Cases

- **Application monitoring:** Tailing `app.log` and forwarding errors to Slack or PagerDuty.
- **Security auditing:** Filtering auth logs for suspicious activity.
- **Metrics extraction:** Parsing logs to extract request latency and error rates.
- **ELK integration:** Streaming logs to Elasticsearch via a Logstash-compatible pipeline.

---

### Sub-Feature 3.2: Multiplexing and Demultiplexing Multiple Data Streams Over a Single Connection

#### Definitions

**Core Definition:** Multiplexing (mux) combines multiple logical data streams into a single physical connection, while demultiplexing (demux) separates them back into individual streams on the receiving end.

**Technical Definition:** `mux-demux` is a library that multiplexes and demultiplexes object streams across any text stream (TCP, WebSocket, etc.). It creates a single Duplex stream that carries multiple logical sub-streams, each identified by metadata. Both endpoints can create read and write streams; the other side emits a `'connection'` event with a stream that can be inspected via `stream.meta`. A critical gotcha is to create a `MuxDemux` instance per connection — do not connect many connections to one instance. Modern alternatives include `bpmux` (with back-pressure on each stream) and `stream-demux`.

**Beginner-Friendly Explanation:** Imagine a highway with multiple lanes. Instead of building a separate road for each lane, you use one road with lane markings. Multiplexing is like putting cars from different lanes onto a single highway, and demultiplexing is like having each car exit at the right destination. This is useful when you have one network connection but need to send multiple independent data streams over it.

#### Purposes

- To send multiple independent data streams over a single connection.
- To reduce the overhead of opening multiple TCP connections or WebSocket channels.
- To enable bidirectional, multiplexed communication (both sides can initiate streams).
- To simplify protocol design by separating concerns into logical channels.

#### Syntax Rules and Structure

```js
const MuxDemux = require('mux-demux');

// Server side
const mx = MuxDemux();
socket.pipe(mx).pipe(socket);

mx.on('connection', (stream) => {
  // stream.meta identifies the logical channel
  stream.pipe(process.stdout);
});

// Client side
const mx2 = MuxDemux();
socket.pipe(mx2).pipe(socket);

const ds = mx2.createWriteStream({ channel: 'logs' });
ds.write('log line');
```

| Component | Breakdown |
|-----------|-----------|
| `MuxDemux()` | Creates a multiplexer/demultiplexer Duplex stream. |
| `mx.createWriteStream(meta)` | Creates a logical WriteStream with metadata. |
| `mx.createReadStream(meta)` | Creates a logical ReadStream. |
| `'connection'` event | Emitted on the other side when a stream is created. |
| `stream.meta` | The metadata identifying the logical channel. |

**Constraints and Limitations:**
- One `MuxDemux` instance per connection; do not share across connections.
- Error handling is critical: invalid data can cause parsing errors.
- Binary support requires the `mux-demux/msgpack` variant.
- Backpressure is handled per-stream in `bpmux` but not in the original `mux-demux`.

#### Annotated Code Example

```js
// mux-demux-example.js
const net = require('node:net');
const MuxDemux = require('mux-demux');

// Server
const server = net.createServer((socket) => {
  const mx = MuxDemux();

  mx.on('connection', (stream) => {
    const { type } = stream.meta;
    console.log(`New stream: ${type}`);

    if (type === 'logs') {
      stream.on('data', (data) => {
        console.log(`[LOG] ${data.toString().trim()}`);
      });
    } else if (type === 'metrics') {
      stream.on('data', (data) => {
        console.log(`[METRIC] ${data.toString().trim()}`);
      });
    }
  });

  // Handle errors
  mx.on('error', () => socket.destroy());
  socket.on('error', () => mx.destroy());

  socket.pipe(mx).pipe(socket);
});

server.listen(3000, () => {
  console.log('Mux server on port 3000');

  // Client
  const socket = net.connect(3000);
  const mx = MuxDemux();
  socket.pipe(mx).pipe(socket);

  const logStream = mx.createWriteStream({ type: 'logs' });
  const metricStream = mx.createWriteStream({ type: 'metrics' });

  logStream.write('Application started\n');
  metricStream.write('cpu=42%\n');
  logStream.write('Request received\n');
  metricStream.write('memory=1.2GB\n');

  setTimeout(() => {
    logStream.end();
    metricStream.end();
    socket.end();
  }, 100);
});
```

**Expected Output:**
```
Mux server on port 3000
New stream: logs
New stream: metrics
[LOG] Application started
[METRIC] cpu=42%
[LOG] Request received
[METRIC] memory=1.2GB
```

**Why this output:** The `MuxDemux` instance on each side multiplexes multiple logical streams over the single TCP connection. The server receives a `'connection'` event for each new logical stream, with `stream.meta` identifying whether it's a `logs` or `metrics` stream. Data written to each logical stream on the client is demultiplexed and delivered to the correct handler on the server.

#### Real-World Cases

- **RPC frameworks:** Sending multiple RPC calls and responses over a single connection.
- **Chat applications:** Multiple chat rooms over one WebSocket connection.
- **Remote procedure calls:** Multiplexing file transfer, terminal, and control channels over SSH-like connections.
- **Game networking:** Sending player position updates, chat messages, and game state over one UDP/TCP connection.

---

## References

- Node.js Documentation — Stream — https://nodejs.org/api/stream.html
- Node.js Documentation — `stream.pipeline()` — https://nodejs.org/api/stream.html#streampipelinestreams-callback
- Node.js Documentation — `stream/promises` API — https://nodejs.org/api/stream.html#streams-promises-api
- Node.js Documentation — Readline — https://nodejs.org/api/readline.html
- Node.js Documentation — Zlib — https://nodejs.org/api/zlib.html
- Node.js Documentation — `zlib.createGzip()` — https://nodejs.org/api/zlib.html#zlibcreategzipoptions
- Node.js Documentation — `zlib.createBrotliCompress()` — https://nodejs.org/api/zlib.html#zlibcreatebrotlicompressoptions
- Node.js Documentation — Crypto — https://nodejs.org/api/crypto.html
- Node.js Documentation — `crypto.createCipheriv()` — https://nodejs.org/api/crypto.html#cryptocreatecipherivalgorithm-key-iv-options
- Node.js Documentation — `crypto.createHash()` — https://nodejs.org/api/crypto.html#cryptocreatehashalgorithm-options
- Node.js Documentation — File System — https://nodejs.org/api/fs.html
- AWS SDK for JavaScript v3 — `@aws-sdk/lib-storage` Upload — https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/modules/_aws_sdk_lib_storage.html
- Busboy — Streaming multipart/form-data parser — https://github.com/mscdex/busboy
- MuxDemux — Multiplex-demultiplex object streams — https://github.com/dominictarr/mux-demux
- `bpmux` — Node stream multiplexing with back-pressure — https://www.npmjs.com/package/bpmux
- `stream-demux` — Consumable stream demultiplexer — https://www.npmjs.com/package/stream-demux
- Node.js Stream Processing for Large Datasets — Grizzly Peak Software — https://grizzlypeaksoftware.com/library/nodejs-stream-processing-for-large-datasets-efm75efe
- Node.js Streams: Processing Large Files Without Running Out of Memory — DEV Community — https://dev.to/whoffagents/nodejs-streams-processing-large-files-without-running-out-of-memory-51nd
- How to Handle File Uploads in Node.js at Scale — OneUptime — https://oneuptime.com/blog/post/2026-01-06-nodejs-file-uploads-scale/view
- Managing High Volume Data Streams with JavaScript — Pluralsight — https://www.pluralsight.com/labs/managing-high-volume-data-streams-with-javascript