# jQuery AJAX Events — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery AJAX Events are a set of custom events triggered by jQuery at specific stages of an AJAX request's lifecycle, allowing developers to subscribe to and react to request initiation, completion, success, failure, and the overall start/stop of all concurrent requests.

**Technical Definition:**
jQuery AJAX events are divided into two categories: **local events** (callbacks configured within the `$.ajax()` settings object: `beforeSend`, `success`, `error`, `complete`) and **global events** (triggered on `document` and observable via `.on()`: `ajaxStart`, `ajaxSend`, `ajaxSuccess`, `ajaxError`, `ajaxComplete`, `ajaxStop`). Global AJAX events can be disabled per request by setting `global: false` in the `$.ajax()` settings. As of jQuery 1.9, all global AJAX event handlers must be attached to `document`. The deprecated shorthand methods (`.ajaxStart()`, `.ajaxComplete()`, etc.) were removed in jQuery 3.5 and replaced by `.on("ajaxStart", handler)`.

**Beginner-Friendly Explanation:**
When you make an AJAX request, jQuery fires a series of events — like a race with checkpoints. You can listen at each checkpoint to run code: show a spinner when a request starts, hide it when all requests finish, log errors when a request fails, or process data when a request succeeds. Global events are like a loudspeaker for the whole page; local events are like a private conversation within a specific request.

### Key Characteristics

- **Two-Tier System:** Local events are configured per request; global events are observed page-wide on `document`.
- **Strict Ordering:** Events fire in a defined sequence: `ajaxStart` → `beforeSend` → `ajaxSend` → `success`/`error` → `ajaxSuccess`/`ajaxError` → `complete` → `ajaxComplete` → `ajaxStop`.
- **Counter-Based Start/Stop:** `ajaxStart` fires only when the first request begins (no other requests active); `ajaxStop` fires only when the last request completes.
- **Global Opt-Out:** A request can disable global events by setting `global: false` in its settings.
- **No Shorthand Methods:** The old `.ajaxStart()`, `.ajaxStop()`, etc., methods are deprecated and removed. Use `.on()` instead.
- **Document Attachment Required:** Since jQuery 1.9, all global AJAX event handlers must be attached to `document`.

### Prerequisites

- Basic knowledge of jQuery event binding with `.on()` and `.off()`
- Familiarity with the `$.ajax()` method and its settings object
- Understanding of event bubbling and event delegation concepts
- Basic experience with browser developer tools (Network panel, Console)

### Related Programming Areas

- **jQuery AJAX Methods:** `$.ajax()`, `$.get()`, `$.post()`, etc., trigger AJAX events.
- **jQuery Deferred Objects:** The `.done()`, `.fail()`, `.always()` methods are the modern alternative to legacy `success`/`error`/`complete` callbacks.
- **Global Event Delegation:** AJAX global events use `document`-level delegation for page-wide monitoring.
- **Single-Page Applications (SPAs):** AJAX events are critical for managing loading states and cleanup in SPAs.
- **Performance Monitoring:** `ajaxStart`/`ajaxStop` enable centralized loading indicators and request tracking.

### Core Concepts / Features

1. Global AJAX Events
2. `ajaxStart`
3. `ajaxStop`
4. `ajaxComplete`
5. `ajaxError`
6. `ajaxSuccess`
7. `ajaxSend`
8. Local AJAX Events
9. Event Registration
10. Event Unbinding
11. Event Best Practices

---

## 1. Global AJAX Events

### Definitions

**Core Definition:**
Global AJAX events are events triggered on the `document` object by jQuery during the lifecycle of any AJAX request, regardless of which code initiated the request.

**Technical Definition:**
Global AJAX events are triggered on `document` and are observable via `$(document).on("ajaxEventName", handler)`. They fire for all AJAX requests unless a specific request sets `global: false`. The global events are: `ajaxStart`, `ajaxSend`, `ajaxSuccess`, `ajaxError`, `ajaxComplete`, and `ajaxStop`. Since jQuery 1.9, all global AJAX event handlers must be attached to `document`; attaching them to other elements will not work. Each global handler receives the event object and additional arguments specific to the event.

**Beginner-Friendly Explanation:**
Global AJAX events are like announcements made over a loudspeaker for the entire page. No matter which part of your code makes an AJAX request, the loudspeaker announces it. You can listen to these announcements to show loading spinners, log errors, or track usage — all from one place.

### Purposes

- To provide a centralized mechanism for monitoring all AJAX activity on a page.
- To implement page-wide loading indicators without modifying every AJAX call.
- To enable global error handling and logging across all requests.
- To coordinate the UI state (e.g., disable buttons) during any network activity.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxEventName", function(event, jqXHR, settings, ...) {
    // Handle the event
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The element to which global AJAX handlers must be attached (jQuery 1.9+). |
| `.on("ajaxEventName", handler)` | The event binding method. |
| `event` | The jQuery event object. |
| `jqXHR` | The jQuery XHR object (for most events). |
| `settings` | The settings object used in the request. |

**Syntax Rules:**

1. Global AJAX handlers must be attached to `document` since jQuery 1.9.
2. Global events fire for every AJAX request unless `global: false` is set.
3. The order of firing is fixed and cannot be changed.
4. `ajaxStart` and `ajaxStop` relate to the group of all requests, not individual requests.
5. The deprecated shorthand methods (`.ajaxStart()`, etc.) are removed; use `.on()`.

**Constraints and Limitations:**

