# Secure File Handling — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Secure file handling is the set of practices, validation layers, storage configurations, and access controls that prevent uploaded files from introducing vulnerabilities — including remote code execution, path traversal, cross-site scripting, and resource exhaustion — into an application.

**Technical Definition:** Secure file handling encompasses the full lifecycle of an uploaded file: receiving it via `multipart/form-data`, validating its true type (not just its declared type), sanitising its filename to prevent path traversal (CWE-22), scanning it for malware, storing it outside the web root or in a private object storage bucket, and controlling access through authenticated proxies or time-limited signed URLs. The OWASP File Upload Cheat Sheet defines the core principles: list allowed extensions, validate the file type (never trust the `Content-Type` header), change the filename to something application-generated, set file size and filename length limits, store files outside the webroot, run antivirus scanning, and protect the upload endpoint from CSRF attacks.

**Beginner-Friendly Explanation:** When someone uploads a file to your server, you cannot trust anything about it. The filename might contain `../../etc/passwd` to try to read your server's password file. The file might be named `photo.jpg` but actually be a malicious script that executes when accessed. The file might be a ZIP bomb designed to crash your server. Secure file handling is the practice of checking everything: what the file actually is (not what it claims to be), what its name is, how big it is, whether it contains malware, and where you store it so that it cannot be executed or accessed without permission.

### Key Characteristics

- **Zero trust:** Every uploaded file is untrusted input, just like a text field.
- **Defence in depth:** Multiple validation layers (extension, MIME type, magic number) work together.
- **Server-side filenames:** The safest pattern is to discard the client-supplied filename entirely and generate a random, server-side identifier (UUID or hash).
- **Storage isolation:** Uploaded files must be stored outside the web root to prevent execution and direct access.
- **Access control:** Private buckets with signed URLs, or authenticated proxy routes, replace public exposure.
- **Malware scanning:** ClamAV or equivalent antivirus scanning is a standard OWASP recommendation.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express multer`.
- **A ClamAV installation** (optional, for malware scanning).
- **A cloud storage account** (AWS S3, GCS, Azure Blob) for object storage examples.
- **Basic understanding of HTTP:** Multipart requests, headers, and the request–response cycle.

### Related Programming Areas

- **File Uploads (Multer):** The middleware that parses multipart requests.
- **File Storage & Processing:** Where files are stored and how they are transformed.
- **Security & Token Storage:** Access control for serving files.
- **OWASP Top 10:** A01 (Broken Access Control), A03 (Injection), A05 (Security Misconfiguration).
- **CWE-434:** Unrestricted Upload of File with Dangerous Type.
- **CWE-22:** Improper Limitation of a Pathname to a Restricted Directory (Path Traversal).

### Core Concepts

1. **Strict File Type Validation** — Preventing executable uploads like `.exe`, `.sh`, `.php`.
2. **Filename Sanitisation** — Preventing directory traversal attacks using `path.basename`.
3. **Malware Scanning** — Integrating ClamAV or virus-scanning APIs.
4. **Storage Isolation** — Storing uploads outside the web root.
5. **Access Control & Privacy** — Public vs. private buckets.
6. **Signed URLs** — Time-limited third-party access for secure file delivery.

---

## Core Concept 1: Strict File Type Validation

### Definitions

**Core Definition:** Strict file type validation is the practice of verifying that an uploaded file is genuinely of an allowed type by inspecting its actual binary content, not merely trusting the filename extension or the client-declared `Content-Type` header.

**Technical Definition:** File type validation operates on three layers. **Extension validation** checks the filename suffix against an allowlist of safe extensions. **MIME type validation** checks the `Content-Type` header sent by the client — which is trivially spoofable. **Magic-number validation** reads the first bytes of the file and compares them against known binary signatures (e.g., PNG is `89 50 4E 47`, JPEG is `FF D8 FF`, PDF is `25 50 44 46`). The OWASP guidance is explicit: "Validate the file type, don't trust the Content-Type header as it can be spoofed" and "Don't trust that the file content matches that you'd expect from the filename or content-type, you will need to do your own file content checking".

**Beginner-Friendly Explanation:** When someone uploads a file, they could rename a malicious script to `photo.jpg` or set the `Content-Type` header to `image/png` while the file is actually a PHP shell. The only way to know what the file truly is is to look at its first few bytes — its "magic number." PNG files always start with the same 8 bytes, JPEG files start with `FF D8 FF`, and PDFs start with `%PDF-`. Reading those bytes is much harder to fake than changing a filename.

### Purposes

- To prevent executable uploads like `.exe`, `.sh`, `.php`, `.jsp`, and `.asp`.
- To prevent attacks that exploit file parsers (e.g., ImageTragick, XXE).
- To ensure that only files matching the application's business requirements are accepted.
- To detect MIME spoofing and extension spoofing attacks.

### Syntax Rules and Structure

**Extension Allowlist (Multer `fileFilter`):**
```js
const fileFilter = (req, file, cb) => {
  const allowedExts = /jpeg|jpg|png|gif|webp/;
  const extname = allowedExts.test(path.extname(file.originalname).toLowerCase());
  if (extname) cb(null, true);
  else cb(new Error('Only image files are allowed'), false);
};
```

**Magic-Number Validation (`file-type` package):**
```js
const { fileTypeFromBuffer } = require('file-type');

