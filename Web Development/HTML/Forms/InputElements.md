# HTML Input Elements: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML input elements are the interactive controls created with the `<input>` element, whose behaviour and data type are determined by the `type` attribute, enabling users to enter, select, and submit data to a web server.

**Technical Definition**

The `<input>` element represents a typed data field, usually with a form control to allow the user to edit the data. It is categorised as flow content, phrasing content, interactive content (if not `type="hidden"`), and palpable content (if not `type="hidden"`). Its content model is nothing (it is a void element with no closing tag). Its permitted parent is any element that accepts phrasing content. Its DOM interface is `HTMLInputElement`. The `type` attribute is an enumerated attribute that determines which of the many input control types the element represents, and therefore how the element behaves, what attributes apply, and what value it submits.

**Beginner-Friendly Explanation**

The `<input>` tag is the Swiss Army knife of HTML forms. It can be a text box, a checkbox, a radio button, a date picker, a colour picker, a file upload button, or many other controls — all depending on what you put in the `type` attribute. If you want a text field, you write `<input type="text">`. If you want a checkbox, you write `<input type="checkbox">`. One tag, many possibilities.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Void element** | `<input>` has no closing tag and no content |
| **Type-driven** | The `type` attribute determines the control's behaviour and appearance |
| **Default type** | If `type` is omitted, the default is `text` |
| **Self-contained** | Each input is a single element; no child elements |
| **Accessibility-critical** | Every input needs a label associated via `for`/`id` or wrapping |
| **Global attributes** | Supports all global attributes plus type-specific attributes |
| **DOM interface** | `HTMLInputElement` |

---

### Prerequisites

- Basic familiarity with HTML document structure
- Understanding of the `<form>` element and form submission
- Awareness of the `<label>` element and accessibility
- Basic knowledge of HTTP methods (GET/POST) and form data

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Labels, ARIA attributes, and keyboard navigation are essential
- **Client-Side Validation** – HTML5 constraint validation uses input attributes
- **Mobile Input** – `type` and `inputmode` control mobile keyboards
- **CSS Styling** – Pseudo-classes like `:valid`, `:invalid`, `:focus` style inputs
- **JavaScript** – The Constraint Validation API and FormData interact with inputs

---

## Core Concepts / Features

---

### 1. The `<input>` Element (Base)

#### Definitions

**Core Definition**

The `<input>` element is a void element that creates an interactive form control, with its behaviour determined by the `type` attribute.

**Technical Definition**

The `<input>` HTML element is used to create interactive controls for web-based forms to accept data from the user. It is one of the most powerful and complex elements in HTML due to the sheer number of combinations of input types and attributes. The element is a void element, meaning it has no content and no closing tag. It is categorised as flow content, phrasing content, interactive content, and palpable content (except for `type="hidden"`, which is not interactive or palpable). Its DOM interface is `HTMLInputElement`.

**Beginner-Friendly Explanation**

The `<input>` tag creates a control that the user can interact with. The `type` attribute tells the browser what kind of control to show — a text box, a checkbox, a button, and so on. Without a `type`, it defaults to a text box.

#### Purposes

- To create interactive form controls for data entry
- To accept user input of various data types
- To submit data to a server via a form
- To provide accessible, keyboard-navigable controls

#### Syntax Rules and Structure

**General Syntax**

```html
<input type="text" name="username" id="username">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<input>` | Void element; no closing tag |
| `type` | Determines the control type; defaults to `text` |
| `name` | The key used when submitting the form |
| `id` | Unique identifier for label association |
| `value` | The initial value of the control |

**Common Attributes**

| Attribute | Applies To | Description |
|---|---|---|
| `name` | All | The form data key |
| `value` | All | Initial/current value |
| `id` | All | Unique identifier |
| `required` | Most | Field must be filled before submission |
| `disabled` | All | Control is not interactive or submitted |
| `readonly` | Text-like | Value cannot be edited but is submitted |
| `placeholder` | Text-like | Hint text shown when empty |
| `autocomplete` | Most | Controls browser autofill |
| `autofocus` | All | Automatically focuses on page load |

**Syntax Rules**

- The `<input>` element must have a `name` attribute to be submitted
- The `type` attribute is case-insensitive
- If `type` is invalid or unsupported, the browser falls back to `text`
- Every input should have an associated `<label>` for accessibility

