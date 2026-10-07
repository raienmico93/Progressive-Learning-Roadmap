# Flask File Uploads: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** File uploads in Flask refer to the process of receiving binary files from a client via an HTTP request, parsing the multipart form data, and persisting the files to a storage destination (local disk, temporary directory, or object storage).

**Technical Definition:** When a client submits an HTML form with `enctype="multipart/form-data"`, the request body is encoded as a multipart stream containing both form fields and file parts. Flask's `Request` object, built on Werkzeug, parses this stream using `werkzeug.formparser.MultiPartParser`. Each file part is wrapped in a `FileStorage` instance and exposed via `request.files` (an `ImmutableMultiDict`). The `FileStorage` object provides access to the file's stream, filename, content type, and a `save()` method for persistence. Flask enforces size limits via the `MAX_CONTENT_LENGTH` configuration and provides `secure_filename()` for path sanitization.

**Beginner-Friendly Explanation:** When a user uploads a file through a web form, Flask receives it and gives you a special object called `FileStorage`. You can read the file, check its name and size, and save it to your server or to cloud storage. Flask also gives you tools to make sure the file is safe and not something malicious.

### Key Characteristics

- **Multipart encoding required:** The form must use `enctype="multipart/form-data"`; otherwise, files are not transmitted.
- **FileStorage objects:** Each uploaded file is wrapped in a `FileStorage` instance, which behaves like a standard Python file object with additional metadata.
- **ImmutableMultiDict:** `request.files` is an `ImmutableMultiDict`, supporting multiple files under the same field name via `getlist()`.
- **Size limits:** `MAX_CONTENT_LENGTH` limits the total request body size; exceeding it raises a `413 Request Entity Too Large` error.
- **Filename sanitization:** `secure_filename()` strips path separators and dangerous characters to prevent directory traversal.
- **Memory vs. disk:** Small files are stored in memory; larger files are spooled to temporary files on disk.
- **Streaming support:** `request.stream` allows chunked reading for large files without loading them entirely into memory.
- **MIME validation:** Extension-based checks can be spoofed; magic-number validation with libraries like `python-magic` provides stronger verification.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTML forms and the HTTP POST method.
- Familiarity with Flask routing and the `request` object.
- Optional: `pip install python-magic` for MIME type validation; `pip install boto3` for AWS S3 integration.

### Related Programming Areas

- **Cloud storage:** Uploading files to AWS S3, Google Cloud Storage, or Azure Blob Storage.
- **Security:** File uploads are a primary attack vector for directory traversal, XSS, and remote code execution.
- **Content delivery:** Serving uploaded files through CDNs or object storage.
- **Media processing:** Resizing images, transcoding videos, or parsing documents after upload.
- **Observability:** Logging upload metadata for auditing and monitoring.

### Core Concepts / Features

1. Multipart Form Data (Handling `multipart/form-data` Encoding Streams)
2. `request.files` and `FileStorage` (Iterating Over Uploaded File Streams)
3. File Validation (Sizes, MIME Types, and Magic Numbers)
4. Filename Handling (Sanitizing Paths to Mitigate Directory Traversal)
5. Upload Size Restrictions (`MAX_CONTENT_LENGTH` and `RequestEntityTooLarge`)
6. Secure Storage (Temporary Directories, Local Paths, and Object Storage)
7. Safe Filenames (`secure_filename` and UUID Re-naming Strategies)
8. Streaming and Chunking (Processing Large Transfers in Chunks)

---

## 1. Multipart Form Data (Handling `multipart/form-data` Encoding Streams)

### Definitions

**Core Definition:** `multipart/form-data` is the standard MIME encoding for submitting forms that contain files, where each form field (including files) is sent as a separate part within a single HTTP request body, separated by a boundary string.

**Technical Definition:** The `multipart/form-data` encoding is defined by RFC 7578. Each part has its own headers (including `Content-Disposition` with the field name and filename, and `Content-Type`) and a body containing the field's value or file bytes. The parts are separated by a boundary string specified in the request's `Content-Type` header. Werkzeug's `MultiPartParser` reads the stream, splits it at the boundaries, and creates `FileStorage` objects for file parts and regular form fields for non-file parts.

**Beginner-Friendly Explanation:** When a form includes a file, the browser packs everything—text fields and file contents—into a single package with dividers between each item. Flask unpacks this package and gives you the text fields in `request.form` and the files in `request.files`.

### Purposes

- To transmit binary file data alongside regular form fields in a single request.
- To support multiple files in one form submission.
- To allow each part to carry its own metadata (filename, content type).
- To provide a standardized, browser-supported mechanism for file uploads.
- To enable streaming parsing so that large files do not need to be fully buffered in memory.

### Syntax Rules and Structure

**Complete General Syntax (HTML):**

```html
<form method="POST" enctype="multipart/form-data">
    <input type="text" name="description">
    <input type="file" name="file">
    <button type="submit">Upload</button>
</form>
```

**Complete General Syntax (Flask):**

```python
from flask import request

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files["file"]
    description = request.form.get("description", "")
    # Process file and description
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `enctype="multipart/form-data"` | Required form attribute; without it, files are not transmitted |
| `request.files` | `ImmutableMultiDict` of `FileStorage` objects |
| `request.form` | `ImmutableMultiDict` of non-file form fields |
| `boundary` | String in `Content-Type` header separating parts |

**Syntax Rules:**

- The form's `enctype` must be `multipart/form-data`; the default `application/x-www-form-urlencoded` cannot transmit files.
- The HTTP method must be `POST` (or `PUT`/`PATCH`).
- Each file input must have a unique `name` attribute; multiple files can share the same name for list retrieval.
- Werkzeug limits the maximum memory size per form field via `MAX_FORM_MEMORY_SIZE` (default 500 kB).

**Constraints and Limitations:**

- Multipart parsing loads small files into memory and spools larger ones to disk.
- The total request body size is limited by `MAX_CONTENT_LENGTH`.
- The number of multipart parts is limited by `MAX_FORM_PARTS` (default 1000).
- Multipart requests cannot be parsed as JSON; use the appropriate content type for each.

### Annotated Code Examples

**Example 1: Basic Multipart Upload**

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

FORM = """
<form method="POST" enctype="multipart/form-data">
    <input type="text" name="description" placeholder="Description">
    <input type="file" name="file">
    <button type="submit">Upload</button>
</form>
"""

@app.route("/upload", methods=["GET", "POST"])
def upload():
    if request.method == "POST":
        file = request.files.get("file")
        description = request.form.get("description", "")
        if not file or file.filename == "":
            return "No file selected", 400
        return f"File: {file.filename}, Description: {description}"
    return render_template_string(FORM)

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /upload` with a file and description → `"File: report.pdf, Description: Monthly report"`
- `POST /upload` without a file → `"No file selected"` with status `400`

