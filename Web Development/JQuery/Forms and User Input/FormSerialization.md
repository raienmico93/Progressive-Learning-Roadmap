# jQuery Form Serialization: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery Form Serialization is the process of converting form control values into a format suitable for transmission — either a URL-encoded query string (`.serialize()`) or a structured array of name-value pairs (`.serializeArray()`).

**Technical Definition**
`.serialize()` and `.serializeArray()` are jQuery methods that encode the values of all successful form controls within a matched set. A successful control is one that has a `name` attribute, is not disabled, and (for checkboxes and radio buttons) is currently checked. Both methods produce output in the standard `application/x-www-form-urlencoded` format. `.serialize()` returns a single encoded string (`name1=value1&name2=value2`), while `.serializeArray()` returns an array of objects with `name` and `value` properties, suitable for JSON conversion or programmatic manipulation .

**Beginner-Friendly Explanation**
When you submit a form, the browser packs all the field values into a format the server can understand. jQuery's serialization methods let you do the same thing manually from JavaScript. `.serialize()` gives you a query string you can append to a URL. `.serializeArray()` gives you a structured list that's easier to read and manipulate in code.

### Key Characteristics

- **W3C-Compliant Encoding**: Output follows the standard `application/x-www-form-urlencoded` format .
- **Successful Controls Only**: Only controls with a `name` attribute, not disabled, and checked (if applicable) are included .
- **Works on Any jQuery Collection**: Serialization operates on any set of form elements, not just `<form>` elements .
- **No File Support**: File inputs cannot be serialized; use `FormData` for file uploads .
- **No Button Values**: Submit button values are typically excluded unless explicitly included .

### Prerequisites

- Basic HTML form elements and their attributes.
- jQuery library included via CDN or local file.
- Understanding of jQuery selectors and DOM traversal.
- Familiarity with AJAX and HTTP request formats.

### Related Programming Areas

- **AJAX**: Sending serialized data to a server with `$.ajax()`, `$.post()`, or `$.get()`.
- **FormData API**: Modern alternative supporting file uploads.
- **URL Encoding**: Understanding percent-encoding and query strings.
- **JSON**: Converting `.serializeArray()` output to JSON.

### Core Concepts / Features

1. `.serialize()` — Query-String Generation
2. `.serializeArray()` — JSON-Ready Array Generation
3. Form Submission Payloads for AJAX
4. Limitations of Serialization

---

## Core Concept 1: `.serialize()`

### Definitions

**Core Definition**
`.serialize()` encodes the values of all successful form controls in a jQuery collection into a single URL-encoded query string.

**Technical Definition**
`.serialize()` is a jQuery method added in version 1.0. It produces a string in the standard `application/x-www-form-urlencoded` format, identical to what a browser would send in a form submission using the GET method. The method internally calls `.serializeArray()` and then joins the results using `jQuery.param()`. Special characters are percent-encoded, and spaces are converted to `+` signs (or `%20` in some contexts) per URL encoding rules .

**Beginner-Friendly Explanation**
`.serialize()` takes all the filled-in form fields and turns them into a single string like `name=John&email=john%40example.com`. You can append this string to a URL or send it as the body of an AJAX request.

### Purposes

- To generate a query string for GET requests.
- To create the request body for POST requests with `application/x-www-form-urlencoded` content type.
- To build URLs with form data as query parameters.
- To log form data in a human-readable format.
- To pass form data to server endpoints that expect URL-encoded strings.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).serialize()
```

**Component Breakdown**

- `$(selector)` : A jQuery object containing form elements (typically a `<form>`).
- `.serialize()` : Returns a URL-encoded string.

**Syntax Rules**

1. The method returns a string; it never returns `null` or `undefined`.
2. Only "successful controls" are serialized: elements with a `name`, not disabled, and checked (if checkbox/radio) .
3. Submit button values are only included if a submit button is the triggering element during form submission; otherwise they are excluded .
4. `<select>` values are serialized as their `value` attribute (or text if no `value`).
5. `<textarea>` values are serialized with newlines encoded as `%0D%0A` .

**Constraints and Limitations**

- **No File Inputs**: Files cannot be serialized; use `FormData` for file uploads .
- **No Nested Data**: The output is flat; complex nested objects are not supported.
- **Character Encoding**: Uses `encodeURIComponent` internally; some characters may be double-encoded in certain edge cases.
- **No Submit Button by Default**: Submit button values are not included unless explicitly named and triggered.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Form Serialization**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.serialize() — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="myForm">
    <input type="text" name="username" value="John Doe">
    <input type="email" name="email" value="john@example.com">
    <input type="checkbox" name="subscribe" value="yes" checked>
    <input type="checkbox" name="newsletter" value="yes">
    <select name="country">
      <option value="us">United States</option>
      <option value="uk" selected>United Kingdom</option>
    </select>
    <button type="button" id="serializeBtn">Serialize</button>
  </form>

  <script>
    $(function () {
      $("#serializeBtn").on("click", function () {
        // Step 1: Serialize the form
        var data = $("#myForm").serialize();

        // Step 2: Log the query string
        console.log("Serialized:", data);
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Serialized: username=John+Doe&email=john%40example.com&subscribe=yes&country=uk
```

**Why This Output Occurs**
- `username=John+Doe` — space encoded as `+`.
- `email=john%40example.com` — `@` encoded as `%40`.
- `subscribe=yes` — checked checkbox included.
- `newsletter` — omitted because unchecked.
- `country=uk` — selected option's value included .

---

**Example 2: Serializing a Subset of Fields**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.serialize() — Subset</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="orderForm">
    <input type="text" name="customer" value="Alice">
    <input type="text" name="product" value="Laptop">
    <input type="text" name="internalCode" value="XYZ123">
    <button type="button" id="serializeSubset">Serialize Customer Only</button>
  </form>

  <script>
    $(function () {
      $("#serializeSubset").on("click", function () {
        // Step 1: Serialize only specific fields
        var data = $("#orderForm input[name='customer']").serialize();

        // Step 2: Log
        console.log("Subset:", data);
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Subset: customer=Alice
```

**Why This Output Occurs**
`.serialize()` works on any jQuery collection, not just `<form>` elements. Only the selected input is serialized .

### Real-World Cases

- **GET Requests**: Appending serialized data to a URL for search queries.
- **Login Forms**: Sending credentials as URL-encoded POST data.
- **Search Filters**: Building query strings from filter forms.
- **Logging**: Recording form data for debugging.

### References

- jQuery API — .serialize() – https://api.jquery.com/serialize/
- W3C — URL Standard – https://url.spec.whatwg.org/

---

## Core Concept 2: `.serializeArray()`

### Definitions

**Core Definition**
`.serializeArray()` encodes the values of all successful form controls in a jQuery collection into an array of objects, each with `name` and `value` properties.

**Technical Definition**
`.serializeArray()` is a jQuery method added in version 1.2. It returns an array of objects in the form `[{ name: "field1", value: "val1" }, { name: "field2", value: "val2" }]`. The array is suitable for JSON conversion (via `JSON.stringify()`) and for programmatic manipulation. It can be combined with `.serialize()` by passing the array to `jQuery.param()` to produce a query string .

**Beginner-Friendly Explanation**
`.serializeArray()` is like `.serialize()` but gives you a structured list instead of a string. Each item in the list has a `name` and a `value`. This is easier to work with in JavaScript, especially if you need to modify the data before sending it.

### Purposes

- To produce structured data for JSON conversion.
- To allow programmatic modification of form data before sending.
- To integrate with APIs that expect JSON payloads.
- To iterate over form fields with their names and values.
- To convert to a query string via `jQuery.param()`.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).serializeArray()
```

**Component Breakdown**

- `$(selector)` : A jQuery object containing form elements.
- `.serializeArray()` : Returns an array of `{ name, value }` objects.

**Syntax Rules**

1. The returned array can be passed to `jQuery.param()` to produce a query string .
2. The array can be converted to JSON with `JSON.stringify()`.
3. The same "successful controls" rules apply as for `.serialize()` .
4. The order of array items follows DOM order.
5. Duplicate names (e.g., multiple checkboxes with the same name) produce multiple entries in the array.

**Constraints and Limitations**

- **No File Inputs**: Files cannot be serialized .
- **Flat Structure**: Nested objects are not supported.
- **No Type Preservation**: All values are strings; numbers and booleans must be converted.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Array Serialization**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.serializeArray() — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="profileForm">
    <input type="text" name="name" value="Jane">
    <input type="number" name="age" value="30">
    <input type="checkbox" name="hobbies" value="reading" checked>
    <input type="checkbox" name="hobbies" value="gaming" checked>
    <input type="checkbox" name="hobbies" value="cooking">
  </form>

  <script>
    $(function () {
      // Step 1: Serialize to array
      var dataArray = $("#profileForm").serializeArray();

      // Step 2: Log the array
      console.log("Array:", dataArray);

      // Step 3: Convert to JSON
      var json = JSON.stringify(dataArray);
      console.log("JSON:", json);
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Array: [
  { name: "name", value: "Jane" },
  { name: "age", value: "30" },
  { name: "hobbies", value: "reading" },
  { name: "hobbies", value: "gaming" }
]
JSON: [{"name":"name","value":"Jane"},{"name":"age","value":"30"},{"name":"hobbies","value":"reading"},{"name":"hobbies","value":"gaming"}]
```

**Why This Output Occurs**
Each successful control produces one array entry. Multiple checkboxes with the same name produce multiple entries. The unchecked "cooking" checkbox is omitted .

---

**Example 2: Converting Array to Object**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.serializeArray() — To Object</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="loginForm">
    <input type="text" name="username" value="admin">
    <input type="password" name="password" value="secret123">
  </form>

  <script>
    $(function () {
      // Step 1: Get serialized array
      var arr = $("#loginForm").serializeArray();

      // Step 2: Convert to plain object
      var obj = {};
      $.each(arr, function (i, field) {
        obj[field.name] = field.value;
      });

      console.log("Object:", obj);
      console.log("Username:", obj.username);
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Object: { username: "admin", password: "secret123" }
Username: admin
```

**Why This Output Occurs**
Iterating over the array and assigning `field.value` to `obj[field.name]` produces a plain JavaScript object .

### Real-World Cases

- **JSON APIs**: Sending form data as JSON to REST endpoints.
- **Data Manipulation**: Adding, removing, or modifying fields before submission.
- **Form Debugging**: Logging structured form data.
- **Client-Side Storage**: Saving form state to `localStorage` as JSON.

### References

- jQuery API — .serializeArray() – https://api.jquery.com/serializeArray/
- jQuery API — jQuery.param() – https://api.jquery.com/jQuery.param/

---

## Core Concept 3: Form Submission Payloads for AJAX

### Definitions

**Core Definition**
Form submission payloads for AJAX are the data structures sent to the server via asynchronous HTTP requests, created by serializing form data.

**Technical Definition**
When submitting a form via AJAX, the serialized output of `.serialize()` or `.serializeArray()` is passed as the `data` parameter to `$.ajax()`, `$.post()`, or `$.get()`. jQuery automatically sets the `Content-Type` header to `application/x-www-form-urlencoded; charset=UTF-8` when a string is passed, or to `application/json` if a plain object or array is stringified. For file uploads, `FormData` must be used instead, with `processData: false` and `contentType: false` .

**Beginner-Friendly Explanation**
When you submit a form without reloading the page (AJAX), you need to package the form data into a format the server understands. jQuery's serialization methods do this packaging. For text-only forms, `.serialize()` or `.serializeArray()` works. For forms with files, you need `FormData`.

### Purposes

- To submit form data without reloading the page.
- To send form data as URL-encoded or JSON payloads.
- To integrate form submission with REST APIs.
- To handle form submissions asynchronously with `.done()` and `.fail()` callbacks.
- To control request headers and content types.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Approach 1: URL-Encoded POST**
```javascript
$.ajax({
  url: "/submit",
  type: "POST",
  data: $("#form").serialize(),
  success: function(response) { }
});
```

**Approach 2: JSON POST**
```javascript
$.ajax({
  url: "/submit",
  type: "POST",
  contentType: "application/json",
  data: JSON.stringify($("#form").serializeArray()),
  success: function(response) { }
});
```

**Approach 3: FormData (Files)**
```javascript
var formData = new FormData($("#form")[0]);
$.ajax({
  url: "/upload",
  type: "POST",
  data: formData,
  processData: false,
  contentType: false,
  success: function(response) { }
});
```

**Component Breakdown**

- `url` : The server endpoint.
- `type` : HTTP method (`GET`, `POST`).
- `data` : The serialized payload.
- `processData: false` : Prevents jQuery from converting `FormData` to a query string.
- `contentType: false` : Lets the browser set the correct `multipart/form-data` header.

**Syntax Rules**

1. `.serialize()` produces a string suitable for `application/x-www-form-urlencoded` .
2. `.serializeArray()` can be stringified to JSON for `application/json` .
3. `FormData` is required for file uploads; `.serialize()` cannot include files .
4. When using `FormData`, set `processData: false` and `contentType: false` .
5. The `success` callback is deprecated in favour of `.done()` and `.fail()` .

**Constraints and Limitations**

- **File Uploads**: `.serialize()` cannot handle files; use `FormData`.
- **Nested Data**: Flat structure only; complex nested objects require manual construction.
- **Character Encoding**: Ensure the server expects the correct content type.

### Multiple Annotated Complete Code Examples

**Example 1: AJAX Form Submission with `.serialize()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>AJAX — serialize()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="contactForm">
    <input type="text" name="name" placeholder="Name" required>
    <input type="email" name="email" placeholder="Email" required>
    <textarea name="message" placeholder="Message"></textarea>
    <button type="submit">Send</button>
  </form>
  <p id="result"></p>

  <script>
    $(function () {
      $("#contactForm").on("submit", function (event) {
        event.preventDefault(); // Step 1: Prevent default submission

        // Step 2: Serialize form data
        var formData = $(this).serialize();

        // Step 3: Send via AJAX
        $.ajax({
          url: "/api/contact",
          type: "POST",
          data: formData,
          success: function (response) {
            $("#result").text("Message sent successfully!");
            $("#contactForm")[0].reset(); // Clear form
          },
          error: function () {
            $("#result").text("An error occurred. Please try again.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Submitting the form sends the data via AJAX without page reload.
- On success, "Message sent successfully!" is displayed and the form is cleared.
- On error, an error message is displayed.

**Why This Output Occurs**
`.serialize()` creates a URL-encoded string. `$.ajax()` sends it as the request body with the default `application/x-www-form-urlencoded` content type .

---

**Example 2: AJAX Form Submission with JSON**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>AJAX — JSON</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="jsonForm">
    <input type="text" name="title" value="Hello">
    <input type="number" name="priority" value="1">
    <button type="submit">Submit JSON</button>
  </form>
  <pre id="output"></pre>

  <script>
    $(function () {
      $("#jsonForm").on("submit", function (event) {
        event.preventDefault();

        // Step 1: Serialize to array
        var dataArray = $(this).serializeArray();

        // Step 2: Convert to plain object
        var dataObject = {};
        $.each(dataArray, function (i, field) {
          dataObject[field.name] = field.value;
        });

        // Step 3: Send as JSON
        $.ajax({
          url: "/api/items",
          type: "POST",
          contentType: "application/json",
          data: JSON.stringify(dataObject),
          success: function (response) {
            $("#output").text("Sent: " + JSON.stringify(dataObject, null, 2));
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Submitting sends the data as a JSON object.
- The output area shows the JSON that was sent.

**Why This Output Occurs**
`JSON.stringify()` converts the object to a JSON string. `contentType: "application/json"` tells the server to expect JSON .

### Real-World Cases

- **Contact Forms**: Sending messages without page reload.
- **Login/Registration**: Authenticating users asynchronously.
- **Search Autocomplete**: Sending query strings as the user types.
- **Shopping Cart Updates**: Sending item quantities via AJAX.

### References

- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- MDN Web Docs — FormData – https://developer.mozilla.org/en-US/docs/Web/API/FormData

---

## Core Concept 4: Limitations of Serialization

### Definitions

**Core Definition**
Serialization limitations are the constraints and restrictions that prevent `.serialize()` and `.serializeArray()` from handling certain types of form data, particularly file uploads and complex nested structures.

**Technical Definition**
jQuery's serialization methods produce flat, URL-encoded output in the `application/x-www-form-urlencoded` format. This format cannot represent binary file data, nested objects, or hierarchical structures. File inputs are excluded entirely. Additionally, serialization only includes "successful controls" — those with a `name` attribute, not disabled, and checked (if applicable). Submit button values are typically excluded. The `FormData` API, introduced with XMLHttpRequest Level 2, addresses these limitations by supporting file uploads and multipart/form-data .

**Beginner-Friendly Explanation**
`.serialize()` is great for text forms, but it has limits. It can't handle file uploads, it can't represent nested data, and it ignores certain fields (like disabled ones). When you need those features, you use `FormData` instead.

### Purposes

- To understand when serialization is appropriate and when it is not.
- To know which form elements are excluded from serialization.
- To choose the correct approach for file uploads.
- To plan form data structures that work with serialization.
- To avoid common pitfalls when sending form data via AJAX.

### Syntax Rules and Structure

**What Serialization Includes**

| Element | Included? | Condition |
|---|---|---|
| `<input type="text">` | Yes | Has `name`, not disabled |
| `<input type="checkbox">` | Yes | Has `name`, checked |
| `<input type="radio">` | Yes | Has `name`, checked |
| `<input type="file">` | **No** | Always excluded |
| `<input type="submit">` | Usually No | Only if triggered by submission |
| `<select>` | Yes | Has `name`, not disabled |
| `<textarea>` | Yes | Has `name`, not disabled |
| Disabled elements | **No** | Excluded |
| Elements without `name` | **No** | Excluded |

**FormData Syntax for Files**
```javascript
var formData = new FormData($("#form")[0]);
$.ajax({
  url: "/upload",
  type: "POST",
  data: formData,
  processData: false,
  contentType: false
});
```

**Component Breakdown**

- `new FormData(formElement)` : Collects all form data including files.
- `processData: false` : Prevents jQuery from converting the data.
- `contentType: false` : Lets the browser set `multipart/form-data` with boundary.

**Syntax Rules**

1. File inputs are **never** included in `.serialize()` or `.serializeArray()` .
2. `FormData` includes files and text fields in `multipart/form-data` format.
3. Disabled elements are excluded from serialization.
4. Elements without a `name` attribute are excluded.
5. Unchecked checkboxes and radio buttons are excluded.

**Constraints and Limitations**

- **No File Uploads**: `.serialize()` cannot handle files .
- **No Nested Data**: Flat structure only.
- **No Binary Data**: Only text values are supported.
- **Submit Button Exclusion**: Submit button values are not included by default.
- **FormData Browser Support**: `FormData` is supported in all modern browsers; IE10+ .

### Multiple Annotated Complete Code Examples

**Example 1: Demonstrating File Exclusion**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Limitations — File Exclusion</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="uploadForm">
    <input type="text" name="title" value="My File">
    <input type="file" name="attachment">
    <button type="button" id="serializeBtn">Serialize</button>
    <button type="button" id="formDataBtn">FormData</button>
  </form>
  <p id="output"></p>

  <script>
    $(function () {
      $("#serializeBtn").on("click", function () {
        // Step 1: Serialize — file input is excluded
        var data = $("#uploadForm").serialize();
        console.log("Serialized:", data);
        $("#output").text("Serialized: " + data);
      });

      $("#formDataBtn").on("click", function () {
        // Step 2: FormData — file input is included
        var formData = new FormData($("#uploadForm")[0]);
        var entries = [];
        for (var pair of formData.entries()) {
          entries.push(pair[0] + "=" + pair[1]);
        }
        console.log("FormData entries:", entries);
        $("#output").text("FormData: " + entries.join(", "));
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- **Serialize**: `"title=My+File"` — the file input is excluded.
- **FormData**: `"title=My File, attachment=[object File]"` — the file is included.

**Why This Output Occurs**
`.serialize()` cannot handle binary file data and excludes file inputs entirely. `FormData` includes them in `multipart/form-data` format .

---

**Example 2: FormData for File Upload via AJAX**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Limitations — File Upload</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="uploadForm">
    <input type="text" name="description" placeholder="Description">
    <input type="file" name="file" id="fileInput">
    <button type="submit">Upload</button>
  </form>
  <p id="status"></p>

  <script>
    $(function () {
      $("#uploadForm").on("submit", function (event) {
        event.preventDefault();

        // Step 1: Create FormData from the form
        var formData = new FormData(this);

        // Step 2: Send via AJAX with correct settings
        $.ajax({
          url: "/api/upload",
          type: "POST",
          data: formData,
          processData: false,  // Don't convert FormData
          contentType: false,  // Let browser set multipart/form-data
          success: function () {
            $("#status").text("Upload successful!");
          },
          error: function () {
            $("#status").text("Upload failed.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Submitting the form uploads both the text description and the file.
- Success message displayed on completion.

**Why This Output Occurs**
`FormData` captures all form data including files. The `processData: false` and `contentType: false` settings prevent jQuery from interfering with the `multipart/form-data` encoding .

### Real-World Cases

- **File Upload Forms**: Using `FormData` instead of `.serialize()`.
- **Complex Nested Forms**: Manually constructing payloads for nested data.
- **Disabled Field Handling**: Understanding why disabled fields are excluded.
- **Submit Button Values**: Explicitly adding submit button values when needed.

### References

- jQuery API — .serialize() – https://api.jquery.com/serialize/
- MDN Web Docs — FormData – https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs — Using FormData Objects – https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects

---

## Summary Comparison Table

| Feature | `.serialize()` | `.serializeArray()` | `FormData` |
|---|---|---|---|
| Output Type | String | Array of objects | Object |
| Format | URL-encoded | Array | multipart/form-data |
| File Support | No | No | Yes |
| JSON-Ready | No | Yes | No |
| Nested Data | No | No | No |
| Disabled Fields | Excluded | Excluded | Excluded |
| Use Case | Query strings, URL-encoded POST | JSON APIs, manipulation | File uploads |

---

## Important Notes on Version-Specific Behaviour and Limitations

1. **`.serialize()` Added in jQuery 1.0**: Available in all versions .
2. **`.serializeArray()` Added in jQuery 1.2**: Returns array of `{name, value}` objects .
3. **File Inputs**: Never included in jQuery serialization; use `FormData` .
4. **`processData` and `contentType`**: Must be set to `false` for `FormData` uploads .
5. **Submit Button Values**: Only included if the submit button triggered the submission and has a `name` .
6. **Disabled Elements**: Always excluded from serialization .
7. **FormData Browser Support**: Supported in all modern browsers (IE10+) .

---

## References

- jQuery API — .serialize() – https://api.jquery.com/serialize/
- jQuery API — .serializeArray() – https://api.jquery.com/serializeArray/
- jQuery API — jQuery.param() – https://api.jquery.com/jQuery.param/
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- MDN Web Docs — FormData – https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs — Using FormData Objects – https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects
- W3C — URL Standard – https://url.spec.whatwg.org/
- Stack Overflow — .serialize() file input limitations – https://stackoverflow.com/revisions/1f66e6d0-8d1b-42f6-9a8c-4a38f0f9e5e5/view-source