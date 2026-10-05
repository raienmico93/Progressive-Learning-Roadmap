# File Uploads — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** File upload is the process by which a client sends binary data (images, documents, videos) to a server over HTTP, typically using the `multipart/form-data` encoding, and the server parses, validates, and persists that data.

**Technical Definition:** File uploads in Express are handled via multipart parsers that implement RFC 7578 (`multipart/form-data`). The encoding splits the request body into multiple parts, each with its own `Content-Disposition` header (including `name` and optional `filename` parameters) and optional `Content-Type`. Multer is the standard middleware for this purpose, built on top of Busboy for streaming efficiency. Alternative parsers (Formidable, Busboy directly) offer varying levels of control over stream handling. Secure upload implementation requires MIME type validation, magic-number (file signature) verification, file-size limits, and safe filename handling.

**Beginner-Friendly Explanation:** When you upload a photo to a website, the browser packages the file into a special format called `multipart/form-data` and sends it to the server. The server needs middleware (like Multer) to unpack that package, separate the file from any text fields, validate that the file is what it claims to be, and save it somewhere. Doing this securely means checking the file's actual content (not just its extension), limiting how big it can be, and never trusting the filename the user provided.

### Key Characteristics

- **Multipart encoding:** Files are sent using `multipart/form-data`, never `application/json` or `urlencoded`.
- **Streaming by design:** Multer and Busboy process the request as a stream, avoiding loading entire files into memory.
- **Middleware-based:** Upload handling is implemented as Express middleware (`upload.single()`, `upload.array()`, `upload.fields()`).
- **Security-critical:** Unrestricted file upload is a top OWASP vulnerability (CWE-434); validation must happen server-side.
- **Validation layers:** MIME type checks, magic-number verification, extension allowlists, and size limits work together.
- **Storage choice matters:** Memory storage is fast but risky for large files; disk storage is safer for production.

### Prerequisites

- **Node.js runtime** (v18 or higher; Formidable v3 requires Node.js >= 20).
- **Express.js installed:** `npm install express`.
- **Multer installed:** `npm install multer` (Multer 2.x is the current stable line).
- **Basic understanding of HTTP:** POST requests, headers, and the request body.
- **Familiarity with Express middleware and routing.**

### Related Programming Areas

- **Security:** OWASP File Upload Cheat Sheet, magic-number validation, path traversal prevention.
- **Cloud storage:** Streaming uploads directly to S3, Cloudinary, or GCS.
- **Image processing:** Sharp for resizing and transcoding after upload.
- **Content Delivery:** Serving uploaded files via CDN with signed URLs.
- **Antivirus scanning:** ClamAV integration for uploaded file scanning.

### Core Concepts

1. **Multipart Forms & `multipart/form-data` Encoding** — the wire format for file uploads.
2. **Multer Middleware** — Memory storage vs. Disk storage.
3. **Alternative Parsers** — Formidable, Busboy for custom stream handling.
4. **File Validation** — MIME types, magic numbers/file signatures.
5. **File-Size Limits** — Handling `LIMIT_FILE_SIZE` errors.
6. **Multiple File Uploads** — Handling arrays and mixed fields.

---

## Core Concept 1: Multipart Forms & `multipart/form-data` Encoding

### Definitions

**Core Definition:** `multipart/form-data` is a media type defined in RFC 7578 that encodes form data — including binary files — as a sequence of parts, each separated by a unique boundary string.

**Technical Definition:** RFC 7578 defines the `multipart/form-data` media type for returning values from forms. Each part MUST have a `Content-Disposition` header field which MUST contain a `name` parameter and MAY contain a `filename` parameter. The body is delimited by a boundary string specified in the `Content-Type` header. The definition is derived from the multipart MIME types defined in RFC 2046. Unlike `application/x-www-form-urlencoded`, which cannot carry binary data efficiently, `multipart/form-data` preserves the raw bytes of each file.

**Beginner-Friendly Explanation:** When a form has a file input, the browser cannot send the file as regular text. Instead, it packages everything into a multi-part message: one part for each text field, one part for each file. Each part has a label (its name) and, for files, a filename. The whole package is separated by a boundary marker so the server knows where one part ends and the next begins. The HTML form must include `enctype="multipart/form-data"` for the browser to use this encoding.

### Purposes

- To encode form data that includes binary files (images, documents, videos) in a single HTTP request.
- To preserve the raw bytes of uploaded files without corruption from text encoding.
- To allow the server to distinguish between text fields and file fields using `Content-Disposition`.
- To provide a standardised format supported by all browsers and HTTP clients.

### Syntax Rules and Structure

**HTML Form:**
```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <input type="text" name="title" />
  <input type="file" name="avatar" />
  <button type="submit">Upload</button>
</form>
```

| Component | Breakdown |
|-----------|-----------|
| `action` | The server endpoint receiving the upload. |
| `method="post"` | File uploads must use POST (or PUT). |
| `enctype="multipart/form-data"` | Required for file uploads. Without it, the browser sends `application/x-www-form-urlencoded`. |
| `name` attribute | Identifies the field; Multer uses this to map `req.file` and `req.files`. |

**HTTP Request Structure:**
```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryABC123

------WebKitFormBoundaryABC123
Content-Disposition: form-data; name="title"

My Upload
------WebKitFormBoundaryABC123
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

(binary JPEG data)
------WebKitFormBoundaryABC123--
```

