# File Storage & Processing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** File storage and processing is the practice of persisting uploaded files — either on the local filesystem or in cloud object storage — and then transforming them (resizing, compressing, converting formats) and managing their metadata in a database.

**Technical Definition:** File storage encompasses the architectural decision of where uploaded bytes live: local disk (simple, single-server), object storage (S3, GCS, Azure Blob — scalable, distributed), or a hybrid pattern where files live in object storage and metadata lives in a SQL/NoSQL database. File processing uses libraries like Sharp (native libvips bindings, ~40–50× faster) or Jimp (pure JavaScript, no native dependencies) to resize, compress, and convert images to modern formats like WebP and AVIF. Metadata management separates the file's binary content from its descriptive attributes (original name, MIME type, size, checksum, dimensions), enabling fast queries and CDN serving.

**Beginner-Friendly Explanation:** When someone uploads a file, you have three decisions to make. First: where do you put the actual file — on your server's disk or in the cloud (S3, Google Cloud, Azure)? Second: what do you do with it — resize it, compress it, convert it to a modern format? Third: how do you remember what it is — store its name, size, and type in a database so you can find it later. This cheat sheet covers all three decisions with production-ready code.

### Key Characteristics

- **Storage tiering:** Local storage for development and small apps; object storage for production scale.
- **SDK abstraction:** `@aws-sdk/client-s3`, `@google-cloud/storage`, and `@azure/storage-blob` provide consistent APIs for upload, download, and listing.
- **Streaming uploads:** `multer-s3` streams multipart uploads directly to S3 without touching the local filesystem.
- **Metadata separation:** Files in object storage; metadata in MongoDB/PostgreSQL for indexed queries.
- **On-the-fly processing:** Sharp resizes and converts images in memory before storage; Jimp offers pure-JS portability.
- **Security-critical:** Local storage requires careful `express.static()` configuration; object storage requires IAM policies and signed URLs.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express multer`.
- **Cloud SDKs installed** as needed: `@aws-sdk/client-s3`, `@google-cloud/storage`, `@azure/storage-blob`.
- **Image processing libraries:** `sharp` or `jimp`.
- **A database** (MongoDB, PostgreSQL, or SQLite) for metadata.
- **Cloud credentials** configured for S3/GCS/Azure.

### Related Programming Areas

- **File Uploads (Multer):** The middleware that parses multipart requests before storage.
- **CDN integration:** Serving processed images via CloudFront, Cloud CDN, or Azure CDN.
- **Image optimisation:** WebP/AVIF conversion for modern browser performance.
- **Backup and versioning:** Object storage versioning and lifecycle policies.
- **Security:** Signed URLs, IAM policies, and content validation.

### Core Concepts

1. **Local Storage** — Managing the public or uploads directory.
2. **Object Storage Integration** — Amazon S3, Google Cloud Storage, Azure Blob.
3. **Using SDKs** — `@aws-sdk/client-s3` and `multer-s3`.
4. **File Metadata Management** — Storing paths, sizes, and original names in databases.
5. **On-the-Fly Image Processing** — Resizing, compressing, and converting to WebP.

---

## Core Concept 1: Local Storage

### Definitions

**Core Definition:** Local storage means writing uploaded files directly to the server's filesystem, typically in a dedicated `uploads/` or `public/` directory, and serving them via Express's static middleware.

**Technical Definition:** Local storage uses Multer's `diskStorage` engine to write files to a directory on the server. The directory must exist before the upload, have appropriate permissions, and be served statically via `express.static()` with security options such as `dotfiles: 'deny'`. The OWASP and Node.js security communities warn that pointing `express.static()` at the application root exposes source code, configuration files, and credentials; the static root must be a dedicated directory containing only intended public assets.

**Beginner-Friendly Explanation:** Local storage is the simplest way to handle files — just save them to a folder on your server and let Express serve them. But you have to be careful: only serve a dedicated `uploads/` folder, never your entire project directory, and always deny access to hidden files (dotfiles) like `.env` or `.git`.

### Purposes

- To provide the simplest possible file storage for development and small applications.
- To serve uploaded files statically via Express with automatic MIME type detection and caching headers.
- To avoid external dependencies and cloud costs during development.
- To enable fast local file access without network latency.

### Syntax Rules and Structure

**Directory Structure:**
```
project/
├── uploads/            # Dedicated upload directory
│   ├── avatars/
│   └── documents/
├── public/             # Static assets (CSS, JS, images)
├── src/
│   ├── routes/
│   └── middleware/
└── server.js
```

**Multer Disk Storage:**
```js
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const safeName = crypto.randomBytes(16).toString('hex') +
                     path.extname(file.originalname).toLowerCase();
    cb(null, safeName);
  }
});
```

**Static Serving:**
```js
app.use('/uploads', express.static(path.join(__dirname, 'uploads'), {
  dotfiles: 'deny',           // Block .env, .git, etc.
  index: false,               // No directory listing
  setHeaders: (res) => {
    res.set('X-Content-Type-Options', 'nosniff');
  }
}));
```

| Option | Purpose |
|--------|---------|
| `dotfiles: 'deny'` | Block access to `.env`, `.git`, etc. |
| `index: false` | Prevent directory listing. |
| `maxAge` | Set cache headers (e.g., `'1d'`). |
| `immutable` | Enable immutable caching for hashed filenames. |

**Rules:**
- Never use `express.static('.')` or `express.static(__dirname)` — this exposes the entire project.
- The upload directory must exist before Multer writes to it; create it at startup.
- Always generate server-side filenames; never use `file.originalname` directly (path traversal risk).
- Deny dotfiles and disable directory listings.
- Set proper file permissions (not `777`).
- Local storage does not scale across multiple servers — files uploaded to one server are not available on others.

### Annotated Code Example

```js
// local-storage.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');
const fs = require('fs');
const app = express();

// Ensure upload directory exists at startup
const UPLOAD_DIR = path.join(__dirname, 'uploads');
if (!fs.existsSync(UPLOAD_DIR)) {
  fs.mkdirSync(UPLOAD_DIR, { recursive: true });
}

// Disk storage with safe random filenames
const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, UPLOAD_DIR),
  filename: (req, file, cb) => {
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname).toLowerCase();
    cb(null, randomName + ext);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/webp'];
    cb(null, allowed.includes(file.mimetype));
  }
});

// Serve uploads statically with security options
app.use('/uploads', express.static(UPLOAD_DIR, {
  dotfiles: 'deny',
  index: false,
  maxAge: '1d',
  setHeaders: (res) => {
    res.set('X-Content-Type-Options', 'nosniff');
  }
}));

app.post('/upload', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No valid file uploaded' });
  }
  res.json({
    message: 'File stored locally',
    url: `/uploads/${req.file.filename}`,
    size: req.file.size
  });
});

app.listen(3000, () => console.log('Local storage on 3000'));
```

**Expected Output (for `POST /upload` with a valid PNG):**
```json
{
  "message": "File stored locally",
  "url": "/uploads/a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.png",
  "size": 24576
}
```

**Expected Output (for `GET /uploads/a1b2c3...png`):**
```
(serves the PNG file with Content-Type: image/png)
```

**Why this output:** Multer writes the file to `uploads/` with a cryptographically random name, preventing path traversal and collisions. Express serves the file statically with `dotfiles: 'deny'` and `index: false`, blocking access to `.env` and other sensitive files. The URL returned to the client is `/uploads/<random-name>`, which Express resolves to the file on disk.

### Real-World Cases

- **Development environments:** Fast iteration without cloud credentials.
- **Small apps and prototypes:** Single-server deployments where files fit on disk.
- **Legacy migrations:** Storing files locally before migrating to object storage.
- **Temporary processing:** Saving uploaded files locally for processing, then deleting them.

---

## Core Concept 2: Object Storage Integration

### Definitions

**Core Definition:** Object storage is a cloud service (Amazon S3, Google Cloud Storage, Azure Blob) that stores files as objects in buckets, providing virtually unlimited scalability, durability, and CDN integration.

**Technical Definition:** Object storage stores data as objects within buckets. Each object has a key (unique identifier), data (the file bytes), and metadata. Amazon S3, Google Cloud Storage (GCS), and Azure Blob Storage are the three dominant providers. Each provides a Node.js SDK: `@aws-sdk/client-s3`, `@google-cloud/storage`, and `@azure/storage-blob`. Objects are immutable — updating a file means uploading a new version. Access control is managed via IAM policies (AWS/GCP) or SAS tokens/role assignments (Azure).

**Beginner-Friendly Explanation:** Object storage is like a giant, infinitely large filing cabinet in the cloud. You put files into "buckets" (folders), and each file gets a unique key. Unlike local storage, you never worry about disk space, and multiple servers can access the same files. Amazon S3, Google Cloud Storage, and Azure Blob are the three main providers — they all work similarly, just with different SDKs.

### Purposes

- To store files at scale without worrying about server disk space.
- To enable multiple servers to access the same files (horizontal scaling).
- To integrate with CDNs for fast global delivery.
- To provide durability guarantees (S3: 99.999999999% — 11 nines).
- To support versioning, lifecycle policies, and cross-region replication.

### Sub-Feature 2.1: Amazon S3 Integration

#### Syntax Rules and Structure

```js
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { Upload } = require('@aws-sdk/lib-storage');

const s3 = new S3Client({ region: 'us-east-1' });

// Simple upload (small files)
await s3.send(new PutObjectCommand({
  Bucket: 'my-bucket',
  Key: 'uploads/file.jpg',
  Body: buffer,
  ContentType: 'image/jpeg'
}));

// Multipart upload (large files, >5MB)
const upload = new Upload({
  client: s3,
  params: {
    Bucket: 'my-bucket',
    Key: 'uploads/large-file.mp4',
    Body: buffer
  }
});
await upload.done();
```

| Command | Use Case |
|---------|----------|
| `PutObjectCommand` | Small files (<5MB). |
| `Upload` (lib-storage) | Large files (multipart, automatic). |
| `GetObjectCommand` | Download a file. |
| `DeleteObjectCommand` | Delete a file. |

#### Annotated Code Example

```js
// s3-integration.js
const express = require('express');
const multer = require('multer');
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
const crypto = require('crypto');
const app = express();

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.S3_BUCKET;

// Memory storage — file is buffered, then streamed to S3
const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 10 * 1024 * 1024 }
});

app.post('/upload', upload.single('file'), async (req, res) => {
  const file = req.file;
  const key = `uploads/${crypto.randomUUID()}-${file.originalname}`;

  try {
    await s3.send(new PutObjectCommand({
      Bucket: BUCKET,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
      CacheControl: 'public, max-age=31536000'
    }));

    // Generate a signed URL for access (private bucket)
    const url = await getSignedUrl(s3, new GetObjectCommand({
      Bucket: BUCKET,
      Key: key
    }), { expiresIn: 3600 });

    res.json({ message: 'Uploaded to S3', key, url });
  } catch (err) {
    res.status(500).json({ error: 'S3 upload failed', detail: err.message });
  }
});

app.listen(3000, () => console.log('S3 upload on 3000'));
```

**Expected Output:**
```json
{
  "message": "Uploaded to S3",
  "key": "uploads/a1b2c3d4-e5f6-...-photo.jpg",
  "url": "https://my-bucket.s3.us-east-1.amazonaws.com/uploads/...?X-Amz-Signature=..."
}
```

**Why this output:** The file is buffered in memory (Multer memory storage), then sent to S3 via `PutObjectCommand`. The bucket is private, so a signed URL is generated with a 1-hour expiry. The client receives the key (for database storage) and a temporary URL for immediate access.

### Sub-Feature 2.2: Google Cloud Storage Integration

#### Syntax Rules and Structure

```js
const { Storage } = require('@google-cloud/storage');
const storage = new Storage();

const bucket = storage.bucket('my-bucket');
await bucket.file('uploads/file.jpg').save(buffer, {
  contentType: 'image/jpeg',
  metadata: { cacheControl: 'public, max-age=31536000' }
});
```

#### Annotated Code Example

```js
// gcs-integration.js
const { Storage } = require('@google-cloud/storage');
const storage = new Storage();

async function uploadToGCS(bucketName, destination, buffer, mimeType) {
  const bucket = storage.bucket(bucketName);
  const file = bucket.file(destination);

  await file.save(buffer, {
    contentType: mimeType,
    metadata: {
      cacheControl: 'public, max-age=31536000',
      metadata: { uploadedBy: 'api' }
    },
    resumable: false  // For small files; true for large files
  });

  return {
    key: destination,
    publicUrl: `https://storage.googleapis.com/${bucketName}/${destination}`
  };
}

uploadToGCS('my-bucket', 'uploads/photo.jpg', imageBuffer, 'image/jpeg')
  .then(result => console.log('Uploaded:', result.publicUrl));
```

**Expected Output:**
```
Uploaded: https://storage.googleapis.com/my-bucket/uploads/photo.jpg
```

**Why this output:** The GCS SDK's `file.save()` method uploads the buffer to the specified bucket and key. The `resumable: false` option is appropriate for small files; for files over 5MB, use `resumable: true` for chunked uploads with automatic retry. The public URL is constructed from the bucket name and object key.

### Sub-Feature 2.3: Azure Blob Storage Integration

#### Syntax Rules and Structure

```js
const { BlobServiceClient } = require('@azure/storage-blob');

const blobServiceClient = BlobServiceClient.fromConnectionString(connectionString);
const containerClient = blobServiceClient.getContainerClient('uploads');
const blockBlobClient = containerClient.getBlockBlobClient('photo.jpg');

await blockBlobClient.uploadData(buffer, {
  blobHTTPHeaders: { blobContentType: 'image/jpeg' }
});
```

#### Annotated Code Example

```js
// azure-blob.js
const { BlobServiceClient } = require('@azure/storage-blob');

const connectionString = process.env.AZURE_STORAGE_CONNECTION_STRING;
const blobServiceClient = BlobServiceClient.fromConnectionString(connectionString);

async function uploadToAzure(containerName, blobName, buffer, mimeType) {
  const containerClient = blobServiceClient.getContainerClient(containerName);
  const blockBlobClient = containerClient.getBlockBlobClient(blobName);

  await blockBlobClient.uploadData(buffer, {
    blobHTTPHeaders: { blobContentType: mimeType },
    metadata: { uploadedBy: 'api' }
  });

  return {
    key: blobName,
    url: blockBlobClient.url
  };
}

uploadToAzure('uploads', 'photo.jpg', imageBuffer, 'image/jpeg')
  .then(result => console.log('Uploaded:', result.url));
```

**Expected Output:**
```
Uploaded: https://myaccount.blob.core.windows.net/uploads/photo.jpg
```

**Why this output:** The Azure SDK uses a container client and block blob client to upload the buffer. `uploadData()` is suitable for small files; `uploadStream()` is used for large files or streams. The blob's URL is automatically constructed from the account, container, and blob name.

---

## Core Concept 3: Using SDKs

### Definitions

**Core Definition:** `multer-s3` is a Multer storage engine that streams multipart uploads directly to Amazon S3, bypassing the local filesystem entirely.

**Technical Definition:** `multer-s3` integrates with Multer's storage engine interface, using the `Upload` class from `@aws-sdk/lib-storage` (AWS SDK v3) to stream file data directly to S3 as it arrives from the client. This avoids buffering the entire file in memory or writing it to disk, making it ideal for large files and high-concurrency uploads. Each uploaded file's `req.file` object includes S3-specific properties: `key`, `bucket`, `location`, `etag`, and `contentType`.

**Beginner-Friendly Explanation:** Normally, Multer saves files to disk, and then you upload them to S3 in a separate step. `multer-s3` skips the middleman — it streams the file directly from the client to S3 as it arrives. This is faster, uses less memory, and avoids disk space issues.

### Purposes

- To stream files directly to S3 without local disk or memory buffering.
- To reduce upload latency and memory usage.
- To simplify code by combining parsing and storage in one middleware.
- To handle large files (videos, large documents) efficiently.

### Syntax Rules and Structure

```js
const { S3Client } = require('@aws-sdk/client-s3');
const multer = require('multer');
const multerS3 = require('multer-s3');

const s3 = new S3Client({ region: 'us-east-1' });

const upload = multer({
  storage: multerS3({
    s3,
    bucket: 'my-bucket',
    contentType: multerS3.AUTO_CONTENT_TYPE,
    key: (req, file, cb) => {
      cb(null, `uploads/${Date.now()}-${file.originalname}`);
    },
    metadata: (req, file, cb) => {
      cb(null, { fieldName: file.fieldname });
    }
  })
});
```

| Option | Description |
|--------|-------------|
| `s3` | The S3Client instance. |
| `bucket` | Target bucket name. |
| `contentType` | `multerS3.AUTO_CONTENT_TYPE` uses the file's MIME type. |
| `key` | Function to generate the S3 object key. |
| `metadata` | Function to set S3 metadata. |
| `acl` | Canned ACL (e.g., `'public-read'`). |

**Rules:**
- `multer-s3` v3.x uses AWS SDK v3 (`@aws-sdk/client-s3` and `@aws-sdk/lib-storage`).
- `multer-s3` v2.x uses AWS SDK v2 (`aws-sdk`) — deprecated.
- The `key` function must generate unique keys to avoid overwrites.
- Files are streamed directly to S3; `req.file.buffer` is not available.
- The `location` property contains the S3 URL of the uploaded file.

### Annotated Code Example

```js
// multer-s3.js
const express = require('express');
const multer = require('multer');
const multerS3 = require('multer-s3');
const { S3Client } = require('@aws-sdk/client-s3');
const crypto = require('crypto');
const app = express();

const s3 = new S3Client({ region: process.env.AWS_REGION });

const upload = multer({
  storage: multerS3({
    s3,
    bucket: process.env.S3_BUCKET,
    contentType: multerS3.AUTO_CONTENT_TYPE,
    key: (req, file, cb) => {
      const randomName = crypto.randomBytes(16).toString('hex');
      const ext = file.originalname.split('.').pop();
      cb(null, `uploads/${randomName}.${ext}`);
    },
    metadata: (req, file, cb) => {
      cb(null, { originalName: file.originalname });
    }
  }),
  limits: { fileSize: 50 * 1024 * 1024 }
});

app.post('/upload', upload.array('photos', 5), (req, res) => {
  res.json({
    message: `Uploaded ${req.files.length} files to S3`,
    files: req.files.map(f => ({
      key: f.key,
      location: f.location,
      size: f.size,
      etag: f.etag
    }))
  });
});

app.listen(3000, () => console.log('multer-s3 on 3000'));
```

**Expected Output:**
```json
{
  "message": "Uploaded 2 files to S3",
  "files": [
    {
      "key": "uploads/a1b2c3d4e5f6a7b8.jpg",
      "location": "https://my-bucket.s3.us-east-1.amazonaws.com/uploads/a1b2c3d4e5f6a7b8.jpg",
      "size": 24576,
      "etag": "\"d41d8cd98f00b204e9800998ecf8427e\""
    },
    {
      "key": "uploads/c3d4e5f6a7b8c9d0.png",
      "location": "https://my-bucket.s3.us-east-1.amazonaws.com/uploads/c3d4e5f6a7b8c9d0.png",
      "size": 40960,
      "etag": "\"098f6bcd4621d373cade4e832627b4f6\""
    }
  ]
}
```

**Why this output:** `multer-s3` streams each file directly to S3 as it is parsed from the multipart request. The `key` function generates a unique S3 object key using a random hex string. The `location` property contains the full S3 URL. The `etag` is the MD5 hash of the uploaded object, which can be used for integrity verification or as a checksum in the database.

### Real-World Cases

- **Video platforms:** Streaming large video uploads directly to S3 for transcoding.
- **High-traffic APIs:** Avoiding disk I/O bottlenecks by streaming to S3.
- **Serverless deployments:** AWS Lambda has no persistent disk; `multer-s3` writes directly to S3.
- **Multi-tenant SaaS:** Each tenant's files stored under a unique S3 prefix.

---

## Core Concept 4: File Metadata Management

### Definitions

**Core Definition:** File metadata management is the practice of storing descriptive attributes about uploaded files — original name, MIME type, size, checksum, storage key, dimensions — in a SQL or NoSQL database, separate from the file's binary content.

**Technical Definition:** The preferred pattern is to store files in object storage (S3, GCS, Azure) and store metadata in MongoDB or PostgreSQL. This separates concerns: object storage handles binary data at scale, and the database handles indexed queries over metadata (search by tag, filter by MIME type, sort by upload date). A typical schema includes: `key` (storage object key), `bucket`, `originalName`, `mimeType`, `size`, `checksum` (MD5/SHA-256), `width`/`height` (for images), `folder`/`tags`, and `uploadedAt`.

**Beginner-Friendly Explanation:** When you upload a file to S3, you get back a key (like `uploads/abc123.jpg`). You then save that key along with the file's original name, size, type, and upload date into your database. Later, when someone wants to find a file, you query the database — not S3 — because databases are much faster for searching and filtering. The database tells you the S3 key, and you use that to retrieve the file.

### Purposes

- To store file paths, sizes, and original names in SQL/NoSQL databases.
- To enable fast, indexed queries over file metadata (search, filter, sort).
- To track ownership, usage, and lifecycle of files.
- To maintain a checksum for integrity verification.
- To decouple file storage from application logic — the database is the source of truth for "what files exist."

### Syntax Rules and Structure

**MongoDB Schema (Mongoose):**
```js
const AssetSchema = new mongoose.Schema({
  key: { type: String, required: true, unique: true },
  bucket: { type: String, required: true },
  cdnUrl: { type: String, required: true },
  storageProvider: { type: String, enum: ['s3', 'gcs', 'azure'], default: 's3' },
  originalName: String,
  mimeType: { type: String, required: true, index: true },
  size: { type: Number, required: true },
  checksum: String,
  width: Number,
  height: Number,
  format: String,
  folder: { type: String, default: '/', index: true },
  tags: { type: [String], index: true },
  uploadedBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  uploadedAt: { type: Date, default: Date.now, index: true }
});
```

| Field | Purpose |
|-------|---------|
| `key` | S3/GCS/Azure object key (unique). |
| `bucket` | Storage bucket name. |
| `cdnUrl` | Public CDN URL for the file. |
| `originalName` | User-provided filename. |
| `mimeType` | File MIME type (indexed for filtering). |
| `size` | File size in bytes. |
| `checksum` | MD5 or SHA-256 for integrity. |
| `width` / `height` | Image dimensions (for images). |
| `tags` | Searchable tags (indexed). |

**Rules:**
- Always store the storage `key` and `bucket` — these are required to retrieve the file.
- Index frequently queried fields (`mimeType`, `uploadedAt`, `tags`).
- Store `originalName` for display, but never use it as the storage key.
- Store `checksum` for integrity verification and deduplication.
- For images, store `width`, `height`, and `format` to avoid re-fetching metadata.
- Use a `variants` array for generated thumbnails (different sizes/formats).

### Annotated Code Example

```js
// metadata-management.js
const express = require('express');
const multer = require('multer');
const mongoose = require('mongoose');
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const crypto = require('crypto');
const sharp = require('sharp');
const app = express();

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.S3_BUCKET;

// MongoDB schema
const AssetSchema = new mongoose.Schema({
  key: { type: String, required: true, unique: true },
  bucket: String,
  cdnUrl: String,
  originalName: String,
  mimeType: { type: String, index: true },
  size: Number,
  checksum: String,
  width: Number,
  height: Number,
  format: String,
  tags: [String],
  uploadedAt: { type: Date, default: Date.now, index: true }
});
const Asset = mongoose.model('Asset', AssetSchema);

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 10 * 1024 * 1024 }
});

app.post('/upload', upload.single('file'), async (req, res) => {
  const file = req.file;
  const key = `uploads/${crypto.randomUUID()}-${file.originalname}`;

  // Upload to S3
  await s3.send(new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    Body: file.buffer,
    ContentType: file.mimetype
  }));

  // Extract image metadata (if image)
  let dimensions = {};
  if (file.mimetype.startsWith('image/')) {
    const metadata = await sharp(file.buffer).metadata();
    dimensions = {
      width: metadata.width,
      height: metadata.height,
      format: metadata.format
    };
  }

  // Compute checksum
  const checksum = crypto.createHash('sha256').update(file.buffer).digest('hex');

  // Store metadata in MongoDB
  const asset = await Asset.create({
    key,
    bucket: BUCKET,
    cdnUrl: `https://cdn.example.com/${key}`,
    originalName: file.originalname,
    mimeType: file.mimetype,
    size: file.size,
    checksum,
    ...dimensions,
    tags: req.body.tags ? req.body.tags.split(',') : []
  });

  res.status(201).json({
    message: 'File uploaded and metadata stored',
    asset: {
      id: asset._id,
      key: asset.key,
      originalName: asset.originalName,
      size: asset.size,
      width: asset.width,
      height: asset.height,
      checksum: asset.checksum
    }
  });
});

// Query files by MIME type
app.get('/files', async (req, res) => {
  const filter = {};
  if (req.query.mimeType) filter.mimeType = req.query.mimeType;
  if (req.query.tag) filter.tags = req.query.tag;

  const files = await Asset.find(filter)
    .sort({ uploadedAt: -1 })
    .limit(50);

  res.json(files);
});

app.listen(3000, () => console.log('Metadata management on 3000'));
```

**Expected Output (for `POST /upload` with an image):**
```json
{
  "message": "File uploaded and metadata stored",
  "asset": {
    "id": "65a1b2c3d4e5f6a7b8c9d0e1",
    "key": "uploads/a1b2c3d4-...-photo.jpg",
    "originalName": "photo.jpg",
    "size": 24576,
    "width": 1920,
    "height": 1080,
    "checksum": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  }
}
```

**Expected Output (for `GET /files?mimeType=image/jpeg`):**
```json
[
  {
    "key": "uploads/a1b2c3d4-...-photo.jpg",
    "originalName": "photo.jpg",
    "mimeType": "image/jpeg",
    "size": 24576,
    "width": 1920,
    "height": 1080,
    "uploadedAt": "2026-01-15T10:30:00.000Z"
  }
]
```

**Why this output:** The file is uploaded to S3, then Sharp extracts image dimensions from the buffer. A SHA-256 checksum is computed for integrity. The metadata is stored in MongoDB with indexed fields (`mimeType`, `uploadedAt`) for fast queries. The `GET /files` endpoint demonstrates querying by MIME type, returning only metadata — not the file itself — which is fast and efficient.

### Real-World Cases

- **Content management systems:** Assets stored in S3; metadata in MongoDB for search.
- **E-commerce:** Product images with dimensions, alt text, and tags.
- **Document management:** Files indexed by MIME type, owner, and upload date.
- **Deduplication:** SHA-256 checksums detect duplicate uploads before storing.

---

## Core Concept 5: On-the-Fly Image Processing

### Definitions

**Core Definition:** On-the-fly image processing uses libraries like Sharp or Jimp to resize, compress, and convert images to modern formats (WebP, AVIF) during or immediately after upload.

**Technical Definition:** **Sharp** is a high-performance Node.js image processing library built on libvips, supporting JPEG, PNG, WebP, AVIF, TIFF, and GIF with ~40–50× faster resize/compress operations than Jimp. It uses native bindings and streams for memory-efficient processing. **Jimp** is a pure JavaScript image processing library with no native dependencies, supporting JPEG, PNG, BMP, TIFF, and GIF — but it does not support WebP or AVIF in core. Sharp is the recommended choice for production; Jimp is suitable for environments where native binaries cannot be installed.

**Beginner-Friendly Explanation:** When someone uploads a 5MB photo, you don't want to serve that full-size image to every visitor. You want to resize it to a reasonable size, compress it, and convert it to WebP — a modern format that's 25–35% smaller than JPEG. Sharp does this extremely fast (it can process hundreds of images per second), while Jimp is slower but works anywhere JavaScript runs.

### Purposes

- To resize, compress, and convert images to WebP using Sharp or Jimp.
- To generate multiple variants (thumbnail, medium, large) from a single upload.
- To reduce bandwidth and improve page load times.
- To convert legacy formats (JPEG, PNG) to modern formats (WebP, AVIF) for better compression.
- To strip metadata (EXIF) for privacy.

### Sub-Feature 5.1: Sharp (Recommended)

#### Syntax Rules and Structure

```js
const sharp = require('sharp');

// Resize and convert to WebP
await sharp('input.jpg')
  .resize(800, 600, { fit: 'cover' })
  .webp({ quality: 80 })
  .toFile('output.webp');

// Generate multiple sizes
const sizes = [
  { name: 'thumb', width: 150 },
  { name: 'medium', width: 800 },
  { name: 'large', width: 1920 }
];
for (const size of sizes) {
  await sharp('input.jpg')
    .resize(size.width, null, { withoutEnlargement: true })
    .webp({ quality: 80 })
    .toFile(`output-${size.name}.webp`);
}
```

| Method | Purpose |
|--------|---------|
| `.resize(w, h, opts)` | Resize; `fit: 'cover'` crops to fill. |
| `.webp({ quality })` | Convert to WebP with quality (1–100). |
| `.avif({ quality })` | Convert to AVIF with quality. |
| `.jpeg({ quality, mozjpeg })` | Convert to JPEG with mozjpeg. |
| `.png({ compressionLevel })` | Convert to PNG (0–9). |
| `.toBuffer()` | Return a Buffer instead of writing to file. |
| `.metadata()` | Get width, height, format, etc. |

**Rules:**
- Sharp processes in the libvips thread pool, not the Node.js event loop — it does not block.
- Use `withoutEnlargement: true` to avoid upscaling small images.
- `.toBuffer()` is useful for uploading to S3 without writing to disk.
- Sharp supports streaming for very large images.
- The `fit` option controls how the image is resized: `'cover'` (crop to fill), `'contain'` (letterbox), `'fill'` (stretch).

#### Annotated Code Example

```js
// sharp-processing.js
const express = require('express');
const multer = require('multer');
const sharp = require('sharp');
const crypto = require('crypto');
const app = express();

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 10 * 1024 * 1024 }
});

app.post('/upload', upload.single('image'), async (req, res) => {
  const file = req.file;

  try {
    // Get original metadata
    const metadata = await sharp(file.buffer).metadata();

    // Generate three variants
    const variants = await Promise.all([
      sharp(file.buffer)
        .resize(150, null, { withoutEnlargement: true })
        .webp({ quality: 80 })
        .toBuffer(),
      sharp(file.buffer)
        .resize(800, null, { withoutEnlargement: true })
        .webp({ quality: 80 })
        .toBuffer(),
      sharp(file.buffer)
        .resize(1920, null, { withoutEnlargement: true })
        .webp({ quality: 80 })
        .toBuffer()
    ]);

    const names = ['thumb', 'medium', 'large'];

    res.json({
      original: {
        width: metadata.width,
        height: metadata.height,
        format: metadata.format,
        size: file.size
      },
      variants: variants.map((buf, i) => ({
        name: names[i],
        size: buf.length,
        savings: `${Math.round((1 - buf.length / file.size) * 100)}%`
      }))
    });
  } catch (err) {
    res.status(500).json({ error: 'Image processing failed', detail: err.message });
  }
});

app.listen(3000, () => console.log('Sharp processing on 3000'));
```

**Expected Output:**
```json
{
  "original": {
    "width": 4000,
    "height": 3000,
    "format": "jpeg",
    "size": 5242880
  },
  "variants": [
    { "name": "thumb", "size": 4096, "savings": "100%" },
    { "name": "medium", "size": 32768, "savings": "99%" },
    { "name": "large", "size": 131072, "savings": "98%" }
  ]
}
```

**Why this output:** The original 5MB JPEG is processed three times in parallel. Each variant is resized to a maximum width and converted to WebP. The `withoutEnlargement: true` option prevents upscaling. The WebP variants are dramatically smaller than the original — the thumbnail is 4KB, the medium is 32KB, and the large is 128KB. This reduces bandwidth by 98–100% compared to serving the original.

### Sub-Feature 5.2: Jimp (Pure JavaScript Alternative)

#### Syntax Rules and Structure

```js
const Jimp = require('jimp');

// Read, resize, convert to WebP
const image = await Jimp.read('input.jpg');
await image
  .resize(800, Jimp.AUTO)
  .quality(80)
  .writeAsync('output.webp');
```

| Method | Purpose |
|--------|---------|
| `Jimp.read(path)` | Load an image. |
| `.resize(w, h)` | Resize; `Jimp.AUTO` maintains aspect ratio. |
| `.quality(n)` | Set quality for JPEG/WebP. |
| `.writeAsync(path)` | Write to file (async). |
| `.getBufferAsync(mime)` | Return a Buffer. |

**Rules:**
- Jimp is **pure JavaScript** — no native dependencies, works anywhere Node.js runs.
- Jimp does **not** support WebP or AVIF in core; you need plugins like `@jimp/wasm-webp` for WebP.
- Jimp is **40–50× slower** than Sharp for resize/compress operations.
- Use Jimp when native binaries cannot be installed (e.g., some serverless environments).

#### Annotated Code Example

```js
// jimp-processing.js
const Jimp = require('jimp');

async function processWithJimp(inputPath) {
  const image = await Jimp.read(inputPath);

  // Resize and convert to JPEG
  await image
    .resize(800, Jimp.AUTO)
    .quality(80)
    .writeAsync('output.jpg');

  // Get dimensions
  const width = image.bitmap.width;
  const height = image.bitmap.height;

  return { width, height };
}

processWithJimp('input.png')
  .then(info => console.log('Processed:', info))
  .catch(err => console.error('Error:', err.message));
```

**Expected Output:**
```
Processed: { width: 800, height: 600 }
```

**Why this output:** Jimp reads the image, resizes it to 800px wide (maintaining aspect ratio with `Jimp.AUTO`), sets quality to 80, and writes it to disk. The result is a smaller, web-friendly image. Jimp is slower than Sharp but requires no native compilation.

### Real-World Cases

- **Social media platforms:** Generating thumbnails, medium, and large variants from user uploads.
- **E-commerce:** Converting product images to WebP for faster page loads.
- **CMS:** Automatically resizing images on upload to fit layout constraints.
- **CDN optimisation:** Generating multiple formats for different browsers (WebP for modern, JPEG for legacy).

---

## References

- Express.js Static Files — https://expressjs.com/en/starter/static-files.html
- Storing Uploaded Files and Serving Them in Express — https://webdev101.hashnode.dev/storing-uploaded-files-and-serving-them-in-express
- Express.js Security Best Practices — https://expressjs.com/en/advanced/best-practice-security.html
- multer-s3 — npm — https://www.npmjs.com/package/multer-s3
- multer-s3 GitHub — https://github.com/expressjs/multer-s3
- @aws-sdk/client-s3 Documentation — https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/s3/
- AWS SDK for JavaScript v3 — S3 Examples — https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/javascriptv3/example_code/s3
- Google Cloud Storage Node.js Client — https://googleapis.dev/nodejs/storage/latest/
- @google-cloud/storage — npm — https://www.npmjs.com/package/@google-cloud/storage
- Azure Blob Storage Node.js Quickstart — https://learn.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-nodejs
- @azure/storage-blob — npm — https://www.npmjs.com/package/@azure/storage-blob
- How to Store and Serve Static Assets Metadata with MongoDB — https://github.com/OneUptime/blog/tree/master/posts/2026-03-31-mongodb-store-serve-static-assets-metadata
- Sharp — High Performance Node.js Image Processing — https://sharp.pixelplumbing.com/
- Sharp GitHub — https://github.com/lovell/sharp
- Jimp — JavaScript Image Manipulation Program — https://jimp-dev.github.io/jimp/
- Jimp GitHub — https://github.com/jimp-dev/jimp
- How to Create Image Processing in Node.js (Sharp & Jimp) — https://github.com/OneUptime/blog/tree/master/posts/2026-01-22-nodejs-image-processing
- OWASP File Upload Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html