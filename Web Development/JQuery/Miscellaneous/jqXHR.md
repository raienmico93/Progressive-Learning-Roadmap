# jqXHR — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** jqXHR is the jQuery XMLHttpRequest object returned by all of jQuery's Ajax methods (such as `$.ajax()`, `$.get()`, and `$.post()`) starting from jQuery 1.5. It is a superset of the native `XMLHttpRequest` object that additionally implements the jQuery Promise interface, providing a unified API for handling asynchronous HTTP requests.

**Technical Definition:** The jqXHR object is a JavaScript object returned by `jQuery.ajax()` and its shorthand derivatives. It implements the Promise interface by inheriting from `jQuery.Deferred`, exposing consumer-facing methods such as `.done()`, `.fail()`, `.always()`, `.then()`, and `.catch()`. Simultaneously, it mimics the native `XMLHttpRequest` API by exposing properties such as `status`, `statusText`, `responseText`, and `responseXML`, as well as methods such as `.setRequestHeader()`, `.getResponseHeader()`, `.getAllResponseHeaders()`, `.abort()`, and `.overrideMimeType()`. The object is returned immediately upon making an Ajax request, before the request completes, allowing callbacks to be attached at any time — even after the request has already finished, in which case the callbacks fire immediately.

**Beginner-Friendly Explanation:** When you ask jQuery to fetch something from a server, it hands you back a “receipt” called a jqXHR object. This receipt lets you do two things: it lets you say “call this function if the request succeeds” (using `.done()`), “call this function if it fails” (using `.fail()`), or “call this function no matter what” (using `.always()`). It also lets you check the details of the response — like the HTTP status code, the response text, or the headers the server sent back. Think of it as a remote control for your Ajax request: you can watch its progress, check its outcome, and even cancel it if needed.

### Key Characteristics

- **Promise interface:** jqXHR implements the jQuery Promise interface, allowing `.done()`, `.fail()`, `.always()`, `.then()`, and `.catch()` to be chained on the returned object.
- **Native XHR compatibility:** For backward compatibility, jqXHR exposes `readyState`, `status`, `statusText`, `responseXML`, `responseText`, `setRequestHeader()`, `getAllResponseHeaders()`, `getResponseHeader()`, and `abort()`.
- **Late-binding callbacks:** Callbacks can be attached even after the request has completed; if the request is already finished, the callback fires immediately.
- **Removal of legacy methods (jQuery 3.0):** The `jqXHR.success()`, `jqXHR.error()`, and `jqXHR.complete()` methods were deprecated in jQuery 1.8 and **removed in jQuery 3.0**. The option-object callbacks (`success`, `error`, `complete` passed to `$.ajax()`) are **not** deprecated and remain available.
- **Promises/A+ compatibility (jQuery 3.0+):** Exceptions thrown in `.then()` callbacks are caught and converted into rejection values; `.catch()` is available as an alias for `.then(null, fn)`.
- **Returned immediately:** The jqXHR object is returned synchronously from `$.ajax()`, even though the HTTP request is asynchronous by default.

### Prerequisites

- Basic understanding of JavaScript, particularly asynchronous programming and callbacks.
- Familiarity with jQuery fundamentals, including the `$` object and event handling.
- Awareness of the jQuery Deferred object, since jqXHR inherits from it.
- Basic knowledge of HTTP concepts (status codes, headers, request/response cycle).

### Related Programming Areas

- **AJAX (Asynchronous JavaScript and XML):** The primary use case for jqXHR is managing Ajax requests.
- **Promise-Based Architecture:** jqXHR is a concrete implementation of the jQuery Deferred/Promise pattern.
- **HTTP Client Programming:** Managing headers, status codes, and response parsing.
- **Event-Driven Programming:** Callbacks are executed in response to request lifecycle events.
- **REST API Consumption:** jqXHR is commonly used to interact with RESTful web services.

### Core Concepts / Features

This cheat sheet covers six core concepts: the AJAX Promise interface, success callbacks, error callbacks, completion handlers, essential jqXHR properties, and header management methods.

---

## Core Concept 1: AJAX Promise Interface (Super-Object Implementing the jQuery Promise Interface)

### Definitions

**Core Definition:** The jqXHR object implements the jQuery Promise interface, making it a “super-object” that combines the functionality of a Promise with the properties and methods of the native XMLHttpRequest object.

