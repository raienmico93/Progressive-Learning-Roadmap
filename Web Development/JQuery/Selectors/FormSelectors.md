# jQuery Form Selectors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery form selectors are a category of jQuery selector expressions that identify form-related elements — input fields, buttons, options, and select elements — based on their type, state, or role within a form.

**Technical Definition**
Form selectors are pseudo-class selectors (prefixed with `:`) that filter the DOM for elements related to HTML forms. They are **jQuery extensions** and not part of the CSS specification, meaning queries using these selectors cannot take advantage of the performance boost provided by the native DOM `querySelectorAll()` method. For better performance in modern browsers, the official documentation recommends using equivalent CSS attribute selectors such as `[type="checkbox"]` or `[type="password"]`.

**Beginner-Friendly Explanation**
When you build a form on a web page, you have many different kinds of elements: text boxes, checkboxes, radio buttons, dropdown menus, and so on. jQuery form selectors are shortcuts that let you find these elements by what they are. Instead of writing `$("input[type='checkbox']")`, you can write `$(":checkbox")`. These selectors make it easy to work with forms — like clearing all text fields, checking which checkboxes are selected, or enabling and disabling buttons.

### Key Characteristics

- **jQuery Extensions**: All form selectors (except `:checked`, `:disabled`, and `:enabled` which have CSS equivalents) are jQuery-specific extensions not found in the CSS specification.
- **Performance Trade-off**: Because they are jQuery extensions, form selectors cannot use native `querySelectorAll()` and are therefore slower than pure CSS selectors.
- **Prefixing Recommended**: The official documentation recommends preceding form selectors with a tag name or other selector (e.g., `$("input:checkbox")` instead of `$(":checkbox")`) to avoid the implied universal selector `*`.
- **State-Aware**: Selectors like `:checked`, `:selected`, `:disabled`, and `:enabled` filter elements based on their current state, which can change dynamically.
- **Form-Specific Scope**: These selectors are designed specifically for form elements and should only be used on elements that support the relevant attributes.

### Prerequisites

- Basic understanding of HTML forms and form elements (`<input>`, `<select>`, `<button>`, `<textarea>`).
- Familiarity with jQuery basic selectors and attribute selectors.
- Knowledge of HTML input types (`text`, `password`, `checkbox`, `radio`, `file`, `submit`, `button`).
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **Form Validation**: Selecting input fields by type for client-side validation.
- **Form Submission**: Handling submit buttons and serialising form data.
- **Dynamic Form Behaviour**: Enabling/disabling fields based on user interaction.
- **State Management**: Reading and updating checkbox, radio, and select values.

### Core Concepts / Features

1. `:input` — All Form Elements
2. `:text` — Text Inputs
3. `:password` — Password Inputs
4. `:email` — Email Inputs (via attribute selector)
5. `:checkbox` — Checkboxes
6. `:radio` — Radio Buttons
7. `:file` — File Inputs
8. `:submit` — Submit Buttons
9. `:button` — Button Elements
10. `:selected` — Selected Options
11. `:checked` — Checked Elements
12. `:disabled` — Disabled Elements
13. `:enabled` — Enabled Elements

---

## Core Concept 1: `:input` — All Form Elements

### Definitions

**Core Definition**
The `:input` selector selects all `<input>`, `<textarea>`, `<select>`, and `<button>` elements.

**Technical Definition**
`jQuery(":input")` returns a jQuery object containing every form control in the document, including input fields of all types, text areas, select dropdowns, and buttons. It is equivalent to the CSS selector `input, textarea, select, button`.

**Beginner-Friendly Explanation**
`:input` is the “everything in a form” selector. It finds every element you can type in or click on within a form — text boxes, checkboxes, dropdowns, buttons, and more.

### Purposes
- To select all form controls in a document for global form operations.
- To serialise or reset all form fields at once.
- To attach event handlers to every form element.
- To count or measure all form controls for diagnostic purposes.

### Syntax Rules and Structure

```javascript
$(":input")
$("form :input")
$("input:input")
```

**Component Breakdown**
- `:input` : No arguments; selects all form elements.
- Returns: A jQuery object containing all `<input>`, `<textarea>`, `<select>`, and `<button>` elements.

**Syntax Rules**
1. Can be used alone (`$(":input")`) or scoped to a form (`$("form :input")`).
2. The selector is a jQuery extension; it cannot use native `querySelectorAll()`.
3. For best performance, use a pure CSS selector first, then `.filter(":input")`.

**Constraints and Limitations**
- Slower than CSS selectors; avoid in performance-critical code.
- Includes elements outside forms if not scoped.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:input Selector — Count All Form Elements</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="text" placeholder="Name">
    <input type="email" placeholder="Email">
    <textarea placeholder="Message"></textarea>
    <select><option>Option 1</option></select>
    <button>Submit</button>
  </form>

  <script>
    $(function () {
      // Step 1: Select all form elements
      var $allInputs = $(":input");

      // Step 2: Log the count
      console.log("Total form elements:", $allInputs.length); // 5

      // Step 3: Highlight them
      $allInputs.css("border", "2px solid green");
    });
  </script>