- Global events fire for all requests, including those from third-party plugins.
- Attaching to `document` means the handlers persist for the page's lifetime unless explicitly removed.
- `global: false` disables all global events for that request, including `ajaxStart` and `ajaxStop`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Global Loading Indicator with `ajaxStart` and `ajaxStop`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Global AJAX Events: Loading Indicator</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #loading { display: none; position: fixed; top: 10px; right: 10px;
                   background: #333; color: white; padding: 10px; }
    </style>
</head>
<body>
    <div id="loading">Loading...</div>
    <button id="loadBtn">Load Data</button>
    <div id="result"></div>

    <script>
        // Step 1: Show loading indicator when the first request starts
        $(document).on("ajaxStart", function() {
            $("#loading").show();
            console.log("ajaxStart: First request began");
        });

        // Step 2: Hide loading indicator when all requests finish
        $(document).on("ajaxStop", function() {
            $("#loading").hide();
            console.log("ajaxStop: All requests finished");
        });

        // Step 3: Make a request
        $("#loadBtn").on("click", function() {
            $.get("https://jsonplaceholder.typicode.com/posts/1", function(data) {
                $("#result").text(data.title);
            });
        });
    </script>
</body>
</html>
```

**Expected Output:**
When the button is clicked, the "Loading..." indicator appears, then disappears when the request completes. Console logs: `ajaxStart: First request began` → `ajaxStop: All requests finished`.

**Why This Code Produces That Result:**
`ajaxStart` fires when the first (and only) request begins, showing the indicator. `ajaxStop` fires when the last request finishes, hiding it. If multiple requests were made concurrently, `ajaxStart` would fire only once (when the first begins) and `ajaxStop` only once (when the last ends). This is the classic pattern for page-wide loading indicators.

### Real-World Cases

- **Page-Wide Loading Spinners:** Showing a top-bar loading indicator for any AJAX activity.
- **Global Error Logging:** Logging all failed requests to a monitoring service.
- **Request Auditing:** Counting or timing all AJAX requests on a page.
- **Disabling UI During Requests:** Preventing double-submissions by disabling buttons globally.

---

## 2. `ajaxStart`

### Definitions

**Core Definition:**
`ajaxStart` is a global AJAX event that fires when the first AJAX request begins, and no other AJAX requests are currently running.

**Technical Definition:**
`.on("ajaxStart", handler)` registers a handler to be called when the first AJAX request begins. Whenever an AJAX request is about to be sent, jQuery checks whether there are any other outstanding AJAX requests. If none are in progress, jQuery triggers the `ajaxStart` event. The handler receives the event object. As of jQuery 1.9, the handler must be attached to `document`. If `$.ajax()` or `$.ajaxSetup()` is called with `global: false`, the event will not fire.

**Beginner-Friendly Explanation:**
`ajaxStart` is like a starting gun for AJAX requests. It fires only once, when the very first request kicks off, and stays silent while other requests are already running. It's perfect for showing a "loading" message that covers all requests.

### Purposes

- To initiate a loading indicator when the first AJAX request begins.
- To disable UI elements during network activity.
- To start performance timers for AJAX operations.
- To log the beginning of a batch of requests.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxStart", function(event) {
    // Handler logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The required attachment point (jQuery 1.9+). |
| `"ajaxStart"` | The event type string. |
| `handler` | Function to execute when the event fires. |
| `event` | The jQuery event object. |

**Syntax Rules:**

1. Must be attached to `document`.
2. Fires only when the first request begins (no other requests active).
3. Does not receive `jqXHR` or `settings` arguments.
4. Will not fire if the request has `global: false`.
5. The deprecated `.ajaxStart()` method is removed; use `.on()`.

**Constraints and Limitations:**

- Fires only once for a batch of concurrent requests.
- Cannot differentiate between different requests.
- Does not fire for requests with `global: false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic `ajaxStart` with Loading Message**

```html
<!DOCTYPE html>
<html>
<head>
    <title>ajaxStart Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #loading { display: none; background: yellow; padding: 5px; }
    </style>
</head>
<body>
    <div id="loading">Loading...</div>
    <div class="trigger">Trigger Request</div>
    <div class="result"></div>

    <script>
        // Step 1: Attach ajaxStart handler to document
        $(document).on("ajaxStart", function() {
            $("#loading").show();
            console.log("ajaxStart fired");
        });

        // Step 2: Bind a click to make a request
        $(".trigger").on("click", function() {
            $(".result").load("https://jsonplaceholder.typicode.com/posts/1");
        });
    </script>
</body>
</html>
```

**Expected Output:**
Clicking "Trigger Request" shows the "Loading..." message and loads the post content. Console logs `ajaxStart fired`.

**Why This Code Produces That Result:**
When the `.load()` request begins, jQuery checks if any other requests are active. Since none are, `ajaxStart` fires, showing the loading message. The message remains visible until the request completes (and `ajaxStop` would hide it, if bound). This demonstrates the canonical use case for `ajaxStart`.

### Real-World Cases

- **SPA Navigation:** Showing a global loading bar when a route change triggers data fetching.
- **Form Submissions:** Disabling all submit buttons while the first request is active.
- **Analytics Tracking:** Recording the start time of a user-initiated request batch.

---

## 3. `ajaxStop`

### Definitions

**Core Definition:**
`ajaxStop` is a global AJAX event that fires when all AJAX requests have completed and no outstanding requests remain.