**Why this output:** The form's `enctype` ensures the file is transmitted. `request.files.get("file")` returns the `FileStorage` object, and `file.filename` gives the client-provided name. The form fields are accessible via `request.form`.

### Real-World Cases

- **Profile picture upload:** A form with a file input and a caption field.
- **Document management:** Uploading PDFs, Word documents, and spreadsheets.
- **Media galleries:** Uploading images and videos with metadata (title, tags).
- **Data import:** Uploading CSV or Excel files for batch processing.

### References

- RFC 7578: Returning Values from Forms: multipart/form-data — https://www.rfc-editor.org/rfc/rfc7578
- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/
- Werkzeug: Dealing with Request Data — https://werkzeug.palletsprojects.com/en/stable/request_data/

---

## 2. `request.files` and `FileStorage` (Iterating Over Uploaded File Streams)

### Definitions

**Core Definition:** `request.files` is a dictionary-like object that contains all uploaded files from a multipart request, with each file wrapped in a `FileStorage` instance that provides access to the file's data, metadata, and a `save()` method.

**Technical Definition:** `request.files` is an `ImmutableMultiDict` mapping form field names to `FileStorage` objects. The `FileStorage` class (from `werkzeug.datastructures`) is a thin wrapper around the incoming file stream. It proxies attributes like `read()`, `seek()`, and `close()` to the underlying stream, and adds `filename`, `name`, `content_type`, `headers`, `mimetype`, `content_length`, and `save(dst, buffer_size=16384)`. The `save()` method copies the stream to a destination path or file object in 16 KB chunks by default.

**Beginner-Friendly Explanation:** `request.files` is like a dictionary where the keys are the names of the file input fields, and the values are special file objects. You can iterate over all uploaded files, read their contents, check their filenames, and save them to disk.

### Purposes

- To access uploaded files by their form field name.
- To iterate over all uploaded files in a request.
- To read file contents without saving them to disk first.
- To save files to a destination using the optimized `save()` method.
- To access file metadata such as filename, content type, and size.
- To handle multiple files uploaded under the same field name.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import request

# Access a single file
file = request.files['field_name']
file = request.files.get('field_name')

# Access multiple files under the same name
files = request.files.getlist('field_name')

# Check if a file was uploaded
if 'field_name' in request.files:
    ...

# Iterate all uploaded files
for field_name in request.files:
    file = request.files[field_name]
    print(field_name, file.filename)

# FileStorage attributes
file.filename        # Client-provided filename
file.name            # Form field name
file.content_type    # MIME type from the client
file.mimetype        # MIME type without parameters
file.content_length  # Content-Length (may be None)
file.headers         # Multipart headers
file.stream          # Underlying file stream

# FileStorage methods
file.read(size)      # Read bytes from the stream
file.seek(offset)    # Seek in the stream
file.save(dst)       # Save to path or file object
file.close()         # Close the underlying stream
```

**Component Breakdown:**

| Attribute/Method | Description |
|------------------|-------------|
| `filename` | Client-provided filename (untrusted) |
| `name` | Form field name |
| `content_type` | Full MIME type from the client |
| `mimetype` | Lowercase MIME type without parameters |
| `content_length` | Content-Length of the file (often `None`) |
| `stream` | The underlying file stream |
| `save(dst, buffer_size=16384)` | Saves the file to a path or file object |
| `read(size=-1)` | Reads from the stream |
| `seek(offset, whence=0)` | Seeks in the stream |
| `close()` | Closes the stream |

**Syntax Rules:**

- `request.files` is populated only for `POST`, `PUT`, or `PATCH` requests with `multipart/form-data`.
- Accessing a missing field with `[]` raises `KeyError`; use `.get()` for safe access.
- `getlist()` returns an empty list if the field is absent.
- The `save()` method handles the copy efficiently, using a 16 KB buffer by default.
- After reading a file, use `seek(0)` to reset the stream position before saving or re-reading.

**Constraints and Limitations:**

- FileStorage objects are valid only during the request context; they cannot be used after the request ends.
- The `content_type` and `filename` attributes are client-provided and can be forged.
- Small files are stored in memory; large files are spooled to temporary files on disk.

### Annotated Code Examples

**Example 1: Accessing File Metadata**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file provided"}), 400
    
    return jsonify({
        "filename": file.filename,
        "field_name": file.name,
        "content_type": file.content_type,
        "mimetype": file.mimetype,
        "content_length": file.content_length
    })

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /upload` with a file → `{"filename": "report.pdf", "field_name": "file", "content_type": "application/pdf", "mimetype": "application/pdf", "content_length": null}`

**Why this output:** `FileStorage` exposes metadata about the uploaded file. `filename` and `content_type` are client-provided and may not reflect the actual file content. `content_length` is often `None` because the browser does not always send it.

**Example 2: Iterating Over Multiple Files**

