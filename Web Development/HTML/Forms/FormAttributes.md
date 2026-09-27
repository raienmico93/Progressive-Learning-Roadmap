# HTML Form Attributes: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML form attributes are the modifiers applied to `<form>` elements and their associated controls (`<input>`, `<textarea>`, `<select>`, `<button>`, etc.) that control data submission, validation, user interaction, and initial state.

**Technical Definition**

Form attributes are content attributes defined by the WHATWG HTML Living Standard that modify the behaviour, constraints, and submission characteristics of form-associated elements. They are categorised as global attributes (available on all elements) or element-specific attributes (available only on particular elements or input types). Key form attributes include `name` (the key used in form submission), `value` (the current value), `placeholder` (hint text), `required` (constraint validation), `readonly` (non-editable but submitted), `disabled` (non-interactive and not submitted), `checked` (initial state for checkboxes and radios), `selected` (initial state for options), `multiple` (multi-value selection), `autocomplete` (autofill behaviour), `min`/`max`/`step` (numeric constraints), and `pattern` (regular expression validation).

**Beginner-Friendly Explanation**

Form attributes are the extra settings you add to form controls to control how they behave. Want a field to be mandatory? Add `required`. Want a text box to show grey hint text? Add `placeholder`. Want to limit a number to between 1 and 10? Add `min="1"` and `max="10"`. Each attribute does a specific job, and together they let you build forms that are validated, accessible, and easy to use.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Element-specific** | Different attributes apply to different elements and input types |
| **Validation-driven** | Attributes like `required`, `pattern`, `min`, `max`, and `step` trigger browser-native validation |
| **Submission-relevant** | `name` and `value` determine what data is sent to the server |
| **State-management** | `checked`, `selected`, `readonly`, and `disabled` control initial and interactive states |
| **Accessibility-critical** | `placeholder` is not a substitute for `<label>`; `disabled` fields are skipped by screen readers |
| **Boolean vs. enumerated** | Some attributes are boolean (`required`, `disabled`); others accept specific values |

---

### Prerequisites

- Basic familiarity with HTML document structure
- Understanding of the `<form>` element and form submission
- Awareness of the `<input>` element and its types
- Basic knowledge of HTML attributes and how they are written
- Basic knowledge of accessibility principles (helpful but not required)

---

### Related Programming Areas

- **HTML Forms** – Attributes are inseparable from form controls
- **Client-Side Validation** – Constraint validation uses attributes like `required`, `pattern`, `min`, `max`
- **Web Accessibility (A11y)** – `disabled` and `readonly` have significant accessibility implications
- **JavaScript** – The Constraint Validation API and DOM properties mirror attribute values
- **CSS** – Pseudo-classes like `:valid`, `:invalid`, `:required`, `:disabled` style based on attributes
- **Server-Side Processing** – `name` and `value` determine the data structure the server receives

---

## Core Concepts / Features

---

### 1. The `name` Attribute

#### Definitions

**Core Definition**

The `name` attribute defines the key under which a form control‘s value is submitted to the server.

**Technical Definition**

The `name` attribute represents the name of the form control. When a form is submitted, the name-value pairs of all submittable elements with a non-empty `name` attribute are included in the form data set. For radio buttons, the `name` attribute groups them into a single logical set. The `name` attribute is not the same as the `id` attribute; `id` is for labelling and scripting, while `name` is for submission.

**Beginner-Friendly Explanation**

The `name` attribute is like the label on a box when you mail it. When you submit a form, the server receives pairs like `name=John&email=john@example.com`. The `name` is “name” and “email,” and the values are what the user typed. Without a `name`, the control’s value is not submitted at all.

#### Purposes

- To identify the form control‘s data in the submission
- To group radio buttons into a single selection set
- To allow the server to reference the submitted value by a known key
- To enable JavaScript access via `form.elements.namedItem()`

#### Syntax Rules and Structure

**General Syntax**