</body>
</html>
```

**Expected Output**
- All five form elements receive a green border.
- Console output: `Total form elements: 5`

**Why This Output Occurs**
`$(":input")` matches every `<input>`, `<textarea>`, `<select>`, and `<button>` in the document. The form contains two inputs, one textarea, one select, and one button — five elements total.

### Real-World Cases
- **Form Reset**: `$("form :input").val("")` clears all text fields.
- **Form Serialisation**: `$("form :input").serialize()` collects all form data.
- **Event Delegation**: `$("form :input").on("change", validate)` validates on any field change.

### References
- :input Selector – https://api.jquery.com/input-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 2: `:text` — Text Inputs

### Definitions

**Core Definition**
The `:text` selector selects all `<input>` elements of type text.

**Technical Definition**
`jQuery(":text")` is equivalent to `jQuery("[type=text]")` and selects all `<input type="text">` elements. As of jQuery 1.5.2, it also selects input elements that have no specified `type` attribute, since `type="text"` is implied in that case.

**Beginner-Friendly Explanation**
`:text` finds all the standard single-line text boxes on a page — the ones where you type your name, address, or search query.

### Purposes
- To select all text inputs for styling, validation, or value manipulation.
- To attach input event handlers to text fields.
- To clear or read values from all text boxes.

### Syntax Rules and Structure

```javascript
$(":text")
$("input:text")
```

**Component Breakdown**
- `:text` : Selects all input elements of type text (or with no type attribute).
- Returns: A jQuery object containing matching text inputs.

**Syntax Rules**
1. Equivalent to `$('[type=text]')`.
2. As of jQuery 1.5.2, selects inputs with no `type` attribute.
3. For performance, use `$('input[type=text]')` instead.
4. Prefer `$("input:text")` over `$(":text")` to avoid the implied universal selector.

**Constraints and Limitations**
- Case-sensitive on the `type` attribute value.
- Does not select `<textarea>` elements.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:text Selector — Highlight Text Inputs</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="text" value="Name">
    <input type="email" value="email@example.com">
    <input type="text" value="Username">
    <input type="password" value="secret">
    <textarea>Message</textarea>
  </form>

  <script>
    $(function () {
      // Step 1: Select all text inputs
      var $textInputs = $("input:text");

      // Step 2: Log count and style
      console.log("Text inputs:", $textInputs.length); // 2
      $textInputs.css("background-color", "lightyellow");

      // Step 3: Log each value
      $textInputs.each(function () {
        console.log(this.value);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two text inputs receive a light yellow background.
- Console output:
```
Text inputs: 2
Name
Username
```

**Why This Output Occurs**
`$("input:text")` matches only `<input>` elements whose `type` is `"text"` (or omitted). The email, password, and textarea elements are excluded.

### Real-World Cases
- **Search Forms**: `$("input:text").val("")` clears all text search fields.
- **Form Validation**: `$("input:text").each(validateText)` validates each text input.
- **Placeholder Setup**: `$("input:text").attr("placeholder", "Enter text...")`.

### References
- :text Selector – https://api.jquery.com/text-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 3: `:password` — Password Inputs

### Definitions

**Core Definition**
The `:password` selector selects all `<input>` elements of type password.

**Technical Definition**
`jQuery(":password")` is equivalent to `jQuery("[type=password]")` and selects all `<input type="password">` elements. It is a jQuery extension and does not benefit from native `querySelectorAll()` performance.

**Beginner-Friendly Explanation**
`:password` finds all the password fields — the ones that show dots or asterisks instead of the actual characters you type.

### Purposes
- To select all password inputs for validation or strength checking.
- To clear password fields after form submission.
- To attach event handlers to password fields.

### Syntax Rules and Structure

```javascript
$(":password")
$("input:password")
```

**Component Breakdown**
- `:password` : Selects all input elements of type password.
- Returns: A jQuery object containing matching password inputs.

**Syntax Rules**
1. Equivalent to `$('[type=password]')`.
2. Prefer `$("input:password")` over `$(":password")`.
3. For performance, use `$('input[type=password]')`.

**Constraints and Limitations**
- Only matches `<input>` elements, not other form elements.
- Case-sensitive on the `type` attribute.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:password Selector — Password Field Styling</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="text" placeholder="Username">
    <input type="password" placeholder="Password">
    <input type="password" placeholder="Confirm Password">
  </form>

  <script>
    $(function () {
      // Step 1: Select all password inputs
      var $passwords = $("input:password");

      // Step 2: Log count and style
      console.log("Password fields:", $passwords.length); // 2
      $passwords.css("border", "2px solid red");

      // Step 3: Check if passwords match
      var pass1 = $passwords.eq(0).val();
      var pass2 = $passwords.eq(1).val();
      console.log("Passwords match:", pass1 === pass2);
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two password fields receive a red border.
- Console output:
```
Password fields: 2
Passwords match: true
```

**Why This Output Occurs**
`$("input:password")` matches only the two `<input type="password">` elements. The text input is excluded. Both password fields have empty values, so the comparison returns `true`.

### Real-World Cases
- **Password Strength Meters**: `$("input:password").on("input", checkStrength)`.
- **Form Reset**: `$("input:password").val("")` clears password fields.
- **Confirmation Validation**: `$("input:password").eq(0).val() === $("input:password").eq(1).val()`.

### References
- :password Selector – https://api.jquery.com/password-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 4: `:email` — Email Inputs (via Attribute Selector)

### Definitions

**Core Definition**
jQuery does not have a dedicated `:email` pseudo-selector. Email inputs must be selected using the attribute selector `[type="email"]`.

**Technical Definition**
Unlike `:text`, `:password`, and `:checkbox`, there is no jQuery extension pseudo-selector for `type="email"`. This is because HTML5 introduced the `email` input type, and jQuery's form selectors were designed before HTML5 was widely adopted. To select email inputs, use `$('input[type="email"]')` or `$("input[type=email]")`. This is a **pure CSS selector** and benefits from native `querySelectorAll()` performance.

**Beginner-Friendly Explanation**
There is no `:email` shortcut in jQuery. If you want to find email fields, you have to use the attribute selector: `$("input[type='email']")`. This is actually better for performance because the browser can find them natively.

### Purposes
- To select all email input fields for validation.
- To attach email-specific event handlers.
- To apply styles to email fields.

### Syntax Rules and Structure

```javascript
$("input[type='email']")
$("input[type=email]")
$("form input[type='email']")
```

**Component Breakdown**
- `input[type='email']` : A pure CSS attribute selector matching `<input type="email">`.
- Returns: A jQuery object containing matching email inputs.

**Syntax Rules**
1. There is **no** `:email` pseudo-selector in jQuery.
2. Use the attribute selector `[type="email"]` instead.
3. This is a CSS-compliant selector and benefits from native `querySelectorAll()`.
4. Quotes around `email` are optional but recommended for clarity.

**Constraints and Limitations**
- Browsers that do not support HTML5 may treat `type="email"` as `type="text"`.
- No jQuery extension shorthand exists.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:email — Attribute Selector for Email Inputs</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="text" placeholder="Name">
    <input type="email" placeholder="Email">
    <input type="email" placeholder="Confirm Email">
  </form>

  <script>
    $(function () {
      // Step 1: Select all email inputs using the attribute selector
      var $emails = $("input[type='email']");

      // Step 2: Log count and validate format
      console.log("Email fields:", $emails.length); // 2

      // Step 3: Check each email format
      $emails.each(function () {
        var value = $(this).val();
        var isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
        console.log("Email '" + value + "' valid:", isValid);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Console output:
```
Email fields: 2
Email '' valid: false
Email '' valid: false
```

**Why This Output Occurs**
`$("input[type='email']")` matches the two email inputs. Both are empty, so the validation regex returns `false` for each.

### Real-World Cases
- **Email Validation**: `$("input[type='email']").on("blur", validateEmail)`.
- **Form Serialisation**: `$("input[type='email']").val()` retrieves email addresses.
- **HTML5 Fallback**: Detecting whether the browser supports `type="email"`.

### References
- Attribute Equals Selector [name=”value”] – https://api.jquery.com/attribute-equals-selector/
- jQuery Bug Tracker — Add pseudo selectors for new HTML5 input types – https://bugs.jquery.com/ticket/9648

---

## Core Concept 5: `:checkbox` — Checkboxes

### Definitions

**Core Definition**
The `:checkbox` selector selects all `<input>` elements of type checkbox.

**Technical Definition**
`jQuery(":checkbox")` is equivalent to `jQuery("[type=checkbox]")` and selects all `<input type="checkbox">` elements. It is a jQuery extension and does not benefit from native `querySelectorAll()` performance.

**Beginner-Friendly Explanation**
`:checkbox` finds all the square boxes you can tick — the ones used for “I agree to the terms” or selecting multiple options.

### Purposes
- To select all checkboxes for state management or styling.
- To check or uncheck all checkboxes at once.
- To count how many checkboxes are checked.
- To attach change event handlers to checkboxes.

### Syntax Rules and Structure

```javascript
$(":checkbox")
$("input:checkbox")
```

**Component Breakdown**
- `:checkbox` : Selects all input elements of type checkbox.
- Returns: A jQuery object containing matching checkboxes.

**Syntax Rules**
1. Equivalent to `$('[type=checkbox]')`.
2. Prefer `$("input:checkbox")` over `$(":checkbox")`.
3. For performance, use `$('input[type=checkbox]')`.

**Constraints and Limitations**
- Does not select radio buttons; use `:radio` for those.
- Case-sensitive on the `type` attribute.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:checkbox Selector — Check All Checkboxes</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="checkbox" value="Option 1"> Option 1<br>
    <input type="checkbox" value="Option 2"> Option 2<br>
    <input type="checkbox" value="Option 3"> Option 3<br>
    <input type="radio" name="choice" value="A"> Radio A
  </form>

  <script>
    $(function () {
      // Step 1: Select all checkboxes
      var $checkboxes = $("input:checkbox");
      console.log("Checkboxes:", $checkboxes.length); // 3

      // Step 2: Check them all
      $checkboxes.prop("checked", true);

      // Step 3: Count checked
      console.log("Checked:", $("input:checked").length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output**
- All three checkboxes become checked.
- Console output:
```
Checkboxes: 3
Checked: 3
```

**Why This Output Occurs**
`$("input:checkbox")` matches only the three checkbox inputs. The radio button is excluded. `.prop("checked", true)` checks all of them, and `$("input:checked")` confirms the count.

### Real-World Cases
- **Select All**: `$("input:checkbox").prop("checked", true)` implements a “select all” feature.
- **Form Validation**: `$("input:checkbox:checked").length > 0` checks that at least one option is selected.
- **Dynamic Lists**: `$("input:checkbox").on("change", updateSummary)`.

### References
- :checkbox Selector – https://api.jquery.com/checkbox-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 6: `:radio` — Radio Buttons

### Definitions

**Core Definition**
The `:radio` selector selects all `<input>` elements of type radio.

**Technical Definition**
`jQuery(":radio")` is equivalent to `jQuery("[type=radio]")` and selects all `<input type="radio">` elements. It is a jQuery extension.

**Beginner-Friendly Explanation**
`:radio` finds all the round buttons where you can pick only one option from a group — like choosing your gender or selecting a payment method.

### Purposes
- To select all radio buttons for state management.
- To check which radio button is selected in a group.
- To attach change event handlers to radio buttons.
- To reset radio button selections.

### Syntax Rules and Structure

```javascript
$(":radio")
$("input:radio")
$("input[name=group]:radio")
```

**Component Breakdown**
- `:radio` : Selects all input elements of type radio.
- Returns: A jQuery object containing matching radio buttons.

**Syntax Rules**
1. Equivalent to `$('[type=radio]')`.
2. Prefer `$("input:radio")` over `$(":radio")`.
3. To select a group, combine with a name attribute selector: `$("input[name=gender]:radio")`.

**Constraints and Limitations**
- Does not select checkboxes; use `:checkbox` for those.
- Radio buttons in the same group share a `name` attribute.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:radio Selector — Detect Selected Radio</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="radio" name="gender" value="male"> Male<br>
    <input type="radio" name="gender" value="female"> Female<br>
    <input type="radio" name="plan" value="basic"> Basic<br>
    <input type="radio" name="plan" value="premium"> Premium
  </form>
  <div id="output"></div>

  <script>
    $(function () {
      // Step 1: Select all radio buttons
      var $radios = $("input:radio");
      console.log("Radio buttons:", $radios.length); // 4

      // Step 2: Listen for changes and log the selected value
      $radios.on("change", function () {
        var group = $(this).attr("name");
        var value = $(this).val();
        $("#output").text(group + ": " + value);
        console.log("Selected " + group + ": " + value);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Selecting a radio button updates the `#output` div.
