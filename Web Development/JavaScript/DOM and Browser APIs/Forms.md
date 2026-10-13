# Forms — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A form is an HTML element (`<form>`) that groups interactive controls (inputs, buttons, selects, textareas) for collecting and submitting user data to a server or processing it client-side.

**Technical Definition:** Per MDN, the `<form>` element represents a document section containing interactive controls for submitting information. Forms are the primary mechanism for user input on the web. The HTML Living Standard defines form elements (`<form>`, `<input>`, `<textarea>`, `<select>`, `<button>`, `<label>`, `<fieldset>`, `<legend>`, `<datalist>`, `<output>`), the constraint validation API, and the form submission algorithm. The `HTMLFormElement` interface represents the form in the DOM and exposes properties like `elements`, `action`, `method`, and methods like `submit()`, `reset()`, and `checkValidity()`.

**Beginner-Friendly Explanation:** A form is a section of a web page where users can type information, select options, or upload files. When they click "Submit," the data is sent somewhere — usually to a server. Forms are everywhere: login pages, search boxes, checkout pages, surveys, and contact forms. JavaScript can validate form data before it's submitted, show error messages, and process data without reloading the page.

### Key Characteristics

- **Interactive controls:** Forms contain input elements (text fields, checkboxes, radio buttons, selects, file uploads, etc.).
- **Submission:** Forms can submit data via GET or POST to a URL, or be handled entirely by JavaScript.
- **Validation:** Built-in HTML5 validation attributes (`required`, `pattern`, `min`, `max`, `minlength`, `maxlength`, `type`) plus the Constraint Validation API for custom logic.
- **Events:** Forms emit events (`submit`, `reset`, `input`, `change`, `focus`, `blur`, `invalid`) that JavaScript can listen for.
- **FormData:** The `FormData` API provides an easy way to collect and send form data, especially for `fetch()` requests.
- **Accessibility:** Forms must be accessible — labels, ARIA attributes, keyboard navigation, and error announcements.
- **Security:** Forms are a common attack surface (XSS, CSRF, injection); proper handling is essential.

### Prerequisites

- Basic understanding of HTML elements and attributes.
- Familiarity with the DOM and selecting elements.
- Knowledge of DOM events and event listeners.
- Awareness of HTTP methods (GET, POST) and URLs.

### Related Programming Areas

- **Client-side validation:** Improving UX by validating before submission.
- **Server-side validation:** Always required for security; never trust client-side validation alone.
- **AJAX/fetch:** Submitting forms without page reloads.
- **Accessibility:** Ensuring forms are usable by screen readers and keyboard users.
- **Security:** Preventing XSS, CSRF, and injection attacks.

---

## Form Elements

### Definitions

**Core Definition:** Form elements are the HTML tags used to build a form: `<form>`, `<input>`, `<textarea>`, `<select>`, `<option>`, `<button>`, `<label>`, `<fieldset>`, `<legend>`, `<datalist>`, and `<output>`.

**Technical Definition:** Per the HTML Living Standard, form-associated elements are elements that are associated with a form and contribute to form submission. These include `<button>`, `<fieldset>`, `<input>`, `<object>`, `<output>`, `<select>`, and `<textarea>`. The `<form>` element itself is the container.

### Purposes

- To provide a structured way to collect user input.
- To associate labels with controls for accessibility.
- To group related controls with fieldsets.
- To submit data to a server or process it client-side.

### Common Form Elements

| Element | Purpose |
|---------|---------|
| `<form>` | Container for form controls |
| `<input>` | Single-line input (many types) |
| `<textarea>` | Multi-line text input |
| `<select>` + `<option>` | Dropdown list or list box |
| `<button>` | Clickable button (submit, reset, or custom) |
| `<label>` | Caption for a form control |
| `<fieldset>` | Groups related controls |
| `<legend>` | Caption for a fieldset |
| `<datalist>` | Suggestions for an input |
| `<output>` | Result of a calculation |
| `<progress>` | Progress indicator |
| `<meter>` | Scalar measurement |

### Annotated Code Example