```html
<input type="text" name="username">
<select name="country">...</select>
<textarea name="comment"></textarea>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `name` | Attribute name |
| `"username"` | The key used in submission and scripting |

**Syntax Rules**

- The `name` attribute is required for a control to be submitted
- The `name` value must not contain spaces or special characters if possible
- Radio buttons with the same `name` form a group
- The `name` attribute is not the same as `id`

**Constraints and Limitations**

- Controls without a `name` are not submitted
- Duplicate `name` values cause multiple values to be sent (useful for checkboxes)
- `name` is not used for label association; use `id` and `for` for that

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Named Text Input**

```html
<form action="/submit" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" value="johndoe">
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Submitting sends `username=johndoe`.

**Why This Output Occurs**

The `name="username"` attribute tells the browser to include this control in the form data with the key “username.” The `value` attribute provides the submitted value.

---

**Example 2: Radio Group with Shared Name**

```html
<form action="/submit" method="post">
    <input type="radio" name="color" value="red" id="red" checked>
    <label for="red">Red</label>
    <input type="radio" name="color" value="blue" id="blue">
    <label for="blue">Blue</label>
</form>
```

**Expected Output**

Only the selected radio button‘s value is submitted (e.g., `color=red`).

**Why This Output Occurs**

Both radio buttons share the same `name="color"`, making them a single group. Only the checked one is submitted.

#### Real-World Cases

**Case 1: Login Forms**

```html
<input type="text" name="username">
<input type="password" name="password">
```

**Case 2: Search Forms**

```html
<input type="search" name="q">
```

**Case 3: E-Commerce Checkout**

```html
<input type="text" name="street">
<input type="text" name="city">
<input type="text" name="zip">
```

---

### 2. The `value` Attribute

#### Definitions

**Core Definition**

The `value` attribute sets the initial or current value of a form control.

**Technical Definition**

The `value` attribute is the current value of the control. For `<input>`, `<textarea>` (via content), `<option>`, and `<button>`, the value attribute defines the data submitted with the form. For checkboxes and radio buttons, the `value` is only submitted if the control is checked. For submit buttons, the `value` is the button label. For `<input type="file">`, the `value` cannot be set programmatically.

**Beginner-Friendly Explanation**

The `value` attribute is what gets sent to the server. For a text input, it‘s the text the user typed. For a radio button, it’s the value associated with that option. For a submit button, it‘s the button’s label. You can set an initial value, and JavaScript can change it later.

#### Purposes

- To set the initial value of a control
- To define the submitted value for radio buttons, checkboxes, and buttons
- To provide default data for editing forms
- To allow JavaScript to read and write the control‘s current value

#### Syntax Rules and Structure

```html
<input type="text" name="username" value="johndoe">
<input type="radio" name="color" value="red">
<input type="submit" value="Send">
<option value="us">United States</option>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `value` | Attribute name |
| `"johndoe"` | The initial or submitted value |

**Syntax Rules**

- The `value` attribute is optional for most inputs
- For radio buttons and checkboxes, `value` is required for meaningful submission
- For submit/reset/button inputs, `value` sets the button label
- For `<input type="file">`, `value` cannot be set programmatically for security

**Constraints and Limitations**

- Setting `value` does not set the `defaultValue` property in the DOM for all cases
- For `<textarea>`, the initial value is the content, not a `value` attribute

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Pre-filled Text Input**

```html
<form action="/update" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" value="jane@example.com">
    <button type="submit">Update</button>
</form>
```

**Expected Output**

The email field is pre-filled with “jane@example.com.”

**Why This Output Occurs**

The `value` attribute provides the initial value.

---

**Example 2: Radio Button Values**

```html
<input type="radio" name="gender" value="female" id="f" checked>
<label for="f">Female</label>
<input type="radio" name="gender" value="male" id="m">
<label for="m">Male</label>
```

**Expected Output**

Selecting “Female” submits `gender=female`.

**Why This Output Occurs**

The `value` attribute defines what is submitted for each radio option.

#### Real-World Cases

**Case 1: Edit Forms**

Edit forms pre-fill inputs with the current values from the database.

**Case 2: E-Commerce Quantities**

```html
<input type="number" name="quantity" value="1" min="1">
```

**Case 3: Submit Buttons**

```html
<input type="submit" value="Place Order">
```

---

### 3. The `placeholder` Attribute

#### Definitions

**Core Definition**

The `placeholder` attribute provides a short hint that describes the expected value of an input, displayed when the field is empty.

**Technical Definition**

The `placeholder` attribute represents a hint (a word or short phrase) intended to aid the user with data entry when the control has no value. The hint is displayed in a lighter colour and disappears when the user starts typing. It is not a substitute for a `<label>` and should not be used as one.

**Beginner-Friendly Explanation**

A placeholder is the grey text that appears inside an empty input — like “Enter your email” or “Search...”. It gives users a hint about what to type. But it disappears when they start typing, so it cannot replace a proper `<label>`.

#### Purposes

- To provide a hint about the expected format or content
- To show an example value
- To supplement (not replace) a visible label

#### Syntax Rules and Structure

```html
<input type="text" name="username" placeholder="Enter username">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `placeholder` | Attribute name |
| `"Enter username"` | The hint text |

