# XMLHttpRequest — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `XMLHttpRequest` (XHR) is a JavaScript API for making HTTP requests from the browser without reloading the page. It was the original technology behind AJAX and remains in use in legacy codebases.

**Technical Definition:** Per MDN, `XMLHttpRequest` (XHR) objects are used to interact with servers. You can retrieve data from a URL without having to do a full page refresh. This enables a Web page to update just part of a page without disrupting what the user is doing. `XMLHttpRequest` is a Web API whose objects are used to interact with servers. It is defined by the WHATWG XMLHttpRequest Living Standard. Despite its name, `XMLHttpRequest` can be used to retrieve any type of data, not just XML. It provides both synchronous and asynchronous modes of operation (synchronous is deprecated). The API is event-based, using `readystatechange` and other events to notify the caller of progress and completion.

**Beginner-Friendly Explanation:** `XMLHttpRequest` is the "old way" of making HTTP requests in JavaScript. It's how AJAX worked before `fetch()` came along. You create an XHR object, open a connection, set up callbacks for when data arrives, and send the request. It's more verbose than `fetch()`, but it's still found in older codebases, and it has one advantage: built-in progress events.

### Key Characteristics

- **Event-based:** Uses events (`readystatechange`, `load`, `error`, `progress`) rather than Promises.
- **Stateful:** The XHR object maintains an internal state (`readyState`) throughout the request lifecycle.
- **Both sync and async:** Supports synchronous requests (deprecated and discouraged) and asynchronous requests (standard).
- **Progress tracking:** Built-in `progress` events for upload and download progress.
- **Cancellable:** The `abort()` method cancels an in-flight request.
- **Not Promise-based:** Requires manual callback or event listener setup.
- **CORS-aware:** Subject to the same-origin policy and CORS.
- **Legacy:** Largely superseded by the Fetch API, but still used in older code and for progress tracking.

### Prerequisites

- Understanding of JavaScript functions, callbacks, and events.
- Familiarity with HTTP basics (methods, status codes, headers).
- Awareness of the same-origin policy and CORS.
- Basic knowledge of JSON and text parsing.

### Related Programming Areas

- **AJAX:** Asynchronous JavaScript and XML — the technique that XHR enabled.
- **Legacy codebases:** Many older applications still use XHR.
- **File uploads:** XHR provides upload progress events.
- **Real-time updates:** Polling servers for updates.
- **Fetch API comparison:** Understanding XHR helps appreciate the Fetch API.

### Core Concepts / Features

---

## `XMLHttpRequest`

### Definitions

**Core Definition:** `XMLHttpRequest` is a constructor that creates an XHR object used to make HTTP requests.

**Technical Definition:** Per MDN, the `XMLHttpRequest` constructor creates a new `XMLHttpRequest` object. The XHR object has methods like `open()`, `send()`, `abort()`, `setRequestHeader()`, and properties like `readyState`, `status`, `response`, `responseText`, `responseType`, and `timeout`.

### Purposes

- To make HTTP requests without reloading the page (AJAX).
- To retrieve data from a server asynchronously.
- To send data to a server (POST, PUT, DELETE).
- To upload files with progress tracking.
- To maintain compatibility with legacy code.

### Syntax Rules and Structure

```js
const xhr = new XMLHttpRequest();
xhr.open(method, url, async);
xhr.setRequestHeader(header, value);
xhr.send(body);
```

**Component Breakdown:**
- `new XMLHttpRequest()`: Creates a new XHR object.
- `open(method, url, async)`: Initialises the request. `async` defaults to `true`.
- `setRequestHeader(header, value)`: Sets a request header (must be called after `open()` and before `send()`).
- `send(body)`: Sends the request. `body` is optional (e.g., for POST).

### Key Properties

| Property | Description |
|----------|-------------|
| `readyState` | The current state of the request (0–4) |
| `status` | The HTTP status code (e.g., 200, 404, 500) |
| `statusText` | The status message (e.g., "OK", "Not Found") |
| `response` | The response body (type depends on `responseType`) |
| `responseText` | The response as text (throws if `responseType` is not `""` or `"text"`) |
| `responseType` | The expected response type (`""`, `"text"`, `"json"`, `"blob"`, `"arraybuffer"`, `"document"`) |
| `responseURL` | The final URL of the response (after redirects) |
| `responseXML` | The response as an XML Document |
| `timeout` | The request timeout in milliseconds |
| `withCredentials` | Whether to send cookies with cross-origin requests |
| `upload` | An `XMLHttpRequestUpload` object for tracking upload progress |