```html
<!-- Step 1: A complete form with common elements. -->
<form id="signup" action="/signup" method="post">
  <!-- Step 2: Group related fields with fieldset. -->
  <fieldset>
    <legend>Account Information</legend>

    <!-- Step 3: Label and input with id association. -->
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

    <label for="password">Password:</label>
    <input type="password" id="password" name="password"
           minlength="8" required>

    <!-- Step 4: Select with options. -->
    <label for="country">Country:</label>
    <select id="country" name="country">
      <option value="">-- Select --</option>
      <option value="us">United States</option>
      <option value="uk">United Kingdom</option>
    </select>

    <!-- Step 5: Textarea. -->
    <label for="bio">Bio:</label>
    <textarea id="bio" name="bio" rows="4"></textarea>

    <!-- Step 6: Checkbox and radio. -->
    <label>
      <input type="checkbox" name="newsletter" value="yes">
      Subscribe to newsletter
    </label>

    <!-- Step 7: Submit button. -->
    <button type="submit">Sign Up</button>
  </fieldset>
</form>
```

**Expected Output:** A structured form with grouped fields, labels, and a submit button.

**Why This Output:** Each control is labelled and named, so its value is included in the form submission. The `fieldset` groups related fields, and the `legend` provides a caption.

### Real-World Cases

- **Login/signup forms:** Email, password, and submission.
- **Checkout forms:** Billing address, payment details, order summary.
- **Search forms:** Query input with autocomplete.
- **Survey forms:** Radio buttons, checkboxes, and text areas.

---

## Form Submission

### Definitions

**Core Definition:** Form submission is the process of sending form data to a server (via GET or POST) or handling it with JavaScript.

**Technical Definition:** Per the HTML Living Standard, the form submission algorithm is triggered when a form is submitted (via a submit button, `form.submit()`, or Enter key in a single-input form). The form's `action` attribute specifies the URL, and the `method` attribute specifies the HTTP method (`get`, `post`, or `dialog`). The `enctype` attribute specifies how data is encoded (`application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`).

### Purposes

- To send collected user data to a server for processing.
- To navigate to a new page based on the form data (GET).
- To perform state-changing operations (POST).
- To handle form data entirely client-side using JavaScript.

### Syntax Rules and Structure

```html
<form action="/submit" method="post" enctype="multipart/form-data">
  <!-- form controls -->
  <button type="submit">Submit</button>
</form>
```

**Component Breakdown:**
- `action`: The URL to submit to. Defaults to the current URL.
- `method`: `get` (data in URL) or `post` (data in request body). Defaults to `get`.
- `enctype`: Encoding type. Use `multipart/form-data` for file uploads.
- `novalidate`: Disables built-in validation.
- `target`: Where to display the response (`_self`, `_blank`, etc.).