```python
@app.route("/upload-multiple", methods=["POST"])
def upload_multiple():
    uploaded = []
    for field_name in request.files:
        files = request.files.getlist(field_name)
        for file in files:
            uploaded.append({
                "field": field_name,
                "filename": file.filename,
                "size": len(file.read())
            })
            file.seek(0)  # Reset for saving
    return jsonify({"uploaded": uploaded})
```

**Expected Output:**
- `POST /upload-multiple` with two files under `files` → `{"uploaded": [{"field": "files", "filename": "a.txt", "size": 100}, {"field": "files", "filename": "b.txt", "size": 200}]}`

**Why this output:** The view iterates over all field names in `request.files`, retrieves all files for each field with `getlist()`, and reads their sizes. The `seek(0)` call resets the stream so the file can be saved later.

### Real-World Cases

- **Multi-file upload:** Users uploading multiple images or documents at once.
- **Batch processing:** Uploading multiple CSV files for data import.
- **Gallery management:** Uploading multiple photos with individual metadata.

### References

- Flask API: `request.files` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.files
- Werkzeug `FileStorage` — https://werkzeug.palletsprojects.com/en/stable/datastructures/#werkzeug.datastructures.FileStorage
- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/

---

## 3. File Validation (Sizes, MIME Types, and Magic Numbers)

### Definitions

**Core Definition:** File validation is the process of verifying that an uploaded file meets the application's requirements for size, type, and content before it is stored or processed, protecting against malicious uploads and resource exhaustion.

**Technical Definition:** File validation encompasses three layers: (1) size validation, checking that the file does not exceed a configured maximum; (2) extension validation, checking that the filename ends with an allowed extension; and (3) content validation, inspecting the file's magic numbers (the first few bytes that identify the file format) using libraries like `python-magic` or manual signature checks. Extension and MIME-type checks based on client-provided data can be spoofed; magic-number validation inspects the actual file content.

**Beginner-Friendly Explanation:** You can't just trust that a file is what the user says it is. A file named `image.jpg` could actually be a malicious script. File validation checks the file's actual content (magic numbers), size, and extension to make sure it's safe before you store it.

### Purposes

- To prevent malicious files from being stored or executed on the server.
- To reject oversized files that could exhaust disk space or memory.
- To enforce business rules about which file types are acceptable.
- To prevent MIME type confusion attacks where a file's content does not match its extension.
- To comply with security best practices and regulatory requirements.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
import os
from werkzeug.utils import secure_filename

ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif', 'pdf'}
MAX_FILE_SIZE = 16 * 1024 * 1024  # 16 MB

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

def get_file_size(file):
    file.seek(0, os.SEEK_END)
    size = file.tell()
    file.seek(0)
    return size

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files.get("file")
    if not file or file.filename == "":
        return "No file selected", 400
    if not allowed_file(file.filename):
        return "File type not allowed", 400
    if get_file_size(file) > MAX_FILE_SIZE:
        return "File too large", 400
    # Save the file
    ...
```

**Magic Number Validation with `python-magic`:**

```python
import magic

ALLOWED_MIMES = {"image/png", "image/jpeg", "application/pdf"}

def validate_magic(file):
    header = file.read(2048)
    file.seek(0)
    detected = magic.from_buffer(header, mime=True)
    return detected in ALLOWED_MIMES
```

**Component Breakdown:**

| Validation Layer | Method | Reliability |
|------------------|--------|-------------|
| Extension | `filename.rsplit('.', 1)[1]` | Low (easily spoofed) |
| MIME type (client) | `file.content_type` | Low (client-controlled) |
| Magic numbers | `magic.from_buffer(header, mime=True)` | High (inspects content) |
| Size | `file.seek(0, os.SEEK_END)` | High (server-measured) |

**Syntax Rules:**

- Always validate the extension and the magic numbers; extension alone is insufficient.
- Read the first 2–4 KB of the file for magic-number detection.
- Reset the file stream with `seek(0)` after reading the header.
- Use a whitelist of allowed MIME types, not a blacklist.
- Combine size validation with `MAX_CONTENT_LENGTH` for defense in depth.

**Constraints and Limitations:**

- Magic-number validation is not foolproof; polyglot files can have valid headers for multiple formats.
- `python-magic` requires `libmagic` to be installed on the system.
- Some file formats (e.g., plain text) do not have reliable magic numbers.
- Content validation adds processing overhead; for large files, consider validating only the header.

### Annotated Code Examples

**Example 1: Extension and Size Validation**

```python
import os
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename

app = Flask(__name__)
app.config["MAX_CONTENT_LENGTH"] = 16 * 1024 * 1024  # 16 MB

ALLOWED_EXTENSIONS = {"png", "jpg", "jpeg", "gif", "pdf"}

def allowed_file(filename):
    return "." in filename and \
           filename.rsplit(".", 1)[1].lower() in ALLOWED_EXTENSIONS

def get_file_size(file):
    file.seek(0, os.SEEK_END)
    size = file.tell()
    file.seek(0)
    return size

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files.get("file")
    if not file or file.filename == "":
        return jsonify({"error": "No file selected"}), 400
    if not allowed_file(file.filename):
        return jsonify({"error": "File type not allowed"}), 400
    if get_file_size(file) > 16 * 1024 * 1024:
        return jsonify({"error": "File too large"}), 400
    filename = secure_filename(file.filename)
    file.save(os.path.join("/tmp/uploads", filename))
    return jsonify({"message": "Uploaded", "filename": filename}), 201

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /upload` with `image.png` → `{"message": "Uploaded", "filename": "image.png"}` with status `201`.
- `POST /upload` with `script.exe` → `{"error": "File type not allowed"}` with status `400`.

**Why this output:** The `allowed_file()` function checks the extension against a whitelist. `get_file_size()` measures the actual file size by seeking to the end. The file is saved only after both checks pass.

**Example 2: Magic Number Validation**

```python
import magic
from flask import Flask, request, jsonify

app = Flask(__name__)
ALLOWED_MIMES = {"image/png", "image/jpeg", "application/pdf"}

@app.route("/upload-secure", methods=["POST"])
def upload_secure():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    
    # Read header for magic number detection
    header = file.read(2048)
    file.seek(0)
    detected = magic.from_buffer(header, mime=True)
    
    if detected not in ALLOWED_MIMES:
        return jsonify({"error": f"Detected type {detected} not allowed"}), 400
    
    return jsonify({"detected_type": detected, "filename": file.filename})
```

**Expected Output:**
- `POST /upload-secure` with a real PNG → `{"detected_type": "image/png", "filename": "photo.png"}`
- `POST /upload-secure` with a PHP script renamed to `.png` → `{"error": "Detected type text/x-php not allowed"}` with status `400`.

**Why this output:** `magic.from_buffer()` inspects the file's actual bytes and returns the detected MIME type. A PHP script renamed to `.png` is detected as `text/x-php`, not `image/png`, and is rejected.

### Real-World Cases

- **Image uploads:** Validating that uploaded files are actual images, not scripts.
- **Document management:** Ensuring PDFs and Office documents are not executables.
- **Security-sensitive applications:** Preventing upload of malware or web shells.
- **Compliance:** Enforcing file type policies for regulatory requirements.

### References

- python-magic Documentation — https://github.com/ahupp/python-magic
- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/
- OWASP: Unrestricted File Upload — https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload

---

## 4. Filename Handling (Sanitizing Paths to Mitigate Directory Traversal)

### Definitions

**Core Definition:** Filename handling is the process of sanitizing client-provided filenames before using them on the server's filesystem, preventing directory traversal attacks where an attacker uses `../` sequences to write files outside the intended directory.

**Technical Definition:** The `werkzeug.utils.secure_filename()` function normalizes a filename by stripping path separators (`/` and `\`), removing non-ASCII characters, and replacing spaces and special characters with underscores. It ensures the resulting name is a safe, flat filename that cannot traverse directories. However, `secure_filename()` does not guarantee uniqueness; developers should combine it with a UUID or timestamp to prevent filename collisions and overwrites.

**Beginner-Friendly Explanation:** If you use the filename the user provides directly, an attacker could upload a file called `../../app.py` and overwrite your application code. `secure_filename()` strips out the dangerous parts and gives you a safe name like `app.py`.

### Purposes

- To prevent directory traversal attacks that write files outside the upload directory.
- To prevent overwriting existing files with malicious content.
- To ensure filenames are compatible with the server's filesystem.
- To eliminate characters that could cause parsing issues or security vulnerabilities.
- To provide a consistent naming convention for stored files.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from werkzeug.utils import secure_filename

# Basic sanitization
safe_name = secure_filename("../../etc/passwd")  # "etc_passwd"

# Combined with UUID for uniqueness
import uuid
safe_name = f"{uuid.uuid4().hex}_{secure_filename(file.filename)}"

# Save with sanitized name
file.save(os.path.join(UPLOAD_FOLDER, safe_name))
```

**Component Breakdown:**

| Function | Behavior |
|----------|----------|
| `secure_filename(name)` | Strips paths, removes non-ASCII, replaces unsafe chars |
| `uuid.uuid4().hex` | Generates a 32-character hexadecimal UUID |
| `os.path.join(base, name)` | Joins the base directory with the safe name |

**Syntax Rules:**

- Always apply `secure_filename()` before using a client-provided filename on the filesystem.
- Combine with a UUID or timestamp to prevent collisions and overwrites.
- Store files in a directory that is not directly executable and is outside the web root.
- Use `os.path.join()` with a trusted base directory to construct the final path.

**Constraints and Limitations:**

- `secure_filename()` may produce an empty string for filenames consisting entirely of unsafe characters; check for this case.
- Non-ASCII characters (e.g., Chinese, Arabic) are removed, not transliterated.
- The function does not validate the file's content; use MIME validation separately.
- `secure_filename()` does not prevent all attacks; it is one layer of defense.

### Annotated Code Examples

**Example 1: Basic Sanitization**

```python
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename
import os

app = Flask(__name__)
UPLOAD_FOLDER = "/tmp/uploads"

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    
    safe_name = secure_filename(file.filename)
    if not safe_name:
        return jsonify({"error": "Invalid filename"}), 400
    
    file.save(os.path.join(UPLOAD_FOLDER, safe_name))
    return jsonify({"stored_as": safe_name})
```

**Expected Output:**
- `POST /upload` with filename `report.pdf` → `{"stored_as": "report.pdf"}`
- `POST /upload` with filename `../../etc/passwd` → `{"stored_as": "etc_passwd"}`
- `POST /upload` with filename `..` → `{"error": "Invalid filename"}` with status `400`.

**Why this output:** `secure_filename()` strips the path traversal sequences and returns a safe name. If the result is empty (e.g., for `..`), the view returns an error.

**Example 2: UUID Re-naming for Uniqueness**

```python
import uuid
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename
import os

app = Flask(__name__)
UPLOAD_FOLDER = "/tmp/uploads"

@app.route("/upload-uuid", methods=["POST"])
def upload_uuid():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    
    original = secure_filename(file.filename)
    ext = original.rsplit(".", 1)[1].lower() if "." in original else ""
    unique_name = f"{uuid.uuid4().hex}.{ext}" if ext else uuid.uuid4().hex
    
    file.save(os.path.join(UPLOAD_FOLDER, unique_name))
    return jsonify({"stored_as": unique_name, "original": file.filename})
```

**Expected Output:**
- `POST /upload-uuid` with `report.pdf` → `{"stored_as": "a1b2c3d4e5f6...pdf", "original": "report.pdf"}`

**Why this output:** The original filename is sanitized, its extension extracted, and a UUID-based name is generated. This prevents collisions and overwrites while preserving the file extension for content-type inference.

### Real-World Cases