**Rules:**
- The `enctype` attribute must be `multipart/form-data`; Multer will not process any form which is not multipart.
- The boundary string must not appear in any of the part bodies.
- Each part has a `Content-Disposition` header with a `name` parameter.
- File parts also have a `filename` parameter and may have a `Content-Type`.
- The request `Content-Type` header includes the boundary value.

### Annotated Code Example

```js
// multipart-basic.js
const express = require('express');
const multer = require('multer');
const app = express();

// Multer with disk storage
const upload = multer({ dest: 'uploads/' });

app.post('/profile', upload.single('avatar'), (req, res) => {
  // req.file contains the uploaded file metadata
  console.log('File:', req.file);
  // req.body contains text fields
  console.log('Body:', req.body);

  res.json({
    file: req.file,
    body: req.body
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for a form with `title=My Upload` and an avatar file):**
```json
{
  "file": {
    "fieldname": "avatar",
    "originalname": "photo.jpg",
    "encoding": "7bit",
    "mimetype": "image/jpeg",
    "destination": "uploads/",
    "filename": "a1b2c3d4e5f6g7h8",
    "path": "uploads/a1b2c3d4e5f6g7h8",
    "size": 24576
  },
  "body": { "title": "My Upload" }
}
```

**Why this output:** Multer parses the multipart request, extracts the file into `req.file`, and populates `req.body` with the text fields. The `fieldname` matches the `name` attribute in the HTML form (`avatar`). The `originalname` is the filename the user provided. The `filename` is a randomly generated name Multer uses on disk to prevent collisions and path traversal.

### Real-World Cases

- **Profile picture uploads:** `POST /profile` with `enctype="multipart/form-data"`.
- **Document submission:** Job applications uploading CVs and cover letters.
- **Media sharing:** Social platforms uploading images and videos.
- **CSV imports:** Admin panels uploading data files for batch processing.

---

## Core Concept 2: Multer Middleware

### Definitions

**Core Definition:** Multer is a Node.js middleware for handling `multipart/form-data`, built on top of Busboy, that adds a `file` or `files` object and a `body` object to the Express request.

**Technical Definition:** Multer adds a `body` object and a `file` or `files` object to the request object. The `body` object contains the values of the text fields of the form, and the `file` or `files` object contains the files uploaded via the form. It supports three storage engines: `diskStorage` (full control over file destination and naming), `memoryStorage` (files held in memory as Buffers), and the default (files written to a temporary directory with random names). Multer provides `upload.single()`, `upload.array()`, `upload.fields()`, and `upload.none()` methods for different field configurations.

**Beginner-Friendly Explanation:** Multer is the tool that does the heavy lifting of unpacking the multipart form. It separates files from text fields and puts them where you can access them: `req.file` for a single file, `req.files` for multiple files, and `req.body` for text fields. You configure it once and then use it as middleware on your upload routes. Multer will not process any form which is not multipart (`multipart/form-data`).

### Purposes

- To parse multipart requests and make file data available on the request object.
- To handle both text fields and file uploads in a single request.
- To provide configurable storage (disk vs. memory) for different use cases.
- To enable file filtering, size limits, and naming control.
- To integrate seamlessly with Express middleware pipeline.

### Sub-Feature 2.1: Memory Storage vs. Disk Storage

#### Definitions

**Core Definition:** Memory storage holds uploaded files in RAM as Buffers, while disk storage writes them to the file system with configurable destination and filename.

**Technical Definition:** In memory storage, the `file.buffer` property contains the file's data as a Node.js Buffer, and the `file.path` property is absent. In disk storage, the `file.destination` and `file.filename` properties are set, and `file.buffer` is absent. Memory storage is suitable for small files processed immediately (e.g., image resizing, direct upload to S3); disk storage is suitable for larger files or files that need to persist.

#### Syntax Rules and Structure

**Memory Storage:**
```js
const storage = multer.memoryStorage();
const upload = multer({ storage, limits: { fileSize: 5 * 1024 * 1024 } });
```
| Property | Present | Description |
|----------|---------|-------------|
| `file.buffer` | Yes | File data as a Buffer. |
| `file.path` | No | Not written to disk. |
| `file.destination` | No | Not applicable. |
| `file.filename` | No | Not applicable. |

**Disk Storage:**
```js
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});
const upload = multer({ storage });
```
| Property | Present | Description |
|----------|---------|-------------|
| `file.buffer` | No | Not held in memory. |
| `file.path` | Yes | Full path on disk. |
| `file.destination` | Yes | Directory where file was saved. |
| `file.filename` | Yes | Name assigned on disk. |

**Rules:**
- Memory storage is **not suitable** for large files; the entire file is held in RAM and can exhaust memory under concurrent uploads.
- Disk storage with a custom `filename` function is essential to prevent path traversal attacks — never use the user-provided `originalname` directly.
- The default (no storage specified) writes to the OS temp directory with a random filename and no extension.
- Multer's default limits apply to both storage engines.

#### Annotated Code Example

```js
// multer-storage.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');
const app = express();

// --- Memory storage (small files, immediate processing) ---
const memoryUpload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 2 * 1024 * 1024 }  // 2MB
});

