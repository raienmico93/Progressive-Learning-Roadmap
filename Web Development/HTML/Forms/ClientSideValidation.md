# HTML Client-Side Validation: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML client-side validation is the browser's native mechanism for checking user input against a set of constraints defined by HTML attributes before the form is submitted to a server.

**Technical Definition**

Client-side validation in HTML is implemented through the Constraint Validation API, a set of attributes, DOM properties, and methods defined by the WHATWG HTML Living Standard. Form controls can be constrained by attributes such as `required`, `min`, `max`, `minlength`, `maxlength`, `pattern`, and `step`. The browser evaluates these constraints against the control's value and marks the control as either valid or invalid. The `checkValidity()` and `reportValidity()` methods and the `validity` property expose the validation state to JavaScript. The `:valid`, `:invalid`, `:required`, `:optional`, `:in-range`, `:out-of-range`, and other CSS pseudo-classes provide visual feedback. Form submission is blocked if any control is invalid, and the browser displays a native validation message (tooltip).

**Beginner-Friendly Explanation**

Client-side validation is the browser's built-in way of checking that a form is filled in correctly before it's sent to the server. If a field is marked `required` and you leave it empty, the browser stops you from submitting and shows a message like "Please fill out this field." If you type letters into a number field, the browser complains. This saves time and reduces server load, but it is not a substitute for server-side validation (since users can bypass it).

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Declarative** | Constraints are defined with HTML attributes, not JavaScript |
| **Automatic** | The browser validates on form submission without extra code |
| **Accessible** | Native validation messages are announced by screen readers |
| **Not secure** | Client-side validation can be bypassed; always validate on the server |
| **CSS-driven feedback** | Pseudo-classes like `:valid` and `:invalid` style the states |
| **JavaScript-extensible** | The Constraint Validation API allows custom logic |
| **Localisable** | Browser validation messages are translated into the user's language |

---

### Prerequisites

- Basic familiarity with HTML forms and the `<input>` element
- Understanding of form attributes (`required`, `pattern`, `min`, `max`, `step`)
- Awareness of the `<form>` element and form submission
- Basic knowledge of CSS selectors and pseudo-classes
- Basic knowledge of regular expressions (for `pattern`)

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Validation messages must be announced correctly; `aria-describedby` supplements native messages
- **Constraint Validation API** – JavaScript API for custom validation
- **CSS** – Pseudo-classes provide visual feedback
- **Server-Side Validation** – The essential companion to client-side validation
- **Web Security** – Client-side validation does not replace server-side security checks
- **Internationalisation** – Browser messages are localised; custom messages must be translated

---

## Core Concepts / Features

---

### 1. Required Fields

#### Definitions

**Core Definition**

Required fields are form controls that must have a value before the form can be submitted, enforced by the `required` attribute.

**Technical Definition**

The `required` attribute is a boolean attribute that, when present on a form control, indicates that the user must specify a value before the form can be submitted. It applies to `<input>` types `text`, `search`, `url`, `tel`, `email`, `password`, `date`, `time`, `datetime-local`, `month`, `week`, `number`, `checkbox`, `radio`, and `file`, as well as `<select>` and `<textarea>`. The attribute is part of the Constraint Validation API and is reflected by the `required` IDL attribute and the `validity.valueMissing` property.

**Beginner-Friendly Explanation**

Adding `required` to a field makes it mandatory. If the user tries to submit the form without filling it in, the browser stops them and says "Please fill out this field."

#### Purposes

- To ensure critical fields are not left empty
- To provide client-side validation without JavaScript
- To improve data quality
- To satisfy WCAG Success Criterion 3.3.2 (Labels or Instructions)

#### Syntax Rules and Structure

**General Syntax**

```html
<input type="text" name="username" required>
<select name="country" required>...</select>
<textarea name="message" required></textarea>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `required` | Boolean attribute; presence enforces a value |
| Control | The form control to which the constraint applies |

**Syntax Rules**

- The attribute is boolean; no value is required
- It applies to most input types, `<select>`, and `<textarea>`
- It does not apply to `type="hidden"`, `type="range"`, `type="color"`, `type="submit"`, `type="reset"`, `type="button"`, `type="image"`
- For checkbox groups (e.g., terms and conditions), apply `required` to a single checkbox
- For radio groups, apply `required` to one radio in the group (or all radios)

**Constraints and Limitations**

- For `type="checkbox"`, `required` means the checkbox must be checked
- For `type="radio"`, `required` means one of the group must be selected
- Browser messages vary across browsers and locales

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Required Text Field**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Required Field Demo</title>
</head>
<body>
    <form action="/submit" method="post">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required>
        <button type="submit">Submit</button>
    </form>
</body>
</html>
```

**Expected Output**

Submitting with an empty username shows a browser validation message ("Please fill out this field" in English locales).

**Why This Output Occurs**

The `required` attribute triggers the `valueMissing` validity state when the field is empty. The browser blocks submission and displays the native validation message.

---

**Example 2: Required Checkbox**

```html
<form action="/register" method="post">
    <label>
        <input type="checkbox" name="terms" required>
        I agree to the terms and conditions
    </label>
    <button type="submit">Register</button>
</form>
```

**Expected Output**

Submitting without checking the box shows a message like "Please check this box if you want to proceed."

**Why This Output Occurs**

For checkboxes, `required` means the box must be checked. The browser blocks submission otherwise.

#### Real-World Cases

**Case 1: Registration Forms**

Username, email, and password fields are marked `required`.

**Case 2: Checkout Forms**

Shipping address, payment information, and terms acceptance are required.

**Case 3: Survey Forms**

Mandatory survey questions use `required` on radio buttons or selects.

---

### 2. Type Validation

#### Definitions

**Core Definition**

Type validation is the browser's automatic checking of an input's value against the expected format defined by the `type` attribute.

**Technical Definition**

The `type` attribute determines both the UI control and the validation rules for an `<input>`. For example, `type="email"` validates that the value matches the email address syntax (defined by a regular expression in the HTML specification), `type="url"` validates that the value is an absolute URL with a scheme, and `type="number"` validates that the value is a valid floating-point number. The browser sets the `validity.typeMismatch` property to `true` when the value does not match the expected type.

**Beginner-Friendly Explanation**

When you set an input to `type="email"`, the browser checks that what you typed looks like an email address. If it doesn't, the browser stops the form from submitting and shows a message.

#### Purposes

- To validate that input matches the expected data type
- To trigger type-specific mobile keyboards
- To provide a first line of defence against malformed data
- To improve the user experience by catching errors early

#### Syntax Rules and Structure

**General Syntax**

```html
<input type="email" name="email">
<input type="url" name="website">
<input type="number" name="age" min="0" max="120">
```

**Component Breakdown**

| Type | Validation Rule |
|---|---|
| `email` | Must match email syntax (basic format) |
| `url` | Must be an absolute URL with scheme |
| `number` | Must be a valid floating-point number |
| `date` | Must be a valid date string (`YYYY-MM-DD`) |
| `time` | Must be a valid time string (`HH:MM`) |
| `tel` | No type validation (use `pattern`) |

**Syntax Rules**

- Type validation occurs on form submission (not while typing, by default)
- The `multiple` attribute on `email` allows comma-separated addresses
- Use `pattern` for stricter format enforcement (e.g., `tel`)

**Constraints and Limitations**

- `type="tel"` does not validate phone format; use `pattern`
- `type="email"` accepts some technically invalid addresses (e.g., `a@b`)
- Browser validation messages for type mismatches vary

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Email Type Validation**

```html
<form action="/subscribe" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <button type="submit">Subscribe</button>
</form>
```

**Expected Output**

Typing "not-an-email" and submitting shows a browser message like "Please enter an email address."

**Why This Output Occurs**

The `type="email"` triggers the `typeMismatch` validity state for invalid values.

---

**Example 2: URL Type Validation**

```html
<form action="/submit" method="post">
    <label for="website">Website:</label>
    <input type="url" id="website" name="website" placeholder="https://example.com">
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Typing "example" (without a scheme) triggers a browser message like "Please enter a URL."

**Why This Output Occurs**

The `type="url"` requires an absolute URL with a scheme.

#### Real-World Cases

**Case 1: Newsletter Signup**

Email fields use `type="email"` to catch typos.

**Case 2: Profile Forms**

Website fields use `type="url"` to validate URL format.

**Case 3: Age Verification**

Age fields use `type="number"` with `min` and `max`.

---

### 3. Length Constraints

#### Definitions

**Core Definition**

Length constraints limit the number of characters a user can enter into a text field, enforced by the `minlength` and `maxlength` attributes.

**Technical Definition**

The `minlength` and `maxlength` attributes define the minimum and maximum number of UTF-16 code units (characters) allowed in a text-like input or `<textarea>`. The `maxlength` attribute prevents the user from typing more characters than the limit; the `minlength` attribute does not prevent typing but makes the input invalid if the value is shorter on submission. Both attributes are part of the Constraint Validation API and are reflected by the `validity.tooShort` and `validity.tooLong` properties.

**Beginner-Friendly Explanation**

The `maxlength` attribute stops the user from typing too many characters. The `minlength` attribute requires at least a certain number of characters.

#### Purposes

- To limit input length for database or API constraints
- To enforce minimum length requirements (e.g., passwords)
- To improve data quality
- To provide immediate feedback with `maxlength`

#### Syntax Rules and Structure

**General Syntax**

```html
<input type="text" name="username" minlength="3" maxlength="20">
<textarea name="comment" minlength="10" maxlength="500"></textarea>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `minlength` | Minimum number of characters |
| `maxlength` | Maximum number of characters |

**Syntax Rules**

- The attributes apply to text-like inputs and `<textarea>`
- `maxlength` prevents input beyond the limit
- `minlength` only triggers validation on submission
- Values must be non-negative integers

**Constraints and Limitations**

- `maxlength` does not prevent paste of longer text in all browsers
- `minlength` is not enforced until submission
- Character counts are based on UTF-16 code units

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Username Length**

```html
<form action="/register" method="post">
    <label for="username">Username (3-20 characters):</label>
    <input type="text" id="username" name="username" minlength="3" maxlength="20" required>
    <button type="submit">Register</button>
</form>
```

**Expected Output**

Typing fewer than 3 characters and submitting shows a message like "Please lengthen this text to 3 characters or more."

**Why This Output Occurs**

The `minlength` attribute triggers the `tooShort` validity state.

---

**Example 2: Comment Length**

```html
<label for="comment">Comment (max 500 characters):</label>
<textarea id="comment" name="comment" maxlength="500"></textarea>
```

**Expected Output**

The textarea prevents typing beyond 500 characters.

**Why This Output Occurs**

The `maxlength` attribute limits input in real time.

#### Real-World Cases

**Case 1: Password Requirements**

Password fields use `minlength="8"` to enforce minimum length.

**Case 2: Tweet-style Input**

Microblogging platforms use `maxlength` to limit post length.

**Case 3: Review Forms**

Review forms use `maxlength` to limit the length of comments.

---

### 4. Numeric Constraints

#### Definitions

**Core Definition**

Numeric constraints limit the value of a numeric input to a specific range and increment, enforced by the `min`, `max`, and `step` attributes.

**Technical Definition**

For numeric and date/time input types, the `min` and `max` attributes define the lower and upper bounds of the value, and the `step` attribute defines the increment. The browser validates that the value is within the range and conforms to the step (offset by `min`). The `validity.rangeUnderflow`, `validity.rangeOverflow`, and `validity.stepMismatch` properties expose these states.

**Beginner-Friendly Explanation**

The `min`, `max`, and `step` attributes constrain numeric inputs. For example, `min="1" max="10" step="2"` allows 1, 3, 5, 7, 9.

#### Purposes

- To constrain numeric input to a valid range
- To enforce increments (e.g., prices in 0.01 steps)
- To provide slider and spinner controls with bounds
- To validate dates and times

#### Syntax Rules and Structure

```html
<input type="number" name="quantity" min="1" max="10" step="1">
<input type="range" name="volume" min="0" max="100" step="10">
<input type="date" name="start" min="2026-01-01" max="2026-12-31">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `min` | Minimum value |
| `max` | Maximum value |
| `step` | Increment value |

**Syntax Rules**

- Applies to `number`, `range`, `date`, `time`, `datetime-local`, `month`, `week`
- The `step` value must be a positive number or `any`
- The valid values are `min + n × step`

**Constraints and Limitations**

- The browser does not prevent typing out-of-range values; validation occurs on submission
- For `range`, the slider constrains the value visually

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Quantity Range**

```html
<form action="/order" method="post">
    <label for="quantity">Quantity (1-10):</label>
    <input type="number" id="quantity" name="quantity" min="1" max="10" step="1" value="1" required>
    <button type="submit">Order</button>
</form>
```

**Expected Output**

Entering 0 or 11 triggers a message like "Value must be greater than or equal to 1."

**Why This Output Occurs**

The `min` and `max` attributes trigger `rangeUnderflow` and `rangeOverflow`.

---

**Example 2: Price with Step**

```html
<label for="price">Price ($0.01 steps):</label>
<input type="number" id="price" name="price" min="0" step="0.01" value="9.99">
```

**Expected Output**

Entering 9.995 triggers a message about the step mismatch.

**Why This Output Occurs**

The value must be a multiple of `step` offset by `min`.

#### Real-World Cases

**Case 1: E-Commerce Quantities**

Quantity selectors use `min="1"` to prevent zero.

**Case 2: Price Inputs**

Price fields use `step="0.01"` for cents.

**Case 3: Date Ranges**

Booking forms use `min` and `max` for check-in dates.

---

### 5. Pattern Validation (Regex)

#### Definitions

**Core Definition**

Pattern validation uses the `pattern` attribute to enforce that the input's value matches a specified regular expression.

**Technical Definition**

The `pattern` attribute contains a JavaScript regular expression (without the leading and trailing slash delimiters). The browser compiles the pattern with the `u` (Unicode) flag and requires that the entire value match the pattern. If the value does not match, the `validity.patternMismatch` property is set to `true`. The `title` attribute should describe the expected format; browsers may include it in the validation message.

**Beginner-Friendly Explanation**

The `pattern` attribute lets you require a specific format — like a 5-digit ZIP code or a phone number. The pattern is a regular expression.

#### Purposes

- To enforce specific text formats (ZIP codes, phone numbers, IDs)
- To validate data without JavaScript
- To provide a first line of defence against malformed input
- To improve data quality

#### Syntax Rules and Structure

```html
<input type="text" name="zip" pattern="[0-9]{5}" title="Five digit ZIP code">
<input type="tel" name="phone" pattern="[0-9\-\+\s\(\)]{7,20}" title="7-20 digits, spaces, dashes, parentheses">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `pattern` | Regular expression (without delimiters) |
| `title` | Description of the expected format |

**Syntax Rules**

- The pattern must match the entire value
- The pattern is compiled with the Unicode flag
- The `title` attribute is used in the validation message
- Only applies to text-like inputs (`text`, `search`, `url`, `tel`, `email`, `password`)

**Constraints and Limitations**

- Not all browsers support every regex feature
- Complex patterns are hard to communicate to users
- The `pattern` attribute is not a substitute for server-side validation

#### Annotated Complete Step-by-Step Code Examples

**Example 1: ZIP Code Pattern**

```html
<form action="/submit" method="post">
    <label for="zip">ZIP Code (5 digits):</label>
    <input type="text" id="zip" name="zip"
           pattern="[0-9]{5}" title="Five digit ZIP code" required>
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Typing "1234" and submitting shows a message like "Please match the requested format: Five digit ZIP code."

**Why This Output Occurs**

The `pattern` attribute triggers `patternMismatch` for values that don't match.

---

**Example 2: Username Pattern**

```html
<label for="username">Username (letters, numbers, underscores; 3-16 chars):</label>
<input type="text" id="username" name="username"
       pattern="[a-zA-Z0-9_]{3,16}"
       title="3-16 characters: letters, numbers, underscores">
```

**Expected Output**

Values like "jane-doe" (with a hyphen) trigger a validation error.

**Why This Output Occurs**

The pattern restricts the character set to letters, numbers, and underscores.

#### Real-World Cases

**Case 1: Postal Codes**

Country-specific postal code formats use `pattern`.

**Case 2: Phone Numbers**

Phone fields use `pattern` for local formats.

**Case 3: Product Codes**

Inventory systems use `pattern` for SKU formats.

---

### 6. Real-Time Validation (on-input/on-blur)

#### Definitions

**Core Definition**

Real-time validation is the practice of validating input as the user types (on `input`) or when the field loses focus (on `blur`), rather than waiting for form submission.

**Technical Definition**

Native HTML validation occurs on form submission, but JavaScript can trigger validation earlier by calling `checkValidity()` or `reportValidity()` on the `input` or `blur` events. The `input` event fires on every keystroke; the `blur` event fires when the field loses focus. Real-time validation is generally better for user experience when applied on `blur` (to avoid interrupting the user mid-typing) or debounced on `input` (to avoid excessive validation).

**Beginner-Friendly Explanation**

Real-time validation means checking the input as the user types or when they leave the field, rather than waiting until they click Submit. This gives faster feedback but can be annoying if it fires on every keystroke.

#### Purposes

- To provide immediate feedback to users
- To reduce submission errors
- To improve the user experience by catching errors early
- To validate dependent fields (e.g., password confirmation)

#### Syntax Rules and Structure

**General Syntax**

```html
<input type="text" id="username" name="username" required minlength="3"
       oninput="checkUsername(this)">
```

**JavaScript Pattern**

```javascript
const input = document.getElementById('username');
input.addEventListener('blur', function() {
    if (!this.checkValidity()) {
        this.reportValidity();
    }
});
```

**Component Breakdown**

| Event | When It Fires |
|---|---|
| `input` | On every keystroke |
| `blur` | When the field loses focus |
| `change` | When the value is committed (on blur for text inputs) |

**Syntax Rules**

- Use `blur` for the best balance of feedback and non-intrusiveness
- Use `input` with debouncing for live feedback
- Use `checkValidity()` to test validity without displaying a message
- Use `reportValidity()` to test and display the browser's message

**Constraints and Limitations**

- `input` validation on every keystroke can be annoying
- `blur` validation only fires after the user leaves the field
- Custom messages require `setCustomValidity()`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Blur Validation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Real-Time Validation</title>
    <style>
        input:invalid { border-color: red; }
        input:valid { border-color: green; }
    </style>
</head>
<body>
    <form action="/submit" method="post">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <button type="submit">Submit</button>
    </form>

    <script>
        const email = document.getElementById('email');

        email.addEventListener('blur', function() {
            if (!this.checkValidity()) {
                this.reportValidity();
            }
        });
    </script>
</body>
</html>
```

**Expected Output**

When the user leaves the email field with an invalid value, the browser shows a validation message.

**Why This Output Occurs**

The `blur` event triggers `checkValidity()`, and `reportValidity()` displays the message.

---

**Example 2: Live Input Validation**

```html
<label for="password">Password (min 8 characters):</label>
<input type="password" id="password" name="password" minlength="8" required>

<script>
    const password = document.getElementById('password');
    password.addEventListener('input', function() {
        if (this.value.length >= 8) {
            this.setCustomValidity('');
        } else {
            this.setCustomValidity('Password must be at least 8 characters.');
        }
    });
</script>
```

**Expected Output**

As the user types, the custom message updates.

**Why This Output Occurs**

The `input` event fires on every keystroke, and `setCustomValidity()` updates the message.

#### Real-World Cases

**Case 1: Password Strength Meters**

Real-time feedback on password strength uses the `input` event.

**Case 2: Username Availability**

Checking username availability on `blur` is a common pattern.

**Case 3: Credit Card Validation**

Card number validation on `input` uses the Luhn algorithm.

---

### 7. Form Submission Blocking

#### Definitions

**Core Definition**

Form submission blocking is the browser's automatic prevention of form submission when one or more controls fail validation.

**Technical Definition**

When a form is submitted, the browser runs the constraint validation algorithm on all submittable elements. If any control is invalid, the browser fires an `invalid` event at the first invalid control, displays the validation message, and aborts the submission. The `submit` event is not fired if validation fails. The `novalidate` attribute on the form or `formnovalidate` on a submit button bypasses this blocking.

**Beginner-Friendly Explanation**

If any field in your form is invalid, the browser won't let the form submit. It shows a message on the first invalid field and stops.

#### Purposes

- To prevent invalid data from reaching the server
- To enforce constraints automatically
- To provide immediate feedback to users
- To reduce server-side validation load

#### Syntax Rules and Structure

**Validation Flow**

| Step | Description |
|---|---|
| 1 | User activates submit |
| 2 | Browser runs constraint validation |
| 3 | If invalid, `invalid` event fires on first invalid control |
| 4 | Browser displays validation message |
| 5 | Submission is aborted |
| 6 | If valid, `submit` event fires and submission proceeds |

**Syntax Rules**

- Validation is skipped if the form has `novalidate`
- Validation is skipped if the submit button has `formnovalidate`
- The `invalid` event can be cancelled with `preventDefault()`
- Custom messages via `setCustomValidity()` override native messages

**Constraints and Limitations**

- Submission blocking is client-side only; server-side validation is still required
- Browser messages are not fully customisable without JavaScript

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Blocking**

```html
<form action="/submit" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Submitting with an empty or invalid email blocks submission and shows a message.

**Why This Output Occurs**

The browser's constraint validation blocks the submission.

---

**Example 2: Bypassing with `novalidate`**

```html
<form action="/submit" method="post" novalidate>
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <button type="submit">Submit (No Validation)</button>
</form>
```

**Expected Output**

The form submits even with an empty email.

**Why This Output Occurs**

The `novalidate` attribute disables constraint validation.

#### Real-World Cases

**Case 1: Multi-Step Forms**

Intermediate steps may use `novalidate` when not all fields are present.

**Case 2: Save Draft**

"Save Draft" buttons use `formnovalidate` to allow saving incomplete forms.

**Case 3: Testing**

Developers use `novalidate` to test server-side validation.

---

### 8. Browser Validation UI/Tooltips

#### Definitions

**Core Definition**

Browser validation UI is the native message (tooltip or bubble) displayed by the browser when a control fails validation.

**Technical Definition**

When validation fails, the browser focuses the first invalid control and displays a validation message. The message text is derived from the validity state (e.g., `valueMissing` → "Please fill out this field") and is localised into the browser's UI language. The message can be customised with `setCustomValidity()`. The UI is rendered by the browser and cannot be styled with CSS.

**Beginner-Friendly Explanation**

When a field fails validation, the browser shows a little pop-up message near the field explaining what's wrong. You can't change how it looks with CSS, but you can change the text with JavaScript.

#### Purposes

- To inform the user why the form cannot be submitted
- To guide the user to the invalid field
- To provide localised, accessible feedback
- To reduce confusion

#### Syntax Rules and Structure

**Native Message States**

| Validity State | Native Message (English) |
|---|---|
| `valueMissing` | Please fill out this field. |
| `typeMismatch` | Please enter an email address. / Please enter a URL. |
| `patternMismatch` | Please match the requested format. |
| `tooShort` | Please lengthen this text to N characters or more. |
| `tooLong` | Please shorten this text to N characters or less. |
| `rangeUnderflow` | Value must be greater than or equal to N. |
| `rangeOverflow` | Value must be less than or equal to N. |
| `stepMismatch` | Please enter a valid value. |
| `badInput` | Please enter a number. |
| `customError` | (Custom message) |

**Syntax Rules**

- Messages are localised by the browser
- The `title` attribute is appended to `patternMismatch` messages
- `setCustomValidity()` sets the `customError` state

**Constraints and Limitations**

- The UI cannot be styled with CSS
- The message text cannot be fully customised without `setCustomValidity()`
- Positioning is browser-controlled

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Custom Message**

```html
<form action="/submit" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" required
           oninvalid="this.setCustomValidity('Please choose a username.')"
           oninput="this.setCustomValidity('')">
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

The browser shows "Please choose a username." instead of the native message.

**Why This Output Occurs**

The `oninvalid` handler sets a custom message; the `oninput` handler clears it when the user types.

#### Real-World Cases

**Case 1: Localised Forms**

Forms serving multiple languages use `setCustomValidity()` to translate messages.

**Case 2: Context-Specific Messages**

Custom messages provide context (e.g., "Please enter your company email").

**Case 3: Multi-Field Validation**

Custom messages explain relationships between fields.

---

### 9. Pseudo-Classes (:valid, :invalid, :required, :optional, :out-of-range)

#### Definitions

**Core Definition**

CSS pseudo-classes are selectors that match form controls based on their validation state, enabling visual feedback without JavaScript.

**Technical Definition**

The CSS pseudo-classes `:valid`, `:invalid`, `:required`, `:optional`, `:in-range`, `:out-of-range`, `:read-only`, `:read-write`, `:disabled`, `:enabled`, `:checked`, `:indeterminate`, `:default`, `:placeholder-shown`, and `:user-invalid` match form controls based on their state. They are defined in the CSS Selectors Level 4 specification and the HTML Living Standard. The `:valid` and `:invalid` pseudo-classes match based on the constraint validation state.

**Beginner-Friendly Explanation**

CSS pseudo-classes let you style form fields based on whether they're valid, required, or out of range. For example, you can make invalid fields red and valid fields green.

#### Purposes

- To provide visual feedback without JavaScript
- To highlight required fields
- To indicate valid and invalid states
- To style controls based on their interaction state

#### Syntax Rules and Structure

**Common Pseudo-Classes**

| Pseudo-class | Matches |
|---|---|
| `:valid` | Controls that pass validation |
| `:invalid` | Controls that fail validation |
| `:required` | Controls with `required` |
| `:optional` | Controls without `required` |
| `:in-range` | Numeric controls within `min`/`max` |
| `:out-of-range` | Numeric controls outside `min`/`max` |
| `:read-only` | Controls with `readonly` |
| `:read-write` | Editable controls |
| `:disabled` | Disabled controls |
| `:enabled` | Enabled controls |
| `:checked` | Checked checkboxes/radios |
| `:indeterminate` | Indeterminate checkboxes |
| `:placeholder-shown` | Inputs showing placeholder |
| `:user-invalid` | Invalid after user interaction |

**Syntax Rules**

- Pseudo-classes are preceded by a colon in CSS
- They can be combined with element selectors
- `:user-invalid` is newer and less widely supported

**Constraints and Limitations**

- `:invalid` matches before user interaction, which can be visually noisy
- `:user-invalid` is preferred for post-interaction styling
- Styling may conflict with browser defaults

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Valid and Invalid Styling**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pseudo-Class Demo</title>
    <style>
        input:valid { border: 2px solid green; }
        input:invalid { border: 2px solid red; }
        input:required { background-color: #fffbe6; }
    </style>
</head>
<body>
    <form action="/submit" method="post">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
    </form>
</body>
</html>
```

**Expected Output**

The email field has a red border when empty (invalid) and turns green when a valid email is entered.

**Why This Output Occurs**

The `:invalid` and `:valid` pseudo-classes match based on the validation state.

---

**Example 2: Out-of-Range Styling**

```html
<style>
    input:out-of-range { border: 2px solid red; }
    input:in-range { border: 2px solid green; }
</style>

<label for="age">Age (18-120):</label>
<input type="number" id="age" name="age" min="18" max="120">
```

**Expected Output**

Entering 15 makes the border red; entering 30 makes it green.

**Why This Output Occurs**

The `:out-of-range` and `:in-range` pseudo-classes match based on the value.

#### Real-World Cases

**Case 1: Form Feedback**

Forms use `:valid` and `:invalid` to provide immediate visual feedback.

**Case 2: Required Field Highlighting**

Forms use `:required` to highlight mandatory fields.

**Case 3: Interactive Validation**

`:user-invalid` is used to style fields after the user has interacted with them.

---

### 10. Disabling Native UI via novalidate

#### Definitions

**Core Definition**

The `novalidate` attribute disables the browser's native constraint validation and UI, allowing custom validation to take over.

**Technical Definition**

The `novalidate` attribute is a boolean attribute on `<form>` elements. When present, the browser skips the constraint validation step during submission and does not display validation messages. The `formnovalidate` attribute on a submit button overrides `novalidate` for that specific submission. The `novalidate` attribute does not disable the Constraint Validation API; `checkValidity()` and `reportValidity()` still work.

**Beginner-Friendly Explanation**

Adding `novalidate` to a form turns off the browser's built-in validation. This is useful when you want to use your own JavaScript validation instead.

#### Purposes

- To disable native validation and use custom validation
- To allow form submission with invalid data for server-side testing
- To enable custom UI and messages
- To bypass validation for specific actions (e.g., save draft)

#### Syntax Rules and Structure

```html
<form action="/submit" method="post" novalidate>
    <input type="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `novalidate` | Boolean attribute on `<form>` |
| `formnovalidate` | Boolean attribute on submit button |

**Syntax Rules**

- The attribute is boolean
- It disables native validation and UI
- It can be overridden per button with `formnovalidate`
- The Constraint Validation API still works

**Constraints and Limitations**

- Disabling native validation does not improve security
- Server-side validation is still required
- Custom validation must replicate all native checks

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Custom Validation with novalidate**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Custom Validation</title>
</head>
<body>
    <form id="myForm" action="/submit" method="post" novalidate>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <span id="error" style="color: red;"></span>
        <button type="submit">Submit</button>
    </form>

    <script>
        const form = document.getElementById('myForm');
        const email = document.getElementById('email');
        const error = document.getElementById('error');

        form.addEventListener('submit', function(event) {
            if (!email.checkValidity()) {
                event.preventDefault();
                error.textContent = 'Please enter a valid email.';
                email.focus();
            } else {
                error.textContent = '';
            }
        });
    </script>
</body>
</html>
```

**Expected Output**

Submitting with an invalid email shows a custom error message and does not submit.

**Why This Output Occurs**

The `novalidate` attribute disables native validation. The custom handler uses `checkValidity()` to validate.

---

**Example 2: Save Draft with formnovalidate**

```html
<form action="/submit" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <button type="submit">Submit</button>
    <button type="submit" formnovalidate formaction="/draft">Save Draft</button>
</form>
```

**Expected Output**

"Submit" validates, "Save Draft" does not.

**Why This Output Occurs**

The `formnovalidate` attribute on the second button bypasses validation for that submission.

#### Real-World Cases

**Case 1: Custom Validation Libraries**

Applications using custom validation disable native UI to provide consistent messages.

**Case 2: Multi-Step Forms**

Intermediate steps disable validation when not all fields are present.

**Case 3: Save Draft**

Applications allow saving incomplete forms with `formnovalidate`.

---

### 11. Choosing the Right Validation Approach

#### Definitions

**Core Definition**

Choosing the right validation approach means selecting between native HTML validation, JavaScript-enhanced validation, or a hybrid, based on the requirements of the form.

**Technical Definition**

Native validation uses HTML attributes and the Constraint Validation API. JavaScript-enhanced validation supplements native validation with custom logic and messages. Hybrid approaches use native validation for baseline checks and JavaScript for advanced validation. The choice depends on the complexity of the validation rules, the desired user experience, and the need for custom messages.

**Beginner-Friendly Explanation**

Start with native HTML validation. It's simple and works without JavaScript. If you need custom messages or complex rules, add JavaScript on top.

#### Decision Guide

| Requirement | Approach |
|---|---|
| Simple required/format checks | Native HTML attributes |
| Custom messages | `setCustomValidity()` |
| Complex rules (cross-field) | JavaScript |
| Real-time feedback | JavaScript on `blur`/`input` |
| Custom UI | `novalidate` + JavaScript |
| Accessibility | Native validation + `aria-describedby` |

---

## References

- MDN Web Docs – Client-side form validation – https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- MDN Web Docs – Constraint validation – https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- MDN Web Docs – Constraint Validation API – https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation
- MDN Web Docs – `required` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/required
- MDN Web Docs – `pattern` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/pattern
- MDN Web Docs – `minlength` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/minlength
- MDN Web Docs – `maxlength` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/maxlength
- MDN Web Docs – `min` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/min
- MDN Web Docs – `max` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/max
- MDN Web Docs – `step` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/step
- MDN Web Docs – `novalidate` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#novalidate
- MDN Web Docs – `formnovalidate` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button#formnovalidate
- MDN Web Docs – `setCustomValidity()` – https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setCustomValidity
- MDN Web Docs – `checkValidity()` – https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/checkValidity
- MDN Web Docs – `reportValidity()` – https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/reportValidity
- MDN Web Docs – `validity` property – https://developer.mozilla.org/en-US/docs/Web/API/ValidityState
- MDN Web Docs – `:valid` CSS pseudo-class – https://developer.mozilla.org/en-US/docs/Web/CSS/:valid
- MDN Web Docs – `:invalid` CSS pseudo-class – https://developer.mozilla.org/en-US/docs/Web/CSS/:invalid
- MDN Web Docs – `:required` CSS pseudo-class – https://developer.mozilla.org/en-US/docs/Web/CSS/:required
- MDN Web Docs – `:optional` CSS pseudo-class – https://developer.mozilla.org/en-US/docs/Web/CSS/:optional
- MDN Web Docs – `:out-of-range` CSS pseudo-class – https://developer.mozilla.org/en-US/docs/Web/CSS/:out-of-range
- MDN Web Docs – `:user-invalid` CSS pseudo-class – https://developer.mozilla.org/en-US/docs/Web/CSS/:user-invalid
- WHATWG HTML Living Standard – Constraint validation – https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#constraint-validation
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.1: Error Identification – https://www.w3.org/WAI/WCAG21/Understanding/error-identification.html
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.2: Labels or Instructions – https://www.w3.org/WAI/WCAG21/Understanding/labels-or-instructions.html
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.3: Error Suggestion – https://www.w3.org/WAI/WCAG21/Understanding/error-suggestion.html
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.5: Identify Input Purpose – https://www.w3.org/WAI/WCAG21/Understanding/identify-input-purpose.html