# Advanced Concepts & Maintenance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced file handling maintenance encompasses the operational practices required to run file upload systems reliably at scale: resumable upload protocols that survive network interruptions, automated cleanup of orphaned files, rate limiting to prevent abuse, and observability through metrics and structured logs.

**Technical Definition:** This domain covers the TUS resumable upload protocol (an open HTTP-based standard for resumable uploads), manual chunked upload implementations using Multer and `fs.createWriteStream`, scheduled garbage collection using `node-cron` or system cron to delete orphaned files based on age or database reconciliation, rate limiting via `express-rate-limit` with Redis-backed stores for distributed deployments, and observability through Prometheus metrics (`prom-client`) and structured logging (`pino-http`).

**Beginner-Friendly Explanation:** Once your file upload system is working, you need to keep it running smoothly. Users upload huge files that get interrupted — you need resumable uploads so they don't have to start over. Uploaded files pile up — you need automated cleanup. Attackers try to spam your upload endpoint — you need rate limiting. And when something goes wrong, you need logs and metrics to figure out what happened. This cheat sheet covers all four.

### Key Characteristics

- **TUS is the standard:** The TUS protocol (tus.io) is the de facto standard for resumable uploads, supported by official Node.js servers and client libraries like `tus-js-client` and Uppy.
- **Three HTTP requests:** TUS uses POST (create upload), PATCH (send chunk at offset), and HEAD (query current offset) — the HEAD request is what enables resumption after a drop.
- **Cleanup is mandatory:** Orphaned temporary files, abandoned chunk directories, and expired uploads accumulate silently and can fill disks.
- **Rate limiting is not optional:** Upload endpoints are prime DoS targets; `express-rate-limit` with Redis stores prevents abuse across multiple server instances.
- **Observability drives reliability:** Prometheus metrics and Pino structured logs provide the data needed to diagnose failures and plan capacity.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express multer`.
- **For TUS:** `npm install @tus/server @tus/file-store`.
- **For cron:** `npm install node-cron`.
- **For rate limiting:** `npm install express-rate-limit` (and `rate-limit-redis` for distributed).
- **For metrics:** `npm install prom-client`.
- **For logging:** `npm install pino pino-http`.
- **A Redis instance** for distributed rate limiting and session stores.

### Related Programming Areas

- **File Uploads:** The foundation that these advanced concepts build upon.
- **File Storage & Processing:** Where files are stored and how they are cleaned up.
- **Security & Token Storage:** Rate limiting and access control for file endpoints.
- **DevOps:** Cron jobs, monitoring, and operational maintenance.
- **Observability:** Metrics, logs, and tracing for production systems.

### Core Concepts

1. **Resumable & Chunked Uploads** — TUS protocol and manual chunking.
2. **Garbage Collection & Cleanup** — Automated deletion of orphaned or temporary files.
3. **Rate Limiting File Endpoints** — Preventing DoS attacks via upload spam.
4. **Monitoring & Logs** — Tracking upload failures, storage usage, and bandwidth.

---

## Core Concept 1: Resumable & Chunked Uploads

### Definitions

**Core Definition:** Resumable uploads allow an interrupted file transfer to continue from where it stopped, rather than restarting from the beginning. Chunked uploads split a large file into smaller pieces that are transmitted separately and reassembled on the server.

**Technical Definition:** The TUS protocol is an open, HTTP-based standard for resumable file uploads, stable since version 1.0 and built on RFC 9110. It defines three core requests: POST (create an upload resource, declare `Upload-Length`, receive a URL), PATCH (send a chunk at a given `Upload-Offset`), and HEAD (query the current offset before resuming). The official Node.js implementation is `@tus/server`, which integrates into any framework and supports disk, S3, GCS, and Azure storage backends. Manual chunking uses client-side `File.slice()` to split files, with each chunk uploaded via Multer and reassembled on the server using `fs.createWriteStream` with append semantics.

**Beginner-Friendly Explanation:** Imagine uploading a 10GB video over a flaky connection. With a normal upload, if the connection drops at 90%, you start over from zero. With a resumable upload, the client asks the server "how many bytes do you already have?" and continues from there. TUS is the standard protocol for this. Manual chunking is the DIY version: split the file into 5MB pieces, upload each piece separately, and glue them together on the server.

### Purposes

- To survive network interruptions without re-uploading previously transferred data.
- To enable pause-and-resume functionality for large file uploads.
- To support uploads of arbitrary sizes (videos, disk images, datasets) over unreliable networks.
- To enable parallel chunk uploads for faster transfer.
- To provide a standardised, interoperable protocol across clients and servers.

### Sub-Feature 1.1: TUS Protocol

#### Syntax Rules and Structure

**TUS HTTP Requests:**

| Request | Purpose | Key Headers |
|---------|---------|-------------|
| `POST /files` | Create upload resource | `Upload-Length`, `Upload-Metadata` |
| `PATCH /files/{id}` | Send a chunk | `Upload-Offset`, `Content-Type: application/offset+octet-stream` |
| `HEAD /files/{id}` | Query current offset | — |

**Server Setup (Standalone):**
```js
import { Server } from "@tus/server";
import { FileStore } from "@tus/file-store";

const server = new Server({
  path: "/files",
  datastore: new FileStore({ directory: "./files" }),
  maxSize: 5 * 1024 * 1024 * 1024,  // 5 GB cap
  onUploadCreate: async (req, upload) => {
    // Auth + validation hook — throw to reject
    const type = upload.metadata?.filetype ?? "";
    if (!type.startsWith("video/")) {
      throw { status_code: 415, body: "Only video uploads allowed" };
    }
    return {};
  },
  onUploadFinish: async (req, upload) => {
    console.log(`Upload complete: ${upload.id} (${upload.size} bytes)`);
    return {};
  }
});
server.listen({ host: "127.0.0.1", port: 1080 });
```

**Client Setup (Browser):**
```js
import * as tus from "tus-js-client";

const upload = new tus.Upload(file, {
  endpoint: "http://localhost:1080/files",
  retryDelays: [0, 1000, 3000, 5000, 10000],  // Auto-retry on failure
  metadata: { filename: file.name, filetype: file.type },
  onProgress: (sent, total) => {
    console.log(`Progress: ${((sent / total) * 100).toFixed(1)}%`);
  },
  onSuccess: () => console.log("Done: " + upload.url)
});

// Resume a previous upload of this file if one exists
upload.findPreviousUploads().then((prev) => {
  if (prev.length) upload.resumeFromPreviousUpload(prev[0]);
  upload.start();
});
```

| Component | Breakdown |
|-----------|-----------|
| `maxSize` | Maximum upload size in bytes (5 GB in example). |
| `onUploadCreate` | Hook for auth and validation — throw to reject before bytes are stored. |
| `onUploadFinish` | Hook called when upload completes. |
| `retryDelays` | Backoff intervals for automatic retry. |
| `findPreviousUploads` | Enables page-reload resumption. |

**Rules:**
- `onUploadCreate` is where authentication and validation live — check tokens and file types before accepting the upload.
- The `maxSize` option caps the total upload size; requests exceeding it are rejected.
- `@tus/server` has no dependencies and integrates into Express, Fastify, and meta-frameworks (Next.js, Nuxt, SvelteKit).
- Storage adapters: `@tus/file-store` (disk), `@tus/s3-store` (S3), `@tus/gcs-store` (GCS), `@tus/azure-store` (Azure).
- The client's `retryDelays` array makes transient drops invisible — it backs off and retries automatically.

#### Annotated Code Example

```js
// tus-express-server.js
import express from "express";
import http from "node:http";
import { Server } from "@tus/server";
import { FileStore } from "@tus/file-store";

const app = express();

// TUS server configuration
const tusServer = new Server({
  path: "/files",
  datastore: new FileStore({ directory: "./uploads" }),
  maxSize: 5 * 1024 * 1024 * 1024,  // 5 GB

  // Authentication + validation hook
  async onUploadCreate(req, upload) {
    const token = req.headers["authorization"];
    if (!token || token !== "Bearer valid-token") {
      throw { status_code: 401, body: "Unauthorized" };
    }

    const filetype = upload.metadata?.filetype ?? "";
    if (!filetype.startsWith("video/")) {
      throw { status_code: 415, body: "Only video uploads allowed" };
    }
    return {};
  },

  // Completion hook
  async onUploadFinish(req, upload) {
    console.log(`✅ Upload complete: ${upload.id} (${upload.size} bytes)`);
    // Trigger post-processing (transcoding, virus scan, etc.)
    return {};
  }
});

// Route TUS requests to the TUS handler
app.all("/files", (req, res) => tusServer.handle(req, res));
app.all("/files/*", (req, res) => tusServer.handle(req, res));

app.listen(3000, () => console.log("TUS server on http://localhost:3000/files"));
```

**Expected Output (server console during upload):**
```
✅ Upload complete: abc123 (52428800 bytes)
```

**Expected Output (client progress):**
```
Progress: 0.0%
Progress: 41.7%
Progress: 62.3%
(connection drops — auto-retry)
Progress: 62.3%
Progress: 100.0%
```

**Why this output:** The `onUploadCreate` hook validates the token and file type before accepting the upload — rejected uploads never store a single byte. The `onUploadFinish` hook fires when all chunks have been received, enabling post-processing. The client's `retryDelays` array handles the connection drop, and `findPreviousUploads` enables resumption after a page reload.

#### Real-World Cases

- **Video platforms:** Uploading multi-gigabyte video files over mobile networks.
- **Cloud storage apps:** Desktop sync clients that resume after network changes.
- **Medical imaging:** Transferring large DICOM files over unreliable hospital networks.
- **Media production:** Uploading raw footage from field locations with poor connectivity.

---

### Sub-Feature 1.2: Manual Chunked Uploads

#### Definitions

**Core Definition:** Manual chunked uploads split a file into fixed-size chunks on the client, upload each chunk separately, and reassemble them on the server.

**Technical Definition:** The client uses `Blob.slice(start, end)` to split the file into chunks (typically 2–10 MB). Each chunk is uploaded as a separate HTTP request with metadata identifying the file, chunk index, and total chunk count. The server stores each chunk temporarily and, after receiving all chunks, concatenates them into the final file using `fs.createWriteStream` with sequential reads. Unlike TUS, manual chunking requires custom implementation on both client and server.

#### Syntax Rules and Structure

**Client-side chunking:**
```js
async function uploadInChunks(file, chunkSize = 5 * 1024 * 1024) {
  const totalChunks = Math.ceil(file.size / chunkSize);
  const uploadId = crypto.randomUUID();

  for (let i = 0; i < totalChunks; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);

    const formData = new FormData();
    formData.append("chunk", chunk);
    formData.append("uploadId", uploadId);
    formData.append("chunkIndex", i);
    formData.append("totalChunks", totalChunks);
    formData.append("filename", file.name);

    await fetch("/api/upload/chunk", {
      method: "POST",
      body: formData
    });
  }

  // Signal completion
  await fetch("/api/upload/complete", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ uploadId, filename: file.name, totalChunks })
  });
}
```

**Server-side chunk handling:**
```js
const upload = multer({ dest: "tmp/chunks/" });

app.post("/api/upload/chunk", upload.single("chunk"), (req, res) => {
  const { uploadId, chunkIndex } = req.body;
  const chunkDir = path.join("tmp/chunks", uploadId);
  fs.mkdirSync(chunkDir, { recursive: true });

  const chunkPath = path.join(chunkDir, `chunk-${chunkIndex}`);
  fs.renameSync(req.file.path, chunkPath);

  res.json({ received: true, chunkIndex });
});

app.post("/api/upload/complete", async (req, res) => {
  const { uploadId, filename, totalChunks } = req.body;
  const chunkDir = path.join("tmp/chunks", uploadId);
  const finalPath = path.join("uploads", filename);

  // Concatenate chunks in order
  const writeStream = fs.createWriteStream(finalPath);
  for (let i = 0; i < totalChunks; i++) {
    const chunkPath = path.join(chunkDir, `chunk-${i}`);
    const data = fs.readFileSync(chunkPath);
    writeStream.write(data);
    fs.unlinkSync(chunkPath);  // Clean up chunk
  }
  writeStream.end();

  // Remove chunk directory
  fs.rmdirSync(chunkDir);

  res.json({ message: "Upload complete", path: finalPath });
});
```

**Rules:**
- Chunk size should be large enough to reduce HTTP overhead but small enough to survive interruptions (2–10 MB is typical).
- Each chunk must include an `uploadId` to associate it with the correct upload session.
- The server must handle out-of-order chunk arrival (store by index, not arrival order).
- Chunks must be cleaned up after successful concatenation to prevent disk exhaustion.
- Manual chunking lacks built-in resumption — the client must track which chunks succeeded and retry failed ones.

#### Annotated Code Example

```js
// manual-chunking.js
const express = require("express");
const multer = require("multer");
const fs = require("fs");
const path = require("path");
const crypto = require("crypto");
const app = express();

app.use(express.json());

const upload = multer({ dest: "tmp/chunks/" });

// Receive a chunk
app.post("/api/upload/chunk", upload.single("chunk"), (req, res) => {
  const { uploadId, chunkIndex, totalChunks } = req.body;
  const chunkDir = path.join("tmp/chunks", uploadId);
  fs.mkdirSync(chunkDir, { recursive: true });

  const chunkPath = path.join(chunkDir, `chunk-${String(chunkIndex).padStart(6, "0")}`);
  fs.renameSync(req.file.path, chunkPath);

  res.json({
    received: true,
    uploadId,
    chunkIndex: parseInt(chunkIndex),
    totalChunks: parseInt(totalChunks)
  });
});

// Complete the upload
app.post("/api/upload/complete", async (req, res) => {
  const { uploadId, filename, totalChunks } = req.body;
  const chunkDir = path.join("tmp/chunks", uploadId);
  const safeName = path.basename(filename);
  const finalPath = path.join("uploads", `${crypto.randomUUID()}-${safeName}`);

  const writeStream = fs.createWriteStream(finalPath);
  let written = 0;

  for (let i = 0; i < totalChunks; i++) {
    const chunkName = `chunk-${String(i).padStart(6, "0")}`;
    const chunkPath = path.join(chunkDir, chunkName);

    if (!fs.existsSync(chunkPath)) {
      return res.status(400).json({ error: `Missing chunk ${i}` });
    }

    const data = fs.readFileSync(chunkPath);
    writeStream.write(data);
    written += data.length;
    fs.unlinkSync(chunkPath);
  }

  writeStream.end();
  await new Promise(resolve => writeStream.on("finish", resolve));

  fs.rmdirSync(chunkDir);

  res.json({
    message: "Upload complete",
    path: finalPath,
    size: written,
    chunksMerged: totalChunks
  });
});

app.listen(3000, () => console.log("Chunked upload server on 3000"));
```

**Expected Output (for `POST /api/upload/complete`):**
```json
{
  "message": "Upload complete",
  "path": "uploads/a1b2c3d4-video.mp4",
  "size": 52428800,
  "chunksMerged": 10
}
```

**Why this output:** Each chunk is stored in a temporary directory with a zero-padded index (`chunk-000000`, `chunk-000001`, etc.) to ensure correct ordering. The completion endpoint reads chunks in order, writes them to the final file, deletes each chunk after writing, and removes the chunk directory. The `path.basename()` call sanitises the filename to prevent path traversal.

#### Real-World Cases

- **Legacy systems:** Applications that cannot adopt TUS but need chunked uploads.
- **Custom protocols:** Systems requiring specific chunk metadata or encryption per chunk.
- **Cloud storage direct uploads:** Uploading chunks directly to S3 with presigned URLs.
- **Resumable uploads without TUS:** When TUS is not available or not desired.

---

## Core Concept 2: Garbage Collection & Cleanup

### Definitions

**Core Definition:** Garbage collection and cleanup is the automated process of identifying and deleting files that are no longer needed — orphaned uploads, abandoned temporary chunks, expired sessions, and old log files — to reclaim disk space and maintain system health.

**Technical Definition:** Orphaned files are files that exist in storage but have no corresponding database record (or whose database record has been deleted). Cleanup strategies include: **age-based cleanup** (delete temporary files older than N hours), **orphan detection** (compare storage listing against database records), **expiration-based cleanup** (TUS uploads with expiration extensions), and **quota-based cleanup** (delete oldest files when storage exceeds a threshold). Scheduling is done via `node-cron` (in-process) or system cron (external), with `node-cron` offering version control and easier testing.

**Beginner-Friendly Explanation:** Uploaded files pile up like dirty dishes. Some are finished uploads that are no longer needed. Some are abandoned chunks from interrupted uploads. Some are temporary files from failed processing jobs. A cron job runs on a schedule (e.g., every hour) and cleans them up automatically, so your disk doesn't fill up and your application doesn't crash.

### Purposes

- To prevent disk exhaustion from orphaned temporary files and abandoned uploads.
- To reclaim storage space consumed by expired or unused files.
- To maintain consistency between the database (source of truth) and storage.
- To automate routine maintenance without manual intervention.
- To comply with data retention policies (GDPR, HIPAA).

### Syntax Rules and Structure

**Cleanup Strategy Comparison:**

| Strategy | Trigger | Best For |
|----------|---------|----------|
| Age-based | File older than N hours/days | Temporary files, chunk directories |
| Orphan detection | No database record | Uploaded files with metadata in DB |
| Expiration-based | TUS `Expiration` extension | Resumable uploads that were abandoned |
| Quota-based | Storage exceeds threshold | Enforcing per-tenant storage limits |

**node-cron Setup:**
```js
const cron = require("node-cron");

// Run every hour at minute 0
cron.schedule("0 * * * *", async () => {
  console.log("Running cleanup...");
  await cleanupTempFiles();
  await cleanupOrphanedFiles();
}, { timezone: "UTC" });

// Run daily at 3 AM
cron.schedule("0 3 * * *", async () => {
  await cleanupOldLogs();
}, { timezone: "UTC", noOverlap: true });
```

| Option | Description |
|--------|-------------|
| `timezone` | Timezone for the schedule (e.g., `"UTC"`). |
| `noOverlap` | Prevents the job from running again if the previous run is still active. |
| `name` | Human-readable job name for logging. |

**Rules:**
- Use `noOverlap: true` to prevent concurrent runs if a job takes longer than the interval.
- Log every cleanup run: how many files were deleted, how much space was reclaimed.
- Always run cleanup in a **dry-run mode** first when implementing to verify it targets the right files.
- Age-based cleanup should use a threshold that accounts for the longest possible upload duration.
- Orphan detection requires a reliable database record for every file — if you store files in S3 without metadata, you cannot detect orphans.

### Annotated Code Example

```js
// cleanup-cron.js
const express = require("express");
const cron = require("node-cron");
const fs = require("fs");
const path = require("path");
const mongoose = require("mongoose");

const app = express();

const TEMP_DIR = path.join(__dirname, "tmp", "chunks");
const UPLOAD_DIR = path.join(__dirname, "uploads");
const MAX_AGE_MS = 24 * 60 * 60 * 1000;  // 24 hours

// Asset schema (for orphan detection)
const AssetSchema = new mongoose.Schema({
  key: { type: String, required: true, unique: true },
  createdAt: { type: Date, default: Date.now }
});
const Asset = mongoose.model("Asset", AssetSchema);

// --- Age-based cleanup of temp chunks ---
async function cleanupTempChunks() {
  if (!fs.existsSync(TEMP_DIR)) return { deleted: 0 };

  const entries = fs.readdirSync(TEMP_DIR);
  let deleted = 0;

  for (const entry of entries) {
    const entryPath = path.join(TEMP_DIR, entry);
    const stat = fs.statSync(entryPath);
    const age = Date.now() - stat.mtimeMs;

    if (age > MAX_AGE_MS) {
      fs.rmSync(entryPath, { recursive: true, force: true });
      deleted++;
      console.log(`Deleted old chunk: ${entry} (age: ${Math.round(age / 3600000)}h)`);
    }
  }

  return { deleted };
}

// --- Orphan detection: files in storage with no DB record ---
async function cleanupOrphanedFiles() {
  const files = fs.readdirSync(UPLOAD_DIR);
  const dbKeys = new Set(
    (await Asset.find({}, "key").lean()).map(a => a.key)
  );

  let deleted = 0;
  for (const file of files) {
    if (!dbKeys.has(file)) {
      fs.unlinkSync(path.join(UPLOAD_DIR, file));
      deleted++;
      console.log(`Deleted orphaned file: ${file}`);
    }
  }

  return { deleted };
}

// --- Schedule cleanup jobs ---
// Hourly: clean temp chunks older than 24 hours
cron.schedule("0 * * * *", async () => {
  console.log(`[${new Date().toISOString()}] Cleanup started`);
  try {
    const chunks = await cleanupTempChunks();
    const orphans = await cleanupOrphanedFiles();
    console.log(`Cleanup complete: ${chunks.deleted} chunks, ${orphans.deleted} orphans deleted`);
  } catch (err) {
    console.error("Cleanup failed:", err.message);
  }
}, { timezone: "UTC", noOverlap: true });

// Daily at 3 AM: report storage usage
cron.schedule("0 3 * * *", async () => {
  const files = fs.readdirSync(UPLOAD_DIR);
  let totalBytes = 0;
  for (const file of files) {
    totalBytes += fs.statSync(path.join(UPLOAD_DIR, file)).size;
  }
  console.log(`Storage report: ${files.length} files, ${(totalBytes / 1024 / 1024).toFixed(2)} MB`);
}, { timezone: "UTC" });

app.listen(3000, () => console.log("Cleanup scheduler on 3000"));
```

**Expected Output (during hourly cleanup):**
```
[2026-01-15T10:00:00.000Z] Cleanup started
Deleted old chunk: abc123/chunk-000001 (age: 26h)
Deleted orphaned file: stale-upload.jpg
Cleanup complete: 1 chunks, 1 orphans deleted
```

**Expected Output (during daily report):**
```
Storage report: 145 files, 2048.50 MB
```

**Why this output:** The hourly job deletes temporary chunks older than 24 hours and orphaned files with no database record. The `noOverlap: true` option prevents concurrent runs. The daily job reports storage usage, which helps with capacity planning. The age threshold (24 hours) accounts for the longest possible upload duration while ensuring abandoned chunks are eventually removed.

### Real-World Cases

- **Chunked upload cleanup:** Deleting abandoned chunk directories from interrupted uploads.
- **TUS expiration:** Using the TUS `Expiration` extension to automatically clean up abandoned resumable uploads.
- **Multi-tenant SaaS:** Enforcing per-tenant storage quotas by deleting oldest files when limits are exceeded.
- **Log rotation:** Deleting old log files to prevent log directory exhaustion.
- **GDPR compliance:** Automatically deleting user files after account deletion.

---

## Core Concept 3: Rate Limiting File Endpoints

### Definitions

**Core Definition:** Rate limiting restricts the number of requests a client can make to file upload endpoints within a time window, preventing denial-of-service (DoS) attacks, storage abuse, and resource exhaustion.

**Technical Definition:** `express-rate-limit` is middleware that tracks request counts per IP (or custom key) and returns a 429 status when the limit is exceeded. For distributed deployments, a Redis store (`rate-limit-redis`) ensures consistent limits across multiple server instances. Upload endpoints require stricter limits than general API routes because each request consumes significant resources (disk I/O, memory, bandwidth). The `max` (or `limit`) option specifies the number of requests, not bytes — capping total bytes requires custom middleware.

**Beginner-Friendly Explanation:** Without rate limiting, an attacker could send thousands of upload requests per second, filling your disk and crashing your server. Rate limiting says "you can upload 10 files per minute" and rejects anything beyond that with a 429 Too Many Requests response. This protects your server from being overwhelmed.

### Purposes

- To prevent Denial of Service (DoS) attacks via massive upload spam.
- To prevent storage abuse (filling disks with garbage uploads).
- To protect downstream services (virus scanners, image processors) from overload.
- To enforce fair usage across users and tenants.
- To comply with security best practices (OWASP API Security Top 10 — API4: Unrestricted Resource Consumption).

### Syntax Rules and Structure

```js
const rateLimit = require("express-rate-limit");
const RedisStore = require("rate-limit-redis");

// Upload-specific limiter
const uploadLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args)
  }),
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 10,                    // 10 uploads per window
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: "Too Many Requests",
      message: "Upload limit exceeded. Try again later.",
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  }
});

// General API limiter (more permissive)
const apiLimiter = rateLimit({
  store: new RedisStore({ sendCommand: (...args) => redisClient.sendCommand(args) }),
  windowMs: 15 * 60 * 1000,
  max: 100
});

app.use("/api/", apiLimiter);
app.use("/api/upload", uploadLimiter);
```

| Option | Description |
|--------|-------------|
| `windowMs` | Time window in milliseconds. |
| `max` | Maximum requests per window (alias: `limit`). |
| `standardHeaders` | Send `RateLimit-*` headers. |
| `legacyHeaders` | Disable `X-RateLimit-*` headers. |
| `store` | Custom store (Redis for distributed). |
| `handler` | Custom 429 response handler. |

**Recommended Limits:**

| Endpoint Type | Limit | Window | Rationale |
|--------------|-------|--------|-----------|
| General API | 1000 | 15 min | Prevent excessive usage. |
| Upload (presigned URL) | 10 | 1 min | Prevent storage abuse. |
| Download | 50 | 15 min | Prevent bandwidth abuse. |
| Auth | 5 | 15 min | Prevent brute force. |

**Rules:**
- Upload endpoints should have **stricter limits** than general API endpoints.
- Use a Redis store for distributed deployments — the default in-memory store resets on restart and is not shared across instances.
- The `max` option counts **requests**, not bytes. Capping total bytes requires custom middleware.
- Return a `Retry-After` header (or `retryAfter` in the JSON body) so clients know when to retry.
- Rate limiting should be applied **before** body parsing on upload endpoints to reject spam early.

### Annotated Code Example

```js
// rate-limiting-uploads.js
const express = require("express");
const rateLimit = require("express-rate-limit");
const RedisStore = require("rate-limit-redis");
const redis = require("redis");
const multer = require("multer");
const app = express();

const redisClient = redis.createClient();
redisClient.connect();

// Upload limiter: 10 uploads per 15 minutes per IP
const uploadLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args),
    prefix: "rl:upload:"
  }),
  windowMs: 15 * 60 * 1000,
  max: 10,
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: "Too Many Requests",
      message: "Upload limit exceeded. Maximum 10 uploads per 15 minutes.",
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  }
});

// General API limiter: 1000 requests per 15 minutes
const apiLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args),
    prefix: "rl:api:"
  }),
  windowMs: 15 * 60 * 1000,
  max: 1000
});

const upload = multer({ dest: "uploads/" });

// Apply general limiter to all API routes
app.use("/api/", apiLimiter);

// Apply stricter limiter to upload routes
app.post("/api/upload", uploadLimiter, upload.single("file"), (req, res) => {
  res.json({ message: "Upload received", file: req.file.filename });
});

// Download limiter: 50 downloads per 15 minutes
const downloadLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args),
    prefix: "rl:download:"
  }),
  windowMs: 15 * 60 * 1000,
  max: 50
});

app.get("/api/download/:file", downloadLimiter, (req, res) => {
  res.sendFile(path.join(__dirname, "uploads", req.params.file));
});

app.listen(3000, () => console.log("Rate-limited server on 3000"));
```

**Expected Output (for the 11th upload within 15 minutes):**
```json
{
  "error": "Too Many Requests",
  "message": "Upload limit exceeded. Maximum 10 uploads per 15 minutes.",
  "retryAfter": 847
}
```

**Expected Output (headers):**
```
RateLimit-Limit: 10
RateLimit-Remaining: 0
RateLimit-Reset: 847
Retry-After: 847
```

**Why this output:** The upload limiter allows 10 requests per 15 minutes per IP. The 11th request is rejected with a 429 status and a structured error message. The `retryAfter` field tells the client to wait 847 seconds before retrying. The `RateLimit-*` headers provide standard rate limit information. The Redis store ensures the limit is enforced across all server instances.

### Real-World Cases

- **Public API gateways:** Protecting upload endpoints from abuse by anonymous clients.
- **SaaS platforms:** Enforcing per-tenant upload quotas.
- **Healthcare:** Preventing DoS attacks on document upload endpoints.
- **E-commerce:** Protecting product image upload endpoints from spam.

---

## Core Concept 4: Monitoring & Logs

### Definitions

**Core Definition:** Monitoring and logging for file uploads is the practice of collecting metrics, structured logs, and alerts to track upload success rates, storage usage, bandwidth consumption, and error patterns.

**Technical Definition:** Metrics are collected using `prom-client` (the standard Prometheus client for Node.js) and exposed on a `/metrics` endpoint for scraping. Key upload metrics include counters (`http_requests_total`), histograms (`video_upload_duration_seconds`), and gauges (`active_viewers_total`). Structured logging uses `pino` or `winston` to emit JSON logs with correlation IDs (`pino-http` middleware). Upload lifecycle events — presigned URL generated, upload complete, status transitions — are logged at each stage. Health checks (`GET /api/health`) test database connectivity and return status.

**Beginner-Friendly Explanation:** Without monitoring, you're flying blind. You don't know how many uploads succeed, how many fail, how much storage you're using, or how long uploads take. Monitoring gives you dashboards and alerts. Logging gives you the details when something goes wrong. Together, they let you diagnose problems, plan capacity, and prove that your system is working.

### Purposes

- To track upload failures and identify patterns (timeouts, size limits, validation errors).
- To measure storage usage and bandwidth consumption for capacity planning.
- To detect anomalies (sudden spike in failures, unusual upload patterns).
- To provide audit trails for compliance and debugging.
- To enable alerting when upload success rates drop below acceptable thresholds.

### Syntax Rules and Structure

**Metrics (prom-client):**
```js
const promClient = require("prom-client");
const register = new promClient.Registry();

// Enable default Node.js metrics (CPU, memory, event loop lag, GC)
promClient.collectDefaultMetrics({ register });

// Upload-specific metrics
const uploadCounter = new promClient.Counter({
  name: "file_uploads_total",
  help: "Total number of file uploads",
  labelNames: ["status", "mime_type"],
  registers: [register]
});

const uploadDuration = new promClient.Histogram({
  name: "file_upload_duration_seconds",
  help: "Upload duration in seconds",
  labelNames: ["status"],
  buckets: [0.1, 0.5, 1, 5, 10, 30, 60],
  registers: [register]
});

const storageGauge = new promClient.Gauge({
  name: "storage_bytes_total",
  help: "Total bytes stored",
  registers: [register]
});

// Metrics endpoint
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});
```

| Metric Type | Use Case | Example |
|-------------|----------|---------|
| Counter | Total count of events | `file_uploads_total` |
| Histogram | Distribution of values | `file_upload_duration_seconds` |
| Gauge | Current value | `storage_bytes_total` |

**Structured Logging (pino-http):**
```js
const pino = require("pino");
const pinoHttp = require("pino-http");

const logger = pino({ level: "info" });
app.use(pinoHttp({ logger }));

// Upload lifecycle logging
logger.info({
  event: "upload_start",
  uploadId: req.params.id,
  fileSize: req.headers["content-length"],
  mimeType: req.headers["content-type"]
});

logger.info({
  event: "upload_complete",
  uploadId: req.params.id,
  fileSize: uploadedBytes,
  durationMs: Date.now() - startTime
});
```

**Rules:**
- Always label metrics with `status` (success/failure) to enable success rate calculations.
- Log upload lifecycle events at each stage: start, chunk received, complete, failed.
- Include a correlation ID (`req.id` from `pino-http`) in every log entry.
- Expose `/metrics` on a separate port or protected route if it should not be public.
- Health checks should test database and storage connectivity, not just return 200.

### Annotated Code Example

```js
// monitoring-logging.js
const express = require("express");
const promClient = require("prom-client");
const pino = require("pino");
const pinoHttp = require("pino-http");
const app = express();

// --- Metrics setup ---
const register = new promClient.Registry();
promClient.collectDefaultMetrics({ register });

const uploadCounter = new promClient.Counter({
  name: "file_uploads_total",
  help: "Total file uploads",
  labelNames: ["status", "mime_type"],
  registers: [register]
});

const uploadDuration = new promClient.Histogram({
  name: "file_upload_duration_seconds",
  help: "Upload duration",
  labelNames: ["status"],
  buckets: [0.1, 0.5, 1, 5, 10, 30],
  registers: [register]
});

const storageGauge = new promClient.Gauge({
  name: "storage_bytes_total",
  help: "Total bytes stored",
  registers: [register]
});

// --- Structured logging ---
const logger = pino({ level: "info" });
app.use(pinoHttp({ logger }));

// --- Upload endpoint with monitoring ---
const upload = require("multer")({ dest: "uploads/" });

app.post("/api/upload", upload.single("file"), (req, res) => {
  const startTime = Date.now();
  const mimeType = req.file.mimetype;

  try {
    // Simulate upload processing
    const fileSize = req.file.size;

    // Record metrics
    uploadCounter.inc({ status: "success", mime_type: mimeType });
    uploadDuration.observe({ status: "success" }, (Date.now() - startTime) / 1000);
    storageGauge.inc(fileSize);

    // Structured log
    logger.info({
      event: "upload_complete",
      uploadId: req.file.filename,
      originalName: req.file.originalname,
      mimeType,
      fileSize,
      durationMs: Date.now() - startTime
    });

    res.json({ message: "Upload successful", file: req.file.filename });
  } catch (err) {
    uploadCounter.inc({ status: "failure", mime_type: mimeType });
    logger.error({
      event: "upload_failed",
      error: err.message,
      mimeType
    });
    res.status(500).json({ error: "Upload failed" });
  }
});

// --- Metrics endpoint ---
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});

// --- Health check ---
app.get("/api/health", async (req, res) => {
  try {
    // Test database connectivity (simulated)
    await new Promise(resolve => setTimeout(resolve, 10));
    res.json({ status: "healthy", timestamp: new Date().toISOString() });
  } catch (err) {
    res.status(503).json({ status: "unhealthy", error: err.message });
  }
});

app.listen(3000, () => console.log("Monitoring server on 3000"));
```

**Expected Output (for `GET /metrics` — Prometheus format):**
```
# HELP file_uploads_total Total file uploads
# TYPE file_uploads_total counter
file_uploads_total{status="success",mime_type="image/jpeg"} 145
file_uploads_total{status="failure",mime_type="image/png"} 3

# HELP file_upload_duration_seconds Upload duration
# TYPE file_upload_duration_seconds histogram
file_upload_duration_seconds_bucket{status="success",le="0.5"} 120
file_upload_duration_seconds_bucket{status="success",le="1"} 140
file_upload_duration_seconds_bucket{status="success",le="+Inf"} 145
file_upload_duration_seconds_sum{status="success"} 42.3
file_upload_duration_seconds_count{status="success"} 145

# HELP storage_bytes_total Total bytes stored
# TYPE storage_bytes_total gauge
storage_bytes_total 1073741824
```

**Expected Output (structured log — JSON):**
```json
{
  "level": 30,
  "time": 1737000000000,
  "event": "upload_complete",
  "uploadId": "a1b2c3d4e5f6",
  "originalName": "photo.jpg",
  "mimeType": "image/jpeg",
  "fileSize": 24576,
  "durationMs": 234,
  "reqId": "req-abc-123"
}
```

**Why this output:** The Prometheus metrics expose upload counts by status and MIME type, duration distributions, and total storage bytes. The structured log entry includes all relevant fields for debugging: upload ID, original name, MIME type, file size, duration, and a correlation ID (`reqId`) that ties the log entry to the HTTP request. The `/api/health` endpoint tests database connectivity and returns a timestamp.

### Real-World Cases

- **Video platforms:** Monitoring upload success rates and transcoding queue depths.
- **E-commerce:** Tracking product image upload failures and storage usage per seller.
- **Healthcare:** Audit logging every document upload with correlation IDs for compliance.
- **SaaS platforms:** Per-tenant storage usage dashboards for billing and capacity planning.
- **CDN-backed applications:** Monitoring bandwidth consumption and cache hit rates.

---

## References

- tus-node-server GitHub — https://github.com/tus/tus-node-server
- Build resumable video uploads with tus, Node, and tus-js-client — https://dev.to/masonwritescode/build-resumable-video-uploads-with-tus-node-and-tus-js-client-2c62
- TUS Protocol Specification — https://tus.io/protocols/resumable-upload.html
- @tus/server on npm — https://www.npmjs.com/package/@tus/server
- Node.js Cron Jobs: System Cron vs node-cron (2026 Guide) — https://www.crongen.com/blog/nodejs-cron-jobs-system-vs-node-cron
- How To Use node-cron to Run Scheduled Jobs in Node.js — https://www.progressiverobot.com/2026/05/12/nodejs-cron-jobs-by-examples/
- find-remove on npm — https://www.npmjs.com/package/find-remove
- express-rate-limit on npm — https://www.npmjs.com/package/express-rate-limit
- Rate Limiting in Node.js: 12 Step [2026] — https://shattered.io/rate-limiting-nodejs/
- Secure API Routes with Rate Limiting (GitHub Issue) — https://github.com/gdg-charusat/Zaplink_backend/issues/2
- prom-client on npm — https://www.npmjs.com/package/prom-client
- pino-http on npm — https://www.npmjs.com/package/pino-http
- Monitor Express JS App using Prometheus & Grafana — https://github.com/farzeen-ali/Monitor-Express-JS-App-using-Prometheus-and-Grafana
- Structured Logging with Pino — https://github.com/oakoss/agent-skills/blob/main/skills/pino-logging/references/http-and-frameworks.md
- Upload completion idempotency and orphan cleanup (llm-driven-system-design) — https://github.com/evgenyvinnik/llm-driven-system-design/blob/main/loom/architecture.md
- clean-orphan-uploads-cli on npm — https://www.npmjs.com/package/clean-orphan-uploads-cli
- Rate limiting and path traversal guard to upload handler (GitHub commit) — https://github.com/taverns-red/speech-evaluator/commit/81fcba954040be878d50c94f360b19de9ab1e4a8
- OWASP API Security Top 10 — API4: Unrestricted Resource Consumption — https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/
- Multer 2.x Documentation — https://www.npmjs.com/package/multer
- TUS Protocol Extensions — https://tus.io/protocols/resumable-upload.html#protocol-extensions