const type = await fileTypeFromBuffer(req.file.buffer);
if (!type || !['image/jpeg', 'image/png', 'image/webp'].includes(type.mime)) {
  throw new Error('Invalid file type');
}
```

| Validation Layer | What It Checks | Spoofable? |
|-----------------|----------------|-----------|
| Extension | Filename suffix | Yes (rename `.exe` to `.jpg`). |
| MIME type (`Content-Type`) | Client-declared type | Yes (header can be set arbitrarily). |
| Magic number | Actual binary content | Very difficult (requires polyglot file). |

**Rules:**
- Use an **allowlist** of permitted types, never a denylist.
- Validate the extension **and** the MIME type **and** the magic number.
- Reject files whose detected type does not match the claimed type.
- The `fileFilter` function runs before the file is written to disk or memory, so rejected files are never stored.
- Common magic numbers: PNG (`89 50 4E 47 0D 0A 1A 0A`), JPEG (`FF D8 FF`), GIF (`47 49 46 38`), PDF (`25 50 44 46 2D`).

### Annotated Code Example

```js
// strict-file-validation.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const { fileTypeFromBuffer } = require('file-type');
const app = express();

// Layer 1: Extension + MIME type allowlist (fast rejection)
const fileFilter = (req, file, cb) => {
  const allowedMimes = ['image/jpeg', 'image/png', 'image/webp'];
  const allowedExts = /jpeg|jpg|png|webp/;

  const extname = allowedExts.test(
    path.extname(file.originalname).toLowerCase()
  );
  const mimetype = allowedMimes.includes(file.mimetype);

  if (extname && mimetype) cb(null, true);
  else cb(new Error('Only JPEG, PNG, and WebP images are allowed'), false);
};

const upload = multer({
  storage: multer.memoryStorage(),
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }
});

// Layer 2: Magic-number validation (authoritative check)
app.post('/upload', upload.single('file'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }

  try {
    // Read first 8KB for magic-number detection
    const type = await fileTypeFromBuffer(req.file.buffer);

    if (!type) {
      return res.status(400).json({ error: 'Unknown file type' });
    }

    const allowedMimes = ['image/jpeg', 'image/png', 'image/webp'];
    if (!allowedMimes.includes(type.mime)) {
      return res.status(400).json({
        error: 'File content does not match an allowed type',
        detected: type.mime,
        allowed: allowedMimes
      });
    }

    // Cross-check: detected type must match claimed MIME type
    if (type.mime !== req.file.mimetype) {
      return res.status(400).json({
        error: 'MIME type mismatch — possible spoofing',
        claimed: req.file.mimetype,
        detected: type.mime
      });
    }

    res.json({
      valid: true,
      detectedMime: type.mime,
      detectedExt: type.ext,
      originalname: req.file.originalname,
      size: req.file.size
    });
  } catch (err) {
    res.status(500).json({ error: 'Validation failed' });
  }
});

app.listen(3000, () => console.log('Strict validation on 3000'));
```

**Expected Output (for a valid JPEG):**
```json
{
  "valid": true,
  "detectedMime": "image/jpeg",
  "detectedExt": "jpg",
  "originalname": "photo.jpg",
  "size": 24576
}
```

**Expected Output (for a PHP script renamed to `photo.jpg` with `Content-Type: image/jpeg`):**
```json
{
  "error": "MIME type mismatch — possible spoofing",
  "claimed": "image/jpeg",
  "detected": "text/x-php"
}
```

**Why this output:** The `fileFilter` (Layer 1) checks the extension and declared MIME type — the PHP script passes because its name ends in `.jpg` and the header claims `image/jpeg`. But the magic-number check (Layer 2) reads the actual file bytes and detects `text/x-php`. The mismatch between the claimed type and the detected type triggers rejection. This is the authoritative validation layer.

### Real-World Cases

- **Profile picture uploads:** Only JPEG, PNG, and WebP allowed; magic-number validation rejects renamed executables.
- **Document management:** PDFs validated with magic numbers before passing to a PDF parser (preventing ImageTragick-style exploits).
- **E-commerce:** Product images validated to prevent XSS via SVG or HTML files disguised as images.
- **Compliance:** Meeting OWASP ASVS 5.0 V5 (File Handling) requirements.

---

## Core Concept 2: Filename Sanitisation

### Definitions

**Core Definition:** Filename sanitisation is the process of removing or neutralising dangerous characters and path components from a user-supplied filename to prevent directory traversal attacks.

**Technical Definition:** Path traversal (CWE-22) occurs when an attacker supplies a filename like `../../etc/passwd` or `..%2f..%2f` to escape the intended upload directory and overwrite or read arbitrary files. The `path.basename()` function strips all directory components from a path, returning only the final portion. However, the strongest pattern — recommended by security practitioners — is to discard the client-supplied filename entirely and generate a random, server-side identifier (UUID or cryptographic hash) for the stored file. The OWASP File Upload Cheat Sheet states: "Change the filename to something generated by the application" and "Set a filename length limit".

**Beginner-Friendly Explanation:** When someone uploads a file called `../../etc/passwd`, they are trying to make your server read or overwrite a system file outside the upload folder. `path.basename()` strips away the `../` parts, leaving just `passwd`. But the safest approach is to ignore the user's filename completely — generate your own random name like `a1b2c3d4.jpg` — so the user has no control over where the file is stored or what it is called on disk.

### Purposes

- To prevent directory traversal attacks (CWE-22) that escape the upload directory.
- To prevent overwriting existing system files.
- To eliminate filename-based injection attacks (null bytes, control characters).
- To prevent extension-based attacks where a file named `.php` is executed by the web server.

### Syntax Rules and Structure

**Using `path.basename()` (minimum protection):**
```js
const path = require('path');

// Strip directory components
const safeName = path.basename(req.file.originalname);
// '../../etc/passwd' → 'passwd'

// Then join with the trusted upload directory
const fullPath = path.join(UPLOAD_DIR, safeName);
```

**Server-generated filenames (recommended):**
```js
const crypto = require('crypto');
const path = require('path');

// Generate a random name — never use the user's filename
const randomName = crypto.randomBytes(16).toString('hex');
const ext = path.extname(req.file.originalname).toLowerCase();
const safeName = randomName + ext;  // e.g., 'a1b2c3d4e5f6.jpg'
```

| Approach | Security Level | Notes |
|----------|---------------|-------|
| Raw `originalname` | **Unsafe** | Path traversal, overwrite, execution risk. |
| `path.basename()` | Moderate | Strips traversal but still user-controlled. |
| Server-generated random name | **Strong** | User has no control over storage path. |
| Server-generated + extension allowlist | **Strongest** | Combined with type validation. |

**Rules:**
- Never use `req.file.originalname` directly as a filename on disk.
- `path.basename()` removes directory components but does not sanitise all characters; combine with random naming for full safety.
- Always use `path.join()` with a trusted base directory — never string concatenation.
- For additional safety, validate that the resolved path starts with the upload directory (`startsWith` guard).
- Set a maximum filename length (e.g., 255 characters) to prevent buffer overflow issues.

### Annotated Code Example

```js
// filename-sanitization.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');
const fs = require('fs');
const app = express();

const UPLOAD_DIR = path.join(__dirname, 'uploads');

// Secure disk storage with server-generated filenames
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, UPLOAD_DIR);
  },
  filename: (req, file, cb) => {
    // Step 1: Strip directory components from the user's filename
    const basename = path.basename(file.originalname);

    // Step 2: Extract and validate the extension
    const ext = path.extname(basename).toLowerCase();
    const allowedExts = ['.jpg', '.jpeg', '.png', '.webp'];
    const safeExt = allowedExts.includes(ext) ? ext : '.bin';

    // Step 3: Generate a random, server-side name
    const randomName = crypto.randomBytes(16).toString('hex');

    // Final filename: random hex + validated extension
    cb(null, randomName + safeExt);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }
});

app.post('/upload', upload.single('file'), (req, res) => {
  // Log the original name for reference (but never use it on disk)
  res.json({
    message: 'File uploaded securely',
    originalName: path.basename(req.file.originalname),  // Sanitised for display
    storedAs: req.file.filename,
    path: req.file.path
  });
});

app.listen(3000, () => console.log('Filename sanitization on 3000'));
```

**Expected Output (for a file named `../../etc/passwd` uploaded):**
```json
{
  "message": "File uploaded securely",
  "originalName": "passwd",
  "storedAs": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.bin",
  "path": "/app/uploads/a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.bin"
}
```

**Expected Output (for a file named `photo.jpg`):**
```json
{
  "message": "File uploaded securely",
  "originalName": "photo.jpg",
  "storedAs": "c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8.jpg",
  "path": "/app/uploads/c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8.jpg"
}
```

**Why this output:** The user's filename `../../etc/passwd` is passed through `path.basename()`, which strips all directory components and returns only `passwd`. The extension `.passwd` is not in the allowlist, so it is replaced with `.bin`. The final filename is a random 32-character hex string plus the safe extension. The file is stored in the trusted `uploads/` directory — the user has no control over the storage path.

### Real-World Cases

- **Multi-tenant SaaS:** Tenant isolation requires that filenames cannot escape the tenant's prefix (e.g., `tenant_123/`). Random filenames with tenant-prefixed keys prevent this.
- **Content management systems:** Original filenames are stored in the database for display but never used on disk.
- **Compliance:** PCI-DSS and HIPAA require that file storage paths are deterministic and not influenced by user input.

---

## Core Concept 3: Malware Scanning

### Definitions

**Core Definition:** Malware scanning is the process of inspecting uploaded files for viruses, trojans, and other malicious content before they are processed or stored, typically using an antivirus engine such as ClamAV.

**Technical Definition:** ClamAV is an open-source antivirus engine designed to detect trojans, viruses, malware, and other malicious threats. Integration with Node.js is achieved through libraries such as `clamscan`, which connects to the ClamAV daemon (`clamd`) via a UNIX socket or TCP and provides `scanStream` for scanning in-memory Buffers without writing to disk. The OWASP File Upload Cheat Sheet recommends running uploaded files through an antivirus or sandbox "to validate that it doesn't contain malicious data" and through Content Disarm & Reconstruct (CDR) for applicable types (PDF, DOCX).

**Beginner-Friendly Explanation:** Antivirus software on your server checks uploaded files for known malware signatures. ClamAV is the standard open-source choice. You install it on the server, and your Node.js code connects to it and says "scan this file." If ClamAV says "malicious," you reject the upload. The OWASP recommendation is to scan before storing anything.

### Purposes

- To prevent malicious files from being stored and potentially served to other users.
- To protect the server from file-parser exploits that malware may trigger.
- To maintain the integrity of the application and comply with security standards.
- To detect known viruses, trojans, and malware signatures.

### Syntax Rules and Structure

**Installation (Ubuntu):**
```bash
sudo apt-get install clamav clamav-daemon
sudo systemctl stop clamav-freshclam
sudo freshclam
sudo systemctl start clamav-freshclam clamav-daemon
```

**Node.js Integration (`clamscan`):**
```js
const ClamScan = require('clamscan');
const clamscan = await new ClamScan().init({
  clamdscan: { socket: '/var/run/clamav/clamd.ctl', timeout: 60000 },
  preference: 'clamdscan'
});

const { isInfected, viruses } = await clamscan.scanStream(
  Readable.from(buffer)
);
```

| Method | Use Case |
|--------|----------|
| `scanFile(path)` | Scan a file on disk. |
| `scanStream(stream)` | Scan a Buffer or stream (no disk I/O). |

**Rules:**
- Wait for the initial virus database download to finish before starting the scanner.
- Configure the ClamAV daemon's `StreamMaxLength`, `MaxFileSize`, and `MaxScanSize` before accepting uploads.
- Use `scanStream` to scan in-memory Buffers — this avoids giving the ClamAV daemon access to the application's private upload directory.
- Treat scan errors (inconclusive results) as failures — a file that could not be fully scanned must never be reported as clean.
- ClamAV scanning is a **supplement** to type validation, not a replacement.

### Annotated Code Example

```js
// clamav-scanning.js
const express = require('express');
const multer = require('multer');
const ClamScan = require('clamscan');
const { Readable } = require('node:stream');
const app = express();

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 25 * 1024 * 1024 }
});

let clamscan = null;

// Initialize ClamAV on startup
(async () => {
  clamscan = await new ClamScan().init({
    removeInfected: false,
    quarantineInfected: false,
    scanLog: null,
    debugMode: false,
    scanRecursively: true,
    preference: 'clamdscan',
    clamdscan: {
      socket: '/var/run/clamav/clamd.ctl',
      timeout: 60000,
      localFallback: false
    }
  });
  console.log('ClamAV initialized');
})();

app.post('/upload', upload.single('file'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }

  if (!clamscan) {
    return res.status(503).json({ error: 'Scanner not ready' });
  }

  try {
    // Scan the in-memory buffer without writing to disk
    const { isInfected, viruses } = await clamscan.scanStream(
      Readable.from(req.file.buffer)
    );

    if (isInfected) {
      return res.status(400).json({
        error: 'Malware detected',
        viruses,
        action: 'Upload rejected'
      });
    }

    res.json({
      message: 'File scanned and accepted',
      scanned: true,
      size: req.file.size
    });
  } catch (err) {
    // Inconclusive scan — reject for safety
    res.status(500).json({
      error: 'Scan failed — rejecting file for safety',
      detail: err.message
    });
  }
});

app.listen(3000, () => console.log('ClamAV scanning on 3000'));
```

**Expected Output (for a clean file):**
```json
{
  "message": "File scanned and accepted",
  "scanned": true,
  "size": 24576
}
```

**Expected Output (for an infected file):**
```json
{
  "error": "Malware detected",
  "viruses": ["Eicar-Test-Signature"],
  "action": "Upload rejected"
}
```

**Expected Output (for a scan error):**
```json
{
  "error": "Scan failed — rejecting file for safety",
  "detail": "ClamAV daemon unavailable"
}
```

**Why this output:** The file is scanned in memory via `scanStream` — it never touches the disk before scanning. If ClamAV detects a virus (e.g., the EICAR test signature), the upload is rejected with the virus name. If the scan fails for any reason (daemon unavailable, file exceeds scan limits), the upload is rejected for safety — a file that could not be fully scanned must never be treated as clean.

### Real-World Cases

- **Email gateways:** Scanning attachments before delivery.
- **Document portals:** Scanning PDFs and Office documents for embedded macros.
- **Cloud storage:** Scanning files before they are stored in S3 or GCS.
- **Compliance:** PCI-DSS and HIPAA require malware scanning for uploaded files.

---

## Core Concept 4: Storage Isolation

### Definitions

**Core Definition:** Storage isolation is the practice of storing uploaded files outside the web root — in a directory that is not directly served by the web server — to prevent direct execution and unauthorised access.

**Technical Definition:** The OWASP File Upload Cheat Sheet states: "Store the files on a different server. If that's not possible, store them outside of the webroot". When files are stored inside the web root, a misconfigured web server may execute files with `.php`, `.jsp`, or `.asp` extensions, or serve files that should be private. Storing files outside the web root means the application must explicitly serve them through a controlled handler (e.g., an authenticated Express route) rather than relying on the web server's static file serving. The server itself should not be able to execute uploaded files — even if an attacker manages to upload a script, it should never run.

**Beginner-Friendly Explanation:** If you store uploaded files inside your website's public folder, anyone who knows the filename can access them directly — and worse, if your server is misconfigured, a file named `malicious.php` might actually execute. Storing files outside the web root means the files are not directly accessible by URL. To serve them, your application reads the file and streams it back through a route that checks authentication and authorisation first.

### Purposes

- To prevent direct execution of uploaded scripts by the web server.
- To prevent unauthorised direct access to private files.
- To enforce access control through the application layer.
- To isolate uploaded files from the application's source code and configuration files.

### Syntax Rules and Structure

**Directory Structure (files outside web root):**
```
/var/www/myapp/               # Web root (public)
├── public/
│   ├── css/
│   ├── js/
│   └── index.html
├── src/
│   ├── routes/
│   └── server.js
└── storage/                   # Outside web root — NOT served statically
    └── uploads/
        ├── a1b2c3.jpg
        └── d4e5f6.pdf
```

**Serving files through a controlled route:**
```js
app.get('/files/:key', requireAuth, (req, res) => {
  const filePath = path.join(STORAGE_DIR, req.params.key);

  // Path traversal guard
  if (!filePath.startsWith(STORAGE_DIR)) {
    return res.status(403).json({ error: 'Invalid path' });
  }

  res.sendFile(filePath);
});
```

**Rules:**
- Never point `express.static()` at the upload directory if the upload directory is inside the web root.
- Store uploads in a directory **outside** the application root or in a dedicated `storage/` folder that is not served statically.
- Use `path.join()` with a trusted base directory and validate with `startsWith()`.
- The web server user must have read access to the storage directory, but the directory must not be executable.
- For maximum isolation, use a separate volume or cloud object storage bucket.

### Annotated Code Example

```js
// storage-isolation.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');
const fs = require('fs');
const app = express();

// Storage directory OUTSIDE the web root
const STORAGE_DIR = path.join(__dirname, '..', 'storage', 'uploads');

// Ensure storage directory exists
if (!fs.existsSync(STORAGE_DIR)) {
  fs.mkdirSync(STORAGE_DIR, { recursive: true });
}

const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, STORAGE_DIR),
  filename: (req, file, cb) => {
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname).toLowerCase();
    cb(null, randomName + ext);
  }
});

const upload = multer({ storage, limits: { fileSize: 10 * 1024 * 1024 } });

// Upload — files stored outside web root
app.post('/upload', upload.single('file'), (req, res) => {
  res.json({
    message: 'File stored outside web root',
    key: req.file.filename,
    path: req.file.path
  });
});

// Serve — controlled route with path traversal guard
app.get('/files/:key', (req, res) => {
  const safeKey = path.basename(req.params.key);
  const filePath = path.join(STORAGE_DIR, safeKey);

  // Path traversal guard
  if (!filePath.startsWith(STORAGE_DIR)) {
    return res.status(403).json({ error: 'Access denied' });
  }

  // Check file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }

  // Serve the file with correct headers
  res.sendFile(filePath);
});

app.listen(3000, () => console.log('Storage isolation on 3000'));
```

**Expected Output (for `POST /upload`):**
```json
{
  "message": "File stored outside web root",
  "key": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg",
  "path": "/var/www/myapp/../storage/uploads/a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg"
}
```

**Expected Output (for `GET /files/a1b2c3...jpg`):**
```
(serves the JPEG file with Content-Type: image/jpeg)
```

**Expected Output (for `GET /files/../../etc/passwd`):**
```json
{"error":"Access denied"}
```

**Why this output:** Files are stored in `storage/uploads/`, which is outside the web root (`/var/www/myapp/`). The web server cannot serve these files directly. The `GET /files/:key` route reads the file and streams it back, but only after `path.basename()` strips any directory components and the `startsWith` guard confirms the resolved path is inside `STORAGE_DIR`. A traversal attempt like `../../etc/passwd` is rejected.

### Real-World Cases

- **Employee document portals:** Files stored outside web root and served through authenticated routes with resource-level authorization.
- **Multi-tenant SaaS:** Tenant files stored in separate directories or buckets with tenant-prefixed keys.
- **Healthcare:** Patient records stored outside the web root and served only through audited, authenticated endpoints.
- **Compliance:** PCI-DSS requires that stored cardholder data is not directly accessible via the web server.

---

## Core Concept 5: Access Control & Privacy

### Definitions

**Core Definition:** Access control for file storage is the practice of configuring buckets and directories as private by default, granting read access only through authenticated and authorised application logic, rather than making files publicly accessible.

**Technical Definition:** In object storage (S3, GCS, Azure Blob), a **private bucket** requires authentication for all operations; a **public bucket** allows anyone who possesses the object URL to read the file. The recommended pattern is to keep buckets private and grant access through **signed URLs** (time-limited, cryptographically signed URLs) or through an authenticated proxy route in the application. The OWASP File Upload Cheat Sheet states: "In the case of public access to the files, use a handler that gets mapped to filenames inside the application (someid -> file.ext)". For truly public assets (e.g., blog images), a CDN with a public bucket is acceptable; for private files (e.g., user documents, invoices), a private bucket with signed URLs is required.

**Beginner-Friendly Explanation:** A public bucket is like a library where anyone can walk in and take any book. A private bucket is like a locked archive — you need permission to enter, and even if you have the exact shelf location, you cannot access it without authorisation. For files that should only be seen by their owner (like invoices or medical records), always use a private bucket. For files that are genuinely public (like a company logo), a public bucket or CDN is fine.

### Purposes

- To prevent unauthorised access to private files.
- To enforce resource-level authorization (only the file owner can access it).
- To comply with privacy regulations (GDPR, HIPAA, PCI-DSS).
- To enable audit logging of file access.

### Syntax Rules and Structure

| Bucket Type | Access | Use Case |
|-------------|--------|----------|
| **Public** | Anyone with the URL can read | Blog images, public assets, CDN content |
| **Private** | Requires authentication + authorization | User documents, invoices, medical records |
| **Private + Signed URLs** | Time-limited, token-authenticated access | Temporary sharing, download links |
| **Private + Proxy** | Application controls access | Full audit trail, complex authorization |

**Rules:**
- Default to **private** buckets; make specific objects public only when necessary.
- Use signed URLs for temporary access to private objects.
- Use an authenticated proxy route when you need audit logging or complex authorization logic.
- Never expose bucket credentials or access keys to the client.
- For multi-tenant applications, enforce tenant isolation at the bucket or prefix level.

### Annotated Code Example

```js
// access-control.js
const express = require('express');
const { S3Client, GetObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
const app = express();

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.S3_BUCKET; // Private bucket

// Simulated file metadata database
const files = new Map([
  ['doc-1', { key: 'user-42/invoice.pdf', ownerId: 'user-42', isPublic: false }],
  ['doc-2', { key: 'public/logo.png', ownerId: null, isPublic: true }]
]);

// Authenticated endpoint — only owner can access private files
app.get('/files/:fileId', async (req, res) => {
  const file = files.get(req.params.fileId);
  if (!file) return res.status(404).json({ error: 'File not found' });

  // Authorization: check if user owns the file (or if it is public)
  const userId = req.user?.id; // Set by auth middleware
  if (!file.isPublic && file.ownerId !== userId) {
    return res.status(403).json({ error: 'Access denied' });
  }

  // Generate a signed URL with a short expiry
  const command = new GetObjectCommand({
    Bucket: BUCKET,
    Key: file.key
  });

  const url = await getSignedUrl(s3, command, {
    expiresIn: 300  // 5 minutes
  });

  res.json({
    fileId: req.params.fileId,
    url,
    expiresIn: 300,
    note: 'This URL expires in 5 minutes'
  });
});

app.listen(3000, () => console.log('Access control on 3000'));
```

**Expected Output (for the file owner requesting `doc-1`):**
```json
{
  "fileId": "doc-1",
  "url": "https://my-bucket.s3.us-east-1.amazonaws.com/user-42/invoice.pdf?X-Amz-Signature=...",
  "expiresIn": 300,
  "note": "This URL expires in 5 minutes"
}
```

**Expected Output (for a different user requesting `doc-1`):**
```json
{"error":"Access denied"}
```

**Expected Output (for any user requesting the public `doc-2`):**
```json
{
  "fileId": "doc-2",
  "url": "https://my-bucket.s3.us-east-1.amazonaws.com/public/logo.png?X-Amz-Signature=...",
  "expiresIn": 300
}
```

**Why this output:** The bucket is private, so files cannot be accessed directly. The application checks authorization: the file owner can access `doc-1`; a different user is denied. For public files (`doc-2`), authorization is skipped. In both cases, a signed URL is generated with a 5-minute expiry — the client uses this URL to download the file directly from S3 without exposing the bucket credentials.

### Real-World Cases

- **SaaS document portals:** Private buckets with signed URLs for temporary download access.
- **E-commerce invoices:** Customers can download their own invoices via signed URLs; other customers are denied.
- **Healthcare:** Patient records stored in private buckets; access logged and time-limited.
- **CDN assets:** Public bucket behind a CDN for blog images and static assets.

---

## Core Concept 6: Signed URLs (Presigned URLs)

### Definitions

**Core Definition:** A signed URL (or presigned URL) is a URL that grants time-limited access to a specific object in a private bucket, using a cryptographic signature that proves the URL was generated by someone with valid credentials.

**Technical Definition:** Signed URLs are generated by the AWS SDK using the `getSignedUrl` function with `@aws-sdk/s3-request-presigner`. The URL contains the object key, the expiration time, and a signature computed from the request parameters and the signer's credentials. Anyone who possesses the URL can access the object until the expiration time — no additional authentication is required. The maximum expiration for SDK-generated presigned URLs is **7 days** (604,800 seconds). The S3 console limits expiration to 12 hours. Best practices include using the shortest possible expiration, restricting content type, and using temporary credentials for signing.

**Beginner-Friendly Explanation:** A signed URL is like a temporary key card that works for a specific door (a specific file) and expires after a set time (e.g., 5 minutes). You generate it on your server using your secret credentials, and give it to the user. They can use it to download the file directly from S3 without your server being involved in the download. After the time expires, the URL stops working. This is much more efficient than proxying every download through your server.

### Purposes

- To provide secure, time-limited access to private files without exposing bucket credentials.
- To offload download bandwidth from the application server to S3/CloudFront.
- To enable temporary sharing of private files (e.g., generating an invoice download link).
- To support direct uploads from the browser to S3 without proxying through the server.

### Syntax Rules and Structure

**Generating a Signed URL (Download):**
```js
const { GetObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

const command = new GetObjectCommand({
  Bucket: 'my-bucket',
  Key: 'user-42/invoice.pdf'
});

const url = await getSignedUrl(s3, command, {
  expiresIn: 300  // 5 minutes
});
```

**Generating a Signed URL (Upload):**
```js
const { PutObjectCommand } = require('@aws-sdk/client-s3');

const command = new PutObjectCommand({
  Bucket: 'my-bucket',
  Key: 'uploads/photo.jpg',
  ContentType: 'image/jpeg'
});

const url = await getSignedUrl(s3, command, {
  expiresIn: 600  // 10 minutes
});
```

| Parameter | Description |
|-----------|-------------|
| `expiresIn` | Seconds until expiry (default 900, max 604800). |
| `Bucket` | Target bucket name. |
| `Key` | Object key (path). |
| `ContentType` | Restrict upload content type. |

**Rules:**
- Use the **shortest possible expiration** — 5–15 minutes for downloads, 10–30 minutes for uploads.
- The maximum expiration for SDK-generated URLs is **7 days** (604,800 seconds).
- Signed URLs expire when the credentials used to create them are revoked, deleted, or deactivated — even if the URL's expiration is later.
- Restrict the `Content-Type` for upload URLs to prevent content-type spoofing.
- Do not log request signatures — they are bearer tokens until they expire.
- Validate MIME type and file size **before** generating presigned upload URLs.

### Annotated Code Example

```js
// signed-urls.js
const express = require('express');
const { S3Client, GetObjectCommand, PutObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
const app = express();

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.S3_BUCKET;

// Generate a signed download URL
app.get('/download/:key', async (req, res) => {
  const key = req.params.key;

  const command = new GetObjectCommand({
    Bucket: BUCKET,
    Key: key
  });

  const url = await getSignedUrl(s3, command, {
    expiresIn: 300  // 5 minutes
  });

  res.json({
    url,
    expiresAt: new Date(Date.now() + 300 * 1000).toISOString(),
    note: 'URL expires in 5 minutes'
  });
});

// Generate a signed upload URL (for direct browser-to-S3 upload)
app.post('/upload-url', async (req, res) => {
  const { filename, contentType } = req.body;

  // Validate content type before generating URL
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  if (!allowedTypes.includes(contentType)) {
    return res.status(400).json({
      error: 'Unsupported content type',
      allowed: allowedTypes
    });
  }

  const key = `uploads/${Date.now()}-${filename}`;

  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType
  });

  const url = await getSignedUrl(s3, command, {
    expiresIn: 600  // 10 minutes
  });

  res.json({
    uploadUrl: url,
    key,
    expiresIn: 600
  });
});

app.listen(3000, () => console.log('Signed URLs on 3000'));
```

**Expected Output (for `GET /download/user-42/invoice.pdf`):**
```json
{
  "url": "https://my-bucket.s3.us-east-1.amazonaws.com/user-42/invoice.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=300&X-Amz-Signature=...",
  "expiresAt": "2026-01-15T10:35:00.000Z",
  "note": "URL expires in 5 minutes"
}
```

**Expected Output (for `POST /upload-url` with `contentType: application/x-php`):**
```json
{
  "error": "Unsupported content type",
  "allowed": ["image/jpeg", "image/png", "application/pdf"]
}
```

**Why this output:** The signed URL contains the signature (`X-Amz-Signature`), the expiration (`X-Amz-Expires=300`), and the credential scope. Anyone with this URL can download the file for 5 minutes — no additional authentication is needed. For uploads, the content type is validated before the URL is generated, preventing attackers from uploading disallowed file types directly to S3.

### Real-World Cases

- **Invoice downloads:** Generating a 5-minute signed URL for a customer to download their invoice.
- **Direct browser uploads:** The browser uploads a large file directly to S3 using a presigned PUT URL, bypassing the application server.
- **Temporary sharing:** Generating a 1-hour signed URL to share a private file with a colleague.
- **CDN integration:** CloudFront signed URLs for private content distribution.

---

## References

- OWASP File Upload Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- OWASP File Upload Considerations (Kirk Jackson, 2011) — https://wiki.owasp.org/images/f/fd/OWASP_NZDay_2011_KirkJackson_FileUploadConsiderations.pdf
- OWASP ASVS 5.0 — V5 File Handling — https://github.com/OWASP/ASVS
- CWE-22: Improper Limitation of a Pathname to a Restricted Directory — https://cwe.mitre.org/data/definitions/22.html
- CWE-434: Unrestricted Upload of File with Dangerous Type — https://cwe.mitre.org/data/definitions/434.html
- Secure File Upload Handling in Node.js and Fastify — https://safeguard.sh/resources/blog/secure-file-upload-handling-nodejs-fastify
- Implementing server-side malware scanning with ClamAV in Node.js — https://transloadit.com/devtips/implementing-server-side-malware-scanning-with-clamav-in-node-js/
- ClamAV Scanning Documentation — https://docs.clamav.net/manual/Usage/Scanning.html
- pompelmi — ClamAV antivirus scanning for Node.js — https://www.npmjs.com/package/pompelmi
- @competentgroove/secure-upload — https://www.npmjs.com/package/@competentgroove/secure-upload
- file-type — npm Documentation — https://www.npmjs.com/package/file-type
- Secure API file uploads with magic numbers — https://assets.transloadit.com/devtips/secure-api-file-uploads-with-magic-numbers/
- Building a Secure Employee Document Portal with Node.js and Express — https://www.sitepoint.com/building-a-secure-employee-document-portal-with-node-js-and-express/
- Amazon S3 examples using SDK for JavaScript (v3) — https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_s3_code_examples.html
- AWS SDK for JavaScript v3 — S3 Presigned URLs — https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_s3_code_examples.html
- Establishing guardrails and monitoring for presigned URLs — https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html
- S3 Presigned URL expiration limits — https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html
- Supabase Storage Buckets — Public vs Private — https://supabase.com/docs/guides/storage/buckets/fundamentals
- eslint-plugin-secure-coding — no-arbitrary-file-access — https://github.com/ofri-peretz/eslint/blob/main/packages/eslint-plugin-secure-coding/docs/rules/no-arbitrary-file-access.md