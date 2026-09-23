# The Core `$.ajax()` Method: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`$.ajax()` is jQuery's low-level interface for performing asynchronous HTTP (Ajax) requests, providing granular control over request configuration, response handling, and error management.

**Technical Definition**
`jQuery.ajax(url, settings)` returns a `jqXHR` object (a superset of the native `XMLHttpRequest`) that implements the Promise interface. It accepts a URL and a configuration object controlling method, data, headers, content type, expected response type, timeout, and callback behaviour. As of jQuery 1.5, the `jqXHR` object exposes `.done()`, `.fail()`, and `.always()` promise methods. As of jQuery 3.0, the legacy `jqXHR.success()`, `jqXHR.error()`, and `jqXHR.complete()` methods were removed, though the corresponding `success`, `error`, and `complete` **callbacks in the settings object** remain supported .

**Beginner-Friendly Explanation**
`$.ajax()` is the "do anything" tool for talking to a server without reloading the page. You tell it where to send data, what kind of data to send, what kind to expect back, and what to do when it succeeds or fails.

### Key Characteristics

- **Promise-Based**: Returns a `jqXHR` object with `.done()`, `.fail()`, `.always()`, and `.then()` methods .
- **Configurable**: Nearly every aspect of the request can be customized.
- **Callback Hooks**: Supports `beforeSend`, `success`, `error`, `complete`, `dataFilter`, and `statusCode`.
- **Automatic Data Handling**: Can serialize request data and parse response data based on `dataType`.
- **Abortable**: The returned `jqXHR` object exposes `.abort()` to cancel the request.

### Prerequisites

- Basic JavaScript and HTTP knowledge (methods, headers, status codes).
- jQuery library included via CDN or local file.
- Understanding of JSON, URL encoding, and MIME types.

### Related Programming Areas

- **jQuery AJAX Shorthands**: `$.get()`, `$.post()`, `$.getJSON()`, `.load()`.
- **Promise API**: `.done()`, `.fail()`, `.always()`, `.then()`.
- **FormData**: For file uploads via AJAX.
- **jQuery.ajaxSetup()**: For setting global AJAX defaults (not recommended).

### Core Concepts / Features

1. Request Configuration: `url`, `method`/`type`, `data`
2. `contentType` vs. `dataType`
3. Custom Request Headers (`headers`)
4. Timeouts (`timeout`) and Request Aborts
5. Legacy Callbacks vs. Modern Promise Configuration

---

## Core Concept 1: Request Configuration (`url`, `method`/`type`, `data`)

### Definitions

**Core Definition**
Request configuration defines the target URL, the HTTP method used to send the request, and the data payload included with the request.

**Technical Definition**
The `url` setting specifies the endpoint. The `type` (or `method` in jQuery 1.9+) setting specifies the HTTP method (GET, POST, PUT, DELETE). The `data` setting can be a plain object, string, or FormData, and is automatically serialized into a query string for GET requests or into the request body for POST requests, using the `application/x-www-form-urlencoded` format by default .

**Beginner-Friendly Explanation**
You tell `$.ajax()` three things: where to send the request (URL), how to send it (GET or POST), and what to send (data).

### Purposes