**Technical Definition:** Starting with jQuery 1.5, all of jQuery's Ajax methods return a jqXHR object that is a superset of the native `XMLHttpRequest` object. This object implements the Promise interface by inheriting from `jQuery.Deferred`, giving it all the properties, methods, and behavior of a Promise. This means it exposes consumer-facing methods (`.done()`, `.fail()`, `.always()`, `.then()`, `.catch()`, `.progress()`, `.state()`) while hiding the state-mutating methods (`.resolve()`, `.reject()`, `.notify()`) that are reserved for the Deferred creator. The Promise interface allows multiple `.done()`, `.fail()`, and `.always()` callbacks to be chained on a single request, and callbacks can be assigned even after the request has completed.

**Beginner-Friendly Explanation:** The jqXHR object is like a Swiss Army knife. On one side, it acts like a Promise — you can attach success, failure, and completion handlers to it, and chain them together. On the other side, it acts like a traditional XMLHttpRequest — you can check the status code, read the response text, and manage headers. It combines both capabilities into a single object, so you do not need to juggle two separate interfaces.

### Purposes

- To provide a unified interface for managing Ajax requests that combines Promise-based callback management with native XHR properties and methods.
- To allow multiple callbacks to be attached to a single request, with support for late binding.
- To enable chaining of callback methods for cleaner, more readable asynchronous code.
- To serve as a drop-in replacement for the native XMLHttpRequest object while adding Promise functionality.
- To integrate seamlessly with jQuery's Deferred system, allowing jqXHR objects to be used with `$.when()` and other Deferred-aware utilities.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
var jqxhr = $.ajax( url [, settings ] );
```

| Component | Description |
|-----------|-------------|
| `$.ajax( url [, settings ] )` | Performs an asynchronous HTTP request. Returns a jqXHR object. |
| `jqxhr` | The returned jqXHR object implementing the Promise interface. |

**Promise Methods Available on jqXHR:**

| Method | Description |
|--------|-------------|
| `.done( fn )` | Adds a callback to be called when the request succeeds. |
| `.fail( fn )` | Adds a callback to be called when the request fails. |
| `.always( fn )` | Adds a callback to be called regardless of outcome. |
| `.then( fn )` | Adds handlers for success, failure, and progress. |
| `.catch( fn )` | Alias for `.then( null, fn )`; handles rejection. |
| `.progress( fn )` | Adds a callback for progress notifications. |
| `.state()` | Returns the current state: `"pending"`, `"resolved"`, or `"rejected"`. |

**Syntax Rules:**

- The jqXHR object is returned **immediately** when the Ajax request is initiated, before the request completes.
- Callbacks can be attached at any time, including after the request has finished; in that case, they execute immediately.
- The jqXHR object is **chainable** — each callback method returns the jqXHR object (or a Promise from `.then()`).
- The jqXHR object implements the full Promise interface, but the state-mutating methods (`.resolve()`, `.reject()`, `.notify()`) are **not** exposed.

**Constraints and Limitations:**

- As of jQuery 3.0, exceptions thrown inside `.then()` callbacks are caught and converted into rejection values (Promises/A+ behavior); use `.catch()` to handle them.
- The `.then()` method follows Promises/A+ semantics in jQuery 3.0+, meaning it passes a single value to handlers and does not set `this` context; use `.done()` and `.fail()` for backward-compatible multi-argument behavior.
- `jqXHR.success()`, `jqXHR.error()`, and `jqXHR.complete()` were removed in jQuery 3.0 and must be replaced with `.done()`, `.fail()`, and `.always()`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Chaining Promise Methods on a jqXHR Object**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>jqXHR promise interface demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    // Step 1: Make an Ajax request and store the jqXHR object
    var jqxhr = $.ajax({
      url: "https://jsonplaceholder.typicode.com/posts/1",
      dataType: "json"
    });

    // Step 2: Chain multiple callbacks on the jqXHR object
    jqxhr
      .done(function(data) {
        // This runs if the request succeeds
        $("#log").append("SUCCESS: " + data.title + "<br>");
      })
      .fail(function(jqXHR, textStatus, errorThrown) {
        // This runs if the request fails
        $("#log").append("FAIL: " + textStatus + " — " + errorThrown + "<br>");
      })
      .always(function() {
        // This runs regardless of outcome
        $("#log").append("COMPLETE: Request finished.<br>");
      });

    // Step 3: Demonstrate late binding — attach another done callback
    // If the request is already complete, this fires immediately
    jqxhr.done(function() {
      $("#log").append("LATE CALLBACK: Also fired.<br>");
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
SUCCESS: sunt aut facere repellat provident occaecati excepturi optio reprehenderit
LATE CALLBACK: Also fired.
COMPLETE: Request finished.
```

