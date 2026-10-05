# File Downloads & Serving — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** File downloads and serving is the practice of transmitting files stored on the server — or in object storage — to HTTP clients, either for inline display in the browser or as a forced download, with efficient handling of large files and support for partial content retrieval.

**Technical Definition:** Express provides three primary mechanisms for serving files: the `express.static` middleware for serving entire directories of static assets; `res.sendFile()` for streaming a specific file with automatic content-type detection, ETag generation, range request support, and conditional request handling (304 Not Modified); and `res.download()` — a convenience wrapper around `res.sendFile()` that additionally sets the `Content-Disposition: attachment` header to prompt a browser download. Both `res.sendFile()` and `res.download()` use the `send` module internally for efficient streaming, with the `acceptRanges` option (default `true`) enabling HTTP Range request support for video/audio seeking and resumable downloads.

**Beginner-Friendly Explanation:** When your server needs to give a file to a browser, you have three main options. `express.static` is like opening a folder and saying "anyone can read anything in here" — perfect for CSS, images, and JavaScript. `res.sendFile` is like handing a specific document to someone and saying "here, look at this in your browser." `res.download` is the same but with a sticky note saying "save this to your computer instead of opening it." For large files, you should stream them (pipe the file directly to the response) rather than loading the whole file into memory first — this prevents your server from running out of RAM. And for videos or large downloads that might get interrupted, HTTP Range requests let the browser ask for "just the part I need" instead of the whole file.

### Key Characteristics

- **Streaming by design:** `res.sendFile()` and `res.download()` stream files using the `send` module, never loading the entire file into memory.
- **Range request support:** Both methods enable `acceptRanges` by default, supporting video/audio seeking and resumable downloads.
- **Automatic content-type detection:** The file extension is used to set the `Content-Type` header automatically.
- **ETag and conditional requests:** Files are served with ETags, enabling 304 Not Modified responses for caching.
- **Security-first path validation:** `res.sendFile()` requires absolute paths or a `root` option to prevent directory traversal.
- **`Content-Disposition` controls behaviour:** `inline` displays in the browser; `attachment` forces a download.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic understanding of HTTP:** Headers, status codes, and the request–response cycle.
- **Familiarity with Node.js streams:** `fs.createReadStream` and piping.
- **Knowledge of MIME types** (e.g., `image/jpeg`, `application/pdf`).

### Related Programming Areas

- **File Uploads:** The inverse operation — receiving files from clients.
- **File Storage & Processing:** Where files are stored (local or object storage) and how they are transformed.
- **Secure File Handling:** Validation, sanitisation, and access control for uploaded files.
- **CDN integration:** Serving files through CloudFront, Cloud CDN, or Azure CDN.
- **Media streaming:** Video/audio delivery with range requests and adaptive bitrate streaming.

### Core Concepts

1. **Serving Static Files** — `express.static` built-in middleware.
2. **Triggering Downloads** — `res.download()` vs. `res.attachment()`.
3. **Content-Disposition Header** — Inline viewing vs. forcing attachments.
4. **Streaming Large Files** — `fs.createReadStream` piped to `res`.
5. **HTTP Range Requests** — `res.sendFile` support for video/audio seeking.

---

## Core Concept 1: Serving Static Files — `express.static`

### Definitions

**Core Definition:** `express.static` is a built-in Express middleware function that serves static files — images, CSS, JavaScript, and other assets — from a specified directory.

**Technical Definition:** `express.static(root, [options])` serves static files from the `root` directory. It is based on the `serve-static` module. When a request matches a file in the root directory, the middleware sends it with automatic MIME type detection (via the `mime-types` module), caching headers, and security features. When a file is not found, it calls `next()` instead of sending a 404, allowing for middleware stacking and fallbacks. The `root` argument specifies the root directory from which to serve static assets.

**Beginner-Friendly Explanation:** `express.static` is like telling your server "anything in this folder is public — just hand it out when someone asks for it by name." You point it at a folder (like `public/`), and Express automatically serves any file inside it with the right content type and caching headers. It's the simplest way to serve CSS, images, and JavaScript files.

### Purposes

- To serve static files such as images, CSS files, and JavaScript files.
- To automatically set MIME types based on file extensions.
- To provide caching headers (ETag, Last-Modified) for efficient client-side caching.
- To serve files from multiple directories in a defined order.
- To mount static assets at a virtual path prefix.

