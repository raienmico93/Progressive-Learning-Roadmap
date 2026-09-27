# Constraint Validation API: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

The Constraint Validation API is a browser-native JavaScript interface that allows developers to programmatically check whether form controls satisfy their validation constraints and to customise validation messages.

**Technical Definition**

The Constraint Validation API is defined in the WHATWG HTML Living Standard under section 4.10.21.3. It provides properties and methods on form-associated elements (HTMLInputElement, HTMLButtonElement, HTMLFieldSetElement, HTMLObjectElement, HTMLOutputElement, HTMLSelectElement, HTMLTextAreaElement) that expose the constraint validation state. The `validity` property returns a `ValidityState` object whose Boolean properties indicate whether the element suffers from any of the defined validity states: `valueMissing`, `typeMismatch`, `patternMismatch`, `tooLong`, `tooShort`, `rangeUnderflow`, `rangeOverflow`, `stepMismatch`, `badInput`, `customError`, and `valid`. The API also provides the `checkValidity()`, `reportValidity()`, and `setCustomValidity()` methods, along with the `validationMessage` and `willValidate` properties.

**Beginner-Friendly Explanation**

The Constraint Validation API is how JavaScript talks to the browser‘s built-in form validation. When you mark an input as `required` or give it a `pattern`, the browser automatically checks it. The Constraint Validation API lets you ask the browser “Is this field valid? What’s wrong with it?” and even set your own custom error messages. It‘s the bridge between the HTML attributes you write and the JavaScript logic you need for complex forms.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Standards-based** | Defined in the WHATWG HTML Living Standard |
| **Live object** | The `ValidityState` object updates dynamically as the value changes |
| **Boolean properties** | Each validity state is a boolean (`true` = failing that constraint) |
| **Custom messages** | `setCustomValidity()` allows fully custom error messages |
| **Method-driven** | `checkValidity()` and `reportValidity()` trigger validation programmatically |
| **Not a security mechanism** | Client-side validation can be bypassed; server-side validation is still required |

---

### Prerequisites

- Basic familiarity with HTML forms and the `<input>` element
- Understanding of form validation attributes (`required`, `pattern`, `min`, `max`, etc.)
- Basic knowledge of JavaScript (event listeners, DOM properties)
- Awareness of the difference between client-side and server-side validation

---

### Related Programming Areas

- **HTML Client-Side Validation** – The attribute-based foundation that the API extends
- **Web Accessibility (A11y)** – Validation messages must be announced correctly
- **Form Submission** – Validation blocks submission when constraints fail
- **CSS Pseudo-Classes** – `:valid`, `:invalid`, `:out-of-range` reflect validity states
- **Server-Side Validation** – The essential companion to client-side checks

---

## Core Concepts / Features

---

### 1. Validity States

#### Definitions

**Core Definition**

Validity states are the individual Boolean properties of the `ValidityState` object that indicate which specific constraint (if any) an element’s value violates.

**Technical Definition**

The `ValidityState` interface represents the validity states that an element can be in with respect to constraint validation. Each property returns `true` if the element suffers from the corresponding validity problem, and `false` otherwise. The exception is the `valid` property, which returns `true` when the element meets all constraints. The `validity` property on a form-associated element returns a live `ValidityState` object.

**Beginner-Friendly Explanation**

When a field fails validation, the browser knows exactly why. The `ValidityState` object is like a checklist: `valueMissing` means the field is required but empty; `patternMismatch` means it doesn‘t match the pattern; `typeMismatch` means it’s not a valid email or URL; and so on. You can check these properties in JavaScript to understand what went wrong and respond accordingly.

#### Purposes

- To identify the specific reason an element failed validation
- To enable precise, contextual error messages
- To allow conditional logic based on the type of failure
- To supplement native browser validation messages

#### Validity State Properties

| Property | `true` When | Associated Attribute |
|---|---|---|
| `valueMissing` | A `required` field is empty | `required` |
| `typeMismatch` | Value doesn‘t match the `type` (email, URL) | `type` |
| `patternMismatch` | Value doesn’t match the `pattern` regex | `pattern` |
| `tooLong` | Value exceeds `maxlength` | `maxlength` |
| `tooShort` | Value is below `minlength` | `minlength` |
| `rangeUnderflow` | Value is below `min` | `min` |
| `rangeOverflow` | Value is above `max` | `max` |
| `stepMismatch` | Value doesn‘t fit the `step` rule | `step` |
| `badInput` | Browser cannot convert the input (e.g., text in number field) | — |
| `customError` | `setCustomValidity()` has set a non-empty message | — |
| `valid` | All constraints pass | — |