**Why this output:** The `.done()` callback fires first with the parsed JSON data (the post title). The `.always()` callback fires after `.done()` and `.fail()` callbacks. The late-bound `.done()` callback fires immediately (if the request has already completed by the time it is attached) or when the request completes, and because it was added after the first `.done()` callback, it executes in registration order relative to other callbacks added at the same time.

---

**Example 2: Using `$.when()` with jqXHR**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>$.when with jqXHR</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="output"></p>

  <script>
    // Step 1: Make two Ajax requests
    var request1 = $.ajax({
      url: "https://jsonplaceholder.typicode.com/posts/1",
      dataType: "json"
    });

    var request2 = $.ajax({
      url: "https://jsonplaceholder.typicode.com/posts/2",
      dataType: "json"
    });

    // Step 2: Use $.when() to wait for both requests
    $.when(request1, request2).done(function(data1, data2) {
      // data1 and data2 are arrays: [data, textStatus, jqXHR]
      var post1 = data1[0];
      var post2 = data2[0];
      $("#output").text(
        "Both loaded: \"" + post1.title + "\" and \"" + post2.title + "\""
      );
    }).fail(function() {
      $("#output").text("One or more requests failed.");
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Both loaded: "sunt aut facere repellat provident occaecati excepturi optio reprehenderit" and "qui est esse"
```

**Why this output:** `$.when()` accepts multiple Deferred/Promise objects and returns a new Promise that resolves when all of them resolve. The `.done()` callback receives the resolution values from each request as arrays containing `[data, textStatus, jqXHR]`. The script extracts the first element of each array to get the parsed JSON data.

### Real-World Cases

- **Parallel data loading:** A dashboard uses `$.when()` with multiple jqXHR objects to load data from several endpoints simultaneously and render the UI only when all data has arrived.
- **Chained API calls:** An authentication flow chains `.then()` on a jqXHR to make a follow-up request using the token received from the first request.
- **Promise composition:** A module returns a jqXHR object from a public method, allowing callers to attach `.done()` and `.fail()` handlers without exposing the internal request logic.

---

## Core Concept 2: Success Callbacks — `.done()` (Replacement of Legacy `success`)

### Definitions

**Core Definition:** `.done()` is a method on the jqXHR object that registers a callback to be executed when the Ajax request completes successfully.

**Technical Definition:** `jqXHR.done( doneCallbacks )` adds handlers to be called when the Deferred (jqXHR) object is resolved. The method accepts one or more functions or arrays of functions, which are executed in the order they were added. Each callback receives the arguments passed to the Deferred's resolution, which for an Ajax request are: the parsed response data, the text status (`"success"`), and the jqXHR object itself (in that order). The method returns the jqXHR object for chaining. This method replaces the legacy `jqXHR.success()` callback, which was deprecated in jQuery 1.8 and removed in jQuery 3.0.

**Beginner-Friendly Explanation:** `.done()` says “when this request finishes successfully, run this function.” The function receives the data from the server, a status string, and the jqXHR object itself. You can attach multiple `.done()` callbacks to the same request, and they all run in the order you added them.

### Purposes

- To register callbacks that execute when an Ajax request completes successfully.
- To process the response data returned by the server.
- To chain multiple success handlers on a single request, each performing a different task.
- To replace the deprecated `jqXHR.success()` method with a Promise-compatible alternative.
- To attach success handlers after the request has already completed, in which case the handler fires immediately.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
jqxhr.done( doneCallbacks [, doneCallbacks] );
```

| Component | Description |
|-----------|-------------|
| `jqxhr` | The jqXHR object returned by an Ajax method. |
| `doneCallbacks` | A function or array of functions called when the request succeeds. Receives `(data, textStatus, jqXHR)`. |

**Callback Arguments:**

| Argument | Description |
|----------|-------------|
| `data` | The parsed response data (format depends on `dataType`). |
| `textStatus` | A string describing the status (`"success"`). |
| `jqXHR` | The jqXHR object itself. |

**Syntax Rules:**

- Multiple callbacks can be passed as separate arguments or as an array.
- Callbacks execute in the order they were added.
- The method returns the jqXHR object for chaining.
- If the request has already succeeded by the time `.done()` is called, the callback fires immediately.

**Constraints and Limitations:**

- In jQuery 3.0+, `.then()` follows Promises/A+ semantics (single value, no `this` context); use `.done()` for backward-compatible multi-argument behavior.
- The legacy `jqXHR.success()` method was removed in jQuery 3.0; `.done()` is the replacement.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Attaching Multiple `.done()` Callbacks**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>done() demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    // Step 1: Make an Ajax request
    $.ajax({
      url: "https://jsonplaceholder.typicode.com/todos/1",
      dataType: "json"
    })
    .done(function(data, textStatus, jqXHR) {
      // First done callback: log the data
      $("#log").append("Data: " + JSON.stringify(data) + "<br>");
    })
    .done(function(data, textStatus, jqXHR) {
      // Second done callback: log the status
      $("#log").append("Status: " + textStatus + "<br>");
    })
    .done(function(data, textStatus, jqXHR) {
      // Third done callback: log the HTTP status code from jqXHR
      $("#log").append("HTTP Status: " + jqXHR.status + "<br>");
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Data: {"userId":1,"id":1,"title":"delectus aut autem","completed":false}
Status: success
HTTP Status: 200
```

**Why this output:** Three `.done()` callbacks are chained in order. Each receives the same arguments: the parsed JSON data, the text status (`"success"`), and the jqXHR object. The callbacks execute in the order they were registered.

### Real-World Cases

- **Rendering server data:** A `.done()` callback takes the JSON response and builds HTML elements to insert into the page.
- **Updating application state:** A `.done()` callback updates a local data store or model with the response data.
- **Triggering follow-up requests:** A `.done()` callback uses the response data (e.g., an authentication token) to make a subsequent Ajax request.

---

## Core Concept 3: Error Callbacks — `.fail()` (Replacement of Legacy `error`)

### Definitions

**Core Definition:** `.fail()` is a method on the jqXHR object that registers a callback to be executed when the Ajax request fails.

**Technical Definition:** `jqXHR.fail( failCallbacks )` adds handlers to be called when the Deferred (jqXHR) object is rejected. The method accepts one or more functions or arrays of functions, which are executed in the order they were added. Each callback receives the arguments passed to the Deferred's rejection, which for an Ajax request are: the jqXHR object, a string describing the type of error that occurred (`"timeout"`, `"error"`, `"abort"`, or `"parsererror"`), and an optional exception object (the textual portion of the HTTP status, such as `"Not Found"` or `"Internal Server Error"`). The method returns the jqXHR object for chaining. This method replaces the legacy `jqXHR.error()` callback, which was deprecated in jQuery 1.8 and removed in jQuery 3.0.

**Beginner-Friendly Explanation:** `.fail()` says “when this request fails, run this function.” The function receives the jqXHR object, a string telling you what went wrong, and an optional error message from the server. You can attach multiple `.fail()` callbacks to handle different aspects of the error.

### Purposes

- To register callbacks that execute when an Ajax request fails.
- To handle HTTP errors (404, 500, etc.) and network failures gracefully.
- To display error messages to the user or log them for debugging.
- To replace the deprecated `jqXHR.error()` method with a Promise-compatible alternative.
- To chain multiple failure handlers on a single request, each performing a different recovery or notification task.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
jqxhr.fail( failCallbacks [, failCallbacks] );
```

| Component | Description |
|-----------|-------------|
| `jqxhr` | The jqXHR object returned by an Ajax method. |
| `failCallbacks` | A function or array of functions called when the request fails. Receives `(jqXHR, textStatus, errorThrown)`. |

**Callback Arguments:**

| Argument | Description |
|----------|-------------|
| `jqXHR` | The jqXHR object. |
| `textStatus` | A string: `"timeout"`, `"error"`, `"abort"`, or `"parsererror"`. |
| `errorThrown` | An optional exception object or string (e.g., `"Not Found"`). |

**Syntax Rules:**

- Multiple callbacks can be passed as separate arguments or as an array.
- Callbacks execute in the order they were added.
- The method returns the jqXHR object for chaining.
- `.fail()` is the Promise-interface replacement for the deprecated `jqXHR.error()` method.

**Constraints and Limitations:**

- The `errorThrown` argument may be `undefined` in some cases (e.g., cross-domain requests without proper CORS headers).
- The legacy `jqXHR.error()` method was removed in jQuery 3.0; `.fail()` is the replacement.
- In jQuery 3.0+, exceptions thrown inside `.then()` callbacks are caught and converted into rejection values; use `.catch()` on `.then()` chains for Promises/A+ behavior.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Handling a Failed Request**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>fail() demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    // Step 1: Request a non-existent resource
    $.ajax({
      url: "https://jsonplaceholder.typicode.com/nonexistent",
      dataType: "json"
    })
    .fail(function(jqXHR, textStatus, errorThrown) {
      // Step 2: Handle the failure
      $("#log").append("Request failed.<br>");
      $("#log").append("HTTP Status: " + jqXHR.status + "<br>");
      $("#log").append("Status Text: " + jqXHR.statusText + "<br>");
      $("#log").append("Error Type: " + textStatus + "<br>");
      $("#log").append("Error Thrown: " + errorThrown + "<br>");
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Request failed.
HTTP Status: 404
Status Text: Not Found
Error Type: error
Error Thrown: Not Found
```

**Why this output:** The request to a non-existent URL returns a 404 status. The `.fail()` callback receives the jqXHR object (with `status = 404` and `statusText = "Not Found"`), the text status `"error"`, and the error thrown string `"Not Found"`. The callback logs all of these details.

### Real-World Cases

- **User-friendly error messages:** A `.fail()` callback displays a “Something went wrong” message to the user instead of a raw error.
- **Retry logic:** A `.fail()` callback checks the status code and retries the request if it was a temporary server error (e.g., 503).
- **Logging and monitoring:** A `.fail()` callback logs the error details to a monitoring service or the browser console.

---

## Core Concept 4: Completion Handlers — `.always()` (Replacement of Legacy `complete`)

### Definitions

**Core Definition:** `.always()` is a method on the jqXHR object that registers a callback to be executed when the Ajax request completes, regardless of whether it succeeded or failed.

**Technical Definition:** `jqXHR.always( alwaysCallbacks )` adds handlers to be called when the Deferred (jqXHR) object is either resolved or rejected. The method accepts one or more functions or arrays of functions, which are executed in the order they were added. The callbacks receive the arguments that were used to resolve or reject the Deferred — for a successful request, these are `(data, textStatus, jqXHR)`; for a failed request, these are `(jqXHR, textStatus, errorThrown)`. Because the arguments differ significantly between success and failure, `.always()` is best used only for actions that do not require inspecting the arguments. The method returns the jqXHR object for chaining. This method was added in jQuery 1.6 and replaces the legacy `jqXHR.complete()` callback, which was deprecated in jQuery 1.8 and removed in jQuery 3.0.

**Beginner-Friendly Explanation:** `.always()` says “no matter what happens — success or failure — run this function.” It is useful for cleanup tasks like hiding a loading spinner or re-enabling a form. But because the arguments passed to `.always()` differ depending on whether the request succeeded or failed, you should not rely on them inside the callback.

### Purposes

- To execute cleanup logic that must run regardless of whether the Ajax request succeeded or failed.
- To hide loading indicators or reset UI state after a request completes.
- To log the completion of a request without caring about its outcome.
- To replace the deprecated `jqXHR.complete()` method with a Promise-compatible alternative.
- To chain a final action that runs after either `.done()` or `.fail()` callbacks have executed.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
jqxhr.always( alwaysCallbacks [, alwaysCallbacks] );
```

| Component | Description |
|-----------|-------------|
| `jqxhr` | The jqXHR object returned by an Ajax method. |
| `alwaysCallbacks` | A function or array of functions called when the request completes (success or failure). |

**Callback Arguments:**

| Outcome | Arguments |
|---------|-----------|
| Success | `(data, textStatus, jqXHR)` |
| Failure | `(jqXHR, textStatus, errorThrown)` |

**Syntax Rules:**

- Multiple callbacks can be passed as separate arguments or as an array.
- Callbacks execute in the order they were added.
- The method returns the jqXHR object for chaining.
- The arguments passed to `.always()` differ between success and failure; avoid inspecting them inside the callback.

**Constraints and Limitations:**

- Because the arguments differ between resolution and rejection, `.always()` is best used only for actions that do not inspect the arguments.
- The legacy `jqXHR.complete()` method was removed in jQuery 3.0; `.always()` is the replacement.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Using `.always()` for Cleanup**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>always() demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="spinner">Loading...</div>
  <p id="result"></p>

  <script>
    // Step 1: Show the loading spinner
    $("#spinner").show();

    // Step 2: Make an Ajax request
    $.ajax({
      url: "https://jsonplaceholder.typicode.com/posts/1",
      dataType: "json"
    })
    .done(function(data) {
      $("#result").text("Loaded: " + data.title);
    })
    .fail(function() {
      $("#result").text("Failed to load.");
    })
    .always(function() {
      // Step 3: This runs regardless of outcome
      $("#spinner").hide();
      $("#result").append(" — Spinner hidden.");
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Loaded: sunt aut facere repellat provident occaecati excepturi optio reprehenderit — Spinner hidden.
```

**Why this output:** The request succeeds, triggering the `.done()` callback, which sets the result text. The `.always()` callback runs regardless of outcome, hiding the spinner and appending the cleanup message.

### Real-World Cases

- **Loading spinners:** A spinner is hidden in an `.always()` callback after an Ajax request completes, regardless of success or failure.
- **Form re-enabling:** A submit button is re-enabled in `.always()` after a form submission request completes.
- **Connection cleanup:** A database connection or resource is released in `.always()` after a request finishes.

---

## Enhanced Topic: Essential jqXHR Properties (`status`, `statusText`, `responseText`, `responseXML`)

### Definitions

**Core Definition:** The jqXHR object exposes several properties inherited from the native `XMLHttpRequest` interface that provide information about the HTTP response, including the status code, status text, and the response body in text or XML format.

**Technical Definition:** For backward compatibility with `XMLHttpRequest`, the jqXHR object exposes the following properties: `readyState` (the client's state), `status` (the numeric HTTP status code), `statusText` (the HTTP status message), `responseText` (the response body as a string), and `responseXML` (the response body as an XML Document, when applicable). Additionally, `responseJSON` is available when the response Content-Type is JSON. These properties are read-only and reflect the state of the underlying HTTP request at the time they are accessed.

**Beginner-Friendly Explanation:** These properties are like the labels on a package you received. `status` tells you the numeric code (e.g., 200 for OK, 404 for Not Found). `statusText` gives you the human-readable message (e.g., “OK,” “Not Found”). `responseText` is the actual content of the response (like the text of a letter). `responseXML` is the same content but formatted as XML. These properties let you inspect the details of the server's response after an Ajax request completes.

### Purposes

- To retrieve the numeric HTTP status code for conditional logic (e.g., handling 404 vs. 500 errors).
- To display human-readable status messages to the user.
- To access the raw response body as a string when the parsed `data` argument is not sufficient.
- To parse XML responses into a DOM Document for further processing.
- To inspect the response body in `.fail()` callbacks for detailed error information.

### Syntax Rules and Structure

**Property Access Syntax:**
```javascript
jqxhr.status        // Number (e.g., 200, 404, 500)
jqxhr.statusText    // String (e.g., "OK", "Not Found")
jqxhr.responseText  // String (response body as text)
jqxhr.responseXML   // Document or null (response body as XML)
jqxhr.responseJSON  // Object or undefined (if Content-Type is JSON)
```

**Property Descriptions:**

| Property | Type | Description |
|----------|------|-------------|
| `status` | Number | HTTP status code (e.g., 200, 404, 500). |
| `statusText` | String | HTTP status message (e.g., `"OK"`, `"Not Found"`). |
| `responseText` | String | Response body as a string. |
| `responseXML` | Document | Response body as an XML Document (if applicable). |
| `responseJSON` | Object | Parsed JSON response (if Content-Type is JSON). |
| `readyState` | Number | The client's current state (0–4). |

**Syntax Rules:**

- These properties are **read-only**; they cannot be set by the consumer.
- `responseText` and `responseXML` are only populated when the response has been fully received.
- `responseXML` is `null` if the response is not XML or if parsing failed.
- `responseJSON` is populated when the response Content-Type is `application/json`.

**Constraints and Limitations:**

- Accessing `responseText` before the request completes may throw an `InvalidStateError` DOMException.
- `responseXML` may be `null` if the response is not valid XML or if the `dataType` is not set to `"xml"`.
- These properties reflect the **raw** response, not the parsed `data` argument passed to `.done()` callbacks.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Inspecting Response Properties in a `.fail()` Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>jqXHR properties demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    $.ajax({
      url: "https://jsonplaceholder.typicode.com/nonexistent",
      dataType: "json"
    })
    .fail(function(jqXHR, textStatus, errorThrown) {
      // Step 1: Access essential jqXHR properties
      $("#log").append("Status: " + jqXHR.status + "<br>");
      $("#log").append("Status Text: " + jqXHR.statusText + "<br>");
      $("#log").append("Response Text: " + jqXHR.responseText.substring(0, 50) + "...<br>");
      $("#log").append("Ready State: " + jqXHR.readyState + "<br>");
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Status: 404
Status Text: Not Found
Response Text: <!DOCTYPE html>...
Ready State: 4
```

**Why this output:** The request returns a 404 status. The `.fail()` callback accesses the jqXHR properties: `status` is 404, `statusText` is `"Not Found"`, `responseText` contains the HTML error page returned by the server (truncated for display), and `readyState` is 4 (indicating the request is complete).

### Real-World Cases

- **Conditional error handling:** A `.fail()` callback uses `jqXHR.status` to differentiate between a 404 (not found), 401 (unauthorized), and 500 (server error), displaying appropriate messages for each.
- **Raw response inspection:** When the server returns a non-JSON error page, `jqXHR.responseText` is used to extract error details.
- **XML processing:** A SOAP or RSS client uses `jqXHR.responseXML` to parse the server response into a DOM Document for further processing.

---

## Enhanced Topic: Header Management Methods (`.setRequestHeader()`, `.getResponseHeader()`, `.getAllResponseHeaders()`)

### Definitions

**Core Definition:** The jqXHR object provides methods for managing HTTP headers: `.setRequestHeader()` for setting request headers before the request is sent, and `.getResponseHeader()` and `.getAllResponseHeaders()` for reading response headers after the request completes.

**Technical Definition:** `.setRequestHeader( name, value )` sets a request header by name and value. Note that this departs from the standard by replacing the old value with the new one rather than concatenating. `.getResponseHeader( key )` returns the string value of a specific response header, or `null` if the header is not present. `.getAllResponseHeaders()` returns a single string containing all response headers, with each header on its own line, or `null` if the request has not completed. These methods mirror the native `XMLHttpRequest` API but are exposed on the jqXHR object for convenience.

**Beginner-Friendly Explanation:** Headers are like the envelopes around a letter. `.setRequestHeader()` lets you write on the envelope before you send it (e.g., “Please send me JSON” or “I have an authentication token”). `.getResponseHeader()` lets you read a specific note the server wrote on the envelope it sent back (e.g., the Content-Type or the Set-Cookie header). `.getAllResponseHeaders()` gives you the entire envelope with all the notes. These methods let you control and inspect the metadata of HTTP requests and responses.

### Purposes

- To set custom request headers (e.g., authentication tokens, content types, or API keys) before the Ajax request is sent.
- To retrieve specific response headers (e.g., `Content-Type`, `Set-Cookie`, or custom headers) after the request completes.
- To retrieve all response headers as a single string for logging or debugging.
- To comply with server requirements that mandate custom request headers.
- To inspect rate-limiting headers or cache-control headers for conditional logic.

### Syntax Rules and Structure

**Complete General Syntaxes with Breakdowns:**

**Set Request Header:**
```javascript
jqxhr.setRequestHeader( name, value );
```

| Component | Description |
|-----------|-------------|
| `name` | The header name (e.g., `"Content-Type"`, `"Authorization"`). |
| `value` | The header value. **Replaces** any existing value rather than concatenating. |

**Get Response Header:**
```javascript
jqxhr.getResponseHeader( key );
```

| Component | Description |
|-----------|-------------|
| `key` | The header name to retrieve (case-insensitive). |
| Return value | The header value as a string, or `null` if not present. |

**Get All Response Headers:**
```javascript
jqxhr.getAllResponseHeaders();
```

| Component | Description |
|-----------|-------------|
| Return value | A string containing all response headers, or `null` if the request has not completed. |

**Syntax Rules:**

- `.setRequestHeader()` must be called **before** the request is sent. In the `$.ajax()` settings, this is typically done in the `beforeSend` callback.
- `.setRequestHeader()` **replaces** the header value rather than concatenating it (this differs from the native XHR behavior).
- `.getResponseHeader()` and `.getAllResponseHeaders()` should be called **after** the request completes, inside `.done()`, `.fail()`, or `.always()` callbacks.
- Header names are case-insensitive.

**Constraints and Limitations:**

- `.setRequestHeader()` cannot be called after the request has been sent.
- Some headers are forbidden by the browser and cannot be set (e.g., `Cookie`, `Host`, `Content-Length` in some browsers).
- `.getAllResponseHeaders()` returns `null` if the request has not completed or if the response headers have not been received.
- Cross-origin requests require the server to expose headers via `Access-Control-Expose-Headers` for `.getResponseHeader()` to work.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Setting Request Headers and Reading Response Headers**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Header management demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    // Step 1: Make an Ajax request with a custom header
    $.ajax({
      url: "https://jsonplaceholder.typicode.com/posts/1",
      dataType: "json",
      beforeSend: function(jqXHR) {
        // Step 2: Set a custom request header before sending
        jqXHR.setRequestHeader("X-Custom-Header", "MyValue");
        jqXHR.setRequestHeader("Accept", "application/json");
      }
    })
    .done(function(data, textStatus, jqXHR) {
      // Step 3: Read a specific response header
      var contentType = jqXHR.getResponseHeader("Content-Type");
      $("#log").append("Content-Type: " + contentType + "<br>");

      // Step 4: Read a custom response header (if exposed by server)
      var customHeader = jqXHR.getResponseHeader("X-Custom-Response");
      $("#log").append("X-Custom-Response: " + customHeader + "<br>");

      // Step 5: Get all response headers
      var allHeaders = jqXHR.getAllResponseHeaders();
      $("#log").append("All headers:<br>" + allHeaders.replace(/\n/g, "<br>"));
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Content-Type: application/json; charset=utf-8
X-Custom-Response: null
All headers:
cache-control: max-age=43200
content-type: application/json; charset=utf-8
...
```

**Why this output:** The `beforeSend` callback sets a custom request header (`X-Custom-Header`) and an `Accept` header. After the request succeeds, the `.done()` callback reads the `Content-Type` response header, which is `"application/json; charset=utf-8"`. The custom response header `X-Custom-Response` is not present, so `getResponseHeader()` returns `null`. Finally, `getAllResponseHeaders()` returns all headers as a newline-delimited string.

### Real-World Cases

- **Authentication:** An API client sets an `Authorization: Bearer <token>` header using `.setRequestHeader()` in the `beforeSend` callback.
- **Content negotiation:** A request sets `Accept: application/xml` to request an XML response instead of JSON.
- **Rate-limit detection:** A `.done()` callback reads the `X-RateLimit-Remaining` response header to determine how many API calls are left.
- **Debugging:** A developer logs all response headers using `.getAllResponseHeaders()` to diagnose a CORS or caching issue.

---

## References

- jQuery API Documentation — jQuery.ajax() — https://api.jquery.com/jquery.ajax/
- jQuery API Documentation — deferred.done() — https://api.jquery.com/deferred.done/
- jQuery API Documentation — deferred.fail() — https://api.jquery.com/deferred.fail/
- jQuery API Documentation — deferred.always() — https://api.jquery.com/deferred.always/
- jQuery API Documentation — deferred.catch() — https://api.jquery.com/deferred.catch/
- jQuery API Documentation — jQuery.when() — https://api.jquery.com/jQuery.when/
- jQuery API Documentation — jQuery.post() — https://api.jquery.com/jQuery.post/
- jQuery API Documentation — jQuery.get() — https://api.jquery.com/jQuery.get/
- jQuery 3.0 Upgrade Guide — Deferreds Updated for Promises/A+ Compatibility — https://jquery.com/upgrade-guide/3.0/#deferreds
- jQuery Bug Tracker — Ticket #14075: jQuery v2+ ajax api uses deprecated success/error/complete — https://bugs.jquery.com/ticket/14075/
- jQuery Bug Tracker — Ticket #12168: Inconsistency: jqXHR.{success|error|complete} deprecated — https://bugs.jquery.com/ticket/12168/
- MDN Web Docs — XMLHttpRequest — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
- MDN Web Docs — XMLHttpRequest.setRequestHeader() — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader
- MDN Web Docs — XMLHttpRequest.getResponseHeader() — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/getResponseHeader
- MDN Web Docs — XMLHttpRequest.getAllResponseHeaders() — https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/getAllResponseHeaders
- DefinitelyTyped — JQueryXHR Interface — http://definitelytyped.org/docs/angularjs--angular-cookies/interfaces/jqueryxhr.html
- PrimeFaces — jqXHR Interface Documentation — https://primefaces.github.io/primefaces/jsdocs/interfaces/src_PrimeFaces.JQuery.jqXHR-1.html