### Syntax Rules and Structure

```js
express.static(root, [options])
```

| Component | Breakdown |
|-----------|-----------|
| `root` | The root directory from which to serve static assets. |
| `options` | Optional configuration object (see table below). |
| Returns | Middleware function for use with `app.use()`. |

**Common Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `dotfiles` | String | `'ignore'` (Express 5) | How to handle dotfiles: `'allow'`, `'deny'`, `'ignore'`. |
| `etag` | Boolean | `true` | Enable/disable ETag generation. |
| `extensions` | Array | `false` | Fallback file extensions (e.g., `['html', 'htm']`). |
| `index` | String/Array/Boolean | `'index.html'` | Send this file for directory requests. |
| `lastModified` | Boolean | `true` | Enable/disable Last-Modified header. |
| `maxAge` | Number/String | `0` | Cache max-age in ms or parseable string. |
| `redirect` | Boolean | `true` | Redirect to trailing `/` for directories. |
| `setHeaders` | Function | — | Function to set custom headers. |

**Rules:**
- The `root` path is relative to the directory from where the Node process is launched. Use `path.join(__dirname, 'public')` for an absolute path. 
- Multiple `express.static` calls can be chained; Express looks up files in the order the middleware functions are mounted. 
- To create a virtual path prefix, specify a mount path: `app.use('/static', express.static('public'))`. 
- **Express 5 change:** The `dotfiles` option now defaults to `'ignore'`, which means dotfiles are treated as if they do not exist (404). This is a change from Express 4's less strict behaviour.
- **Security warning:** Never use `express.static('.')` or `express.static(__dirname)` — this exposes the entire application directory, including source code, configuration files, and credentials. Always point to a dedicated `public/` directory. 
- For best performance, use a reverse proxy (Nginx, CloudFront) to cache static assets. 

### Annotated Code Example

```js
// express-static.js
const express = require('express');
const path = require('path');
const app = express();

// Serve static files from the 'public' directory
// Files are accessible at the root URL (e.g., /images/logo.png)
app.use(express.static(path.join(__dirname, 'public'), {
  dotfiles: 'deny',           // Block access to .env, .git, etc.
  etag: true,                 // Enable ETag generation
  index: 'index.html',        // Serve index.html for directory requests
  maxAge: '1d',               // Cache for 1 day
  setHeaders: (res, filePath) => {
    // Add security headers to static files
    res.set('X-Content-Type-Options', 'nosniff');
  }
}));

// Serve additional static files with a virtual path prefix
// Files are accessible at /static/... (e.g., /static/css/style.css)
app.use('/static', express.static(path.join(__dirname, 'assets')));

app.get('/', (req, res) => {
  res.send('<h1>Home Page</h1>');
});

app.listen(3000, () => console.log('Static server on 3000'));
```

**Expected Output (for `GET /images/logo.png`):**
```
HTTP/1.1 200 OK
Content-Type: image/png
ETag: "abc123"
Cache-Control: public, max-age=86400
X-Content-Type-Options: nosniff

(binary PNG data)
```

**Expected Output (for `GET /.env`):**
```
HTTP/1.1 404 Not Found
```

**Expected Output (for `GET /static/css/style.css`):**
```
HTTP/1.1 200 OK
Content-Type: text/css

(CS‌S file content)
```

**Why this output:** The `express.static` middleware serves files from the `public/` directory. When `/images/logo.png` is requested, Express finds the file, sets the correct `Content-Type: image/png`, generates an ETag for caching, and sends the file. The `dotfiles: 'deny'` option blocks access to `.env` and other dotfiles, returning a 404. The second `express.static` call mounts the `assets/` directory at the `/static` virtual path prefix, so files are accessible at `/static/css/style.css` instead of `/css/style.css`.

### Real-World Cases

- **Web applications:** Serving CSS, JavaScript, and image assets from a `public/` directory.
- **Single-page applications (SPAs):** Serving the built `dist/` folder with `index.html` as the fallback.
- **Documentation sites:** Serving static HTML, CSS, and images from a `docs/` directory.
- **Multiple asset directories:** Serving vendor assets and application assets from separate directories.

---

## Core Concept 2: Triggering Downloads — `res.download()` vs. `res.attachment()`

### Definitions