**Syntax Rules**

- The `placeholder` attribute applies to text-like inputs and `<textarea>`
- It should not contain line breaks
- It is not a substitute for a `<label>`

**Constraints and Limitations**

- Placeholder text is not announced consistently by screen readers
- It disappears when typing, causing cognitive load issues for some users
- Poor colour contrast can make placeholders hard to read

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Search Input**

```html
<label for="search">Search:</label>
<input type="search" id="search" name="q" placeholder="Search products...">
```

**Expected Output**

A search field with grey hint text “Search products...”

**Why This Output Occurs**

The `placeholder` attribute displays the hint when the field is empty.

#### Real-World Cases

**Case 1: Search Bars**

```html
<input type="search" name="q" placeholder="Search...">
```

**Case 2: Email Forms**

```html
<input type="email" name="email" placeholder="you@example.com">
```

**Case 3: Phone Forms**

```html
<input type="tel" name="phone" placeholder="+1 (555) 123-4567">
```

---

### 4. The `required` Attribute

#### Definitions

**Core Definition**

The `required` attribute specifies that a form control must have a value before the form can be submitted.

**Technical Definition**

The `required` attribute is a boolean attribute. When present, it indicates that the user must specify a value for the input before the owning form can be submitted. The attribute is part of the constraint validation API. If a required field is empty, the form cannot be submitted, and the browser displays a validation message.

**Beginner-Friendly Explanation**

The `required` attribute makes a field mandatory. If the user tries to submit the form without filling it in, the browser stops them and shows an error message.

#### Purposes

- To enforce that a field must be filled before submission
- To provide client-side validation without JavaScript
- To improve data quality
- To satisfy WCAG Success Criterion 3.3.2

#### Syntax Rules and Structure

```html
<input type="text" name="username" required>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `required` | Boolean attribute; no value required |

**Syntax Rules**

- The attribute applies to most input types and `<textarea>`
- It does not apply to `type="hidden"`, `type="range"`, `type="color"`, `type="submit"`, `type="reset"`, `type="button"`, or `type="image"`
- The attribute is boolean; its presence alone is sufficient

**Constraints and Limitations**

- Browser validation messages vary; custom validation may be needed for consistency
- Server-side validation is still required for security

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Required Email**

```html
<form action="/submit" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

Submitting without an email shows a browser validation error.

**Why This Output Occurs**

The `required` attribute triggers constraint validation.

#### Real-World Cases

**Case 1: Registration Forms**

```html
<input type="text" name="username" required>
<input type="email" name="email" required>
```

**Case 2: Checkout Forms**

```html
<input type="text" name="street" required>
```

**Case 3: Contact Forms**

```html
<textarea name="message" required></textarea>
```

---

### 5. The `readonly` Attribute

#### Definitions

**Core Definition**

The `readonly` attribute makes a form control non-editable while still allowing its value to be submitted.

**Technical Definition**

The `readonly` attribute is a boolean attribute. When present, it indicates that the user cannot edit the value of the control. However, the control‘s value is still included in form submission and can be focused and selected. The attribute applies to text-like inputs and `<textarea>`, but not to `type="hidden"`, `type="range"`, `type="color"`, `type="checkbox"`, `type="radio"`, `type="file"`, `type="submit"`, `type="reset"`, `type="button"`, or `type="image"`.

**Beginner-Friendly Explanation**

A readonly field is like a locked text box. The user can see the value and copy it, but they can’t change it. Unlike a disabled field, a readonly field‘s value is still submitted with the form.

#### Purposes

- To display a value that should not be changed
- To prevent editing while still submitting the value
- To show calculated or system-generated values

#### Syntax Rules and Structure

```html
<input type="text" name="userid" value="12345" readonly>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `readonly` | Boolean attribute; no value required |

**Syntax Rules**

- The attribute applies only to text-like inputs and `<textarea>`
- The control is focusable and its text is selectable
- The value is submitted with the form

**Constraints and Limitations**

- Readonly does not prevent JavaScript from changing the value
- Readonly fields should be visually distinguishable from editable fields

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Readonly User ID**

```html
<form action="/update" method="post">
    <label for="userid">User ID:</label>
    <input type="text" id="userid" name="userid" value="12345" readonly>
    <label for="name">Name:</label>
    <input type="text" id="name" name="name">
    <button type="submit">Update</button>
</form>
```

**Expected Output**

The User ID field shows “12345” but cannot be edited. Both `userid` and `name` are submitted.

**Why This Output Occurs**

The `readonly` attribute prevents editing but allows submission.

#### Real-World Cases

**Case 1: Record Editing**

```html
<input type="text" name="record_id" value="12345" readonly>
```

**Case 2: Calculated Totals**

```html
<input type="text" name="total" value="$99.99" readonly>
```

**Case 3: Timestamps**

```html
<input type="text" name="created_at" value="2026-09-27" readonly>
```

---

### 6. The `disabled` Attribute

#### Definitions

**Core Definition**

The `disabled` attribute makes a form control non-interactive, unfocusable, and excluded from form submission.

**Technical Definition**

The `disabled` attribute is a boolean attribute. When present, it indicates that the user cannot interact with the control, and the control is not focusable. Disabled controls are not submitted with the form and are not validated. The attribute can be applied to `<button>`, `<fieldset>`, `<input>`, `<optgroup>`, `<option>`, `<select>`, and `<textarea>`. When applied to a `<fieldset>`, all descendant form controls are disabled.

**Beginner-Friendly Explanation**

A disabled field is greyed out and cannot be clicked or typed into. Unlike readonly, a disabled field‘s value is **not** submitted with the form.

#### Purposes

- To prevent interaction with a control
- To exclude a control‘s value from submission
- To disable groups of controls via `<fieldset>`
- To indicate that a control is not currently applicable

#### Syntax Rules and Structure

```html
<input type="text" name="promo" disabled>
<fieldset disabled>
    <!-- all controls inside are disabled -->
</fieldset>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `disabled` | Boolean attribute; no value required |

**Syntax Rules**

- The attribute is boolean
- Disabled controls are not focusable
- Disabled controls are not submitted
- The `disabled` attribute on `<fieldset>` disables all descendants

**Constraints and Limitations**

- Disabled controls are skipped by screen readers
- Use `aria-disabled` if you need the control to be focusable but visually disabled
- Disabled controls cannot be validated

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Disabled Input**

```html
<form action="/submit" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" value="johndoe" disabled>
    <label for="email">Email:</label>
    <input type="email" id="email" name="email">
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

The Username field is greyed out and not submitted. Only the Email value is sent.

**Why This Output Occurs**

The `disabled` attribute prevents interaction and excludes the control from submission.

#### Real-World Cases

**Case 1: Dependent Fields**

Fields that are not applicable until an option is selected are disabled.

**Case 2: Non-Editable IDs**

```html
<input type="text" name="id" value="123" disabled>
```

**Case 3: Group Disabling**

```html
<fieldset disabled>
    <legend>Billing Address (same as shipping)</legend>
    ...
</fieldset>
```

---

### 7. The `checked` Attribute

#### Definitions

**Core Definition**

The `checked` attribute sets the initial state of a checkbox or radio button to selected.

**Technical Definition**

The `checked` attribute is a boolean attribute. When present on a checkbox or radio button, it indicates that the control is selected by default. The attribute reflects the `defaultChecked` property in the DOM; the current checked state is reflected by the `checked` IDL attribute, which can be changed by the user or JavaScript.

**Beginner-Friendly Explanation**

The `checked` attribute pre-selects a checkbox or radio button when the page loads. The user can still change it.

#### Purposes

- To pre-select a checkbox or radio button
- To set a default option in a group
- To reflect the initial state of a form

#### Syntax Rules and Structure

```html
<input type="checkbox" name="newsletter" checked>
<input type="radio" name="gender" value="female" checked>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `checked` | Boolean attribute; no value required |

**Syntax Rules**

- The attribute applies to `type="checkbox"` and `type="radio"`
- Only one radio button in a group can be `checked`
- Multiple checkboxes can be `checked`

**Constraints and Limitations**

- The `checked` attribute sets the default state; the user can change it
- Use the DOM `checked` property to read the current state

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Pre-Checked Checkbox**

```html
<label>
    <input type="checkbox" name="terms" checked>
    I agree to the terms and conditions
</label>
```

**Expected Output**

The checkbox is checked by default.

**Why This Output Occurs**

The `checked` attribute pre-selects the checkbox.

#### Real-World Cases

**Case 1: Newsletter Opt-In**

```html
<input type="checkbox" name="newsletter" checked>
```

**Case 2: Default Radio Selection**

```html
<input type="radio" name="plan" value="basic" checked>
```

**Case 3: Remember Me**

```html
<input type="checkbox" name="remember" checked>
```

---

### 8. The `selected` Attribute

#### Definitions

**Core Definition**

The `selected` attribute sets the initial selection of an `<option>` within a `<select>` element.

**Technical Definition**

The `selected` attribute is a boolean attribute on `<option>` elements. When present, it indicates that the option is selected by default. In a single-select `<select>`, only one option should have the `selected` attribute; if multiple are present, the last one wins. In a multi-select `<select>` (with the `multiple` attribute), multiple options can have `selected`.

**Beginner-Friendly Explanation**

The `selected` attribute pre-selects an option in a dropdown menu when the page loads.

#### Purposes

- To pre-select an option in a dropdown
- To set a default value for a `<select>`
- To reflect the current state of a form

#### Syntax Rules and Structure

```html
<select name="country">
    <option value="us">United States</option>
    <option value="uk" selected>United Kingdom</option>
    <option value="ca">Canada</option>
</select>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `selected` | Boolean attribute on `<option>` |

**Syntax Rules**

- Only one option should be `selected` in a single-select `<select>`
- Multiple options can be `selected` when `multiple` is present
- The `selected` attribute sets the default selection

**Constraints and Limitations**

- The `selected` attribute reflects the `defaultSelected` property
- The current selection is reflected by the `selected` IDL property

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Pre-Selected Option**

```html
<label for="country">Country:</label>
<select id="country" name="country">
    <option value="">-- Select --</option>
    <option value="us">United States</option>
    <option value="uk" selected>United Kingdom</option>
    <option value="ca">Canada</option>
</select>
```

**Expected Output**

“United Kingdom” is pre-selected in the dropdown.

**Why This Output Occurs**

The `selected` attribute on the UK option sets it as the default selection.

#### Real-World Cases

**Case 1: Country Selection**

```html
<option value="us" selected>United States</option>
```

**Case 2: Month Selection**

```html
<option value="09" selected>September</option>
```

**Case 3: Sort Order**

```html
<option value="price" selected>Price: Low to High</option>
```

---

### 9. The `multiple` Attribute

#### Definitions

**Core Definition**

The `multiple` attribute allows a `<select>` element or file input to accept multiple selections.

**Technical Definition**

The `multiple` attribute is a boolean attribute. When present on a `<select>` element, it indicates that multiple options can be selected. When present on `<input type="file">`, it allows multiple files to be selected. For `<select multiple>`, the control renders as a listbox rather than a dropdown, and the `size` attribute controls the number of visible rows.

**Beginner-Friendly Explanation**

The `multiple` attribute lets users select more than one option at a time — either multiple files or multiple items in a list.

#### Purposes

- To allow multiple selections in a listbox
- To allow multiple file uploads
- To collect multiple values under a single name

#### Syntax Rules and Structure

```html
<select name="skills" multiple size="5">
    <option value="html">HTML</option>
    <option value="css">CSS</option>
</select>

<input type="file" name="documents" multiple>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `multiple` | Boolean attribute |

**Syntax Rules**

- The attribute applies to `<select>` and `<input type="file">`
- For `<select multiple>`, use `size` to control visible rows
- Multiple selected options are submitted with the same `name`

**Constraints and Limitations**

- Multiple selections are harder to make on mobile devices
- Users must hold Ctrl/Cmd to select multiple options in a listbox

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Multiple Select**

```html
<label for="skills">Skills:</label>
<select id="skills" name="skills" multiple size="4">
    <option value="html">HTML</option>
    <option value="css">CSS</option>
    <option value="js">JavaScript</option>
    <option value="py">Python</option>
</select>
```

**Expected Output**

A listbox where the user can select multiple skills.

**Why This Output Occurs**

The `multiple` attribute allows multiple selections.

#### Real-World Cases

**Case 1: Skill Selection**

```html
<select name="skills" multiple>
```

**Case 2: File Uploads**

```html
<input type="file" name="photos" multiple accept="image/*">
```

**Case 3: Category Filters**

```html
<select name="categories" multiple>
```

---

### 10. The `autocomplete` Attribute

#### Definitions

**Core Definition**

The `autocomplete` attribute controls whether the browser can automatically fill in form values.

**Technical Definition**

The `autocomplete` attribute is an enumerated attribute that controls how the browser handles autofill for a form or input. It accepts values like `on` (enable autofill), `off` (disable autofill), or specific tokens like `name`, `email`, `username`, `current-password`, `new-password`, `street-address`, `postal-code`, and many others. The attribute can be set on a `<form>` (applying to all controls) or on individual controls. Specific tokens help browsers and password managers understand the purpose of the field.

**Beginner-Friendly Explanation**

The `autocomplete` attribute tells the browser whether it can automatically fill in a field. You can use it to enable autofill for convenience, or disable it for sensitive fields. You can also use specific values like `email` or `new-password` to help browsers understand what the field is for.

#### Purposes

- To enable or disable browser autofill
- To help browsers and password managers identify fields
- To improve user experience by reducing typing
- To comply with WCAG Success Criterion 1.3.5 (Identify Input Purpose)

#### Syntax Rules and Structure

```html
<input type="email" name="email" autocomplete="email">
<input type="password" name="password" autocomplete="current-password">
<input type="password" name="new-password" autocomplete="new-password">
<input type="text" name="username" autocomplete="username">
```

**Common Autocomplete Tokens**

| Token | Use Case |
|---|---|
| `name` | Full name |
| `email` | Email address |
| `username` | Username |
| `current-password` | Login password |
| `new-password` | Registration password |
| `street-address` | Street address |
| `postal-code` | Postal/ZIP code |
| `cc-number` | Credit card number |
| `tel` | Telephone number |
| `off` | Disable autofill |
| `on` | Enable autofill (generic) |

**Syntax Rules**

- The attribute can be set on `<form>` or individual controls
- Values like `off` and `on` are generic; specific tokens are more helpful
- The attribute is case-insensitive

**Constraints and Limitations**

- Browsers may ignore `autocomplete="off"` for certain fields (e.g., passwords)
- Specific tokens are more reliable for password managers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Login Form with Autocomplete**

```html
<form action="/login" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" autocomplete="username">

    <label for="password">Password:</label>
    <input type="password" id="password" name="password" autocomplete="current-password">

    <button type="submit">Log In</button>
</form>
```

**Expected Output**

The browser offers to autofill the username and password.

**Why This Output Occurs**

The `autocomplete` tokens tell the browser what each field is for.

#### Real-World Cases

**Case 1: Login Forms**

```html
autocomplete="username"
autocomplete="current-password"
```

**Case 2: Registration Forms**

```html
autocomplete="email"
autocomplete="new-password"
```

**Case 3: Checkout Forms**

```html
autocomplete="street-address"
autocomplete="postal-code"
autocomplete="cc-number"
```

---

### 11. The `min` and `max` Attributes

#### Definitions

**Core Definition**

The `min` and `max` attributes define the minimum and maximum values for numeric and date inputs.

**Technical Definition**

The `min` and `max` attributes define the range of values for `<input>` elements of type `number`, `range`, `date`, `time`, `datetime-local`, `month`, and `week`. They are part of the constraint validation API. If the value is less than `min` or greater than `max`, the input is invalid.

**Beginner-Friendly Explanation**

The `min` and `max` attributes set the smallest and largest values a user can enter. For example, a quantity field might have `min="1"` and `max="10"`.

#### Purposes

- To constrain numeric input to a valid range
- To provide validation without JavaScript
- To guide users toward acceptable values
- To enable browser-native validation

#### Syntax Rules and Structure

```html
<input type="number" name="quantity" min="1" max="10">
<input type="range" name="volume" min="0" max="100">
<input type="date" name="start" min="2026-01-01" max="2026-12-31">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `min` | Minimum allowed value |
| `max` | Maximum allowed value |

**Syntax Rules**

- The attribute applies to numeric and date/time input types
- The value must be a valid number or date string
- The input is invalid if the value is outside the range

**Constraints and Limitations**

- The `min`/`max` attributes do not prevent typing out-of-range values; validation occurs on submission
- For `type="range"`, the slider constrains the value

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Number Range**

```html
<label for="quantity">Quantity (1-10):</label>
<input type="number" id="quantity" name="quantity" min="1" max="10" value="1">
```

**Expected Output**

A numeric input constrained to 1–10.

**Why This Output Occurs**

The `min` and `max` attributes define the valid range.

#### Real-World Cases

**Case 1: Quantity Selectors**

```html
<input type="number" name="quantity" min="1" max="99">
```

**Case 2: Date Ranges**

```html
<input type="date" name="checkin" min="2026-01-01">
```

**Case 3: Volume Sliders**

```html
<input type="range" name="volume" min="0" max="100">
```

---

### 12. The `step` Attribute

#### Definitions

**Core Definition**

The `step` attribute specifies the granularity of numeric input — the intervals at which values can be incremented or decremented.

**Technical Definition**

The `step` attribute defines the increment for numeric input types (`number`, `range`, `date`, `time`, `datetime-local`, `month`, `week`). It can be a positive floating-point number or the keyword `any`. The default step for `number` is 1; for `range` it is 1; for `time` it is 60 seconds; for `date` it is 1 day. When `step="any"`, no stepping constraint is applied.

**Beginner-Friendly Explanation**

The `step` attribute controls how much the value changes when you click the up/down arrows or drag a slider. For example, `step="10"` means the value goes 0, 10, 20, 30.

#### Purposes

- To control the increment of numeric inputs
- To enforce specific value granularity
- To allow any decimal value with `step="any"`

#### Syntax Rules and Structure

```html
<input type="number" name="quantity" min="0" max="100" step="10">
<input type="number" name="price" step="0.01">
<input type="time" name="appointment" step="900">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `step` | Attribute name |
| `"10"` | The increment value |
| `"any"` | No stepping constraint |

**Syntax Rules**

- The value must be a positive number or `any`
- The `min` value, if present, is the base for stepping
- The value must be a multiple of `step` (offset by `min`)

**Constraints and Limitations**

- If the value does not match a step, the input is invalid
- `step="any"` removes all stepping constraints

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Stepped Number**

```html
<label for="quantity">Quantity (steps of 5):</label>
<input type="number" id="quantity" name="quantity" min="0" max="100" step="5" value="10">
```

**Expected Output**

A numeric input where arrows increment by 5.

**Why This Output Occurs**

The `step="5"` attribute sets the increment.

#### Real-World Cases

**Case 1: Price Inputs**

```html
<input type="number" name="price" step="0.01">
```

**Case 2: Time Slots**

```html
<input type="time" name="appointment" step="900">
```

**Case 3: Volume Controls**

```html
<input type="range" name="volume" min="0" max="100" step="10">
```

---

### 13. The `pattern` Attribute

#### Definitions

**Core Definition**

The `pattern` attribute specifies a regular expression that the input‘s value must match to be valid.

**Technical Definition**

The `pattern` attribute is a regular expression against which the control’s value is checked. The pattern is compiled with the `u` (Unicode) flag. The value must match the entire value, not just a part. The pattern is part of the constraint validation API. It applies to text-like inputs: `text`, `search`, `url`, `tel`, `email`, and `password`.

**Beginner-Friendly Explanation**

The `pattern` attribute lets you enforce a specific format for a text field using a regular expression. For example, you can require a phone number to be exactly 10 digits.

#### Purposes

- To enforce a specific format for text input
- To provide client-side validation without JavaScript
- To guide users toward correct formatting
- To improve data quality

#### Syntax Rules and Structure

```html
<input type="text" name="zip" pattern="[0-9]{5}" title="Five digit ZIP code">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `pattern` | Regular expression (without delimiters) |
| `title` | Tooltip shown when the pattern fails |

**Syntax Rules**

- The pattern must match the entire value
- The pattern is compiled with the Unicode flag
- The `title` attribute should describe the expected format
- The pattern applies to text-like inputs

**Constraints and Limitations**

- Not all browsers support the same regular expression features
- The pattern is not a substitute for server-side validation
- Complex patterns can be hard to communicate to users

#### Annotated Complete Step-by-Step Code Examples

**Example 1: ZIP Code Pattern**

```html
<label for="zip">ZIP Code (5 digits):</label>
<input type="text" id="zip" name="zip"
       pattern="[0-9]{5}" title="Five digit ZIP code" required>
```

**Expected Output**

A text field that only accepts five digits.

**Why This Output Occurs**

The `pattern` attribute enforces the format.

#### Real-World Cases

**Case 1: Phone Numbers**

```html
<input type="tel" name="phone" pattern="[0-9\-\+\s\(\)]{7,20}">
```

**Case 2: Postal Codes**

```html
<input type="text" name="zip" pattern="[0-9]{5}">
```

**Case 3: Usernames**

```html
<input type="text" name="username" pattern="[a-zA-Z0-9_]{3,16}">
```

---

### 14. Choosing the Right Attributes

#### Definitions

**Core Definition**

Choosing the right attributes means applying the appropriate combination of attributes for each control based on the data type, required behaviour, and accessibility requirements.

**Technical Definition**

Attribute selection depends on the control type, the nature of the data, and the desired user experience. `required` is for mandatory fields; `readonly` is for non-editable but submitted values; `disabled` is for non-interactive and non-submitted values. `min`, `max`, and `step` constrain numeric and date inputs. `pattern` enforces text formats. `autocomplete` improves UX and accessibility. The correct combination ensures validation, accessibility, and data quality.

**Beginner-Friendly Explanation**

Different fields need different settings. A password field needs `autocomplete="current-password"` and `required`. A quantity field needs `min`, `max`, and `step`. A ZIP code field needs `pattern` and `required`. Choose the attributes that match what the field is for.

#### Decision Guide

| Field Type | Recommended Attributes |
|---|---|
| Username | `required`, `autocomplete="username"`, `minlength` |
| Password | `required`, `autocomplete="current-password"`, `minlength` |
| Email | `type="email"`, `required`, `autocomplete="email"` |
| Quantity | `type="number"`, `min`, `max`, `step`, `required` |
| ZIP Code | `pattern`, `required`, `autocomplete="postal-code"` |
| Phone | `type="tel"`, `pattern`, `autocomplete="tel"` |
| Date | `type="date"`, `min`, `max`, `required` |
| Checkbox | `checked` (if default), `required` (for terms) |
| Dropdown | `required`, `multiple` (if multi-select) |

---

## References

- MDN Web Docs – HTML attribute reference – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes
- MDN Web Docs – `<input>`: The Input element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input
- MDN Web Docs – `name` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#name
- MDN Web Docs – `value` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#value
- MDN Web Docs – `placeholder` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#placeholder
- MDN Web Docs – `required` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#required
- MDN Web Docs – `readonly` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#readonly
- MDN Web Docs – `disabled` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#disabled
- MDN Web Docs – `checked` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox#checked
- MDN Web Docs – `selected` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/option#selected
- MDN Web Docs – `multiple` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/select#multiple
- MDN Web Docs – `autocomplete` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete
- MDN Web Docs – `min` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/min
- MDN Web Docs – `max` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/max
- MDN Web Docs – `step` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/step
- MDN Web Docs – `pattern` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/pattern
- WHATWG HTML Living Standard – Form control infrastructure – https://html.spec.whatwg.org/multipage/form-control-infrastructure.html
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.5: Identify Input Purpose – https://www.w3.org/WAI/WCAG21/Understanding/identify-input-purpose.html
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.2: Labels or Instructions – https://www.w3.org/WAI/WCAG21/Understanding/labels-or-instructions.html
- W3C – HTML5: Attributes for form submission – https://dev.w3.org/html5/spec-author-view/association-of-controls-and-forms.html