**Technical Definition:**
`.on("ajaxStop", handler)` registers a handler to be called when all AJAX requests have completed. Whenever an AJAX request completes, jQuery checks whether there are any other outstanding AJAX requests. If none remain, jQuery triggers the `ajaxStop` event. The `ajaxStop` event is also triggered if the last outstanding AJAX request is cancelled by returning `false` within the `beforeSend` callback function. As of jQuery 1.9, the handler must be attached to `document`.

**Beginner-Friendly Explanation:**
`ajaxStop` is like the finish line for a batch of AJAX requests. It fires only when the very last request finishes, making it perfect for hiding loading indicators and re-enabling UI elements.

### Purposes

- To hide a global loading indicator when all requests finish.
- To re-enable UI elements that were disabled during network activity.
- To log the completion of a batch of requests.
- To trigger cleanup or follow-up actions after all network activity ceases.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxStop", function(event) {
    // Handler logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The required attachment point. |
| `"ajaxStop"` | The event type string. |
| `handler` | Function to execute when the event fires. |

**Syntax Rules:**

1. Must be attached to `document`.
2. Fires only when the last outstanding request completes.
3. Also fires if the last request is cancelled via `beforeSend` returning `false`.
4. Does not receive `jqXHR` or `settings` arguments.
5. Will not fire if all requests have `global: false`.

**Constraints and Limitations:**

- Fires only once for a batch of concurrent requests.
- Cannot differentiate between different requests.
- Does not fire for requests with `global: false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Hiding Loading Indicator with `ajaxStop`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>ajaxStop Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #loading { display: none; background: green; color: white; padding: 5px; }
    </style>
</head>
<body>
    <div id="loading">Loading...</div>
    <button id="loadBtn">Load Data</button>

    <script>
        // Step 1: Show loading on ajaxStart
        $(document).on("ajaxStart", function() {
            $("#loading").show();
        });

        // Step 2: Hide loading on ajaxStop
        $(document).on("ajaxStop", function() {
            $("#loading").hide();
            console.log("ajaxStop: All requests finished");
        });

        // Step 3: Make a request
        $("#loadBtn").on("click", function() {
            $.get("https://jsonplaceholder.typicode.com/posts/1");
        });
    </script>
</body>
</html>
```

**Expected Output:**
The "Loading..." indicator appears when the request starts and disappears when it finishes. Console logs `ajaxStop: All requests finished`.

**Why This Code Produces That Result:**
`ajaxStart` shows the indicator when the first request begins. `ajaxStop` hides it when the last request completes. Together they create a seamless loading indicator that works for any number of concurrent requests. The handler is attached to `document` as required by jQuery 1.9+.

### Real-World Cases

- **SPA Loading States:** Hiding a global spinner after all initial data loads complete.
- **Bulk Operations:** Re-enabling a "Save All" button after all save requests finish.
- **Request Queues:** Triggering a refresh or notification after all background requests complete.

---

## 4. `ajaxComplete`

### Definitions

**Core Definition:**
`ajaxComplete` is a global AJAX event that fires whenever any AJAX request completes, regardless of whether it succeeded or failed.

**Technical Definition:**
`.on("ajaxComplete", handler)` registers a handler to be called when AJAX requests complete. Whenever an AJAX request completes, jQuery triggers the `ajaxComplete` event. All registered `ajaxComplete` handlers are executed at this time. Each handler receives the event object, the `jqXHR` object, and the settings object used in the request. You can differentiate between requests by inspecting `settings.url` or other settings properties. As of jQuery 1.9, the handler must be attached to `document`.

**Beginner-Friendly Explanation:**
`ajaxComplete` fires after every single AJAX request finishes — whether it succeeded or failed. It's like a cleanup crew that shows up after every job, regardless of the outcome.

### Purposes

- To perform cleanup after every AJAX request.
- To log or audit all AJAX requests.
- To update a request counter or progress indicator.
- To trigger follow-up actions regardless of success or failure.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxComplete", function(event, jqXHR, settings) {
    // Handler logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The required attachment point. |
| `"ajaxComplete"` | The event type string. |
| `event` | The jQuery event object. |
| `jqXHR` | The jQuery XHR object (prior to jQuery 1.5, the XHR object). |
| `settings` | The settings object used in the request. |

**Syntax Rules:**

1. Must be attached to `document`.
2. Fires for every request, success or failure.
3. Receives `jqXHR` and `settings` arguments.
4. Use `settings.url` to differentiate between requests.
5. Fires after `ajaxSuccess`/`ajaxError` and before `ajaxStop`.

**Constraints and Limitations:**

- Fires for all requests, which can be noisy in complex applications.
- Cannot prevent or modify the request outcome.
- Does not fire for requests with `global: false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Request Auditing with `ajaxComplete`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>ajaxComplete Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="log"></div>

    <script>
        // Step 1: Log every completed request
        $(document).on("ajaxComplete", function(event, jqXHR, settings) {
            var status = jqXHR.status;
            var url = settings.url;
            $("#log").append(
                "Request to " + url + " completed with status " + status + "<br>"
            );
            console.log("ajaxComplete:", url, status);
        });

        // Step 2: Make two requests — one success, one failure
        $.get("https://jsonplaceholder.typicode.com/posts/1");
        $.get("https://jsonplaceholder.typicode.com/nonexistent");
    </script>
</body>
</html>
```

**Expected Output:**
The log div shows two entries: one for the successful request (status 200) and one for the failed request (status 404).

**Why This Code Produces That Result:**
`ajaxComplete` fires for both requests regardless of outcome. The handler receives the `jqXHR` object (containing `status`) and `settings` (containing `url`). This allows centralized logging or auditing of all AJAX activity. The event fires after the success/error handlers but before `ajaxStop`.