**Core Definition:** `res.download()` transfers a file as an attachment, prompting the browser to download it, while `res.attachment()` only sets the `Content-Disposition: attachment` header without sending the file itself.

**Technical Definition:** `res.download(path, [filename], [options], [callback])` transfers the file at `path` as an "attachment," typically prompting the user for download. It is a convenience wrapper around `res.sendFile()` that automatically sets the `Content-Disposition: attachment` header and uses the `content-disposition` module to safely encode filenames (including non-ASCII characters per RFC 5987). `res.attachment([filename])` sets the `Content-Disposition` header to `"attachment"` and, if a filename is given, sets the `Content-Type` based on the extension via `res.type()` and sets the `filename=` parameter. `res.attachment()` does not send the file — it only sets headers, typically used with `res.sendFile()` or `res.send()`.

**Beginner-Friendly Explanation:** `res.download()` is like handing someone a file and saying "save this to your computer." It does everything: finds the file, sets the download headers, and streams the data. `res.attachment()` is just the sticky note that says "this is a download" — you still have to send the file yourself. In practice, `res.download()` is the more convenient option because it does both steps in one call.

### Purposes

- To prompt the browser to download a file rather than display it inline.
- To allow custom filenames in the download dialog via the `filename` parameter.
- To provide a completion callback for logging or cleanup after download.
- To separate the "set attachment header" step (`res.attachment()`) from the "send file" step (`res.sendFile()`).

### Syntax Rules and Structure

**`res.download()`:**
```js
res.download(path, [filename], [options], [callback])
```

| Parameter | Description |
|-----------|-------------|
| `path` | Path to the file on the server. |
| `filename` | Optional. Name to display in the browser's download dialog. |
| `options` | Optional. Passed to the underlying `res.sendFile()` call. |
| `callback` | Optional. Invoked on completion or error. |

**`res.attachment()`:**
```js
res.attachment([filename])
```

| Parameter | Description |
|-----------|-------------|
| `filename` | Optional. Sets the `filename=` parameter and `Content-Type`. |

**Rules:**
- `res.download()` automatically sets the `Content-Disposition: attachment` header. 
- `res.attachment()` only sets the header; it does not send the file. 
- `res.download()` uses `res.sendFile()` internally and supports the same options (e.g., `root`, `dotfiles`, `headers`). 
- `res.download()` has a callback parameter for error handling; `res.attachment()` returns the `res` object for chaining. 
- The `filename` parameter in `res.download()` overrides the default filename derived from `path`. 
- **Express 5:** `res.download()` uses the `content-disposition` module for RFC 5987-compliant filename encoding (handles non-ASCII characters safely). 

### Annotated Code Example

```js
// res-download-vs-attachment.js
const express = require('express');
const path = require('path');
const app = express();

// --- res.download(): sets header AND sends the file ---
app.get('/download/report', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'report.pdf');
  res.download(filePath, 'monthly-report.pdf', (err) => {
    if (err) {
      console.error('Download failed:', err.message);
    } else {
      console.log('Download completed');
    }
  });
});

// --- res.attachment() + res.sendFile(): manual two-step ---
app.get('/download/manual', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'data.csv');
  res.attachment('export.csv');        // Set Content-Disposition
  res.type('text/csv');                 // Set Content-Type
  res.sendFile(filePath);               // Send the file
});

// --- res.attachment() + res.send(): buffer-based ---
app.get('/download/buffer', (req, res) => {
  const csvContent = 'name,age\nAlice,30\nBob,25';
  res.attachment('users.csv');
  res.send(csvContent);                 // Send from memory
});

app.listen(3000, () => console.log('Download server on 3000'));
```

**Expected Output (for `GET /download/report`):**
```
HTTP/1.1 200 OK
Content-Disposition: attachment; filename="monthly-report.pdf"
Content-Type: application/pdf

(binary PDF data)
```

**Expected Output (for `GET /download/manual`):**
```
HTTP/1.1 200 OK
Content-Disposition: attachment; filename="export.csv"
Content-Type: text/csv

(name,age\nAlice,30\nBob,25)
```

**Why this output:** `res.download()` sets the `Content-Disposition: attachment` header with the custom filename and streams the PDF file. The browser sees the attachment header and prompts the user to save the file as `monthly-report.pdf`. The manual approach using `res.attachment()` + `res.sendFile()` produces the same result but requires two separate calls. The buffer-based approach demonstrates using `res.attachment()` with `res.send()` when the data is generated in memory rather than stored on disk.

