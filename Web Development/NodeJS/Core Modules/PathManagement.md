# Node.js Path Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `node:path` module provides utilities for working with file and directory paths, offering a consistent API for path manipulation across operating systems.

**Technical Definition:** The `node:path` module provides utilities for working with file and directory paths. It can be accessed using `const path = require('node:path')` or `import path from 'node:path`. The default operation of the module varies based on the operating system on which a Node.js application is running. Specifically, when running on Windows, the module assumes Windows-style paths; on POSIX systems (Linux, macOS), it assumes POSIX-style paths.

**Beginner-Friendly Explanation:** Think of `path` as a universal translator for file addresses. Windows uses backslashes (`C:\Users\file.txt`) while Linux and macOS use forward slashes (`/home/user/file.txt`). Instead of hardcoding slashes and hoping your code works on every computer, you ask `path` to build, split, or fix paths — and it automatically uses the right format for whatever operating system the code runs on.

### Key Characteristics

- **Platform-aware by default:** The module automatically adapts to the operating system's path conventions (Windows vs. POSIX).
- **Explicit platform overrides:** `path.win32` and `path.posix` provide consistent behavior regardless of the current OS.
- **Pure string manipulation:** Path methods do not access the file system; they only transform strings. Paths do not need to exist for the methods to work.
- **Normalization built in:** Most methods return normalized paths (resolving `.` and `..` segments) as part of their operation.
- **URL interoperability:** The `node:url` module's `fileURLToPath` bridges the gap between file URLs (used in ESM `import.meta.url`) and file-system paths.

### Prerequisites

- **Node.js runtime:** The `path` module is built into Node.js; no external installation is required. It has been stable since v0.10.0.
- **Basic JavaScript knowledge:** Understanding of strings and objects.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Awareness of file-system concepts:** Understanding what files, directories, and extensions are.

### Related Programming Areas

- **File System (`node:fs`):** Every `fs` operation that accepts a path can benefit from pre-processing with `path`.
- **URL handling (`node:url`):** `fileURLToPath` converts `file://` URLs to paths for use with `fs`.
- **ES Modules:** `import.meta.url` provides a `file://` URL that must be converted with `fileURLToPath` for path operations.
- **Build tools and bundlers:** Webpack, Vite, and similar tools rely heavily on path resolution.
- **CLI applications:** Parsing user-supplied paths and constructing output paths.

### Core Concepts

The following core concepts are covered in this cheat sheet:

1. **The `path` Module and `fileURLToPath`** — the module import and URL-to-path conversion.
2. **Joining vs. Resolving** — `path.join` and `path.resolve`.
3. **Extracting Path Components** — `path.extname`, `path.basename`, `path.dirname`.
4. **Normalizing and Parsing** — `path.normalize`, `path.parse`, `path.format`.
5. **Handling Platform Differences** — `path.sep`, `path.delimiter`, `path.win32`, `path.posix`.

---

## Core Concept 1: The `path` Module and `fileURLToPath`

### Definitions

**Core Definition:** The `node:path` module is imported to access path utilities, while `fileURLToPath` (from `node:url`) converts a `file://` URL — such as `import.meta.url` in ES modules — into a file-system path string.

**Technical Definition:** `import { fileURLToPath } from 'node:url'` is a function that takes a `URL` object or string and returns the equivalent file-system path. The `import.meta.url` property is the absolute `file:` URL of the current module, defined exactly the same as in browsers providing the URL of the current module file. `path.dirname(fileURLToPath(import.meta.url))` replicates the CommonJS `__dirname` in ES modules.

**Beginner-Friendly Explanation:** In CommonJS (`require`), you have `__dirname` — a variable that tells you which folder your script is in. In ES modules (`import`), that variable doesn't exist. Instead, you get `import.meta.url`, which looks like `file:///home/user/script.js`. The `fileURLToPath` function turns that URL into a regular path like `/home/user/script.js` so you can use it with `path` and `fs`.

### Purposes

- To import the `path` module for path manipulation operations.
- To convert `file://` URLs into file-system paths compatible with `fs` and `path` functions.
- To replicate `__dirname` and `__filename` in ES module environments.
- To construct relative paths from the current module's location in ESM.

### Syntax Rules and Structure

**CommonJS import:**
```js
const path = require('node:path');
```
| Component | Breakdown |
|-----------|-----------|
| `require` | CommonJS import function. |
| `'node:path'` | Built-in module specifier (the `node:` prefix is recommended). |
| Returns | The `path` module object. |

**ESM import:**
```js
import path from 'node:path';
import { fileURLToPath } from 'node:url';
```
| Component | Breakdown |
|-----------|-----------|
| `import path from 'node:path'` | Default import of the path module. |
| `fileURLToPath` | Named import from the URL module. |
| `import.meta.url` | A `file://` URL string of the current module. |

**Converting URL to path:**
```js
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```
| Component | Breakdown |
|-----------|-----------|
| `fileURLToPath(import.meta.url)` | Converts `file:///home/user/script.js` → `/home/user/script.js`. |
| `path.dirname(__filename)` | Extracts the directory portion. |

**Constraints and Limitations:**
- `fileURLToPath` is only meaningful for `file:` protocol URLs. Other protocols (e.g., `http:`) will produce incorrect results.
- `import.meta.url` is only available in ES modules, not in CommonJS.
- `import.meta.url` is `undefined` in modules loaded from non-file protocols.

### Annotated Code Example

```js
// esm-example.mjs — Must be run as an ES module
import path from 'node:path';
import { fileURLToPath } from 'node:url';

// import.meta.url is a file:// URL of THIS file
console.log('import.meta.url:', import.meta.url);
// Example: file:///home/user/projects/esm-example.mjs

// Convert the URL to a regular file path
const __filename = fileURLToPath(import.meta.url);
console.log('__filename:', __filename);
// Example: /home/user/projects/esm-example.mjs

// Derive the directory name (equivalent to __dirname in CommonJS)
const __dirname = path.dirname(__filename);
console.log('__dirname:', __dirname);
// Example: /home/user/projects

// Build a path to a sibling file
const configPath = path.join(__dirname, 'config.json');
console.log('configPath:', configPath);
// Example: /home/user/projects/config.json
```

**Expected Output (varies by machine):**
```
import.meta.url: file:///home/user/projects/esm-example.mjs
__filename: /home/user/projects/esm-example.mjs
__dirname: /home/user/projects
configPath: /home/user/projects/config.json
```

**Why this output:** `import.meta.url` provides the complete `file://` URL. `fileURLToPath` strips the `file://` prefix and decodes any URL-encoded characters, producing a standard file path. `path.dirname` then extracts the parent directory. The final `path.join` builds a cross-platform path to a sibling file.

### Real-World Cases

- **ESM configuration loading:** Reading a `config.json` file relative to the module using `fileURLToPath` + `path.join`.
- **CLI tools in ESM:** Resolving template directories relative to the script's location.
- **Migration from CommonJS to ESM:** Replacing `__dirname` with `path.dirname(fileURLToPath(import.meta.url))`.

---

## Core Concept 2: Joining Paths (`path.join`) vs. Resolving Absolute Paths (`path.resolve`)

### Definitions

**Core Definition:** `path.join` concatenates path segments with the platform-specific separator and normalizes the result, while `path.resolve` processes segments from right to left until an absolute path is constructed.

**Technical Definition:** The `path.join()` method joins all given path segments together using the platform-specific separator as a delimiter, then normalizes the resulting path. Zero-length path segments are ignored. The `path.resolve()` method resolves a sequence of paths or path segments into an absolute path. The given sequence is processed from right to left, with each subsequent path prepended until an absolute path is constructed.

**Beginner-Friendly Explanation:** `path.join` is like gluing pieces of a puzzle together — it takes fragments like `'folder'`, `'subfolder'`, and `'file.txt'` and connects them with the correct slash. `path.resolve` is like using a GPS — it doesn't just glue things together, it always gives you the complete, from-the-root-of-the-drive address. If it doesn't find a full address, it uses your current location (working directory) as the starting point.

### Purposes

- To combine path fragments into a single valid path for the current platform.
- To construct absolute paths from relative fragments.
- To normalize paths (resolve `.` and `..`) during construction.
- To ensure paths work correctly across Windows and POSIX systems.

### Syntax Rules and Structure

**`path.join`:**
```js
path.join(...paths);
```
| Component | Breakdown |
|-----------|-----------|
| `...paths` | Zero or more string segments to join. |
| Returns | A normalized path string using the platform separator. |
| Empty input | Returns `'.'` (current directory). |

**`path.resolve`:**
```js
path.resolve(...paths);
```
| Component | Breakdown |
|-----------|-----------|
| `...paths` | Zero or more path segments. |
| Returns | An absolute path string. |
| No segments | Returns the absolute path of the current working directory. |
| Right-to-left | Each segment is prepended until an absolute path is built. |

**Key difference:** `path.join` returns a normalized path but does not guarantee it is absolute. `path.resolve` always returns an absolute path, using the current working directory as a fallback when no absolute segment is found.

**Constraints and Limitations:**
- Both throw `TypeError` if any segment is not a string.
- `path.join` can produce relative paths; `path.resolve` never does.
- `path.resolve` depends on `process.cwd()` when no absolute path is provided — the result changes depending on where the script is executed.

### Multiple Annotated Code Examples

#### Example 1: Joining vs. Resolving

```js
// join-vs-resolve.js
const path = require('node:path');

// --- path.join: concatenates and normalizes ---
console.log('join 1:', path.join('/foo', 'bar', 'baz/asdf', 'quux', '..'));
// → '/foo/bar/baz/asdf'

console.log('join 2:', path.join('app/libs/oauth', '/../ssl'));
// → 'app/libs/ssl'  (relative path — does NOT become absolute)

// --- path.resolve: always absolute ---
console.log('resolve 1:', path.resolve('/foo/bar', './baz'));
// → '/foo/bar/baz'

console.log('resolve 2:', path.resolve('/foo/bar', '/tmp/file/'));
// → '/tmp/file'  (the absolute '/tmp/file/' resets the path)

console.log('resolve 3:', path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif'));
// → '/current/working/dir/wwwroot/static_files/gif/image.gif'
// (depends on process.cwd())
```

**Expected Output (cwd = `/home/user`):**
```
join 1: /foo/bar/baz/asdf
join 2: app/libs/ssl
resolve 1: /foo/bar/baz
resolve 2: /tmp/file
resolve 3: /home/user/wwwroot/static_files/gif/image.gif
```

**Why this output:** `path.join` concatenates segments and resolves `..` but keeps the result relative if the input is relative. `path.resolve` processes segments right-to-left. In `resolve 2`, `/tmp/file/` is already absolute, so everything to its left is discarded. In `resolve 3`, no segment is absolute, so the current working directory is prepended.

#### Example 2: Using with `__dirname` (CommonJS)

```js
// dirname-example.js
const path = require('node:path');

// Both produce the same result when the first argument is absolute
console.log(path.join(__dirname, 'src', 'index.js'));
console.log(path.resolve(__dirname, 'src', 'index.js'));

// Both return: /home/user/project/src/index.js
```

**Expected Output:**
```
/home/user/project/src/index.js
/home/user/project/src/index.js
```

**Why this output:** When `__dirname` (which is always absolute) is the first argument, both methods produce identical absolute results. The difference only becomes visible when starting from relative paths.

#### Example 3: A Critical Difference — The Leading Slash Trap

```js
// leading-slash-trap.js
const path = require('node:path');

const base = '/home/user/project';

// join: the leading slash in the second segment is treated as a separator
console.log('join:', path.join(base, '/public/assets'));
// → '/home/user/project/public/assets'

// resolve: the leading slash in the second segment makes it absolute
console.log('resolve:', path.resolve(base, '/public/assets'));
// → '/public/assets'  ← WARNING: base is discarded!
```

**Expected Output:**
```
join: /home/user/project/public/assets
resolve: /public/assets
```

**Why this output:** `path.join` simply concatenates and inserts a separator where needed, so `/public/assets` becomes a sub-path of `base`. `path.resolve` sees `/public/assets` as an absolute path and discards everything that came before it. This is a common source of bugs when building paths dynamically. When you want to append to a base path, prefer `path.join`; when you genuinely need an absolute path from possibly-relative pieces, use `path.resolve`.

### Real-World Cases

- **Static file serving (join):** `path.join(__dirname, 'public', req.url)` — safely appends the request path to the public directory.
- **Loading plugins (resolve):** Resolving a user-supplied plugin path to an absolute path before passing it to `require()`.
- **File uploads (join):** Constructing destination paths: `path.join(uploadDir, file.originalname)`.
- **Configuration (resolve):** Resolving relative config paths to absolute paths at application startup.

---

## Core Concept 3: Extracting File Extensions, Base Names, and Directory Names

### Definitions

**Core Definition:** These three functions extract individual components from a path string: the extension (`.txt`), the base name (`file.txt`), and the directory name (`/home/user`).

**Technical Definition:** The `path.basename()` method returns the last portion of a path, similar to the Unix `basename` command. The `path.dirname()` method returns the directory name of a path, similar to the Unix `dirname` command. The `path.extname()` method returns the extension of the path, from the last occurrence of the `.` character to the end of the string in the last portion of the path.

**Beginner-Friendly Explanation:** Given a full path like `/photos/vacation/beach.jpg`, you can ask three different questions: "What's the file name?" (`beach.jpg`), "What folder is it in?" (`/photos/vacation`), and "What type of file is it?" (`.jpg`). These three functions answer those questions.

### Purposes

- To extract the filename from a full path for display or logging.
- To determine the file extension to decide how to process a file (e.g., `.json` → parse as JSON).
- To obtain the parent directory of a file for relative operations.
- To strip known extensions from filenames to generate related output names.

### Syntax Rules and Structure

**`path.basename`:**
```js
path.basename(path, suffix?);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | The full path string. |
| `suffix` | Optional string to remove from the result (if present). |
| Returns | The last portion of the path. |

**`path.dirname`:**
```js
path.dirname(path);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | The full path string. |
| Returns | The directory portion of the path. |

**`path.extname`:**
```js
path.extname(path);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | The full path string. |
| Returns | The extension string (including the dot), or `''`. |

**Edge cases for `path.extname`:**
| Input | Output | Reason |
|-------|--------|--------|
| `'index.html'` | `'.html'` | Standard extension. |
| `'index.coffee.md'` | `'.md'` | Last extension only. |
| `'index.'` | `'.'` | Dot with nothing after it. |
| `'index'` | `''` | No dot at all. |
| `'.index'` | `''` | Dotfile — the dot is the first character of the basename. |
| `'.index.md'` | `'.md'` | Dotfile with a real extension. |

**Constraints and Limitations:**
- `path.basename` with a `suffix` is case-sensitive on Windows, even though Windows file names are generally case-insensitive.
- `path.extname` returns only the last extension for files like `archive.tar.gz` → `.gz`.
- These functions do not check whether the path actually exists.

### Annotated Code Example

```js
// path-components.js
const path = require('node:path');

const filePath = '/home/user/projects/notes/report.final.pdf';

// Extract the base name (with extension)
console.log('basename:', path.basename(filePath));
// → 'report.final.pdf'

// Extract the base name without a specific extension
console.log('basename (no .pdf):', path.basename(filePath, '.pdf'));
// → 'report.final'

// Extract the directory portion
console.log('dirname:', path.dirname(filePath));
// → '/home/user/projects/notes'

// Extract the extension
console.log('extname:', path.extname(filePath));
// → '.pdf'

// Edge cases
console.log('extname of .gitignore:', path.extname('.gitignore'));
// → ''

console.log('extname of archive.tar.gz:', path.extname('archive.tar.gz'));
// → '.gz'
```

**Expected Output:**
```
basename: report.final.pdf
basename (no .pdf): report.final
dirname: /home/user/projects/notes
extname: .pdf
extname of .gitignore: 
extname of archive.tar.gz: .gz
```

**Why this output:** `path.basename` returns everything after the last separator. Providing the `suffix` argument strips that exact suffix if present. `path.dirname` returns everything before the last separator. `path.extname` finds the last `.` in the final path segment; for `.gitignore` the dot is the first character of the basename, so no extension is reported. For `archive.tar.gz`, only `.gz` is returned.

### Real-World Cases

- **File type detection:** `const ext = path.extname(filePath); if (ext === '.json') { /* parse JSON */ }`.
- **Generating output names:** `const name = path.basename(filePath, '.md') + '.html'`.
- **Displaying file info:** Showing the filename in a UI without the full path: `path.basename(filePath)`.
- **Directory walking:** Using `path.dirname` to move up the directory tree during recursive operations.

---

## Core Concept 4: Normalizing and Parsing Path Strings

### Definitions

**Core Definition:** `path.normalize` cleans up a path by resolving `.` and `..` segments and collapsing redundant separators, while `path.parse` breaks a path into its logical components and `path.format` reassembles them.

**Technical Definition:** The `path.normalize()` method normalizes the given path, resolving `'..'` and `'.'` segments. When multiple sequential path separators are found, they are replaced by a single instance of the platform-specific separator. Trailing separators are preserved. The `path.parse()` method returns an object with properties representing the path's significant elements: `root`, `dir`, `base`, `ext`, and `name`. The `path.format()` method returns a path string from an object — the opposite of `path.parse()`.

**Beginner-Friendly Explanation:** Normalizing is like cleaning up a messy address: `/home//user/../user/docs/` becomes `/home/user/docs`. Parsing is like taking apart a LEGO model to see every individual piece. Formatting is like snapping the pieces back together.

### Purposes

- To clean up paths constructed from multiple sources (config files, user input, environment variables).
- To break a path into named components for inspection or manipulation.
- To reconstruct a path after modifying one of its components.
- To safely resolve `..` segments before passing a path to file-system operations.

### Syntax Rules and Structure

**`path.normalize`:**
```js
path.normalize(path);
```
| Component | Breakdown |
|-----------|-----------|
| `path` | The path string to normalize. |
| Returns | The normalized path string. |
| Empty string | Returns `'.'`. |

**`path.parse`:**
```js
path.parse(path);
```
| Property | Breakdown |
|----------|-----------|
| `root` | The root of the path (e.g., `/` or `C:\`). |
| `dir` | The directory path (from root to the parent of the file). |
| `base` | The file name including extension (e.g., `file.txt`). |
| `ext` | The file extension (e.g., `.txt`). |
| `name` | The file name without extension (e.g., `file`). |

**`path.format`:**
```js
path.format(pathObject);
```
| Component | Breakdown |
|-----------|-----------|
| `pathObject` | An object with `root`, `dir`, `base`, `ext`, `name` properties. |
| Priority | `dir` over `root`; `base` over `ext`+`name`. |
| Dot handling | A dot is added to `ext` if not already present. |

**Constraints and Limitations:**
- `path.normalize` does not strictly adhere to the POSIX specification for paths beginning with exactly two forward slashes (`//`), which some POSIX systems treat specially.
- `path.parse` always returns a `root` property (possibly an empty string).
- In `path.format`, if `base` exists, `name` and `ext` are ignored. If `dir` exists, `root` is ignored.

### Multiple Annotated Code Examples

#### Example 1: Normalizing Messy Paths

```js
// normalize-example.js
const path = require('node:path');

// Collapse redundant slashes and resolve dots
console.log(path.normalize('/foo/bar//baz/asdf/quux/..'));
// → '/foo/bar/baz/asdf'

console.log(path.normalize('foo/bar/../baz'));
// → 'foo/baz'

console.log(path.normalize(''));
// → '.'

// Windows example (using path.win32 for consistency)
console.log(path.win32.normalize('C:\\temp\\\\foo\\bar\\..\\'));
// → 'C:\\temp\\foo\\'
```

**Expected Output:**
```
/foo/bar/baz/asdf
foo/baz
.
C:\temp\foo\
```

**Why this output:** The `..` segments are resolved by removing the preceding segment. Multiple slashes are collapsed into one. An empty string normalizes to `'.'`. On Windows, `path.win32.normalize` converts mixed separators to backslashes and resolves the `..`.

#### Example 2: Parsing a Path into Components

```js
// parse-example.js
const path = require('node:path');

const parsed = path.parse('/home/user/dir/file.txt');
console.log(parsed);
/* Output:
{
  root: '/',
  dir: '/home/user/dir',
  base: 'file.txt',
  ext: '.txt',
  name: 'file'
}
*/

// Windows example
const winParsed = path.win32.parse('C:\\path\\dir\\file.txt');
console.log(winParsed);
/* Output:
{
  root: 'C:\\',
  dir: 'C:\\path\\dir',
  base: 'file.txt',
  ext: '.txt',
  name: 'file'
}
*/
```

**Expected Output:**
```
{ root: '/', dir: '/home/user/dir', base: 'file.txt', ext: '.txt', name: 'file' }
{ root: 'C:\\', dir: 'C:\\path\\dir', base: 'file.txt', ext: '.txt', name: 'file' }
```

**Why this output:** `path.parse` identifies the root (leading slash or drive letter), the directory portion, the base filename, the extension, and the name without extension.

#### Example 3: Rebuilding a Path with `path.format`

```js
// format-example.js
const path = require('node:path');

// Modify the name from 'report' to 'summary' while keeping everything else
const original = path.parse('/home/user/report.txt');
original.name = 'summary';
original.base = undefined;          // Let format rebuild from name + ext
const rebuilt = path.format(original);
console.log(rebuilt);
// → '/home/user/summary.txt'

// Build from scratch using name + ext
console.log(path.format({ root: '/', name: 'notes', ext: 'md' }));
// → '/notes.md'   (note: the dot is added automatically)

// Using dir + base has priority over root
console.log(path.format({
  root: '/ignored',
  dir: '/home/user/dir',
  base: 'file.txt',
}));
// → '/home/user/dir/file.txt'
```

**Expected Output:**
```
/home/user/summary.txt
/notes.md
/home/user/dir/file.txt
```

**Why this output:** `path.format` reassembles the object. When `base` is not provided, it uses `name` + `ext` and adds a dot if the extension lacks one. When `dir` is provided, `root` is ignored, and the platform separator is inserted between `dir` and `base`.

### Real-World Cases

- **Sanitising user input:** Normalizing a path supplied by a user before using it in file operations.
- **File renaming workflows:** Parsing a path, modifying the `name`, and formatting it back.
- **Configuration merging:** Combining base directories from different config sources with `path.normalize` to eliminate `..` and `//`.
- **Displaying structured file info:** Using `path.parse` to show a file's name, extension, and directory separately in a UI.

---

## Core Concept 5: Handling Platform Differences (POSIX vs. Windows)

### Definitions

**Core Definition:** Node.js provides `path.win32` and `path.posix` as explicit platform-specific implementations of the path module, along with `path.sep` and `path.delimiter` for platform-specific separator and delimiter characters.

**Technical Definition:** The `path.sep` property provides the platform-specific path segment separator: `\` on Windows and `/` on POSIX. The `path.delimiter` property provides the platform-specific path delimiter: `;` on Windows and `:` on POSIX. The `path.win32` and `path.posix` properties provide access to Windows-specific and POSIX-specific implementations of the path methods, respectively.

**Beginner-Friendly Explanation:** Windows and Linux/macOS disagree on how to write file paths: Windows uses `C:\Users\file.txt` while Linux uses `/home/user/file.txt`. Node.js normally picks the right format for your computer. But sometimes you need to handle paths from a different system — for example, a web server that receives Windows paths from a client. `path.win32` and `path.posix` let you say "treat this path as Windows" or "treat this path as POSIX" no matter what machine you're on.

### Purposes

- To detect the platform's path separator and delimiter programmatically.
- To process paths from a specific platform consistently, regardless of the host OS.
- To split environment variables like `PATH` into individual directories.
- To write cross-platform code that produces identical results on all systems.

### Syntax Rules and Structure

**`path.sep`:**
```js
path.sep; // '\' on Windows, '/' on POSIX
```
| Platform | Value |
|----------|-------|
| Windows | `'\\'` |
| POSIX | `'/'` |

**`path.delimiter`:**
```js
path.delimiter; // ';' on Windows, ':' on POSIX
```
| Platform | Value |
|----------|-------|
| Windows | `';'` |
| POSIX | `':'` |

**`path.win32` and `path.posix`:**
```js
path.win32.basename('C:\\temp\\myfile.html'); // 'myfile.html' on ALL platforms
path.posix.basename('/tmp/myfile.html');       // 'myfile.html' on ALL platforms
```

**Constraints and Limitations:**
- Using the wrong platform variant (e.g., `path.win32` on POSIX paths) will produce incorrect results.
- On Windows, both `/` and `\` are accepted as separators, but path methods only generate `\`. On POSIX, `\` is a valid filename character, not a separator.
- `path.win32` is not a replacement for actually testing on Windows; some behaviours (e.g., UNC paths) are OS-specific.

### Multiple Annotated Code Examples

#### Example 1: Splitting the PATH Environment Variable

```js
// path-delimiter.js
const path = require('node:path');

// Split PATH into individual directories
const pathDirs = process.env.PATH.split(path.delimiter);

console.log('Number of PATH entries:', pathDirs.length);
console.log('First entry:', pathDirs[0]);

// On POSIX, path.delimiter is ':'
// On Windows, path.delimiter is ';'
```

**Expected Output (POSIX):**
```
Number of PATH entries: 5
First entry: /usr/bin
```

**Expected Output (Windows):**
```
Number of PATH entries: 3
First entry: C:\Windows\system32
```

**Why this output:** The `PATH` environment variable contains multiple directory paths separated by the platform delimiter. Splitting on `path.delimiter` correctly separates them regardless of platform. On POSIX the delimiter is `:`, on Windows it is `;`.

#### Example 2: Cross-Platform Consistency with `path.posix` and `path.win32`

```js
// cross-platform.js
const path = require('node:path');

// The default path module produces different results on different OSes
console.log('Default basename:', path.basename('C:\\temp\\myfile.html'));
// On POSIX: 'C:\\temp\\myfile.html'  (backslash is not a separator)
// On Windows: 'myfile.html'

// path.win32 produces CONSISTENT results on ALL platforms
console.log('win32 basename:', path.win32.basename('C:\\temp\\myfile.html'));
// → 'myfile.html' on both POSIX and Windows

// path.posix produces CONSISTENT results on ALL platforms
console.log('posix basename:', path.posix.basename('/tmp/myfile.html'));
// → 'myfile.html' on both POSIX and Windows
```

**Expected Output (on POSIX):**
```
Default basename: C:\temp\myfile.html
win32 basename: myfile.html
posix basename: myfile.html
```

**Expected Output (on Windows):**
```
Default basename: myfile.html
win32 basename: myfile.html
posix basename: myfile.html
```

**Why this output:** On POSIX, backslash is a legal filename character, so the default `path.basename` does not treat `\` as a separator. Using `path.win32` forces Windows semantics regardless of the host OS, producing consistent results everywhere.

#### Example 3: Building Paths with `path.sep`

```js
// path-sep.js
const path = require('node:path');

// Manually building a path using the platform separator
const parts = ['users', 'alice', 'documents', 'report.txt'];
const built = parts.join(path.sep);

console.log('Built path:', built);
// POSIX: 'users/alice/documents/report.txt'
// Windows: 'users\\alice\\documents\\report.txt'

// Splitting a path into segments using path.sep
console.log('Split:', built.split(path.sep));
// POSIX: [ 'users', 'alice', 'documents', 'report.txt' ]
// Windows: [ 'users', 'alice', 'documents', 'report.txt' ]
```

**Expected Output (POSIX):**
```
Built path: users/alice/documents/report.txt
Split: [ 'users', 'alice', 'documents', 'report.txt' ]
```

**Why this output:** `path.sep` provides the correct separator for the current platform. Joining with `path.sep` and splitting with `path.sep` ensures round-trip consistency.

### Real-World Cases

- **Parsing server logs:** A Node.js log analyser on Linux that processes paths generated on Windows servers uses `path.win32` to correctly extract filenames.
- **Cross-platform CLI tools:** Ensuring that a CLI tool produces the same output regardless of whether it runs on Windows, macOS, or Linux.
- **Environment variable handling:** Splitting `PATH`, `NODE_PATH`, or other path-delimited variables with `path.delimiter`.
- **Testing:** Using `path.posix` in tests to assert consistent path behaviour without depending on the CI runner's OS.

---

## References

- Node.js Documentation — Path — https://nodejs.org/api/path.html
- Node.js Documentation — URL (`fileURLToPath`) — https://nodejs.org/api/url.html#urlfileurltopathurl
- Node.js Documentation — `import.meta` — https://nodejs.org/api/esm.html#importmeta
- Node.js Documentation — `path.join` — https://nodejs.org/api/path.html#pathjoinpaths
- Node.js Documentation — `path.resolve` — https://nodejs.org/api/path.html#pathresolvepaths
- Node.js Documentation — `path.basename` — https://nodejs.org/api/path.html#pathbasenamepath-suffix
- Node.js Documentation — `path.dirname` — https://nodejs.org/api/path.html#pathdirnamepath
- Node.js Documentation — `path.extname` — https://nodejs.org/api/path.html#pathextnamepath
- Node.js Documentation — `path.normalize` — https://nodejs.org/api/path.html#pathnormalizepath
- Node.js Documentation — `path.parse` — https://nodejs.org/api/path.html#pathparsepath
- Node.js Documentation — `path.format` — https://nodejs.org/api/path.html#pathformatpathobject
- Node.js Documentation — `path.sep` — https://nodejs.org/api/path.html#pathsep
- Node.js Documentation — `path.delimiter` — https://nodejs.org/api/path.html#pathdelimiter
- Node.js Documentation — `path.win32` — https://nodejs.org/api/path.html#pathwin32
- Node.js Documentation — `path.posix` — https://nodejs.org/api/path.html#pathposix
- Node.js Documentation — `path.isAbsolute` — https://nodejs.org/api/path.html#pathisabsolutepath
- Node.js Documentation — `path.relative` — https://nodejs.org/api/path.html#pathrelativefrom-to
- Stack Overflow — Difference between `path.join` and `path.resolve` — https://stackoverflow.com/questions/39110801/path-join-vs-path-resolve-with-dirname