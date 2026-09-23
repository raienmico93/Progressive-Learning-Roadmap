# Comprehensive Error Handling & Resiliency: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Error handling and resiliency in jQuery AJAX refers to the strategies and techniques for detecting, categorising, and recovering from failures that occur during asynchronous requests, ensuring the application remains functional and provides meaningful feedback to users.

**Technical Definition**
jQuery AJAX error handling operates through the `.fail()` callback (or the `error` option), which receives three arguments: the `jqXHR` object, a `textStatus` string, and an `errorThrown` string. Error categories include HTTP errors (4xx client errors, 5xx server errors), network timeouts (`textStatus: "timeout"`), aborts (`textStatus: "abort"`), and parsing errors (`textStatus: "parsererror"`) . Resiliency strategies include retry mechanisms with exponential backoff and jitter, graceful UI degradation, and structured user feedback .

**Beginner-Friendly Explanation**
Things go wrong with network requests — servers go down, connections time out, or the data comes back in the wrong format. Good error handling means you catch these problems, tell the user what happened in a helpful way, and maybe try again automatically if it's a temporary glitch.

### Key Characteristics

- **Categorised Failures**: Errors are distinguished by type: HTTP status, timeout, abort, and parser error .
- **Promise-Based Handling**: `.fail()` provides a structured way to handle errors in the Promise chain.
- **Resiliency Patterns**: Retry logic with exponential backoff and jitter prevents overwhelming failing servers .
- **User-Centric Feedback**: Error messages should be actionable, descriptive, and accessible .

### Prerequisites

- Solid understanding of `$.ajax()` and its configuration options.
- Familiarity with Promise methods (`.done()`, `.fail()`, `.always()`).
- Basic knowledge of HTTP status codes and network concepts.

### Related Programming Areas

- **UI/UX Design**: Designing error states and user feedback.
- **Server-Side Development**: Understanding what causes 4xx and 5xx errors.
- **Network Programming**: Timeouts, retries, circuit breakers, and idempotency .

### Core Concepts / Features

1. Differentiating Error Categories
2. Graceful Degradation and User-Facing Error States
3. Basic Retry Strategies for Transient Failures

---

## Core Concept 1: Differentiating Error Categories

### Definitions

**Core Definition**
Error categorisation is the practice of distinguishing between different types of AJAX failures based on the `textStatus` string and HTTP status code, enabling appropriate responses to each type.

**Technical Definition**
The `.fail()` callback receives `(jqXHR, textStatus, errorThrown)`. The `textStatus` parameter distinguishes error types: `"timeout"` (request exceeded the `timeout` setting), `"error"` (HTTP error, including 4xx and 5xx), `"abort"` (request cancelled via `.abort()`), and `"parsererror"` (response could not be parsed according to `dataType`) . The `jqXHR.status` property provides the HTTP status code (e.g., 404, 500). The `jqXHR.statusText` property provides the HTTP status text (e.g., "Not Found") .

**Beginner-Friendly Explanation**
When a request fails, jQuery tells you *why* it failed. The `textStatus` says whether it was a timeout, an HTTP error, an abort, or a parsing problem. The `jqXHR.status` tells you the specific HTTP code (like 404 for "not found" or 500 for "server error").

### Purposes

- To distinguish between recoverable and non-recoverable failures.
- To provide appropriate user feedback for each error type.
- To implement retry logic only for transient failures (timeouts, 5xx errors).
- To avoid retrying errors that will always fail (4xx client errors).
- To debug issues by understanding the exact failure mode.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/data",
  type: "GET"
})
.fail(function(jqXHR, textStatus, errorThrown) {
  // textStatus: "timeout", "error", "abort", "parsererror"
  // jqXHR.status: HTTP status code
  // errorThrown: HTTP status text or exception
});
```

**Component Breakdown**

- `jqXHR` : The jqXHR object containing `status`, `statusText`, `responseText`.
- `textStatus` : String indicating the failure category.
- `errorThrown` : String containing the HTTP status text or exception message .

**Error Category Table**

| `textStatus` | Cause | `jqXHR.status` | Retry? |
|---|---|---|---|
| `"timeout"` | Request exceeded `timeout` setting | 0 | Yes (transient)  |
| `"error"` | HTTP error (4xx, 5xx) | 4xx/5xx | 5xx: Yes; 4xx: No  |
| `"abort"` | Request cancelled via `.abort()` | 0 | No |
| `"parsererror"` | Response failed to parse | 200 (usually) | No (data issue)  |

**Syntax Rules**

1. `textStatus` is `"success"` for successful requests, `"notmodified"` for 304 responses.
2. `jqXHR.status` is `0` for timeouts and aborts.
3. `parsererror` often occurs when `dataType` doesn't match the actual response format .
4. The `errorThrown` parameter may be empty for abort errors.
5. **4xx errors** indicate a client-side problem (e.g., bad request, unauthorized, not found) — the client must change something to succeed .
6. **5xx errors** indicate a server-side problem (e.g., internal error, service unavailable) — the same request may succeed later .

**Constraints and Limitations**

- **Status 0 Ambiguity**: Timeouts and aborts both return status 0; use `textStatus` to distinguish .
- **Parser Errors on Success**: Parser errors can occur even when the HTTP status is 200 .
- **Browser Variance**: Some error details vary across browsers.

### Multiple Annotated Complete Code Examples

**Example 1: Categorising Errors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Error Categories — Detection</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="testBtn">Test Error Handling</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#testBtn").on("click", function () {
        // Step 1: Request with intentional error scenario
        $.ajax({
          url: "/api/nonexistent",  // 404 error
          type: "GET",
          dataType: "json",
          timeout: 5000
        })
        .done(function (data) {
          $("#output").text("Success: " + data);
        })
        .fail(function (jqXHR, textStatus, errorThrown) {
          // Step 2: Categorise the error
          var category = "";
          var message = "";

          switch (textStatus) {
            case "timeout":
              category = "TIMEOUT";
              message = "The server took too long to respond.";
              break;
            case "error":
              if (jqXHR.status >= 500) {
                category = "SERVER ERROR";
                message = "The server encountered an error. Please try again.";
              } else if (jqXHR.status === 404) {
                category = "NOT FOUND";
                message = "The requested resource was not found.";
              } else if (jqXHR.status >= 400) {
                category = "CLIENT ERROR";
                message = "The request was invalid.";
              }
              break;
            case "abort":
              category = "ABORTED";
              message = "The request was cancelled.";
              break;
            case "parsererror":
              category = "PARSE ERROR";
              message = "The server returned an unexpected format.";
              break;
            default:
              category = "UNKNOWN";
              message = "An unexpected error occurred.";
          }

          // Step 3: Display categorised error
          $("#output").html(
            "<strong>" + category + "</strong><br>" +
            message + "<br>" +
            "<small>Status: " + jqXHR.status + " " + jqXHR.statusText + "</small>"
          );
          console.log("textStatus:", textStatus);
          console.log("HTTP Status:", jqXHR.status);
          console.log("errorThrown:", errorThrown);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking the button triggers a 404 error.
- Displays "NOT FOUND" with the message "The requested resource was not found."
- Console logs the textStatus, HTTP status, and errorThrown.

**Why This Output Occurs**
The `textStatus` is `"error"` because an HTTP error occurred. The `jqXHR.status` is `404`, which is categorised as a client error .

---

**Example 2: Distinguishing Timeout from HTTP Errors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Error Categories — Timeout vs HTTP</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="timeoutBtn">Trigger Timeout (1ms)</button>
  <button id="httpBtn">Trigger 404</button>
  <div id="output"></div>

  <script>
    $(function () {
      // Step 1: Timeout scenario
      $("#timeoutBtn").on("click", function () {
        $.ajax({
          url: "https://jsonplaceholder.typicode.com/posts/1",
          type: "GET",
          timeout: 1  // Extremely short timeout to force failure
        })
        .fail(function (jqXHR, textStatus) {
          $("#output").html(
            "<strong>Failure Type:</strong> " + textStatus + "<br>" +
            "<strong>HTTP Status:</strong> " + jqXHR.status
          );
        });
      });

      // Step 2: HTTP error scenario
      $("#httpBtn").on("click", function () {
        $.ajax({
          url: "https://jsonplaceholder.typicode.com/nonexistent",
          type: "GET"
        })
        .fail(function (jqXHR, textStatus) {
          $("#output").html(
            "<strong>Failure Type:</strong> " + textStatus + "<br>" +
            "<strong>HTTP Status:</strong> " + jqXHR.status
          );
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- **Timeout**: "Failure Type: timeout", "HTTP Status: 0".
- **HTTP Error**: "Failure Type: error", "HTTP Status: 404".

**Why This Output Occurs**
Timeouts set `textStatus` to `"timeout"` and `status` to `0`. HTTP errors set `textStatus` to `"error"` and `status` to the actual HTTP code .

### Real-World Cases

- **Form Submission**: Distinguishing validation errors (400) from server errors (500) .
- **Data Loading**: Retrying timeouts but not 404s.
- **Authentication**: Redirecting to login on 401 errors.
- **Debugging**: Logging error categories for troubleshooting.

### References

- Stack Overflow — textStatus values in jQuery AJAX – https://stackoverflow.com/revisions/1487258f-4ebe-4bff-b020-365cb5164917/view-source
- Stack Overflow — When are different jQuery Ajax error codes returned? – https://stackoverflow.com/questions/40106906/when-are-the-different-jquery-ajax-error-codes-returned/40107127
- Microsoft Learn — HTTP Status Code Overview – https://learn.microsoft.com/zh-tw/troubleshoot/developer/webapps/iis/health-diagnostic-performance/http-status-code

---

## Core Concept 2: Graceful Degradation and User-Facing Error States

### Definitions

**Core Definition**
Graceful degradation is the practice of ensuring that when an AJAX request fails, the application remains usable and the user receives clear, actionable feedback about the problem.

**Technical Definition**
Graceful degradation in AJAX involves: (1) detecting failures through `.fail()` handlers, (2) displaying user-facing error messages (alerts, inline messages, field-level errors), (3) providing recovery options (retry buttons, fallback content), and (4) ensuring non-critical failures do not break the entire UI. For forms, field-level validation errors can be returned from the server as JSON and mapped to specific form fields .

**Beginner-Friendly Explanation**
When something goes wrong, don't just leave the user staring at a broken page. Show them what happened, give them a way to try again, and make sure the rest of the page still works.

### Purposes

- To maintain application usability despite request failures.
- To provide clear, actionable error messages to users.
- To enable recovery through retry actions or fallback content.
- To prevent one failed request from breaking the entire UI.
- To map server-side validation errors to specific form fields.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({ url: "/api/data" })
.fail(function(jqXHR, textStatus) {
  // Display user-facing error
  showErrorState(textStatus, jqXHR.status);
  // Provide recovery option
  showRetryButton();
})
.always(function() {
  // Cleanup: hide loading indicators
  hideSpinner();
});
```

**Error Display Strategies**

| Strategy | Use Case | Implementation |
|---|---|---|
| Error Summary | Form validation | List errors at top with links to fields  |
| Inline Message | General errors | Insert error text near the form |
| Field-Level Message | Validation errors | Insert error next to specific field  |
| Skeleton/Placeholder | Data loading failures | Show placeholder content |
| Cached Data | Offline/fallback | Display last known data |

**Syntax Rules**

1. Error messages should be actionable, not vague (e.g., "Check your connection" vs. "Error occurred") .
2. Field-level errors require the server to return a structured response (e.g., JSON with field names as keys) .
3. Always clean up loading states in `.always()` regardless of outcome.
4. Provide retry options for transient failures.
5. **Accessibility**: Text descriptions of errors are required, not just red borders or asterisks .

**Constraints and Limitations**

- **Vague Messages**: Generic messages frustrate users; be specific .
- **No Recovery**: Without retry options, users are stuck.
- **UI Breakage**: Unhandled errors can leave the UI in an inconsistent state.

### Multiple Annotated Complete Code Examples

**Example 1: User-Friendly Error Display**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Graceful Degradation — Error UI</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .error-box { padding: 15px; background: #fee; border: 1px solid #f00; margin: 10px 0; }
    .spinner { display: none; }
    #retryBtn { display: none; }
  </style>
</head>
<body>
  <button id="loadBtn">Load Data</button>
  <div id="spinner" class="spinner">Loading...</div>
  <div id="errorBox" class="error-box" style="display:none;">
    <p id="errorMessage"></p>
    <button id="retryBtn">Try Again</button>
  </div>
  <div id="content"></div>

  <script>
    $(function () {
      function loadData() {
        // Step 1: Show loading, hide previous errors
        $("#spinner").show();
        $("#errorBox").hide();
        $("#content").empty();

        $.ajax({
          url: "/api/data",
          type: "GET",
          dataType: "json"
        })
        .done(function (data) {
          // Step 2: Render data on success
          $("#content").text("Data loaded: " + JSON.stringify(data));
        })
        .fail(function (jqXHR, textStatus, errorThrown) {
          // Step 3: Display user-friendly error
          var message = "";

          if (textStatus === "timeout") {
            message = "The request took too long. Please check your connection and try again.";
          } else if (jqXHR.status >= 500) {
            message = "The server is having trouble. Please try again in a moment.";
          } else if (jqXHR.status === 404) {
            message = "The requested data could not be found.";
          } else {
            message = "Unable to load data. Please try again.";
          }

          $("#errorMessage").text(message);
          $("#errorBox").show();
          $("#retryBtn").show();
        })
        .always(function () {
          // Step 4: Always hide spinner
          $("#spinner").hide();
        });
      }

      // Step 5: Wire up retry
      $("#loadBtn").on("click", loadData);
      $("#retryBtn").on("click", loadData);
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Load Data" shows a spinner briefly.
- On failure, an error box appears with a specific message and a "Try Again" button.
- Clicking "Try Again" re-runs the request.

**Why This Output Occurs**
The `.fail()` handler categorises the error and displays an appropriate message. `.always()` ensures the spinner is hidden regardless of outcome .

---

**Example 2: Field-Level Validation Errors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Graceful Degradation — Field Errors</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .field-error { color: red; font-size: 0.9em; display: block; margin-top: 4px; }
    .input-error { border: 2px solid red; }
  </style>
</head>
<body>
  <form id="signupForm">
    <div>
      <input type="text" name="username" placeholder="Username">
      <span class="field-error" data-field="username"></span>
    </div>
    <div>
      <input type="email" name="email" placeholder="Email">
      <span class="field-error" data-field="email"></span>
    </div>
    <button type="submit">Sign Up</button>
  </form>

  <script>
    $(function () {
      $("#signupForm").on("submit", function (event) {
        event.preventDefault();

        // Step 1: Clear previous errors
        $(".field-error").text("");
        $("input").removeClass("input-error");

        $.ajax({
          url: "/api/signup",
          type: "POST",
          data: $(this).serialize(),
          dataType: "json"
        })
        .done(function (response) {
          // Success: redirect or show success
          alert("Signup successful!");
        })
        .fail(function (jqXHR) {
          // Step 2: Handle field-level errors from server
          if (jqXHR.status === 422 && jqXHR.responseJSON) {
            // Server returns { errors: { username: "Taken", email: "Invalid" } }
            var errors = jqXHR.responseJSON.errors || jqXHR.responseJSON;

            $.each(errors, function (field, message) {
              // Step 3: Find the field and its error span
              var $field = $("input[name='" + field + "']");
              var $errorSpan = $(".field-error[data-field='" + field + "']");

              if ($field.length) {
                $field.addClass("input-error");
              }
              if ($errorSpan.length) {
                $errorSpan.text(message);
              }
            });
          } else {
            // General error
            alert("An error occurred. Please try again.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Submitting with duplicate username and invalid email displays field-specific errors.
- The username and email inputs receive red borders.
- Error messages appear next to the respective fields.

**Why This Output Occurs**
The server returns a structured JSON response with field-level errors. The client maps each error to the corresponding field using `name` attributes and `data-field` selectors .

### Real-World Cases

- **Form Validation**: Displaying server-side validation errors next to fields .
- **Data Dashboards**: Showing fallback content when API calls fail.
- **E-commerce**: Displaying cached cart data when the server is unreachable.
- **Progressive Web Apps**: Showing offline states with retry options.

### References

- W3C — Technique G84: Providing a text description – https://www.w3.org/WAI/WCAG22/Techniques/general/G84
- W3C — Technique SCR32: Client-side validation and error text via DOM – https://w3c.github.io/wcag/techniques/client-side-script/SCR32
- Scottish Government Design System — Error summary – https://designsystem.gov.scot/components/error-summary

---

## Core Concept 3: Basic Retry Strategies for Transient Network Failures

### Definitions

**Core Definition**
Retry strategies are mechanisms that automatically re-attempt failed AJAX requests, typically using exponential backoff and jitter to avoid overwhelming the server.

**Technical Definition**
A retry strategy wraps the AJAX request in a function that calls itself recursively on failure, decrementing a retry counter and increasing the delay between attempts. Exponential backoff doubles the delay with each retry, while jitter adds random variation to prevent synchronized retry storms . Retries should only be applied to transient failures (timeouts, 5xx errors, 429 rate limits), not to client errors (4xx) or parser errors .

**Beginner-Friendly Explanation**
If a request fails because the network hiccupped, you can try again automatically. But you shouldn't try forever — set a limit and wait longer between each attempt so you don't overwhelm the server.

### Purposes

- To recover from transient network failures automatically.
- To improve user experience by reducing visible errors.
- To implement exponential backoff to avoid overwhelming servers .
- To add jitter to prevent retry storms .
- To distinguish between retryable and non-retryable failures.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
function ajaxWithRetry(url, options, retries, baseDelay) {
  return $.ajax(url, options).fail(function(jqXHR, textStatus) {
    var shouldRetry = textStatus === "timeout" || jqXHR.status >= 500;
    if (shouldRetry && retries > 0) {
      var delay = baseDelay * Math.pow(2, maxRetries - retries);
      var jitter = Math.random() * 250;
      setTimeout(function() {
        ajaxWithRetry(url, options, retries - 1, baseDelay);
      }, delay + jitter);
    }
  });
}
```

**Component Breakdown**

- `retries` : Maximum number of retry attempts.
- `baseDelay` : Initial delay in milliseconds.
- `delay * Math.pow(2, n)` : Exponential backoff (doubles each retry) .
- `jitter` : Random delay added to prevent synchronized retries .
- `shouldRetry` : Condition determining if the error is retryable.

**Retry Decision Matrix**

| Error Type | Retry? | Reason |
|---|---|---|
| `timeout` | Yes | Transient network issue  |
| 5xx (Server Error) | Yes | Server may recover  |
| 429 (Rate Limit) | Yes | Temporary throttling  |
| 4xx (Client Error) | No | Request will fail again  |
| `parsererror` | No | Data format issue |
| `abort` | No | User cancelled |

**Syntax Rules**

1. Always limit retries (e.g., 3 attempts) .
2. Use exponential backoff (e.g., 1s, 2s, 4s) .
3. Add jitter to prevent synchronized retries .
4. Only retry transient failures .
5. Ensure the Promise is only rejected after all retries are exhausted.
6. **Idempotency**: Only retry operations that are safe to repeat (GET, PUT, DELETE) .

**Constraints and Limitations**

- **No Infinite Retries**: Always set a maximum .
- **Jitter**: Without jitter, many clients may retry simultaneously, causing retry storms .
- **Promise Resolution**: The wrapped function should resolve on success and reject only after all retries fail.
- **Non-Idempotent Operations**: Retrying POST may create duplicate resources .

### Multiple Annotated Complete Code Examples

**Example 1: Basic Retry with Exponential Backoff**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Retry — Exponential Backoff</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="loadBtn">Load with Retry</button>
  <div id="output"></div>

  <script>
    $(function () {
      function ajaxWithRetry(url, retries, delay) {
        return $.ajax({
          url: url,
          type: "GET",
          dataType: "json",
          timeout: 5000
        })
        .fail(function (jqXHR, textStatus) {
          // Step 1: Determine if retryable
          var shouldRetry = textStatus === "timeout" || jqXHR.status >= 500;

          if (shouldRetry && retries > 0) {
            console.log("Retrying... Attempts left:", retries - 1, "Delay:", delay + "ms");

            // Step 2: Retry after delay
            return new Promise(function (resolve, reject) {
              setTimeout(function () {
                ajaxWithRetry(url, retries - 1, delay * 2)
                  .done(resolve)
                  .fail(reject);
              }, delay);
            });
          } else {
            console.log("No retry. Reason:", textStatus);
            return Promise.reject(jqXHR);
          }
        });
      }

      // Step 3: Use the retry wrapper
      $("#loadBtn").on("click", function () {
        $("#output").text("Loading with retries...");

        ajaxWithRetry(
          "https://jsonplaceholder.typicode.com/posts/1",
          3,    // max retries
          1000  // initial delay
        )
        .done(function (data) {
          $("#output").text("Loaded: " + data.title);
        })
        .fail(function () {
          $("#output").text("Failed after all retries.");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- On success, displays the post title.
- On transient failure, retries up to 3 times with increasing delays.
- On permanent failure, displays "Failed after all retries."

**Why This Output Occurs**
The recursive wrapper retries on transient errors, doubling the delay each time. It rejects the Promise only after all retries are exhausted .

---

**Example 2: Retry with Jitter**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Retry — With Jitter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="loadBtn">Load with Jittered Retry</button>
  <div id="output"></div>

  <script>
    $(function () {
      function ajaxWithJitteredRetry(url, retries, baseDelay) {
        return $.ajax({
          url: url,
          type: "GET",
          dataType: "json"
        })
        .fail(function (jqXHR, textStatus) {
          var shouldRetry = textStatus === "timeout" || jqXHR.status >= 500;

          if (shouldRetry && retries > 0) {
            // Step 1: Calculate delay with jitter
            var exponentialDelay = baseDelay * Math.pow(2, 3 - retries);
            var jitter = Math.random() * 250;  // 0-250ms random jitter
            var totalDelay = exponentialDelay + jitter;

            console.log("Retry in " + Math.round(totalDelay) + "ms (base: " + exponentialDelay + "ms, jitter: " + Math.round(jitter) + "ms)");

            // Step 2: Retry
            return new Promise(function (resolve, reject) {
              setTimeout(function () {
                ajaxWithJitteredRetry(url, retries - 1, baseDelay)
                  .done(resolve)
                  .fail(reject);
              }, totalDelay);
            });
          }

          return Promise.reject(jqXHR);
        });
      }

      $("#loadBtn").on("click", function () {
        ajaxWithJitteredRetry(
          "https://jsonplaceholder.typicode.com/posts/1",
          3,    // max retries
          500   // base delay
        )
        .done(function (data) {
          $("#output").text("Loaded: " + data.title);
        })
        .fail(function () {
          $("#output").text("Failed after retries.");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Each retry logs the calculated delay with jitter.
- Delays increase exponentially (500ms, 1000ms, 2000ms) plus random jitter.

**Why This Output Occurs**
The jitter adds randomness to prevent multiple clients from retrying simultaneously, reducing server load .

### Real-World Cases

- **API Rate Limiting**: Retrying after rate limit errors (429) with appropriate delays .
- **Mobile Networks**: Handling intermittent connectivity on mobile devices .
- **Server Maintenance**: Retrying during brief server restarts.
- **Load Balancing**: Retrying when a specific server node is temporarily unavailable.

### References

- Microsoft Azure Well-Architected Framework — 有关处理暂时性故障的建议 – https://learn.microsoft.com/zh-cn/Azure/well-architected/design-guides/handle-transient-faults
- AWS Well-Architected Framework — 可靠性支柱 (实施重试) – https://docs.aws.amazon.com/zh_cn/well-architected/latest/reliability-pillar/wellarchitected-reliability-pillar.pdf
- Microsoft Learn — Build MCP Servers with Error Handling and Fallback – https://learn.microsoft.com/en-us/training/modules/aaai-build-enterprise-tool-ecosystems-mcp-azure/3-build-mcp-servers-error-handle-fallback
- Go Packages — retryfs (Error Classification) – https://pkg.go.dev/github.com/absfs/retryfs

---

## Summary Comparison Table

| Concept | Key Detection | Key Strategy | Key Limitation |
|---|---|---|---|
| **Error Categories** | `textStatus`, `jqXHR.status` | Retry transient, skip permanent  | Status 0 ambiguity |
| **Graceful Degradation** | `.fail()`, `.always()` | User-friendly messages, retry UI  | Vague messages frustrate users |
| **Retry Strategies** | `textStatus === "timeout"` or `status >= 500` | Exponential backoff + jitter  | Always limit retries |

---

## Important Notes on Version-Specific Behaviour and Limitations

1. **jQuery 1.5+**: `.fail()` and `.always()` available.
2. **jQuery 3.0**: `jqXHR.error()` and `jqXHR.complete()` removed; use `.fail()` and `.always()`.
3. **Timeout Status**: `jqXHR.status` is `0` for timeouts and aborts .
4. **Parser Errors**: Occur when response doesn't match `dataType`; often status 200 .
5. **Retry Budget**: Apply per-request, not per-endpoint, to avoid one slow API consuming retry capacity .
6. **Idempotency**: Only retry operations that are safe to repeat; POST retries may create duplicates .
7. **Jitter**: Always add jitter to prevent synchronized retry storms .

---

## References

- Microsoft Azure Well-Architected Framework — 有关处理暂时性故障的建议 – https://learn.microsoft.com/zh-cn/Azure/well-architected/design-guides/handle-transient-faults
- AWS Well-Architected Framework — 可靠性支柱 (实施重试) – https://docs.aws.amazon.com/zh_cn/well-architected/latest/reliability-pillar/wellarchitected-reliability-pillar.pdf
- Microsoft Learn — Build MCP Servers with Error Handling and Fallback – https://learn.microsoft.com/en-us/training/modules/aaai-build-enterprise-tool-ecosystems-mcp-azure/3-build-mcp-servers-error-handle-fallback
- W3C — Technique G84: Providing a text description – https://www.w3.org/WAI/WCAG22/Techniques/general/G84
- W3C — Technique SCR32: Client-side validation and error text via DOM – https://w3c.github.io/wcag/techniques/client-side-script/SCR32
- Scottish Government Design System — Error summary – https://designsystem.gov.scot/components/error-summary
- Stack Overflow — When are different jQuery Ajax error codes returned? – https://stackoverflow.com/questions/40106906/when-are-the-different-jquery-ajax-error-codes-returned/40107127
- Stack Overflow — textStatus values in jQuery AJAX – https://stackoverflow.com/revisions/1487258f-4ebe-4bff-b020-365cb5164917/view-source
- Microsoft Learn — HTTP Status Code Overview – https://learn.microsoft.com/zh-tw/troubleshoot/developer/webapps/iis/health-diagnostic-performance/http-status-code
- Go Packages — retryfs (Error Classification) – https://pkg.go.dev/github.com/absfs/retryfs