### Real-World Cases

- **Report generation:** Downloading a generated PDF report with a user-friendly filename.
- **Data export:** Exporting user data as a CSV file with a descriptive filename.
- **Invoice downloads:** Downloading an invoice as a PDF from a private bucket.
- **Backup files:** Downloading database backups or archive files.

---

## Core Concept 3: Content-Disposition Header

### Definitions

**Core Definition:** The `Content-Disposition` response header tells the browser whether to display the content inline in the browser window or treat it as a downloadable attachment.

**Technical Definition:** RFC 6266 defines the `Content-Disposition` header field for HTTP, taking over the definition from RFC 2183 (MIME). The header has two primary disposition types: **`inline`** (display automatically, default) and **`attachment`** (user-controlled display — most browsers show a "Save As" dialog). The header supports parameters including `filename` (the name to use when saving) and `filename*` (RFC 5987-encoded, for non-ASCII characters). The Express `content-disposition` module generates both `filename` and `filename*` parameters for maximum compatibility.

**Beginner-Friendly Explanation:** The `Content-Disposition` header is like a label on a package that tells the postal service (the browser) what to do with it. `inline` means "open it and show the contents." `attachment` means "don't open it — save it to a file." The `filename` parameter tells the browser what to name the file when saving.

### Purposes

- To control whether the browser displays a file inline or downloads it.
- To specify the filename that appears in the browser's download dialog.
- To safely encode non-ASCII filenames using RFC 5987.
- To enable content negotiation between inline viewing and forced download.

### Syntax Rules and Structure

```
Content-Disposition: inline
Content-Disposition: attachment
Content-Disposition: attachment; filename="report.pdf"
Content-Disposition: attachment; filename="report.pdf"; filename*=UTF-8''report%20with%20spaces.pdf
```

| Disposition Type | Behaviour | Use Case |
|-----------------|-----------|----------|
| `inline` | Display in browser (default) | PDFs, images, HTML pages |
| `attachment` | Prompt download | Reports, exports, installers |
| `form-data` | Multipart form part | Form submissions |

| Parameter | Purpose |
|-----------|---------|
| `filename` | Filename for the "Save As" dialog. |
| `filename*` | RFC 5987-encoded filename (UTF-8, percent-encoded). |

**Rules:**
- The `inline` disposition tells the user agent to display the response directly. 
- The `attachment` disposition tells the user agent to save the response to disk. 
- The `filename` parameter is used by most browsers to pre-populate the "Save As" dialog. 
- The `filename*` parameter uses RFC 5987 encoding for non-ASCII characters (e.g., Chinese, Arabic, emoji). 
- Express's `res.download()` uses the `content-disposition` module, which generates both `filename` and `filename*` for maximum compatibility. 
- When using `res.sendFile()` without `res.attachment()`, the `Content-Disposition` header is not set, so the browser defaults to `inline` behaviour. 

### Annotated Code Example

```js
// content-disposition.js
const express = require('express');
const path = require('path');
const app = express();

// Inline display (PDF opens in browser)
app.get('/view/report', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'report.pdf');
  // res.sendFile() does NOT set Content-Disposition — defaults to inline
  res.sendFile(filePath);
});

// Attachment (force download) with ASCII filename
app.get('/download/ascii', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'report.pdf');
  res.download(filePath, 'annual-report.pdf');
});

// Attachment with non-ASCII filename (RFC 5987 encoding)
app.get('/download/unicode', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'report.pdf');
  // Express's content-disposition module generates both filename and filename*
  res.download(filePath, '年度報告.pdf');
});

// Manual Content-Disposition with res.attachment()
app.get('/download/manual-header', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'data.csv');
  res.attachment('data-export.csv');
  res.sendFile(filePath);
});

app.listen(3000, () => console.log('Content-Disposition on 3000'));
```

**Expected Output (for `GET /view/report`):**
```
HTTP/1.1 200 OK
Content-Type: application/pdf
(no Content-Disposition header — browser displays inline)
```

**Expected Output (for `GET /download/ascii`):**
```
HTTP/1.1 200 OK
Content-Disposition: attachment; filename="annual-report.pdf"
Content-Type: application/pdf
```

