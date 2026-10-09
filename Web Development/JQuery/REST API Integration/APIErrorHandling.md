# API Error Handling with jQuery — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API Error Handling with jQuery is the practice of detecting, interpreting, and responding to HTTP error responses returned by API endpoints, ensuring that users receive meaningful feedback and that the application recovers gracefully from failures.

**Technical Definition:** API Error Handling encompasses the interception and processing of HTTP error responses (4xx and 5xx status codes) in jQuery AJAX calls. Each status class carries distinct semantics: 4xx indicates a client-side error (bad request, missing credentials, insufficient permissions, non-existent resource, validation failure); 5xx indicates a server-side failure. jQuery routes error responses to the `.fail()` callback or the `error` option, providing a `jqXHR` object with `.status`, `.statusText`, `.responseJSON`, and `.responseText` properties. Global error handling can be configured via `$(document).ajaxError()`, and resilience can be improved through explicit timeouts and retry logic with exponential backoff.

**Beginner-Friendly Explanation:** When you ask a server for something, sometimes it says "no" — and it tells you why with a number. A 400 means "your request was malformed." A 401 means "you are not logged in." A 403 means "you are logged in but not allowed." A 404 means "that thing does not exist." A 422 means "your data has errors." A 500 means "the server broke." This cheat sheet covers how to read those numbers and respond appropriately, so users see helpful messages instead of silent failures or cryptic errors.

### Key Characteristics

- **Status-code-driven:** Each HTTP status code has a specific meaning and requires a specific response.
- **Fail-callback routing:** jQuery routes errors to `.fail()` or the `error` option, never to `.done()`.
- **Structured error payloads:** Modern APIs return JSON error bodies with `error`, `message`, or `errors` fields.
- **User-facing vs. developer-facing:** Some errors should be shown to users; others should be logged for developers.
- **Recoverable vs. terminal:** Some errors can be retried (500, timeout); others cannot (400, 422).
- **Global vs. local:** Errors can be handled per-request or globally via `$(document).ajaxError()`.

### Prerequisites

- Proficiency in jQuery AJAX: `$.ajax()`, `.done()`, `.fail()`, and `.always()`.
- Understanding of HTTP status codes and their classes (2xx, 4xx, 5xx).
- Familiarity with JSON response structures and validation error payloads.
- Awareness of authentication flows (tokens, sessions) and retry strategies.

### Related Programming Areas

- **REST Fundamentals:** Status codes, methods, and idempotency.
- **Authentication and Authorization:** 401 and 403 handling.
- **Validation:** Parsing 422 responses and displaying field errors.
- **Resilience Engineering:** Timeouts, retries, and circuit breakers.
- **Observability:** Logging errors and reporting to monitoring services.

### Core Concepts / Features

This cheat sheet covers eight core concepts: 400, 401, 403, 404, 422, 500 errors, global error catching, and network timeout with retry logic.

---

## Core Concept 1: 400 Errors — Handling Bad Request Structure or Invalid Client Payloads

### Definitions

**Core Definition:** A 400 (Bad Request) error indicates that the server cannot process the request because the client sent malformed syntax, invalid parameters, or an unparseable payload.

**Technical Definition:** The 400 status code is a generic client-error response indicating that the request is syntactically incorrect or violates the server's expectations. Common causes include malformed JSON (missing braces, trailing commas), missing required headers (`Content-Type`, `Authorization`), invalid query parameter types, or a request body that does not match the expected schema. The server may return a JSON error body with details about what was wrong. Unlike 422 (which is semantic validation), 400 is about structural or syntactic correctness.

**Beginner-Friendly Explanation:** A 400 error means "I could not understand your request." It is like mailing a letter with no address — the post office cannot deliver it. The client sent something the server could not parse or accept.

### Purposes

- To distinguish structural errors from validation errors (422).
- To alert developers to malformed requests during development.
- To provide a generic error response when the request cannot be processed.
- To avoid revealing internal details about why parsing failed.
- To prompt the client to fix the request structure before retrying.

### Syntax Rules and Structure

**Complete General Syntax (Client Handling):**
```javascript
$.ajax({
    url: "/api/users",
    type: "POST",
    contentType: "application/json",
    data: JSON.stringify(payload),
    success: function(response) { ... },
    error: function(jqXHR) {
        if (jqXHR.status === 400) {
            // Malformed request — log for developers
            console.error("Bad Request:", jqXHR.responseJSON);
            // Show generic message to user
            $("#error").text("There was a problem with your request. Please try again.");
        }
    }
});
```

**Common Causes of 400 Errors:**

| Cause | Example | Fix |
|-------|---------|-----|
| Malformed JSON | Missing closing brace | Validate JSON before sending |
| Missing Content-Type | POST without `contentType: "application/json"` | Set the header |
| Invalid query parameter | `?page=abc` when page expects integer | Validate types |
| Empty request body | POST with no data | Include required payload |
| Invalid encoding | Non-UTF-8 characters | Ensure UTF-8 encoding |

**Syntax Rules:**

- Check `jqXHR.status === 400` in the error callback.
- Inspect `jqXHR.responseJSON` for server-provided details.
- Do not retry automatically; 400 is not a transient error.
- Log the raw response for debugging during development.
- Display a generic message to users; avoid exposing internal details.

**Constraints and Limitations:**

- Some servers use 400 for validation errors instead of 422; consult the API documentation.
- The error body format varies by API; always check the documentation.
- 400 errors during development usually indicate a bug in the client code.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Handling a 400 Bad Request**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>400 Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="sendBad">Send Bad Request</button>
  <p id="message"></p>

  <script>
    $(function() {
      $("#sendBad").click(function() {
        // Deliberately send malformed data
        $.ajax({
          url: "https://httpbin.org/post",
          type: "POST",
          contentType: "application/json",
          data: "{malformed json", // Invalid JSON
          dataType: "json",
          error: function(jqXHR) {
            if (jqXHR.status === 400) {
              $("#message").text(
                "Bad Request (400): The server could not parse your request. " +
                "Please check the data and try again."
              );
              console.error("Response:", jqXHR.responseText);
            } else {
              $("#message").text("Error: " + jqXHR.status);
            }
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Send Bad Request" displays "Bad Request (400): The server could not parse your request. Please check the data and try again." The Console logs the server's response.

**Why this output:** The malformed JSON string causes the server to return a 400 status. The error callback checks `jqXHR.status === 400` and displays a user-friendly message while logging the details for developers.

### Real-World Cases

- **API development:** Debugging malformed requests during integration.
- **Form submissions:** Handling cases where the client sends unexpected data.
- **Third-party APIs:** Adapting to strict schema requirements.
- **Mobile clients:** Handling version mismatches between client and server.

---

## Core Concept 2: 401 Errors — Handling Expired or Invalid Authentication Credentials

### Definitions

**Core Definition:** A 401 (Unauthorized) error indicates that the request lacks valid authentication credentials or that the provided credentials are invalid or expired.

**Technical Definition:** The 401 status code is returned when the server requires authentication and the request either lacks credentials or provides credentials that fail verification. The response typically includes a `WWW-Authenticate` header describing the authentication scheme. In token-based authentication (JWT), 401 is returned when the token is missing, malformed, expired, or signed with the wrong key. The client's correct response is to prompt the user to log in again, clear the invalid token, and redirect to the login page or show a login modal.

**Beginner-Friendly Explanation:** A 401 error means "I do not know who you are." You either forgot to show your ID, or your ID has expired. The fix is to log in again and get a fresh ID (token).

### Purposes

- To signal that authentication is required or has failed.
- To prompt the client to clear invalid credentials and re-authenticate.
- To protect sensitive endpoints from unauthenticated access.
- To distinguish authentication failures (401) from authorization failures (403).
- To support session expiration handling.

### Syntax Rules and Structure

**Complete General Syntax (Client Handling):**
```javascript
$.ajax({
    url: "/api/profile",
    type: "GET",
    headers: { "Authorization": "Bearer " + token },
    success: function(response) { ... },
    error: function(jqXHR) {
        if (jqXHR.status === 401) {
            // Clear the invalid token
            localStorage.removeItem("authToken");
            // Redirect to login or show login modal
            window.location.href = "/login?redirect=" + encodeURIComponent(window.location.pathname);
        }
    }
});
```

**Complete General Syntax (Global 401 Handler):**
```javascript
$(document).ajaxError(function(event, jqXHR, settings, errorThrown) {
    if (jqXHR.status === 401) {
        localStorage.removeItem("authToken");
        window.location.href = "/login";
    }
});
```

| Scenario | 401 Response | Client Action |
|----------|-------------|---------------|
| No token | 401 + `WWW-Authenticate` | Redirect to login |
| Expired token | 401 | Clear token, redirect to login |
| Invalid signature | 401 | Clear token, redirect to login |
| Wrong scheme | 401 | Correct the `Authorization` header |

**Syntax Rules:**

- Check `jqXHR.status === 401` in the error callback.
- Clear the invalid token from storage.
- Redirect to the login page, preserving the intended destination.
- Avoid infinite redirect loops by excluding the login endpoint from the 401 handler.
- Consider using refresh tokens to silently renew expired access tokens.

**Constraints and Limitations:**

- The `Authorization` header may be stripped by proxies; check `REDIRECT_HTTP_AUTHORIZATION` on the server.
- Cross-origin 401 responses may be blocked by CORS unless `Access-Control-Allow-Credentials` is set.
- Refresh token flows require careful handling to avoid race conditions.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Handling an Expired Token**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>401 Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="fetchProtected">Fetch Protected Data</button>
  <p id="message"></p>

  <script>
    $(function() {
      // Simulate an expired token
      localStorage.setItem("authToken", "expired-token-123");

      $("#fetchProtected").click(function() {
        $.ajax({
          url: "https://httpbin.org/bearer",
          type: "GET",
          headers: { "Authorization": "Bearer " + localStorage.getItem("authToken") },
          error: function(jqXHR) {
            if (jqXHR.status === 401) {
              // Clear the invalid token
              localStorage.removeItem("authToken");
              $("#message").text(
                "Your session has expired. Please log in again."
              );
              // In a real app: window.location.href = "/login";
            } else {
              $("#message").text("Error: " + jqXHR.status);
            }
          },
          success: function() {
            $("#message").text("Data loaded successfully.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Fetch Protected Data" sends a request with an invalid token. The server returns 401. The message displays "Your session has expired. Please log in again," and the token is cleared from `localStorage`.

**Why this output:** The invalid token causes the server to reject the request with 401. The error callback clears the token and displays a re-authentication message, simulating the redirect to a login page.

### Real-World Cases

- **SPA authentication:** Handling token expiration and redirecting to login.
- **Mobile apps:** Prompting re-login when the access token expires.
- **Multi-tab applications:** Synchronizing logout across tabs.
- **API clients:** Implementing refresh token flows.

---

## Core Concept 3: 403 Errors — Handling Unauthorized Access Permissions to Resources

### Definitions

**Core Definition:** A 403 (Forbidden) error indicates that the server understood the request and identified the client, but the client does not have permission to access the resource.

**Technical Definition:** The 403 status code differs from 401 in that authentication has succeeded — the client is known — but authorization has failed. The user is logged in but lacks the required role, permission, or ownership to perform the requested action. Unlike 401, providing different credentials will not help; the user simply does not have access. The correct client response is to inform the user that they do not have permission, not to redirect to login.

**Beginner-Friendly Explanation:** A 403 error means "I know who you are, but you are not allowed in here." You are logged in, but you do not have the right permissions. Logging in again will not help — you need someone to grant you access.

### Purposes

- To distinguish authorization failures (403) from authentication failures (401).
- To inform users that they lack the required permissions.
- To prevent unauthorized access to sensitive resources.
- To avoid revealing whether a resource exists when the user is not authorized.
- To guide users to request access from an administrator.

### Syntax Rules and Structure

**Complete General Syntax (Client Handling):**
```javascript
$.ajax({
    url: "/api/admin/users",
    type: "GET",
    success: function(response) { ... },
    error: function(jqXHR) {
        if (jqXHR.status === 403) {
            $("#message").text(
                "You do not have permission to access this resource. " +
                "Contact your administrator if you believe this is an error."
            );
        }
    }
});
```

**401 vs. 403 Comparison:**

| Aspect | 401 Unauthorized | 403 Forbidden |
|--------|------------------|---------------|
| Meaning | Not authenticated | Authenticated but not authorized |
| Credentials | Missing or invalid | Valid |
| Retry with login | Yes, may help | No, will not help |
| Client action | Redirect to login | Show permission error |
| Example | Expired token | Regular user accessing admin panel |

**Syntax Rules:**

- Check `jqXHR.status === 403` in the error callback.
- Do not redirect to login; the user is already authenticated.
- Display a clear message that the user lacks permission.
- Log the attempt for security monitoring.
- Consider hiding UI elements the user cannot access to prevent 403s.

**Constraints and Limitations:**

- Some APIs return 404 instead of 403 to hide the existence of resources from unauthorized users.
- 403 errors on cross-origin requests may appear as CORS errors.
- Authorization logic should be enforced server-side regardless of client-side UI hiding.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Handling a 403 Forbidden**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>403 Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="accessAdmin">Access Admin Panel</button>
  <p id="message"></p>

  <script>
    $(function() {
      $("#accessAdmin").click(function() {
        // Simulate a request to an admin-only endpoint
        $.ajax({
          url: "https://httpbin.org/status/403",
          type: "GET",
          error: function(jqXHR) {
            if (jqXHR.status === 403) {
              $("#message").html(
                "<strong>Access Denied (403):</strong> You do not have permission " +
                "to access this resource. Please contact your administrator."
              );
            } else {
              $("#message").text("Error: " + jqXHR.status);
            }
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Access Admin Panel" sends a request that returns 403. The message displays "Access Denied (403): You do not have permission to access this resource. Please contact your administrator."

**Why this output:** The 403 status indicates the user is authenticated but lacks permission. The error callback displays a permission-specific message rather than redirecting to login.

### Real-World Cases

- **Admin panels:** Regular users attempting to access admin-only features.
- **Multi-tenant applications:** Users accessing another tenant's data.
- **Role-based access control:** Users without the required role.
- **Document sharing:** Users accessing documents not shared with them.

---

## Core Concept 4: 404 Errors — Gracefully Notifying Users When a Resource Does Not Exist

### Definitions

**Core Definition:** A 404 (Not Found) error indicates that the server could not find the requested resource at the specified URI.

**Technical Definition:** The 404 status code is returned when the server cannot map the request URI to an existing resource. This can occur when the resource was deleted, the URI is misspelled, the resource never existed, or the server is configured to hide the resource's existence from unauthorized users (returning 404 instead of 403). The client should inform the user that the resource was not found and offer alternatives (e.g., return to the list, search, or contact support).

**Beginner-Friendly Explanation:** A 404 error means "I could not find what you asked for." It is like looking for a book in the library and finding an empty shelf. The book might have been removed, or you might have the wrong call number.

### Purposes

- To inform users that a requested resource does not exist.
- To provide a graceful fallback (redirect, message, or search).
- To distinguish "not found" from "not allowed" (403).
- To support deep linking with error recovery.
- To hide the existence of resources from unauthorized users (404 instead of 403).

### Syntax Rules and Structure

**Complete General Syntax (Client Handling):**
```javascript
$.ajax({
    url: "/api/users/999",
    type: "GET",
    success: function(response) { ... },
    error: function(jqXHR) {
        if (jqXHR.status === 404) {
            $("#detail").html(
                "<p>The requested user was not found.</p>" +
                "<a href='/users'>Back to user list</a>"
            );
        }
    }
});
```

**404 Scenarios:**

| Scenario | Meaning | Client Action |
|----------|---------|---------------|
| Resource deleted | It existed but is gone | Show "not found" message |
| URI misspelled | Wrong ID or path | Verify the URI |
| Never existed | Invalid ID | Show "not found" message |
| Hidden for security | User not authorized | Show "not found" (same as 403 but concealed) |

**Syntax Rules:**

- Check `jqXHR.status === 404` in the error callback.
- Display a user-friendly "not found" message.
- Offer navigation alternatives (back to list, search, home).
- Do not retry automatically; 404 is not transient.
- Distinguish between "deleted" and "never existed" if the API provides that detail.

**Constraints and Limitations:**

- 404 can be used to mask 403 for security reasons; treat it as "not available" rather than "not authorized."
- Some SPAs return 200 for all requests and handle 404 client-side; ensure the API returns proper status codes.
- Custom 404 pages should maintain the application's navigation and branding.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Graceful 404 Handling**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>404 Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <input type="text" id="userId" placeholder="User ID">
  <button id="loadUser">Load User</button>
  <div id="detail"></div>

  <script>
    $(function() {
      $("#loadUser").click(function() {
        var id = $("#userId").val();
        if (!id) {
          $("#detail").text("Please enter a user ID.");
          return;
        }

        $.ajax({
          url: "https://jsonplaceholder.typicode.com/users/" + id,
          type: "GET",
          dataType: "json",
          success: function(user) {
            $("#detail").html(
              "<h3>" + user.name + "</h3>" +
              "<p>Email: " + user.email + "</p>"
            );
          },
          error: function(jqXHR) {
            if (jqXHR.status === 404) {
              $("#detail").html(
                "<p>User with ID <strong>" + id + "</strong> was not found.</p>" +
                "<p>Please check the ID and try again.</p>"
              );
            } else {
              $("#detail").text("Error: " + jqXHR.status);
            }
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Entering an invalid user ID and clicking "Load User" displays "User with ID [id] was not found. Please check the ID and try again." Entering a valid ID displays the user's details.

**Why this output:** The 404 response indicates the resource does not exist. The error callback displays a clear message with the attempted ID and a suggestion to retry.

### Real-World Cases

- **Product pages:** Products that have been removed from the catalog.
- **User profiles:** Deleted or non-existent user profiles.
- **Deep links:** Old bookmarks pointing to removed content.
- **API integrations:** Handling invalid IDs from external systems.

---

## Core Concept 5: 422 Errors — Parsing Validation Payload Arrays to Display Fields with Errors

### Definitions

**Core Definition:** A 422 (Unprocessable Entity) error indicates that the server understood the request and the payload is syntactically correct, but the data fails semantic validation (e.g., an email is malformed, a required field is missing, a value is out of range).

**Technical Definition:** The 422 status code is defined in RFC 4918 (WebDAV) and is widely used by frameworks like Laravel, Rails, and ASP.NET Core for validation errors. The response body typically contains a structured JSON object mapping field names to arrays of error messages. The client parses this payload and displays each error message next to the corresponding form field, highlighting the field with an error class. Unlike 400 (structural error), 422 indicates the request was well-formed but semantically invalid.

**Beginner-Friendly Explanation:** A 422 error means "I understood your request, but the data is not valid." For example, you submitted a form with an invalid email address. The server tells you exactly which fields have problems and what is wrong with them, so you can fix them and try again.

### Purposes

- To provide field-specific validation errors to the client.
- To display errors inline next to the offending form fields.
- To preserve user input while highlighting what needs correction.
- To support complex validation rules (unique emails, date ranges, conditional requirements).
- To distinguish semantic validation (422) from structural errors (400).

### Syntax Rules and Structure

**Complete General Syntax (Typical 422 Response):**
```json
{
    "message": "The given data was invalid.",
    "errors": {
        "email": [
            "The email field is required.",
            "The email must be a valid email address."
        ],
        "password": [
            "The password must be at least 8 characters."
        ]
    }
}
```

**Complete General Syntax (Client Parsing):**
```javascript
$.ajax({
    url: "/api/register",
    type: "POST",
    contentType: "application/json",
    data: JSON.stringify(formData),
    error: function(jqXHR) {
        if (jqXHR.status === 422) {
            var errors = jqXHR.responseJSON.errors;
            // Clear previous errors
            $(".is-invalid").removeClass("is-invalid");
            $(".error-message").text("");
            // Display new errors
            $.each(errors, function(field, messages) {
                $("[name='" + field + "']").addClass("is-invalid");
                $("[data-error-for='" + field + "']").text(messages[0]);
            });
        }
    }
});
```

| Component | Description |
|-----------|-------------|
| `jqXHR.status` | 422 for validation errors. |
| `jqXHR.responseJSON.errors` | Object mapping field names to error arrays. |
| `errors[field][0]` | The first error message for a field. |
| `[data-error-for]` | Custom attribute for error message containers. |

**Syntax Rules:**

- Check `jqXHR.status === 422` before parsing `responseJSON.errors`.
- Clear previous errors before displaying new ones.
- Iterate over the `errors` object and display each field's first (or all) messages.
- Highlight invalid fields with a CSS class (e.g., `is-invalid`).
- Preserve user input so they can correct the errors without retyping.

**Constraints and Limitations:**

- The error payload structure varies by framework; always check the API documentation.
- Some APIs return 400 for validation errors; handle both cases if necessary.
- Displaying all error messages for a field can clutter the UI; the first message is usually sufficient.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Displaying Validation Errors from a 422 Response**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>422 Error Demo</title>
  <style>
    .is-invalid { border-color: red; }
    .error-message { color: red; font-size: 12px; display: block; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <form id="registerForm">
    <div>
      <input type="text" name="name" placeholder="Name">
      <span class="error-message" data-error-for="name"></span>
    </div>
    <div>
      <input type="email" name="email" placeholder="Email">
      <span class="error-message" data-error-for="email"></span>
    </div>
    <div>
      <input type="password" name="password" placeholder="Password">
      <span class="error-message" data-error-for="password"></span>
    </div>
    <button type="submit">Register</button>
  </form>
  <p id="successMessage"></p>

  <script>
    $(function() {
      $("#registerForm").on("submit", function(e) {
        e.preventDefault();
        var $form = $(this);

        // Clear previous errors
        $form.find(".is-invalid").removeClass("is-invalid");
        $form.find(".error-message").text("");
        $("#successMessage").text("");

        $.ajax({
          url: "https://httpbin.org/post",
          type: "POST",
          contentType: "application/json",
          data: JSON.stringify({
            name: $("[name=name]").val(),
            email: $("[name=email]").val(),
            password: $("[name=password]").val()
          }),
          dataType: "json",
          success: function() {
            $("#successMessage").text("Registration successful!");
          },
          error: function(jqXHR) {
            // Simulated 422 handling
            if (jqXHR.status === 422) {
              var errors = jqXHR.responseJSON.errors;
              $.each(errors, function(field, messages) {
                $("[name='" + field + "']").addClass("is-invalid");
                $("[data-error-for='" + field + "']").text(messages[0]);
              });
            } else {
              $("#successMessage").text("Error: " + jqXHR.status);
            }
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Submitting the form with invalid data displays error messages next to the corresponding fields. The fields are highlighted with a red border. When all fields are valid, "Registration successful!" is displayed.

**Why this output:** The 422 response contains an `errors` object mapping field names to error messages. The error callback iterates over this object, adds the `is-invalid` class to the corresponding inputs, and displays the first error message for each field.

### Real-World Cases

- **Registration forms:** Validating email uniqueness, password strength, and required fields.
- **Checkout forms:** Validating credit card numbers, expiry dates, and shipping addresses.
- **Profile updates:** Validating name, email, and password changes.
- **Multi-step forms:** Validating each step before proceeding.

---

## Core Concept 6: 500 Errors — Displaying Generic System Failures When the Server Crashes

### Definitions

**Core Definition:** A 500 (Internal Server Error) indicates that the server encountered an unexpected condition that prevented it from fulfilling the request. It is a generic server-side error that reveals no specific details.

**Technical Definition:** The 500 status code is returned when the server encounters an unhandled exception, a database failure, a misconfiguration, or any other internal error. Unlike 4xx errors, 5xx errors are not the client's fault. The server may return a generic error page or a JSON body with a `message` field (in development, it may include stack traces). The client should display a generic "something went wrong" message, log the error for monitoring, and optionally offer a retry.

**Beginner-Friendly Explanation:** A 500 error means "the server broke." It is not your fault — something went wrong on the server side. The best the client can do is apologize to the user, log the error, and suggest trying again later.

### Purposes

- To indicate that the server failed to process a valid request.
- To display a user-friendly error message without exposing internal details.
- To log the error for developer investigation.
- To optionally offer a retry for transient failures.
- To distinguish server errors (5xx) from client errors (4xx).

### Syntax Rules and Structure

**Complete General Syntax (Client Handling):**
```javascript
$.ajax({
    url: "/api/data",
    type: "GET",
    success: function(response) { ... },
    error: function(jqXHR) {
        if (jqXHR.status >= 500) {
            // Log for monitoring
            console.error("Server error:", jqXHR.status, jqXHR.responseText);
            // Show generic message to user
            $("#error").html(
                "<p>Something went wrong on our end. " +
                "Please try again in a few moments.</p>" +
                "<button id='retryBtn'>Retry</button>"
            );
        }
    }
});
```

**5xx Status Codes:**

| Code | Meaning | Client Action |
|------|---------|---------------|
| 500 | Internal Server Error | Generic message, log, retry |
| 502 | Bad Gateway | Retry with backoff |
| 503 | Service Unavailable | Retry with backoff, show maintenance message |
| 504 | Gateway Timeout | Retry with backoff |

**Syntax Rules:**

- Check `jqXHR.status >= 500` to handle all server errors uniformly.
- Display a generic message; never expose stack traces or internal details.
- Log the error with the status code and response text for monitoring.
- Offer a retry option for transient errors.
- Implement exponential backoff for automatic retries.

**Constraints and Limitations:**

- 500 errors may be intermittent; retries may succeed.
- In development, servers may return stack traces in the response body; never display these to users.
- Some APIs return 200 with an error object for business errors; check the response body as well.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Handling a 500 Server Error with Retry**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>500 Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="loadData">Load Data</button>
  <div id="result"></div>

  <script>
    $(function() {
      function loadData(retries) {
        retries = retries || 0;

        $.ajax({
          url: "https://httpbin.org/status/500",
          type: "GET",
          success: function(data) {
            $("#result").html("<p>Data loaded successfully.</p>");
          },
          error: function(jqXHR) {
            if (jqXHR.status >= 500) {
              if (retries < 2) {
                $("#result").html(
                  "<p>Something went wrong. Retrying... (attempt " +
                  (retries + 1) + " of 2)</p>"
                );
                // Exponential backoff: 1s, then 2s
                setTimeout(function() {
                  loadData(retries + 1);
                }, Math.pow(2, retries) * 1000);
              } else {
                $("#result").html(
                  "<p>We are sorry, but something went wrong on our end. " +
                  "Please try again later.</p>" +
                  "<button id='retryBtn'>Retry</button>"
                );
                $("#retryBtn").click(function() {
                  loadData(0);
                });
              }
            } else {
              $("#result").text("Error: " + jqXHR.status);
            }
          }
        });
      }

      $("#loadData").click(function() {
        loadData(0);
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Load Data" sends a request that returns 500. The result displays "Something went wrong. Retrying... (attempt 1 of 2)", then "attempt 2 of 2", and finally the generic error message with a "Retry" button.

**Why this output:** The error callback checks `jqXHR.status >= 500` and implements exponential backoff (1s, 2s) before giving up. After two retries, it displays a generic message and offers a manual retry button.

### Real-World Cases

- **Database failures:** Connection timeouts, deadlocks, or query errors.
- **Third-party service outages:** Payment gateways, email services, or SMS providers.
- **Deployment issues:** Rolling restarts, misconfigured environment variables.
- **Resource exhaustion:** Memory limits, file descriptor limits, or CPU throttling.

---

## Core Concept 7: Global Error Catching — Configuring Global Fallbacks Using `$(document).ajaxError()`

### Definitions

**Core Definition:** Global error catching is the practice of registering a single handler for all AJAX errors using `$(document).ajaxError()`, ensuring that no error goes unhandled regardless of which request triggered it.

**Technical Definition:** jQuery provides global AJAX event handlers: `.ajaxStart()`, `.ajaxStop()`, `.ajaxComplete()`, `.ajaxSuccess()`, and `.ajaxError()`. The `$(document).ajaxError(handler)` method registers a callback that fires whenever any AJAX request fails. The handler receives `(event, jqXHR, ajaxSettings, thrownError)` and can inspect the status code, URL, and error details. Global handlers are useful for cross-cutting concerns like authentication redirects (401), server error logging (5xx), and session expiration. However, they fire **in addition to** per-request error handlers, so care must be taken to avoid duplicate handling.

**Beginner-Friendly Explanation:** Instead of handling errors separately in every AJAX call, you can set up one global error handler that catches all failures. It is like having a single security guard at the door instead of one in every room. The global handler can redirect to login when a token expires or log all server errors to a monitoring service.

### Purposes

- To handle cross-cutting error concerns (401 redirects, 5xx logging) in one place.
- To ensure that no AJAX error goes unhandled.
- To reduce code duplication across multiple AJAX calls.
- To provide consistent error behavior across the application.
- To centralize error reporting and monitoring.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
$(document).ajaxError(function(event, jqXHR, ajaxSettings, thrownError) {
    var url = ajaxSettings.url;
    var status = jqXHR.status;

    if (status === 401) {
        // Redirect to login
        localStorage.removeItem("authToken");
        window.location.href = "/login";
    } else if (status === 403) {
        // Show permission error
        $("#global-error").text("You do not have permission to perform this action.");
    } else if (status >= 500) {
        // Log server errors for monitoring
        console.error("Server error at " + url + ": " + status);
        $("#global-error").text("Something went wrong. Please try again later.");
    }
});
```

| Global Event | Fires When | Handler Signature |
|--------------|------------|-------------------|
| `ajaxStart` | First AJAX request starts | `(event)` |
| `ajaxStop` | Last AJAX request completes | `(event)` |
| `ajaxComplete` | Every AJAX request completes | `(event, jqXHR, settings)` |
| `ajaxSuccess` | Every AJAX request succeeds | `(event, jqXHR, settings, data)` |
| `ajaxError` | Every AJAX request fails | `(event, jqXHR, settings, thrownError)` |

**Syntax Rules:**

- Register the global handler once, typically after `$(document).ready()`.
- Use `ajaxSettings.url` to determine which request failed.
- Avoid duplicating logic that is also in per-request handlers.
- Use the global handler for cross-cutting concerns (401, 5xx), not for request-specific validation errors (422).
- Use namespaced events (`.ajaxError.myApp`) for easy removal.

**Constraints and Limitations:**

- Global handlers fire for **all** AJAX requests, including those from third-party libraries.
- Global handlers fire **after** the per-request error handlers; both will execute.
- Global handlers cannot prevent the per-request handler from running.
- Be cautious with redirects in global handlers; exclude the login endpoint to avoid loops.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Global Error Handler with 401 Redirect**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Global Error Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="global-error" style="color: red;"></div>
  <button id="loadData">Load Data</button>

  <script>
    $(function() {
      // Step 1: Register the global error handler
      $(document).ajaxError(function(event, jqXHR, settings, thrownError) {
        var status = jqXHR.status;
        var url = settings.url;

        console.log("AJAX Error:", url, status, thrownError);

        if (status === 401) {
          localStorage.removeItem("authToken");
          $("#global-error").text("Session expired. Redirecting to login...");
          // In a real app: window.location.href = "/login";
        } else if (status === 403) {
          $("#global-error").text("You do not have permission to perform this action.");
        } else if (status >= 500) {
          $("#global-error").text("Server error. Please try again later.");
        } else if (status === 0) {
          $("#global-error").text("Network error. Please check your connection.");
        }
      });

      // Step 2: Per-request handler for specific errors (e.g., 422)
      $("#loadData").click(function() {
        $.ajax({
          url: "https://httpbin.org/status/403",
          type: "GET",
          error: function(jqXHR) {
            if (jqXHR.status === 422) {
              // Handle validation errors specifically
              $("#global-error").text("Validation errors.");
            }
            // The global handler also fires
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Load Data" sends a request that returns 403. The global error handler displays "You do not have permission to perform this action." The Console logs the error details.

**Why this output:** The `$(document).ajaxError()` handler fires for every failed AJAX request, regardless of which call triggered it. It handles 401, 403, 5xx, and network errors in one place.

### Real-World Cases

- **Session management:** Redirecting to login on any 401.
- **Error monitoring:** Logging all 5xx errors to a monitoring service.
- **User notifications:** Displaying a global error banner for server failures.
- **Network detection:** Showing "offline" messages when status is 0.

---

## Core Concept 8: Network Timeout & Retry Logic — Setting Explicit Timeout Thresholds and Exponential Backoffs

### Definitions

**Core Definition:** Network timeout handling is the practice of setting a maximum time to wait for a response and retrying the request with increasing delays (exponential backoff) if the timeout occurs or a transient error is returned.

**Technical Definition:** The `timeout` option in `$.ajax()` specifies the number of milliseconds to wait before aborting the request. When the timeout is exceeded, the request fails with `textStatus === "timeout"` and `jqXHR.status === 0`. Retry logic wraps the AJAX call in a function that retries on timeout, network errors (status 0), and 5xx responses, using exponential backoff (e.g., 1s, 2s, 4s, 8s) to avoid overwhelming the server. A maximum retry count prevents infinite loops. The `Retry-After` header (used with 429 and 503) provides the server's recommended delay.

**Beginner-Friendly Explanation:** A timeout is like waiting for a friend who is late. After a certain amount of time, you decide they are not coming and make other plans. Retry logic is like calling them again after a few minutes, then waiting longer before calling again. Exponential backoff means you wait longer each time, so you do not annoy them with constant calls.

### Purposes

- To prevent requests from hanging indefinitely.
- To improve resilience against transient network failures.
- To handle server errors (500, 503) that may be temporary.
- To respect server rate limits (429) by backing off.
- To provide a better user experience when the network is unreliable.

### Syntax Rules and Structure

**Complete General Syntax (Timeout):**
```javascript
$.ajax({
    url: "/api/data",
    type: "GET",
    timeout: 5000, // 5 seconds
    error: function(jqXHR, textStatus) {
        if (textStatus === "timeout") {
            $("#error").text("The request timed out. Please try again.");
        }
    }
});
```

**Complete General Syntax (Retry with Exponential Backoff):**
```javascript
function ajaxWithRetry(options, maxRetries, baseDelay) {
    maxRetries = maxRetries || 3;
    baseDelay = baseDelay || 1000;

    return $.ajax(options).fail(function(jqXHR, textStatus) {
        var shouldRetry = (
            textStatus === "timeout" ||
            jqXHR.status === 0 ||
            jqXHR.status >= 500
        );

        if (shouldRetry && maxRetries > 0) {
            var delay = baseDelay * Math.pow(2, (3 - maxRetries));
            console.log("Retrying in " + delay + "ms... (" + maxRetries + " retries left)");

            return new Promise(function(resolve, reject) {
                setTimeout(function() {
                    ajaxWithRetry(options, maxRetries - 1, baseDelay)
                        .done(resolve)
                        .fail(reject);
                }, delay);
            });
        }
    });
}

// Usage
ajaxWithRetry({
    url: "/api/data",
    type: "GET",
    timeout: 5000
}, 3, 1000)
    .done(function(data) {
        $("#result").text("Success: " + data);
    })
    .fail(function() {
        $("#result").text("All retries failed. Please try again later.");
    });
```

| Scenario | Retry? | Backoff |
|----------|--------|---------|
| Timeout | Yes | Exponential |
| Network error (status 0) | Yes | Exponential |
| 500 Internal Server Error | Yes | Exponential |
| 502 Bad Gateway | Yes | Exponential |
| 503 Service Unavailable | Yes (respect `Retry-After`) | Server-specified |
| 429 Too Many Requests | Yes (respect `Retry-After`) | Server-specified |
| 400 Bad Request | No | — |
| 401 Unauthorized | No (redirect to login) | — |
| 403 Forbidden | No | — |
| 404 Not Found | No | — |
| 422 Validation Error | No | — |

**Syntax Rules:**

- Set a `timeout` value appropriate for the operation (shorter for interactive, longer for uploads).
- Retry only on transient errors (timeout, status 0, 5xx, 429).
- Use exponential backoff with jitter to avoid thundering herd.
- Set a maximum retry count (typically 3).
- Respect the `Retry-After` header when present.
- Log retries for debugging and monitoring.

**Constraints and Limitations:**

- Retrying non-idempotent requests (POST) can create duplicates; use idempotency keys.
- Some browsers limit concurrent connections; excessive retries can block other requests.
- The `timeout` option applies to the entire request, including the response download.
- Exponential backoff increases the total time to failure; set reasonable maximums.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: AJAX with Timeout and Exponential Backoff Retry**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Retry Logic Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="loadData">Load Data</button>
  <pre id="output"></pre>

  <script>
    $(function() {
      function ajaxWithRetry(options, maxRetries, baseDelay) {
        maxRetries = maxRetries || 3;
        baseDelay = baseDelay || 1000;

        return $.ajax(options).fail(function(jqXHR, textStatus) {
          var shouldRetry = (
            textStatus === "timeout" ||
            jqXHR.status === 0 ||
            jqXHR.status >= 500
          );

          if (shouldRetry && maxRetries > 0) {
            var delay = baseDelay * Math.pow(2, (3 - maxRetries));
            $("#output").append(
              "Request failed (" + (textStatus || jqXHR.status) + "). " +
              "Retrying in " + delay + "ms... (" + maxRetries + " retries left)\n"
            );

            return new Promise(function(resolve, reject) {
              setTimeout(function() {
                ajaxWithRetry(options, maxRetries - 1, baseDelay)
                  .done(resolve)
                  .fail(reject);
              }, delay);
            });
          }
        });
      }

      $("#loadData").click(function() {
        $("#output").text("Starting request...\n");

        ajaxWithRetry({
          url: "https://httpbin.org/status/500",
          type: "GET",
          timeout: 3000
        }, 3, 1000)
          .done(function() {
            $("#output").append("Success!\n");
          })
          .fail(function() {
            $("#output").append("All retries failed. Please try again later.\n");
          });
      });
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Starting request...
Request failed (error). Retrying in 1000ms... (3 retries left)
Request failed (error). Retrying in 2000ms... (2 retries left)
Request failed (error). Retrying in 4000ms... (1 retries left)
All retries failed. Please try again later.
```

**Why this output:** The `ajaxWithRetry` function wraps the AJAX call and checks whether the failure is retryable (timeout, network error, or 5xx). It retries with exponential backoff (1s, 2s, 4s) up to three times. After all retries fail, it displays a final error message.

### Real-World Cases

- **Unreliable networks:** Mobile users with intermittent connectivity.
- **Rate-limited APIs:** Backing off on 429 responses.
- **Server deployments:** Retrying during rolling restarts (503).
- **Database timeouts:** Retrying on transient connection errors.
- **File uploads:** Retrying large uploads that time out.

---

## References

- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 4918 — HTTP Extensions for Web Distributed Authoring and Versioning (WebDAV) — https://www.rfc-editor.org/rfc/rfc4918
- MDN Web Docs — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- MDN Web Docs — 401 Unauthorized — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401
- MDN Web Docs — 403 Forbidden — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403
- MDN Web Docs — 404 Not Found — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404
- MDN Web Docs — 422 Unprocessable Entity — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/422
- MDN Web Docs — 500 Internal Server Error — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500
- jQuery API Documentation — jQuery.ajax() — https://api.jquery.com/jQuery.ajax/
- jQuery API Documentation — ajaxError event — https://api.jquery.com/ajaxError/
- jQuery API Documentation — deferred.fail() — https://api.jquery.com/deferred.fail/
- OWASP — REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- Google API Design Guide — Errors — https://cloud.google.com/apis/design/errors
- Microsoft REST API Guidelines — Error condition responses — https://github.com/microsoft/api-guidelines/
- AWS — Exponential Backoff and Jitter — https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- Stripe API — Errors — https://stripe.com/docs/api/errors
- 阮一峰 — HTTP 状态码详解 — https://www.ruanyifeng.com/blog/2014/05/restful_api.html
- 掘金 — 前端错误处理最佳实践 — https://juejin.cn/
- CSDN — jQuery AJAX 错误处理 — https://blog.csdn.net/