- Console output (when "Female" is clicked): `Selected gender: female`

**Why This Output Occurs**
`$("input:radio")` selects all four radio buttons across both groups. The `change` event handler reads the `name` and `value` of the clicked radio button and displays the selection.

### Real-World Cases
- **Survey Forms**: `$("input:radio[name=rating]").on("change", submitRating)`.
- **Payment Selection**: `$("input:radio[name=payment]:checked").val()` retrieves the chosen method.
- **Form Reset**: `$("input:radio").prop("checked", false)` clears all radio selections.

### References
- :radio Selector – https://api.jquery.com/radio-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 7: `:file` — File Inputs

### Definitions

**Core Definition**
The `:file` selector selects all `<input>` elements of type file.

**Technical Definition**
`jQuery(":file")` is equivalent to `jQuery("[type=file]")` and selects all `<input type="file">` elements. It is a jQuery extension and cannot use native `querySelectorAll()`.

**Beginner-Friendly Explanation**
`:file` finds all the file upload fields — the ones with a “Browse” button that let users choose a file from their computer.

### Purposes
- To select all file upload fields for validation or styling.
- To check file size or type before upload.
- To attach change event handlers to file inputs.
- To clear file selections.

### Syntax Rules and Structure

```javascript
$(":file")
$("input:file")
```