**Constraints and Limitations**

- The `type` attribute cannot be changed dynamically without side effects
- Some types have browser-specific rendering differences
- `disabled` inputs are not submitted with the form

---

### 2. Text Input (`type="text"`)

#### Definitions

**Core Definition**

A single-line text input that accepts any character.

**Technical Definition**

The `text` type creates a single-line text field. It is the default type if the `type` attribute is omitted or invalid. It supports `maxlength`, `minlength`, `pattern`, `placeholder`, `readonly`, `size`, and `spellcheck`. The value is sanitised with newlines stripped.

**Beginner-Friendly Explanation**

A text box. The user can type anything. This is the most basic and common input type.

#### Purposes

- To accept free-form single-line text
- To collect names, usernames, and short answers
- To serve as the default input when no type is specified

#### Syntax Rules and Structure

```html
<input type="text" name="username" id="username"
       maxlength="20" minlength="3" placeholder="Enter username" required>
```

**Constraints and Limitations**

- No format validation; accepts any text
- Use `pattern` for simple validation

#### Annotated Code Example

```html
<form action="/submit" method="post">
    <label for="fullname">Full Name:</label>
    <input type="text" id="fullname" name="fullname"
           placeholder="e.g., Jane Doe" maxlength="50" required>
    <button type="submit">Submit</button>
</form>
```

**Expected Output**

A text box with placeholder “e.g., Jane Doe.” Submitting sends `fullname=Jane+Doe`.

**Why This Output Occurs**

The `text` type renders a single-line input. The `placeholder` shows hint text. The `maxlength` limits input to 50 characters. The `required` attribute prevents empty submission.

#### Real-World Cases

- **Registration forms** – Username and full name fields
- **Search bars** – Free-text search queries
- **Comment forms** – Short subject lines

---

### 3. Password Input (`type="password"`)

#### Definitions

**Core Definition**

A single-line text input that obscures entered characters.

**Technical Definition**

The `password` type creates a text field where the value is obscured (typically with dots or asterisks) to prevent shoulder-surfing. It supports the same attributes as `text`. The value is not masked at the protocol level — it is transmitted in plaintext unless HTTPS is used.

**Beginner-Friendly Explanation**

A text box where the characters are hidden. Use it for passwords and other secrets.

#### Purposes

- To collect sensitive credentials
- To prevent visual exposure of typed characters
- To provide a familiar password-entry experience

#### Syntax Rules and Structure

```html
<input type="password" name="password" id="password"
       minlength="8" autocomplete="current-password" required>
```

**Constraints and Limitations**

- Obscuring is visual only; HTTPS is required for real security
- Browsers may offer password managers

#### Annotated Code Example

```html
<form action="/login" method="post">
    <label for="pwd">Password:</label>
    <input type="password" id="pwd" name="password"
           minlength="8" autocomplete="current-password" required>
    <button type="submit">Log In</button>
</form>
```

**Expected Output**

A text field where typed characters appear as dots. Submitting sends the password in the request body.

**Why This Output Occurs**

The `password` type obscures the value visually. The `autocomplete="current-password"` hints to password managers. The `minlength="8"` enforces a minimum length.

#### Real-World Cases

- **Login forms** – Password entry
- **Registration** – Creating new passwords
- **Admin panels** – Sensitive credential entry

---

### 4. Email Input (`type="email"`)

#### Definitions

**Core Definition**

A text input that validates the value against the email address format.

**Technical Definition**

The `email` type creates a text field that validates the value as an email address. It supports the `multiple` attribute to allow comma-separated addresses. On mobile devices, it triggers an email-optimised keyboard.

**Beginner-Friendly Explanation**

A text box specifically for email addresses. The browser checks that what you type looks like an email before submitting.

#### Purposes

- To collect email addresses with validation
- To trigger email-optimised mobile keyboards
- To support multiple email addresses

#### Syntax Rules and Structure

```html
<input type="email" name="email" id="email" multiple required>
```

**Constraints and Limitations**

- Basic format validation only; does not verify the address exists
- Use `pattern` for stricter validation

#### Annotated Code Example

```html
<label for="email">Email Address:</label>
<input type="email" id="email" name="email"
       placeholder="you@example.com" required>
```

**Expected Output**

A text field that validates email format. Submitting an invalid email shows a browser error.

**Why This Output Occurs**