- **User avatars:** Storing avatars with UUID names to prevent collisions.
- **Document uploads:** Sanitizing filenames before storing in a shared upload directory.
- **Multi-tenant applications:** Using tenant-specific prefixes combined with sanitized names.
- **CDN integration:** Generating unique names for files served through a CDN.

### References

- Werkzeug `secure_filename` — https://werkzeug.palletsprojects.com/en/stable/utils/#werkzeug.utils.secure_filename
- OWASP: Path Traversal — https://owasp.org/www-community/attacks/Path_Traversal
- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/

---

## 5. Upload Size Restrictions (`MAX_CONTENT_LENGTH` and `RequestEntityTooLarge`)

### Definitions

**Core Definition:** `MAX_CONTENT_LENGTH` is a Flask configuration that limits the maximum size of an incoming request body in bytes. When a client sends a request larger than this limit, Flask raises a `413 Request Entity Too Large` error before reading the full body into memory.

**Technical Definition:** `MAX_CONTENT_LENGTH` is mapped to `Request.max_content_length` in Werkzeug. When the `Content-Length` header of an incoming request exceeds this value, Werkzeug raises a `RequestEntityTooLarge` exception (HTTP 413). Flask's default error handling returns an HTML error page; developers can register a custom error handler to return JSON or a custom template. The limit can be overridden per-request by setting `request.max_content_length` before the body is read.

**Beginner-Friendly Explanation:** You can tell Flask "don't accept any request larger than 16 MB." If someone tries to upload a 100 MB file, Flask rejects it with a 413 error before it wastes memory or disk space.

### Purposes

- To prevent denial-of-service attacks through oversized uploads.
- To protect server memory and disk resources.
- To enforce reasonable limits on user input size.
- To provide clear error responses when limits are exceeded.
- To allow different endpoints to have different size limits.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)
app.config["MAX_CONTENT_LENGTH"] = 16 * 1024 * 1024  # 16 MB

@app.errorhandler(413)
def request_entity_too_large(error):
    return jsonify({"error": "File too large", "max_size": "16 MB"}), 413

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files.get("file")
    # File is guaranteed to be within the limit
    ...
```

**Component Breakdown:**

| Configuration | Description |
|---------------|-------------|
| `MAX_CONTENT_LENGTH` | Maximum total request body size in bytes |
| `request.max_content_length` | Per-request override |
| `413 Request Entity Too Large` | HTTP status raised when the limit is exceeded |
| `@app.errorhandler(413)` | Custom handler for 413 errors |

**Syntax Rules:**

- Set `MAX_CONTENT_LENGTH` in bytes; use multiplication for readability (e.g., `16 * 1024 * 1024`).
- The limit applies to the entire request body, including all files and form fields.
- `MAX_CONTENT_LENGTH` can be overridden per-request by setting `request.max_content_length` in a `before_request` handler.
- The `MAX_FORM_MEMORY_SIZE` (default 500 kB) limits individual form fields; `MAX_FORM_PARTS` (default 1000) limits the number of multipart parts.

**Constraints and Limitations:**

- `MAX_CONTENT_LENGTH` is not set by default; the WSGI server may impose its own limits.
- Setting the limit too low may reject legitimate large files.
- The limit is based on the `Content-Length` header, which can be spoofed; the actual body size is also enforced.
- In some Flask/Werkzeug versions, `MAX_CONTENT_LENGTH` may not be honored consistently with certain WSGI servers.

### Annotated Code Examples

**Example 1: Global Limit with Custom Error Handler**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)
app.config["MAX_CONTENT_LENGTH"] = 5 * 1024 * 1024  # 5 MB

@app.errorhandler(413)
def request_entity_too_large(error):
    return jsonify({
        "error": "Payload too large",
        "max_size": "5 MB"
    }), 413

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    return jsonify({"filename": file.filename, "size_ok": True})

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /upload` with a 2 MB file → `{"filename": "photo.jpg", "size_ok": true}`
- `POST /upload` with a 10 MB file → `{"error": "Payload too large", "max_size": "5 MB"}` with status `413`.

**Why this output:** Flask checks the `Content-Length` header against `MAX_CONTENT_LENGTH` before reading the body. If the limit is exceeded, a 413 error is raised and handled by the custom error handler, which returns a JSON response.

**Example 2: Per-Request Limit Override**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)
app.config["MAX_CONTENT_LENGTH"] = 5 * 1024 * 1024  # Global 5 MB

@app.route("/upload-large", methods=["POST"])
def upload_large():
    # Allow larger uploads for this endpoint
    request.max_content_length = 50 * 1024 * 1024  # 50 MB
    file = request.files.get("file")
    return jsonify({"filename": file.filename})
```

**Expected Output:**
- `POST /upload-large` with a 20 MB file → `{"filename": "video.mp4"}`
- `POST /upload-large` with a 60 MB file → `413 Request Entity Too Large`

**Why this output:** Setting `request.max_content_length` overrides the global configuration for this specific request, allowing larger payloads for this endpoint while keeping the default limit strict for others.

### Real-World Cases

- **Media uploads:** Allowing larger limits for video/audio uploads while keeping strict limits for other endpoints.
- **Public APIs:** Enforcing small limits to prevent abuse.
- **Internal services:** Setting generous limits for trusted internal clients.
- **Batch operations:** Allowing larger payloads for bulk data import.

### References

- Flask Configuration: `MAX_CONTENT_LENGTH` — https://flask.palletsprojects.com/en/stable/config/#MAX_CONTENT_LENGTH
- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Werkzeug `RequestEntityTooLarge` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.RequestEntityTooLarge

---

## 6. Secure Storage (Temporary Directories, Local Paths, and Object Storage)

### Definitions

**Core Definition:** Secure storage refers to the strategies and practices for persisting uploaded files safely, including using temporary directories for processing, local persistent paths with proper permissions, or direct streaming to object storage services like AWS S3.

**Technical Definition:** Uploaded files can be stored in three primary locations: (1) a temporary directory (via `tempfile.NamedTemporaryFile` or `tempfile.mkdtemp`) for short-lived processing; (2) a local persistent path on the server's filesystem, configured via `UPLOAD_FOLDER`; or (3) an object storage service (AWS S3, Google Cloud Storage, Azure Blob Storage) via streaming uploads. For object storage, the `FileStorage.stream` attribute can be passed directly to the storage client's upload method, avoiding local disk writes.

**Beginner-Friendly Explanation:** After receiving an uploaded file, you need to decide where to put it. You can save it to a temporary folder if you just need to process it, save it to a permanent folder on your server, or send it directly to cloud storage like Amazon S3. Each option has different trade-offs for security, scalability, and cost.

### Purposes

- To persist uploaded files for later retrieval and processing.
- To isolate uploaded files from the application's executable code.
- To scale storage beyond a single server's disk capacity.
- To leverage cloud storage's durability, availability, and CDN integration.
- To avoid disk space exhaustion on the application server.
- To provide secure, time-limited access to uploaded files.

### Syntax Rules and Structure

**Temporary Directory (Processing Only):**

```python
import tempfile
from flask import request