**Component Breakdown**
- `:file` : Selects all input elements of type file.
- Returns: A jQuery object containing matching file inputs.

**Syntax Rules**
1. Equivalent to `$('[type=file]')`.
2. Prefer `$("input:file")` over `$(":file")`.
3. For performance, use `$('input[type=file]')`.

**Constraints and Limitations**
- File inputs are difficult to style consistently across browsers.
- The `.val()` method cannot set a file path for security reasons; it can only be used to clear the selection.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:file Selector — File Upload Validation</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="file" id="avatar" accept="image/*">
    <input type="text" placeholder="Not a file input">
  </form>
  <div id="output"></div>

  <script>
    $(function () {
      // Step 1: Select all file inputs
      var $fileInputs = $("input:file");
      console.log("File inputs:", $fileInputs.length); // 1

      // Step 2: Listen for file selection
      $fileInputs.on("change", function () {
        var file = this.files[0];
        if (file) {
          $("#output").text("Selected: " + file.name + " (" + file.size + " bytes)");
          console.log("File selected:", file.name, file.size);
        }
      });

      // Step 3: Clear file input
      $fileInputs.val("");
    });
  </script>
</body>
</html>
```

**Expected Output**
- When a file is selected, the `#output` div shows the file name and size.
- Console output: `File inputs: 1`