**Syntax Rules:**
- A submit button (`<button type="submit">` or `<input type="submit">`) triggers submission.
- Pressing Enter in a text input submits the form (if there's a submit button).
- `form.submit()` submits the form programmatically (bypasses validation and submit event).
- `form.requestSubmit()` triggers validation and the submit event.

### Annotated Code Examples

#### Example 1: Standard POST Submission

```html
<form action="/api/users" method="post">
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" required>

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>

  <button type="submit">Create User</button>
</form>
```

**Expected Output:** On submit, the browser sends a POST request to `/api/users` with the form data in the request body and navigates to the response.

**Why This Output:** The `action` and `method` attributes determine where and how the data is sent. `required` prevents submission if the fields are empty.

#### Example 2: JavaScript-Handled Submission (AJAX)

```js
// Step 1: Select the form.
const form = document.getElementById("signup");

// Step 2: Listen for submit.
form.addEventListener("submit", async (event) => {
  // Step 3: Prevent default browser submission.
  event.preventDefault();

  // Step 4: Collect form data.
  const data = new FormData(form);

  // Step 5: Send via fetch.
  try {
    const response = await fetch("/api/signup", {
      method: "POST",
      body: data,
    });
    if (response.ok) {
      console.log("Signup successful");
    }
  } catch (error) {
    console.error("Signup failed:", error);
  }
});
```

**Expected Output:** The form is submitted via AJAX without a page reload. The response is handled in JavaScript.

**Why This Output:** `event.preventDefault()` stops the browser's default submission. `FormData` collects the form's data. `fetch` sends it asynchronously.

### Real-World Cases

- **Single-page applications:** Submitting forms via AJAX to avoid page reloads.
- **File uploads:** Using `multipart/form-data` to upload files.
- **Search forms:** Using GET to submit a query as a URL parameter.
- **Multi-step forms:** Submitting each step and progressing to the next.

---

## Form Validation

### Definitions

**Core Definition:** Form validation is the process of checking that user input meets the required criteria before the form is submitted.

**Technical Definition:** Per MDN, form validation is the process of ensuring that the data entered by the user is correct and complete. HTML5 introduced built-in validation attributes and the Constraint Validation API, allowing browsers to validate forms without JavaScript. However, server-side validation is always required for security.

### Purposes

- To improve user experience by providing immediate feedback.
- To reduce server load by catching errors before submission.
- To ensure data integrity and consistency.
- To prevent security vulnerabilities (though server-side validation is still essential).

### Two Types of Validation

| Type | Where | Purpose |
|------|-------|---------|
| **Client-side** | Browser (HTML + JS) | UX, immediate feedback |
| **Server-side** | Server | Security, data integrity (always required) |

### Syntax Rules and Structure

```html
<input type="email" required>
<input type="text" minlength="3" maxlength="20">
<input type="number" min="1" max="100">
<input type="text" pattern="[A-Za-z]{3,}">
```

**Component Breakdown:**
- `required`: The field must be filled.
- `minlength` / `maxlength`: Minimum/maximum character length.
- `min` / `max`: Minimum/maximum numeric value.
- `pattern`: A regular expression the value must match.
- `type`: Determines the expected format (email, url, number, etc.).

### Annotated Code Example

```html
<form id="registration">
  <label for="username">Username (3-20 characters):</label>
  <input type="text" id="username" name="username"
         minlength="3" maxlength="20" required>

  <label for="age">Age (18+):</label>
  <input type="number" id="age" name="age"
         min="18" max="120" required>

  <label for="zip">ZIP Code (5 digits):</label>
  <input type="text" id="zip" name="zip"
         pattern="\d{5}" required>

  <button type="submit">Register</button>
</form>
```

**Expected Output:** The browser prevents submission and shows error messages if any field fails validation.

**Why This Output:** The `required`, `minlength`, `maxlength`, `min`, `max`, and `pattern` attributes trigger native browser validation.

### Real-World Cases

- **Signup forms:** Validating email format, password strength, and required fields.
- **Payment forms:** Validating card numbers, expiry dates, and CVC.
- **Address forms:** Validating ZIP codes, phone numbers, and country selection.
- **Survey forms:** Ensuring required questions are answered.

---

## Form Data

### Definitions

**Core Definition:** Form data is the collection of values entered into a form's controls, ready to be submitted or processed.

**Technical Definition:** When a form is submitted, the browser constructs a set of name/value pairs from the form's controls. Each successful control (one with a `name` attribute, not disabled, and not in a disabled fieldset) contributes its name and value. The `FormData` API provides a programmatic way to construct and manipulate this data.

### Purposes

- To collect user input for submission or processing.
- To send form data via `fetch` or `XMLHttpRequest`.
- To construct data for AJAX requests.
- To inspect and validate form data before submission.

### Accessing Form Data

```js
const form = document.getElementById("myForm");

// Method 1: Access individual controls.
const email = form.elements.email.value;

// Method 2: Use FormData.
const data = new FormData(form);

// Method 3: Use form.elements to iterate.
for (const element of form.elements) {
  console.log(element.name, element.value);
}
```

### Annotated Code Example

```js
const form = document.getElementById("signup");

form.addEventListener("submit", (event) => {
  event.preventDefault();

  // Collect all form data.
  const data = new FormData(form);

  // Access individual values.
  console.log(data.get("email"));
  console.log(data.get("password"));

  // Iterate over all entries.
  for (const [name, value] of data.entries()) {
    console.log(name, ":", value);
  }
});
```

**Expected Output:** The form's name/value pairs are logged to the console.

**Why This Output:** `FormData` collects all successful controls from the form. `get()` retrieves a single value; `entries()` iterates over all name/value pairs.

### Real-World Cases

- **AJAX form submission:** Sending form data without a page reload.
- **Form pre-filling:** Populating form fields from existing data.
- **Multi-step forms:** Collecting data across multiple steps.

---

## `FormData`

### Definitions

**Core Definition:** `FormData` is an API that provides a way to construct a set of key/value pairs representing form fields and their values, which can be sent using `fetch()` or `XMLHttpRequest`.

**Technical Definition:** Per MDN, the `FormData` interface provides a way to easily construct a set of key/value pairs representing form fields and their values, which can then be easily sent using the `fetch()` or `XMLHttpRequest.send()` method. It uses the same format a form would use if the encoding type were set to `multipart/form-data`.

### Purposes

- To collect form data programmatically.
- To send form data via AJAX (especially file uploads).
- To construct data for requests without an HTML form.
- To append additional data to a form before submission.

### Syntax Rules and Structure

```js
const formData = new FormData();
formData.append(name, value);
formData.append(name, blob, filename);

const formDataFromForm = new FormData(formElement);
```

**Component Breakdown:**
- `new FormData()`: Creates an empty `FormData` object.
- `new FormData(form)`: Creates a `FormData` object populated with the form's data.
- `append(name, value)`: Adds a new entry.
- `get(name)`, `getAll(name)`: Retrieves entries.
- `set(name, value)`: Sets or replaces an entry.
- `delete(name)`: Removes an entry.
- `has(name)`: Checks if an entry exists.
- `entries()`, `keys()`, `values()`: Iterators.

### Annotated Code Examples

#### Example 1: Collecting Form Data

```js
const form = document.getElementById("upload");
const data = new FormData(form);

// Access values:
console.log(data.get("title")); // Title field value
console.log(data.get("file"));  // File object

// Add extra data:
data.append("timestamp", Date.now());

// Send:
fetch("/api/upload", {
  method: "POST",
  body: data,
});
```

**Expected Output:** The form data (including the file) is sent to the server.

**Why This Output:** `FormData` handles file uploads automatically when used with `fetch`.

#### Example 2: Building FormData Manually

```js
const data = new FormData();
data.append("name", "Alice");
data.append("email", "alice@example.com");
data.append("avatar", fileInput.files[0]);

// Convert to URL-encoded string if needed:
const params = new URLSearchParams(data);
console.log(params.toString());
// "name=Alice&email=alice%40example.com"
```

**Expected Output:** The data is constructed and can be sent or converted.

### Real-World Cases

- **File uploads:** Sending files with form data.
- **AJAX forms:** Submitting forms without page reloads.
- **Multipart requests:** Sending data as `multipart/form-data`.
- **Form cloning:** Duplicating form data for multiple submissions.

---

## Input Types

### Definitions

**Core Definition:** Input types are the values of the `type` attribute on `<input>` elements that determine the kind of input control rendered and the kind of data accepted.

**Technical Definition:** Per MDN, the `<input>` element is one of the most powerful and complex in all of HTML due to the sheer number of combinations of input types and attributes. The `type` attribute determines the behaviour and appearance of the input.

### Common Input Types

| Type | Description | Validation |
|------|-------------|------------|
| `text` | Single-line text | `minlength`, `maxlength`, `pattern` |
| `password` | Masked text | `minlength`, `maxlength`, `pattern` |
| `email` | Email address | Email format |
| `url` | URL | URL format |
| `tel` | Telephone number | `pattern` (no built-in format) |
| `number` | Numeric input | `min`, `max`, `step` |
| `range` | Slider | `min`, `max`, `step` |
| `date` | Date picker | `min`, `max` |
| `time` | Time picker | `min`, `max` |
| `datetime-local` | Date and time | `min`, `max` |
| `month` | Month picker | `min`, `max` |
| `week` | Week picker | `min`, `max` |
| `color` | Colour picker | — |
| `file` | File upload | `accept`, `multiple` |
| `checkbox` | Checkbox | `checked` |
| `radio` | Radio button | `checked` |
| `hidden` | Hidden field | — |
| `submit` | Submit button | — |
| `reset` | Reset button | — |
| `button` | Generic button | — |

### Annotated Code Example

```html
<form>
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>

  <label for="birthday">Birthday:</label>
  <input type="date" id="birthday" name="birthday"
         min="1900-01-01" max="2026-12-31">

  <label for="rating">Rating (1-10):</label>
  <input type="range" id="rating" name="rating"
         min="1" max="10" value="5">

  <label for="avatar">Avatar:</label>
  <input type="file" id="avatar" name="avatar" accept="image/*">

  <button type="submit">Submit</button>
</form>
```

**Expected Output:** Each input renders with its type-specific control (date picker, slider, file selector, etc.) and enforces its type-specific validation.

### Real-World Cases

- **Signup forms:** Email, password, date of birth.
- **E-commerce:** Quantity (number), colour (color), file upload (file).
- **Surveys:** Range sliders, radio buttons, checkboxes.
- **Booking forms:** Date and time pickers.

---

## Input Validation

### Definitions

**Core Definition:** Input validation is the process of checking that user input meets specified criteria, using HTML attributes, the Constraint Validation API, or custom JavaScript.

**Technical Definition:** Per MDN, HTML5 introduced built-in form validation using attributes like `required`, `min`, `max`, `minlength`, `maxlength`, `pattern`, and `type`. The Constraint Validation API provides methods for custom validation logic.

### Validation Attributes

| Attribute | Applies To | Description |
|-----------|------------|-------------|
| `required` | All | Field must be filled |
| `minlength` / `maxlength` | Text inputs | Character length |
| `min` / `max` | Number, date, range | Numeric/date bounds |
| `step` | Number, range, date | Increment step |
| `pattern` | Text inputs | Regular expression match |
| `type` | Input | Built-in format validation |
| `accept` | File | File type restriction |
| `multiple` | File, email | Allow multiple values |

### Annotated Code Example

```html
<form id="payment">
  <label for="card">Card Number (16 digits):</label>
  <input type="text" id="card" name="card"
         pattern="\d{16}" required
         title="Please enter 16 digits">

  <label for="expiry">Expiry (MM/YY):</label>
  <input type="text" id="expiry" name="expiry"
         pattern="(0[1-9]|1[0-2])\/\d{2}" required
         placeholder="MM/YY">

  <label for="cvc">CVC (3-4 digits):</label>
  <input type="text" id="cvc" name="cvc"
         pattern="\d{3,4}" required>

  <button type="submit">Pay</button>
</form>
```

**Expected Output:** The browser prevents submission and shows error messages if any field fails validation.

**Why This Output:** The `pattern` attribute enforces the regular expression. The `title` attribute provides a hint message.

### Real-World Cases

- **Payment forms:** Card number, expiry, CVC validation.
- **Registration forms:** Username, email, password validation.
- **Address forms:** ZIP/postal code, phone number validation.
- **Custom validation:** Business-specific rules (e.g., age restrictions).

---

## Constraint Validation API

### Definitions

**Core Definition:** The Constraint Validation API is a set of DOM interfaces and methods that allow JavaScript to programmatically validate form controls and display custom error messages.

**Technical Definition:** Per MDN, the Constraint Validation API enables you to write custom validation logic and display custom error messages. It is available on form elements and provides properties like `validity`, `validationMessage`, `willValidate`, and methods like `checkValidity()`, `reportValidity()`, and `setCustomValidity()`.

### Key Properties

| Property | Description |
|----------|-------------|
| `validity` | A `ValidityState` object with boolean properties (see below) |
| `validationMessage` | The error message shown to the user |
| `willValidate` | Whether the element is a candidate for validation |

### `ValidityState` Properties

| Property | Description |
|----------|-------------|
| `valueMissing` | `required` field is empty |
| `typeMismatch` | Value doesn't match the type (e.g., email) |
| `patternMismatch` | Value doesn't match the `pattern` |
| `tooLong` / `tooShort` | Value exceeds `maxlength` / is below `minlength` |
| `rangeUnderflow` / `rangeOverflow` | Value is below `min` / above `max` |
| `stepMismatch` | Value doesn't match the `step` |
| `badInput` | Browser can't convert the value |
| `customError` | `setCustomValidity()` has been called with a non-empty string |
| `valid` | All validation checks pass |

### `checkValidity()`

**Definition:** `checkValidity()` returns `true` if the element's value is valid; otherwise, it returns `false` and fires an `invalid` event.

```js
const input = document.getElementById("email");
if (!input.checkValidity()) {
  console.log("Invalid:", input.validationMessage);
}
```

### `reportValidity()`

**Definition:** `reportValidity()` is similar to `checkValidity()`, but also displays the browser's validation UI to the user.

```js
const input = document.getElementById("email");
input.reportValidity(); // Shows the browser error message if invalid
```

### `setCustomValidity()`

**Definition:** `setCustomValidity()` sets a custom error message for the element. If the string is non-empty, the element is considered invalid.

```js
const password = document.getElementById("password");
const confirm = document.getElementById("confirm");

confirm.addEventListener("input", () => {
  if (confirm.value !== password.value) {
    confirm.setCustomValidity("Passwords do not match");
  } else {
    confirm.setCustomValidity(""); // Clear the error
  }
});
```

### Annotated Code Example: Full Validation

```js
const form = document.getElementById("signup");
const password = document.getElementById("password");
const confirm = document.getElementById("confirm");

// Step 1: Custom validation for password confirmation.
confirm.addEventListener("input", () => {
  if (confirm.value !== password.value) {
    confirm.setCustomValidity("Passwords do not match.");
  } else {
    confirm.setCustomValidity("");
  }
});

// Step 2: Handle form submission.
form.addEventListener("submit", (event) => {
  // Step 3: Check overall validity.
  if (!form.checkValidity()) {
    event.preventDefault();
    // Step 4: Show browser validation UI.
    form.reportValidity();
    return;
  }

  // Step 5: Proceed with submission.
  console.log("Form is valid!");
});
```

**Expected Output:** The form prevents submission if validation fails and shows the appropriate error messages.

**Why This Output:** `checkValidity()` returns `false` if any control is invalid. `reportValidity()` shows the browser's error messages.

### Real-World Cases

- **Password confirmation:** Using `setCustomValidity()` to check that passwords match.
- **Age verification:** Using `setCustomValidity()` to enforce a minimum age.
- **Dynamic validation:** Validating fields based on other field values.
- **Custom error messages:** Providing user-friendly error messages beyond the browser defaults.

---

## Form Events

### Definitions

**Core Definition:** Form events are DOM events emitted by form elements during user interaction, submission, and validation.

**Technical Definition:** Per the HTML Living Standard, form elements emit events during their lifecycle: `submit` (form submission), `reset` (form reset), `input` (value change), `change` (committed value change), `focus` (element gains focus), `blur` (element loses focus), and `invalid` (validation fails).

### Event Reference

| Event | Target | Description |
|-------|--------|-------------|
| `submit` | `<form>` | Fired when a form is submitted |
| `reset` | `<form>` | Fired when a form is reset |
| `input` | Input, textarea, select | Fired immediately when the value changes |
| `change` | Input, textarea, select | Fired when the value is committed (e.g., on blur for text inputs) |
| `focus` | Any element | Fired when an element gains focus (does not bubble) |
| `focusin` | Any element | Fired when an element gains focus (bubbles) |
| `blur` | Any element | Fired when an element loses focus (does not bubble) |
| `focusout` | Any element | Fired when an element loses focus (bubbles) |
| `invalid` | Form controls | Fired when a control fails validation |
| `select` | Input, textarea | Fired when text is selected |

### `submit`

**Definition:** Fired when a form is submitted. Can be cancelled with `preventDefault()`.

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();
  // Handle submission
});
```

### `reset`

**Definition:** Fired when a form is reset. Can be cancelled.

```js
form.addEventListener("reset", (event) => {
  if (!confirm("Reset the form?")) {
    event.preventDefault();
  }
});
```

### `input`

**Definition:** Fired immediately when the value of an input, textarea, or select changes.

```js
input.addEventListener("input", (event) => {
  console.log("Current value:", event.target.value);
});
```

### `change`

**Definition:** Fired when the value is committed. For text inputs, this is typically on blur; for selects, checkboxes, and radios, it's immediately on change.

```js
select.addEventListener("change", (event) => {
  console.log("Selected:", event.target.value);
});
```

### `focus` and `blur`

**Definition:** Fired when an element gains or loses focus. `focus` and `blur` do not bubble; use `focusin` and `focusout` for delegation.

```js
input.addEventListener("focus", () => {
  input.classList.add("focused");
});