@app.route("/process", methods=["POST"])
def process():
    file = request.files["file"]
    with tempfile.NamedTemporaryFile(delete=False, suffix=".pdf") as tmp:
        file.save(tmp.name)
        # Process tmp.name
    # File is deleted when process exits (if delete=True)
```

**Local Persistent Path:**

```python
import os
from flask import request, current_app

UPLOAD_FOLDER = "/var/uploads"
app.config["UPLOAD_FOLDER"] = UPLOAD_FOLDER
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files["file"]
    file.save(os.path.join(app.config["UPLOAD_FOLDER"], secure_filename(file.filename)))
```

**Object Storage (AWS S3):**

```python
import boto3
from flask import request

s3 = boto3.client("s3")

@app.route("/upload-s3", methods=["POST"])
def upload_s3():
    file = request.files["file"]
    s3.upload_fileobj(
        file.stream,
        "my-bucket",
        f"uploads/{file.filename}",
        ExtraArgs={"ContentType": file.content_type}
    )
    return "Uploaded to S3"
```

**Component Breakdown:**

| Storage Strategy | Use Case | Pros | Cons |
|------------------|----------|------|------|
| Temporary directory | Processing, then discard | Automatic cleanup, isolated | Not persistent |
| Local persistent path | Single-server deployments | Simple, fast access | Limited by disk, not scalable |
| Object storage (S3) | Multi-server, scalable apps | Unlimited capacity, durable | Network latency, cost |

**Syntax Rules:**

- Temporary files should be created with `NamedTemporaryFile(delete=False)` if the path needs to be accessed outside the `with` block.
- Local upload folders should be outside the web root and not executable.
- For S3, use `upload_fileobj()` with `file.stream` to stream directly without loading the entire file into memory.
- Set appropriate permissions on local upload directories (e.g., `0o755` for directories, `0o644` for files).

**Constraints and Limitations:**

- Temporary files must be cleaned up manually if `delete=False` is used.
- Local storage is not suitable for multi-instance deployments without a shared filesystem.
- Object storage uploads may fail due to network issues; implement retry logic.
- S3 uploads from a Flask server consume bandwidth and may block the request thread.

### Annotated Code Examples

**Example 1: Temporary File for Processing**

```python
import tempfile
import os
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/process-pdf", methods=["POST"])
def process_pdf():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    
    # Save to a temporary file for processing
    with tempfile.NamedTemporaryFile(delete=False, suffix=".pdf") as tmp:
        file.save(tmp.name)
        tmp_path = tmp.name
    
    try:
        # Process the file (e.g., extract text)
        with open(tmp_path, "rb") as f:
            content = f.read(100)
        return jsonify({"processed_bytes": len(content)})
    finally:
        os.unlink(tmp_path)  # Clean up
```

**Expected Output:**
- `POST /process-pdf` with a PDF → `{"processed_bytes": 100}`

**Why this output:** The file is saved to a temporary location, processed, and then deleted. The `delete=False` parameter ensures the file persists after the `with` block, allowing processing outside it.

**Example 2: Streaming to AWS S3**

```python
import boto3
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename

app = Flask(__name__)
s3 = boto3.client("s3", region_name="us-east-1")
BUCKET = "my-upload-bucket"

@app.route("/upload-s3", methods=["POST"])
def upload_s3():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    
    safe_name = secure_filename(file.filename)
    key = f"uploads/{safe_name}"
    
    try:
        s3.upload_fileobj(
            file.stream,
            BUCKET,
            key,
            ExtraArgs={"ContentType": file.content_type or "application/octet-stream"}
        )
        return jsonify({"uploaded": key, "bucket": BUCKET}), 201
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

**Expected Output:**
- `POST /upload-s3` with a file → `{"uploaded": "uploads/report.pdf", "bucket": "my-upload-bucket"}` with status `201`.

**Why this output:** `file.stream` is passed directly to `s3.upload_fileobj()`, which streams the file to S3 in chunks without loading it entirely into memory. The `ContentType` is set from the client-provided content type.

### Real-World Cases

- **Image processing pipelines:** Upload to temporary storage, resize, then upload to S3.
- **Document management systems:** Store files in S3 with metadata in a database.
- **User-generated content:** Store avatars, attachments, and media in object storage.
- **Data import:** Upload CSV files temporarily, process them, then discard.

### References

- Python `tempfile` Documentation — https://docs.python.org/3/library/tempfile.html
- boto3 S3 `upload_fileobj` — https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3/client/upload_fileobj.html
- AWS S3 Multipart Upload — https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html
- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/

---

## 7. Safe Filenames (`secure_filename` and UUID Re-naming Strategies)

### Definitions

**Core Definition:** Safe filenames are filenames that have been sanitized to remove path separators, special characters, and other dangerous content, and made unique to prevent collisions and overwrites.

**Technical Definition:** `werkzeug.utils.secure_filename()` performs a series of transformations: it normalizes Unicode to ASCII, removes non-ASCII characters, replaces spaces and path separators with underscores, and strips leading/trailing dots and underscores. The result is a flat filename safe for use on the filesystem. Because `secure_filename()` does not guarantee uniqueness, developers should prefix a UUID (e.g., `uuid.uuid4().hex`) or timestamp to create a unique, collision-resistant filename.

**Beginner-Friendly Explanation:** `secure_filename()` cleans up a filename so it can't be used to attack your server. But two users could upload `photo.jpg` and one would overwrite the other. Adding a UUID makes each filename unique, so no file is ever overwritten.

### Purposes

- To prevent directory traversal attacks via filenames.
- To prevent filename collisions and accidental overwrites.
- To ensure filenames are compatible with the filesystem (no special characters).
- To provide a predictable, safe naming scheme for stored files.
- To preserve the original file extension for content-type inference.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from werkzeug.utils import secure_filename
import uuid

# Basic sanitization
safe_name = secure_filename(file.filename)

# UUID prefix for uniqueness (preserves extension)
original = secure_filename(file.filename)
ext = original.rsplit(".", 1)[1].lower() if "." in original else ""
unique_name = f"{uuid.uuid4().hex}.{ext}" if ext else uuid.uuid4().hex

# Save with the unique name
file.save(os.path.join(UPLOAD_FOLDER, unique_name))
```

**Component Breakdown:**

| Function | Behavior |
|----------|----------|
| `secure_filename()` | Sanitizes the filename; strips paths and special chars |
| `uuid.uuid4().hex` | Generates a 32-character hexadecimal UUID |
| Extension extraction | `filename.rsplit(".", 1)[1]` gets the extension |

**Syntax Rules:**

- Always sanitize the filename with `secure_filename()` before use.
- Extract the extension from the sanitized name, not the original (to avoid malicious extensions).
- Use `uuid.uuid4().hex` for a collision-resistant unique identifier.
- Store the original filename in a database if you need to display it to users.
- The sanitized name may be an empty string; check for this case.

**Constraints and Limitations:**

- `secure_filename()` removes non-ASCII characters, which may make filenames unrecognizable for international users.
- The UUID-based name hides the original filename; store it separately if needed.
- Very long filenames may exceed filesystem limits (typically 255 characters).

### Annotated Code Examples

**Example 1: Sanitization and UUID Combination**

```python
import os
import uuid
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename

app = Flask(__name__)
UPLOAD_FOLDER = "/tmp/uploads"

def safe_unique_name(original_filename):
    """Generate a safe, unique filename preserving the extension."""
    safe = secure_filename(original_filename)
    if not safe:
        return None
    if "." in safe:
        ext = safe.rsplit(".", 1)[1].lower()
        return f"{uuid.uuid4().hex}.{ext}"
    return uuid.uuid4().hex

@app.route("/upload-safe", methods=["POST"])
def upload_safe():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "No file"}), 400
    
    unique_name = safe_unique_name(file.filename)
    if not unique_name:
        return jsonify({"error": "Invalid filename"}), 400
    
    file.save(os.path.join(UPLOAD_FOLDER, unique_name))
    return jsonify({
        "stored_as": unique_name,
        "original": file.filename
    })
```

**Expected Output:**
- `POST /upload-safe` with `../../etc/passwd` → `{"stored_as": "a1b2c3d4e5f6...", "original": "../../etc/passwd"}`
- `POST /upload-safe` with `photo.jpg` → `{"stored_as": "f7e8d9c0b1a2...jpg", "original": "photo.jpg"}`

**Why this output:** `secure_filename()` strips the path traversal, and `uuid.uuid4().hex` generates a unique name. The original filename is returned for reference but is never used on the filesystem.

### Real-World Cases

- **User avatars:** Storing avatars with UUID names in a public folder.
- **Document uploads:** Storing documents with UUID names and a database mapping.
- **Media libraries:** Storing images and videos with unique names for CDN delivery.
- **Multi-tenant SaaS:** Prefixing UUID names with tenant identifiers.

### References

- Werkzeug `secure_filename` — https://werkzeug.palletsprojects.com/en/stable/utils/#werkzeug.utils.secure_filename
- OWASP: Path Traversal — https://owasp.org/www-community/attacks/Path_Traversal
- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/

---

## 8. Streaming and Chunking (Processing Large Transfers in Chunks)

### Definitions

**Core Definition:** Streaming and chunking are techniques for handling large file uploads by reading and processing the request body in small, fixed-size chunks rather than loading the entire file into memory at once.

**Technical Definition:** Flask's `request.stream` provides a file-like object that yields raw bytes from the request body as they arrive from the client. By reading from `request.stream` in a loop with a fixed chunk size (e.g., 4096 or 8192 bytes), developers can write the data directly to a destination (disk, S3) without buffering the entire payload. This bypasses Flask's default multipart parsing, which loads files into memory or spools them to disk based on size. For true streaming with multipart data, a streaming parser such as `streaming-form-data` is required.

**Beginner-Friendly Explanation:** Instead of waiting for an entire 2 GB file to arrive and then saving it, streaming lets you save the file in small pieces as it arrives. This keeps memory usage low and allows the upload to start processing immediately.

### Purposes

- To handle uploads larger than available server memory.
- To reduce memory usage and prevent memory exhaustion attacks.
- To start processing large files before the upload is complete.
- To stream uploads directly to object storage (S3, GCS) without local disk writes.
- To enable resumable uploads and progress tracking.

### Syntax Rules and Structure

**Complete General Syntax (Raw Streaming):**

```python
from flask import request

@app.route("/upload-stream", methods=["POST"])
def upload_stream():
    with open("/tmp/output_file", "wb") as f:
        chunk_size = 4096
        while True:
            chunk = request.stream.read(chunk_size)
            if not chunk:
                break
            f.write(chunk)
    return "Uploaded"