### Real-World Cases

- **Request Logging:** Logging all AJAX URLs and statuses for debugging.
- **Progress Tracking:** Incrementing a counter for each completed request.
- **Cleanup:** Removing temporary event handlers after each request.

---

## 5. `ajaxError`

### Definitions

**Core Definition:**
`ajaxError` is a global AJAX event that fires whenever an AJAX request completes with an error.

**Technical Definition:**
`.on("ajaxError", handler)` registers a handler to be called when AJAX requests complete with an error. Whenever an AJAX request fails, jQuery triggers the `ajaxError` event. The handler receives the event object, the `jqXHR` object, the settings object, and (when an HTTP error occurs) the `thrownError` argument containing the textual portion of the HTTP status, such as "Not Found" or "Internal Server Error". **This handler is not called for cross-domain script and cross-domain JSONP requests**. As of jQuery 1.9, the handler must be attached to `document`.

**Beginner-Friendly Explanation:**
`ajaxError` fires when an AJAX request fails. It's your central place to handle all failed requests — show error messages, log problems, or notify the user. Just remember it doesn't fire for cross-domain script or JSONP requests.

### Purposes

- To provide centralized error handling for all AJAX requests.
- To display user-friendly error notifications.
- To log failed requests for debugging and monitoring.
- To implement retry logic or fallback behavior.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxError", function(event, jqXHR, settings, thrownError) {
    // Handler logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The required attachment point. |
| `"ajaxError"` | The event type string. |
| `event` | The jQuery event object. |
| `jqXHR` | The jQuery XHR object. |
| `settings` | The settings object used in the request. |
| `thrownError` | The textual portion of the HTTP status (e.g., "Not Found"). |

**Syntax Rules:**

1. Must be attached to `document`.
2. Fires only when a request fails.
3. Receives four arguments: `event`, `jqXHR`, `settings`, `thrownError`.
4. Does **not** fire for cross-domain script and JSONP requests.
5. Use `settings.url` to differentiate between requests.

**Constraints and Limitations:**

- Not called for cross-domain script and JSONP requests.
- Cannot prevent the error or modify the response.
- Does not fire for requests with `global: false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Centralized Error Handling with `ajaxError`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>ajaxError Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="errorLog"></div>

    <script>
        // Step 1: Centralized error handler
        $(document).on("ajaxError", function(event, jqXHR, settings, thrownError) {
            $("#errorLog").append(
                "Error loading " + settings.url + ": " +
                jqXHR.status + " " + thrownError + "<br>"
            );
            console.log("AJAX Error:", settings.url, jqXHR.status, thrownError);
        });

        // Step 2: Make a request to a nonexistent resource
        $.get("https://jsonplaceholder.typicode.com/nonexistent");
    </script>
</body>
</html>
```

**Expected Output:**
The error log div shows: `Error loading https://jsonplaceholder.typicode.com/nonexistent: 404 Not Found`.

**Why This Code Produces That Result:**
The request fails with a 404 status. `ajaxError` fires, passing the `jqXHR` (with `status: 404`), `settings` (with `url`), and `thrownError` ("Not Found"). The handler logs the details. This demonstrates how to centralize error handling for all AJAX requests on a page.

### Real-World Cases

- **Global Error Notifications:** Showing a toast or banner when any request fails.
- **Error Logging:** Sending error details to a logging service or console.
- **Retry Logic:** Automatically retrying failed requests with exponential backoff.

---

## 6. `ajaxSuccess`

### Definitions

**Core Definition:**
`ajaxSuccess` is a global AJAX event that fires whenever an AJAX request completes successfully.

**Technical Definition:**
`.on("ajaxSuccess", handler)` registers a handler to be called when AJAX requests complete successfully. Whenever an AJAX request succeeds, jQuery triggers the `ajaxSuccess` event. The handler receives the event object, the `jqXHR` object, and the settings object. The returned contents can be accessed via `xhr.responseText` or `xhr.responseXML`. As of jQuery 1.9, the handler must be attached to `document`.

**Beginner-Friendly Explanation:**
`ajaxSuccess` fires when an AJAX request succeeds. It's useful for logging successful requests, updating a success counter, or triggering follow-up actions after data is loaded.

### Purposes

- To log or track successful AJAX requests.
- To update UI elements when data loads successfully.
- To trigger follow-up actions after successful data retrieval.
- To implement analytics tracking for successful API calls.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxSuccess", function(event, jqXHR, settings) {
    // Handler logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The required attachment point. |
| `"ajaxSuccess"` | The event type string. |
| `event` | The jQuery event object. |
| `jqXHR` | The jQuery XHR object. |
| `settings` | The settings object used in the request. |

**Syntax Rules:**

1. Must be attached to `document`.
2. Fires only when a request succeeds.
3. Receives `jqXHR` and `settings` arguments.
4. Access response content via `jqXHR.responseText` or `jqXHR.responseXML`.
5. Fires after `success` (local) and before `complete` (local).

**Constraints and Limitations:**

- Fires for all successful requests, which can be noisy.
- Cannot modify the response or prevent success.
- Does not fire for requests with `global: false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Success Logging with `ajaxSuccess`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>ajaxSuccess Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="successLog"></div>

    <script>
        // Step 1: Log all successful requests
        $(document).on("ajaxSuccess", function(event, jqXHR, settings) {
            $("#successLog").append(
                "Success: " + settings.url + " (status " + jqXHR.status + ")<br>"
            );
            console.log("ajaxSuccess:", settings.url, jqXHR.status);
        });

        // Step 2: Make a successful request
        $.get("https://jsonplaceholder.typicode.com/posts/1");
    </script>
</body>
</html>
```

**Expected Output:**
The success log div shows: `Success: https://jsonplaceholder.typicode.com/posts/1 (status 200)`.

**Why This Code Produces That Result:**
The request succeeds with a 200 status. `ajaxSuccess` fires, passing the `jqXHR` and `settings`. The handler logs the success. This demonstrates how to track successful requests globally.

### Real-World Cases

- **Analytics:** Tracking successful API calls for usage metrics.
- **UI Updates:** Showing a success message after data loads.
- **Chain Reactions:** Triggering dependent requests after a successful load.

---

## 7. `ajaxSend`

### Definitions

**Core Definition:**
`ajaxSend` is a global AJAX event that fires before an AJAX request is sent.

**Technical Definition:**
`.on("ajaxSend", handler)` attaches a function to be executed before an AJAX request is sent. Whenever an AJAX request is about to be sent, jQuery triggers the `ajaxSend` event. The handler receives the event object, the `jqXHR` object, and the settings object. You can differentiate between requests by inspecting `settings.url`. As of jQuery 1.9, the handler must be attached to `document`.

**Beginner-Friendly Explanation:**
`ajaxSend` fires just before an AJAX request goes out. It's like a pre-flight check — you can log the request, add headers, or show a specific loading message.

### Purposes

- To log or audit outgoing AJAX requests.
- To add common headers to all requests.
- To show a request-specific loading indicator.
- To start performance timers for individual requests.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$(document).on("ajaxSend", function(event, jqXHR, settings) {
    // Handler logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The required attachment point. |
| `"ajaxSend"` | The event type string. |
| `event` | The jQuery event object. |
| `jqXHR` | The jQuery XHR object. |
| `settings` | The settings object used in the request. |

**Syntax Rules:**

1. Must be attached to `document`.
2. Fires before the request is sent.
3. Receives `jqXHR` and `settings` arguments.
4. Can modify the request via the `jqXHR` object (e.g., set headers).
5. Fires after `beforeSend` (local) and before `success`/`error`.

**Constraints and Limitations:**

- Fires for all requests, which can be noisy.
- Cannot cancel the request (use `beforeSend` returning `false` for that).
- Does not fire for requests with `global: false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Logging Outgoing Requests with `ajaxSend`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>ajaxSend Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="sendLog"></div>

    <script>
        // Step 1: Log all outgoing requests
        $(document).on("ajaxSend", function(event, jqXHR, settings) {
            $("#sendLog").append(
                "Sending: " + settings.url + " (" + settings.type + ")<br>"
            );
            console.log("ajaxSend:", settings.url, settings.type);
        });

        // Step 2: Make a request
        $.get("https://jsonplaceholder.typicode.com/posts/1");
    </script>
</body>
</html>
```

**Expected Output:**
The send log div shows: `Sending: https://jsonplaceholder.typicode.com/posts/1 (GET)`.

**Why This Code Produces That Result:**
`ajaxSend` fires before the request is sent. The handler receives the `settings` object containing `url` and `type` (HTTP method). This allows logging or auditing of all outgoing requests. The event fires after `beforeSend` but before the actual network request.

### Real-World Cases

- **Request Auditing:** Logging all outgoing requests for debugging.
- **Header Injection:** Adding authentication headers to all requests globally.
- **Performance Timing:** Recording the start time of each request.

---

## 8. Local AJAX Events

### Definitions

**Core Definition:**
Local AJAX events are callbacks configured within the `$.ajax()` settings object that fire for a specific request only, not globally.

**Technical Definition:**
Local AJAX events are passed as properties of the settings object to `$.ajax()`. They include `beforeSend`, `success`, `error`, and `complete`. These callbacks fire only for the specific request they are configured on. The `success` and `error` callbacks are mutually exclusive (never both fire for the same request). The `complete` callback always fires, regardless of outcome, even for synchronous requests.

**Beginner-Friendly Explanation:**
Local AJAX events are private to a single request. You configure them inside the `$.ajax()` call, and they only run for that specific request. They're like a personal assistant for one task.

### Purposes

- To handle the outcome of a specific request with custom logic.
- To perform request-specific cleanup or UI updates.
- To pass request-specific data to success/error handlers.
- To implement per-request loading indicators.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$.ajax({
    url: "api/endpoint",
    beforeSend: function(jqXHR, settings) { /* ... */ },
    success: function(data, textStatus, jqXHR) { /* ... */ },
    error: function(jqXHR, textStatus, errorThrown) { /* ... */ },
    complete: function(jqXHR, textStatus) { /* ... */ }
});
```

**Component Breakdown:**

| Callback | When | Arguments |
|----------|------|-----------|
| `beforeSend` | Before request is sent | `jqXHR`, `settings` |
| `success` | On success | `data`, `textStatus`, `jqXHR` |
| `error` | On failure | `jqXHR`, `textStatus`, `errorThrown` |
| `complete` | Always, after success/error | `jqXHR`, `textStatus` |

**Syntax Rules:**

1. Configured in the `$.ajax()` settings object.
2. `success` and `error` are mutually exclusive.
3. `complete` always fires, even for synchronous requests.
4. Local events fire before their global counterparts.
5. The legacy callbacks are less flexible than Promise methods (`.done()`, `.fail()`, `.always()`).

**Constraints and Limitations:**

- Only one `success`/`error`/`complete` callback can be set per request in the options object.
- Cannot be chained like Promise methods.
- Error thrown inside callbacks is not automatically caught.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Local Events for a Specific Request**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Local AJAX Events</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>

    <script>
        // Step 1: Configure local events for a specific request
        $.ajax({
            url: "https://jsonplaceholder.typicode.com/posts/1",
            dataType: "json",
            beforeSend: function() {
                $("#output").text("Loading...");
                console.log("beforeSend: request about to start");
            },
            success: function(data) {
                $("#output").html("<h3>" + data.title + "</h3>");
                console.log("success: data received");
            },
            error: function(jqXHR, textStatus, errorThrown) {
                $("#output").text("Error: " + textStatus);
                console.log("error:", textStatus, errorThrown);
            },
            complete: function(jqXHR, textStatus) {
                console.log("complete: request finished with status", textStatus);
            }
        });
    </script>
</body>
</html>
```

**Expected Output:**
The output div shows "Loading...", then displays the post title. Console logs: `beforeSend`, `success`, `complete`.

**Why This Code Produces That Result:**
The local events are configured in the `$.ajax()` settings object and fire only for this request. `beforeSend` fires first, updating the UI. `success` fires with the parsed JSON data. `complete` fires last regardless of outcome. The `error` callback would fire instead of `success` if the request failed. This demonstrates the full lifecycle of local AJAX events.

### Real-World Cases

- **Form Submission:** Handling success and error for a specific form submission.
- **Data Loading:** Updating a specific container with loaded data.
- **Custom Error Messages:** Showing request-specific error messages.

---

## 9. Event Registration

### Definitions

**Core Definition:**
Event registration is the process of attaching handlers to AJAX events, either globally via `$(document).on("ajaxEventName", handler)` or locally via the `$.ajax()` settings object.

**Technical Definition:**
Global AJAX events are registered using `.on()` attached to `document`. Local events are registered by including callback functions in the `$.ajax()` settings object. Since jQuery 1.9, all global AJAX event handlers must be attached to `document`; attaching them to other elements will not work. The deprecated shorthand methods (`.ajaxStart()`, `.ajaxComplete()`, etc.) were removed in jQuery 3.5 and should be replaced by `.on()`.

**Beginner-Friendly Explanation:**
Registering an event handler means telling jQuery "when this event happens, run this function." For global events, you attach the handler to the document. For local events, you put the function directly in the `$.ajax()` call.

### Purposes

- To attach handlers that respond to AJAX lifecycle events.
- To centralize monitoring and control of AJAX activity.
- To implement loading indicators and error handling.
- To log and audit AJAX requests.

### Syntax Rules and Structure

**Global Event Registration:**

```javascript
$(document).on("ajaxStart", function() { /* ... */ });
$(document).on("ajaxStop", function() { /* ... */ });
$(document).on("ajaxComplete", function(event, jqXHR, settings) { /* ... */ });
$(document).on("ajaxError", function(event, jqXHR, settings, thrownError) { /* ... */ });
$(document).on("ajaxSuccess", function(event, jqXHR, settings) { /* ... */ });
$(document).on("ajaxSend", function(event, jqXHR, settings) { /* ... */ });
```

**Local Event Registration:**

```javascript
$.ajax({
    url: "api/endpoint",
    beforeSend: function() { /* ... */ },
    success: function(data) { /* ... */ },
    error: function(jqXHR, textStatus, errorThrown) { /* ... */ },
    complete: function() { /* ... */ }
});
```

**Component Breakdown:**

| Registration Type | Method | Attachment Point |
|-------------------|--------|------------------|
| Global | `.on("ajaxEvent", handler)` | `document` |
| Local | `$.ajax({ eventName: handler })` | Settings object |

**Syntax Rules:**

1. Global handlers must be attached to `document`.
2. Global handlers fire for all requests unless `global: false`.
3. Local handlers fire only for their specific request.
4. Use `.on()` for global registration; the shorthand methods are removed.
5. Multiple global handlers can be attached for the same event.

**Constraints and Limitations:**

- Global handlers attached to non-`document` elements will not work (jQuery 1.9+).
- Local handlers cannot be registered after the request is sent.
- Global handlers add overhead to every AJAX request.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Registering Global and Local Events Together**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Event Registration</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="log"></div>

    <script>
        // Step 1: Register a global ajaxStart handler
        $(document).on("ajaxStart", function() {
            $("#log").append("Global: Request started<br>");
        });

        // Step 2: Register a global ajaxComplete handler
        $(document).on("ajaxComplete", function(event, jqXHR, settings) {
            $("#log").append("Global: Request completed (" + settings.url + ")<br>");
        });

        // Step 3: Make a request with local events
        $.ajax({
            url: "https://jsonplaceholder.typicode.com/posts/1",
            dataType: "json",
            beforeSend: function() {
                $("#log").append("Local: beforeSend<br>");
            },
            success: function(data) {
                $("#log").append("Local: success (" + data.title + ")<br>");
            },
            complete: function() {
                $("#log").append("Local: complete<br>");
            }
        });
    </script>
</body>
</html>
```

**Expected Output:**
The log div shows, in order:
```
Global: Request started
Local: beforeSend
Local: success (sunt aut facere...)
Local: complete
Global: Request completed (https://jsonplaceholder.typicode.com/posts/1)
```

**Why This Code Produces That Result:**
Global and local handlers coexist. The global `ajaxStart` fires first. Then the local `beforeSend` fires. After the request succeeds, local `success` and `complete` fire. Finally, global `ajaxComplete` fires. This demonstrates the interleaving of global and local events in the AJAX lifecycle.

### Real-World Cases

- **Hybrid Monitoring:** Using global events for page-wide indicators and local events for request-specific logic.
- **Plugin Development:** Plugins register local events for their requests and global events for cross-plugin coordination.
- **Debugging:** Registering global handlers temporarily to log all AJAX activity.

---

## 10. Event Unbinding

### Definitions

**Core Definition:**
Event unbinding is the process of removing previously registered AJAX event handlers using `.off()`, which is critical for preventing memory leaks and stale event handling in dynamic applications.

**Technical Definition:**
Global AJAX event handlers are removed with `.off()` attached to `document`, using the event type and optionally a namespace for selective removal. Local AJAX events are removed by simply not including them in future requests — they have no persistent registration. Namespacing global handlers (e.g., `.on("ajaxStart.myModule")`) allows selective unbinding without affecting other modules' handlers. In SPAs, failing to unbind global AJAX handlers during route changes or component destruction can cause memory leaks and duplicate handler execution.

**Beginner-Friendly Explanation:**
When you remove a page or component from a single-page application, you need to remove its AJAX event handlers too. Otherwise, they stay in memory and may run again when new content loads, causing bugs and slowdowns.

### Purposes

- To prevent memory leaks from stale AJAX event handlers.
- To avoid duplicate handler execution in dynamic applications.
- To clean up module-specific AJAX handlers during teardown.
- To reset global AJAX event state before re-rendering content.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
// Remove all handlers for a global AJAX event
$(document).off("ajaxStart");

// Remove handlers by namespace
$(document).off(".myModule");

// Remove a specific handler
$(document).off("ajaxComplete", myHandler);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$(document)` | The attachment point for global handlers. |
| `.off("eventType")` | Removes all handlers for the event type. |
| `.off(".namespace")` | Removes all handlers in the namespace. |
| `myHandler` | A reference to a specific handler function. |

**Syntax Rules:**

1. Use `.off()` instead of deprecated `.unbind()`.
2. Namespacing is recommended for selective unbinding.
3. Global handlers must be removed from `document`.
4. Local events do not need explicit removal.
5. Always unbind before removing or replacing DOM elements that have handlers.

**Constraints and Limitations:**

- Unbinding removes handlers but does not free memory immediately; garbage collection must run.
- Anonymous functions cannot be individually removed without a namespace.
- Unbinding global handlers affects all requests, so use namespaces for selective removal.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Namespaced Unbinding in an SPA**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Event Unbinding</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="log"></div>
    <button id="destroy">Destroy Module</button>

    <script>
        // Step 1: Register a namespaced global handler
        $(document).on("ajaxStart.myModule", function() {
            $("#log").append("Module request started<br>");
        });

        $(document).on("ajaxComplete.myModule", function() {
            $("#log").append("Module request completed<br>");
        });

        // Step 2: Make a request — handlers fire
        $.get("https://jsonplaceholder.typicode.com/posts/1");

        // Step 3: Destroy the module and unbind its handlers
        $("#destroy").on("click", function() {
            $(document).off(".myModule");
            $("#log").append("Module handlers removed<br>");
            console.log("Unbound .myModule handlers");
        });

        // Step 4: After destruction, make another request — no module handlers fire
        // $.get("https://jsonplaceholder.typicode.com/posts/2");
    </script>
</body>
</html>
```

**Expected Output:**
The log div shows "Module request started" and "Module request completed". After clicking "Destroy Module", it shows "Module handlers removed". Subsequent requests produce no module log entries.

**Why This Code Produces That Result:**
The `.myModule` namespace groups the module's handlers. When `off(".myModule")` is called, both handlers are removed from `document`. Any future AJAX requests will not trigger these handlers. This is the recommended pattern for cleaning up global AJAX handlers in SPAs.

### Real-World Cases

- **SPA Route Changes:** Unbinding a route's AJAX handlers before navigating away.
- **Plugin Destruction:** Removing a plugin's global AJAX handlers when the plugin is destroyed.
- **Modal Cleanup:** Unbinding modal-specific AJAX handlers when the modal closes.

---

## 11. Event Best Practices

### Definitions

**Core Definition:**
AJAX event best practices are empirically established guidelines for registering, handling, and unbinding AJAX events to ensure performance, reliability, and maintainability.

**Technical Definition:**
Best practices encompass using `.on()` instead of deprecated shorthand methods, namespacing global handlers for selective cleanup, avoiding `$.ajaxSetup()` for global configuration, preferring Promise methods (`.done()`, `.fail()`, `.always()`) over legacy callbacks, setting timeouts, and cleaning up handlers in SPAs. Global handlers should be attached to `document` and used sparingly to avoid overhead.

**Beginner-Friendly Explanation:**
These are the proven tips that experienced developers follow when working with AJAX events. Following them keeps your page fast, prevents memory leaks, and avoids conflicts between different parts of your code.

### Purposes

- To ensure AJAX events are registered and unbound correctly.
- To prevent memory leaks and duplicate handler execution in SPAs.
- To maintain clean, conflict-free event management in multi-module codebases.
- To optimize performance by minimizing global handler overhead.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Use `.on()` not shorthand | `.ajaxStart(fn)` | `.on("ajaxStart", fn)` |
| 2 | Namespace global handlers | `.on("ajaxStart", fn)` | `.on("ajaxStart.myModule", fn)` |
| 3 | Attach global handlers to `document` | `$("#el").on("ajaxStart", fn)` | `$(document).on("ajaxStart", fn)` |
| 4 | Avoid `$.ajaxSetup()` for global config | `$.ajaxSetup({ url: "/api" })` | Define settings objects per request |
| 5 | Prefer Promise methods over legacy callbacks | `success: fn` | `.done(fn).fail(fn)` |
| 6 | Unbind global handlers on teardown | Leave handlers attached | `.off(".myModule")` |
| 7 | Use `global: false` for non-monitored requests | Global handlers fire for all | `global: false` in request settings |
| 8 | Clean up before re-render in SPAs | Overwrite HTML without unbinding | `.off(".ns")` then re-render |
| 9 | Use `ajaxStop` for hiding indicators | Hide indicator in each request | Hide in `ajaxStop` |
| 10 | Set timeouts for all requests | No timeout | `timeout: 5000` |

**Syntax Rules:**

1. Always use `.on()` for global AJAX event registration; shorthand methods are removed.
2. Namespace all global handlers for selective unbinding.
3. Attach global handlers to `document` only.
4. Avoid `$.ajaxSetup()`; its use is not recommended.
5. Unbind handlers before re-rendering content in SPAs.
6. Use Promise methods for request-specific handling.

**Constraints and Limitations:**

- Global handlers add overhead to every AJAX request; use them sparingly.
- Namespacing requires discipline to apply consistently.
- `global: false` disables all global events, including loading indicators.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Best Practices in an SPA Module**

```html
<!DOCTYPE html>
<html>
<head>
    <title>AJAX Event Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #loading { display: none; position: fixed; top: 10px; right: 10px;
                   background: #333; color: white; padding: 10px; }
    </style>
</head>
<body>
    <div id="loading">Loading...</div>
    <button id="loadBtn">Load Data</button>
    <button id="destroy">Destroy Module</button>
    <div id="result"></div>

    <script>
        // ✅ BEST PRACTICE: Namespace all global handlers
        var NS = ".dataModule";

        // ✅ BEST PRACTICE: Attach global handlers to document
        $(document).on("ajaxStart" + NS, function() {
            $("#loading").show();
        });

        $(document).on("ajaxStop" + NS, function() {
            $("#loading").hide();
        });

        $(document).on("ajaxError" + NS, function(event, jqXHR, settings, thrownError) {
            $("#result").text("Error: " + thrownError);
        });

        // ✅ BEST PRACTICE: Use Promise methods for local handling
        $("#loadBtn").on("click", function() {
            $.ajax({
                url: "https://jsonplaceholder.typicode.com/posts/1",
                dataType: "json",
                timeout: 5000  // ✅ BEST PRACTICE: Set timeout
            })
            .done(function(data) {
                $("#result").html("<h3>" + data.title + "</h3>");
            })
            .fail(function(jqXHR, textStatus) {
                if (textStatus !== "abort") {
                    $("#result").text("Request failed: " + textStatus);
                }
            });
        });

        // ✅ BEST PRACTICE: Unbind all module handlers on destroy
        $("#destroy").on("click", function() {
            $(document).off(NS);
            $("#result").text("Module destroyed, handlers removed");
            console.log("Unbound all" + NS + " handlers");
        });
    </script>
</body>
</html>
```

**Expected Output:**
Clicking "Load Data" shows the loading indicator, loads the post title, and hides the indicator. Clicking "Destroy Module" removes all module handlers. Subsequent requests produce no loading indicator or error handling from the module.

**Why This Code Produces That Result:**
The code applies multiple best practices: namespaced global handlers (`.dataModule`) for selective unbinding, attachment to `document`, Promise methods for local handling, a timeout for reliability, and cleanup via `off(NS)` on destroy. This creates a self-contained module that does not leak handlers or conflict with other code.

### Real-World Cases

- **SPA Route Modules:** Each route module registers namespaced global handlers and cleans them up on navigation.
- **Plugin Development:** Plugins namespace their AJAX handlers and provide a `destroy()` method for cleanup.
- **Large Teams:** Consistent AJAX event conventions prevent conflicts between team members' code.

---

## References

- jQuery API Documentation — Ajax Events - https://api.jquery.com/Ajax_Events/
- jQuery API Documentation — ajaxStart event - https://api.jquery.com/ajaxStart/
- jQuery API Documentation — ajaxStop event - https://api.jquery.com/ajaxStop/
- jQuery API Documentation — ajaxComplete event - https://api.jquery.com/ajaxComplete/
- jQuery API Documentation — ajaxError event - https://api.jquery.com/ajaxError/
- jQuery API Documentation — ajaxSuccess event - https://api.jquery.com/ajaxSuccess/
- jQuery API Documentation — ajaxSend event - https://api.jquery.com/ajaxSend/
- jQuery Learning Center — Ajax Events - https://learn.jquery.com/ajax/ajax-events/
- jQuery API Documentation — .on() - https://api.jquery.com/on/
- jQuery API Documentation — .off() - https://api.jquery.com/off/
- jQuery API Documentation — jQuery.ajax() - https://api.jquery.com/jQuery.ajax/
- jQuery API Documentation — Ajax Category - https://api.jquery.com/category/ajax/
- LeetCode Discussion — SPA jQuery Resource Recycling Checklist - https://leetcode.cn/discuss/post/3838559/dan-ye-ying-yong-zhong-jquery-zi-yuan-hu-z96d/