### Key Methods

| Method | Description |
|--------|-------------|
| `open(method, url, async, user, password)` | Initialises the request |
| `send(body)` | Sends the request |
| `abort()` | Cancels the request |
| `setRequestHeader(header, value)` | Sets a request header |
| `getResponseHeader(header)` | Gets a response header value |
| `getAllResponseHeaders()` | Gets all response headers as a string |
| `overrideMimeType(mimeType)` | Overrides the MIME type of the response |

### Annotated Code Example: Basic GET Request

```js
// Step 1: Create an XHR object.
const xhr = new XMLHttpRequest();

// Step 2: Configure the request.
xhr.open("GET", "https://api.example.com/users/1", true);

// Step 3: Set up the response handler.
xhr.onload = function () {
  if (xhr.status === 200) {
    // Step 4: Parse the response.
    const user = JSON.parse(xhr.responseText);
    console.log(user);
  } else {
    console.error(`HTTP error: ${xhr.status}`);
  }
};

// Step 5: Handle network errors.
xhr.onerror = function () {
  console.error("Network error");
};

// Step 6: Send the request.
xhr.send();
```

**Expected Output:**
```
{ id: 1, name: 'Alice', email: 'alice@example.com' }
```

**Why This Output:** The XHR object is created, configured with `open()`, and sent with `send()`. The `onload` handler checks the status and parses the response.

### Real-World Cases

- **Legacy AJAX applications:** Older web apps that predate `fetch()`.
- **File uploads:** Uploading files with progress tracking.
- **Polling:** Periodically checking a server for updates.
- **Form submission:** Submitting forms via AJAX without page reloads.

---

## Request Lifecycle

### Definitions

**Core Definition:** The request lifecycle is the sequence of states an XHR request goes through, from creation to completion.

**Technical Definition:** Per MDN, the `readyState` property of `XMLHttpRequest` returns the current state of the XMLHttpRequest. The possible values are `UNSENT` (0), `OPENED` (1), `HEADERS_RECEIVED` (2), `LOADING` (3), and `DONE` (4).

### The Lifecycle

| Stage | `readyState` | Description |
|-------|-------------|-------------|
| Created | 0 (`UNSENT`) | The XHR object has been created but `open()` has not been called |
| Opened | 1 (`OPENED`) | `open()` has been called; request is ready to be sent |
| Headers Received | 2 (`HEADERS_RECEIVED`) | `send()` has been called; response headers are available |
| Loading | 3 (`LOADING`) | The response body is being received |
| Done | 4 (`DONE`) | The request is complete (success or failure) |

### Annotated Code Example: Tracking Lifecycle

```js
const xhr = new XMLHttpRequest();

xhr.onreadystatechange = function () {
  console.log(`readyState: ${xhr.readyState}`);

  if (xhr.readyState === 4) {
    // Request is complete
    if (xhr.status === 200) {
      console.log("Success:", xhr.responseText);
    } else {
      console.error("Failed:", xhr.status);
    }
  }
};

xhr.open("GET", "/api/data", true);
xhr.send();
```

**Expected Output:**
```
readyState: 1
readyState: 2
readyState: 3
readyState: 4
Success: {...}
```

**Why This Output:** The `onreadystatechange` event fires each time `readyState` changes. At state 4 (DONE), the status determines success or failure.

### Real-World Cases

- **Progress monitoring:** Tracking the request lifecycle for debugging.
- **State-based logic:** Performing actions at different stages.
- **Legacy code:** Understanding the XHR lifecycle in older codebases.

---

## Ready States

### Definitions

**Core Definition:** Ready states are the numeric values (0–4) representing the current state of an XHR request.

**Technical Definition:** Per MDN, the `readyState` property returns the current state of the XMLHttpRequest. The values are defined as constants on `XMLHttpRequest`: `UNSENT` (0), `OPENED` (1), `HEADERS_RECEIVED` (2), `LOADING` (3), `DONE` (4).