**Expected Output (for `GET /download/unicode`):**
```
HTTP/1.1 200 OK
Content-Disposition: attachment; filename="???.pdf"; filename*=UTF-8''%E5%B9%B4%E5%BA%A6%E5%A0%B1%E5%91%8A.pdf
Content-Type: application/pdf
```

**Why this output:** `res.sendFile()` does not set `Content-Disposition`, so the browser defaults to `inline` and displays the PDF in the browser's PDF viewer. `res.download()` sets `Content-Disposition: attachment`, prompting a download. For non-ASCII filenames, the `content-disposition` module generates both `filename` (with fallback characters) and `filename*` (RFC 5987-encoded) so that modern browsers display the correct Unicode filename.

### Real-World Cases

- **PDF viewers:** Inline display for reports that users should read in the browser.
- **Invoice downloads:** Attachment disposition with the invoice number as the filename.
- **International applications:** RFC 5987 encoding for filenames with non-ASCII characters (Chinese, Arabic, accented characters).
- **CSV exports:** Attachment disposition with a descriptive filename like `users-2026-01-15.csv`.

---

## Core Concept 4: Streaming Large Files

### Definitions

**Core Definition:** Streaming large files means reading the file from disk in chunks and piping each chunk to the HTTP response, rather than loading the entire file into memory before sending it.

**Technical Definition:** Node.js's `fs.createReadStream(path)` creates a readable stream that emits data in chunks (default 64 KiB). When piped to the Express response object (`readStream.pipe(res)`), the `pipe()` method automatically handles backpressure — if the client's network is slow, the write stream signals the read stream to pause, preventing memory exhaustion. Express's `res.sendFile()` and `res.download()` use the `send` module internally, which implements the same streaming approach. For custom streaming (e.g., streaming from a database or generating content on the fly), manual `pipe()` is required.

**Beginner-Friendly Explanation:** Imagine trying to drink from a fire hose. If you try to catch all the water at once, you'll drown. Streaming is like using a cup — you take a sip, swallow, take another sip. In Node.js, `createReadStream` reads the file in small chunks, and `pipe(res)` sends each chunk to the client before reading the next one. This means a 10GB file can be served by a server with only 100MB of RAM.

### Purposes

- To prevent Node.js memory overflow when serving large files.
- To start sending data to the client before the entire file is read.
- To handle backpressure automatically through the pipe mechanism.
- To enable custom streaming logic (e.g., from a database, encryption, compression).

### Syntax Rules and Structure

```js
const fs = require('fs');

app.get('/download/:filename', (req, res) => {
  const filePath = path.join(UPLOAD_DIR, path.basename(req.params.filename));

  // Set headers
  const stat = fs.statSync(filePath);
  res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
  res.setHeader('Content-Type', 'application/octet-stream');
  res.setHeader('Content-Length', stat.size);

  // Create read stream and pipe to response
  const readStream = fs.createReadStream(filePath);
  readStream.on('error', (err) => {
    if (!res.headersSent) {
      res.status(500).json({ error: 'Error streaming file' });
    }
  });
  readStream.pipe(res);
});
```

| Component | Breakdown |
|-----------|-----------|
| `fs.createReadStream(path)` | Creates a readable stream from the file. |
| `.pipe(res)` | Pipes chunks to the response, handling backpressure. |
| `readStream.on('error')` | Handles errors (file not found, permission denied). |

**Rules:**
- Always set `Content-Length` when known — it allows the client to show a progress bar. 
- Handle stream errors before headers are sent; once headers are sent, you cannot change the status code. 
- Use `path.basename()` to sanitise filenames and prevent path traversal. 
- The `pipe()` method automatically handles backpressure — you do not need to manage it manually. 
- For truly massive files, consider using `stream.pipeline()` for better error handling. 
- **Backpressure warning:** If you use `res.write()` manually instead of `pipe()`, you must respect the return value — `res.write()` returns `false` when the buffer is full, and you should wait for the `'drain'` event before writing more. 

### Annotated Code Example