The `email` type triggers built-in validation. The browser checks for an `@` symbol and a domain.

#### Real-World Cases

- **Newsletter signup** – Email collection
- **Contact forms** – Reply-to addresses
- **Account registration** – Email verification

---

### 5. Search Input (`type="search"`)

#### Definitions

**Core Definition**

A text input semantically intended for search queries.

**Technical Definition**

The `search` type creates a text field for entering search queries. It may render with a clear button in some browsers. It is functionally similar to `text` but carries semantic meaning.

**Beginner-Friendly Explanation**

A text box for searching. Some browsers add a little “x” to clear the field.

#### Purposes

- To semantically identify search fields
- To trigger search-optimised mobile keyboards
- To enable browser-specific search UI

#### Syntax Rules and Structure

```html
<input type="search" name="q" id="search" placeholder="Search...">
```

#### Annotated Code Example

```html
<form action="/search" method="get">
    <label for="search">Search:</label>
    <input type="search" id="search" name="q" placeholder="Search products...">
    <button type="submit">Go</button>
</form>
```

**Expected Output**

A search field with placeholder text. Submitting navigates to `/search?q=...`.

**Why This Output Occurs**

The `search` type signals to the browser that this is a search field, enabling search-specific UI and mobile keyboards.

#### Real-World Cases

- **Site search** – Product or content search
- **Documentation** – API or guide search
- **E-commerce** – Catalogue search

---

### 6. URL Input (`type="url"`)

#### Definitions

**Core Definition**

A text input that validates the value as an absolute URL.

**Technical Definition**

The `url` type creates a text field that validates the value as an absolute URL (must include a scheme like `https://`). It triggers a URL-optimised keyboard on mobile.

**Beginner-Friendly Explanation**

A text box for web addresses. The browser checks that what you type looks like a valid URL.

#### Purposes

- To collect website addresses with validation
- To trigger URL-optimised mobile keyboards
- To ensure values include a scheme

#### Syntax Rules and Structure

```html
<input type="url" name="website" id="website" placeholder="https://example.com">
```

#### Annotated Code Example

```html
<label for="website">Your Website:</label>
<input type="url" id="website" name="website"
       placeholder="https://example.com" required>
```

**Expected Output**

A text field that validates URL format. Invalid URLs (e.g., “example”) trigger a browser error.

**Why This Output Occurs**

The `url` type requires an absolute URL with a scheme.

#### Real-World Cases

- **Profile forms** – Personal website
- **Business listings** – Company URL
- **Portfolio submissions** – Project links

---

### 7. Telephone Input (`type="tel"`)

#### Definitions

**Core Definition**

A text input semantically intended for telephone numbers.

**Technical Definition**

The `tel` type creates a text field for telephone numbers. It does not perform format validation (telephone formats vary globally) but triggers a telephone-optimised keyboard on mobile. Use `pattern` for format validation.

**Beginner-Friendly Explanation**

A text box for phone numbers. It doesn‘t validate the format because phone numbers vary by country, but it shows a number keyboard on mobile.

#### Purposes

- To collect telephone numbers
- To trigger telephone-optimised mobile keyboards
- To semantically identify phone fields

#### Syntax Rules and Structure

```html
<input type="tel" name="phone" id="phone" pattern="[0-9\-\+\s\(\)]{7,20}">
```

#### Annotated Code Example

```html
<label for="phone">Phone Number:</label>
<input type="tel" id="phone" name="phone"
       placeholder="+1 (555) 123-4567"
       pattern="[0-9\-\+\s\(\)]{7,20}">
```

**Expected Output**

A text field that accepts phone numbers. On mobile, a numeric keypad appears.

**Why This Output Occurs**

The `tel` type triggers a telephone keyboard. The `pattern` attribute enforces a basic format.

#### Real-World Cases

- **Contact forms** – Phone number collection
- **Checkout** – Billing phone
- **Appointment booking** – Contact number

---

### 8. Number Input (`type="number"`)

#### Definitions

**Core Definition**

A numeric input with spinner controls and optional range constraints.

**Technical Definition**

The `number` type creates a field for entering a number. It supports `min`, `max`, and `step` attributes. Browsers render spinner buttons. On mobile, a numeric keyboard appears.

**Beginner-Friendly Explanation**

A text box for numbers with up/down arrows. You can set a minimum, maximum, and step value.

#### Purposes