### Ready State Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `XMLHttpRequest.UNSENT` | 0 | Client has been created; `open()` not called |
| `XMLHttpRequest.OPENED` | 1 | `open()` has been called |
| `XMLHttpRequest.HEADERS_RECEIVED` | 2 | `send()` has been called; headers received |
| `XMLHttpRequest.LOADING` | 3 | Downloading the response body |
| `XMLHttpRequest.DONE` | 4 | Operation complete |

### Annotated Code Example: Using Ready States

```js
const xhr = new XMLHttpRequest();

xhr.onreadystatechange = function () {
  switch (xhr.readyState) {
    case XMLHttpRequest.UNSENT:
      console.log("UNSENT: Client created");
      break;
    case XMLHttpRequest.OPENED:
      console.log("OPENED: open() called");
      break;
    case XMLHttpRequest.HEADERS_RECEIVED:
      console.log("HEADERS_RECEIVED: Headers available");
      console.log(xhr.getAllResponseHeaders());
      break;
    case XMLHttpRequest.LOADING:
      console.log("LOADING: Receiving body");
      break;
    case XMLHttpRequest.DONE:
      console.log("DONE: Request complete");
      console.log("Status:", xhr.status);
      console.log("Response:", xhr.responseText);
      break;
  }
};

xhr.open("GET", "/api/data", true);
xhr.send();
```

**Expected Output:**
```
UNSENT: Client created
OPENED: open() called
HEADERS_RECEIVED: Headers available
LOADING: Receiving body
DONE: Request complete
Status: 200
Response: {...}
```

### Real-World Cases

- **Debugging:** Understanding where a request is in its lifecycle.
- **Progress UI:** Updating the UI based on the current state.
- **Legacy code maintenance:** Reading and modifying XHR-based code.

---

## Status Codes

### Definitions

**Core Definition:** HTTP status codes are numeric codes returned by the server indicating the result of the request.

**Technical Definition:** Per MDN, HTTP response status codes indicate whether a specific HTTP request has been successfully completed. They are grouped into five classes: informational responses (1xx), successful responses (2xx), redirects (3xx), client errors (4xx), and server errors (5xx).

### Common Status Codes

| Code | Name | Description |
|------|------|-------------|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created (POST) |
| 204 | No Content | Success, no body |
| 301 | Moved Permanently | Resource moved permanently |
| 302 | Found | Temporary redirect |
| 304 | Not Modified | Cached version is still valid |
| 400 | Bad Request | Invalid request syntax |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 500 | Internal Server Error | Server error |
| 502 | Bad Gateway | Invalid response from upstream |
| 503 | Service Unavailable | Server temporarily unavailable |

### Annotated Code Example: Handling Status Codes

```js
const xhr = new XMLHttpRequest();

xhr.onload = function () {
  switch (xhr.status) {
    case 200:
      console.log("Success:", JSON.parse(xhr.responseText));
      break;
    case 201:
      console.log("Created:", JSON.parse(xhr.responseText));
      break;
    case 204:
      console.log("No content");
      break;
    case 400:
      console.error("Bad request");
      break;
    case 401:
      console.error("Unauthorized — please log in");
      break;
    case 404:
      console.error("Resource not found");
      break;
    case 500:
      console.error("Server error");
      break;
    default:
      console.error(`Unexpected status: ${xhr.status}`);
  }
};

xhr.open("GET", "/api/users/1", true);
xhr.send();
```

**Expected Output:** The appropriate message is logged based on the status code.

### Real-World Cases

- **Error handling:** Distinguishing between different error types.
- **User feedback:** Showing appropriate messages for different statuses.
- **Retry logic:** Retrying on 5xx errors but not 4xx.

---

## Response Types

### Definitions

**Core Definition:** The `responseType` property specifies the type of data expected in the response, controlling how the response is parsed.

**Technical Definition:** Per MDN, the `XMLHttpRequest.responseType` property is an enumerated string value specifying the type of data contained in the response. The possible values are `""` (default, text), `"text"`, `"json"`, `"blob"`, `"arraybuffer"`, and `"document"`.

### Response Type Values

