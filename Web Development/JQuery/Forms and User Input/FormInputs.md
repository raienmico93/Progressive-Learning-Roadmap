# jQuery File Inputs and AJAX Uploads: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
File Inputs and AJAX Uploads refers to the process of selecting files via `<input type="file">`, accessing their metadata through the native File API, packaging them for transmission with `FormData`, and sending them asynchronously to a server using jQuery's `$.ajax()`.

**Technical Definition**
The HTML File API exposes selected files as `File` objects within a `FileList` collection. Each `File` object provides read-only metadata properties: `name` (filename), `size` (bytes), `type` (MIME type), and `lastModified` (timestamp) . The `FormData` interface constructs a `multipart/form-data` payload that can include both text fields and binary files . When uploading via jQuery AJAX, the `processData: false` and `contentType: false` settings prevent jQuery from transforming the `FormData` object into a URL-encoded string . Upload progress is monitored through a custom `xhr` function that attaches an `upload.addEventListener("progress", ...)` listener .

**Beginner-Friendly Explanation**
When users upload files, the browser gives JavaScript access to information about those files (name, size, type). To send them to a server without reloading the page, you package them using `FormData`, then send them via AJAX. jQuery needs special settings to handle files properly. You can also show a progress bar so users see how the upload is going.

### Key Characteristics

- **Metadata Access**: File API exposes `name`, `size`, `type`, and `lastModified` on each `File` object .
- **FormData Required**: `FormData` is the only standard way to send files via AJAX; `.serialize()` cannot handle binary data .
- **Special AJAX Settings**: `processData: false` and `contentType: false` are mandatory for `FormData` uploads .
- **Progress Monitoring**: `xhr.upload` provides `progress` events for real-time upload percentage .
- **MIME Type Unreliability**: `File.type` is based on extension and can be spoofed; server-side validation must sniff file signatures .

### Prerequisites

- Basic HTML and JavaScript knowledge.
- jQuery library included via CDN or local file.
- Understanding of AJAX and HTTP requests.
- Familiarity with the File API and `FormData`.

### Related Programming Areas

- **File API**: Native browser interface for accessing file metadata and content .
- **FormData**: Interface for constructing `multipart/form-data` payloads .
- **AJAX**: Asynchronous HTTP requests via `$.ajax()` .
- **Server-Side Validation**: Security checks that must be performed on the server .

### Core Concepts / Features

1. Accessing File Metadata via Native `File` Objects
2. Using the Native `FormData` API with jQuery
3. AJAX Configuration for Files
4. Upload Progress Monitoring
5. Server-Side Validation Requirements

---

## Core Concept 1: Accessing File Metadata via Native `File` Objects

### Definitions

**Core Definition**
The `File` object represents a file selected by the user and exposes read-only metadata properties describing its name, size, type, and last-modified date.

**Technical Definition**
When a user selects files through `<input type="file">`, the browser creates a `FileList` collection containing one `File` object per selected file. Each `File` object inherits from `Blob` and provides four standard metadata properties: `name` (string, filename without path), `size` (number, bytes), `type` (string, MIME type or empty string if undetermined), and `lastModified` (number, timestamp in milliseconds) . The `type` property is derived from the file extension and can be unreliable for security purposes .

**Beginner-Friendly Explanation**
When you pick a file, the browser tells JavaScript things about it: what it's called, how big it is, and what kind of file it thinks it is. You can use this information to show the user what they selected or to do basic checks before uploading.

### Purposes

- To display selected file information to the user before uploading.
- To perform client-side size validation to prevent oversized uploads.
- To filter files by type for basic user experience.
- To preview images or other content before upload.
- To build custom file selection interfaces.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Access FileList from input
var files = document.getElementById("fileInput").files;

// Access individual File object
var file = files[0];