input.addEventListener("blur", () => {
  input.classList.remove("focused");
});
```

### `invalid`

**Definition:** Fired when a form control fails validation.

```js
input.addEventListener("invalid", (event) => {
  console.log("Invalid:", event.target.validationMessage);
});
```

### Annotated Code Example: Complete Form Event Handling

```js
const form = document.getElementById("signup");
const email = document.getElementById("email");

// Real-time validation feedback.
email.addEventListener("input", () => {
  if (email.validity.typeMismatch) {
    email.setCustomValidity("Please enter a valid email address.");
  } else {
    email.setCustomValidity("");
  }
});

// Handle submission.
form.addEventListener("submit", async (event) => {
  event.preventDefault();

  if (!form.checkValidity()) {
    form.reportValidity();
    return;
  }

  const data = new FormData(form);
  try {
    const response = await fetch("/api/signup", {
      method: "POST",
      body: data,
    });
    console.log("Success:", response.status);
  } catch (error) {
    console.error("Error:", error);
  }
});

// Confirm reset.
form.addEventListener("reset", (event) => {
  if (!confirm("Are you sure you want to reset?")) {
    event.preventDefault();
  }
});
```

**Expected Output:** The form validates in real time, submits via AJAX, and confirms reset.

**Why This Output:** Each event handler responds to a specific user interaction, providing a complete form experience.

### Real-World Cases

- **Real-time validation:** Using `input` to validate as the user types.
- **AJAX submission:** Using `submit` with `preventDefault()`.
- **Confirmation dialogs:** Using `reset` to confirm before clearing.
- **Focus styling:** Using `focus` and `blur` to highlight the active field.

---

## Form Accessibility

### Definitions

**Core Definition:** Form accessibility is the practice of designing and implementing forms that can be used by everyone, including people with disabilities who use screen readers, keyboards, or other assistive technologies.

**Technical Definition:** Per MDN, accessible forms require: labels associated with controls, clear error messages, keyboard navigation, appropriate ARIA attributes, and semantic HTML. The Web Content Accessibility Guidelines (WCAG) provide the standards for accessible forms.

### Key Principles

| Principle | Description |
|-----------|-------------|
| **Labels** | Every control must have a label associated via `for`/`id` or wrapping |
| **Keyboard navigation** | All controls must be reachable and usable via keyboard |
| **Error identification** | Errors must be clearly identified and described |
| **Error suggestion** | Suggestions for fixing errors must be provided |
| **Grouping** | Related controls should be grouped with `<fieldset>` and `<legend>` |
| **ARIA attributes** | Use `aria-required`, `aria-invalid`, `aria-describedby` when needed |
| **Focus management** | Focus should move logically; errors should receive focus |

### Annotated Code Example: Accessible Form

```html
<form id="signup" novalidate>
  <fieldset>
    <legend>Account Information</legend>

    <!-- Step 1: Explicit label association. -->
    <label for="email">Email address (required):</label>
    <input type="email" id="email" name="email"
           required aria-required="true"
           aria-describedby="email-error">
    <!-- Step 2: Error message with id for aria-describedby. -->
    <span id="email-error" class="error" role="alert"></span>

    <!-- Step 3: Grouped radio buttons with fieldset. -->
    <fieldset>
      <legend>Preferred contact method</legend>
      <label>
        <input type="radio" name="contact" value="email" required>
        Email
      </label>
      <label>
        <input type="radio" name="contact" value="phone">
        Phone
      </label>
    </fieldset>

    <button type="submit">Sign Up</button>
  </fieldset>