```js
// streaming-large-files.js
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

const UPLOAD_DIR = path.join(__dirname, 'files');

app.get('/download/:filename', (req, res) => {
  // Sanitise the filename to prevent path traversal
  const filename = path.basename(req.params.filename);
  const filePath = path.join(UPLOAD_DIR, filename);

  // Check if file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }

  // Get file stats
  const stat = fs.statSync(filePath);

  // Set headers
  res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
  res.setHeader('Content-Type', 'application/octet-stream');
  res.setHeader('Content-Length', stat.size);

  // Create read stream
  const readStream = fs.createReadStream(filePath);

  // Handle stream errors
  readStream.on('error', (err) => {
    console.error('Stream error:', err.message);
    if (!res.headersSent) {
      res.status(500).json({ error: 'Error streaming file' });
    } else {
      res.end();
    }
  });

  // Pipe the stream to the response
  // pipe() handles backpressure automatically
  readStream.pipe(res);
});

// Stream with a progress-tracking Transform (optional)
app.get('/download/with-progress/:filename', (req, res) => {
  const filename = path.basename(req.params.filename);
  const filePath = path.join(UPLOAD_DIR, filename);
  const stat = fs.statSync(filePath);

  res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
  res.setHeader('Content-Length', stat.size);

  let bytesSent = 0;
  const readStream = fs.createReadStream(filePath);

  readStream.on('data', (chunk) => {
    bytesSent += chunk.length;
    console.log(`Progress: ${Math.round((bytesSent / stat.size) * 100)}%`);
  });

  readStream.pipe(res);
});

app.listen(3000, () => console.log('Streaming server on 3000'));
```

**Expected Output (for `GET /download/large-video.mp4`):**
```
HTTP/1.1 200 OK
Content-Disposition: attachment; filename="large-video.mp4"
Content-Type: application/octet-stream
Content-Length: 1073741824

(binary data streamed in 64KB chunks)
```

**Expected Output (console during streaming):**
```
Progress: 1%
Progress: 2%
Progress: 3%
...
Progress: 100%
```

**Why this output:** The file is read in 64KB chunks and piped to the response. The `Content-Length` header tells the client the total size, allowing progress tracking. The `pipe()` method handles backpressure — if the client's network is slow, the read stream is paused until the write buffer drains. The file never fully resides in memory; only one chunk at a time is buffered.

### Real-World Cases

- **Video streaming:** Serving large video files without loading them into memory.
- **Large file downloads:** Downloading multi-gigabyte files (datasets, backups, archives).
- **Database exports:** Streaming query results directly to the response as CSV.
- **Log file downloads:** Streaming large log files for analysis.

---

## Core Concept 5: HTTP Range Requests

### Definitions

**Core Definition:** HTTP Range requests allow a client to request only a portion of a resource, enabling video/audio seeking, resumable downloads, and efficient partial content retrieval.

**Technical Definition:** A client sends a `Range` header (e.g., `Range: bytes=0-1023`) to request a specific byte range. The server responds with status `206 Partial Content` and a `Content-Range` header indicating which portion of the resource is being returned. Express's `res.sendFile()` and `res.download()` support range requests automatically via the `acceptRanges` option (default `true`) in the underlying `send` module. The `send` module also handles conditional requests (`If-Range`, `If-Modified-Since`, `If-None-Match`) and returns `304 Not Modified` when appropriate. For custom streaming, range handling must be implemented manually by parsing the `Range` header and using `createReadStream` with `start` and `end` options.

**Beginner-Friendly Explanation:** When you watch a video online and skip ahead, the browser doesn't download the entire video again — it asks the server for "just the part from 5:30 onwards." That's a Range request. The server responds with only that portion of the file (status 206 Partial Content) instead of the whole thing. This is what makes video seeking, resumable downloads, and efficient media streaming possible.

### Purposes

- To enable video/audio seeking in browsers (skipping to a specific timestamp).
- To support resumable downloads (continue after interruption).
- To reduce bandwidth by downloading only the needed portion.
- To enable parallel downloading of file segments.

### Sub-Feature 5.1: Automatic Range Support via `res.sendFile()`

#### Syntax Rules and Structure

```js
app.get('/video', (req, res) => {
  res.sendFile(videoPath, {
    acceptRanges: true,    // Enable range support (default: true)
    cacheControl: true,    // Enable Cache-Control header (default: true)
    headers: {
      'Content-Type': 'video/mp4'
    }
  });
});
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `acceptRanges` | Boolean | `true` | Enable/disable Accept-Ranges support. |
| `cacheControl` | Boolean | `true` | Enable/disable Cache-Control header. |
| `lastModified` | Boolean | `true` | Enable/disable Last-Modified header. |
| `start` | Number | — | Start byte position for partial content. |
| `end` | Number | — | End byte position for partial content. |

**Rules:**
- `res.sendFile()` and `res.download()` support range requests automatically when `acceptRanges` is `true` (the default). 
- The `send` module handles the `Range` header, sets `Content-Range` and `Content-Length`, and returns `206 Partial Content`. 
- Conditional requests (`If-Range`, `If-Modified-Since`, `If-None-Match`) are also handled automatically. 
- If the range is invalid, the server returns `416 Range Not Satisfiable`. 
- To disable range support, set `acceptRanges: false`. 

#### Annotated Code Example

```js
// range-requests-sendfile.js
const express = require('express');
const path = require('path');
const app = express();