- To collect numeric values with validation
- To provide spinner controls for incrementing/decrementing
- To constrain values to a range

#### Syntax Rules and Structure

```html
<input type="number" name="quantity" id="quantity"
       min="1" max="100" step="1" value="1">
```

**Constraints and Limitations**

- Not suitable for credit card numbers or phone numbers (use `text` with `inputmode="numeric"`)

#### Annotated Code Example

```html
<label for="quantity">Quantity:</label>
<input type="number" id="quantity" name="quantity"
       min="1" max="10" step="1" value="1" required>
```

**Expected Output**

A numeric field with spinner arrows, constrained between 1 and 10.

**Why This Output Occurs**

The `min`, `max`, and `step` attributes constrain the value. The browser provides spinner controls.

#### Real-World Cases

- **E-commerce** – Quantity selection
- **Booking** – Number of guests
- **Surveys** – Age or rating input

---

### 9. Range Input (`type="range"`)

#### Definitions

**Core Definition**

A slider control for selecting a numeric value within a range.

**Technical Definition**

The `range` type creates a slider. It supports `min`, `max`, `step`, and `value`. The exact value is not shown to the user by default (use `<output>` or JavaScript to display it). It is ideal for imprecise numeric input.

**Beginner-Friendly Explanation**

A slider you drag left or right to pick a number. Good for things like volume or price filters.

#### Purposes

- To provide a visual, imprecise numeric selector
- To improve UX for bounded numeric input
- To enable touch-friendly value selection

#### Syntax Rules and Structure

```html
<input type="range" name="volume" id="volume"
       min="0" max="100" step="10" value="50">
```

#### Annotated Code Example

```html
<label for="volume">Volume:</label>
<input type="range" id="volume" name="volume"
       min="0" max="100" step="10" value="50"
       oninput="document.getElementById('vol-out').value = this.value">
<output id="vol-out">50</output>
```

**Expected Output**

A slider with a value display that updates as you drag.

**Why This Output Occurs**

The `range` type renders a slider. The `oninput` handler updates the `<output>` element.

#### Real-World Cases

- **Media players** – Volume control
- **E-commerce** – Price range filters
- **Settings** – Brightness or zoom sliders

---

### 10. Color Input (`type="color"`)

#### Definitions

**Core Definition**

A colour picker control that returns a hexadecimal colour value.

**Technical Definition**

The `color` type creates a control that opens the browser‘s native colour picker. The value is always a 7-character hexadecimal string (e.g., `#ff0000`). The default value is `#000000` if none is specified.

**Beginner-Friendly Explanation**

A button that opens a colour picker. The value is a hex colour code.

#### Purposes

- To let users pick colours visually
- To collect colour preferences
- To enable theme customisation

#### Syntax Rules and Structure

```html
<input type="color" name="theme" id="theme" value="#0066cc">
```

#### Annotated Code Example

```html
<label for="theme">Choose a theme colour:</label>
<input type="color" id="theme" name="theme" value="#4a6cf7">
```

**Expected Output**

A colour swatch button. Clicking it opens the native colour picker.

**Why This Output Occurs**

The `color` type triggers the browser‘s colour picker.

#### Real-World Cases

- **Profile customisation** – Avatar or theme colours
- **Design tools** – Colour palette selection
- **Form builders** – Colour settings

---

### 11. Date Input (`type="date"`)

#### Definitions

**Core Definition**

A date picker control for selecting a calendar date.

**Technical Definition**

The `date` type creates a control for entering a date (year, month, day). The value is in `YYYY-MM-DD` format. Browsers provide a date picker UI. It supports `min`, `max`, and `step` (in days).

**Beginner-Friendly Explanation**

A date picker. The browser shows a calendar, and the value is a date like “2026-09-27.”

#### Purposes

- To collect dates with a picker UI
- To validate date ranges
- To provide mobile-optimised date entry

#### Syntax Rules and Structure

```html
<input type="date" name="birthdate" id="birthdate"
       min="1900-01-01" max="2026-12-31">
```

#### Annotated Code Example

```html
<label for="start">Start Date:</label>
<input type="date" id="start" name="start"
       min="2026-01-01" max="2026-12-31" required>
```

**Expected Output**

A date field with a calendar picker, constrained to 2026.

**Why This Output Occurs**

The `date` type renders a date picker. The `min` and `max` attributes restrict the range.