</form>

<script>
const form = document.getElementById("signup");
const email = document.getElementById("email");
const emailError = document.getElementById("email-error");

// Step 4: Validate on submit and show accessible errors.
form.addEventListener("submit", (event) => {
  event.preventDefault();

  if (email.validity.valueMissing) {
    emailError.textContent = "Email is required.";
    email.setAttribute("aria-invalid", "true");
    email.focus(); // Move focus to the invalid field
    return;
  }

  if (email.validity.typeMismatch) {
    emailError.textContent = "Please enter a valid email address.";
    email.setAttribute("aria-invalid", "true");
    email.focus();
    return;
  }

  // Valid — proceed
  emailError.textContent = "";
  email.removeAttribute("aria-invalid");
  console.log("Form submitted");
});
</script>
```

**Expected Output:** The form is fully accessible: labels are associated, errors are announced via `role="alert"`, and focus moves to the invalid field.

**Why This Output:** The `aria-describedby` links the error message to the input. `role="alert"` announces the error to screen readers. `aria-invalid` indicates the field's state.

### Real-World Cases

- **Government forms:** Must meet WCAG 2.1 AA standards.
- **Healthcare forms:** Accessible to patients with disabilities.
- **E-commerce checkout:** Accessible to all users for legal compliance.
- **Public-facing forms:** Any form used by the general public.

---

## Form Best Practices

### Definitions

**Core Definition:** Form best practices are guidelines for building forms that are usable, accessible, secure, and performant.

### Key Practices

**1. Always Use Labels**

```html
<!-- GOOD: Explicit label -->
<label for="email">Email:</label>
<input type="email" id="email" name="email">