```

**Complete General Syntax (Streaming to S3):**

```python
import boto3
from flask import request

s3 = boto3.client("s3")

@app.route("/upload-stream-s3", methods=["POST"])
def upload_stream_s3():
    s3.upload_fileobj(
        request.stream,
        "my-bucket",
        "uploads/large-file.bin"
    )
    return "Uploaded to S3"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `request.stream` | File-like object yielding raw request body bytes |
| `chunk_size` | Number of bytes to read per iteration (e.g., 4096, 8192) |
| `while True` loop | Reads chunks until the stream is exhausted |
| `f.write(chunk)` | Writes each chunk to the destination |
| `upload_fileobj(stream, ...)` | Streams directly to S3 without buffering |

**Syntax Rules:**

- `request.stream` reads the raw request body; it does not parse multipart boundaries.
- The chunk size should be a power of 2 (e.g., 4096, 8192, 16384) for efficiency.
- For multipart streaming, use a streaming parser like `streaming-form-data`.
- Always close the destination file in a `finally` block or use a context manager.
- Disable Flask's automatic request parsing by accessing `request.stream` before `request.form` or `request.files`.

**Constraints and Limitations:**

- `request.stream` yields raw bytes; for multipart data, you must parse the boundaries yourself.
- Flask's `request.files` loads files into memory; true streaming bypasses this.
- Streaming uploads cannot be resumed easily without additional protocol support (e.g., tus).
- Error handling is more complex with streaming; partial writes must be cleaned up.

### Annotated Code Examples

**Example 1: Raw Streaming to Disk**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/upload-stream", methods=["POST"])
def upload_stream():
    chunk_size = 4096
    total_bytes = 0
    
    with open("/tmp/streamed_upload.bin", "wb") as f:
        while True:
            chunk = request.stream.read(chunk_size)
            if not chunk:
                break
            f.write(chunk)
            total_bytes += len(chunk)
    
    return jsonify({"bytes_written": total_bytes})

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /upload-stream` with a 10 MB body → `{"bytes_written": 10485760}` (approximately).

**Why this output:** `request.stream.read(chunk_size)` reads the body in 4 KB chunks. Each chunk is written to the file immediately, keeping memory usage constant regardless of the total file size.

**Example 2: Streaming to S3 with Boto3**

```python
import boto3
from flask import Flask, request, jsonify

app = Flask(__name__)
s3 = boto3.client("s3", region_name="us-east-1")

@app.route("/upload-s3-stream", methods=["POST"])
def upload_s3_stream():
    try:
        s3.upload_fileobj(
            request.stream,
            "my-upload-bucket",
            "streamed/large-file.bin",
            ExtraArgs={"ContentType": "application/octet-stream"}
        )
        return jsonify({"status": "uploaded"}), 201
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

**Expected Output:**
- `POST /upload-s3-stream` with a large file → `{"status": "uploaded"}` with status `201`.

**Why this output:** `request.stream` is passed directly to `s3.upload_fileobj()`, which streams the data to S3 in chunks. No local disk writes occur, and memory usage remains low.

### Real-World Cases

- **Video uploads:** Streaming large video files to S3 without local storage.
- **Data imports:** Streaming large CSV files to a processing pipeline.
- **Backup systems:** Uploading large backup files to cloud storage.
- **Real-time data:** Streaming sensor data or logs to storage as they arrive.

### References

- Flask API: `request.stream` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.stream
- Werkzeug: Dealing with Request Data — https://werkzeug.palletsprojects.com/en/stable/request_data/
- boto3 S3 `upload_fileobj` — https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3/client/upload_fileobj.html
- streaming-form-data — https://github.com/siddhantgoel/streaming-form-data
- Stack Overflow: Flask streaming file upload — https://stackoverflow.com/questions/44727052/handling-large-file-uploads-with-flask

---

## References

- Flask Patterns: File Uploads — https://flask.palletsprojects.com/en/stable/patterns/fileuploads/
- Flask API: `request.files` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.files
- Flask API: `request.stream` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.stream
- Flask Configuration: `MAX_CONTENT_LENGTH` — https://flask.palletsprojects.com/en/stable/config/#MAX_CONTENT_LENGTH
- Flask Configuration: `MAX_FORM_MEMORY_SIZE` — https://flask.palletsprojects.com/en/stable/config/#MAX_FORM_MEMORY_SIZE
- Flask Configuration: `MAX_FORM_PARTS` — https://flask.palletsprojects.com/en/stable/config/#MAX_FORM_PARTS
- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Werkzeug `FileStorage` — https://werkzeug.palletsprojects.com/en/stable/datastructures/#werkzeug.datastructures.FileStorage
- Werkzeug `secure_filename` — https://werkzeug.palletsprojects.com/en/stable/utils/#werkzeug.utils.secure_filename
- Werkzeug: Dealing with Request Data — https://werkzeug.palletsprojects.com/en/stable/request_data/
- RFC 7578: multipart/form-data — https://www.rfc-editor.org/rfc/rfc7578
- OWASP: Unrestricted File Upload — https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
- OWASP: Path Traversal — https://owasp.org/www-community/attacks/Path_Traversal
- python-magic Documentation — https://github.com/ahupp/python-magic
- boto3 S3 `upload_fileobj` — https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3/client/upload_fileobj.html
- AWS S3 Multipart Upload — https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html
- Python `tempfile` Documentation — https://docs.python.org/3/library/tempfile.html
- streaming-form-data — https://github.com/siddhantgoel/streaming-form-data
- Stack Overflow: Flask streaming file upload — https://stackoverflow.com/questions/44727052/handling-large-file-uploads-with-flask
- Stack Overflow: Upload large file to S3 through Flask — https://stackoverflow.com/questions/78593090/upload-large-file-to-s3-through-a-flask-rest-api-dont-want-to-use-s3-presigned-url