#### Real-World Cases

- **Booking systems** – Check-in dates
- **Forms** – Date of birth
- **Reports** – Reporting period selection

---

### 12. Time Input (`type="time"`)

#### Definitions

**Core Definition**

A time picker control for selecting a time of day.

**Technical Definition**

The `time` type creates a control for entering a time (hours, minutes, optionally seconds). The value is in `HH:MM` or `HH:MM:SS` format. It supports `min`, `max`, and `step` (in seconds).

**Beginner-Friendly Explanation**

A time picker. The browser shows a clock, and the value is like “14:30.”

#### Purposes

- To collect times with a picker UI
- To validate time ranges
- To provide mobile-optimised time entry

#### Syntax Rules and Structure

```html
<input type="time" name="appointment" id="appointment"
       min="09:00" max="17:00" step="900">
```

#### Annotated Code Example

```html
<label for="meeting">Meeting Time:</label>
<input type="time" id="meeting" name="meeting"
       min="09:00" max="17:00" required>
```

**Expected Output**

A time picker constrained to 9 AM – 5 PM.

**Why This Output Occurs**

The `time` type renders a time picker. The `min` and `max` attributes restrict the range.

#### Real-World Cases

- **Scheduling** – Appointment times
- **Booking** – Reservation times
- **Forms** – Preferred contact times

---

### 13. Datetime-Local Input (`type="datetime-local"`)

#### Definitions

**Core Definition**

A control for selecting both a date and a time, without timezone information.

**Technical Definition**

The `datetime-local` type creates a control for entering a date and time (year, month, day, hours, minutes). The value is in `YYYY-MM-DDTHH:MM` format. It supports `min`, `max`, and `step`.

**Beginner-Friendly Explanation**

A picker that lets you choose both a date and a time. Useful for scheduling.

#### Purposes

- To collect date-and-time combinations
- To schedule events or appointments
- To provide a combined picker UI

#### Syntax Rules and Structure

```html
<input type="datetime-local" name="event" id="event"
       min="2026-01-01T00:00" max="2026-12-31T23:59">
```

#### Annotated Code Example

```html
<label for="appointment">Appointment:</label>
<input type="datetime-local" id="appointment" name="appointment"
       min="2026-09-27T09:00" max="2026-09-27T17:00" required>
```

**Expected Output**

A combined date-and-time picker.

**Why This Output Occurs**

The `datetime-local` type renders a date picker and a time picker together.

#### Real-World Cases

- **Event scheduling** – Meeting or event times
- **Booking systems** – Appointment selection
- **Reminders** – Date and time for reminders

---

### 14. Month Input (`type="month"`)

#### Definitions

**Core Definition**

A control for selecting a month and year.

**Technical Definition**

The `month` type creates a control for entering a month and year. The value is in `YYYY-MM` format. It supports `min`, `max`, and `step`.

**Beginner-Friendly Explanation**

A picker for a month and year. Useful for things like credit card expiration dates.

#### Purposes

- To collect month-and-year values
- To provide a picker for monthly data
- To validate month ranges

#### Syntax Rules and Structure

```html
<input type="month" name="expiry" id="expiry" min="2026-01" max="2036-12">
```

#### Annotated Code Example

```html
<label for="expiry">Card Expiry:</label>
<input type="month" id="expiry" name="expiry"
       min="2026-01" max="2036-12" required>
```

**Expected Output**

A month-and-year picker.

**Why This Output Occurs**

The `month` type renders a month picker.

#### Real-World Cases

- **Payment forms** – Card expiry
- **Reports** – Monthly reporting periods
- **Subscriptions** – Billing month

---

### 15. Week Input (`type="week"`)

#### Definitions

**Core Definition**

A control for selecting a week and year.

**Technical Definition**

The `week` type creates a control for entering a week number and year. The value is in `YYYY-Www` format (e.g., `2026-W39`). It supports `min`, `max`, and `step`.

**Beginner-Friendly Explanation**

A picker for a week of the year. Less common but useful for weekly planning.

#### Purposes

- To collect week-and-year values
- To provide a picker for weekly data
- To validate week ranges

#### Syntax Rules and Structure

```html
<input type="week" name="week" id="week" min="2026-W01" max="2026-W52">
```

#### Annotated Code Example

```html
<label for="week">Select Week:</label>
<input type="week" id="week" name="week" required>
```