**Why This Output Occurs**
`$("input:file")` matches only the file input. The text input is excluded. The `change` event handler accesses the native `files` property to read file metadata.

### Real-World Cases
- **Image Upload Preview**: `$("input:file").on("change", previewImage)`.
- **File Size Validation**: Checking `this.files[0].size` before upload.
- **Drag-and-Drop Uploads**: Enhancing file inputs with custom drop zones.

### References
- :file Selector – https://api.jquery.com/file-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 8: `:submit` — Submit Buttons

### Definitions

**Core Definition**
The `:submit` selector selects all `<button>` and `<input>` elements of type submit.

**Technical Definition**
`jQuery(":submit")` selects all `<input type="submit">` and `<button type="submit">` elements. If a `<button>` element has no defined `type`, most browsers treat it as `type="submit"`. It is a jQuery extension.

**Beginner-Friendly Explanation**
`:submit` finds the buttons that submit a form — the “Send” or “Log In” buttons you click to send your data.

### Purposes
- To select submit buttons for event handling or disabling.
- To prevent double submissions by disabling the button.
- To apply styles to submit buttons.
- To programmatically trigger form submission.

### Syntax Rules and Structure

```javascript
$(":submit")
$("button:submit")
$("input:submit")
```

**Component Breakdown**
- `:submit` : Selects all button and input elements of type submit.
- Returns: A jQuery object containing matching submit buttons.

**Syntax Rules**
1. Selects both `<input type="submit">` and `<button type="submit">`.
2. Using `input:submit` will **not** select `<button>` elements.
3. For performance, use `$('button[type=submit], input[type=submit]')`.

**Constraints and Limitations**
- A `<button>` without a `type` attribute is treated as submit by most browsers but may not be matched by `:submit` in all browsers.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:submit Selector — Prevent Double Submission</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="myForm">
    <input type="text" placeholder="Name">
    <input type="submit" value="Submit">
    <button type="submit">Send</button>
    <button type="button">Cancel</button>
  </form>

  <script>
    $(function () {
      // Step 1: Select all submit buttons
      var $submitButtons = $(":submit");
      console.log("Submit buttons:", $submitButtons.length); // 2

      // Step 2: Disable on click to prevent double submission
      $submitButtons.on("click", function () {
        $(this).prop("disabled", true).val("Submitting...");
      });

      // Step 3: Re-enable after 3 seconds (simulated)
      setTimeout(function () {
        $submitButtons.prop("disabled", false).val("Submit");
      }, 3000);
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking either submit button disables it and changes its text to “Submitting…”.
- After 3 seconds, the buttons are re-enabled.
- Console output: `Submit buttons: 2`

**Why This Output Occurs**
`$(":submit")` matches both the `<input type="submit">` and the `<button type="submit">`. The `type="button"` cancel button is excluded.

### Real-World Cases
- **Prevent Double Submission**: `$(":submit").prop("disabled", true)`.
- **Loading States**: `$(":submit").val("Processing...")`.
- **Form Validation**: `$(":submit").on("click", validateForm)`.

### References
- :submit Selector – https://api.jquery.com/submit-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 9: `:button` — Button Elements

### Definitions

**Core Definition**
The `:button` selector selects all `<button>` elements and all `<input>` elements of type button.

**Technical Definition**
`jQuery(":button")` selects all `<button>` elements and all `<input type="button">` elements. An equivalent CSS selector is `button, input[type='button']`. It is a jQuery extension.

**Beginner-Friendly Explanation**
`:button` finds all buttons on the page — both `<button>` tags and `<input type="button">` elements.

### Purposes
- To select all buttons for event handling.
- To apply consistent styling to all buttons.
- To enable or disable all buttons at once.
- To count or measure buttons on a page.

### Syntax Rules and Structure

```javascript
$(":button")
$("button, input[type=button]")
```

**Component Breakdown**
- `:button` : Selects all `<button>` elements and `<input type="button">` elements.
- Returns: A jQuery object containing matching buttons.

**Syntax Rules**
1. Equivalent to `$("button, input[type='button']")`.
2. Includes `<button>` elements regardless of their `type` attribute.
3. For performance, use the CSS equivalent.

**Constraints and Limitations**
- Does not include `<input type="submit">` or `<input type="reset">` unless they are `<button>` elements.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:button Selector — Style All Buttons</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="button" value="Input Button">
    <button>Button Element</button>
    <input type="submit" value="Submit">
    <input type="reset" value="Reset">
  </form>

  <script>
    $(function () {
      // Step 1: Select all buttons
      var $buttons = $(":button");
      console.log("Buttons found:", $buttons.length); // 2

      // Step 2: Add a class to all buttons
      $buttons.addClass("styled").css({
        "padding": "8px 16px",
        "border-radius": "4px",
        "background": "#007bff",
        "color": "white"
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `<input type="button">` and `<button>` elements receive the button styling.
- The submit and reset inputs are unchanged.
- Console output: 
```
Buttons found: 2
```

**Why This Output Occurs**
`$(":button")` matches `<input type="button">` and `<button>` but excludes `<input type="submit">` and `<input type="reset">`.

### Real-World Cases
- **Toolbar Buttons**: `$(":button").addClass("toolbar-btn")`.
- **Bulk Enable/Disable**: `$(":button").prop("disabled", true)`.
- **Event Delegation**: `$(":button").on("click", handleClick)`.

### References
- :button Selector – https://api.jquery.com/button-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 10: `:selected` — Selected Options

### Definitions

**Core Definition**
The `:selected` selector selects all `<option>` elements that are currently selected in a `<select>` element.

**Technical Definition**
`jQuery(":selected")` selects all `<option>` elements that are selected. It works for `<option>` elements only and does **not** work for checkboxes or radio inputs; use `:checked` for those.

**Beginner-Friendly Explanation**
`:selected` finds the option that is currently chosen in a dropdown menu. If you have a “Country” dropdown, `:selected` finds the country the user picked.

### Purposes
- To retrieve the currently selected option from a dropdown.
- To get the text or value of the selected option.
- To detect changes in select elements.
- To pre-select an option programmatically.

### Syntax Rules and Structure

```javascript
$(":selected")
$("select option:selected")
$("select :selected")
```

**Component Breakdown**
- `:selected` : Selects all selected `<option>` elements.
- Returns: A jQuery object containing matching selected options.

**Syntax Rules**
1. Works only for `<option>` elements, not checkboxes or radios.
2. For best performance, use `$("select").find("option:selected")` or `.filter(":selected")`.
3. To get the selected value, use `.val()` on the `<select>` element.

**Constraints and Limitations**
- Does not work with checkboxes or radio buttons.
- For multi-select elements, returns all selected options.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:selected Selector — Get Selected Option</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <select id="garden" multiple>
    <option>Flowers</option>
    <option selected>Shrubs</option>
    <option>Trees</option>
    <option selected>Bushes</option>
    <option>Grass</option>
  </select>
  <div id="output"></div>

  <script>
    $(function () {
      // Step 1: Get all selected options
      var $selected = $("#garden option:selected");
      console.log("Selected count:", $selected.length); // 2

      // Step 2: Build a string of selected texts
      var str = "";
      $selected.each(function () {
        str += $(this).text() + " ";
      });

      // Step 3: Display the result
      $("#output").text(str); // "Shrubs Bushes "
      console.log("Selected:", str);
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#output` div displays “Shrubs Bushes”.
- Console output:
```
Selected count: 2
Selected: Shrubs Bushes 
```

**Why This Output Occurs**
`$("#garden option:selected")` matches the two `<option>` elements with the `selected` attribute. The `.each()` loop collects their text content.

### Real-World Cases
- **Country/State Dropdowns**: `$("#country option:selected").val()`.
- **Multi-Select Forms**: Collecting all selected values from a multi-select.
- **Dynamic Filtering**: `$("select").on("change", filterResults)`.

### References
- :selected Selector – https://api.jquery.com/selected-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 11: `:checked` — Checked Elements

### Definitions

**Core Definition**
The `:checked` selector matches all checkboxes, radio buttons, and options of select elements that are currently checked or selected.

**Technical Definition**
`jQuery(":checked")` works for checkboxes, radio buttons, and `<option>` elements of `<select>` elements. To retrieve only the selected options of select elements, use the `:selected` selector instead.

**Beginner-Friendly Explanation**
`:checked` finds anything that is currently ticked or selected — checkboxes that are ticked, radio buttons that are chosen, and options in a dropdown.

### Purposes
- To count how many checkboxes are checked.
- To identify which radio button is selected.
- To build arrays of selected values.
- To validate that at least one option is checked.

### Syntax Rules and Structure

```javascript
$(":checked")
$("input:checked")
$("input[type=checkbox]:checked")
$("select option:checked")
```

**Component Breakdown**
- `:checked` : Matches all checked or selected elements.
- Returns: A jQuery object containing matching elements.

**Syntax Rules**
1. Works for checkboxes, radio buttons, and select options.
2. For select options, the `:selected` selector is more specific.
3. Can be combined with `:checkbox` or `:radio` for specificity.

**Constraints and Limitations**
- Does not work on `<select>` elements themselves, only their `<option>` children.
- Does not work on arbitrary elements with a `checked` attribute.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:checked Selector — Count Checked Checkboxes</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input type="checkbox" name="newsletter" value="Hourly" checked> Hourly<br>
    <input type="checkbox" name="newsletter" value="Daily"> Daily<br>
    <input type="checkbox" name="newsletter" value="Weekly"> Weekly<br>
    <input type="checkbox" name="newsletter" value="Monthly" checked> Monthly<br>
    <input type="radio" name="fruit" value="apple" checked> Apple
  </form>
  <div id="output"></div>

  <script>
    $(function () {
      // Step 1: Count checked elements
      var count = $("input:checked").length;
      console.log("Checked elements:", count); // 3

      // Step 2: Update output on change
      var updateCount = function () {
        var n = $("input:checked").length;
        $("#output").text(n + (n === 1 ? " is" : " are") + " checked!");
      };

      updateCount();
      $("input[type=checkbox]").on("click", updateCount);
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#output` div displays “3 are checked!”.
- Clicking checkboxes updates the count.
- Console output: `Checked elements: 3`

**Why This Output Occurs**
`$("input:checked")` matches the two checked checkboxes and the one checked radio button. The count is 3.

### Real-World Cases
- **Select All Checkboxes**: `$("input:checkbox").prop("checked", true)`.
- **Form Submission**: `$("input:checked").each(function(){ values.push($(this).val()); })`.
- **Validation**: `if ($("input:checked").length === 0) alert("Please select an option");`.

### References
- :checked Selector – https://api.jquery.com/checked-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 12: `:disabled` — Disabled Elements

### Definitions

**Core Definition**
The `:disabled` selector selects all form elements that are currently disabled.

**Technical Definition**
`jQuery(":disabled")` matches elements that are actually disabled, meaning their boolean `disabled` property is `true`. This is subtly different from the `[disabled]` attribute selector, which only checks for the existence of the attribute regardless of its value. It should only be used on elements that support the `disabled` attribute: `<button>`, `<input>`, `<optgroup>`, `<option>`, `<select>`, `<textarea>`, `<menuitem>`, and `<fieldset>`.

**Beginner-Friendly Explanation**
`:disabled` finds all the form fields that are greyed out and cannot be used — the ones that are turned off.

### Purposes
- To select all disabled form elements for styling.
- To count or measure disabled fields.
- To detect which fields are disabled for form logic.
- To style disabled elements differently from enabled ones.

### Syntax Rules and Structure

```javascript
$(":disabled")
$("input:disabled")
$("form :disabled")
```

**Component Breakdown**
- `:disabled` : Selects all disabled form elements.
- Returns: A jQuery object containing matching disabled elements.

**Syntax Rules**
1. Matches elements whose `disabled` property is `true`.
2. Differs from `[disabled]` which only checks for the attribute's existence.
3. Should only be used on elements that support the `disabled` attribute.
4. Prefer `$("input:disabled")` over `$(":disabled")`.

**Constraints and Limitations**
- Using on elements that do not support `disabled` produces unexpected results.
- A disabled `<fieldset>` disables all its child form elements.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:disabled Selector — Style Disabled Fields</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input name="email" disabled="disabled" placeholder="Disabled">
    <input name="id" placeholder="Enabled">
    <button disabled>Disabled Button</button>
    <button>Enabled Button</button>
  </form>

  <script>
    $(function () {
      // Step 1: Select all disabled elements
      var $disabled = $(":disabled");
      console.log("Disabled elements:", $disabled.length); // 2

      // Step 2: Style them with a grey background
      $disabled.css("background-color", "#e0e0e0").css("color", "#999");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The disabled input and disabled button receive a grey background.
- Console output: `Disabled elements: 2`

**Why This Output Occurs**
`$(":disabled")` matches the input and button with the `disabled` attribute. The enabled input and button are excluded.

### Real-World Cases
- **Form Styling**: `$(":disabled").addClass("disabled-style")`.
- **Conditional Logic**: Checking if a field is disabled before submission.
- **Accessibility**: Adding `aria-disabled="true"` to disabled elements.

### References
- :disabled Selector – https://api.jquery.com/disabled-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Core Concept 13: `:enabled` — Enabled Elements

### Definitions

**Core Definition**
The `:enabled` selector selects all form elements that are currently enabled.

**Technical Definition**
`jQuery(":enabled")` matches elements that have their boolean `disabled` property strictly equal to `false`. This is subtly different from `:not([disabled])`, which selects elements that do not have a `disabled` attribute set regardless of its value. It should only be used on elements that support the `disabled` attribute.

**Beginner-Friendly Explanation**
`:enabled` finds all the form fields that are active and usable — the ones that are turned on.

### Purposes
- To select all enabled form elements for styling or event binding.
- To count or measure enabled fields.
- To enable or disable groups of elements conditionally.
- To style enabled elements differently from disabled ones.

### Syntax Rules and Structure

```javascript
$(":enabled")
$("input:enabled")
$("form :enabled")
```

**Component Breakdown**
- `:enabled` : Selects all enabled form elements.
- Returns: A jQuery object containing matching enabled elements.

**Syntax Rules**
1. Matches elements whose `disabled` property is `false`.
2. Differs from `:not([disabled])` which checks for the absence of the attribute.
3. Should only be used on elements that support the `disabled` attribute.
4. Prefer `$("input:enabled")` over `$(":enabled")`.

**Constraints and Limitations**
- Using on elements that do not support `disabled` produces unexpected results.
- A disabled `<fieldset>` makes all its child form elements disabled.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:enabled Selector — Enable All Fields</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form>
    <input name="email" disabled="disabled" placeholder="Disabled">
    <input name="id" placeholder="Enabled 1">
    <input name="name" placeholder="Enabled 2">
    <button>Enabled Button</button>
  </form>

  <script>
    $(function () {
      // Step 1: Select all enabled input fields
      var $enabled = $("input:enabled");
      console.log("Enabled inputs:", $enabled.length); // 2

      // Step 2: Add a placeholder to all enabled inputs
      $enabled.attr("placeholder", "This is enabled");

      // Step 3: Log the enabled elements
      $enabled.each(function () {
        console.log(this.name + ": " + this.placeholder);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two enabled inputs receive the placeholder “This is enabled”.
- Console output:
```
Enabled inputs: 2
id: This is enabled
name: This is enabled
```

**Why This Output Occurs**
`$("input:enabled")` matches only the two inputs that are not disabled. The disabled email input is excluded.

### Real-World Cases
- **Form Activation**: `$(":enabled").prop("disabled", false)` enables all fields.
- **Conditional Logic**: Checking if at least one field is enabled.
- **Styling**: `$("input:enabled").css("border", "1px solid green")`.

### References
- :enabled Selector – https://api.jquery.com/enabled-selector/
- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/

---

## Summary Table: jQuery Form Selectors at a Glance

| Selector | Selects | CSS Equivalent | Performance Note |
|---|---|---|---|
| `:input` | All input, textarea, select, button | `input, textarea, select, button` | Use `.filter(":input")` for speed |
| `:text` | `<input type="text">` | `input[type=text]` | Use CSS equivalent |
| `:password` | `<input type="password">` | `input[type=password]` | Use CSS equivalent |
| `:email` | — | `input[type=email]` | No jQuery extension exists |
| `:checkbox` | `<input type="checkbox">` | `input[type=checkbox]` | Use CSS equivalent |
| `:radio` | `<input type="radio">` | `input[type=radio]` | Use CSS equivalent |
| `:file` | `<input type="file">` | `input[type=file]` | Use CSS equivalent |
| `:submit` | `<input type="submit">`, `<button type="submit">` | `button[type=submit], input[type=submit]` | Use CSS equivalent |
| `:button` | `<button>`, `<input type="button">` | `button, input[type=button]` | Use CSS equivalent |
| `:selected` | Selected `<option>` elements | — (CSS) | Use `.filter(":selected")` |
| `:checked` | Checked checkboxes, radios, options | `:checked` (CSS) | Native CSS available |
| `:disabled` | Disabled form elements | `:disabled` (CSS) | Native CSS available |
| `:enabled` | Enabled form elements | `:enabled` (CSS) | Native CSS available |

---

## General References

- Form | jQuery API Documentation – https://api.jquery.com/category/selectors/form-selectors/
- :input Selector – https://api.jquery.com/input-selector/
- :text Selector – https://api.jquery.com/text-selector/
- :password Selector – https://api.jquery.com/password-selector/
- :checkbox Selector – https://api.jquery.com/checkbox-selector/
- :radio Selector – https://api.jquery.com/radio-selector/
- :file Selector – https://api.jquery.com/file-selector/
- :submit Selector – https://api.jquery.com/submit-selector/
- :button Selector – https://api.jquery.com/button-selector/
- :selected Selector – https://api.jquery.com/selected-selector/
- :checked Selector – https://api.jquery.com/checked-selector/
- :disabled Selector – https://api.jquery.com/disabled-selector/
- :enabled Selector – https://api.jquery.com/enabled-selector/
- jQuery Learning Center — Selecting Elements – https://learn.jquery.com/using-jquery-core/selecting-elements/
- MDN Web Docs — HTML Forms Guide – https://developer.mozilla.org/en-US/docs/Learn/Forms
- W3C — HTML Living Standard: Forms – https://html.spec.whatwg.org/multipage/forms.html