app.get('/video/:filename', (req, res) => {
  const filename = path.basename(req.params.filename);
  const videoPath = path.join(__dirname, 'videos', filename);

  // res.sendFile() handles Range requests automatically
  res.sendFile(videoPath, {
    acceptRanges: true,       // Default: true
    cacheControl: true,       // Default: true
    headers: {
      'Content-Type': 'video/mp4'
    }
  }, (err) => {
    if (err) {
      console.error('SendFile error:', err.message);
      if (!res.headersSent) {
        res.status(500).json({ error: 'Error serving video' });
      }
    }
  });
});

app.listen(3000, () => console.log('Video server on 3000'));
```

**Expected Output (for `GET /video/lecture.mp4` without Range header):**
```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Type: video/mp4
Content-Length: 104857600

(full video data)
```

**Expected Output (for `GET /video/lecture.mp4` with `Range: bytes=0-1023`):**
```
HTTP/1.1 206 Partial Content
Accept-Ranges: bytes
Content-Range: bytes 0-1023/104857600
Content-Type: video/mp4
Content-Length: 1024

(first 1024 bytes of video)
```

**Why this output:** When the client sends a `Range` header, the `send` module (used internally by `res.sendFile()`) parses the range, reads only that portion of the file, and responds with `206 Partial Content`. The `Content-Range` header tells the client which bytes are being returned and the total file size. The `Accept-Ranges: bytes` header advertises that the server supports range requests. Without a `Range` header, the full file is sent with status 200.

### Sub-Feature 5.2: Manual Range Handling with `createReadStream`

#### Syntax Rules and Structure

```js
app.get('/download/:filename', (req, res) => {
  const filePath = path.join(UPLOAD_DIR, path.basename(req.params.filename));
  const stat = fs.statSync(filePath);
  const fileSize = stat.size;
  const range = req.headers.range;

  if (range) {
    // Parse Range header: "bytes=start-end"
    const parts = range.replace(/bytes=/, '').split('-');
    const start = parseInt(parts[0], 10);
    const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;
    const chunkSize = end - start + 1;

    res.writeHead(206, {
      'Content-Range': `bytes ${start}-${end}/${fileSize}`,
      'Accept-Ranges': 'bytes',
      'Content-Length': chunkSize,
      'Content-Type': 'application/octet-stream'
    });

    const stream = fs.createReadStream(filePath, { start, end });
    stream.pipe(res);
  } else {
    // No Range header — send the whole file
    res.writeHead(200, {
      'Content-Length': fileSize,
      'Content-Type': 'application/octet-stream'
    });
    fs.createReadStream(filePath).pipe(res);
  }
});
```

**Rules:**
- Always validate the range: `start` must be >= 0 and `end` must be < fileSize. 
- If the range is invalid, return `416 Range Not Satisfiable` with a `Content-Range: bytes */fileSize` header. 
- The `Content-Range` header format is `bytes start-end/totalSize`. 
- Use `createReadStream(path, { start, end })` to read only the requested byte range. 
- The `end` value is inclusive — `start=0, end=1023` reads 1024 bytes. 

#### Annotated Code Example

```js
// range-requests-manual.js
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

const VIDEO_DIR = path.join(__dirname, 'videos');