app.post('/avatar/memory', memoryUpload.single('avatar'), (req, res) => {
  // req.file.buffer contains the file data
  const fileSize = req.file.buffer.length;
  const isImage = req.file.mimetype.startsWith('image/');

  res.json({
    storage: 'memory',
    size: fileSize,
    isImage,
    hasBuffer: !!req.file.buffer,
    hasPath: !!req.file.path
  });
});

// --- Disk storage (larger files, persistent storage) ---
const diskStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    // Generate a safe random filename — never trust originalname
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname).toLowerCase();
    cb(null, randomName + ext);
  }
});

const diskUpload = multer({
  storage: diskStorage,
  limits: { fileSize: 50 * 1024 * 1024 }  // 50MB
});

app.post('/documents/disk', diskUpload.single('document'), (req, res) => {
  res.json({
    storage: 'disk',
    path: req.file.path,
    destination: req.file.destination,
    filename: req.file.filename,
    originalname: req.file.originalname,
    hasBuffer: !!req.file.buffer
  });
});

app.listen(3000, () => console.log('Multer storage demo on 3000'));
```

**Expected Output (for memory storage):**
```json
{
  "storage": "memory",
  "size": 24576,
  "isImage": true,
  "hasBuffer": true,
  "hasPath": false
}
```

**Expected Output (for disk storage):**
```json
{
  "storage": "disk",
  "path": "uploads/a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg",
  "destination": "uploads/",
  "filename": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg",
  "originalname": "photo.jpg",
  "hasBuffer": false
}
```

**Why this output:** Memory storage keeps the file in `req.file.buffer` and does not write to disk. Disk storage writes the file to the `uploads/` directory with a cryptographically random filename, preventing path traversal and collisions. The `originalname` is preserved for reference but is never used as the actual filename.

#### Constraints and Limitations

- **Memory storage risks:** Holding many large files in RAM can crash the process; always set a `fileSize` limit.
- **Disk storage cleanup:** Uploaded files are not automatically deleted; implement cleanup for abandoned uploads.
- **Destination must exist:** Multer does not create the destination directory; it must exist before the upload.
- **Filename sanitisation:** Always generate server-side filenames; never use `originalname` directly.

#### Real-World Cases

- **Memory storage:** Profile picture uploads that are immediately resized and uploaded to S3.
- **Disk storage:** Document management systems where files persist on disk.
- **Hybrid:** Memory storage for small thumbnails, disk storage for full-resolution originals.

---

### Sub-Feature 2.2: Multer Upload Methods

#### Syntax Rules and Structure

| Method | `req` Property | Use Case |
|--------|---------------|----------|
| `upload.single(fieldname)` | `req.file` | One file from one field. |
| `upload.array(fieldname, maxCount)` | `req.files` (array) | Multiple files from one field. |
| `upload.fields(fields)` | `req.files` (object) | Files from multiple named fields. |
| `upload.none()` | `req.body` only | Text-only multipart forms. |

#### Annotated Code Example

```js
// multer-methods.js
const express = require('express');
const multer = require('multer');
const app = express();

const upload = multer({ dest: 'uploads/' });

// Single file
app.post('/single', upload.single('avatar'), (req, res) => {
  res.json({ file: req.file?.fieldname, body: req.body });
});

// Array of files (same field, up to 5)
app.post('/array', upload.array('photos', 5), (req, res) => {
  res.json({
    count: req.files?.length,
    files: req.files?.map(f => f.fieldname)
  });
});

// Mixed fields
const cpUpload = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]);
app.post('/fields', cpUpload, (req, res) => {
  res.json({
    avatar: req.files['avatar']?.[0]?.fieldname,
    galleryCount: req.files['gallery']?.length,
    body: req.body
  });
});

// Text-only multipart
app.post('/none', upload.none(), (req, res) => {
  res.json({ body: req.body });
});

app.listen(3000, () => console.log('Multer methods on 3000'));
```

**Expected Output (for `/array` with 3 photos):**
```json
{"count":3,"files":["photos","photos","photos"]}
```

**Expected Output (for `/fields` with avatar + 2 gallery images):**
```json
{"avatar":"avatar","galleryCount":2,"body":{}}
```

**Why this output:** `upload.single()` puts one file in `req.file`. `upload.array()` puts an array in `req.files`. `upload.fields()` puts an object keyed by field name, where each value is an array of files. `upload.none()` only parses text fields, rejecting any file uploads.

---

## Core Concept 3: Alternative Parsers (Formidable, Busboy)

### Definitions

**Core Definition:** Formidable and Busboy are lower-level multipart parsers that offer more control over stream handling than Multer, at the cost of manual implementation of storage, validation, and response logic.

**Technical Definition:** **Busboy** is a streaming parser for incoming HTML form data. It is a Writable stream that emits `file` and `field` events as it parses. **Formidable** is a higher-level module for parsing form data, especially file uploads, offering a fast (~900–2500 mb/sec) streaming multipart parser, automatic writing to disk, and a plugin API. Multer itself is built on top of Busboy. Formidable v3 requires Node.js >= 20 and is the current stable line; v1 and v2 are deprecated and vulnerable if not implemented properly.

**Beginner-Friendly Explanation:** Multer is a ready-made tool — it does the parsing and gives you the files. Busboy and Formidable are more like raw materials. Busboy gives you events as it parses ("here's a file part, here's a field"), and you decide what to do with each. Formidable is a middle ground: it parses and can write files to disk automatically, but gives you more control than Multer.

### Purposes

- To handle multipart parsing with custom stream processing logic.
- To stream files directly to cloud storage (S3, GCS) without touching disk.
- To process files in real time as they are parsed (e.g., virus scanning, format conversion).
- To support advanced use cases not covered by Multer's storage engine abstraction.

### Sub-Feature 3.1: Busboy (Raw Streaming)

#### Syntax Rules and Structure

```js
const Busboy = require('busboy');

app.post('/upload', (req, res) => {
  const busboy = Busboy({ headers: req.headers });

  busboy.on('file', (fieldname, file, filename, encoding, mimetype) => {
    console.log('File:', fieldname, filename, mimetype);
    file.on('data', (chunk) => { /* process chunk */ });
    file.on('end', () => { /* file finished */ });
  });

  busboy.on('field', (fieldname, val) => {
    console.log('Field:', fieldname, val);
  });

  busboy.on('finish', () => {
    res.json({ received: true });
  });

  req.pipe(busboy);
});
```

| Event | Arguments | Description |
|-------|-----------|-------------|
| `file` | `fieldname, file, filename, encoding, mimetype` | Emitted for each file part. |
| `field` | `fieldname, val, ...` | Emitted for each text field. |
| `finish` | — | All parts parsed. |

#### Annotated Code Example

```js
// busboy-upload.js
const express = require('express');
const Busboy = require('busboy');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');
const app = express();

app.post('/upload', (req, res) => {
  const busboy = Busboy({ headers: req.headers });
  const uploadedFiles = [];

  busboy.on('file', (fieldname, file, filename, encoding, mimetype) => {
    // Generate safe filename
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(filename).toLowerCase();
    const saveTo = path.join('uploads', randomName + ext);

    // Stream directly to disk
    const writeStream = fs.createWriteStream(saveTo);
    file.pipe(writeStream);

    file.on('end', () => {
      uploadedFiles.push({
        fieldname,
        originalname: filename,
        savedAs: randomName + ext,
        mimetype
      });
    });
  });

  busboy.on('field', (fieldname, val) => {
    console.log('Field:', fieldname, val);
  });

  busboy.on('finish', () => {
    res.json({ files: uploadedFiles });
  });

  req.pipe(busboy);
});

app.listen(3000, () => console.log('Busboy upload on 3000'));
```

**Expected Output:**
```json
{
  "files": [
    {
      "fieldname": "avatar",
      "originalname": "photo.jpg",
      "savedAs": "a1b2c3d4e5f6a7b8.jpg",
      "mimetype": "image/jpeg"
    }
  ]
}
```

**Why this output:** Busboy emits the `file` event as soon as it encounters a file part, providing a readable stream (`file`) that can be piped directly to a write stream, S3 upload stream, or processing pipeline. No intermediate buffering is required.

#### Constraints and Limitations

- Busboy does **not** automatically write files to disk; you must pipe the stream yourself.
- Busboy does **not** provide a `body` object; you must collect fields manually.
- Busboy does not perform file size limits by default; you must enforce them manually or use its `limits` option.

---

### Sub-Feature 3.2: Formidable

#### Syntax Rules and Structure

```js
const formidable = require('formidable');

app.post('/upload', (req, res) => {
  const form = formidable({
    uploadDir: 'uploads/',
    keepExtensions: true,
    maxFileSize: 10 * 1024 * 1024  // 10MB
  });

  form.parse(req, (err, fields, files) => {
    if (err) return res.status(400).json({ error: err.message });
    res.json({ fields, files });
  });
});
```

| Option | Description |
|--------|-------------|
| `uploadDir` | Directory for uploaded files. |
| `keepExtensions` | Preserve original file extension. |
| `maxFileSize` | Maximum file size in bytes. |
| `multiples` | Allow multiple files with the same field name. |

#### Annotated Code Example

```js
// formidable-upload.js
const express = require('express');
const formidable = require('formidable');
const path = require('path');
const app = express();

app.post('/upload', (req, res) => {
  const form = formidable({
    uploadDir: 'uploads/',
    keepExtensions: true,
    maxFileSize: 10 * 1024 * 1024,  // 10MB
    multiples: true
  });

  form.parse(req, (err, fields, files) => {
    if (err) {
      if (err.code === 'ETOOBIG') {
        return res.status(400).json({ error: 'File too large. Max 10MB.' });
      }
      return res.status(500).json({ error: err.message });
    }

    // fields: { title: ['My Upload'] }
    // files: { avatar: [{ newFilename, originalFilename, mimetype, size }] }
    res.json({
      fields,
      fileCount: Object.keys(files).length,
      files: Object.fromEntries(
        Object.entries(files).map(([key, arr]) => [
          key,
          arr.map(f => ({
            newFilename: f.newFilename,
            originalFilename: f.originalFilename,
            mimetype: f.mimetype,
            size: f.size
          }))
        ])
      )
    });
  });
});

app.listen(3000, () => console.log('Formidable upload on 3000'));
```

**Expected Output:**
```json
{
  "fields": { "title": ["My Upload"] },
  "fileCount": 1,
  "files": {
    "avatar": [
      {
        "newFilename": "a1b2c3d4e5f6.jpg",
        "originalFilename": "photo.jpg",
        "mimetype": "image/jpeg",
        "size": 24576
      }
    ]
  }
}
```

**Why this output:** Formidable parses the multipart request, writes files to `uploads/` with safe random names, and provides both `fields` and `files` objects in the callback. The `newFilename` is the server-generated safe name; `originalFilename` is the user-provided name. The `multiples: true` option allows multiple files with the same field name.

#### Constraints and Limitations

- Formidable v1 and v2 are **deprecated** and vulnerable if not implemented properly; use v3 (Node.js >= 20).
- Formidable v3 is a dual ESM/CommonJS package; ensure your project configuration is compatible.
- Formidable does not perform magic-number validation; you must add it separately.

---

## Core Concept 4: File Validation

### Definitions

**Core Definition:** File validation is the process of verifying that an uploaded file conforms to expected type, size, and content constraints before it is processed or persisted.

**Technical Definition:** Secure file validation operates on multiple layers: **extension validation** (allowlist of safe extensions), **MIME type validation** (checking `file.mimetype` against an allowlist), **magic-number validation** (reading the first bytes of the file to verify its true format), and **content validation** (AV scanning, CDR). The OWASP File Upload Cheat Sheet states: "Validate the file type, don't trust the Content-Type header as it can be spoofed." Magic numbers are unique byte sequences at the start of a file that unambiguously identify the true format; forging them is much harder than changing a filename or header.

**Beginner-Friendly Explanation:** When someone uploads a file, they could lie about what it is. They could rename a malicious `.exe` to `.jpg`, or set the `Content-Type` header to `image/png` when it's actually a script. Checking the file's magic number — the first few bytes that every file format has — is the most reliable way to know what the file really is. It's like checking a person's fingerprint instead of just their ID card.

### Purposes

- To prevent malicious files from being uploaded and executed on the server.
- To ensure only allowed file types are accepted (e.g., only JPEG and PNG for avatars).
- To detect and reject files whose content does not match their claimed type.
- To comply with OWASP ASVS 5.0 V5 (File Handling) requirements.

### Sub-Feature 4.1: MIME Type and Extension Validation (Multer `fileFilter`)

#### Syntax Rules and Structure

```js
const fileFilter = (req, file, cb) => {
  const allowedMimes = ['image/jpeg', 'image/png', 'image/gif'];
  const allowedExts = /jpeg|jpg|png|gif/;

  const extname = allowedExts.test(
    path.extname(file.originalname).toLowerCase()
  );
  const mimetype = allowedMimes.includes(file.mimetype);

  if (extname && mimetype) {
    cb(null, true);   // Accept the file
  } else {
    cb(new Error('Only JPEG, PNG, and GIF images are allowed'), false);
  }
};

const upload = multer({
  storage: multer.memoryStorage(),
  fileFilter,
  limits: { fileSize: 2 * 1024 * 1024 }
});
```

**Rules:**
- Validate **both** MIME type and file extension; an attacker may spoof one but not both.
- Use an allowlist, never a denylist.
- The `fileFilter` function is called for each file; call `cb(null, true)` to accept or `cb(null, false)` to reject.
- `fileFilter` runs **before** the file is written to disk or memory, so rejected files are never stored.

#### Annotated Code Example

```js
// multer-filefilter.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const app = express();

const fileFilter = (req, file, cb) => {
  const allowedMimes = ['image/jpeg', 'image/png', 'image/gif'];
  const allowedExts = /jpeg|jpg|png|gif/;

  const extname = allowedExts.test(
    path.extname(file.originalname).toLowerCase()
  );
  const mimetype = allowedMimes.includes(file.mimetype);

  console.log(`Checking ${file.originalname}: extname=${extname}, mimetype=${mimetype}`);

  if (extname && mimetype) {
    cb(null, true);
  } else {
    cb(new Error('Only JPEG, PNG, and GIF images are allowed'), false);
  }
};

const upload = multer({
  storage: multer.memoryStorage(),
  fileFilter,
  limits: { fileSize: 2 * 1024 * 1024 }
});

app.post('/avatar', upload.single('avatar'), (req, res) => {
  res.json({
    accepted: true,
    originalname: req.file.originalname,
    mimetype: req.file.mimetype,
    size: req.file.size
  });
});

// Error handler for fileFilter rejections
app.use((err, req, res, next) => {
  if (err) {
    return res.status(400).json({ error: err.message });
  }
  next();
});

app.listen(3000, () => console.log('FileFilter validation on 3000'));
```

**Expected Output (for a valid JPEG):**
```json
{"accepted":true,"originalname":"photo.jpg","mimetype":"image/jpeg","size":24576}
```

**Expected Output (for a spoofed `.jpg` that is actually an executable):**
```json
{"error":"Only JPEG, PNG, and GIF images are allowed"}
```

**Why this output:** The `fileFilter` function checks both the file extension and the MIME type. A file named `photo.jpg` with `Content-Type: image/jpeg` passes both checks. A malicious executable renamed to `photo.jpg` but with `Content-Type: application/x-executable` fails the MIME check. The file is rejected before any data is written.

### Sub-Feature 4.2: Magic-Number (File Signature) Validation

#### Definitions

**Core Definition:** Magic-number validation reads the first few bytes of a file and compares them against known byte signatures to determine the file's true format.

**Technical Definition:** Magic numbers are unique byte sequences — usually found right at the start of a file — that unambiguously identify the true format. Common signatures include PNG (`89 50 4E 47 0D 0A 1A 0A`), JPEG (`FF D8 FF`), GIF (`47 49 46 38`), and PDF (`25 50 44 46 2D`). The `file-type` npm package detects signatures from a Buffer or stream and returns the detected MIME type. Magic-number validation is considered the most reliable form of file-type verification because it inspects the file's binary structure, which is much harder to forge than a filename or header.

#### Syntax Rules and Structure

```js
const { fileTypeFromBuffer } = require('file-type');

async function validateMagicNumber(buffer, allowedMimes) {
  const type = await fileTypeFromBuffer(buffer);
  if (!type) throw new Error('Unknown or unsupported file type');
  if (!allowedMimes.includes(type.mime)) {
    throw new Error(`Disallowed file type: ${type.mime}`);
  }
  return type;
}
```

**Common Magic Numbers:**

| Format | Hex (offset 0) | Notes |
|--------|----------------|-------|
| PNG | `89 50 4E 47 0D 0A 1A 0A` | Always 8 bytes. |
| JPEG | `FF D8 FF` | Covers JFIF and EXIF variants. |
| GIF | `47 49 46 38 37 61` (GIF87a) / `47 49 46 38 39 61` (GIF89a) | Six bytes. |
| PDF | `25 50 44 46 2D` (`%PDF-`) | Five bytes. |
| ZIP | `50 4B 03 04` / `50 4B 05 06` | DOCX, ODT, and APK are ZIP containers. |
| MP4 | `66 74 79 70` (offset 4) | Preceded by four-byte size field. |

**Rules:**
- Read only the first 8 KiB of the file; magic numbers are at the start.
- Magic-number validation should be performed **in addition to** MIME type and extension checks.
- For streaming uploads, use `fileTypeFromStream` to inspect the beginning of the stream without buffering the entire file.
- The `file-type` package is ESM-only in v22+; ensure your project supports ESM.

#### Annotated Code Example

```js
// magic-number-validation.js
const express = require('express');
const multer = require('multer');
const { fileTypeFromBuffer } = require('file-type');
const app = express();

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 5 * 1024 * 1024 }
});

app.post('/upload', upload.single('file'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }

  try {
    // Validate magic number (first 8 KiB is sufficient)
    const type = await fileTypeFromBuffer(req.file.buffer);

    if (!type) {
      return res.status(400).json({ error: 'Unknown file type' });
    }

    const allowedMimes = ['image/png', 'image/jpeg', 'application/pdf'];
    if (!allowedMimes.includes(type.mime)) {
      return res.status(400).json({
        error: 'Disallowed file type',
        detected: type.mime,
        allowed: allowedMimes
      });
    }

    res.json({
      valid: true,
      detectedMime: type.mime,
      detectedExt: type.ext,
      originalname: req.file.originalname,
      claimedMime: req.file.mimetype,
      size: req.file.size
    });
  } catch (err) {
    res.status(500).json({ error: 'Validation failed' });
  }
});

app.listen(3000, () => console.log('Magic number validation on 3000'));
```

**Expected Output (for a valid PNG):**
```json
{
  "valid": true,
  "detectedMime": "image/png",
  "detectedExt": "png",
  "originalname": "image.png",
  "claimedMime": "image/png",
  "size": 4096
}
```

**Expected Output (for a spoofed file — a ZIP renamed to `.png`):**
```json
{
  "valid": true,
  "detectedMime": "application/zip",
  "detectedExt": "zip",
  "originalname": "malicious.png",
  "claimedMime": "image/png",
  "size": 8192
}
```

**Expected Output (when the detected type is not in the allowlist):**
```json
{
  "error": "Disallowed file type",
  "detected": "application/zip",
  "allowed": ["image/png", "image/jpeg", "application/pdf"]
}
```

**Why this output:** The magic-number check reads the actual bytes of the file and detects the true format. A ZIP file renamed to `.png` is detected as `application/zip` — not `image/png` — and is rejected if ZIP is not in the allowlist. This is the most reliable form of file-type validation because it inspects the binary content, not the filename or header.

### Real-World Cases

- **Image upload security:** Rejecting files that claim to be images but are actually scripts.
- **PDF processing:** Ensuring uploaded PDFs are genuine PDFs before passing to a parser.
- **Cloud storage:** Validating file types before streaming to S3 or GCS.
- **Compliance:** Meeting OWASP ASVS 5.0 V5 file handling requirements.

---

## Core Concept 5: File-Size Limits and `LIMIT_FILE_SIZE` Errors

### Definitions

**Core Definition:** File-size limits are server-enforced maximums on the size of uploaded files, designed to prevent denial-of-service (DoS) attacks and resource exhaustion.

**Technical Definition:** Multer enforces size limits via the `limits` option. The `fileSize` limit specifies the maximum size of each file in bytes. When a file exceeds this limit, Multer aborts the upload and emits a `LIMIT_FILE_SIZE` error (a `MulterError` with `code === 'LIMIT_FILE_SIZE'`). Multer's `limits` object also supports `fieldNameSize`, `fieldSize`, `fields`, `files`, `parts`, and `headerPairs`. When a limit is exceeded, Multer delegates the error to Express via `next(err)`, where it can be handled by error-handling middleware.

**Beginner-Friendly Explanation:** Without a size limit, someone could upload a 100GB file and fill up your server's disk, crashing the entire application. Setting a `fileSize` limit tells Multer to stop the upload as soon as the limit is exceeded. When that happens, Multer throws an error with the code `LIMIT_FILE_SIZE`. You catch that error and return a friendly message like "File must be 2MB or smaller" instead of letting the server crash.

### Purposes

- To prevent denial-of-service (DoS) attacks via large file uploads.
- To enforce business rules (e.g., avatars must be under 2MB).
- To protect server memory and disk from exhaustion.
- To provide clear error messages to users when their file is too large.

### Syntax Rules and Structure

```js
const upload = multer({
  storage: multer.diskStorage({ destination: 'uploads/' }),
  limits: {
    fileSize: 5 * 1024 * 1024,   // 5MB per file
    files: 10,                    // Maximum 10 files
    fieldSize: 1024 * 1024,       // 1MB per text field
    fields: 20                    // Maximum 20 text fields
  }
});
```

| Limit | Default | Description |
|-------|---------|-------------|
| `fileSize` | `Infinity` | Max bytes per file. |
| `files` | `Infinity` | Max number of file fields. |
| `fields` | `Infinity` | Max number of text fields. |
| `fieldSize` | `1MB` | Max bytes per text field. |
| `parts` | `Infinity` | Max total parts (fields + files). |
| `headerPairs` | `2000` | Max header pairs to parse. |

**Rules:**
- `fileSize` is per file, not per request.
- When the limit is exceeded, Multer aborts the upload and passes a `MulterError` to Express.
- In Multer 2.3+, `limits.fileSize` is inclusive: exactly `maxUploadSize` is accepted; one extra byte triggers `LIMIT_FILE_SIZE`.
- Multer's `limits.fileSize` counts bytes as they stream; the connection is aborted immediately when exceeded.
- Always handle `LIMIT_FILE_SIZE` in error-handling middleware with a user-friendly response.

### Annotated Code Example

```js
// file-size-limit.js
const express = require('express');
const multer = require('multer');
const app = express();

const upload = multer({
  storage: multer.diskStorage({
    destination: 'uploads/',
    filename: (req, file, cb) => {
      cb(null, Date.now() + '-' + file.originalname);
    }
  }),
  limits: {
    fileSize: 2 * 1024 * 1024,   // 2MB per file
    files: 5                      // Maximum 5 files
  }
});

app.post('/avatar', upload.single('avatar'), (req, res) => {
  res.json({
    message: 'Upload successful',
    file: req.file.filename,
    size: req.file.size
  });
});

// Error-handling middleware for Multer errors
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({
        error: 'File too large',
        message: 'Maximum file size is 2MB',
        code: err.code
      });
    }
    if (err.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).json({
        error: 'Too many files',
        message: 'Maximum 5 files allowed',
        code: err.code
      });
    }
    return res.status(400).json({ error: err.message, code: err.code });
  }

  if (err) {
    return res.status(500).json({ error: 'Upload failed' });
  }
  next();
});

app.listen(3000, () => console.log('File size limit on 3000'));
```

**Expected Output (for a file under 2MB):**
```json
{"message":"Upload successful","file":"1737000000000-photo.jpg","size":1048576}
```

**Expected Output (for a file over 2MB):**
```json
{
  "error": "File too large",
  "message": "Maximum file size is 2MB",
  "code": "LIMIT_FILE_SIZE"
}
```

**Why this output:** The `fileSize` limit is set to 2MB (2 * 1024 * 1024 bytes). When a file exceeds this, Multer aborts the stream, emits a `MulterError` with `code: 'LIMIT_FILE_SIZE'`, and passes it to the error-handling middleware. The middleware checks the error type and returns a structured 400 response. Without this error handler, the default Express error handler would return a 500 with a stack trace.

### Real-World Cases

- **Profile pictures:** 2MB limit with resize-on-upload.
- **Document uploads:** 10MB limit for PDFs.
- **Video uploads:** 100MB limit with chunked upload for larger files.
- **CSV imports:** 5MB limit to prevent memory exhaustion during parsing.

---

## Core Concept 6: Multiple File Uploads (Arrays and Mixed Fields)

### Definitions

**Core Definition:** Multiple file uploads allow a single request to carry several files, either from the same field (array) or from different named fields (mixed fields).

**Technical Definition:** Multer provides `upload.array(fieldname, maxCount)` for multiple files from the same field, and `upload.fields([{ name, maxCount }])` for files from different named fields. In the array case, files are available in `req.files` as an array. In the fields case, `req.files` is an object where each key is a field name and each value is an array of files. The `maxCount` option limits the number of files per field.

**Beginner-Friendly Explanation:** Sometimes a form needs to upload more than one file at once. A photo gallery might have one "photos" field with multiple files. A profile edit form might have an "avatar" field and a "gallery" field. Multer handles both: `upload.array()` for one field with many files, `upload.fields()` for multiple fields with different names.

### Purposes

- To accept multiple files in a single request without multiple round trips.
- To handle mixed-field forms (e.g., avatar + gallery + documents).
- To enforce per-field file count limits.
- To simplify client-side code by batching uploads.

### Syntax Rules and Structure

**Array Upload:**
```js
app.post('/photos', upload.array('photos', 12), (req, res) => {
  // req.files is an array of files
  // req.body contains text fields
});
```

**Fields Upload:**
```js
const cpUpload = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]);
app.post('/cool-profile', cpUpload, (req, res) => {
  // req.files['avatar'][0] -> File
  // req.files['gallery'] -> Array
});
```

**Rules:**
- `upload.array()` — all files must use the same field name; `maxCount` limits the count.
- `upload.fields()` — each field is specified with `name` and optional `maxCount`.
- In the array case, `req.files` is an array.
- In the fields case, `req.files` is an object keyed by field name.
- Text fields are always in `req.body`, regardless of file upload method.

### Annotated Code Example

```js
// multiple-uploads.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');
const app = express();

const storage = multer.diskStorage({
  destination: 'uploads/',
  filename: (req, file, cb) => {
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname).toLowerCase();
    cb(null, randomName + ext);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 },
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/gif'];
    cb(null, allowed.includes(file.mimetype));
  }
});

// --- Array: multiple files from one field ---
app.post('/photos', upload.array('photos', 10), (req, res) => {
  res.json({
    type: 'array',
    count: req.files.length,
    files: req.files.map(f => ({
      fieldname: f.fieldname,
      originalname: f.originalname,
      savedAs: f.filename,
      size: f.size
    })),
    body: req.body
  });
});

// --- Fields: mixed fields with different names ---
const cpUpload = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]);

app.post('/profile', cpUpload, (req, res) => {
  const avatar = req.files['avatar']?.[0];
  const gallery = req.files['gallery'] || [];

  res.json({
    type: 'fields',
    avatar: avatar ? {
      originalname: avatar.originalname,
      savedAs: avatar.filename,
      size: avatar.size
    } : null,
    galleryCount: gallery.length,
    gallery: gallery.map(f => ({
      originalname: f.originalname,
      savedAs: f.filename
    })),
    body: req.body
  });
});

app.listen(3000, () => console.log('Multiple uploads on 3000'));
```

**Expected Output (for `/photos` with 3 images):**
```json
{
  "type": "array",
  "count": 3,
  "files": [
    {"fieldname":"photos","originalname":"a.jpg","savedAs":"a1b2...jpg","size":1024},
    {"fieldname":"photos","originalname":"b.jpg","savedAs":"c3d4...jpg","size":2048},
    {"fieldname":"photos","originalname":"c.jpg","savedAs":"e5f6...jpg","size":3072}
  ],
  "body": {}
}
```

**Expected Output (for `/profile` with avatar + 2 gallery images):**
```json
{
  "type": "fields",
  "avatar": {"originalname":"me.jpg","savedAs":"a1b2...jpg","size":1024},
  "galleryCount": 2,
  "gallery": [
    {"originalname":"g1.jpg","savedAs":"c3d4...jpg"},
    {"originalname":"g2.jpg","savedAs":"e5f6...jpg"}
  ],
  "body": {}
}
```

**Why this output:** `upload.array('photos', 10)` collects all files from the `photos` field into an array in `req.files`. `upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'gallery', maxCount: 8 }])` collects files into an object where `req.files['avatar']` is an array (with at most 1 file) and `req.files['gallery']` is an array (with at most 8 files). The `maxCount` limits prevent excessive uploads per field.

### Real-World Cases

- **E-commerce product listings:** `upload.fields([{ name: 'thumbnail', maxCount: 1 }, { name: 'images', maxCount: 10 }])`.
- **Social media posts:** `upload.array('media', 4)` for up to 4 images/videos per post.
- **Document management:** `upload.fields([{ name: 'contract', maxCount: 1 }, { name: 'attachments', maxCount: 5 }])`.
- **Profile settings:** `upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'banner', maxCount: 1 }])`.

---

## References

- Multer — npm Documentation — https://www.npmjs.com/package/multer
- Express multer middleware — https://expressjs.com/en/resources/middleware/multer.html
- Multer GitHub README — https://github.com/expressjs/multer
- Formidable — npm Documentation — https://www.npmjs.com/package/formidable
- Busboy — npm Documentation (fastify fork) — https://www.npmjs.com/package/@fastify/busboy
- Busboy — original README (rdrr.io mirror) — https://rdrr.io/github/sebsilas/SAA/f/test_apps/aSAA/node/node_modules/busboy/README.md
- OWASP File Upload Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- RFC 7578 — Returning Values from Forms: multipart/form-data — https://www.rfc-editor.org/rfc/rfc7578
- Secure API file uploads with magic numbers (Transloadit) — https://assets.transloadit.com/devtips/secure-api-file-uploads-with-magic-numbers/
- file-type — npm Documentation — https://www.npmjs.com/package/file-type
- Missing File Size Limits in Multer Uploads (GitHub Issue) — https://github.com/gdg-charusat/Zaplink_backend/issues/7
- Multer 2.3 limits.fileSize is inclusive (GitHub) — https://github.com/expressjs/multer/pull/1096
- How to validate file uploads in Node.js (CoreUI) — https://coreui.io/blog/how-to-validate-file-uploads-in-node-js/
- How to upload multiple files in Node.js (CoreUI) — https://coreui.io/blog/how-to-upload-multiple-files-in-node-js/