- To specify the server endpoint for the request.
- To choose the HTTP method appropriate for the operation.
- To pass data to the server as query parameters or request body.
- To configure the request to match server expectations.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/endpoint",      // Required: target URL
  type: "POST",              // Optional: HTTP method (default: "GET")
  data: { key: "value" },    // Optional: request payload
  dataType: "json"           // Optional: expected response type
});
```

**Component Breakdown**

- `url` : String containing the URL to which the request is sent .
- `type` / `method` : HTTP method (GET, POST, PUT, DELETE). Default is `"GET"` .
- `data` : Data to be sent to the server. If an object, jQuery serializes it to a query string .
- `dataType` : Expected response type (see Core Concept 2).

**Syntax Rules**

1. If `type` is `"GET"`, data is appended to the URL as a query string.
2. If `type` is `"POST"`, data is sent in the request body.
3. For `data` objects, jQuery uses `jQuery.param()` to serialize them.
4. Passing a string for `data` sends it as-is.
5. The `method` alias was added in jQuery 1.9; `type` is the original name.

**Constraints and Limitations**

- **Data Size**: GET requests have URL length limits; use POST for large payloads.
- **Character Encoding**: Data is URL-encoded automatically; special characters are handled by jQuery.

### Multiple Annotated Complete Code Examples

**Example 1: GET Request with Data**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — GET Request</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="loadBtn">Load User</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#loadBtn").on("click", function () {
        // Step 1: Send GET request with query parameters
        $.ajax({
          url: "/api/user",
          type: "GET",
          data: { id: 123, fields: "name,email" },
          dataType: "json"
        }).done(function (response) {
          // Step 2: Handle successful response
          $("#output").text("User: " + response.name);
        }).fail(function (xhr, status, error) {
          // Step 3: Handle errors
          $("#output").text("Error: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Load User" sends `GET /api/user?id=123&fields=name%2Cemail`.
- On success, displays the user name.
- On failure, displays an error message.

**Why This Output Occurs**
jQuery serializes the `data` object into a query string and appends it to the URL for GET requests .

---

**Example 2: POST Request with Object Data**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — POST Request</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="saveBtn">Save Data</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#saveBtn").on("click", function () {
        // Step 1: Send POST request with data object
        $.ajax({
          url: "/api/save",
          type: "POST",
          data: {
            title: "Hello World",
            body: "This is a test.",
            userId: 456
          },
          dataType: "json"
        }).done(function (response) {
          $("#output").text("Saved with ID: " + response.id);
        }).fail(function (xhr, status, error) {
          $("#output").text("Save failed: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Save Data" sends `POST /api/save` with body `title=Hello+World&body=This+is+a+test.&userId=456`.
- On success, displays the saved ID.

**Why This Output Occurs**
For POST requests, jQuery serializes the data object into the request body using `application/x-www-form-urlencoded` format .

### Real-World Cases

- **REST APIs**: GET for reading, POST for creating, PUT for updating, DELETE for removing.
- **Search Forms**: GET requests with query parameters for search terms.
- **Form Submission**: POST requests with form data serialized from inputs.

### References

- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/

---

## Core Concept 2: `contentType` vs. `dataType`

### Definitions

**Core Definition**
`contentType` specifies the format of the data **being sent** to the server, while `dataType` specifies the format of the response data **expected back** from the server.

**Technical Definition**
`contentType` sets the `Content-Type` HTTP request header, telling the server what format the request body is in. Default is `application/x-www-form-urlencoded; charset=UTF-8` . `dataType` tells jQuery what kind of response to expect (`text`, `html`, `xml`, `json`, `jsonp`, `script`), determining how the response is parsed before being passed to the success handler . If `dataType` is not specified, jQuery infers it from the response's `Content-Type` header.

**Beginner-Friendly Explanation**
`contentType` is what you're **sending** — "Here's my data in JSON format." `dataType` is what you're **expecting back** — "Please send me back JSON." They are independent settings that describe opposite directions of the same request .

### Purposes

- To specify the format of the request body being sent.
- To declare the expected format of the response.
- To control how jQuery parses the response before passing it to callbacks.
- To ensure the server and client agree on data formats.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/endpoint",
  type: "POST",
  contentType: "application/json; charset=utf-8",  // What we're SENDING
  dataType: "json",                                // What we EXPECT BACK
  data: JSON.stringify({ key: "value" })
});
```

**Common `dataType` Values**

| Value | Description | Processing |
|---|---|---|
| `"text"` | Plain text | No processing |
| `"html"` | HTML | Inserted into DOM when used with `.load()` |
| `"xml"` | XML document | Parsed with `jQuery.parseXML` |
| `"json"` | JSON | Parsed with `jQuery.parseJSON` |
| `"jsonp"` | JSONP | Appends callback parameter |
| `"script"` | JavaScript | Executes the script |

**Component Breakdown**

- `contentType` : Sets the `Content-Type` header for the request .
- `dataType` : Tells jQuery how to parse the response .
- `Accept` header: Automatically set based on `dataType` .

**Syntax Rules**

1. `contentType` defaults to `application/x-www-form-urlencoded; charset=UTF-8` .
2. `dataType` defaults to intelligent guessing based on response `Content-Type` .
3. Setting `dataType` causes jQuery to ignore the response's `Content-Type` for parsing decisions .
4. `contentType` is sent in the request header; `dataType` affects response parsing.
5. For `FormData` uploads, set `contentType: false` to let the browser set the multipart boundary .

**Constraints and Limitations**

- **Charset**: The `charset` in `contentType` specifies the request body encoding, not the response encoding.
- **JSONP**: `dataType: "jsonp"` appends a callback parameter to the URL and expects JavaScript execution.

### Multiple Annotated Complete Code Examples

**Example 1: Sending JSON, Expecting JSON**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — contentType and dataType</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="sendJson">Send JSON</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#sendJson").on("click", function () {
        $.ajax({
          url: "/api/data",
          type: "POST",
          // Sending JSON
          contentType: "application/json; charset=utf-8",
          // Expecting JSON back
          dataType: "json",
          // Stringify the object for JSON body
          data: JSON.stringify({ name: "Alice", age: 30 })
        }).done(function (response) {
          // response is already a parsed JavaScript object
          $("#output").text("Received: " + response.message);
        }).fail(function (xhr, status, error) {
          $("#output").text("Error: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Request body is JSON: `{"name":"Alice","age":30}`.
- Response is parsed as JSON automatically.
- `response.message` is accessible as an object property.

**Why This Output Occurs**
`contentType` tells the server the request body is JSON. `dataType: "json"` tells jQuery to parse the response as JSON .

---

**Example 2: Sending Form-Encoded, Expecting Text**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Form-Encoded</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="sendForm">Send Form Data</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#sendForm").on("click", function () {
        $.ajax({
          url: "/api/form",
          type: "POST",
          // Default contentType: application/x-www-form-urlencoded
          dataType: "text",  // Expect plain text back
          data: { username: "bob", password: "secret" }
        }).done(function (response) {
          // response is a string
          $("#output").text("Response: " + response);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Request body is `username=bob&password=secret`.
- Response is treated as plain text.

**Why This Output Occurs**
Without `contentType`, jQuery uses the default form encoding. `dataType: "text"` prevents parsing, returning the raw response string .

### Real-World Cases

- **REST APIs**: `contentType: "application/json"` for JSON APIs, `dataType: "json"` for responses.
- **Legacy APIs**: `contentType: "application/x-www-form-urlencoded"` for traditional form endpoints.
- **HTML Fragments**: `dataType: "html"` when loading partial page content.
- **JSONP**: `dataType: "jsonp"` for cross-domain requests (legacy).

### References

- Learn Microsoft — Difference between dataType and contentType – https://learn.microsoft.com/da-dk/archive/msdn-technet-forums/c5ee5f75-beb6-4d1f-8439-e9d678bf087f
- Stack Overflow — DataType vs ContentType – https://stackoverflow.com/questions/17828250/datatype-vs-contenttype-in-jquery-ajax

---

## Core Concept 3: Custom Request Headers (`headers`)

### Definitions

**Core Definition**
The `headers` option allows adding custom HTTP headers to individual AJAX requests, used for authentication tokens, API versioning, or custom metadata.

**Technical Definition**
`headers` is an object of key-value pairs that jQuery adds to the request via `XMLHttpRequest.setRequestHeader()`. It is equivalent to using the `beforeSend` callback to call `setRequestHeader()`, but more concise. Custom headers are sent with every request configured with that `headers` object .

**Beginner-Friendly Explanation**
Custom headers are like extra labels you put on a package before shipping it. They can carry authentication tokens, tell the server which API version you want, or pass any other metadata the server needs.

### Purposes

- To send authentication tokens (e.g., `Authorization`, `X-CSRF-Token`).
- To specify API versions or client information.
- To pass custom metadata required by the server.
- To set headers dynamically per request.
- To avoid modifying global defaults with `$.ajaxSetup()`.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/endpoint",
  headers: {
    "X-Custom-Header": "value",
    "Authorization": "Bearer " + token
  }
});
```

**Component Breakdown**

- `headers` : Plain object with header names as keys and values as strings.
- Header names are case-insensitive but conventionally use kebab-case.

**Syntax Rules**

1. `headers` is applied per-request .
2. For global defaults, use `$.ajaxSetup({ headers: { ... } })` .
3. `beforeSend` can also set headers via `xhr.setRequestHeader()` .
4. When both `headers` and `beforeSend` set the same header, `beforeSend` takes precedence.
5. Multiple headers are supported as multiple key-value pairs.

**Constraints and Limitations**

- **CORS**: Custom headers may trigger preflight OPTIONS requests for cross-origin requests.
- **Restricted Headers**: Some headers (`Accept-Charset`, `Content-Length`, etc.) are restricted by the browser.
- **`ajaxSetup` Overwrites**: Calling `ajaxSetup` multiple times overwrites previous default headers .

### Multiple Annotated Complete Code Examples

**Example 1: Authorization Header**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Custom Headers</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="fetchData">Fetch Protected Data</button>
  <div id="output"></div>

  <script>
    $(function () {
      var authToken = "eyJhbGciOiJIUzI1NiIs...";

      $("#fetchData").on("click", function () {
        $.ajax({
          url: "/api/protected",
          type: "GET",
          headers: {
            "Authorization": "Bearer " + authToken,
            "X-Requested-With": "XMLHttpRequest"
          },
          dataType: "json"
        }).done(function (data) {
          $("#output").text("Data: " + JSON.stringify(data));
        }).fail(function (xhr, status, error) {
          $("#output").text("Error: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Request includes `Authorization: Bearer eyJhbGci...` and `X-Requested-With: XMLHttpRequest`.
- Server authenticates using the token.

**Why This Output Occurs**
The `headers` object adds each key-value pair to the request via `setRequestHeader()` .

---

**Example 2: Dynamic Headers per Request**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Dynamic Headers</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <select id="apiVersion">
    <option value="v1">Version 1</option>
    <option value="v2">Version 2</option>
  </select>
  <button id="callApi">Call API</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#callApi").on("click", function () {
        var version = $("#apiVersion").val();

        $.ajax({
          url: "/api/data",
          type: "GET",
          headers: {
            "X-API-Version": version,
            "X-Client": "jQuery-Example"
          },
          dataType: "json"
        }).done(function (data) {
          $("#output").text("Version " + version + " response: " + data.message);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting "Version 2" and calling sends `X-API-Version: v2`.
- The server responds according to the requested version.

**Why This Output Occurs**
The `headers` object is built dynamically based on the selected value .

### Real-World Cases

- **JWT Authentication**: Sending `Authorization: Bearer <token>` with each request.
- **CSRF Protection**: Sending `X-CSRF-Token` headers for form submissions.
- **API Versioning**: `X-API-Version` headers for versioned endpoints.
- **Client Identification**: `X-Client` headers for analytics or debugging.

### References

- Stack Overflow — Custom HTTP headers in jQuery – https://stackoverflow.com/posts/14655768/revisions
- Apache JIRA — Custom headers with jQuery AJAX – https://issues.apache.org/jira/browse/THRIFT-4737

---

## Core Concept 4: Timeouts (`timeout`) and Request Aborts

### Definitions

**Core Definition**
The `timeout` option specifies the maximum time (in milliseconds) to wait for a response before aborting the request. The `.abort()` method on the returned `jqXHR` object manually cancels the request.

**Technical Definition**
When `timeout` is reached, the request is aborted and the `error` callback (or `.fail()` handler) is invoked with `textStatus` set to `"timeout"`. Calling `.abort()` on the `jqXHR` object immediately cancels the request and triggers the error callback with `textStatus` set to `"abort"` . The `status` code for aborted requests is typically `0` .

**Beginner-Friendly Explanation**
A timeout says "give up if the server doesn't respond in X milliseconds." `.abort()` says "give up right now." Both are ways to stop waiting for a slow server.

### Purposes

- To prevent requests from hanging indefinitely.
- To improve user experience by failing fast on slow connections.
- To cancel requests when the user navigates away or changes context.
- To implement polling with controlled request lifecycles.
- To clean up pending requests before starting new ones.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Timeout:**
```javascript
$.ajax({
  url: "/api/slow",
  timeout: 5000,  // 5 seconds
  error: function(xhr, status, error) {
    if (status === "timeout") {
      // Handle timeout
    }
  }
});
```

**Abort:**
```javascript
var request = $.ajax({
  url: "/api/data",
  type: "GET"
});

// Later, to cancel:
request.abort();
```

**Component Breakdown**

- `timeout` : Integer milliseconds. No default (no timeout).
- `request.abort()` : Cancels the request immediately.
- `status` in error callback: `"timeout"` or `"abort"`.

**Syntax Rules**

1. `timeout` is measured from the time the request is sent.
2. `.abort()` fires the error callback with `textStatus: "abort"` .
3. Both timeout and abort result in `status` code `0` .
4. The `error` callback receives `textStatus` indicating the reason.
5. `.abort()` is safe to call on completed requests (no effect).

**Constraints and Limitations**

- **Network Timeouts**: Server-side timeouts may occur before the client `timeout` is reached.
- **Status Code 0**: Aborted and timed-out requests both have HTTP status 0, making them indistinguishable by status alone .
- **Error Callback Required**: Without an error handler, timeouts may be silent.

### Multiple Annotated Complete Code Examples

**Example 1: Timeout with Error Handling**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Timeout</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="slowRequest">Call Slow API (5s timeout)</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#slowRequest").on("click", function () {
        $("#output").text("Requesting...");

        $.ajax({
          url: "/api/slow-endpoint",
          type: "GET",
          timeout: 5000,  // Abort after 5 seconds
          dataType: "json"
        }).done(function (data) {
          $("#output").text("Success: " + data.message);
        }).fail(function (xhr, status, error) {
          // Step 1: Check if timeout caused the failure
          if (status === "timeout") {
            $("#output").text("Request timed out after 5 seconds.");
          } else {
            $("#output").text("Error: " + error);
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Call Slow API" sends the request.
- If no response within 5 seconds, displays "Request timed out after 5 seconds."
- If response arrives, displays success message.

**Why This Output Occurs**
`timeout: 5000` aborts the request after 5 seconds, triggering `.fail()` with `status === "timeout"` .

---

**Example 2: Manual Abort**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Abort</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="startReq">Start Request</button>
  <button id="cancelReq">Cancel Request</button>
  <div id="output"></div>

  <script>
    $(function () {
      var currentRequest = null;

      $("#startReq").on("click", function () {
        // Step 1: Abort any existing request first
        if (currentRequest) {
          currentRequest.abort();
        }

        $("#output").text("Loading...");

        // Step 2: Store the jqXHR object
        currentRequest = $.ajax({
          url: "/api/data",
          type: "GET",
          dataType: "json"
        }).done(function (data) {
          $("#output").text("Loaded: " + data.message);
        }).fail(function (xhr, status, error) {
          // Step 3: Handle abort specifically
          if (status === "abort") {
            $("#output").text("Request was cancelled.");
          } else {
            $("#output").text("Error: " + error);
          }
        });
      });

      $("#cancelReq").on("click", function () {
        if (currentRequest) {
          currentRequest.abort();
        }
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Start Request" begins loading.
- Clicking "Cancel Request" aborts and displays "Request was cancelled."

**Why This Output Occurs**
`.abort()` on the stored `jqXHR` object cancels the request and triggers `.fail()` with `status === "abort"` .

### Real-World Cases

- **Search Autocomplete**: Abort previous search requests when a new keystroke occurs.
- **Long-Polling**: Timeout and retry long-polling requests.
- **Page Navigation**: Abort pending requests when the user leaves the page.
- **File Uploads**: Allow users to cancel large uploads.

### References

- Stack Overflow — jQuery timeout option – https://stackoverflow.com/questions/21964045/how-to-cancel-an-ajax-request-or-third-party-js-request-when-no-response-after-x
- Stack Overflow — Abort and timeout distinction – https://stackoverflow.com/revisions/95daa398-6025-42f9-ab45-602e9f703593/view-source

---

## Core Concept 5: Legacy Callbacks vs. Modern Promise Configuration

### Definitions

**Core Definition**
Legacy callbacks (`success`, `error`, `complete` in the settings object) are the original jQuery AJAX callback mechanism. Modern configuration uses Promise methods (`.done()`, `.fail()`, `.always()`) on the returned `jqXHR` object.

**Technical Definition**
Prior to jQuery 3.0, `$.ajax()` settings included `success`, `error`, and `complete` callbacks, and the returned `jqXHR` object exposed corresponding methods `jqXHR.success()`, `jqXHR.error()`, and `jqXHR.complete()`. As of jQuery 3.0, the **jqXHR methods** were removed, but the **settings callbacks** (`success`, `error`, `complete`) remain supported. The Promise methods `.done()`, `.fail()`, and `.always()` are the recommended modern approach .

**Beginner-Friendly Explanation**
jQuery changed the recommended way to handle AJAX results. Old code used `success`, `error`, and `complete` inside the settings object. New code uses `.done()`, `.fail()`, and `.always()` chained after the `$.ajax()` call. Both still work inside the settings object, but the chained methods are cleaner and support multiple handlers.

### Purposes

- To handle successful responses (`.done()` / `success`).
- To handle errors and failures (`.fail()` / `error`).
- To execute code regardless of outcome (`.always()` / `complete`).
- To register multiple callbacks for the same outcome (Promise advantage).
- To write code compatible with modern JavaScript promise patterns.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Legacy Settings (Still Supported):**
```javascript
$.ajax({
  url: "/api/data",
  success: function(data, textStatus, jqXHR) { },
  error: function(jqXHR, textStatus, errorThrown) { },
  complete: function(jqXHR, textStatus) { }
});
```

**Modern Promise Chain (Recommended):**
```javascript
$.ajax({
  url: "/api/data"
})
.done(function(data, textStatus, jqXHR) { })
.fail(function(jqXHR, textStatus, errorThrown) { })
.always(function(jqXHR, textStatus) { });
```

**Component Breakdown**

- `.done()` : Replaces `success` callback .
- `.fail()` : Replaces `error` callback .
- `.always()` : Replaces `complete` callback .
- `.then()` : Combined success/error promise method.

**Syntax Rules**

1. `.done()`, `.fail()`, and `.always()` are Promise methods on the `jqXHR` object .
2. The settings callbacks `success`, `error`, and `complete` **remain supported** in jQuery 3.x .
3. The jqXHR methods `success()`, `error()`, and `complete()` were **removed** in jQuery 3.0 .
4. Multiple `.done()` and `.fail()` handlers can be registered; they execute in order .
5. `.then()` is the ES6-compliant alternative combining success and failure.

**Constraints and Limitations**

- **Removed Methods**: `jqXHR.success()`, `jqXHR.error()`, `jqXHR.complete()` no longer exist .
- **Callback Order**: Settings callbacks and Promise callbacks fire in a defined order (settings first, then promise) .
- **Legacy Code**: Existing code using settings callbacks continues to work but is not the modern recommendation.

### Multiple Annotated Complete Code Examples

**Example 1: Legacy vs. Modern Syntax**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Legacy vs Modern</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="legacyBtn">Legacy Callbacks</button>
  <button id="modernBtn">Modern Promise</button>
  <div id="output"></div>

  <script>
    $(function () {
      // Legacy approach (still works in jQuery 3.x)
      $("#legacyBtn").on("click", function () {
        $.ajax({
          url: "/api/data",
          type: "GET",
          dataType: "json",
          success: function (data) {
            $("#output").text("Legacy success: " + data.message);
          },
          error: function (xhr, status, error) {
            $("#output").text("Legacy error: " + error);
          },
          complete: function () {
            console.log("Legacy complete.");
          }
        });
      });

      // Modern approach (recommended)
      $("#modernBtn").on("click", function () {
        $.ajax({
          url: "/api/data",
          type: "GET",
          dataType: "json"
        })
        .done(function (data) {
          $("#output").text("Modern success: " + data.message);
        })
        .fail(function (xhr, status, error) {
          $("#output").text("Modern error: " + error);
        })
        .always(function () {
          console.log("Modern always.");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Both buttons produce the same user-facing result.
- Console logs from `complete` and `always` are identical in purpose.

**Why This Output Occurs**
The settings callbacks (`success`, `error`, `complete`) and Promise methods (`.done()`, `.fail()`, `.always()`) are functionally equivalent, with Promise methods being the modern recommendation .

---

**Example 2: Multiple Handlers with Promise**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ajax — Multiple Handlers</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="fetch">Fetch</button>
  <div id="log"></div>

  <script>
    $(function () {
      $("#fetch").on("click", function () {
        var request = $.ajax({
          url: "/api/data",
          type: "GET",
          dataType: "json"
        });

        // Step 1: First handler — logs raw data
        request.done(function (data) {
          $("#log").append("<p>Handler 1: " + data.message + "</p>");
        });

        // Step 2: Second handler — updates UI
        request.done(function (data) {
          $("#log").append("<p>Handler 2: Received " + Object.keys(data).length + " keys.</p>");
        });

        // Step 3: Always runs regardless
        request.always(function () {
          $("#log").append("<p>Request completed.</p>");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Both `.done()` handlers execute in order after success.
- `.always()` handler executes after the `.done()` handlers.

**Why This Output Occurs**
Promise methods allow multiple callbacks for the same outcome, unlike the single `success` settings callback .

### Real-World Cases

- **Modern Codebases**: Using `.done()`/`.fail()` for consistency with ES6 Promises.
- **Multiple UI Updates**: Multiple `.done()` handlers for logging, rendering, and analytics.
- **Legacy Maintenance**: Understanding old code that uses `success`/`error`/`complete`.
- **Migration**: Converting legacy settings callbacks to Promise chains.

### References

- jQuery API — jQuery.ajax() Deprecation Notice – https://api.jquery.com/jQuery.ajax/
- Stack Overflow — jQuery 3.0 callback changes – https://stackoverflow.com/revisions/c87895cc-6494-41cb-aed5-e38712329003/view-source
- Tencent Cloud — jqXHR methods in jQuery 3.x – https://cloud.tencent.cn/developer/ask/sof/112868493

---

## Summary Comparison Table

| Setting | Purpose | Default | Notes |
|---|---|---|---|
| `url` | Target endpoint | None (required) | String |
| `type` / `method` | HTTP method | `"GET"` | `method` alias added in 1.9 |
| `data` | Request payload | None | Object, string, or FormData |
| `contentType` | Format of sent data | `application/x-www-form-urlencoded` | Sets request `Content-Type` header |
| `dataType` | Expected response format | Auto-detected | `json`, `xml`, `html`, `text`, `script`, `jsonp` |
| `headers` | Custom request headers | None | Object of key-value pairs |
| `timeout` | Max wait time (ms) | None | Triggers `"timeout"` error |
| `success` / `.done()` | Success handler | None | `.done()` is modern |
| `error` / `.fail()` | Error handler | None | `.fail()` is modern |
| `complete` / `.always()` | Always handler | None | `.always()` is modern |

---

## Important Notes on Version-Specific Behaviour

1. **jQuery 1.5**: Promise methods (`.done()`, `.fail()`) introduced .
2. **jQuery 1.6**: `.always()` added.
3. **jQuery 1.8**: `jqXHR.success()`, `jqXHR.error()`, `jqXHR.complete()` deprecated .
4. **jQuery 1.9**: `method` alias added for `type`.
5. **jQuery 3.0**: `jqXHR.success()`, `jqXHR.error()`, `jqXHR.complete()` **removed**. Settings callbacks (`success`, `error`, `complete`) **remain supported** .
6. **jQuery 4.1**: `responseURL` property added .

---

## References

- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- Learn Microsoft — Difference between dataType and contentType – https://learn.microsoft.com/da-dk/archive/msdn-technet-forums/c5ee5f75-beb6-4d1f-8439-e9d678bf087f
- Stack Overflow — jQuery 3.0 callback changes – https://stackoverflow.com/revisions/c87895cc-6494-41cb-aed5-e38712329003/view-source
- Stack Overflow — Custom HTTP headers in jQuery – https://stackoverflow.com/posts/14655768/revisions
- Stack Overflow — Timeout and abort – https://stackoverflow.com/questions/21964045/how-to-cancel-an-ajax-request-or-third-party-js-request-when-no-response-after-x
- Stack Overflow — Abort and timeout distinction – https://stackoverflow.com/revisions/95daa398-6025-42f9-ab45-602e9f703593/view-source
- Tencent Cloud — jqXHR methods in jQuery 3.x – https://cloud.tencent.cn/developer/ask/sof/112868493