**Syntax Rules**

- Each property is read-only and boolean
- The `validity` object is live; it updates as the value changes
- `valid` is the only property where `true` means success

**Constraints and Limitations**

- `tooLong` is never `true` in Gecko-based browsers because the value is prevented from exceeding `maxlength`
- `badInput` is only `true` for certain input types (e.g., `number`, `date`) when the user enters unparseable text

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inspecting Validity States**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ValidityState Demo</title>
</head>
<body>
    <form id="myForm">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <button type="submit">Submit</button>
    </form>

    <script>
        const email = document.getElementById('email');

        email.addEventListener('input', function() {
            const v = this.validity;

            if (v.valueMissing) {
                console.log('The field is empty.');
            } else if (v.typeMismatch) {
                console.log('Not a valid email address.');
            } else if (v.valid) {
                console.log('The email is valid.');
            }
        });
    </script>
</body>
</html>
```

**Expected Output**

As the user types, the console logs the specific validity state.

**Why This Output Occurs**

The `validity` property returns a live `ValidityState` object. Each Boolean property indicates whether the corresponding constraint has failed.

---

**Example 2: Custom Error Based on Validity State**

```html
<input type="text" id="username" minlength="3" pattern="[a-zA-Z0-9]+">

<script>
    const username = document.getElementById('username');

    username.addEventListener('blur', function() {
        if (this.validity.tooShort) {
            this.setCustomValidity('Username must be at least 3 characters.');
        } else if (this.validity.patternMismatch) {
            this.setCustomValidity('Only letters and numbers are allowed.');
        } else {
            this.setCustomValidity('');
        }
    });
</script>
```

**Expected Output**

On blur, the field’s custom message updates based on which constraint failed.

**Why This Output Occurs**

The validity state properties allow conditional logic to set context-specific messages.

#### Real-World Cases

**Case 1: Password Strength Meters**

Password fields check `tooShort` and `patternMismatch` to display strength feedback.

**Case 2: Multi-Field Validation**

Forms use validity states to determine which field is causing the submission failure.

**Case 3: Conditional Error Messages**

Error messages are tailored to the specific failure (e.g., “Email is required” vs. “Email format is invalid”).

---

### 2. The `required` Attribute

#### Definitions

**Core Definition**

The `required` attribute specifies that a form control must have a value before the form can be submitted.

**Technical Definition**

The `required` attribute is a boolean attribute that applies to `<input>` (except `hidden`, `range`, `color`, `submit`, `reset`, `button`, `image`), `<select>`, and `<textarea>`. When present, the element is suffering from `valueMissing` if its value is the empty string. The `required` IDL attribute reflects the content attribute.

**Beginner-Friendly Explanation**

Adding `required` makes a field mandatory. If you try to submit the form without filling it in, the browser stops you and says “Please fill out this field.”

#### Purposes

- To ensure critical fields are not left empty
- To provide client-side validation without JavaScript
- To improve data quality

#### Syntax Rules and Structure

```html
<input type="text" name="username" required>
<select name="country" required>...</select>
<textarea name="message" required></textarea>
```

**Constraints and Limitations**

- For checkboxes, `required` means the box must be checked
- For radio groups, `required` means one of the group must be selected
- Disabled fields are not validated

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Required Text Field**

```html
<form action="/submit" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" required>
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Submitting with an empty username blocks submission and shows “Please fill out this field.”

**Why This Output Occurs**

The `required` attribute triggers the `valueMissing` validity state.

#### Real-World Cases

- **Registration forms** – Username and email fields
- **Checkout forms** – Shipping address fields
- **Surveys** – Mandatory questions

---

### 3. The `minlength` Attribute

#### Definitions

**Core Definition**

The `minlength` attribute specifies the minimum number of characters required in a text input.

**Technical Definition**

The `minlength` attribute applies to `<input>` types `text`, `search`, `url`, `tel`, `email`, `password`, and `<textarea>`. The value must be a non-negative integer. If the value‘s length (in code points) is less than the specified minimum, the element is suffering from `tooShort`.

**Beginner-Friendly Explanation**

The `minlength` attribute requires at least a certain number of characters. If the user types fewer, the field is invalid.

#### Purposes

- To enforce minimum length requirements (e.g., passwords)
- To improve data quality
- To provide validation without JavaScript

#### Syntax Rules and Structure

```html
<input type="text" name="username" minlength="3">
<textarea name="comment" minlength="10"></textarea>
```

**Constraints and Limitations**

- `minlength` only triggers validation on submission (not while typing)
- Character counts are based on UTF-16 code points

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Minimum Length**

```html
<form action="/register" method="post">
    <label for="username">Username (min 3 chars):</label>
    <input type="text" id="username" name="username" minlength="3" required>
    <button type="submit">Register</button>
</form>
```

**Expected Output**

Typing fewer than 3 characters and submitting shows “Please lengthen this text to 3 characters or more.”

**Why This Output Occurs**

The `minlength` attribute triggers `tooShort`.

#### Real-World Cases

- **Password fields** – Minimum length requirements
- **Review forms** – Minimum comment length

---

### 4. The `maxlength` Attribute

#### Definitions

**Core Definition**

The `maxlength` attribute specifies the maximum number of characters allowed in a text input.

**Technical Definition**

The `maxlength` attribute applies to the same input types as `minlength`. If the value’s length exceeds the specified maximum, the element is suffering from `tooLong`. However, browsers typically prevent the user from typing beyond the limit, so `tooLong` is rarely `true` in practice.

**Beginner-Friendly Explanation**

The `maxlength` attribute stops the user from typing more than a certain number of characters.

#### Purposes

- To limit input length for database or API constraints
- To improve data quality
- To provide immediate input restriction

#### Syntax Rules and Structure

```html
<input type="text" name="username" maxlength="20">
<textarea name="comment" maxlength="500"></textarea>
```

**Constraints and Limitations**

- `maxlength` prevents input beyond the limit in most browsers
- The `tooLong` validity state is rarely true because of this prevention

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Maximum Length**

```html
<label for="tweet">Tweet (max 280 chars):</label>
<textarea id="tweet" name="tweet" maxlength="280"></textarea>
```

**Expected Output**

The textarea prevents typing beyond 280 characters.

**Why This Output Occurs**

The `maxlength` attribute limits input in real time.

#### Real-World Cases

- **Social media posts** – Character limits
- **Database fields** – Length constraints

---

### 5. The `min` and `max` Attributes

#### Definitions

**Core Definition**

The `min` and `max` attributes define the minimum and maximum values for numeric and date/time inputs.

**Technical Definition**

The `min` and `max` attributes apply to `<input>` types `number`, `range`, `date`, `time`, `datetime-local`, `month`, and `week`. If the value is less than `min`, the element is suffering from `rangeUnderflow`; if greater than `max`, it is suffering from `rangeOverflow`.

**Beginner-Friendly Explanation**

The `min` and `max` attributes set the smallest and largest values allowed. For example, an age field might have `min="18"` and `max="120"`.

#### Purposes

- To constrain numeric input to a valid range
- To validate dates and times
- To provide slider and spinner controls with bounds

#### Syntax Rules and Structure

```html
<input type="number" name="age" min="18" max="120">
<input type="date" name="start" min="2026-01-01" max="2026-12-31">
```

**Constraints and Limitations**

- The browser does not prevent typing out-of-range values; validation occurs on submission
- For `range`, the slider constrains the value visually

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Age Range**

```html
<form action="/submit" method="post">
    <label for="age">Age (18-120):</label>
    <input type="number" id="age" name="age" min="18" max="120" required>
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Entering 15 or 130 triggers a validation error.

**Why This Output Occurs**

The `min` and `max` attributes trigger `rangeUnderflow` and `rangeOverflow`.

#### Real-World Cases

- **Age verification** – Minimum age requirements
- **Booking forms** – Date ranges

---

### 6. The `step` Attribute

#### Definitions

**Core Definition**

The `step` attribute specifies the granularity of numeric input — the intervals at which values can be incremented or decremented.

**Technical Definition**

The `step` attribute applies to the same input types as `min` and `max`. The value must be a positive floating-point number or the keyword `any`. If the value does not fit the step rule (i.e., it is not `min` plus an integral multiple of `step`), the element is suffering from `stepMismatch`.

**Beginner-Friendly Explanation**

The `step` attribute controls how much the value changes when you click the up/down arrows or drag a slider. For example, `step="10"` means the value goes 0, 10, 20, 30.

#### Purposes

- To control the increment of numeric inputs
- To enforce specific value granularity
- To allow any decimal value with `step="any"`

#### Syntax Rules and Structure

```html
<input type="number" name="quantity" min="0" max="100" step="5">
<input type="number" name="price" step="0.01">
```

**Constraints and Limitations**

- The value must be a multiple of `step` offset by `min`
- `step="any"` removes all stepping constraints

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Stepped Quantity**

```html
<label for="quantity">Quantity (steps of 5):</label>
<input type="number" id="quantity" name="quantity" min="0" max="100" step="5" value="10">
```

**Expected Output**

The arrows increment by 5. Entering 12 triggers a step mismatch error.

**Why This Output Occurs**

The `step="5"` attribute sets the valid increments.

#### Real-World Cases

- **Price inputs** – `step="0.01"` for cents
- **Time slots** – `step="900"` for 15-minute intervals

---

### 7. The `pattern` Attribute

#### Definitions

**Core Definition**

The `pattern` attribute specifies a regular expression that the input‘s value must match to be valid.

**Technical Definition**

The `pattern` attribute applies to `<input>` types `text`, `search`, `url`, `tel`, `email`, and `password`. The value is compiled as a JavaScript regular expression with the `u` (Unicode) flag. The value must match the entire pattern. If it does not, the element is suffering from `patternMismatch`.

**Beginner-Friendly Explanation**

The `pattern` attribute lets you require a specific format for a text field using a regular expression. For example, you can require a 5-digit ZIP code.

#### Purposes

- To enforce specific text formats (ZIP codes, phone numbers, IDs)
- To validate data without JavaScript
- To improve data quality

#### Syntax Rules and Structure

```html
<input type="text" name="zip" pattern="[0-9]{5}" title="Five digit ZIP code">
```

**Constraints and Limitations**

- The pattern must match the entire value
- The `title` attribute should describe the expected format
- Not a substitute for server-side validation

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

Typing “1234” triggers a pattern mismatch error.

**Why This Output Occurs**

The `pattern` attribute enforces the format.

#### Real-World Cases

- **Postal codes** – Country-specific formats
- **Phone numbers** – Local formats
- **Product codes** – SKU formats

---

### 8. The `type` Attribute (Intrinsic Constraints)

#### Definitions

**Core Definition**

The `type` attribute determines both the UI control and the intrinsic validation constraints for an `<input>` element.

**Technical Definition**

Certain input types have built-in constraints. `type="email"` requires a syntactically valid email address; `type="url"` requires an absolute URL. If the value does not match the required syntax, the element is suffering from `typeMismatch`. Other types like `number` also have intrinsic constraints (the value must be a valid floating-point number) and may trigger `badInput` if the browser cannot convert the input.

**Beginner-Friendly Explanation**

The `type` attribute does double duty: it changes the UI (e.g., a date picker for `type="date"`) and it adds automatic validation. For example, `type="email"` checks that the value looks like an email address.

#### Purposes

- To validate that input matches the expected data type
- To trigger type-specific mobile keyboards
- To provide a first line of defence against malformed data

#### Syntax Rules and Structure

```html
<input type="email" name="email">
<input type="url" name="website">
<input type="number" name="age">
```

**Constraints and Limitations**

- `type="tel"` has no intrinsic validation; use `pattern`
- `type="email"` accepts some technically invalid addresses (e.g., `a@b`)

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

Typing “not-an-email” triggers a type mismatch error.

**Why This Output Occurs**

The `type="email"` triggers the `typeMismatch` validity state.

#### Real-World Cases

- **Newsletter signup** – Email fields
- **Profile forms** – Website URL fields
- **Age verification** – Number fields

---

### 9. The Constraint Validation API Methods and Properties

#### Definitions

**Core Definition**

The Constraint Validation API provides methods (`checkValidity()`, `reportValidity()`, `setCustomValidity()`) and properties (`validity`, `validationMessage`, `willValidate`) to programmatically manage validation.

**Technical Definition**

The `checkValidity()` method returns `true` if the element meets all constraints and fires an `invalid` event if not. The `reportValidity()` method does the same but also displays the browser‘s validation message. The `setCustomValidity(message)` method sets a custom error message; passing an empty string clears the custom error. The `validationMessage` property returns the current error message (native or custom). The `willValidate` property returns `true` if the element is a candidate for constraint validation.

**Beginner-Friendly Explanation**

These are the tools you use in JavaScript to work with validation. `checkValidity()` asks “Is this valid?” `reportValidity()` asks the same but also shows the error message. `setCustomValidity()` lets you write your own error message.

#### Methods and Properties

| Member | Type | Description |
|---|---|---|
| `validity` | Property | Returns a `ValidityState` object |
| `validationMessage` | Property | The current error message (empty string if valid) |
| `willValidate` | Property | `true` if the element is validated on submission |
| `checkValidity()` | Method | Returns validity; fires `invalid` if false |
| `reportValidity()` | Method | Returns validity; displays message if false |
| `setCustomValidity(message)` | Method | Sets a custom error message |

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Using `checkValidity()` and `reportValidity()`**

```html
<form id="myForm">
    <label for="age">Age (18-65):</label>
    <input type="number" id="age" name="age" min="18" max="65" required>
    <button type="button" onclick="validate()">Validate</button>
    <button type="submit">Submit</button>
</form>

<script>
    function validate() {
        const age = document.getElementById('age');
        if (!age.checkValidity()) {
            age.reportValidity(); // Displays the browser message
        } else {
            console.log('Valid!');
        }
    }
</script>
```

**Expected Output**

Clicking “Validate” shows the browser message if the age is invalid.

**Why This Output Occurs**

`checkValidity()` returns false and `reportValidity()` displays the message.

---

**Example 2: Custom Validity Message**

```html
<input type="text" id="username" required>
<button type="button" onclick="setCustomMessage()">Set Custom Error</button>

<script>
    function setCustomMessage() {
        const username = document.getElementById('username');
        username.setCustomValidity('This username is already taken.');
        username.reportValidity();
    }
</script>
```

**Expected Output**

The browser shows the custom message instead of the native one.

**Why This Output Occurs**

`setCustomValidity()` overrides the native message and sets `customError` to `true`.

#### Real-World Cases

- **Custom validation logic** – Cross-field validation, server-side checks
- **Localised messages** – Translating error messages
- **Progressive enhancement** – Adding validation on top of native checks

---

### 10. Choosing the Right Validation Approach

#### Definitions

**Core Definition**

Choosing the right validation approach means selecting between native HTML validation, JavaScript-enhanced validation, or a hybrid, based on the complexity of the requirements.

**Technical Definition**

Native validation uses HTML attributes and the browser’s built-in UI. JavaScript-enhanced validation uses the Constraint Validation API to supplement or replace native behaviour. Hybrid approaches use native attributes for baseline checks and JavaScript for complex, cross-field, or custom-message validation.

**Beginner-Friendly Explanation**

Start with native HTML validation. If you need custom messages or complex rules, use the Constraint Validation API. Don‘t reinvent the wheel.

#### Decision Guide

| Requirement | Approach |
|---|---|
| Simple required/format checks | Native HTML attributes |
| Custom messages | `setCustomValidity()` |
| Complex rules (cross-field) | JavaScript with Constraint Validation API |
| Real-time feedback | JavaScript on `blur`/`input` |
| Custom UI | `novalidate` + JavaScript |

---

## References

- MDN Web Docs – Constraint validation – https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- MDN Web Docs – ValidityState – https://developer.mozilla.org/en-US/docs/Web/API/ValidityState
- MDN Web Docs – checkValidity() – https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/checkValidity
- MDN Web Docs – reportValidity() – https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/reportValidity
- MDN Web Docs – setCustomValidity() – https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setCustomValidity
- MDN Web Docs – Client-side form validation – https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- WHATWG HTML Living Standard – Constraint validation – https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#constraint-validation
- WHATWG HTML Living Standard – The constraint validation API – https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#the-constraint-validation-api
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.1: Error Identification – https://www.w3.org/WAI/WCAG21/Understanding/error-identification.html
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.3: Error Suggestion – https://www.w3.org/WAI/WCAG21/Understanding/error-suggestion.html