| Value | Description | `response` Property Type |
|-------|-------------|-------------------------|
| `""` | Default; response is text | `String` |
| `"text"` | Response is text | `String` |
| `"json"` | Response is parsed JSON | `Object` |
| `"blob"` | Response is binary data | `Blob` |
| `"arraybuffer"` | Response is binary data | `ArrayBuffer` |
| `"document"` | Response is an HTML/XML document | `Document` |

### Annotated Code Examples

#### Example 1: JSON Response

```js
const xhr = new XMLHttpRequest();
xhr.open("GET", "/api/users");
xhr.responseType = "json"; // Automatically parse JSON

xhr.onload = function () {
  console.log(xhr.response); // Parsed JavaScript object
  console.log(xhr.response[0].name);
};

xhr.send();
```

**Expected Output:**
```
[{ id: 1, name: 'Alice' }]
Alice
```

#### Example 2: Blob Response (Image)

```js
const xhr = new XMLHttpRequest();
xhr.open("GET", "/image.png");
xhr.responseType = "blob";

xhr.onload = function () {
  const blob = xhr.response;
  const imageUrl = URL.createObjectURL(blob);

  const img = document.createElement("img");
  img.src = imageUrl;
  document.body.appendChild(img);
};

xhr.send();
```

**Expected Output:** The image is displayed on the page.

### Real-World Cases

- **JSON APIs:** Using `responseType = "json"` for automatic parsing.
- **File downloads:** Using `responseType = "blob"` for images, PDFs, etc.
- **Binary processing:** Using `responseType = "arraybuffer"`.
- **HTML parsing:** Using `responseType = "document"`.

---

## Progress Events

### Definitions

**Core Definition:** Progress events are events fired by XHR during the request lifecycle to report progress on uploads and downloads.

**Technical Definition:** Per MDN, the `ProgressEvent` interface represents events that measure the progress of an underlying process, like an HTTP request. XHR supports `progress` events on the `XMLHttpRequest` object (for downloads) and on `XMLHttpRequest.upload` (for uploads).

### Progress Event Types

| Event | Description |
|-------|-------------|
| `loadstart` | Fired when the request starts |
| `progress` | Fired periodically as data is transferred |
| `load` | Fired when the request completes successfully |
| `loadend` | Fired when the request completes (success or failure) |
| `error` | Fired when the request fails |
| `abort` | Fired when the request is aborted |
| `timeout` | Fired when the request times out |

### Annotated Code Example: Download Progress

```js
const xhr = new XMLHttpRequest();
xhr.open("GET", "/large-file.zip", true);

xhr.onprogress = function (event) {
  if (event.lengthComputable) {
    const percent = (event.loaded / event.total) * 100;
    console.log(`Downloaded: ${percent.toFixed(1)}%`);
  } else {
    console.log(`Downloaded: ${event.loaded} bytes`);
  }
};

xhr.onload = function () {
  console.log("Download complete");
};

xhr.send();
```

**Expected Output:**
```
Downloaded: 12.5%
Downloaded: 45.0%
Downloaded: 78.3%
Downloaded: 100.0%
Download complete
```

### Annotated Code Example: Upload Progress

```js
const xhr = new XMLHttpRequest();
xhr.open("POST", "/upload", true);

xhr.upload.onprogress = function (event) {
  if (event.lengthComputable) {
    const percent = (event.loaded / event.total) * 100;
    console.log(`Uploaded: ${percent.toFixed(1)}%`);
  }
};

const formData = new FormData();
formData.append("file", fileInput.files[0]);

xhr.send(formData);
```

**Expected Output:**
```
Uploaded: 25.0%
Uploaded: 50.0%
Uploaded: 75.0%
Uploaded: 100.0%
```

### Real-World Cases

- **File uploads:** Showing upload progress bars.
- **Large downloads:** Displaying download progress.
- **User feedback:** Keeping the user informed during long operations.

---

## Error Handling

### Definitions

**Core Definition:** Error handling in XHR involves handling network errors, HTTP errors, timeouts, and aborts.

**Technical Definition:** Per MDN, XHR provides several error-related events: `error` (network failure), `abort` (request aborted), `timeout` (request timed out), and `load` (successful completion). The `status` property indicates HTTP errors.

### Error Types