<!-- GOOD: Wrapping label -->
<label>
  Email:
  <input type="email" name="email">
</label>

<!-- BAD: No label -->
<input type="email" name="email" placeholder="Email">
```

**2. Use the Right Input Type**

```html
<!-- GOOD: Type-specific inputs -->
<input type="email" name="email">
<input type="tel" name="phone">
<input type="number" name="age" min="0">

<!-- BAD: Generic text input -->
<input type="text" name="email">
```

**3. Validate on Both Client and Server**

```js
// Client-side (for UX):
input.addEventListener("input", () => {
  input.setCustomValidity(input.validity.valid ? "" : "Invalid input");
});

// Server-side (for security):
// Always validate and sanitise on the server.
```

**4. Provide Clear Error Messages**

```html
<label for="password">Password:</label>
<input type="password" id="password" name="password"
       minlength="8" required
       aria-describedby="password-hint">
<span id="password-hint">Must be at least 8 characters.</span>
```

**5. Group Related Controls**

```html
<fieldset>
  <legend>Shipping Address</legend>
  <label for="street">Street:</label>
  <input type="text" id="street" name="street">
  <!-- ... -->
</fieldset>
```

**6. Use `novalidate` for Custom Validation**

```html
<form id="signup" novalidate>
  <!-- Custom validation logic in JavaScript -->