**Expected Output**

A week-and-year picker.

**Why This Output Occurs**

The `week` type renders a week picker.

#### Real-World Cases

- **Project planning** – Weekly sprints
- **Timesheets** – Weekly reporting
- **Scheduling** – Week selection

---

### 16. Checkbox Input (`type="checkbox"`)

#### Definitions

**Core Definition**

A toggle control that allows zero or more selections from a set of options.

**Technical Definition**

The `checkbox` type creates a toggle that can be checked or unchecked. Multiple checkboxes can be selected independently. The `value` attribute defines what is submitted when checked. The `checked` attribute sets the initial state. The `indeterminate` property can be set via JavaScript for a third state.

**Beginner-Friendly Explanation**

A small box you click to check or uncheck. You can select multiple checkboxes at once. Good for “choose all that apply” questions.

#### Purposes

- To allow multiple independent selections
- To toggle options on and off
- To collect multiple values under one name

#### Syntax Rules and Structure

```html
<input type="checkbox" name="interests" value="coding" id="coding" checked>
<label for="coding">Coding</label>
```

**Constraints and Limitations**

- Only the checked checkboxes‘ values are submitted
- The `value` attribute is required for meaningful submission

#### Annotated Code Example

```html
<fieldset>
    <legend>Interests:</legend>
    <input type="checkbox" id="coding" name="interests" value="coding" checked>
    <label for="coding">Coding</label>

    <input type="checkbox" id="design" name="interests" value="design">
    <label for="design">Design</label>
</fieldset>
```

**Expected Output**

Two checkboxes, one pre-checked. Submitting sends `interests=coding`.

**Why This Output Occurs**

The `checked` attribute pre-selects the first checkbox. Only checked values are submitted.

#### Real-World Cases

- **Signup forms** – Newsletter preferences
- **Surveys** – Multiple-choice questions
- **Settings** – Feature toggles

---

### 17. Radio Input (`type="radio"`)

#### Definitions

**Core Definition**

A selection control that allows exactly one choice from a set of options.

**Technical Definition**

The `radio` type creates a radio button. Radio buttons with the same `name` attribute form a group, and only one can be selected at a time. The `value` attribute defines what is submitted when selected. The `checked` attribute sets the initial selection.

**Beginner-Friendly Explanation**

A round button you click to select one option from a group. You can only pick one.

#### Purposes

- To allow exactly one selection from a group
- To present mutually exclusive options
- To collect a single value from a set

#### Syntax Rules and Structure

```html
<input type="radio" name="gender" value="female" id="female" checked>
<label for="female">Female</label>
```

**Constraints and Limitations**

- Radio buttons with the same `name` must have different `value` attributes
- No radio button in a group can be pre-selected if none has `checked`

#### Annotated Code Example

```html
<fieldset>
    <legend>Gender:</legend>
    <input type="radio" id="female" name="gender" value="female" checked>
    <label for="female">Female</label>

    <input type="radio" id="male" name="gender" value="male">
    <label for="male">Male</label>

    <input type="radio" id="other" name="gender" value="other">
    <label for="other">Other</label>
</fieldset>
```

**Expected Output**

Three radio buttons, one pre-selected. Selecting another deselects the first.

**Why This Output Occurs**

All three share the same `name`, making them a group. The `checked` attribute pre-selects “Female.”

#### Real-World Cases

- **Forms** – Gender, title, or salutation
- **Surveys** – Single-choice questions
- **Checkout** – Shipping method

---

### 18. File Input (`type="file"`)

#### Definitions

**Core Definition**

A control for selecting files from the user‘s device for upload.

**Technical Definition**

The `file` type creates a control that allows the user to select one or more files. It requires the form to use `enctype="multipart/form-data"` for submission. The `accept` attribute filters file types. The `multiple` attribute allows multiple file selection. The `capture` attribute specifies the camera to use on mobile.

**Beginner-Friendly Explanation**

A button that opens your device‘s file picker. Use it to upload documents, images, or other files.

#### Purposes

- To upload files to a server
- To filter selectable file types
- To allow multiple file selection

#### Syntax Rules and Structure

```html
<form enctype="multipart/form-data" method="post">
    <input type="file" name="avatar" id="avatar" accept="image/*">
</form>
```

**Constraints and Limitations**

- Requires `enctype="multipart/form-data"`
- The `value` cannot be set programmatically for security reasons

#### Annotated Code Example

```html
<form action="/upload" method="post" enctype="multipart/form-data">
    <label for="avatar">Profile Picture:</label>
    <input type="file" id="avatar" name="avatar"
           accept="image/png, image/jpeg" required>
    <button type="submit">Upload</button>
</form>
```

**Expected Output**

A file picker that only accepts PNG and JPEG images.

**Why This Output Occurs**

The `accept` attribute filters the file picker. The `enctype` on the form enables multipart upload.

#### Real-World Cases

- **Profile pictures** – Avatar uploads
- **Document submission** – PDF uploads
- **Image sharing** – Photo uploads

---

### 19. Hidden Input (`type="hidden"`)

#### Definitions

**Core Definition**

A non-visible input that stores data to be submitted with the form.

**Technical Definition**

The `hidden` type creates an input that is not displayed but whose value is submitted. It is not interactive and has no accessible role. It is useful for CSRF tokens, record IDs, and state tracking. Hidden inputs are not rendered and cannot be focused.

**Beginner-Friendly Explanation**

A form field the user can‘t see. It stores data that gets submitted without the user knowing — like a record ID or a security token.

#### Purposes

- To store data that should be submitted but not displayed
- To include CSRF tokens
- To track record IDs
- To persist state across form submissions

#### Syntax Rules and Structure

```html
<input type="hidden" name="record_id" value="12345">
```

**Constraints and Limitations**

- Not accessible; do not use for important information the user needs
- Values are visible in the page source and can be modified

#### Annotated Code Example

```html
<form action="/update" method="post">
    <input type="hidden" name="record_id" value="12345">
    <input type="hidden" name="csrf_token" value="abc123xyz">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>
    <button type="submit">Update</button>
</form>
```

**Expected Output**

A visible Name field and two hidden inputs. Submitting sends all three values.

**Why This Output Occurs**

Hidden inputs are not rendered but are included in the form data set.

#### Real-World Cases

- **CRUD operations** – Record IDs
- **Security** – CSRF tokens
- **Multi-step forms** – Step tracking

---

### 20. Submit Input (`type="submit"`)

#### Definitions

**Core Definition**

A button that submits the form when activated.

**Technical Definition**

The `submit` type creates a button that triggers form submission. The `value` attribute sets the button label. It can be overridden by `formaction`, `formmethod`, `formenctype`, `formnovalidate`, and `formtarget` attributes.

**Beginner-Friendly Explanation**

The button you click to submit the form.

#### Purposes

- To trigger form submission
- To provide a labelled submit control
- To override form-level submission attributes

#### Syntax Rules and Structure

```html
<input type="submit" value="Send">
```

#### Annotated Code Example

```html
<form action="/submit" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <input type="submit" value="Subscribe">
</form>
```

**Expected Output**

A button labelled “Subscribe” that submits the form.

**Why This Output Occurs**

The `submit` type creates a submission button with the label from `value`.

#### Real-World Cases

- **All forms** – The primary action button
- **Multi-button forms** – Different submit actions

---

### 21. Button Input (`type="button"`)

#### Definitions

**Core Definition**

A clickable button with no default behaviour, typically used with JavaScript.

**Technical Definition**

The `button` type creates a button with no built-in action. It is used with JavaScript event handlers. It does not submit the form. The `value` attribute sets the button label.

**Beginner-Friendly Explanation**

A button that doesn‘t do anything by itself. You attach JavaScript to make it do something.

#### Purposes

- To create a button for JavaScript interaction
- To provide a non-submitting action button
- To trigger custom behaviour

#### Syntax Rules and Structure

```html
<input type="button" value="Click Me" onclick="doSomething()">
```

#### Annotated Code Example

```html
<form action="/submit" method="post">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name">
    <input type="button" value="Check Availability" onclick="checkName()">
    <input type="submit" value="Submit">
</form>
```

**Expected Output**

Two buttons: “Check Availability” (runs JavaScript) and “Submit” (submits the form).

**Why This Output Occurs**

The `button` type does not submit. The `submit` type does.

#### Real-World Cases

- **Form validation** – Check before submit
- **Interactive forms** – Dynamic field addition
- **Preview buttons** – Show a preview

---

### 22. Reset Input (`type="reset"`)

#### Definitions