app.get('/stream/:filename', (req, res) => {
  const filename = path.basename(req.params.filename);
  const filePath = path.join(VIDEO_DIR, filename);

  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }

  const stat = fs.statSync(filePath);
  const fileSize = stat.size;
  const range = req.headers.range;

  if (range) {
    // Parse Range header
    const parts = range.replace(/bytes=/, '').split('-');
    const start = parseInt(parts[0], 10);
    const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;

    // Validate range
    if (start >= fileSize || end >= fileSize || start > end) {
      res.writeHead(416, {
        'Content-Range': `bytes */${fileSize}`
      });
      return res.end();
    }

    const chunkSize = end - start + 1;

    res.writeHead(206, {
      'Content-Range': `bytes ${start}-${end}/${fileSize}`,
      'Accept-Ranges': 'bytes',
      'Content-Length': chunkSize,
      'Content-Type': 'video/mp4'
    });

    const stream = fs.createReadStream(filePath, { start, end });
    stream.on('error', (err) => {
      console.error('Stream error:', err.message);
      if (!res.headersSent) {
        res.status(500).end();
      }
    });
    stream.pipe(res);
  } else {
    // No Range header — send full file
    res.writeHead(200, {
      'Content-Length': fileSize,
      'Accept-Ranges': 'bytes',
      'Content-Type': 'video/mp4'
    });
    fs.createReadStream(filePath).pipe(res);
  }
});

app.listen(3000, () => console.log('Manual range server on 3000'));
```

**Expected Output (for `GET /stream/lecture.mp4` with `Range: bytes=0-1023`):**
```
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1023/104857600
Accept-Ranges: bytes
Content-Length: 1024
Content-Type: video/mp4

(first 1024 bytes)
```

**Expected Output (for `GET /stream/lecture.mp4` with `Range: bytes=999999999-`):**
```
HTTP/1.1 416 Range Not Satisfiable
Content-Range: bytes */104857600
```

**Expected Output (for `GET /stream/lecture.mp4` without Range header):**
```
HTTP/1.1 200 OK
Content-Length: 104857600
Accept-Ranges: bytes
Content-Type: video/mp4

(full video data)
```

**Why this output:** The manual implementation parses the `Range` header, validates the requested range, and uses `createReadStream` with `start` and `end` options to read only the requested bytes. Invalid ranges return `416 Range Not Satisfiable`. Without a `Range` header, the full file is streamed with status 200.

### Real-World Cases

- **Video streaming platforms:** YouTube, Netflix, and Vimeo use range requests for video seeking.
- **Audio players:** Spotify and SoundCloud use range requests for audio playback.
- **Resumable downloads:** Download managers resume interrupted downloads using range requests.
- **PDF viewers:** Inline PDF viewing uses range requests to load pages on demand.

---

## References

- Serving static files in Express — https://expressjs.com/en/starter/static-files.html
- Express.js API — `express.static` — https://expressjs.com/en/5x/api.html#express.static
- Express.js API — `res.download` — https://expressjs.com/en/4x/api.html#res.download
- Express.js API — `res.sendFile` — https://expressjs.com/en/4x/api.html#res.sendFile
- Express.js API — `res.attachment` — https://expressjs.com/en/4x/api.html#res.attachment
- Express.js — res.download vs res.attachment (Stack Overflow) — https://stackoverflow.com/questions/40910198/express-js-what-is-the-difference-between-res-attachment-and-res-download
- RFC 6266 — Use of the Content-Disposition Header Field in HTTP — https://httpwg.org/specs/rfc6266.html
- RFC 2183 — Communicating Presentation Information in Internet Messages — https://www.rfc-editor.org/rfc/rfc2183
- RFC 5987 — Character Set and Language Encoding for HTTP Header Field Parameters — https://www.rfc-editor.org/rfc/rfc5987
- RFC 7233 — HTTP/1.1: Range Requests — https://www.rfc-editor.org/rfc/rfc7233
- How to stream file downloads in Node.js (CoreUI) — https://coreui.io/answers/how-to-stream-file-downloads-in-nodejs/
- How to use streams in Node.js (CoreUI) — https://coreui.io/answers/how-to-use-streams-in-nodejs/
- send-ranges — Express middleware for HTTP range requests — https://github.com/rexxars/send-ranges
- Express.js File Operations (DeepWiki) — https://deepwiki.com/expressjs/express/5.2-file-operations
- express.static `dotfiles` option change in Express 5 — https://github.com/expressjs/expressjs.com/issues/1987
- Path Traversal via res.sendFile() in Express (Sourcery) — https://www.sourcery.ai/vulnerabilities/path-traversal-via-res-sendfile-in-express
- Node.js Stream Backpressure Guide — https://nodejs.org/en/learn/modules/backpressuring-in-streams
- MDN — Content-Disposition — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition
- MDN — HTTP Range Requests — https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests