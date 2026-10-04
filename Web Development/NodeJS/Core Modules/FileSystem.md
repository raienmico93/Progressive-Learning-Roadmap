# Node.js File System & Storage (fs) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `node:fs` module is Node.js's built-in interface for interacting with the file system in a manner modeled on standard POSIX functions, enabling programs to read, write, manipulate, and inspect files and directories on the host operating system.

**Technical Definition:** The `node:fs` module provides an API for interacting with the file system in a manner closely modeled around standard POSIX functions. All file system operations have synchronous, callback, and promise-based forms, accessible using both CommonJS syntax and ES6 Modules (ESM). The promise-based APIs are exposed via `node:fs/promises` and use the underlying Node.js threadpool to perform file system operations off the event loop thread.

**Beginner-Friendly Explanation:** Think of `fs` as Node.js's toolbox for working with your computer's files and folders. Just like you use a file explorer to create, open, rename, or delete files, `fs` lets your JavaScript code do the same thing programmatically — but with three different "speeds": one that waits (synchronous), one that takes a callback (asynchronous), and one that uses modern promises (also asynchronous).

### Key Characteristics

- **Tri-modal API:** Every operation has synchronous, callback-based, and promise-based variants.
- **POSIX-modeled:** The API closely mirrors POSIX file system functions, making it familiar to developers with systems programming experience.
- **Buffer and string support:** File paths can be strings, Buffers, or URL objects using the `file:` protocol.
- **Stream-enabled:** Large files can be processed incrementally through `Readable` and `Writable` streams rather than loading entire files into memory.
- **FileHandle abstraction:** The promise-based API uses `FileHandle` objects to wrap numeric file descriptors, helping avoid accidental file descriptor leaks.
- **Threadpool offloading:** Promise-based operations run on the libuv threadpool, keeping the main event loop unblocked.

### Prerequisites

- **Node.js runtime:** `fs` is built into Node.js; no external installation is required. Promise-based APIs (`fs/promises`) are available since Node.js v10.0.0 and became non-experimental in v11.14.0 and v10.17.0.
- **Basic JavaScript knowledge:** Understanding of callbacks, Promises, and `async/await` is essential.
- **Terminal/command-line familiarity:** Ability to run Node.js scripts (`node script.js`).
- **Understanding of asynchronous programming:** The event loop, non-blocking I/O, and error-first callbacks.

### Related Programming Areas

- **Streams and Buffers:** File streams (`fs.createReadStream`, `fs.createWriteStream`) build upon Node.js's stream architecture.
- **Path manipulation:** The `node:path` module is frequently used alongside `fs` for constructing cross-platform file paths.
- **HTTP servers:** Serving static files, handling uploads, and logging all depend on `fs`.
- **Process management:** Reading configuration files, writing logs, and managing temporary files.
- **Database alternatives:** Simple file-based storage for lightweight applications.
- **Child processes:** Passing file descriptors between processes.

### Core Concepts

The following core concepts are covered in this cheat sheet, each with a uniform structure:

1. **Callbacks vs. Promises vs. Sync** — the three execution models.
2. **File Operations** — creation, reading, writing, appending, renaming, unlinking.
3. **Directory & Metadata Management** — recursive `mkdir`, `readdir`, `fs.stat`.
4. **Resource Optimization** — buffer-based loading vs. memory-efficient streams.

---

## Core Concept 1: Callbacks vs. Promises vs. Sync

### Definitions

**Core Definition:** Node.js file system operations can be executed in three distinct forms: synchronous (blocking), callback-based asynchronous (non-blocking with error-first callbacks), and Promise-based asynchronous (non-blocking with `async/await` syntax).

**Technical Definition:** The synchronous form blocks the Node.js event loop and further JavaScript execution until the operation is complete. The callback form takes a completion callback function as its last argument and invokes the operation asynchronously. The Promise-based form returns a Promise that is fulfilled when the asynchronous operation is complete, and uses the underlying Node.js threadpool to perform operations off the event loop thread.

**Beginner-Friendly Explanation:** Imagine ordering food at three types of restaurants. **Sync** is a restaurant where the cashier stops serving anyone else until your food is ready — nobody else can order. **Callback** is a restaurant where you place your order, get a buzzer, and go sit down; the buzzer rings when your food is ready. **Promise** is the same buzzer system, but with a modern app that neatly tracks your order status and handles any problems more cleanly.

### Purposes

- To provide flexible execution models that suit different performance and readability requirements.
- To allow blocking file operations for simple scripts where sequential execution is acceptable.
- To enable non-blocking I/O that keeps the Node.js event loop responsive under load.
- To offer modern, readable asynchronous code through `async/await` and Promise chaining.
- To ensure that every file system operation has a consistent interface across all three forms.

### Syntax Rules and Structure

#### General Syntaxes

**Synchronous form:**
```js
const data = fs.readFileSync(path, options);
```
| Component | Breakdown |
|-----------|-----------|
| `fs` | The `node:fs` module (callback/sync API). |
| `readFileSync` | Synchronous method name (suffix `Sync`). |
| `path` | String, Buffer, or URL identifying the file. |
| `options` | Optional object or string (e.g., `'utf8'`). |
| Returns | The file contents (Buffer or string); throws on error. |

**Callback form:**
```js
fs.readFile(path, options, (err, data) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `readFile` | Asynchronous callback-based method. |
| `path` | File path. |
| `options` | Optional encoding/flag object. |
| `callback` | `(err, data) => {}` — first argument is always the error (or `null`/`undefined` on success). |

**Promise form:**
```js
import { readFile } from 'node:fs/promises';
const data = await readFile(path, options);
```
| Component | Breakdown |
|-----------|-----------|
| `node:fs/promises` | Promise-based API namespace. |
| `readFile` | Returns a Promise. |
| `await` | Pauses async function execution until the Promise settles. |
| Throws | Rejected Promise on error (caught via `try/catch`). |

**Callback form (performance note):** The callback-based versions are preferable over Promise APIs when maximal performance (execution time and memory allocation) is required. Promise APIs add overhead from Promise allocation and the threadpool queue.

**Synchronous form (blocking warning):** Synchronous APIs block the event loop and further JavaScript execution until the operation is complete. Exceptions are thrown immediately and can be handled with `try…catch`. Use sync APIs only in scripts that run once (e.g., build tools) or during application startup, never in request handlers.

#### Syntax Rules

- Sync methods always end with the `Sync` suffix (`readFileSync`, `writeFileSync`, `unlinkSync`).
- Callback methods always take the callback as the **last argument**, and the callback's **first argument is reserved for an error** (`null` or `undefined` on success).
- Promise methods are imported from `node:fs/promises` (or accessed as `fs.promises`) and must be used within an `async` function (or with `.then()`/`.catch()`).
- In ESM, use `import * as fs from 'node:fs/promises'`; in CommonJS, use `const fs = require('node:fs/promises')`

#### Constraints and Limitations

- Sync operations block the entire event loop — never use them in HTTP request handlers or other latency-sensitive contexts.
- Promise APIs are not synchronized or threadsafe; concurrent modifications to the same file may corrupt data.
- Callback APIs can lead to deeply nested code ("callback hell") if not managed carefully.
- `fs.exists()` is deprecated; use `fs.access()` or handle `ENOENT` errors directly.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading a File in All Three Forms

**Setup:** Create a file `greeting.txt` containing `Hello, Node.js!`.

```bash
echo "Hello, Node.js!" > greeting.txt
```
\
**sync-read.js**
```js
const fs = require('node:fs');

// Synchronous: blocks until the file is fully read.
try {
  const data = fs.readFileSync('greeting.txt', 'utf8');   // Blocking read
  console.log('[Sync]', data);                            // Output: Hello, Node.js!
} catch (err) {
  console.error('[Sync] Error:', err.message);
}
```

**Expected Output:**
```
[Sync] Hello, Node.js!
```

**Why this output:** `readFileSync` reads the entire file into memory and returns the string immediately. Because the operation is synchronous, the `console.log` runs only after the read completes. Any error (e.g., missing file) would throw an exception caught by `try/catch`.
\
**callback-read.js**
```js
const fs = require('node:fs');

// Callback: non-blocking; error is the first callback argument.
fs.readFile('greeting.txt', 'utf8', (err, data) => {
  if (err) throw err;                                 // Handle error first
  console.log('[Callback]', data);                    // Output: Hello, Node.js!
});
console.log('[Callback] Reading started...');         // Runs BEFORE the file read finishes
```

**Expected Output:**
```
[Callback] Reading started...
[Callback] Hello, Node.js!
```

**Why this output:** `fs.readFile` is asynchronous. The callback is scheduled to run later, after the file read completes, while the subsequent `console.log` executes immediately. The order demonstrates the non-blocking nature of callback-based I/O.

\
**promise-read.js**
```js
const fs = require('node:fs/promises');

(async () => {
  try {
    const data = await fs.readFile('greeting.txt', 'utf8'); // Await the Promise
    console.log('[Promise]', data);                         // Output: Hello, Node.js!
  } catch (err) {
    console.error('[Promise] Error:', err.message);
  }
})();
```

**Expected Output:**
```
[Promise] Hello, Node.js!
```

**Why this output:** `fs.readFile` from `node:fs/promises` returns a Promise. The `await` keyword pauses the async function until the Promise resolves with the file contents, then `console.log` runs. Errors are caught by `try/catch`.

---

#### Example 2: Copying a File — Performance-Conscious Choice

**copy-callback.js** — callback version for maximum performance
```js
const fs = require('node:fs');

fs.readFile('source.txt', (err, data) => {
  if (err) throw err;
  fs.writeFile('dest-callback.txt', data, (err) => {
    if (err) throw err;
    console.log('File copied (callback).');
  });
});
```

**copy-promise.js** — readable async/await version
```js
const fs = require('node:fs/promises');

(async () => {
  try {
    const data = await fs.readFile('source.txt');
    await fs.writeFile('dest-promise.txt', data);
    console.log('File copied (promise).');
  } catch (err) {
    console.error(err);
  }
})();
```

**Expected Output:**
```
File copied (callback).
File copied (promise).
```

**Why this output:** Both versions accomplish the same result. The callback version avoids Promise allocation overhead and is marginally faster in high-throughput scenarios. The Promise version is more readable and easier to maintain. Choose based on the context: high-frequency, performance-critical operations favor callbacks; application logic favors Promises.

### Real-World Cases

- **Configuration loaders (sync):** At application startup, reading `config.json` synchronously is acceptable because the app has not yet begun serving requests.
- **HTTP request handlers (promise):** Reading or writing files inside an Express route handler must use Promise-based or callback-based APIs to avoid blocking other requests.
- **Build scripts (sync):** CLI tools that process files sequentially benefit from the simplicity of synchronous code.
- **High-throughput log processing (callback):** When handling thousands of file operations per second, the callback API's lower overhead becomes measurable.

---

## Core Concept 2: File Operations

### Sub-Feature 2.1: Handling Basic File Creation

#### Definitions

**Core Definition:** File creation in Node.js is typically achieved implicitly through write or append operations, which create the file if it does not already exist.

**Technical Definition:** `fs.writeFile()` and `fs.appendFile()` create the target file if it does not exist, using the default flag `'w'` (write) or `'a'` (append) respectively. The `fs.open()` method with the `'w'` flag can also explicitly create a file and return a file descriptor.

**Beginner-Friendly Explanation:** You don't need a separate "create file" command. When you tell Node.js to write something to a file that doesn't exist, it creates the file automatically — like a notebook that appears the moment you start writing in it.

#### Purposes

- To persist data generated by an application to disk.
- To initialise configuration or output files programmatically.
- To create placeholder files as part of a build or setup process.

#### Syntax Rules and Structure

**Promise-based creation via writeFile:**
```js
await fs.writeFile(path, data, options);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | File path (string, Buffer, or URL). |
| `data` | String, Buffer, TypedArray, or DataView. |
| `options` | Optional: `{ encoding, mode, flag }`. Default flag `'w'`. |
| Creates | File if it does not exist; truncates if it does. |

**Callback-based explicit creation via open:**
```js
fs.open(path, 'w', (err, fd) => { /* use fd */ });
```
| Component | Breakdown |
|-----------|-----------|
| `path` | File path. |
| `'w'` | Open for writing; creates if absent, truncates if present. |
| `fd` | Numeric file descriptor. |

**Constraints and limitations:**
- `writeFile` with default flag `'w'` **overwrites** an existing file. Use flag `'wx'` to fail if the file exists.
- The `mode` option (default `0o666`) is subject to the process umask.

#### Annotated Code Example

```js
// create-file.js
const fs = require('node:fs/promises');

(async () => {
  try {
    // writeFile creates 'notes.txt' because it does not exist.
    await fs.writeFile('notes.txt', 'First line.\n', 'utf8');
    console.log('File created successfully.');

    // Verify by reading it back
    const content = await fs.readFile('notes.txt', 'utf8');
    console.log('Content:', content);
  } catch (err) {
    console.error('Error:', err.message);
  }
})();
```

**notes.txt**
```
First line

```

**Expected Output:**
```
File created successfully.
Content: First line.
```

**Why this output:** `fs.writeFile` opens the path with flag `'w'`, creates the file because it does not exist, writes the string `'First line.\n'` using UTF-8 encoding, and closes the file. Reading it back confirms the content.

#### Real-World Cases

- **Log initialisation:** Creating today's log file at midnight if it doesn't exist.
- **User data:** Creating a new user's profile JSON file upon registration.
- **Build output:** Generating a `dist/` artifact file.

---

### Sub-Feature 2.2: Reading Files

#### Definitions

**Core Definition:** Reading a file means retrieving its contents into memory as a Buffer or string.

**Technical Definition:** `fs.readFile()` reads the entire contents of a file asynchronously, returning a Buffer unless an encoding is specified. `fs.readFileSync()` is the blocking equivalent. For large files, `fs.createReadStream()` provides incremental reading.

**Beginner-Friendly Explanation:** Reading a file is like opening a book and reading all its pages at once — or, for large files, reading a few pages at a time so you don't have to hold the whole book in your hands.

#### Purposes

- To load configuration, templates, or user-generated content.
- To process data files (CSV, JSON, text).
- To inspect file contents for validation or transformation.

#### Syntax Rules and Structure

```js
const data = await fs.readFile(path, options);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | File path. |
| `options` | Optional encoding (e.g., `'utf8'`), or `{ encoding, flag }`. |
| Returns | `<Buffer>` if no encoding; `<string>` if encoding specified. |

**Constraints:**
- Loading a very large file with `readFile` may exhaust memory; use streams instead.
- The entire file must fit in memory.

#### Annotated Code Example

```js
// read-file.js
const fs = require('node:fs/promises');

(async () => {
  // Write a sample file first
  await fs.writeFile('data.json', JSON.stringify({ name: 'Alice', age: 30 }));

  // Read as UTF-8 string
  const text = await fs.readFile('data.json', 'utf8');
  console.log('String:', text);

  // Read as Buffer (no encoding)
  const buf = await fs.readFile('data.json');
  console.log('Buffer length:', buf.length);
  console.log('Buffer toString:', buf.toString('utf8'));
})();
```

**data.json**
```json
{
  "name":"Alice",
  "age":30
}
```

**Expected Output:**
```
String: {"name":"Alice","age":30}
Buffer length: 28
Buffer toString: {"name":"Alice","age":30}
```

**Why this output:** Without an encoding, `readFile` returns a Buffer whose `.length` is the byte count. With `'utf8'`, the Buffer is decoded to a string. The JSON string is 28 bytes because each character is one byte in UTF-8 for this ASCII content.

#### Real-World Cases

- **Loading JSON configuration:** `const config = JSON.parse(await fs.readFile('config.json', 'utf8'));`
- **Template rendering:** Reading an HTML template file before injecting data.
- **Data processing:** Reading a CSV file for parsing and analysis.

---

### Sub-Feature 2.3: Writing Files

#### Definitions

**Core Definition:** Writing a file means creating or overwriting a file with new content.

**Technical Definition:** `fs.writeFile()` asynchronously writes data to a file, replacing the file if it already exists. The default flag is `'w'`. To write without overwriting existing content, use flag `'wx'` or use `appendFile()`

**Beginner-Friendly Explanation:** Writing a file is like using a whiteboard eraser and marker — whatever was there before is gone, and your new message takes its place.

#### Purposes

- To save application state, user data, or generated output.
- To overwrite outdated configuration or cache files.
- To produce reports, logs, or exports.

#### Syntax Rules and Structure

```js
await fs.writeFile(path, data, options);
```
| Component | Breakdown |
|-----------|-----------|
| `data` | String, Buffer, TypedArray, or DataView. |
| `options` | `{ encoding, mode, flag }`; default flag `'w'`. |
| Flag `'w'` | Create or truncate. |
| Flag `'wx'` | Create exclusively; fails if file exists. |

**Constraints:**
- Overwrites existing content by default.
- Not atomic; concurrent writes to the same file can corrupt data.

#### Annotated Code Example

```js
// write-file.js
const fs = require('node:fs/promises');

(async () => {
  // Write initial content
  await fs.writeFile('output.txt', 'Version 1\n');
  console.log('Written v1.');

  // Overwrite with new content
  await fs.writeFile('output.txt', 'Version 2\n');
  console.log('Written v2 (overwrote v1).');

  // Read back to confirm
  console.log(await fs.readFile('output.txt', 'utf8'));

  // Exclusive write — fails if file exists
  try {
    await fs.writeFile('output.txt', 'Should not write', { flag: 'wx' });
  } catch (err) {
    console.log('Exclusive write failed:', err.code); // EEXIST
  }
})();
```

**Expected Output:**
```
Written v1.
Written v2 (overwrote v1).
Version 2

Exclusive write failed: EEXIST
```

**Why this output:** The first `writeFile` creates the file with `'Version 1\n'`. The second `writeFile` truncates the file and writes `'Version 2\n'`. Reading confirms the overwrite. The `'wx'` flag prevents writing because the file already exists, producing an `EEXIST` error.

#### Real-World Cases

- **Saving user preferences:** Overwriting `user-settings.json` when settings change.
- **Generating reports:** Writing a daily sales report to a timestamped file.
- **Cache invalidation:** Overwriting a cache file with fresh data.

---

### Sub-Feature 2.4: Appending to Files

#### Definitions

**Core Definition:** Appending adds data to the end of a file without overwriting existing content, creating the file if it does not exist.

**Technical Definition:** `fs.appendFile()` asynchronously appends data to a file using the `'a'` flag, which positions the write at the end of the file. If the file does not exist, it is created.

**Beginner-Friendly Explanation:** Appending is like writing on the next blank line of a notebook rather than erasing the whole page. Everything you wrote before stays intact.

#### Purposes

- To add log entries without losing previous entries.
- To accumulate data over time (e.g., sensor readings).
- To build files incrementally without re-reading existing content.

#### Syntax Rules and Structure

```js
await fs.appendFile(path, data, options);
```
| Component | Breakdown |
|-----------|-----------|
| `data` | String or Buffer to append. |
| `options` | `{ encoding, mode, flag }`; flag is always `'a'` internally. |
| Creates | File if absent. |

**Constraints:**
- Append operations are not atomic across multiple processes without OS-level locking.
- The data is added at the end; you cannot specify an offset.

#### Annotated Code Example

```js
// append-file.js
const fs = require('node:fs/promises');

(async () => {
  // Create initial log
  await fs.writeFile('app.log', '[INFO] Server started\n');
  console.log('Log initialised.');

  // Append two more entries
  await fs.appendFile('app.log', '[INFO] Request received\n');
  await fs.appendFile('app.log', '[ERROR] Timeout\n');
  console.log('Appended two entries.');

  // Read the full log
  const log = await fs.readFile('app.log', 'utf8');
  console.log('--- Full Log ---');
  console.log(log);
})();
```

**Expected Output:**
```
Log initialised.
Appended two entries.
--- Full Log ---
[INFO] Server started
[INFO] Request received
[ERROR] Timeout

```

**Why this output:** `writeFile` creates the file with the first log line. Each `appendFile` call adds its string at the current end of the file. The final read shows all three lines in order. Note the trailing newline from the last append.

#### Real-World Cases

- **Application logging:** Continuously appending events to `app.log`.
- **Audit trails:** Recording user actions in a sequential file.
- **IoT data collection:** Appending sensor readings to a daily CSV file.

---

### Sub-Feature 2.5: Renaming and Moving Files

#### Definitions

**Core Definition:** Renaming changes a file's name or moves it to a different path within the same file system.

**Technical Definition:** `fs.rename(oldPath, newPath)` asynchronously renames a file or directory. If `oldPath` and `newPath` are on different file systems, the operation may fail (POSIX `rename(2)` semantics).

**Beginner-Friendly Explanation:** Renaming is like giving a file a new label, or moving it to a new folder. The file's contents stay the same — only its location changes.

#### Purposes

- To organise files into directories based on processing stage.
- To add timestamps or version numbers to filenames.
- To replace an old file atomically with a new one (write to temp, then rename).

#### Syntax Rules and Structure

```js
await fs.rename(oldPath, newPath);
```
| Component | Breakdown |
|-----------|-----------|
| `oldPath` | Current path. |
| `newPath` | Target path. |
| Behaviour | Overwrites `newPath` if it exists (POSIX). |

**Constraints:**
- May fail with `EXDEV` if source and destination are on different mount points.
- Renaming a directory also renames all its contents.
- Not guaranteed atomic across all file systems.

#### Annotated Code Example

```js
// rename-file.js
const fs = require('node:fs/promises');

(async () => {
  await fs.writeFile('draft.txt', 'This is a draft.\n');

  // Rename within the same directory
  await fs.rename('draft.txt', 'final.txt');
  console.log('Renamed draft.txt -> final.txt');

  // Verify old name is gone
  try {
    await fs.access('draft.txt');
  } catch {
    console.log('draft.txt no longer exists.');
  }

  // Read the renamed file
  console.log(await fs.readFile('final.txt', 'utf8'));
})();
```

**Expected Output:**
```
Renamed draft.txt -> final.txt
draft.txt no longer exists.
This is a draft.
```

**Why this output:** `fs.rename` changes the directory entry from `draft.txt` to `final.txt`. The inode and file contents remain unchanged. The subsequent `fs.access` fails because the old path no longer exists.

#### Real-World Cases

- **Upload processing:** Renaming uploaded files from temporary names to user-friendly names.
- **Log rotation:** Renaming `app.log` to `app-2026-01-01.log` and creating a fresh log.
- **Atomic writes:** Writing to `file.tmp` then renaming to `file.txt` for atomic replacement.

---

### Sub-Feature 2.6: Unlinking (Deleting) Files

#### Definitions

**Core Definition:** Unlinking removes a file's directory entry, effectively deleting it from the file system.

**Technical Definition:** `fs.unlink(path)` asynchronously removes a file or symbolic link. It does not work on directories; use `fs.rmdir()` (deprecated for recursive) or `fs.rm()` for directories.

**Beginner-Friendly Explanation:** Unlinking is like tearing a page out of a notebook. The page (data) may still exist on disk until overwritten, but the notebook's index no longer points to it.

#### Purposes

- To clean up temporary files after processing.
- To remove outdated or corrupted data files.
- To manage disk space by deleting unneeded files.

#### Syntax Rules and Structure

```js
await fs.unlink(path);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | File or symbolic link to remove. |
| Error | `ENOENT` if file does not exist. |

**Constraints:**
- Cannot delete directories.
- The file is removed immediately from the directory; open file descriptors may keep data accessible until closed.
- On Windows, unlinking a file with open handles may fail.

#### Annotated Code Example

```js
// unlink-file.js
const fs = require('node:fs/promises');

(async () => {
  await fs.writeFile('temp.txt', 'Temporary data');
  console.log('temp.txt created.');

  await fs.unlink('temp.txt');
  console.log('temp.txt deleted.');

  // Attempt to delete again — should fail
  try {
    await fs.unlink('temp.txt');
  } catch (err) {
    console.log('Second unlink failed:', err.code); // ENOENT
  }
})();
```

**Expected Output:**
```
temp.txt created.
temp.txt deleted.
Second unlink failed: ENOENT
```

**Why this output:** The first `unlink` removes the file. The second attempt fails with `ENOENT` ("Error NO ENTry") because the file no longer exists. Always handle this error gracefully in production code.

#### Real-World Cases

- **Temporary file cleanup:** Deleting `.tmp` files after a successful upload.
- **Cache eviction:** Removing expired cache files.
- **User-initiated deletion:** Implementing a "delete my data" feature.

---

## Core Concept 3: Directory & Metadata Management

### Sub-Feature 3.1: Creating Directories Recursively

#### Definitions

**Core Definition:** Recursive directory creation creates a directory and all necessary parent directories in a single operation.

**Technical Definition:** `fs.mkdir(path, { recursive: true })` creates the directory at `path` along with any missing parent directories. If `recursive` is `false` (the default), the operation fails with `ENOENT` if the parent does not exist.

**Beginner-Friendly Explanation:** Imagine you want to create a folder called `2026/photos/vacation`. Instead of manually creating `2026`, then `photos`, then `vacation`, you can create all three at once with one command. That's what `recursive: true` does.

#### Purposes

- To scaffold project directory structures without checking for parent existence.
- To create nested output directories for build tools or data pipelines.
- To simplify setup scripts and deployment automation.

#### Syntax Rules and Structure

```js
await fs.mkdir(path, { recursive: true });
```
| Component | Breakdown |
|-----------|-----------|
| `path` | Directory path to create. |
| `recursive` | Boolean; when `true`, creates parent directories as needed. |
| Returns | The first directory path created (when `recursive: true`). |

**Constraints:**
- Without `recursive: true`, `mkdir` throws `ENOENT` if the parent does not exist.
- With `recursive: true`, calling `mkdir` on an existing directory does **not** throw an error (unlike without the option, which throws `EEXIST`).

#### Annotated Code Example

```js
// mkdir-recursive.js
const fs = require('node:fs/promises');

(async () => {
  const nestedPath = './project/src/components/buttons';

  // Create nested directories in one call
  await fs.mkdir(nestedPath, { recursive: true });
  console.log('Nested directories created:', nestedPath);

  // Verify by listing the tree
  const entries = await fs.readdir('./project', { recursive: true });
  console.log('Tree:', entries);

  // Calling again does not throw (recursive mode)
  await fs.mkdir(nestedPath, { recursive: true });
  console.log('Second mkdir (recursive) succeeded without error.');
})();
```

**Expected Output:**
```
Nested directories created: ./project/src/components/buttons
Tree: [ 'src', 'src/components', 'src/components/buttons' ]
Second mkdir (recursive) succeeded without error.
```

**Why this output:** `mkdir` with `recursive: true` creates `project`, then `project/src`, then `project/src/components`, then `project/src/components/buttons`. Listing with `recursive: true` shows all levels. The second `mkdir` succeeds silently because the directory already exists and `recursive` mode suppresses the `EEXIST` error.

#### Real-World Cases

- **Project scaffolding:** Creating `src/assets/images` during `npm init` scripts.
- **Data pipelines:** Creating dated output directories like `output/2026/01/15/`.
- **User uploads:** Creating `uploads/{userId}/{date}/` upon file upload.

---

### Sub-Feature 3.2: Reading Directory Trees

#### Definitions

**Core Definition:** Reading a directory retrieves the names and optionally the types of its immediate contents.

**Technical Definition:** `fs.readdir(path, options)` reads the contents of a directory. With `{ withFileTypes: true }`, it returns `Dirent` objects that expose `.isFile()` and `.isDirectory()` methods. With `{ recursive: true }` (Node.js v20.1.0+), it recursively lists all nested entries.

**Beginner-Friendly Explanation:** Reading a directory is like listing everything inside a folder. You can choose to see just the names, or also whether each item is a file or a subfolder — and you can even ask for everything inside subfolders too.

#### Purposes

- To discover files for batch processing.
- To build file trees for UI display or indexing.
- To validate directory contents before operations.

#### Syntax Rules and Structure

```js
const entries = await fs.readdir(path, { withFileTypes: true, recursive: true });
```
| Option | Breakdown |
|--------|-----------|
| `withFileTypes` | Returns `Dirent` objects instead of strings. |
| `recursive` | Lists all nested entries (Node.js v20.1.0+). |

**Constraints:**
- Directory entries are returned in no particular order (as provided by the OS).
- `recursive` may be memory-intensive for very deep or large trees.

#### Annotated Code Example

```js
// readdir-tree.js
const fs = require('node:fs/promises');

(async () => {
  // Create a small tree for demonstration
  await fs.mkdir('tree/a', { recursive: true });
  await fs.mkdir('tree/b', { recursive: true });
  await fs.writeFile('tree/a/file1.txt', 'A1');
  await fs.writeFile('tree/b/file2.txt', 'B2');

  // List immediate contents with type information
  const entries = await fs.readdir('tree', { withFileTypes: true });
  for (const entry of entries) {
    console.log(
      entry.name,
      entry.isDirectory() ? '(Directory)' : '(File)'
    );
  }

  // Recursive listing (Node.js v20.1.0+)
  const all = await fs.readdir('tree', { recursive: true });
  console.log('Recursive:', all);
})();
```

**Expected Output:**
```
a (Directory)
b (Directory)
Recursive: [ 'a', 'a/file1.txt', 'b', 'b/file2.txt' ]
```

**Why this output:** The first `readdir` returns only immediate children (`a` and `b`, both directories). The recursive call returns all entries at every depth, with paths relative to the root directory.

#### Real-World Cases

- **Static site generators:** Walking a `content/` directory to build pages.
- **File upload processors:** Scanning an `incoming/` folder for new files.
- **Backup tools:** Enumerating all files under a root directory.

---

### Sub-Feature 3.3: Inspecting File Statistics (`fs.stat`)

#### Definitions

**Core Definition:** `fs.stat` retrieves metadata about a file or directory, including size, permissions, and timestamps.

**Technical Definition:** `fs.stat(path)` returns an `fs.Stats` object containing properties such as `size`, `mode`, `atime`, `mtime`, `ctime`, `birthtime`, and methods such as `isFile()` and `isDirectory()`. Note that `ctime` is the file node change time (metadata), not creation time.

**Beginner-Friendly Explanation:** `fs.stat` is like checking a file's ID card. It tells you how big the file is, when it was last opened, when it was last changed, and whether it's a file or a folder.

#### Purposes

- To determine whether a path is a file or directory before operating on it.
- To check file size before reading (to decide between buffer and stream).
- To implement cache invalidation based on modification time.

#### Syntax Rules and Structure

```js
const stats = await fs.stat(path);
```
| Property/Method | Breakdown |
|-----------------|-----------|
| `stats.size` | File size in bytes. |
| `stats.mtime` | Last modification time. |
| `stats.isFile()` | Returns `true` if path is a regular file. |
| `stats.isDirectory()` | Returns `true` if path is a directory. |

**Constraints:**
- `fs.stat` follows symbolic links; use `fs.lstat` to stat the link itself.
- Using `fs.stat()` to check for existence before `open`/`readFile`/`writeFile` is discouraged; handle errors directly.

#### Annotated Code Example

```js
// stat-file.js
const fs = require('node:fs/promises');

(async () => {
  await fs.writeFile('info.txt', 'Hello, stats!');

  const stats = await fs.stat('info.txt');
  console.log('Size:', stats.size, 'bytes');
  console.log('Is file:', stats.isFile());
  console.log('Is directory:', stats.isDirectory());
  console.log('Modified:', stats.mtime.toISOString());
  console.log('Created:', stats.birthtime.toISOString());

  // Compare with a directory
  const dirStats = await fs.stat('.');
  console.log('Current dir isDirectory:', dirStats.isDirectory());
})();
```

**Expected Output:**
```
Size: 13 bytes
Is file: true
Is directory: false
Modified: 2026-01-15T12:00:00.000Z
Created: 2026-01-15T12:00:00.000Z
Current dir isDirectory: true
```

**Why this output:** The file contains `"Hello, stats!"` which is 13 characters (13 bytes in ASCII). `isFile()` returns `true` and `isDirectory()` returns `false` for a regular file. The timestamps reflect the write operation. The current directory `.` is confirmed to be a directory.

#### Real-World Cases

- **Upload validators:** Checking `stats.size` to reject files over a size limit.
- **Cache managers:** Comparing `stats.mtime` against a cached timestamp to decide whether to re-read a file.
- **Directory walkers:** Using `stats.isDirectory()` to decide whether to recurse.

---

## Core Concept 4: Resource Optimization — Buffer vs. Stream

### Definitions

**Core Definition:** Buffer-based loading reads an entire file into memory at once, while stream-based loading reads and processes data in small chunks, keeping memory usage low regardless of file size.

**Technical Definition:** `fs.readFile()` loads the complete file contents into a Buffer in memory. `fs.createReadStream()` returns a `Readable` stream that emits data in chunks (default 64 KiB, configurable via `highWaterMark`). Streams are suitable for files larger than available memory or for piping data between sources and destinations.

**Beginner-Friendly Explanation:** Reading a file with `readFile` is like trying to drink an entire swimming pool in one gulp — impossible if the pool is huge. Streams are like sipping through a straw: you handle the water in small, manageable amounts, no matter how big the pool is.

### Purposes

- To prevent memory exhaustion when processing large files.
- To begin processing data before the entire file is read.
- To pipe file data directly to network responses or other files without intermediate storage.
- To reduce latency for real-time data processing.

### Syntax Rules and Structure

**Buffer-based reading:**
```js
const data = await fs.readFile(path); // entire file in memory
```

**Stream-based reading:**
```js
const stream = fs.createReadStream(path, { highWaterMark: 64 * 1024 });
stream.on('data', (chunk) => { /* process chunk */ });
stream.on('end', () => { /* finished */ });
```

| Component | Breakdown |
|-----------|-----------|
| `createReadStream(path)` | Returns a `Readable` stream. |
| `highWaterMark` | Chunk size in bytes (default 64 KiB). |
| `'data'` event | Emitted for each chunk. |
| `'end'` event | Emitted when all data has been read. |

**Constraints:**
- `readFile` fails or hangs if the file exceeds available memory (or the Buffer max size).
- Streams are asynchronous and event-driven; error handling must listen for the `'error'` event.
- Backpressure must be managed; if the writable side is slower than the readable side, memory can still accumulate unless `pipe()` or backpressure signals are used.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading a Large File — Buffer vs. Stream

```js
// buffer-vs-stream.js
const fs = require('node:fs');
const fsPromises = require('node:fs/promises');

(async () => {
  // Create a moderately large file (~5 MB) for demonstration
  const largeContent = 'A'.repeat(5 * 1024 * 1024);
  await fsPromises.writeFile('large.txt', largeContent);
  console.log('Large file created (5 MB).');

  // --- Buffer approach ---
  console.log('--- Buffer approach ---');
  const buf = await fsPromises.readFile('large.txt');
  console.log('Buffer length:', buf.length, 'bytes');

  // --- Stream approach ---
  console.log('--- Stream approach ---');
  let totalBytes = 0;
  let chunkCount = 0;
  const stream = fs.createReadStream('large.txt');

  stream.on('data', (chunk) => {
    totalBytes += chunk.length;
    chunkCount++;
  });

  stream.on('end', () => {
    console.log('Total bytes read:', totalBytes);
    console.log('Number of chunks:', chunkCount);
  });

  stream.on('error', (err) => {
    console.error('Stream error:', err.message);
  });
})();
```

**Expected Output:**
```
Large file created (5 MB).
--- Buffer approach ---
Buffer length: 5242880 bytes
--- Stream approach ---
Total bytes read: 5242880
Number of chunks: 80
```

**Why this output:** The buffer approach reads all 5 MB into a single Buffer (5,242,880 bytes). The stream approach reads the same data in chunks of 64 KiB (65,536 bytes). 5,242,880 / 65,536 = 80 chunks. The total bytes match, but the stream never holds more than one chunk in memory at a time.

#### Example 2: Piping a File to an HTTP Response

```js
// stream-server.js
const http = require('node:http');
const fs = require('node:fs');

const server = http.createServer((req, res) => {
  const stream = fs.createReadStream('large.txt');
  stream.pipe(res);            // Pipe file data directly to the HTTP response
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

**Expected Output (when visited in a browser):**
```
The browser downloads or displays the 5 MB file content without the server holding the entire file in memory.
```

**Why this output:** `stream.pipe(res)` connects the readable file stream to the writable HTTP response stream. Data flows chunk by chunk, and backpressure is managed automatically by the pipe mechanism.

### Real-World Cases

- **Video streaming:** Serving large video files with `createReadStream` and `pipe` to the HTTP response.
- **Log analysis:** Processing multi-gigabyte log files line by line with `readline` over a stream.
- **File uploads:** Streaming incoming request data to disk with `createWriteStream`.
- **Data ETL:** Reading CSV files in chunks, transforming each chunk, and writing to a database or output file.

---

## References

- Node.js Documentation — File System — https://nodejs.org/api/fs.html
- Node.js Documentation — `fs/promises` API — https://nodejs.org/api/fs.html#promises-api
- Node.js Documentation — Streams — https://nodejs.org/api/stream.html
- Node.js Documentation — `fs.createReadStream` — https://nodejs.org/api/fs.html#fscreatereadstreampath-options
- Node.js Documentation — `fs.Stats` — https://nodejs.org/api/fs.html#class-fsstats
- Node.js Documentation — `fs.mkdir` — https://nodejs.org/api/fs.html#fspromisesmkdirpath-options
- Node.js Documentation — `fs.readdir` — https://nodejs.org/api/fs.html#fspromisesreaddirpath-options
- Node.js Documentation — `fs.rename` — https://nodejs.org/api/fs.html#fspromisesrenameoldpath-newpath
- Node.js Documentation — `fs.unlink` — https://nodejs.org/api/fs.html#fspromisesunlinkpath
- Node.js Documentation — `fs.appendFile` — https://nodejs.org/api/fs.html#fspromisesappendfilepath-data-options
- Node.js Documentation — FileHandle Class — https://nodejs.org/api/fs.html#class-filehandle
- Node.js Documentation — Deprecation DEP0147 (`fs.rmdir` recursive) — https://nodejs.org/api/deprecations.html#dep0147-fsrmdirpath--recursive-true
- Node.js Documentation — `fs.exists` deprecation — https://nodejs.org/api/fs.html#fsexistspath-callback
- Node.js Documentation — `fs.rm` — https://nodejs.org/api/fs.html#fspromisesrmpath-options