</form>
```

**7. Prevent Double Submission**

```js
form.addEventListener("submit", async (event) => {
  event.preventDefault();
  const submitButton = form.querySelector("button[type='submit']");
  submitButton.disabled = true;
  try {
    await submitForm();
  } finally {
    submitButton.disabled = false;
  }
});
```

**8. Handle Enter Key Appropriately**

```html
<!-- Single-input forms submit on Enter by default. -->
<!-- For multi-step forms, prevent Enter from submitting prematurely. -->
<input type="text" onkeydown="return event.key !== 'Enter'">
```

**9. Use Autocomplete Attributes**

```html
<input type="email" name="email" autocomplete="email">
<input type="password" name="password" autocomplete="current-password">
<input type="text" name="name" autocomplete="name">
```

**10. Sanitise and Validate on the Server**

```js
// NEVER trust client-side validation for security.
// Always validate and sanitise on the server.
```

### Real-World Cases

- **Login forms:** Autocomplete, clear errors, secure submission.
- **Checkout forms:** Grouped fields, progress indicators, accessible errors.
- **Survey forms:** Radio buttons, checkboxes, conditional logic.
- **Search forms:** Type-specific inputs, AJAX submission, debouncing.

---

## References

- MDN Web Docs – Web forms — https://mdn.org.cn/en-US/docs/Learn_web_development/Extensions/Forms
- MDN Web Docs – Form validation — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation
- MDN Web Docs – Constraint validation — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation
- MDN Web Docs – FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs – HTMLFormElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement
- MDN Web Docs – The HTML5 input types — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/HTML5_input_types
- MDN Web Docs – Sending form data — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data
- MDN Web Docs – HTMLInputElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement
- MDN Web Docs – checkValidity() — https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/checkValidity
- MDN Web Docs – reportValidity() — https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/reportValidity
- MDN Web Docs – setCustomValidity() — https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setCustomValidity
- MDN Web Docs – ValidityState — https://developer.mozilla.org/en-US/docs/Web/API/ValidityState
- MDN Web Docs – HTML forms accessibility — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/HTML_forms_accessibility
- WHATWG HTML Living Standard – Forms — https://html.spec.whatwg.org/multipage/forms.html
- W3C – Web Content Accessibility Guidelines (WCAG) — https://www.w3.org/WAI/standards-guidelines/wcag/
- OWASP – Input Validation Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP – Cross-Site Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
