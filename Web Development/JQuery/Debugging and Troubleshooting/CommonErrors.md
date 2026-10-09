# Common jQuery Errors — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Common jQuery Errors are the recurring categories of mistakes, misconfigurations, and environmental issues that cause jQuery code to fail at runtime. They span script loading, selector syntax, event binding, AJAX communication, plugin integration, and version compatibility.

**Technical Definition:** jQuery errors manifest as JavaScript exceptions (typically `ReferenceError`, `TypeError`, or `SyntaxError`), silent failures (handlers not firing, selectors returning empty sets), or network-level failures (AJAX requests rejected by CORS). They arise from four root causes: (1) **timing** — code executing before its dependencies are available; (2) **syntax** — incorrect selectors, event names, or API usage; (3) **environment** — conflicts with other libraries, CDN failures, or MIME type misconfigurations; and (4) **version drift** — using APIs that have been deprecated or removed in newer jQuery versions.

**Beginner-Friendly Explanation:** jQuery errors are the things that go wrong when your code is not talking to the browser the way you expected. Sometimes jQuery is not loaded yet, sometimes you typed a selector wrong, sometimes another library stole the `$` symbol, and sometimes you are using an old command that modern jQuery no longer supports. This cheat sheet catalogues the most common errors, explains why they happen, and shows how to fix them.

### Key Characteristics

- **Silent vs. loud failures:** Some errors throw visible exceptions in the Console; others fail silently (selectors returning empty sets, handlers never firing).
- **Timing is the most common cause:** The majority of jQuery errors stem from code executing before jQuery or the DOM is ready.
- **Version drift is increasingly common:** Applications using `.live()`, `.bind()`, or `.size()` break on jQuery 3.x and 4.x.
- **Environment matters:** CDN failures, ad blockers, and conflicting libraries can all produce the same `$ is not defined` symptom.

### Prerequisites

- Basic understanding of HTML, CSS, and JavaScript.
- Familiarity with jQuery fundamentals: selectors, `.on()`, `.ready()`, and AJAX.
- Access to browser Developer Tools (Console and Network panels) for diagnosis.

### Related Programming Areas

- **Debugging and Troubleshooting:** Using DevTools to diagnose and fix errors.
- **Script Loading and Dependency Management:** Ensuring correct load order.
- **AJAX and Network Security:** CORS, same-origin policy, and MIME types.
- **Plugin Architecture:** Initialization, dependencies, and version compatibility.

### Core Concepts / Features

This cheat sheet covers nine common jQuery errors, each with definitions, purposes, syntax rules, annotated examples, and real-world cases.

---

## Error 1: `$ is not defined` / `jQuery is not defined` — CDN Downtime and Path Errors

### Definitions

**Core Definition:** `$ is not defined` is a `ReferenceError` that occurs when JavaScript code attempts to use the `$` function (jQuery's alias) before the jQuery library has been loaded, or when the library failed to load entirely.

**Technical Definition:** When the browser encounters `$(...)` in JavaScript but the `$` variable has not been assigned to the jQuery function, it throws `Uncaught ReferenceError: $ is not defined`. The equivalent error using the full name is `jQuery is not defined`. The root causes are: (1) the jQuery script tag is missing, mispelled, or has an incorrect `src` path; (2) the jQuery script is loaded **after** the code that depends on it; (3) the CDN hosting jQuery is unreachable due to network issues, regional blocks, or corporate firewalls; (4) the jQuery file is loaded with `async` or `defer`, causing a race condition; or (5) an ad blocker or privacy extension has blocked the jQuery request.

**Beginner-Friendly Explanation:** This error means your code tried to use jQuery before jQuery was ready, or jQuery never arrived at all. It is like trying to use a tool before it has been delivered to the toolbox.

### Purposes

- To diagnose whether jQuery is loading correctly by checking the Network tab in DevTools.
- To implement CDN fallbacks that load a local copy of jQuery if the CDN fails.
- To ensure correct script ordering so jQuery loads before dependent code.
- To avoid `async` and `defer` on jQuery script tags unless the dependent code is also deferred.

### Syntax Rules and Structure

**Complete General Syntax (CDN Fallback):**
```html
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script>
    // Fallback if CDN fails
    window.jQuery || document.write('<script src="/local/jquery-3.7.1.min.js"><\/script>');
</script>
```

**Complete General Syntax (Correct Script Order):**
```html
<!-- jQuery MUST load first -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<!-- Then your code -->
<script src="/js/app.js"></script>
```

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Missing script tag | Network tab shows no jQuery request | Add the script tag |
| Wrong path | Network tab shows 404 for jQuery | Correct the `src` path |
| CDN failure | Network tab shows blocked/timeout | Add a fallback |
| Wrong order | jQuery request appears after app.js | Move jQuery tag earlier |
| `async`/`defer` | Race condition | Remove or add `defer` to both |

**Syntax Rules:**

- Always load jQuery **before** any script that depends on it.
- Use `window.jQuery` for the fallback check; `$` may be undefined even if jQuery loaded (in noConflict mode).
- Avoid `async` on jQuery script tags; `defer` can work if all dependent scripts also use `defer` and are in order.
- Check the Console and Network tabs together: the Console shows the error, the Network tab shows whether jQuery loaded.

**Constraints and Limitations:**

- The `document.write` fallback is deprecated in some contexts and may not work in modern frameworks.
- Ad blockers can block jQuery from popular CDNs even when the script tag is correct.
- In WordPress, jQuery is loaded in noConflict mode, so `$` may be undefined even though `jQuery` is available.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: CDN Fallback Implementation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>jQuery CDN Fallback Demo</title>
</head>
<body>
  <p id="log"></p>

  <!-- Step 1: Primary CDN -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script>
    // Step 2: Fallback to local copy if CDN failed
    if (!window.jQuery) {
      document.write('<script src="/js/jquery-3.7.1.min.js"><\/script>');
    }
  </script>
  <script>
    // Step 3: Now jQuery is guaranteed to be loaded
    $(function() {
      $("#log").text("jQuery " + $.fn.jquery + " loaded successfully.");
    });
  </script>
</body>
</html>
```

**Expected Output:** If the CDN is reachable, the log displays "jQuery 3.7.1 loaded successfully." If the CDN fails, the local copy loads, and the log still displays the version.

**Why this output:** The fallback check `if (!window.jQuery)` runs immediately after the CDN script tag. If the CDN failed to load, `window.jQuery` is undefined, and the `document.write` injects a local script tag synchronously before the next script executes.

### Real-World Cases

- **Corporate environments:** Firewalls block popular CDNs; the fallback ensures jQuery still loads.
- **WordPress:** jQuery is enqueued via `wp_enqueue_script('jquery')`, and custom scripts must use `jQuery` instead of `$` or wrap code in an IIFE.
- **Offline development:** A local copy of jQuery ensures the application works without internet access.

---

## Error 2: Incorrect Selector — Typos, Missing `.` or `#`, Invalid Attribute Syntax

### Definitions

**Core Definition:** An incorrect selector is a jQuery selector expression that does not match any elements in the DOM because of a typo, a missing class (`.`) or ID (`#`) prefix, a case mismatch, or invalid syntax.

**Technical Definition:** jQuery selectors are passed to `$()` and use CSS selector syntax with additional jQuery extensions. Common mistakes include: using `#` for a class (should be `.`), using `.` for an ID (should be `#`), omitting the prefix entirely (which causes jQuery to search for tag names), using an ID with special characters (e.g., `.` or `:` in the ID) without escaping, and case mismatches (IDs and classes are case-sensitive in standards mode). When the selector does not match any elements, `$(selector)` returns an empty jQuery object (`.length === 0`), and subsequent method calls silently do nothing.

**Beginner-Friendly Explanation:** This error is like looking for someone by the wrong name. If you ask for "John" but the person is named "Jon," you will not find them. jQuery selectors must match exactly: `#` for IDs, `.` for classes, and no prefix for tag names.

### Purposes

- To ensure that selectors match the intended elements by verifying ID and class names in the DOM.
- To use the correct prefix: `#` for IDs, `.` for classes, and no prefix for tags.
- To escape special characters in IDs or class names that conflict with CSS selector syntax.
- To check `.length` to confirm that a selector matched elements.

### Syntax Rules and Structure

**Complete General Syntax (Correct Selectors):**
```javascript
$("#myId")       // ID selector
$(".myClass")    // Class selector
$("div")         // Tag selector
$("div.myClass") // Combined tag + class
$("[data-role='button']") // Attribute selector
```

**Common Selector Mistakes:**

| Mistake | Incorrect | Correct |
|---------|-----------|---------|
| Missing prefix | `$("myId")` | `$("#myId")` or `$(".myId")` |
| Wrong prefix | `$("#myClass")` (class with #) | `$(".myClass")` |
| Case mismatch | `$("#myid")` (actual ID: `myId`) | `$("#myId")` |
| Unescaped special char | `$("#my.id")` | `$("#my\\.id")` |
| Extra space | `$("#myId ")` | `$("#myId")` |

**Syntax Rules:**

- Always check `.length` after selecting to verify that elements were matched: `if ($("#myId").length === 0) { /* not found */ }`.
- Use DevTools' Console to test selectors interactively: type `$("#myId").length` and press Enter.
- Escape special characters in IDs with a backslash: `$("#foo\\:bar")`.
- IDs must be unique in the document; duplicate IDs cause unpredictable selection.

**Constraints and Limitations:**

- jQuery's `:visible` and `:hidden` selectors are extensions that do not work with native `querySelectorAll()`.
- Numeric IDs (e.g., `id="123"`) are invalid in HTML and require escaping in selectors.
- Case sensitivity depends on the document mode; standards mode is case-sensitive for classes and IDs.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Diagnosing a Selector Mismatch**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Selector Mismatch Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="box" id="myBox">Content</div>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: Incorrect selector — # for a class
      var $wrong = $("#box");
      $("#log").append("Wrong selector (#box) length: " + $wrong.length + "<br>");

      // Step 2: Correct selector — . for a class
      var $correct = $(".box");
      $("#log").append("Correct selector (.box) length: " + $correct.length + "<br>");

      // Step 3: Case-sensitive ID
      var $caseWrong = $("#mybox");  // lowercase 'b'
      $("#log").append("Case-wrong selector (#mybox) length: " + $caseWrong.length + "<br>");

      // Step 4: Correct case
      var $caseCorrect = $("#myBox");
      $("#log").append("Case-correct selector (#myBox) length: " + $caseCorrect.length);
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Wrong selector (#box) length: 0
Correct selector (.box) length: 1
Case-wrong selector (#mybox) length: 0
Case-correct selector (#myBox) length: 1
```

**Why this output:** `#box` looks for an element with `id="box"`, but the element has `class="box"` and `id="myBox"`. The class selector `.box` matches correctly. The ID selector `#mybox` fails because IDs are case-sensitive (`myBox` ≠ `mybox`).

### Real-World Cases

- **Form validation:** A typo in a field selector causes validation to silently fail.
- **Event binding:** A mismatched selector means the handler is never attached.
- **Plugin initialization:** An empty selection causes the plugin to initialize on zero elements.

---

## Error 3: Script Loaded in Wrong Order — jQuery Running Before Library Load

### Definitions

**Core Definition:** Script loading order errors occur when JavaScript code that depends on jQuery executes before the jQuery library has finished loading, or when multiple versions of jQuery are loaded and overwrite each other's plugin registrations.

**Technical Definition:** Browsers execute `<script>` tags in the order they appear in the HTML, unless `async` or `defer` attributes are present. If a plugin or application script appears before the jQuery script tag, it executes first, and `$` is undefined. Additionally, loading jQuery twice — common in WordPress themes and plugins — causes the second jQuery instance to replace the first, wiping out all plugins registered on the first instance's `$.fn`. The correct order is: jQuery first, then plugins, then application code.

**Beginner-Friendly Explanation:** This is like trying to build a house before the foundation is poured. jQuery is the foundation; plugins and application code are the house. If you put the house first, it collapses.

### Purposes

- To ensure that jQuery loads before any plugin or application script that depends on it.
- To prevent the "load jQuery twice" problem that wipes out plugin registrations.
- To understand how `async` and `defer` affect execution order.
- To debug plugin failures caused by incorrect script ordering.

### Syntax Rules and Structure

**Complete General Syntax (Correct Order):**
```html
<!-- 1. jQuery first -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<!-- 2. Plugins second -->
<script src="/js/jquery.plugin1.js"></script>
<script src="/js/jquery.plugin2.js"></script>
<!-- 3. Application code last -->
<script src="/js/app.js"></script>
```

**Incorrect Order (Plugin Before jQuery):**
```html
<!-- WRONG: Plugin loads before jQuery -->
<script src="/js/jquery.plugin1.js"></script>
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
```

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `$ is not defined` | Plugin before jQuery | Move jQuery first |
| Plugin method undefined | jQuery loaded twice | Remove duplicate jQuery |
| Some plugins work, others not | Multiple jQuery versions | Use one version |

**Syntax Rules:**

- Load jQuery once, and only once.
- Load plugins after jQuery.
- Load application code after plugins (or inside `$(document).ready()`).
- Avoid `async` on jQuery and plugin scripts unless the dependency graph is carefully managed.

**Constraints and Limitations:**

- `defer` preserves execution order for scripts with `defer`, but inline scripts without `defer` still execute immediately.
- WordPress loads jQuery in the footer by default; plugins that assume jQuery is in the `<head>` may fail.
- Dynamically loaded scripts (`$.getScript()`) execute asynchronously and may arrive out of order.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Diagnosing Plugin Failure from Wrong Order**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Load Order Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    $(function() {
      // Simulate a plugin that registers itself on $.fn
      $.fn.myPlugin = function() {
        return this.each(function() {
          $(this).text("Plugin applied!");
        });
      };

      // Step 1: Check if the plugin exists
      if (typeof $.fn.myPlugin === "function") {
        $("#log").text("Plugin registered successfully.");
      } else {
        $("#log").text("Plugin is undefined — jQuery may have been reloaded.");
      }

      // Step 2: Simulate loading jQuery again (which wipes plugins)
      // In a real page, this would be a second <script> tag
      // After reloading, $.fn.myPlugin would be undefined
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Plugin registered successfully.
```

**Why this output:** The plugin is registered after jQuery loads. If a second jQuery instance were loaded after this script, the plugin would be lost. The example demonstrates the principle without actually reloading jQuery.

### Real-World Cases

- **WordPress:** Themes and plugins frequently load their own jQuery, causing conflicts; use `wp_enqueue_script` with `jquery` as a dependency.
- **Magento:** Requires jQuery to be loaded before other scripts; the `requirejs-config.js` manages dependencies.
- **Custom CMS:** Always verify the order of `<script>` tags in the page source.

---

## Error 4: DOM Not Ready — Running Code Before Elements Exist

### Definitions

**Core Definition:** A DOM-not-ready error occurs when jQuery code attempts to select or manipulate DOM elements before those elements have been parsed and inserted into the document tree by the browser.

**Technical Definition:** When a `<script>` tag is placed in the `<head>` or before the elements it targets, the elements do not exist yet when the script executes. `$(selector)` returns an empty jQuery object, and event handlers are never bound because there are no elements to bind to. The solution is to wrap the code in `$(document).ready()` (or its shorthand `$(function() { ... })`), which defers execution until the DOM is fully parsed. The `DOMContentLoaded` event fires after the HTML is parsed but before images and stylesheets finish loading; `window.onload` fires later, after all resources are loaded.

**Beginner-Friendly Explanation:** This is like arriving at a store before it opens. You want to buy something, but the shelves are not stocked yet. `$(document).ready()` tells your code to wait until the store is open — that is, until all the HTML elements exist.

### Purposes

- To ensure that DOM elements exist before attempting to select or manipulate them.
- To bind event handlers to elements that have been parsed.
- To initialize jQuery plugins after their target elements are available.
- To avoid silent failures where selectors return empty sets.

### Syntax Rules and Structure

**Complete General Syntax (Document Ready):**
```javascript
$(document).ready(function() {
    // DOM is fully parsed; safe to manipulate elements
});

// Shorthand
$(function() {
    // Same as above
});
```

**Complete General Syntax (Alternative — Script at End of Body):**
```html
<body>
  <div id="myElement">Content</div>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script>
    // DOM is already parsed because the script is at the end of the body
    $("#myElement").css("color", "red");
  </script>
</body>
```

| Placement | DOM Ready? | Recommended? |
|-----------|-----------|--------------|
| `<head>` without `ready()` | No | No |
| `<head>` with `ready()` | Deferred | Yes |
| End of `<body>` | Yes | Yes |

**Syntax Rules:**

- Always wrap DOM manipulation code in `$(document).ready()` or `$(function() { ... })` when the script is in the `<head>`.
- Scripts at the end of `<body>` do not need `ready()` because the DOM is already parsed.
- Avoid mixing `$(document).ready()` with scripts at the end of the body; it works but is redundant.
- Use `$(window).on("load", ...)` only when you need images and stylesheets to be loaded.

**Constraints and Limitations:**

- `$(document).ready()` does not wait for images or stylesheets.
- Dynamically added elements (via AJAX or JavaScript) are not covered by `ready()`; use event delegation for those.
- Multiple `ready()` calls are queued and executed in order.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: DOM Not Ready vs. DOM Ready**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>DOM Ready Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script>
    // Step 1: This runs immediately — element does NOT exist yet
    console.log("Before ready — length:", $("#myElement").length);

    // Step 2: This runs after DOM is ready — element exists
    $(document).ready(function() {
      console.log("Inside ready — length:", $("#myElement").length);
    });
  </script>
</head>
<body>
  <div id="myElement">Content</div>
</body>
</html>
```

**Expected Output (in Console):**
```
Before ready — length: 0
Inside ready — length: 1
```

**Why this output:** The first `console.log` executes while the browser is still parsing the `<head>`, so `#myElement` does not exist yet. The `ready()` callback executes after the entire DOM (including `<body>`) is parsed, so the element is found.

### Real-World Cases

- **Plugin initialization:** Plugins must be initialized after their target elements exist.
- **Event binding:** Handlers bound before elements exist are silently lost.
- **AJAX-loaded content:** Content loaded via AJAX is not covered by `ready()`; use `.on()` with delegation.

---

## Error 5: Event Handler Not Triggered — Typos, Overwriting, and `return false`

### Definitions

**Core Definition:** An event handler not triggered error occurs when a jQuery event binding fails to attach or execute, typically due to incorrect event names, overwriting handlers, binding before the element exists, or using `return false` incorrectly.

**Technical Definition:** jQuery event handlers are bound with `.on(eventType, handler)`. Common causes of failure include: (1) a typo in the event name (e.g., `"clcik"` instead of `"click"`); (2) binding to an element that does not exist at binding time; (3) overwriting a handler by binding a second handler to the same event without using `.on()` correctly; (4) using `return false` inside a handler, which calls both `preventDefault()` and `stopPropagation()`, potentially breaking other handlers; and (5) binding to dynamically added elements without using event delegation.

**Beginner-Friendly Explanation:** This is like putting a note on a door that does not exist yet. When the door is finally installed, the note is not there. Or you wrote the wrong event name — like "click" misspelled — so the browser never knows what to listen for.

### Purposes

- To ensure event handlers are bound to the correct event type.
- To bind handlers only after the target elements exist in the DOM.
- To use event delegation for dynamically added elements.
- To avoid `return false` misuse that breaks event propagation.

### Syntax Rules and Structure

**Complete General Syntax (Correct Event Binding):**
```javascript
// Direct binding (element must exist)
$("#myButton").on("click", function() {
    // Handler code
});

// Delegated binding (works for current and future elements)
$(document).on("click", "#myButton", function() {
    // Handler code
});
```

**Common Event Binding Mistakes:**

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Typo in event name | Handler never fires | Correct the event name |
| Binding before element exists | Handler never fires | Use `ready()` or delegation |
| Overwriting with `.click()` after `.on()` | Only last handler fires | Use `.on()` consistently |
| `return false` in handler | Stops propagation and default | Use `preventDefault()` only |

**Syntax Rules:**

- Use `.on()` instead of `.click()`, `.bind()`, or `.live()` for all event binding.
- Use event delegation (`$(parent).on(event, selector, handler)`) for elements that are added dynamically.
- Avoid `return false` unless you intend to both prevent the default action and stop propagation.
- Check the Event Listeners pane in DevTools to verify that handlers are bound.

**Constraints and Limitations:**

- `.click()` shorthand is deprecated in jQuery 3.x; use `.on("click", ...)`.
- `.live()` was removed in jQuery 1.9; use `.on()` with delegation.
- Event delegation cannot be used for non-bubbling events (e.g., `focus`, `blur`, `load`).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Direct Binding vs. Delegated Binding**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Event Delegation Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="container">
    <button class="btn">Existing Button</button>
  </div>
  <button id="addBtn">Add Button</button>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: Direct binding — only works for the existing button
      $(".btn").on("click", function() {
        $("#log").append("Direct handler fired.<br>");
      });

      // Step 2: Delegated binding — works for current and future buttons
      $("#container").on("click", ".btn", function() {
        $("#log").append("Delegated handler fired.<br>");
      });

      // Step 3: Add a new button dynamically
      $("#addBtn").click(function() {
        $("#container").append('<button class="btn">New Button</button>');
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking the existing button fires both handlers (direct and delegated). Clicking a newly added button fires only the delegated handler. The log shows "Direct handler fired" and "Delegated handler fired" for the existing button, and only "Delegated handler fired" for the new button.

**Why this output:** Direct binding is applied only to elements that exist at binding time. The new button did not exist, so the direct handler is not attached. Delegated binding is attached to the parent (`#container`), so it catches events from both existing and future child elements.

### Real-World Cases

- **Dynamic forms:** Delegated events handle dynamically added form fields.
- **Chat applications:** Delegated events handle newly added messages.
- **Single-page applications:** Delegation ensures events work after route changes.

---

## Error 6: AJAX Request Failure — CORS Errors, Status Codes, Mismatched Data Types

### Definitions

**Core Definition:** An AJAX request failure occurs when a jQuery AJAX call cannot complete successfully due to cross-origin restrictions (CORS), HTTP error status codes, or a mismatch between the expected `dataType` and the actual response `Content-Type`.

**Technical Definition:** jQuery AJAX failures manifest in the `.fail()` callback (or the `error` option) with three arguments: `jqXHR`, `textStatus`, and `errorThrown`. The `textStatus` values include `"timeout"`, `"error"`, `"abort"`, `"parsererror"`, and `"nocontent"`. CORS failures occur when the browser blocks the response because the server did not send the appropriate `Access-Control-Allow-Origin` header. Status code failures (404, 500, etc.) occur when the server responds with an error status. Parser errors occur when the `dataType` does not match the response format (e.g., expecting JSON but receiving HTML).

**Beginner-Friendly Explanation:** An AJAX request is like sending a letter and waiting for a reply. It can fail because the post office blocked the delivery (CORS), the recipient returned it to sender (404/500), or the reply was in a language you did not expect (parser error). This error type covers all three scenarios.

### Purposes

- To diagnose CORS failures by checking the Console and Network tab for the `Access-Control-Allow-Origin` header.
- To handle HTTP error status codes gracefully in the `.fail()` callback.
- To ensure that the `dataType` matches the response `Content-Type`.
- To implement retry logic or user-friendly error messages for failed requests.

### Syntax Rules and Structure

**Complete General Syntax (Error Handling):**
```javascript
$.ajax({
    url: "/api/data",
    type: "GET",
    dataType: "json",
    success: function(data) {
        // Handle success
    },
    error: function(jqXHR, textStatus, errorThrown) {
        if (textStatus === "parsererror") {
            // Response was not valid JSON
        } else if (jqXHR.status === 404) {
            // Not found
        } else if (jqXHR.status === 0) {
            // CORS or network failure
        }
    }
});
```

**Common AJAX Failure Causes:**

| `textStatus` | Cause | Fix |
|--------------|-------|-----|
| `"error"` | HTTP error (404, 500) | Check server endpoint |
| `"timeout"` | Request timed out | Increase timeout or check server |
| `"abort"` | Request was aborted | Handle in abort handler |
| `"parsererror"` | `dataType` mismatch | Correct the `dataType` |
| `"nocontent"` | 204 No Content | Handle empty response |

**Syntax Rules:**

- Always provide an `error` or `.fail()` callback to handle failures.
- Check `jqXHR.status` to determine the HTTP status code.
- For CORS, ensure the server sends `Access-Control-Allow-Origin` matching the requesting origin.
- Use `dataType: "json"` only when the server returns valid JSON with the correct `Content-Type`.

**Constraints and Limitations:**

- CORS failures cannot be caught by the `error` callback in some browsers; the response is blocked before jQuery sees it.
- `dataType: "json"` expects a JSON response; if the server returns JSON with the wrong MIME type, use `dataType: "text json"`.
- Cross-domain requests with credentials require `xhrFields: { withCredentials: true }` on the client and `Access-Control-Allow-Credentials: true` on the server.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Handling a 404 and a Parser Error**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>AJAX Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="load404">Load 404</button>
  <button id="loadParser">Load Parser Error</button>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: 404 error
      $("#load404").click(function() {
        $.ajax({
          url: "https://jsonplaceholder.typicode.com/nonexistent",
          dataType: "json"
        }).fail(function(jqXHR, textStatus, errorThrown) {
          $("#log").text(
            "404 Test — Status: " + jqXHR.status +
            " | textStatus: " + textStatus +
            " | errorThrown: " + errorThrown
          );
        });
      });

      // Step 2: Parser error (expecting JSON, receiving HTML)
      $("#loadParser").click(function() {
        $.ajax({
          url: "https://example.com",  // Returns HTML, not JSON
          dataType: "json"
        }).fail(function(jqXHR, textStatus, errorThrown) {
          $("#log").text(
            "Parser Test — Status: " + jqXHR.status +
            " | textStatus: " + textStatus +
            " | errorThrown: " + errorThrown
          );
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:**
- Clicking "Load 404" displays: `404 Test — Status: 404 | textStatus: error | errorThrown: Not Found`
- Clicking "Load Parser Error" displays: `Parser Test — Status: 200 | textStatus: parsererror | errorThrown: SyntaxError: Unexpected token <`

**Why this output:** The first request returns a 404 status, so `textStatus` is `"error"` and `errorThrown` is `"Not Found"`. The second request returns HTML, but `dataType` is `"json"`, so jQuery attempts to parse HTML as JSON and fails with a `parsererror`.

### Real-World Cases

- **Third-party API integration:** CORS failures are common when calling APIs from a different origin.
- **Form submission:** Server validation errors return 400 status codes with JSON error messages.
- **Data loading:** Parser errors occur when the server returns HTML error pages instead of JSON.

---

## Error 7: Plugin Initialization Failure — Missing Dependencies, Calling Methods Before Initialization

### Definitions

**Core Definition:** A plugin initialization failure occurs when a jQuery plugin cannot be initialized because its dependencies are missing, the plugin script is loaded incorrectly, or methods are called on the plugin before it has been initialized on an element.

**Technical Definition:** jQuery plugins are functions assigned to `$.fn.pluginName`. They fail to initialize when: (1) the plugin script is not loaded (404, wrong path, or loaded before jQuery); (2) the plugin depends on other libraries or plugins that are not loaded; (3) the plugin is initialized on an empty selection (the target element does not exist); (4) methods are called on a plugin instance before the plugin has been initialized on that element; or (5) the plugin is incompatible with the current jQuery version.

**Beginner-Friendly Explanation:** This is like trying to use a machine before it has been assembled. The plugin is the machine; it needs all its parts (dependencies) and the right instructions (jQuery version) before it can work. And you cannot operate the machine before it is turned on (initialized).

### Purposes

- To verify that all plugin dependencies are loaded before initializing the plugin.
- To ensure the plugin script is loaded after jQuery and before application code.
- To initialize the plugin only on elements that exist.
- To call plugin methods only after the plugin has been initialized.

### Syntax Rules and Structure

**Complete General Syntax (Plugin Initialization):**
```javascript
// Step 1: Ensure DOM is ready
$(function() {
    // Step 2: Ensure the plugin is loaded
    if (typeof $.fn.myPlugin === "function") {
        // Step 3: Initialize on an existing element
        if ($("#target").length > 0) {
            $("#target").myPlugin({ option: "value" });
        }
    }
});
```

**Common Plugin Initialization Mistakes:**

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Plugin before jQuery | `myPlugin is not a function` | Move plugin after jQuery |
| Plugin dependency missing | Plugin fails silently | Load dependency first |
| Empty selection | Plugin does nothing | Check `.length` |
| Method before init | `cannot read property of undefined` | Initialize first |
| Version mismatch | Plugin throws errors | Use compatible jQuery version |

**Syntax Rules:**

- Always load plugin dependencies (e.g., jQuery UI for a plugin that extends it) before the plugin itself.
- Check `typeof $.fn.pluginName === "function"` before initializing.
- Check `$("#target").length > 0` before initializing.
- Initialize the plugin inside `$(document).ready()` or at the end of the body.

**Constraints and Limitations:**

- Some plugins have their own dependency requirements (e.g., Moment.js for date plugins).
- Plugins that use `$.fn.extend` may conflict with other plugins.
- Plugin initialization may fail silently if the target element is hidden.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Diagnosing a Missing Plugin**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Plugin Init Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="target">Target</div>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: Check if the plugin exists
      if (typeof $.fn.myPlugin === "function") {
        $("#log").text("Plugin is loaded.");
      } else {
        $("#log").text("Plugin is NOT loaded. Check the script tag.");
        return;
      }

      // Step 2: Check if the target exists
      if ($("#target").length === 0) {
        $("#log").text("Target element not found.");
        return;
      }

      // Step 3: Initialize the plugin
      $("#target").myPlugin();
      $("#log").text("Plugin initialized successfully.");
    });
  </script>
</body>
</html>
```

**Expected Output:** If the plugin is not loaded, the log displays "Plugin is NOT loaded. Check the script tag." If the plugin is loaded and the target exists, the log displays "Plugin initialized successfully."

**Why this output:** The script checks for the plugin's existence on `$.fn` and the target's existence before attempting initialization. This prevents the cryptic `myPlugin is not a function` error.

### Real-World Cases

- **jQuery UI:** Widgets must be initialized after jQuery UI is loaded.
- **DataTables:** Requires jQuery and optionally Bootstrap or Foundation CSS.
- **Select2:** Requires jQuery and must be initialized on `<select>` elements after they exist.

---

## Error 8: `$ is not a function` — Conflict with Other Libraries or `noConflict()` Mode

### Definitions

**Core Definition:** `$ is not a function` is a `TypeError` that occurs when the `$` variable has been overwritten by another library (such as Prototype or MooTools) or when jQuery is running in `noConflict()` mode and `$` no longer refers to jQuery.

**Technical Definition:** Many JavaScript libraries use `$` as their primary function or variable name. When jQuery is loaded alongside such a library, the last-loaded library claims `$`. If jQuery is loaded first and another library overwrites `$`, jQuery's alias is lost. Conversely, if jQuery is loaded second, it overwrites the other library's `$`. To coexist, jQuery provides `jQuery.noConflict()`, which relinquishes control of `$` and restores the previous library's `$`. After calling `noConflict()`, all jQuery code must use `jQuery` instead of `$`, or wrap code in an IIFE that receives jQuery as `$`. WordPress loads jQuery in `noConflict()` mode by default.

**Beginner-Friendly Explanation:** Imagine two people want to use the same nickname, `$`. If both answer to `$`, confusion ensues. `noConflict()` is like one person saying, "You can have the nickname `$`; I will go by my full name, `jQuery`." This error means someone else has taken the `$` nickname, and your code needs to use `jQuery` instead.

### Purposes

- To resolve conflicts between jQuery and other libraries that use `$`.
- To use `jQuery.noConflict()` to return `$` to the other library.
- To wrap jQuery code in an IIFE that receives `jQuery` as `$` for local use.
- To recognize WordPress's noConflict mode and use `jQuery` instead of `$`.

### Syntax Rules and Structure

**Complete General Syntax (noConflict):**
```javascript
// Step 1: Load other library first, then jQuery
// Step 2: Call noConflict to release $
var jq = jQuery.noConflict();

// Step 3: Use jQuery via the new alias
jq(document).ready(function() {
    jq("#element").hide();
});

// Step 4: Other library can use $ again
$("element").style.display = "none";
```

**Complete General Syntax (IIFE with Local $):**
```javascript
jQuery.noConflict();

(function($) {
    $(function() {
        // $ is jQuery inside this function
        $("#element").hide();
    });
})(jQuery);

// Outside the IIFE, $ is the other library
```

| Scenario | Solution |
|----------|----------|
| Another library overwrites `$` | Use `jQuery` instead of `$` |
| Both libraries need `$` | Use `noConflict()` and assign a new alias |
| WordPress | Use `jQuery` or wrap in IIFE |
| Plugin uses `$` internally | Wrap plugin code in IIFE |

**Syntax Rules:**

- Call `jQuery.noConflict()` after jQuery is loaded and before any code that uses `$`.
- Use the return value of `noConflict()` as the jQuery alias: `var jq = jQuery.noConflict()`.
- Wrap all jQuery code in an IIFE that receives `jQuery` as `$` when using noConflict mode.
- In WordPress, always use `jQuery` or the IIFE pattern.

**Constraints and Limitations:**

- `noConflict(true)` removes all jQuery variables from the global scope; most plugins rely on the `jQuery` variable and may break.
- Multiple versions of jQuery on the same page can cause plugin registrations to be lost.
- Some plugins use `$` internally without wrapping in an IIFE; these plugins break in noConflict mode.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Resolving a Conflict with noConflict()**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>noConflict Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script>
    // Step 1: Simulate another library taking $ after jQuery loads
    var $ = "I am NOT jQuery";

    // Step 2: Use jQuery.noConflict() to release $
    var jq = jQuery.noConflict();

    // Step 3: Use jq as the jQuery alias
    jq(function() {
      jq("#log").text("jQuery works via jq alias. $ is: " + $);
    });
  </script>
</head>
<body>
  <p id="log"></p>
</body>
</html>
```

**Expected Output:**
```
jQuery works via jq alias. $ is: I am NOT jQuery
```

**Why this output:** After another library overwrites `$`, `jQuery.noConflict()` restores the original `$` (the string) and returns the jQuery object, which is assigned to `jq`. The code uses `jq` for jQuery operations, and `$` remains the other library's value.

### Real-World Cases

- **WordPress:** jQuery is loaded in noConflict mode; themes and plugins must use `jQuery` or the IIFE pattern.
- **Prototype.js + jQuery:** Both libraries use `$`; `noConflict()` is required.
- **MooTools + jQuery:** Same conflict; use `noConflict()` and a custom alias.

---

## Error 9: Mismatched Version Methods — Deprecated and Removed Functions

### Definitions

**Core Definition:** A version mismatch error occurs when code written for an older version of jQuery uses methods that have been deprecated or removed in the current version, such as `.live()`, `.bind()`, `.size()`, `.load()` (as an event shortcut), or `.attr()` for boolean properties.

**Technical Definition:** jQuery's API has evolved significantly across major versions. jQuery 1.7 introduced `.on()` and `.off()` as unified event binding methods; `.live()` was deprecated in 1.7 and removed in 1.9. `.bind()` and `.delegate()` were superseded by `.on()`. `.size()` was deprecated in 1.8 and removed in 3.0; use `.length` instead. `.load()`, `.error()`, and `.unload()` as event shortcuts were removed in 3.0; use `.on("load", handler)` etc. `.attr()` for boolean properties (e.g., `checked`, `disabled`) was replaced by `.prop()` in 1.6. The jQuery Migrate plugin can restore removed APIs and log deprecation warnings.

**Beginner-Friendly Explanation:** This is like using an old instruction manual for a new machine. The machine has changed — some buttons have been moved or removed — and the old instructions no longer work. The fix is to update your code to use the new methods.

### Purposes

- To migrate code from older jQuery versions to modern versions.
- To replace `.live()` with `.on()` using event delegation.
- To replace `.bind()` with `.on()`.
- To replace `.size()` with `.length`.
- To replace `.attr()` with `.prop()` for boolean properties.

### Syntax Rules and Structure

**Migration Reference Table:**

| Removed/Deprecated | Replacement | Version Removed |
|--------------------|-------------|-----------------|
| `.live()` | `.on()` with delegation | 1.9 |
| `.bind()` | `.on()` | Deprecated 1.7 |
| `.delegate()` | `.on()` | Deprecated 1.7 |
| `.size()` | `.length` | 3.0 |
| `.load()` (event) | `.on("load", ...)` | 3.0 |
| `.error()` (event) | `.on("error", ...)` | 3.0 |
| `.unload()` (event) | `.on("unload", ...)` | 3.0 |
| `.attr("checked")` | `.prop("checked")` | 1.6 |

**Complete General Syntax (Before and After):**
```javascript
// OLD: .live() — removed in 1.9
$("a.link").live("click", handler);

// NEW: .on() with delegation
$(document).on("click", "a.link", handler);

// OLD: .bind()
$("#btn").bind("click", handler);

// NEW: .on()
$("#btn").on("click", handler);

// OLD: .size()
var count = $("div").size();

// NEW: .length
var count = $("div").length;
```

**Syntax Rules:**

- Replace `.live()` with `$(document).on(event, selector, handler)` or `$(closestParent).on(event, selector, handler)`.
- Replace `.bind()` with `.on()`.
- Replace `.size()` with `.length`.
- Replace `.attr()` with `.prop()` for boolean properties (`checked`, `disabled`, `selected`).
- Use jQuery Migrate to identify deprecated API usage before upgrading.

**Constraints and Limitations:**

- `.live()` cannot be simply replaced with `.on()` without a delegate; `.on()` without a selector binds directly to existing elements only.
- jQuery Migrate should be removed after the migration is complete.
- Some plugins may still use removed APIs; update the plugin or replace it.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Migrating from `.live()` to `.on()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Migration Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="container">
    <a href="#" class="link">Link 1</a>
  </div>
  <button id="addLink">Add Link</button>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: OLD CODE (would fail on jQuery 3.x)
      // $("a.link").live("click", function() { ... });

      // Step 2: NEW CODE — delegated .on()
      $("#container").on("click", "a.link", function(e) {
        e.preventDefault();
        $("#log").append("Link clicked: " + $(this).text() + "<br>");
      });

      // Step 3: Add a new link dynamically
      $("#addLink").click(function() {
        $("#container").append('<a href="#" class="link">New Link</a>');
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Link 1" logs "Link clicked: Link 1". Clicking "Add Link" adds a new link, and clicking that new link logs "Link clicked: New Link". The delegated `.on()` works for both existing and future links.

**Why this output:** `.on()` with a selector on a parent element provides the same delegation behavior as `.live()`, but is compatible with jQuery 1.7+ and recommended for all versions.

### Real-World Cases

- **Upgrading from jQuery 1.x to 3.x:** Replacing `.live()`, `.bind()`, `.size()`, and `.load()`.
- **Plugin compatibility:** Some older plugins use removed APIs; update or replace them.
- **WordPress themes:** Themes written for jQuery 1.x may use `.live()`; update to `.on()`.

---

## References

- jQuery CDN Best Practices — Liquid Web — https://www.liquidweb.com/wordpress/errors/jquery-not-defined/
- How to fix `$ is not defined` — TrackJS — https://trackjs.com/javascript-errors/jquery-is-not-defined/
- Why Does jQuery or a DOM Method Fail to Find an Element? — tsecurity.de — https://tsecurity.de/de/2456259/
- Changing the Load Order of Theme and Plugin — WordPress.org — https://wordpress.org/support/topic/loading-of-colourbox-plugin-after-newspress-theme/
- jQuery元素加载完成 — Tencent Cloud — https://cloud.tencent.cn/developer/information/jquery%20%E5%85%83%E7%B4%A0%E5%8A%A0%E8%BD%BD%E5%AE%8C%E6%88%90
- Troubleshooting jQuery events not working — Fluid Checkout — https://fluidcheckout.com/docs/troubleshoot-js-events/
- JQuery Ajax跨域 — Tencent Cloud — https://cloud.tencent.cn/developer/information/JQuery Ajax跨域Java Web服务调用出错-article
- jQuery插件如何解决常见问题 — 亿速云 — https://m.yisu.com/zixun/1079579.html
- jQuery.noConflict() — jQuery API Documentation — https://api.jquery.com/jQuery.noConflict/
- jQueryのバージョンアップに伴うエラー発生時の対応ガイド — Zendesk — https://mkt-apps.zendesk.com/hc/ja/articles/4900197112606
- jQuery .on() — jQuery API Documentation — https://api.jquery.com/on/
- jQuery .live() — jQuery API Documentation — https://api.jquery.com/live/
- jQuery .size() — jQuery API Documentation — https://api.jquery.com/size/
- jQuery .prop() — jQuery API Documentation — https://api.jquery.com/prop/
- jQuery Migrate Plugin — jQuery — https://github.com/jquery/jquery-migrate
- jQuery .bind() — jQuery API Documentation — https://api.jquery.com/bind/
- jQuery .delegate() — jQuery API Documentation — https://api.jquery.com/delegate/
- $(document).ready() — jQuery Learning Center — https://learn.jquery.com/using-jquery-core/document-ready/