// Metadata properties
file.name;          // String: filename
file.size;          // Number: bytes
file.type;          // String: MIME type
file.lastModified;  // Number: timestamp
```

**Component Breakdown**

- `.files` : A `FileList` object containing selected files.
- `file.name` : The filename including extension, without path .
- `file.size` : File size in bytes .
- `file.type` : Browser-estimated MIME type based on extension .
- `file.lastModified` : Last modification timestamp in milliseconds.

**Syntax Rules**

1. `.files` is a `FileList` object, not a true array; use `.length` and index access.
2. `File` objects are immutable; metadata cannot be modified.
3. `file.type` returns `""` (empty string) if the type cannot be determined .
4. The `multiple` attribute on the input allows selecting multiple files.
5. Access files on the `change` event of the input.

**Constraints and Limitations**

- **MIME Type Unreliability**: `file.type` is based on extension and can be spoofed by renaming files .
- **No Content Access**: Metadata only; reading file contents requires `FileReader` .
- **Browser Inconsistency**: MIME type detection varies across browsers and operating systems.

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Displaying File Metadata**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>File Metadata — Reading</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="file" id="fileInput" multiple>
  <div id="fileList"></div>

  <script>
    $(function () {
      $("#fileInput").on("change", function () {
        // Step 1: Access the FileList
        var files = this.files;
        var output = "";

        // Step 2: Iterate over files and read metadata
        for (var i = 0; i < files.length; i++) {
          var file = files[i];
          output += "<p>";
          output += "<strong>" + file.name + "</strong><br>";
          output += "Size: " + file.size + " bytes<br>";
          output += "Type: " + (file.type || "unknown") + "<br>";
          output += "Last Modified: " + new Date(file.lastModified).toLocaleString();
          output += "</p>";
        }

        // Step 3: Display in the page
        $("#fileList").html(output);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting files displays each file's name, size, type, and modification date.

**Why This Output Occurs**
The `change` event fires when files are selected. The `FileList` provides access to each `File` object, whose properties contain the metadata .

---

**Example 2: Client-Side Size Validation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>File Metadata — Size Validation</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="file" id="fileInput">
  <p id="status"></p>

  <script>
    $(function () {
      var MAX_SIZE = 5 * 1024 * 1024; // 5 MB in bytes

      $("#fileInput").on("change", function () {
        var file = this.files[0];
        if (!file) return;

        // Step 1: Check file size
        if (file.size > MAX_SIZE) {
          $("#status").text(
            "File is too large (" + (file.size / 1024 / 1024).toFixed(2) + " MB). " +
            "Maximum is 5 MB."
          );
          $(this).val(""); // Clear the input
        } else {
          $("#status").text("File size OK: " + (file.size / 1024).toFixed(1) + " KB");
        }
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting a file larger than 5 MB shows an error and clears the input.
- Selecting a smaller file shows the size in KB.

**Why This Output Occurs**
`file.size` returns the exact byte count. Comparing against `MAX_SIZE` performs client-side validation .

### Real-World Cases

- **Image Previews**: Reading `file.type` to determine if a thumbnail can be shown.
- **Size Warnings**: Warning users before uploading large files.
- **Upload Queues**: Building a list of files with metadata before submission.
- **Progress Calculations**: Using `file.size` to calculate upload progress percentages.

### References

- MDN Web Docs — File API – https://developer.mozilla.org/en-US/docs/Web/API/File_API
- MDN Web Docs — Using files from web applications – https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications
- php.cn — File API Metadata – https://www.php.cn/faq/2888963.html

---

## Core Concept 2: Using the Native `FormData` API with jQuery

### Definitions

**Core Definition**
`FormData` is a native browser interface for constructing sets of key-value pairs representing form fields and their values, including binary file data, encoded as `multipart/form-data`.

**Technical Definition**
`FormData` provides a constructor and methods for building form submission payloads. Passing a `<form>` element to the constructor automatically captures all its successful controls, including files . The `append()` method adds fields manually, accepting strings or `Blob`/`File` objects. `FormData` objects can be sent directly as the `data` option in `$.ajax()` without serialization .

**Beginner-Friendly Explanation**
`FormData` is a container that holds all the pieces of a form — text fields and files — in a format the server understands. You can either create it from an existing form or build it manually, adding files and text fields one by one.

### Purposes

- To package form data including files for AJAX transmission.
- To automatically collect all form fields from a `<form>` element.
- To manually construct payloads with specific fields and files.
- To append additional data not present in the form (e.g., CSRF tokens).
- To send files alongside text data in a single request.

### Syntax Rules and Structure

**Complete General Syntaxes**

**From Existing Form:**
```javascript
var formData = new FormData($("#myForm")[0]);
```

**Manual Construction:**
```javascript
var formData = new FormData();
formData.append("fieldName", "value");
formData.append("fileField", fileInput.files[0]);
```

**Component Breakdown**

- `new FormData(formElement)` : Captures all form fields including files .
- `.append(name, value)` : Adds a field; value can be string or `File`/`Blob` .
- `.get(name)` / `.getAll(name)` : Retrieves values .

**Syntax Rules**

1. Passing a jQuery object to `FormData` requires native DOM access: `$("#form")[0]` .
2. `FormData` automatically sets the `Content-Type` to `multipart/form-data` when sent .
3. Files are appended as `File` or `Blob` objects.
4. Multiple values with the same name can be appended.
5. `FormData` cannot be serialized to a query string; it must be sent as the request body .

**Constraints and Limitations**

- **Requires Modern Browsers**: `FormData` is supported in all modern browsers (IE10+) .
- **No Direct Manipulation**: `FormData` is not a plain object; use `.append()`, `.delete()`, `.set()`.
- **File Exclusion in Serialization**: `.serialize()` excludes files; `FormData` is required .

### Multiple Annotated Complete Code Examples

**Example 1: FormData from Form Element**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>FormData — From Form</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="uploadForm" enctype="multipart/form-data">
    <input type="text" name="title" value="My Document">
    <input type="file" name="document" id="fileInput">
    <button type="submit">Upload</button>
  </form>
  <p id="status"></p>

  <script>
    $(function () {
      $("#uploadForm").on("submit", function (event) {
        event.preventDefault();

        // Step 1: Create FormData from the form element
        var formData = new FormData(this);

        // Step 2: Log the entries
        for (var pair of formData.entries()) {
          console.log(pair[0] + ": " + pair[1]);
        }

        $("#status").text("FormData created with " + formData.getAll("title").length + " title field(s).");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting a file and submitting logs `title: My Document` and `document: [object File]`.
- Status shows "FormData created with 1 title field(s)."

**Why This Output Occurs**
`new FormData(formElement)` captures all named form controls, including files, into a multipart payload .

---

**Example 2: Manual FormData Construction**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>FormData — Manual</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="file" id="fileInput">
  <button id="uploadBtn">Upload Manually</button>
  <p id="status"></p>

  <script>
    $(function () {
      $("#uploadBtn").on("click", function () {
        var file = $("#fileInput")[0].files[0];

        if (!file) {
          $("#status").text("Please select a file.");
          return;
        }

        // Step 1: Create FormData manually
        var formData = new FormData();

        // Step 2: Append file and additional fields
        formData.append("file", file);
        formData.append("userId", "12345");
        formData.append("timestamp", Date.now());

        // Step 3: Log entries
        for (var pair of formData.entries()) {
          console.log(pair[0] + ": " + pair[1]);
        }

        $("#status").text("FormData ready: " + file.name);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting a file and clicking "Upload Manually" logs `file: [object File]`, `userId: 12345`, and a timestamp.
- Status shows "FormData ready: [filename]".

**Why This Output Occurs**
`.append()` adds fields one by one. Files are added as `File` objects, text fields as strings .

### Real-World Cases

- **File Upload Forms**: Sending documents, images, or videos with metadata.
- **Profile Pictures**: Uploading images with user ID and description.
- **CSRF Protection**: Appending CSRF tokens to FormData before submission.
- **Multi-Part APIs**: Sending mixed text and binary data to REST endpoints.

### References

- MDN Web Docs — FormData – https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs — Sending forms through JavaScript – https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_forms_through_JavaScript
- Tencent Cloud — jQuery Submit with Image – https://cloud.tencent.cn/developer/information/jquery%20%E6%8F%90%E4%BA%A4%E5%B8%A6%E5%9B%BE%E7%89%87%E7%9A%84%E8%A1%A8%E5%8D%95-salon

---

## Core Concept 3: AJAX Configuration for Files

### Definitions

**Core Definition**
AJAX configuration for files refers to the specific `$.ajax()` settings required to send `FormData` payloads containing binary file data.

**Technical Definition**
When sending `FormData` via `$.ajax()`, the `processData` option must be set to `false` to prevent jQuery from converting the data into a query string, and `contentType` must be set to `false` to prevent jQuery from overriding the automatically-set `multipart/form-data` header with the boundary parameter . Without these settings, the file data is corrupted or the request fails.

**Beginner-Friendly Explanation**
jQuery normally converts data into a URL string. For files, that would destroy the binary content. Setting `processData: false` and `contentType: false` tells jQuery "don't touch this data — let the browser handle it."

### Purposes

- To upload files without corrupting binary data.
- To send `FormData` payloads correctly via jQuery AJAX.
- To allow the browser to set the proper `multipart/form-data` content type.
- To integrate file uploads with existing AJAX workflows.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/upload",
  type: "POST",
  data: formData,
  processData: false,   // Required for FormData
  contentType: false,   // Required for FormData
  success: function(response) { }
});
```

**Component Breakdown**

- `processData: false` : Prevents jQuery from transforming `FormData` .
- `contentType: false` : Lets the browser set `multipart/form-data; boundary=...` .
- `data: formData` : The `FormData` object as request body.

**Syntax Rules**

1. Both `processData: false` and `contentType: false` are **required** for file uploads .
2. `FormData` automatically includes the correct boundary in the Content-Type header.
3. The `type` should be `POST` for file uploads.
4. The `enctype` attribute on the form should be `multipart/form-data` for native submission .

**Constraints and Limitations**

- **Setting Required**: Omitting either setting causes upload failure or file corruption.
- **No `.serialize()`**: Files cannot be serialized; `FormData` is mandatory .
- **Server Expectation**: Server must accept `multipart/form-data`.

### Multiple Annotated Complete Code Examples

**Example 1: Basic File Upload with Correct Settings**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>AJAX Config — Basic Upload</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="uploadForm" enctype="multipart/form-data">
    <input type="file" name="file" id="fileInput">
    <button type="submit">Upload</button>
  </form>
  <p id="status"></p>

  <script>
    $(function () {
      $("#uploadForm").on("submit", function (event) {
        event.preventDefault();

        // Step 1: Create FormData from form
        var formData = new FormData(this);

        // Step 2: Send with required settings
        $.ajax({
          url: "/api/upload",
          type: "POST",
          data: formData,
          processData: false,  // Don't process the data
          contentType: false,  // Don't set Content-Type
          success: function (response) {
            $("#status").text("Upload successful!");
          },
          error: function (xhr, status, error) {
            $("#status").text("Upload failed: " + error);
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting a file and submitting sends it to the server.
- Success message displayed on completion.

**Why This Output Occurs**
`processData: false` keeps the `FormData` intact; `contentType: false` lets the browser set the proper `multipart/form-data` header .

### Real-World Cases

- **Image Uploads**: Sending profile pictures with correct encoding.
- **Document Submission**: Uploading PDFs with metadata.
- **Video Uploads**: Handling large binary files.
- **Multi-File Uploads**: Sending multiple files in one request.

### References

- Stack Overflow — jQuery AJAX File Upload – https://stackoverflow.com/revisions/a8ed88a5-f62e-4460-88f7-0ba076472ea1/view-source
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/

---

## Core Concept 4: Upload Progress Monitoring

### Definitions

**Core Definition**
Upload progress monitoring is the process of tracking and displaying the percentage of a file upload that has been transmitted to the server.

**Technical Definition**
The `xhr` option in `$.ajax()` accepts a function that returns a custom `XMLHttpRequest` object. By calling `xhr.upload.addEventListener("progress", callback)`, developers can listen for progress events during the upload phase. The event object provides `loaded` (bytes uploaded) and `total` (total bytes), allowing calculation of a percentage .

**Beginner-Friendly Explanation**
When uploading a large file, you can show a progress bar so users know how much is done. The browser tracks how many bytes have been sent and tells you through "progress" events.

### Purposes

- To provide visual feedback during lengthy uploads.
- To display upload percentage to the user.
- To enable cancellation of uploads in progress.
- To improve user experience by reducing uncertainty.
- To log upload performance metrics.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  xhr: function() {
    var xhr = new window.XMLHttpRequest();
    xhr.upload.addEventListener("progress", function(evt) {
      if (evt.lengthComputable) {
        var percent = Math.round((evt.loaded / evt.total) * 100);
        // Update progress UI
      }
    }, false);
    return xhr;
  }
});
```

**Component Breakdown**

- `xhr` : A function returning a custom `XMLHttpRequest` object .
- `xhr.upload` : The upload progress interface.
- `evt.lengthComputable` : Boolean indicating if total is known.
- `evt.loaded` / `evt.total` : Bytes uploaded / total bytes.

**Syntax Rules**

1. The `xhr` option receives `jqXHR` and `settings` as arguments.
2. `xhr.upload` is only available for upload progress (not download).
3. `lengthComputable` may be `false` in some scenarios.
4. Progress events fire frequently; throttle UI updates if necessary.

**Constraints and Limitations**

- **Browser Support**: `xhr.upload` is supported in modern browsers.
- **Event Frequency**: Progress events fire very often; excessive DOM updates can cause performance issues .
- **No Total**: `lengthComputable` may be `false`, making percentage calculation impossible.

### Multiple Annotated Complete Code Examples

**Example 1: Upload Progress Bar**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Progress — Upload Bar</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #progressBar { width: 100%; height: 20px; background: #eee; }
    #progressFill { width: 0%; height: 100%; background: green; }
  </style>
</head>
<body>
  <input type="file" id="fileInput">
  <button id="uploadBtn">Upload with Progress</button>
  <div id="progressBar"><div id="progressFill"></div></div>
  <p id="status"></p>

  <script>
    $(function () {
      $("#uploadBtn").on("click", function () {
        var file = $("#fileInput")[0].files[0];
        if (!file) { $("#status").text("Select a file."); return; }

        var formData = new FormData();
        formData.append("file", file);

        $.ajax({
          url: "/api/upload",
          type: "POST",
          data: formData,
          processData: false,
          contentType: false,
          xhr: function () {
            // Step 1: Create custom XHR
            var xhr = new window.XMLHttpRequest();

            // Step 2: Listen for upload progress
            xhr.upload.addEventListener("progress", function (evt) {
              if (evt.lengthComputable) {
                var percent = Math.round((evt.loaded / evt.total) * 100);
                $("#progressFill").css("width", percent + "%");
                $("#status").text("Uploading: " + percent + "%");
              }
            }, false);

            return xhr;
          },
          success: function () {
            $("#status").text("Upload complete!");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Upload with Progress" shows a progress bar filling from 0% to 100%.
- Status text updates with the current percentage.

**Why This Output Occurs**
The custom `xhr` function attaches a progress listener to `xhr.upload`, which fires as bytes are transmitted .

### Real-World Cases

- **Large File Uploads**: Showing progress for videos or large documents.
- **Multiple File Uploads**: Tracking overall progress across a batch.
- **Upload Cancellation**: Enabling a "Cancel" button during upload.
- **User Experience**: Reducing perceived wait time with visual feedback.

### References

- Stack Overflow — XHR Progress Listener – https://stackoverflow.com/revisions/ba374d73-2052-4069-b65f-06d1422bcb14/view-source
- Tencent Cloud — AJAX Upload Progress – https://cloud.tencent.cn/developer/information/js%20ajax%e6%96%87%e4%bb%b6%e4%b8%8a%e4%bc%a0%e8%bf%9b%e5%ba%a6%e6%9d%a1-article

---

## Core Concept 5: Server-Side Validation Requirements

### Definitions

**Core Definition**
Server-side validation requirements are the security checks that must be performed on uploaded files after they reach the server, including MIME type verification, file size limits, and filename safety.

**Technical Definition**
Client-side validation is insufficient for security because attackers can bypass JavaScript entirely. Server-side validation must include: (1) file size limits enforced before parsing to prevent resource exhaustion, (2) MIME type detection via file signature (magic number) sniffing, not the client-supplied `Content-Type`, (3) extension allowlisting after filename decoding, and (4) filename sanitization to prevent path traversal and overwrites . OWASP recommends a defense-in-depth approach combining multiple validation techniques .

**Beginner-Friendly Explanation**
You cannot trust anything the browser says about a file. An attacker can rename a virus to "photo.jpg" and the browser will say it's an image. The server must open the file and check its actual contents to verify it's safe.

### Purposes

- To prevent malicious files from being stored or executed.
- To protect server resources from exhaustion attacks.
- To ensure only allowed file types are accepted.
- To prevent path traversal and filename-based attacks.
- To comply with security best practices.

### Syntax Rules and Structure

**Server-Side Validation Checklist**

| Check | Purpose | Implementation |
|---|---|---|
| **Size Limit (Pre-Parse)** | Prevent body exhaustion | `http.MaxBytesReader`, `RequestSizeLimit`  |
| **Size Limit (Post-Parse)** | Business logic enforcement | Check `file.Size` after parsing  |
| **MIME Type (Sniff)** | Verify actual file type | Read magic bytes, compare to known signatures  |
| **Extension Allowlist** | Limit file types | Compare decoded extension against allowlist  |
| **Filename Sanitization** | Prevent path traversal | Generate UUID, strip special characters  |
| **Storage Isolation** | Prevent execution | Store outside webroot, disable execution  |

**Component Breakdown**

- **Magic Number Sniffing**: Reading the first few bytes to determine actual file type .
- **Allowlist**: Only permitting explicitly approved MIME types/extensions .
- **Pre-Parse Limits**: Enforcing size before multipart parsing to prevent DoS .

**Syntax Rules**

1. **Never trust** `Content-Type` header or `file.type` from the client .
2. **Always enforce size limits** before and after multipart parsing .
3. **Use allowlists** of approved extensions and MIME types, not denylists .
4. **Sniff file signatures** (magic numbers) to verify actual type .
5. **Generate random filenames** (UUID/GUID) instead of using user-supplied names .

**Constraints and Limitations**

- **No Single Solution**: Defense in depth is required; no single check is sufficient .
- **Magic Number Limitations**: Complex formats (ZIP, DOCX) may share signatures; deeper analysis needed .
- **Performance**: Sniffing and validating adds processing overhead.

### Multiple Annotated Complete Code Examples

**Example 1: MIME Type Sniffing Concept**

```javascript
// Server-side pseudocode (Node.js concept)
function validateUpload(fileBuffer, originalName) {
  // Step 1: Check magic number for PNG
  var PNG_SIGNATURE = [0x89, 0x50, 0x4E, 0x47];
  var isPng = PNG_SIGNATURE.every(function(byte, i) {
    return fileBuffer[i] === byte;
  });

  if (!isPng) {
    throw new Error("File is not a valid PNG.");
  }

  // Step 2: Check extension allowlist
  var ext = originalName.split('.').pop().toLowerCase();
  var allowed = ["png", "jpg", "jpeg", "gif"];
  if (allowed.indexOf(ext) === -1) {
    throw new Error("File extension not allowed.");
  }

  return true;
}
```

**Expected Output**
- Files with PNG magic numbers but `.exe` extension are rejected.
- Files with valid PNG signature and `.png` extension pass.

**Why This Output Occurs**
Magic numbers verify actual file content; extension allowlist limits accepted types .

### Real-World Cases

- **Image Hosting**: Verifying uploaded images are actually images before serving.
- **Document Management**: Ensuring only PDFs and Office documents are stored.
- **Security-Critical Applications**: Banking, healthcare, and government systems.
- **Public Uploads**: Any system where users can upload files accessed by others.

### References

- OWASP — File Upload Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- OWASP — Test Upload of Unexpected File Types – https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/10_Business_Logic_Testing/08-Test_Upload_of_Unexpected_File_Types
- Go Packages — upload validation – https://pkg.go.dev/github.com/spectrum-labs-tech/go-toolkit/pkg/upload

---

## Summary Comparison Table

| Concept | Key Method/Property | Purpose | Critical Note |
|---|---|---|---|
| File Metadata | `file.name`, `.size`, `.type` | Read file info | `type` is unreliable for security |
| FormData | `new FormData(form)` | Package files + text | Required for AJAX uploads |
| AJAX Config | `processData: false`, `contentType: false` | Send files correctly | Both settings mandatory |
| Progress | `xhr.upload.addEventListener` | Track upload | Fires frequently; throttle UI |
| Server Validation | Magic number sniffing, allowlists | Security | Client-side alone insufficient |

---

## Important Notes on Version-Specific Behaviour and Limitations

1. **File API Support**: Supported in all modern browsers; IE10+ .
2. **FormData Support**: Baseline widely available since July 2015 .
3. **`processData` / `contentType`**: Required for `FormData` uploads with jQuery .
4. **MIME Type Spoofing**: `file.type` is derived from extension and can be spoofed .
5. **Pre-Parse Size Limits**: Essential to prevent DoS attacks before multipart parsing .
6. **OWASP Recommendation**: Defense in depth — no single validation technique is sufficient .

---

## References

- MDN Web Docs — File API – https://developer.mozilla.org/en-US/docs/Web/API/File_API
- MDN Web Docs — Using files from web applications – https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications
- MDN Web Docs — FormData – https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs — Sending forms through JavaScript – https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_forms_through_JavaScript
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- OWASP — File Upload Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- OWASP — Test Upload of Unexpected File Types – https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/10_Business_Logic_Testing/08-Test_Upload_of_Unexpected_File_Types
- Stack Overflow — jQuery AJAX File Upload Settings – https://stackoverflow.com/revisions/a8ed88a5-f62e-4460-88f7-0ba076472ea1/view-source
- Stack Overflow — XHR Progress Listener – https://stackoverflow.com/revisions/ba374d73-2052-4069-b65f-06d1422bcb14/view-source
- Go Packages — Upload Validation – https://pkg.go.dev/github.com/spectrum-labs-tech/go-toolkit/pkg/upload
- php.cn — File API Metadata – https://www.php.cn/faq/2888963.html
- Tencent Cloud — jQuery Submit with Image – https://cloud.tencent.cn/developer/information/jquery%20%E6%8F%90%E4%BA%A4%E5%B8%A6%E5%9B%BE%E7%89%87%E7%9A%84%E8%A1%A8%E5%8D%95-salon