**Core Definition**

A button that resets all form controls to their initial values.

**Technical Definition**

The `reset` type creates a button that resets the form‘s controls to their default values (the values specified in the HTML, or empty). It fires a `reset` event that can be cancelled. It is generally discouraged for usability reasons.

**Beginner-Friendly Explanation**

A button that clears the form and puts everything back to how it started. Use it carefully — users sometimes click it by accident.

#### Purposes

- To reset form fields to their initial values
- To provide a “clear form” action
- To undo user input

#### Syntax Rules and Structure

```html
<input type="reset" value="Reset Form">
```

**Constraints and Limitations**

- Can cause accidental data loss
- Not recommended for large or complex forms

#### Annotated Code Example

```html
<form action="/submit" method="post">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" value="">
    <input type="reset" value="Clear">
    <input type="submit" value="Submit">
</form>
```

**Expected Output**

A “Clear” button that empties the form and a “Submit” button that submits it.

**Why This Output Occurs**

The `reset` type restores controls to their initial values.

#### Real-World Cases

- **Simple forms** – Quick clear
- **Search forms** – Reset filters
- **Internal tools** – Testing workflows

---

### 23. Image Input (`type="image"`)

#### Definitions

**Core Definition**

A graphical submit button that sends the click coordinates along with the form data.

**Technical Definition**

The `image` type creates a submit button rendered as an image. It requires the `src` attribute and supports `alt`, `width`, `height`, and the form-override attributes. When clicked, it submits the form and includes the `x` and `y` coordinates of the click relative to the image.

**Beginner-Friendly Explanation**

An image that acts as a submit button. When clicked, it also sends the coordinates of where you clicked.

#### Purposes

- To provide a graphically styled submit button
- To capture click coordinates
- To create image-based form controls

#### Syntax Rules and Structure

```html
<input type="image" src="submit-button.png" alt="Submit" width="100" height="40">
```

#### Annotated Code Example

```html
<form action="/submit" method="post">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>
    <input type="image" src="submit.png" alt="Submit Form" width="120" height="40">
</form>
```

**Expected Output**

An image that submits the form when clicked, sending `name` plus `x` and `y` coordinates.

**Why This Output Occurs**

The `image` type renders the `src` image as a submit button and includes click coordinates in the submission.

#### Real-World Cases

- **Legacy forms** – Image-based submit buttons
- **Graphical interfaces** – Custom button designs
- **Image maps** – Coordinate-based interaction

---

## References

- MDN Web Docs – `<input>`: The Input (Form Input) element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input
- MDN Web Docs – The HTML5 input types – https://developer.mozilla.org/en-US/docs/Learn/Forms/HTML5_input_types
- MDN Web Docs – `<input type="text">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/text
- MDN Web Docs – `<input type="password">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/password
- MDN Web Docs – `<input type="email">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/email
- MDN Web Docs – `<input type="search">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/search
- MDN Web Docs – `<input type="url">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/url
- MDN Web Docs – `<input type="tel">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/tel
- MDN Web Docs – `<input type="number">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/number
- MDN Web Docs – `<input type="range">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/range
- MDN Web Docs – `<input type="color">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/color
- MDN Web Docs – `<input type="date">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/date
- MDN Web Docs – `<input type="time">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/time
- MDN Web Docs – `<input type="datetime-local">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/datetime-local
- MDN Web Docs – `<input type="month">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/month
- MDN Web Docs – `<input type="week">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/week
- MDN Web Docs – `<input type="checkbox">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox
- MDN Web Docs – `<input type="radio">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/radio
- MDN Web Docs – `<input type="file">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/file
- MDN Web Docs – `<input type="hidden">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/hidden
- MDN Web Docs – `<input type="submit">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/submit
- MDN Web Docs – `<input type="button">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/button
- MDN Web Docs – `<input type="reset">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/reset
- MDN Web Docs – `<input type="image">` – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/image
- WHATWG HTML Living Standard – Form control infrastructure – https://html.spec.whatwg.org/multipage/form-control-infrastructure.html
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.5: Identify Input Purpose – https://www.w3.org/WAI/WCAG21/Understanding/identify-input-purpose.html
- W3C – WCAG 2.1 Understanding Success Criterion 3.3.2: Labels or Instructions – https://www.w3.org/WAI/WCAG21/Understanding/labels-or-instructions.html