| Error Type | Event | How to Handle |
|------------|-------|---------------|
| Network error | `error` | Check `xhr.onerror` |
| HTTP error (4xx, 5xx) | `load` | Check `xhr.status` |
| Timeout | `timeout` | Set `xhr.timeout` and handle `ontimeout` |
| Abort | `abort` | Call `xhr.abort()` and handle `onabort` |
| Parse error | `load` | Wrap `JSON.parse()` in `try...catch` |

### Annotated Code Example: Complete Error Handling

```js
const xhr = new XMLHttpRequest();
xhr.open("GET", "/api/data", true);
xhr.timeout = 5000; // 5 seconds

xhr.onload = function () {
  if (xhr.status >= 200 && xhr.status < 300) {
    try {
      const data = JSON.parse(xhr.responseText);
      console.log("Success:", data);
    } catch (e) {
      console.error("Parse error:", e.message);
    }
  } else {
    console.error(`HTTP error: ${xhr.status} ${xhr.statusText}`);
  }
};

xhr.onerror = function () {
  console.error("Network error");
};

xhr.ontimeout = function () {
  console.error("Request timed out");
};

xhr.onabort = function () {
  console.error("Request aborted");
};

xhr.send();
```

**Expected Output:** Different error types produce different messages.

### Real-World Cases

- **Offline handling:** Detecting network errors.
- **Timeout handling:** Aborting slow requests.
- **User cancellation:** Handling user-initiated aborts.
- **Error reporting:** Logging errors for debugging.

---

## XHR vs Fetch

### Definitions

**Core Definition:** XHR and Fetch are two APIs for making HTTP requests. Fetch is the modern replacement, but each has strengths and weaknesses.

### Comparison

| Feature | XHR | Fetch |
|---------|-----|-------|
| **API style** | Event-based, callback | Promise-based, `async/await` |
| **Readability** | Verbose | Concise |
| **Progress events** | Built-in (`onprogress`) | Requires manual stream reading |
| **Request cancellation** | `xhr.abort()` | `AbortController` |
| **HTTP error rejection** | No (must check `status`) | No (must check `response.ok`) |
| **Synchronous mode** | Yes (deprecated) | No |
| **Streaming** | Limited | Yes (`ReadableStream`) |
| **Service Worker support** | No | Yes |
| **Cookies** | `withCredentials` | `credentials` option |
| **Timeout** | `xhr.timeout` | `AbortSignal.timeout()` |
| **Redirect control** | `xhr.responseURL` | `redirect` option |
| **Caching control** | Limited | `cache` option |
| **Browser support** | Universal | Modern browsers |

### Annotated Code Example: XHR vs Fetch

```js
// XHR:
function xhrGet(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`HTTP ${xhr.status}`));
      }
    };
    xhr.onerror = () => reject(new Error("Network error"));
    xhr.send();
  });
}

// Fetch:
async function fetchGet(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}
```

**Expected Output:** Both functions return the parsed JSON data, but `fetchGet` is more concise.

### When to Use XHR vs Fetch

| Use XHR When | Use Fetch When |
|--------------|----------------|
| You need upload progress events | You want clean, Promise-based code |
| You're maintaining legacy code | You're writing new code |
| You need synchronous requests (rare) | You need streaming responses |
| You need fine-grained control over the request | You want modern, standard APIs |

### Real-World Cases

- **Legacy codebases:** XHR is still common in older applications.
- **New projects:** Fetch is the standard choice.
- **Progress tracking:** XHR's upload progress is easier to use.
- **Service Workers:** Only Fetch is available.

---

## References

- MDN Web Docs – XMLHttpRequest — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
- MDN Web Docs – Using XMLHttpRequest — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_XMLHttpRequest
- MDN Web Docs – XMLHttpRequest: readyState — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/readyState
- MDN Web Docs – XMLHttpRequest: responseType — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseType
- MDN Web Docs – XMLHttpRequest: upload — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/upload
- MDN Web Docs – ProgressEvent — https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent
- MDN Web Docs – HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- WHATWG – XMLHttpRequest Living Standard — https://xhr.spec.whatwg.org/
- WHATWG – Fetch Living Standard — https://fetch.spec.whatwg.org/
- Google Developers – High Performance Ajax Applications — https://developers.google.com/
- SitePoint – XMLHttpRequest vs Fetch API — https://www.sitepoint.com/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf