# HTML Input Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
The `<input>` element is the most versatile interactive control in HTML. Through its `type` attribute it morphs into a text field, a password box, an email address field, a date picker, a colour swatch, a checkbox, a radio button, a file picker, a range slider, or a button — all using the same element name and a large shared attribute vocabulary.

**Technical Definition**
`HTMLInputElement` is a form-associated, phrasing-content, interactive, palpable element with a content model of *nothing* (it is a void element). Its behaviour is the union of the selected `type`'s state and the attributes applicable to that state. The input participates in constraint validation via `validity`, `validationMessage`, `willValidate`, and `setCustomValidity()`. It is listed in the form's `elements` collection and contributes a name/value entry on submission if it is a *successful control*.

**Beginner-Friendly Explanation**
`<input>` is the “Swiss Army knife” of HTML. With one tag and a `type` setting, you can ask for a name, a password, an email, a date, a colour, a file — or create a clickable button. Its `type` and its attributes determine what the user sees and what the browser checks before submission.

---

### Key Characteristics

- **Void element**: `<input>` has no closing tag and no children.
- **Type-driven behaviour**: The `type` attribute selects an input state; each state has its own value semantics, default UI, applicable attributes, and validation rules.
- **Form-associated**: It contributes a name/value to its owning form unless disabled, unnamed, or not successful.
- **Constraint-validated**: Most types participate in HTML constraint validation (`required`, `pattern`, `min`, `max`, `step`, `minlength`, `maxlength`, `type`-specific rules).
- **Scriptable**: Exposes `value`, `checked`, `files`, `valueAsNumber`, `valueAsDate`, `selectionStart`, `setSelectionRange()`, `checkValidity()`, `select()`, etc.
- **Accessible when labelled**: Every input must have an accessible name (`<label>`, `aria-label`, or `aria-labelledby`).

---

### Prerequisites

1. **HTML form fundamentals** — `<form>`, `action`, `method`, `enctype`, submission.
2. **Element fundamentals** — attributes, void elements, content models.
3. **Basic CSS** — sizing and layout of controls.
4. **Browser developer tools** — Elements panel, Console, Network panel.
5. **Basic HTTP awareness** — to understand what a form's data looks like when submitted.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Forms & submission** | `<input>` provides the values that forms submit. |
| **Constraint validation** | The HTML validity API and CSS `:valid`/`:invalid` selectors. |
| **Accessibility** | Labels, `aria-describedby`, error messaging, keyboard support. |
| **JavaScript/DOM** | `value`, `files`, `validity`, events (`input`, `change`, `invalid`). |
| **Mobile UX** | `type` and `inputmode` select virtual keyboard layouts. |
| **Security** | File upload restrictions, secret handling, autofill and password managers. |

---

### Core Concepts / Features

1. **`<input>`** (the element)
2. **Input types** — `text`, `password`, `email`, `url`, `tel`, `number`, `range`, `date`, `time`, `datetime-local`, `month`, `week`, `color`, `file`, `checkbox`, `radio`, `hidden`, `submit`, `reset`, `button`, `image`, `search`
3. **Input attributes** — `type`, `name`, `value`, `placeholder`, `required`, `disabled`, `readonly`, `autocomplete`, `autofocus`, `multiple`, `accept`, `min`, `max`, `step`, `pattern`, `minlength`, `maxlength`, `size`, `list`, `form`, `formaction`, `formenctype`, `formmethod`, `formnovalidate`, `formtarget`, `dirname`
4. **Input best practices**

Every concept below follows the same uniform structure: Definitions → Purposes → Syntax Rules and Structure → Annotated Code Examples → Real-World Cases → References.

---

## 1. `<input>`

### Definitions

**Core Definition**
`<input>` is a void, form-associated element whose type, appearance, behaviour, and validation rules are determined by its `type` attribute and the other attributes present.

**Technical Definition**
`HTMLInputElement` implements the `HTMLInputElement` interface. It has a `type` IDL attribute that reflects one of 22 standard type states, each with its own definition of *value sanitisation algorithm*, *default value*, *value*, and *applicability* of attributes. It is phrasing content, flow content, interactive content, and palpable content. Its content model is *nothing*; end tags are not permitted.

**Beginner-Friendly Explanation**
`<input>` is the interactive field you type into, tick, or click. What kind of field it is depends on the `type` you give it.

---

### Purposes

- **To collect user data** of many shapes (text, numbers, dates, files, choices).
- **To trigger actions** such as submitting or resetting a form via button-like types.
- **To submit auxiliary data** (e.g., `hidden` inputs carrying identifiers).
- **To provide UI affordances** with built-in browser controls (date pickers, colour pickers, sliders).
- **To enable constraint validation** so invalid input is caught before submission.

---

### Syntax Rules and Structure

**Complete general syntax**

```html
<input
  type="text"
  name="fieldName"
  value="initial value"
  placeholder="hint"
  required
  minlength="2"
  maxlength="64"
  autocomplete="name">
```

**Component breakdown**

| Component | Description |
|---|---|
| `<input` | Start tag; no end tag is permitted. |
| `type="…"` | Selects the input state; defaults to `text`. |
| `name="…"` | The key under which the value is submitted. |
| Other attributes | Apply conditionally based on `type`. |
| `>` | Closes the start tag; the element has no content. |

#### Syntax Rules

1. `<input>` is a **void element**; `</input>` is invalid.
2. Unknown `type` values fall back to `text` (the *invalid value default*).
3. Attributes not applicable to the current type are *ignored* (but still reflected on the element).
4. Only successful controls (named, not disabled, and, where relevant, checked/selected) submit data.
5. The `value` attribute sets the initial value; the IDL property `value` reflects the current value.

#### Constraints and Limitations

- `<input>` has no child content; use `aria-label`/`aria-labelledby` or `<label>` for accessible names.
- Some attributes (`pattern`, `minlength`, `maxlength`, `min`, `max`, `step`) only apply to specific types.
- Hidden inputs are invisible to users but visible in the page source; never store secrets there.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Text Input

**Setup guide**: Save as `input-basic.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Basic Input</title>
</head>
<body>
  <form action="/submit" method="post">
    <label for="fullname">Full name</label>
    <!-- type="text" is the default; name is the submission key -->
    <input id="fullname" type="text" name="fullname" placeholder="Jane Doe" required>
    <button>Send</button>
  </form>
</body>
</html>
```

**Expected output**: A single text field with the placeholder “Jane Doe”, labelled “Full name”, and a Send button. Submitting sends `fullname=Jane+Doe` (or an error bubble if empty).

**Why this output occurs**: `<input type="text">` creates a single-line text control. The `required` attribute causes constraint validation; the label is programmatically associated via `for`/`id`; and `name="fullname"` makes the value part of the submission.

---

#### Example 2: Several Input Types Together

**Setup guide**: Save as `input-many.html`; submit and inspect the request.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Multiple Inputs</title>
</head>
<body>
  <form action="/demo" method="get">
    <p><label for="name">Name</label>
      <input id="name" type="text" name="name" value="Ada"></p>
    <p><label for="color">Favourite colour</label>
      <input id="color" type="color" name="color" value="#3366cc"></p>
    <p><label for="age">Age</label>
      <input id="age" type="number" name="age" min="0" max="120" value="36"></p>
    <button>Submit</button>
  </form>
</body>
</html>
```

**Expected output**: On submit, URL becomes something like `/demo?name=Ada&color=%233366cc&age=36`.

**Why this output occurs**: Each named input contributes one entry. The colour value’s `#` is percent-encoded (`%23`).

---

### Real-World Cases

1. **Login and registration** — `<input type="email">` and `<input type="password">`.
2. **Checkout** — number, date, tel, and address fields with autocomplete tokens.
3. **Settings pages** — checkboxes and radio groups for preferences.
4. **Admin panels** — hidden inputs carrying IDs; file inputs for uploads.

---

### References

- MDN Web Docs — `<input>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input
- WHATWG — The input element — https://html.spec.whatwg.org/multipage/input.html#the-input-element
- MDN Web Docs — HTMLInputElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement

---

## 2. Input Types

Each input type is a distinct *state* with its own UI, value format, and applicable attributes. All twenty-two standard types are covered below with the same structure.

---

### 2.1 `text`

#### Definitions

**Core Definition**
The `text` type is a single-line plain text field.

**Technical Definition**
The *text* state has no automatic value sanitisation beyond newline removal (which the value sanitisation algorithm strips). It supports `dirname`, `list`, `maxlength`, `minlength`, `pattern`, `placeholder`, `readonly`, `required`, `size`, `autocomplete`. Its `value` is a free-form string.

**Beginner-Friendly Explanation**
A normal text box where users can type anything.

#### Purposes

- **To collect short free-form strings** (names, titles, usernames).
- **To allow arbitrary input** where no specialised type applies.

#### Syntax Rules and Structure

```html
<input type="text" name="username" minlength="3" maxlength="20" required>
```

**Constraints**: No automatic format validation; use `pattern`, `minlength`, `maxlength`, or script for control.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>text</title></head>
<body>
  <form action="/save" method="post">
    <label for="title">Title</label>
    <input id="title" type="text" name="title" minlength="2" maxlength="80" required>
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: A text field enforcing length 2–80; submitting with one character shows a validation error.

**Why this output occurs**: `minlength`/`maxlength` participate in constraint validation for the text state.

#### Real-World Cases

- Display names, usernames, post titles, search keywords.
- Forms that accept structured-looking strings (for which `pattern` adds validation).

#### References

- MDN Web Docs — `<input type="text">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/text
- WHATWG — Text state — https://html.spec.whatwg.org/multipage/input.html#text-(type=text)-state-and-search-state-(type=search)

---

### 2.2 `password`

#### Definitions

**Core Definition**
The `password` type is a single-line text field whose value is obscured by the user agent.

**Technical Definition**
The *password* state obscures the display of its value but otherwise behaves like a text input. It supports `maxlength`, `minlength`, `pattern`, `placeholder`, `readonly`, `required`, `size`, `autocomplete` (with `new-password`/`current-password` tokens).

**Beginner-Friendly Explanation**
A text box that hides the characters as dots, so nobody looking over your shoulder can read a password.

#### Purposes

- **To collect passwords** without displaying them on screen.

#### Syntax Rules and Structure

```html
<input type="password" name="password" autocomplete="current-password" required>
```

**Constraints**: The value is *not* encrypted; HTTPS is required to protect it in transit.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>password</title></head>
<body>
  <form action="/login" method="post">
    <label for="pw">Password</label>
    <input id="pw" type="password" name="password"
           autocomplete="current-password" minlength="8" required>
    <button>Log in</button>
  </form>
</body>
</html>
```

**Expected output**: Dots appear as you type. Submitting with fewer than 8 characters fails validation.

**Why this output occurs**: The UA masks the value and enforces `minlength`.

#### Real-World Cases

- Login, registration (with `autocomplete="new-password"`), password reset.
- Should *never* be used for non-secret hidden strings — use `hidden` or server-side state.

#### References

- MDN Web Docs — `<input type="password">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/password

---

### 2.3 `email`

#### Definitions

**Core Definition**
The `email` type is a text field for a valid email address; with `multiple`, it accepts a comma-separated list.

**Technical Definition**
The *email* state applies an *email address validation algorithm* — a simplified version of RFC 5322 that permits basic `local@domain` forms and IDN domains. The `multiple` attribute allows a list; browsers auto-strip surrounding whitespace and split on commas.

**Beginner-Friendly Explanation**
A text field that knows what an email address looks like and warns you if you typed something wrong.

#### Purposes

- **To collect one or more email addresses** with format validation.
- **To trigger email-optimised mobile keyboards**.

#### Syntax Rules and Structure

```html
<input type="email" name="email" autocomplete="email" required>
<input type="email" name="cc" multiple>
```

**Constraints**: The validation is deliberately loose (technically `a@b` is accepted); the only real proof is sending a confirmation email.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>email</title></head>
<body>
  <form action="/subscribe" method="post">
    <label for="em">Email</label>
    <input id="em" type="email" name="email" autocomplete="email" required>
    <label for="cc">Notify also (optional, comma-separated)</label>
    <input id="cc" type="email" name="cc" multiple>
    <button>Subscribe</button>
  </form>
</body>
</html>
```

**Expected output**: `abc` triggers a validation error; `a@b.com` passes; the `cc` field accepts `a@b.com, c@d.com`.

**Why this output occurs**: The email state uses the standard email validation algorithm; `multiple` enables comma-separated values.

#### Real-World Cases

- Newsletters, contact forms, account recovery addresses.
- Invitation forms that accept several recipients at once.

#### References

- MDN Web Docs — `<input type="email">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/email
- WHATWG — E-mail state — https://html.spec.whatwg.org/multipage/input.html#e-mail-state-(type=email)

---

### 2.4 `url`

#### Definitions

**Core Definition**
The `url` type is a text field constrained to a valid absolute URL.

**Technical Definition**
The *URL* state requires a value matching the URL parser with a non-empty scheme (e.g. `https://`). It supports `maxlength`, `minlength`, `pattern`, `placeholder`, `readonly`, `required`, `size`.

**Beginner-Friendly Explanation**
A text box that only accepts a proper web address.

#### Purposes

- **To collect absolute URLs** (websites, links, redirect targets).

#### Syntax Rules and Structure

```html
<input type="url" name="website" placeholder="https://example.com" required>
```

**Constraints**: Relative URLs are invalid; only absolute URLs pass native validation.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>url</title></head>
<body>
  <form action="/save" method="post">
    <label for="site">Website</label>
    <input id="site" type="url" name="website" placeholder="https://example.com" required>
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: `example.com` fails validation; `https://example.com` passes.

**Why this output occurs**: The URL parser requires a scheme for the value to be valid.

#### Real-World Cases

- Profile website fields, redirect target configuration.
- Backlink submissions and partner referral forms.

#### References

- MDN Web Docs — `<input type="url">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/url

---

### 2.5 `tel`

#### Definitions

**Core Definition**
The `tel` type is a text field for a telephone number; the browser does **not** validate the format.

**Technical Definition**
The *telephone* state has no automatic validation. It supports `maxlength`, `minlength`, `pattern`, `placeholder`, `readonly`, `required`, `size`. Its main benefit is invoking a telephone-optimised virtual keyboard on mobile devices.

**Beginner-Friendly Explanation**
A phone-number field that brings up the numeric keypad on phones. It doesn’t check whether the number is real.

#### Purposes

- **To collect phone numbers** with a convenient keyboard.
- **To avoid over-constraining formats** that vary worldwide.

#### Syntax Rules and Structure

```html
<input type="tel" name="phone" pattern="\+?[0-9 ()-]{7,20}" autocomplete="tel">
```

**Constraints**: No built-in validation; use `pattern` if you want a format check.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>tel</title></head>
<body>
  <form action="/contact" method="post">
    <label for="tel">Phone</label>
    <input id="tel" type="tel" name="phone"
           pattern="\+?[0-9 ()-]{7,20}"
           placeholder="+1 (555) 123-4567"
           autocomplete="tel">
    <button>Send</button>
  </form>
</body>
</html>
```

**Expected output**: `123` fails the pattern; `+1 (555) 123-4567` passes.

**Why this output occurs**: `pattern` supplies what the browser does not.

#### Real-World Cases

- Contact and delivery forms where numbers vary internationally.
- Two-factor authentication backup numbers.

#### References

- MDN Web Docs — `<input type="tel">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/tel

---

### 2.6 `number`

#### Definitions

**Core Definition**
The `number` type is a numeric input with optional spinners and built-in range/step validation.

**Technical Definition**
The *number* state uses the *floating-point number* value sanitisation algorithm; its `valueAsNumber` property gives a double. It supports `min`, `max`, `step`, `required`, `readonly`. Invalid non-numeric text is treated as an empty (bad input) value for validation.

**Beginner-Friendly Explanation**
A box that only accepts numbers, with arrows to increase or decrease the value.

#### Purposes

- **To collect numeric quantities** with built-in constraints.
- **To allow fine control** via `step` and keyboard arrows.

#### Syntax Rules and Structure

```html
<input type="number" name="qty" min="1" max="10" step="1" value="1">
```

**Constraints**: The value must be a valid floating-point number; otherwise the input is invalid. Localisation of decimal separators varies by browser locale.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>number</title></head>
<body>
  <form action="/order" method="post">
    <label for="qty">Quantity</label>
    <input id="qty" type="number" name="qty" min="1" max="10" step="1" value="1" required>
    <button>Order</button>
  </form>
</body>
</html>
```

**Expected output**: Values below 1 or above 10 fail validation; arrows step by 1.

**Why this output occurs**: `min`/`max`/`step` participate in constraint validation for the number state.

#### Real-World Cases

- Quantities, ages, dimensions.
- Not recommended for phone numbers, postal codes, or credit card numbers (leading zeros are lost).

#### References

- MDN Web Docs — `<input type="number">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/number

---

### 2.7 `range`

#### Definitions

**Core Definition**
The `range` type is a slider for choosing a number within a given range.

**Technical Definition**
The *range* state shares the numeric value algorithm with `number` but renders a slider control. It supports `min`, `max`, `step`; a default of `min=0, max=100, step=1` applies when absent.

**Beginner-Friendly Explanation**
A draggable slider that picks a value between a minimum and maximum.

#### Purposes

- **To provide a low-precision numeric choice** with immediate visual feedback.
- **To avoid typing** for values such as volume or brightness.

#### Syntax Rules and Structure

```html
<input type="range" name="volume" min="0" max="100" step="5" value="50">
```

**Constraints**: Ranges have an *implied* default when attributes are omitted; users cannot type arbitrary values.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>range</title></head>
<body>
  <form action="/settings" method="post">
    <label for="vol">Volume</label>
    <input id="vol" type="range" name="volume" min="0" max="100" step="10" value="50">
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: A slider starting at 50; moving it snaps to multiples of 10 between 0 and 100.

**Why this output occurs**: `step` snaps values; `value` sets initial position.

#### Real-World Cases

- Audio/video volume, brightness, price filters.
- Surveys with 1–10 scales.

#### References

- MDN Web Docs — `<input type="range">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/range

---

### 2.8 `date`

#### Definitions

**Core Definition**
The `date` type accepts a calendar date (year, month, day) in `YYYY-MM-DD` form.

**Technical Definition**
The *date* state uses the *date* value sanitisation algorithm; its `valueAsDate` returns a `Date` or `null`. It supports `min`, `max`, `step` (in days), `required`, `readonly`. The wire format is always `YYYY-MM-DD` regardless of local display.

**Beginner-Friendly Explanation**
A date picker that lets the user choose a day from a calendar.

#### Purposes

- **To collect calendar dates** without manual format checking.

#### Syntax Rules and Structure

```html
<input type="date" name="dob" min="1900-01-01" max="2100-12-31">
```

**Constraints**: The submitted value is always ISO-formatted; browsers with no picker fall back to text and still enforce the format.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>date</title></head>
<body>
  <form action="/book" method="post">
    <label for="d">Booking date</label>
    <input id="d" type="date" name="date" min="2025-01-01" required>
    <button>Book</button>
  </form>
</body>
</html>
```

**Expected output**: A date picker; selecting a date before 2025-01-01 fails validation.

**Why this output occurs**: `min` constrains the date state.

#### Real-World Cases

- Booking, scheduling, date of birth.
- Filtering by date in admin dashboards.

#### References

- MDN Web Docs — `<input type="date">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/date

---

### 2.9 `time`

#### Definitions

**Core Definition**
The `time` type accepts a time of day, in `HH:MM` or `HH:MM:SS` form (optional fractional seconds).

**Technical Definition**
The *time* state uses the *time* value sanitisation algorithm; `valueAsNumber` returns milliseconds since midnight. Supports `min`, `max`, `step` (seconds), `required`, `readonly`.

**Beginner-Friendly Explanation**
A field for choosing a clock time.

#### Purposes

- **To collect times of day** with built-in validation.

#### Syntax Rules and Structure

```html
<input type="time" name="alarm" min="06:00" max="22:00" step="60">
```

**Constraints**: Time zone is local; no date component.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>time</title></head>
<body>
  <form action="/schedule" method="post">
    <label for="start">Start time</label>
    <input id="start" type="time" name="start" min="08:00" max="18:00" required>
    <button>Schedule</button>
  </form>
</body>
</html>
```

**Expected output**: A time picker restricted to 08:00–18:00.

**Why this output occurs**: `min`/`max` define the acceptable time window.

#### Real-World Cases

- Appointment scheduling, shift planning, alarms.

#### References

- MDN Web Docs — `<input type="time">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/time

---

### 2.10 `datetime-local`

#### Definitions

**Core Definition**
The `datetime-local` type accepts a date and a time *without* time-zone information, e.g. `YYYY-MM-DDTHH:MM`.

**Technical Definition**
The *local date and time* state combines date and time algorithms. It is intended for wall-clock timestamps without time zone, and serialises as `YYYY-MM-DDTHH:MM(:SS(.sss))`.

**Beginner-Friendly Explanation**
A field for a specific date *and* time — like “2025-03-15 at 14:30” — with no time zone attached.

#### Purposes

- **To collect combined date/time** values in a single control.

#### Syntax Rules and Structure

```html
<input type="datetime-local" name="meeting" step="900">
```

**Constraints**: No time-zone; servers must interpret it in a known zone.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>datetime-local</title></head>
<body>
  <form action="/meet" method="post">
    <label for="m">Meeting start</label>
    <input id="m" type="datetime-local" name="meeting" step="900" required>
    <button>Create</button>
  </form>
</body>
</html>
```

**Expected output**: A combined picker; `step="900"` snaps to 15-minute intervals.

**Why this output occurs**: The step is measured in seconds for datetime-local.

#### Real-World Cases

- Event planning, meeting scheduling.
- Appointment forms that don’t need time zones.

#### References

- MDN Web Docs — `<input type="datetime-local">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/datetime-local

---

### 2.11 `month`

#### Definitions

**Core Definition**
The `month` type accepts a year and month, in `YYYY-MM` form.

**Technical Definition**
The *month* state uses the *month* value sanitisation algorithm; `valueAsNumber` returns the number of months since January 1970 UTC.

**Beginner-Friendly Explanation**
A picker for a month in a year — like “2025-06” — without a specific day.

#### Purposes

- **To collect month-level periods** (expiry dates, reporting months).

#### Syntax Rules and Structure

```html
<input type="month" name="expiry" min="2025-01" max="2035-12">
```

**Constraints**: No day component; wire format `YYYY-MM`.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>month</title></head>
<body>
  <form action="/report" method="get">
    <label for="r">Reporting month</label>
    <input id="r" type="month" name="month" min="2020-01" required>
    <button>Run report</button>
  </form>
</body>
</html>
```

**Expected output**: A month picker; submitting yields `?month=2025-06` (for example).

**Why this output occurs**: The month state serialises exactly in `YYYY-MM`.

#### Real-World Cases

- Billing cycles and credit-card expiry.
- Monthly analytics and reports.

#### References

- MDN Web Docs — `<input type="month">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/month

---

### 2.12 `week`

#### Definitions

**Core Definition**
The `week` type accepts an ISO week (year and week number), in `YYYY-Www` form (e.g. `2025-W07`).

**Technical Definition**
The *week* state uses the *week* value sanitisation algorithm; `valueAsNumber` returns the number of weeks since 1970-W01.

**Beginner-Friendly Explanation**
A picker for a calendar week — like “2025 week 07”.

#### Purposes

- **To collect week-level periods**.

#### Syntax Rules and Structure

```html
<input type="week" name="week" min="2025-W01" max="2025-W52">
```

**Constraints**: ISO 8601 week numbering; the year can differ from the calendar year around January 1.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>week</title></head>
<body>
  <form action="/timesheet" method="post">
    <label for="w">Week</label>
    <input id="w" type="week" name="week" required>
    <button>Submit</button>
  </form>
</body>
</html>
```

**Expected output**: A week picker; submitting sends `week=2025-W07` for the seventh ISO week of 2025.

**Why this output occurs**: Wire format uses `YYYY-Www`.

#### Real-World Cases

- Timesheets, sprint planning, weekly reporting.

#### References

- MDN Web Docs — `<input type="week">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/week

---

### 2.13 `color`

#### Definitions

**Core Definition**
The `color` type provides a colour picker, returning a 7-character lowercase `#rrggbb` value.

**Technical Definition**
The *colour* state uses the *simple colour* value sanitisation algorithm. Its value is always `#rrggbb` in lowercase; alpha channels are not supported by the standard. Browsers may present additional UI (alpha/eyedropper).

**Beginner-Friendly Explanation**
A field that opens a colour picker and returns a colour code like `#3366cc`.

#### Purposes

- **To let users pick a colour** visually.

#### Syntax Rules and Structure

```html
<input type="color" name="theme" value="#3366cc">
```

**Constraints**: Value must be a valid simple colour; the browser normalises it to lowercase.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>color</title></head>
<body>
  <form action="/theme" method="post">
    <label for="c">Theme colour</label>
    <input id="c" type="color" name="theme" value="#3366cc">
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: A colour swatch opens the system picker; submitting sends `theme=%233366cc`.

**Why this output occurs**: The wire format is `#rrggbb`; `#` is URL-encoded in urlencoded submissions.

#### Real-World Cases

- Theme/branding pickers, drawing apps, chart configuration.

#### References

- MDN Web Docs — `<input type="color">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/color

---

### 2.14 `file`

#### Definitions

**Core Definition**
The `file` type lets users choose files from their device; the selected files are exposed via the `files` property as a `FileList`.

**Technical Definition**
The *file upload* state returns a `FileList` whose entries are `File` objects. The `accept` attribute hints allowed MIME types/extensions; `multiple` allows multiple selections. The form must use `enctype="multipart/form-data"` to transmit file bytes. The `value` property can only be set to the empty string programmatically for security reasons.

**Beginner-Friendly Explanation**
A button that opens the file picker so users can upload documents or pictures.

#### Purposes

- **To upload files** to a server.
- **To constrain** the picker by type and quantity.

#### Syntax Rules and Structure

```html
<form method="post" enctype="multipart/form-data">
  <input type="file" name="docs" accept=".pdf,image/*" multiple required>
</form>
```

**Constraints**: Must use multipart encoding; you cannot prefill `value` except to empty; file type validation on the client is only a hint.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>file</title></head>
<body>
  <form action="/upload" method="post" enctype="multipart/form-data">
    <label for="att">Attachments</label>
    <input id="att" type="file" name="attachments" accept=".pdf,image/*" multiple required>
    <button>Upload</button>
  </form>
  <script>
    const input = document.getElementById('att');
    input.addEventListener('change', () => {
      console.log('Selected files:', [...input.files].map(f => f.name));
    });
  </script>
</body>
</html>
```

**Expected output**: The picker allows PDFs and images; multiple selection; the console logs the selected filenames.

**Why this output occurs**: `accept` filters choices; `multiple` enables several files; the `files` property exposes the selection.

#### Real-World Cases

- Profile picture upload, document submission, media upload in CMS.

#### References

- MDN Web Docs — `<input type="file">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/file
- WHATWG — File upload state — https://html.spec.whatwg.org/multipage/input.html#file-upload-state-(type=file)

---

### 2.15 `checkbox`

#### Definitions

**Core Definition**
The `checkbox` type is an independently toggleable control; it can be checked or unchecked, and multiple checkboxes may be checked at once.

**Technical Definition**
The *checkbox* state has the `checked` content attribute and `indeterminate` IDL property. If checked, its `value` (default `on`) is submitted; if unchecked, nothing is submitted. It supports `required` — a checkbox group requires at least one checked member if any member of the same-named group has `required`.

**Beginner-Friendly Explanation**
A tick box: on or off, independent of other tick boxes.

#### Purposes

- **To allow independent yes/no choices**.
- **To let users pick zero or more options** from a list.

#### Syntax Rules and Structure

```html
<input type="checkbox" id="terms" name="terms" value="accepted" required>
<label for="terms">I accept the terms</label>
```

**Constraints**: Unchecked checkboxes submit nothing; checkboxes must use `<label>` for accessible names.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>checkbox</title></head>
<body>
  <form action="/signup" method="post">
    <fieldset>
      <legend>Interests</legend>
      <label><input type="checkbox" name="interest" value="js" checked> JavaScript</label>
      <label><input type="checkbox" name="interest" value="css"> CSS</label>
      <label><input type="checkbox" name="interest" value="html"> HTML</label>
    </fieldset>
    <label><input type="checkbox" name="terms" required> I accept the terms</label>
    <button>Sign up</button>
  </form>
</body>
</html>
```

**Expected output**: Ticking JS and HTML and submitting yields `interest=js&interest=html&terms=on`; leaving the terms unticked prevents submission.

**Why this output occurs**: Each checked checkbox produces an entry; `required` on the terms checkbox enforces acceptance.

#### Real-World Cases

- Filter lists, newsletter preferences, T&C acceptance.

#### References

- MDN Web Docs — `<input type="checkbox">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox

---

### 2.16 `radio`

#### Definitions

**Core Definition**
The `radio` type is a mutually exclusive choice: only one radio button in a group may be selected at a time.

**Technical Definition**
The *radio button* state groups radios sharing the same `name` and the same form owner. Selecting one unselects others. A radio group is *required* if any of its radios has `required`; the selected radio contributes its value.

**Beginner-Friendly Explanation**
A set of small circles where you can pick only one.

#### Purposes

- **To collect single-choice values** from a small set of options.

#### Syntax Rules and Structure

```html
<input type="radio" id="a" name="size" value="s" checked>
<label for="a">Small</label>
<input type="radio" id="b" name="size" value="m">
<label for="b">Medium</label>
```

**Constraints**: All radios in a group must share `name`; only the checked one submits.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>radio</title></head>
<body>
  <form action="/order" method="post">
    <fieldset>
      <legend>Size</legend>
      <label><input type="radio" name="size" value="s"> Small</label>
      <label><input type="radio" name="size" value="m" checked> Medium</label>
      <label><input type="radio" name="size" value="l"> Large</label>
    </fieldset>
    <button>Order</button>
  </form>
</body>
</html>
```

**Expected output**: Only the checked radio submits (`size=m`); choosing another replaces it.

**Why this output occurs**: Radio groups enforce mutual exclusivity by shared `name`.

#### Real-World Cases

- Size choice, payment method, shipping speed, plan selection.

#### References

- MDN Web Docs — `<input type="radio">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/radio

---

### 2.17 `hidden`

#### Definitions

**Core Definition**
The `hidden` type provides a non-visible form field whose value is submitted with the form.

**Technical Definition**
The *hidden* state is not rendered, not focusable, and cannot be activated. It is a successful control if named. Its value can be set declaratively or by script. `hidden` has no constraint validation (the `required` attribute does not apply).

**Beginner-Friendly Explanation**
A value stored invisibly in the form — like an ID or a CSRF token — that travels with the submission.

#### Purposes

- **To carry metadata** (IDs, tokens, redirect URLs) with submissions.
- **To preserve state** across multi-page forms.

#### Syntax Rules and Structure

```html
<input type="hidden" name="csrf_token" value="…">
<input type="hidden" name="product_id" value="42">
```

**Constraints**: Value is visible in page source; never store secrets.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>hidden</title></head>
<body>
  <form action="/buy" method="post">
    <input type="hidden" name="product_id" value="42">
    <input type="hidden" name="csrf_token" value="abc123">
    <label for="q">Quantity</label>
    <input id="q" type="number" name="qty" value="1" min="1">
    <button>Buy</button>
  </form>
</body>
</html>
```

**Expected output**: Only the quantity field is visible; submitting sends `product_id=42&csrf_token=abc123&qty=1`.

**Why this output occurs**: Hidden inputs are successful controls with fixed values.

#### Real-World Cases

- CSRF tokens, product IDs, multi-step form state, A/B-test bucket markers.

#### References

- MDN Web Docs — `<input type="hidden">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/hidden

---

### 2.18 `submit`

#### Definitions

**Core Definition**
The `submit` type renders a button that submits the form when activated.

**Technical Definition**
The *submit button* state is a button whose activation behaviour is to submit the form (firing the `submit` event and running validation unless suppressed). Its `value` becomes the button's label (in `input`) and is included in the submission as an entry if it has a `name`.

**Beginner-Friendly Explanation**
A button that sends the form.

#### Purposes

- **To provide a native submit trigger**.

#### Syntax Rules and Structure

```html
<input type="submit" value="Save changes">
```

**Constraints**: Prefer `<button type="submit">` for richer content and consistent styling; use `form*` attributes (e.g. `formaction`) to override the form's settings.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>submit</title></head>
<body>
  <form action="/save" method="post">
    <label for="t">Title</label>
    <input id="t" name="title" required>
    <input type="submit" value="Save">
  </form>
</body>
</html>
```

**Expected output**: A "Save" button that validates and submits.

**Why this output occurs**: The submit button triggers the submission algorithm.

#### Real-World Cases

- Simple forms where `<input type="submit">` is sufficient.

#### References

- MDN Web Docs — `<input type="submit">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/submit

---

### 2.19 `reset`

#### Definitions

**Core Definition**
The `reset` type renders a button that resets the form to its initial values.

**Technical Definition**
The *reset button* state fires a cancellable `reset` event and, unless cancelled, resets controls to their default values (content-attribute initial state). Reset buttons are not included in submissions.

**Beginner-Friendly Explanation**
A button that undoes all your typing, restoring the form to how it looked when loaded.

#### Purposes

- **To let users clear form entries** in one click.

#### Syntax Rules and Structure

```html
<input type="reset" value="Clear form">
```

**Constraints**: Resets are destructive and often harmful to UX; use sparingly.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>reset</title></head>
<body>
  <form action="/save" method="post">
    <label for="n">Name</label>
    <input id="n" name="name" value="Alice">
    <input type="reset" value="Reset">
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: Changing "Alice" to something else and clicking Reset restores "Alice".

**Why this output occurs**: Reset restores the `value` content attributes as the default values.

#### Real-World Cases

- Rarely recommended; mostly legacy.

#### References

- MDN Web Docs — `<input type="reset">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/reset

---

### 2.20 `button`

#### Definitions

**Core Definition**
The `button` type renders a button with no default behaviour.

**Technical Definition**
The *button* state is a generic button that does nothing unless given a script handler. It is not a submit or reset button; it is form-associated but does not submit.

**Beginner-Friendly Explanation**
A button with no default action — you attach an onclick to make it do something.

#### Purposes

- **To provide a trigger for scripts**.

#### Syntax Rules and Structure

```html
<input type="button" value="Do something" onclick="doIt()">
```

**Constraints**: Prefer `<button type="button">` for rich content and easier styling.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>button</title></head>
<body>
  <form action="/save" method="post">
    <input id="t" name="title">
    <input type="button" value="Preview" onclick="alert(document.getElementById('t').value)">
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: "Preview" shows an alert with the current title; "Save" submits the form.

**Why this output occurs**: `type="button"` performs no default action; the `onclick` script runs on click.

#### Real-World Cases

- Preview actions, custom widgets, “add another” controls.

#### References

- MDN Web Docs — `<input type="button">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/button

---

### 2.21 `image`

#### Definitions

**Core Definition**
The `image` type is a submit button rendered as an image.

**Technical Definition**
The *image button* state submits the form when clicked, including the coordinates `name.x` and `name.y` in the submission when the `name` attribute is present. `src` is required; `alt` is strongly recommended.

**Beginner-Friendly Explanation**
A clickable image that submits the form.

#### Purposes

- **To submit forms with an image button** (legacy, decorative).

#### Syntax Rules and Structure

```html
<input type="image" src="submit.png" alt="Submit" width="120" height="32">
```

**Constraints**: `alt` and `src` are required; coordinates are submitted as `<name>.x` and `<name>.y` if `name` is set.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>image</title></head>
<body>
  <form action="/search" method="get">
    <input name="q" value="test">
    <input type="image" src="go.png" alt="Submit search" name="go" width="40" height="40">
  </form>
</body>
</html>
```

**Expected output**: Clicking the image submits to `/search?q=test&go.x=20&go.y=10` (coordinates vary).

**Why this output occurs**: Image buttons add click coordinates to the submission when named.

#### Real-World Cases

- Legacy image submit buttons; custom-branded buttons (prefer CSS-styled `<button>` now).

#### References

- MDN Web Docs — `<input type="image">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/image

---

### 2.22 `search`

#### Definitions

**Core Definition**
The `search` type is a text field semantically intended for search queries.

**Technical Definition**
The *search* state shares behaviour with `text`, but exposes additional UI (e.g. clear button in Safari) and has the semantic role `searchbox`. It supports `list`, `maxlength`, `minlength`, `pattern`, `placeholder`, `readonly`, `required`, `size`.

**Beginner-Friendly Explanation**
A text box for searching, often with an X button to clear it.

#### Purposes

- **To mark fields as search queries** for accessibility and UA UI.

#### Syntax Rules and Structure

```html
<input type="search" name="q" placeholder="Search…">
```

**Constraints**: Fallback is text where unsupported; form must still be GET for shareable results.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>search</title></head>
<body>
  <form action="/search" method="get">
    <label for="q">Search</label>
    <input id="q" type="search" name="q" placeholder="Type keywords">
    <button>Search</button>
  </form>
</body>
</html>
```

**Expected output**: A search-style field; submitting yields `?q=…`.

**Why this output occurs**: The search state provides UI hints and shares text-submission semantics.

#### Real-World Cases

- Site search, filters, admin tables.

#### References

- MDN Web Docs — `<input type="search">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/search

---

## 3. Input Attributes

Each attribute below follows the same uniform structure. Attributes apply only to specific types unless noted.

---

### 3.1 `type`

#### Definitions

**Core Definition**
`type` selects the input state, determining UI, value semantics, and applicable attributes.

**Technical Definition**
Enumerated attribute with keywords matching the 22 standard input states; unknown keywords fall back to `text` (the *invalid value default*).

**Beginner-Friendly Explanation**
It decides what kind of input box it is.

#### Purposes

- **To select the input's behaviour**.

#### Syntax Rules and Structure

```html
<input type="email">
```

**Constraints**: Certain attributes only apply to certain types; type-specific value sanitisation runs on each value change.

#### Annotated Example

```html
<input type="email" name="email" required>
```

**Expected output**: Email keyboard on mobile, format validation on submit.

**Why this output occurs**: The `email` state applies email validation and mobile hints.

#### Real-World Cases

- Everywhere; `type` is the primary discriminator.

#### References

- MDN Web Docs — `type` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#type

---

### 3.2 `name`

#### Definitions

**Core Definition**
`name` is the key under which the input's value is submitted.

**Technical Definition**
Content attribute reflected to `HTMLInputElement.name`. Only named, successful controls contribute to the entry list. Radio groups share `name`.

**Beginner-Friendly Explanation**
The label on the answer so the server knows what it is.

#### Purposes

- **To identify the value on submission**.
- **To group radios** by shared name.

#### Syntax Rules and Structure

```html
<input name="email" type="email">
```

**Constraints**: Unnamed inputs never submit; duplicate names for non-radio inputs can be valid but ambiguous.

#### Annotated Example

```html
<input type="text" name="first_name" value="Ada">
```

**Expected output**: Submits `first_name=Ada`.

**Why this output occurs**: The `name` provides the key.

#### Real-World Cases

- Every form control in production.

#### References

- MDN Web Docs — `name` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#name

---

### 3.3 `value`

#### Definitions

**Core Definition**
`value` is the control's current/default value as a string.

**Technical Definition**
Content attribute reflected to `HTMLInputElement.value` as the *default value*, but the live *value* is the IDL property's current value (dirty value flag). For non-text types (number, date, etc.) the content attribute provides a default that is parsed/sanitised against the type's grammar.

**Beginner-Friendly Explanation**
The starting text inside the field.

#### Purposes

- **To prefill controls**.
- **To set button labels** (submit/reset/button).

#### Syntax Rules and Structure

```html
<input type="text" value="Hello">
<input type="submit" value="Save">
```

**Constraints**: Programmatically setting `value` does not update the `value` attribute (and vice versa); use `defaultValue`/`value` to control separately.

#### Annotated Example

```html
<input type="text" name="city" value="Manila">
```

**Expected output**: Field starts with "Manila".

**Why this output occurs**: The content attribute seeds the initial value.

#### Real-World Cases

- Edit forms pre-populated with server data.

#### References

- MDN Web Docs — `value` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#value

---

### 3.4 `placeholder`

#### Definitions

**Core Definition**
`placeholder` is a hint shown inside the field when empty.

**Technical Definition**
Content attribute applicable to text-like types; not a label and not a substitute for one. It is not announced reliably on all ATs and is not used by validation.

**Beginner-Friendly Explanation**
Faint grey example text inside an empty box.

#### Purposes

- **To give a formatting hint**.

#### Syntax Rules and Structure

```html
<input type="email" placeholder="you@example.com">
```

**Constraints**: Do not rely on it for labelling; it disappears on typing; screenshots/autofill may show it.

#### Annotated Example

```html
<input type="tel" name="phone" placeholder="+1 (555) 123-4567">
```

**Expected output**: Hint appears; typing clears it.

**Why this output occurs**: Placeholder is a presentation aid only.

#### Real-World Cases

- Search boxes, format hints (never the only label).

#### References

- MDN Web Docs — `placeholder` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#placeholder

---

### 3.5 `required`

#### Definitions

**Core Definition**
`required` marks a control as mandatory.

**Technical Definition**
Boolean constraint attribute. For text-like types, the value must be non-empty; for checkbox, it must be checked; for radio, at least one in the same-named group must be checked. It participates in constraint validation before submission (unless novalidate/formnovalidate).

**Beginner-Friendly Explanation**
Makes the field compulsory.

#### Purposes

- **To enforce mandatory data**.

#### Syntax Rules and Structure

```html
<input type="text" name="username" required>
```

**Constraints**: Client-side only; must be re-checked on the server.

#### Annotated Example

```html
<input type="email" name="email" required>
```

**Expected output**: Submitting empty triggers the browser's "please fill out" bubble.

**Why this output occurs**: `required` sets a *valueMissing* validity state.

#### Real-World Cases

- Sign-up fields, T&C acceptance, mandatory survey answers.

#### References

- MDN Web Docs — `required` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/required

---

### 3.6 `disabled`

#### Definitions

**Core Definition**
`disabled` makes a control non-interactive, unfocusable, and non-submitting.

**Technical Definition**
Boolean attribute; sets `disabled` IDL. Disabled controls are barred from constraint validation and from the submission entry list. They are still rendered (usually greyed out).

**Beginner-Friendly Explanation**
Greys out a field and stops it from being used or sent.

#### Purposes

- **To reflect an unavailable state** (e.g. plan limitations).

#### Syntax Rules and Structure

```html
<input type="text" name="coupon" disabled>
```

**Constraints**: Its value is *never* submitted; if you need the value, use `readonly` instead.

#### Annotated Example

```html
<label>Coupon <input name="coupon" value="SAVE10" disabled></label>
```

**Expected output**: Field is greyed out; no `coupon` entry on submit.

**Why this output occurs**: Disabled controls are not successful.

#### Real-World Cases

- Temporarily unavailable options, plan-gated features.

#### References

- MDN Web Docs — `disabled` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/disabled

---

### 3.7 `readonly`

#### Definitions

**Core Definition**
`readonly` makes a control's value non-editable by the user but still submitted and focusable.

**Technical Definition**
Boolean attribute applicable to text-like types (not checkbox/radio/file/buttons). A read-only control's value participates in constraint validation (except `required`, which is ignored for read-only). The user can still focus and select, and it is submitted normally.

**Beginner-Friendly Explanation**
A locked field you can read but not change; its value still goes to the server.

#### Purposes

- **To display a value the user shouldn't edit** while still submitting it.

#### Syntax Rules and Structure

```html
<input type="text" name="account_id" value="A-42" readonly>
```

**Constraints**: Not supported on checkbox, radio, file, range, colour, or button-like types; `required` has no effect.

#### Annotated Example

```html
<input name="order_no" value="SO-2025-0001" readonly>
```

**Expected output**: Field is selectable but not editable; submits `order_no=SO-2025-0001`.

**Why this output occurs**: The value is preserved and included in the entry list.

#### Real-World Cases

- Auto-generated IDs, pre-filled but fixed values.

#### References

- MDN Web Docs — `readonly` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/readonly

---

### 3.8 `autocomplete`

#### Definitions

**Core Definition**
`autocomplete` hints whether the UA may auto-fill a field and with what kind of data.

**Technical Definition**
Enumerated attribute. Values include `on`/`off` and standard tokens like `email`, `name`, `tel`, `street-address`, `cc-number`, `current-password`, `new-password`, `one-time-code`. Browsers may ignore `off` for password fields.

**Beginner-Friendly Explanation**
Tells the browser whether to help fill this field, and with what kind of saved info.

#### Purposes

- **To improve autofill reliability** with tokens.
- **To disable autofill for sensitive/non-standard fields**.

#### Syntax Rules and Structure

```html
<input type="email" autocomplete="email">
<input type="password" autocomplete="new-password">
```

**Constraints**: Tokens must be spelled correctly to be useful; `off` is a hint, not a guarantee.

#### Annotated Example

```html
<input type="text" name="city" autocomplete="address-level2">
```

**Expected output**: Password managers and browser profiles offer the city value.

**Why this output occurs**: Recognised tokens map to fields in the user's profile.

#### Real-World Cases

- Login/registration, checkout, address forms.

#### References

- MDN Web Docs — `autocomplete` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/autocomplete
- WHATWG — Autofill — https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofill

---

### 3.9 `autofocus`

#### Definitions

**Core Definition**
`autofocus` makes an element receive focus when the page loads.

**Technical Definition**
Boolean attribute. Only one element per document should be autofocused; UAs may refuse (e.g. on mobile to avoid keyboard popup). Not applicable to `hidden` or `disabled` controls.

**Beginner-Friendly Explanation**
Puts the cursor in the field automatically when the page opens.

#### Purposes

- **To speed up single-field flows** (search, login).

#### Syntax Rules and Structure

```html
<input type="search" name="q" autofocus>
```

**Constraints**: Avoid when there are multiple interactive tasks; accessibility guidance advises against autofocus for users with cognitive disabilities.

#### Annotated Example

```html
<form action="/search" method="get">
  <input type="search" name="q" autofocus>
  <button>Search</button>
</form>
```

**Expected output**: The search field is focused on load; typing goes straight in.

**Why this output occurs**: The UA focuses the autofocused control once the document loads.

#### Real-World Cases

- Search pages, single-input login pages.

#### References

- MDN Web Docs — `autofocus` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/autofocus

---

### 3.10 `multiple`

#### Definitions

**Core Definition**
`multiple` allows more than one value in an input.

**Technical Definition**
Boolean attribute applicable to `email`, `file`, and `select`. In `email`, comma-separated lists are accepted; in `file`, several files may be selected; in `<select>`, several options.

**Beginner-Friendly Explanation**
Lets the field hold several values at once.

#### Purposes

- **To collect multiple files, emails, or options**.

#### Syntax Rules and Structure

```html
<input type="file" name="docs" multiple>
<input type="email" name="cc" multiple>
```

**Constraints**: Not supported on checkbox/radio/text/number.

#### Annotated Example

```html
<input type="file" name="photos" accept="image/*" multiple>
```

**Expected output**: The file picker permits selecting several photos; the `files` list contains all of them.

**Why this output occurs**: `multiple` opens a multi-selection picker.

#### Real-World Cases

- Media uploads, bulk email entry, filter selections.

#### References

- MDN Web Docs — `multiple` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/multiple

---

### 3.11 `accept`

#### Definitions

**Core Definition**
`accept` hints which file types are valid for a file input.

**Technical Definition**
Attribute applicable to `<input type="file">`; value is a comma-separated list of unique file type specifiers (MIME types, extensions like `.pdf`, or `audio/*`). It is a *hint* to the picker, not a security filter.

**Beginner-Friendly Explanation**
Tells the file picker which types of files to show.

#### Purposes

- **To filter the file picker** by type.

#### Syntax Rules and Structure

```html
<input type="file" name="avatar" accept="image/png, image/jpeg">
```

**Constraints**: Easily bypassed by renaming files or using drag-and-drop; always validate server-side.

#### Annotated Example

```html
<input type="file" name="cv" accept=".pdf,.doc,.docx">
```

**Expected output**: The picker shows PDFs and Word documents by default.

**Why this output occurs**: Browsers translate specifiers into picker filters.

#### Real-World Cases

- Upload flows for images, PDFs, documents.

#### References

- MDN Web Docs — `accept` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/accept

---

### 3.12 `min`

#### Definitions

**Core Definition**
`min` specifies the minimum acceptable value for numeric/date/time/range types.

**Technical Definition**
Attribute interpreted according to the type: number → smallest float; date/time → smallest timestamp; range → lower bound. Participates in the *rangeUnderflow* validity check.

**Beginner-Friendly Explanation**
The lowest value allowed.

#### Purposes

- **To enforce a lower bound**.

#### Syntax Rules and Structure

```html
<input type="number" min="0">
<input type="date" min="2025-01-01">
```

**Constraints**: Ignored for text/email/etc.

#### Annotated Example

```html
<input type="number" name="qty" min="1" value="1">
```

**Expected output**: Setting 0 triggers rangeUnderflow.

**Why this output occurs**: Value < min invalidates.

#### Real-World Cases

- Quantities, dates in the future, price ranges.

#### References

- MDN Web Docs — `min` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/min

---

### 3.13 `max`

#### Definitions

**Core Definition**
`max` specifies the maximum acceptable value for numeric/date/time/range types.

**Technical Definition**
Attribute interpreted according to the type; participates in *rangeOverflow*.

**Beginner-Friendly Explanation**
The highest value allowed.

#### Purposes

- **To enforce an upper bound**.

#### Syntax Rules and Structure

```html
<input type="number" max="999">
```

**Constraints**: Ignored for non-applicable types.

#### Annotated Example

```html
<input type="date" name="dob" max="2025-12-31">
```

**Expected output**: Selecting a later date invalidates.

**Why this output occurs**: Value > max fails rangeOverflow.

#### Real-World Cases

- Age, quantity limits, booking windows.

#### References

- MDN Web Docs — `max` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/max

---

### 3.14 `step`

#### Definitions

**Core Definition**
`step` specifies the granularity of numeric/date/time values.

**Technical Definition**
Attribute applicable to number, range, date, time, datetime-local, month, week. Value is either a positive float or the keyword `any`. It participates in the *stepMismatch* validation.

**Beginner-Friendly Explanation**
The allowed increment (e.g. multiples of 5 minutes).

#### Purposes

- **To constrain values to a grid**.

#### Syntax Rules and Structure

```html
<input type="number" step="0.01">
<input type="time" step="900"> <!-- 15 minutes -->
```

**Constraints**: `step="any"` disables the check but you may also want to handle precision manually.

#### Annotated Example

```html
<input type="number" name="price" min="0" step="0.01">
```

**Expected output**: 10.005 triggers stepMismatch; 10.01 does not.

**Why this output occurs**: Value must be min + n × step.

#### Real-World Cases

- Currency (0.01), time slots (900 s), quantities (1).

#### References

- MDN Web Docs — `step` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/step

---

### 3.15 `pattern`

#### Definitions

**Core Definition**
`pattern` is a regular expression the entire value must match.

**Technical Definition**
Attribute applicable to text, search, url, tel, email, password. Regex compiled with `v` flag (or `u` in some UAs); implicit anchoring (^…$). Participates in *patternMismatch*.

**Beginner-Friendly Explanation**
A rule describing what the text must look like.

#### Purposes

- **To enforce a format**.

#### Syntax Rules and Structure

```html
<input type="text" name="zip" pattern="\d{5}(-\d{4})?" title="5 or 9-digit ZIP">
```

**Constraints**: Provide a `title` describing the rule; unsupported lookarounds in some engines; not a substitute for server validation.

#### Annotated Example

```html
<input type="text" name="username" pattern="[A-Za-z0-9_]{3,16}" required>
```

**Expected output**: "ab!" fails; "ada_42" passes.

**Why this output occurs**: The regex tests the whole value.

#### Real-World Cases

- Usernames, ZIP/postal codes, coupon codes.

#### References

- MDN Web Docs — `pattern` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/pattern

---

### 3.16 `minlength`

#### Definitions

**Core Definition**
`minlength` requires the value to be at least N characters.

**Technical Definition**
Applies to text-like types. Participates in *tooShort*. Ignored when the value is empty (use `required` for that). Measured in UTF-16 code units.

**Beginner-Friendly Explanation**
The field must have at least N characters.

#### Purposes

- **To enforce a lower bound on length**.

#### Syntax Rules and Structure

```html
<input type="text" name="bio" minlength="10">
```

**Constraints**: Values with emoji may count in code units; pair with `required` for "non-empty".

#### Annotated Example

```html
<input type="password" minlength="8">
```

**Expected output**: 7 characters fails; 8 passes.

**Why this output occurs**: tooShort triggers under the minimum.

#### Real-World Cases

- Password minimums, bio minimums.

#### References

- MDN Web Docs — `minlength` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/minlength

---

### 3.17 `maxlength`

#### Definitions

**Core Definition**
`maxlength` limits the value to at most N characters.

**Technical Definition**
Applies to text-like types; participates in *tooLong*; the UA generally prevents further input beyond the limit.

**Beginner-Friendly Explanation**
The field can hold at most N characters.

#### Purposes

- **To enforce an upper bound on length**.

#### Syntax Rules and Structure

```html
<input type="text" name="tweet" maxlength="280">
```

**Constraints**: Measured in code units (counts emoji as multiple).

#### Annotated Example

```html
<input type="text" name="nickname" maxlength="20">
```

**Expected output**: You cannot type more than 20 characters.

**Why this output occurs**: The UA blocks further input and validates tooLong.

#### Real-World Cases

- Title limits, tweet-like limits, fixed-format fields.

#### References

- MDN Web Docs — `maxlength` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/maxlength

---

### 3.18 `size`

#### Definitions

**Core Definition**
`size` sets the visual width of a text input in “characters”.

**Technical Definition**
Positive integer; affects CSS width in character units (typically ~1ch * size). Does not impose a length limit; use `minlength`/`maxlength` for that.

**Beginner-Friendly Explanation**
Roughly how wide the box appears.

#### Purposes

- **To control the field's visible width**.

#### Syntax Rules and Structure

```html
<input type="text" name="code" size="10">
```

**Constraints**: Presentation only; CSS `width` overrides it.

#### Annotated Example

```html
<input type="text" name="zip" size="6">
```

**Expected output**: A short text box fitting about 6 characters.

**Why this output occurs**: The UA sizes the field accordingly.

#### Real-World Cases

- Compact fields (ZIP, CVV), longer fields (address).

#### References

- MDN Web Docs — `size` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/size

---

### 3.19 `list`

#### Definitions

**Core Definition**
`list` associates an input with a `<datalist>` providing suggested values.

**Technical Definition**
References a `<datalist>` by id. Supported on text, search, url, tel, email, date, month, week, time, datetime-local, number, range, colour. Suggestions appear in a dropdown; the user may still type free-form values (except on range/colour, where the datalist renders tickmarks).

**Beginner-Friendly Explanation**
A dropdown of suggestions the user can pick from or ignore.

#### Purposes

- **To provide autocompletion suggestions** without restricting values.

#### Syntax Rules and Structure

```html
<input list="fruits" name="fruit">
<datalist id="fruits">
  <option value="Apple">
  <option value="Banana">
</datalist>
```

**Constraints**: Suggestions only; enforce validity separately.

#### Annotated Example

```html
<label for="browser">Browser</label>
<input id="browser" name="browser" list="browsers">
<datalist id="browsers">
  <option value="Chrome">
  <option value="Firefox">
  <option value="Safari">
</datalist>
```

**Expected output**: Typing shows Chrome/Firefox/Safari; the user can still enter others.

**Why this output occurs**: The UA uses the datalist as a suggestion source.

#### Real-World Cases

- Country/city pickers, tag input, browser choice.

#### References

- MDN Web Docs — `list` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#list
- MDN Web Docs — `<datalist>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/datalist

---

### 3.20 `form`

#### Definitions

**Core Definition**
`form` associates an input with a form by its `id`, even if the input is outside that form in the DOM.

**Technical Definition**
Attribute holding the id of a form element. If present, the input's form owner becomes that form. Enables controls outside a `<form>` (e.g. after a table, in a portal) to participate in submission.

**Beginner-Friendly Explanation**
Links a field to a form even if it's not inside it.

#### Purposes

- **To decouple placement from association** (e.g. toolbar outside a table).

#### Syntax Rules and Structure

```html
<input form="myform" name="x">
<form id="myform">…</form>
```

**Constraints**: The referenced form must exist; otherwise, the input has no owner.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>form attr</title></head>
<body>
  <input type="text" name="coupon" form="checkout">
  <form id="checkout" action="/pay" method="post">
    <input name="total" value="10">
    <button>Pay</button>
  </form>
</body>
</html>
```

**Expected output**: Submitting includes `coupon` even though it’s outside the form.

**Why this output occurs**: `form="checkout"` gives the input that form owner.

#### Real-World Cases

- Toolbars above wide tables; portal-rendered inputs.

#### References

- MDN Web Docs — `form` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/form

---

### 3.21 `formaction`

#### Definitions

**Core Definition**
`formaction` overrides the form's `action` for a specific submit button.

**Technical Definition**
Applicable to submit-capable controls (`submit`, `image`). Content attribute whose value is a URL, overriding `form.action` for that submission.

**Beginner-Friendly Explanation**
Lets one submit button send to a different URL.

#### Purposes

- **To offer alternative destinations** from one form.

#### Syntax Rules and Structure

```html
<input type="submit" value="Save draft" formaction="/drafts" formnovalidate>
<input type="submit" value="Publish" formaction="/publish">
```

**Constraints**: Applies only to submit-capable controls.

#### Annotated Example

```html
<form action="/default" method="post">
  <input name="title">
  <input type="submit" value="Save" formaction="/save">
  <input type="submit" value="Preview" formaction="/preview">
</form>
```

**Expected output**: "Save" posts to `/save`; "Preview" to `/preview`.

**Why this output occurs**: The submitter's `formaction` overrides the form.

#### Real-World Cases

- Save vs Publish vs Delete buttons.

#### References

- MDN Web Docs — `formaction` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#formaction

---

### 3.22 `formenctype`

#### Definitions

**Core Definition**
`formenctype` overrides `enctype` for a specific submit button.

**Technical Definition**
Applicable to submit controls; enumerated like `enctype`.

**Beginner-Friendly Explanation**
Lets one submit button use a different encoding.

#### Purposes

- **To support file-including vs file-excluding submit paths**.

#### Syntax Rules and Structure

```html
<input type="submit" formenctype="multipart/form-data">
```

**Constraints**: Applies only to submit controls.

#### Annotated Example

```html
<form action="/x" method="post" enctype="application/x-www-form-urlencoded">
  <input type="file" name="f">
  <input type="submit" value="Upload" formenctype="multipart/form-data">
  <input type="submit" value="Text only">
</form>
```

**Expected output**: "Upload" sends multipart; "Text only" sends urlencoded (the file is lost).

**Why this output occurs**: The submitter's formenctype overrides the form.

#### Real-World Cases

- Hybrid forms with optional file input.

#### References

- MDN Web Docs — `formenctype` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#formenctype

---

### 3.23 `formmethod`

#### Definitions

**Core Definition**
`formmethod` overrides `method` for a specific submit button.

**Technical Definition**
Applicable to submit controls; keywords `get`, `post`, `dialog`.

**Beginner-Friendly Explanation**
Lets one submit button use GET while the other uses POST.

#### Purposes

- **To mix idempotent preview and state-changing submit**.

#### Syntax Rules and Structure

```html
<input type="submit" formmethod="get" value="Preview">
<input type="submit" formmethod="post" value="Save">
```

**Constraints**: Only on submit controls.

#### Annotated Example

```html
<form action="/editor" method="post">
  <textarea name="body">Hello</textarea>
  <input type="submit" value="Preview" formmethod="get" formaction="/preview">
  <input type="submit" value="Publish" formaction="/publish">
</form>
```

**Expected output**: "Preview" navigates with query string; "Publish" posts.

**Why this output occurs**: formmethod overrides form.method.

#### Real-World Cases

- Editor previews, read-only vs mutating operations.

#### References

- MDN Web Docs — `formmethod` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#formmethod

---

### 3.24 `formnovalidate`

#### Definitions

**Core Definition**
`formnovalidate` disables constraint validation for a specific submission.

**Technical Definition**
Boolean attribute applicable to submit controls; equivalent to setting `novalidate` on the form for this submission only.

**Beginner-Friendly Explanation**
Skips checking for this button only.

#### Purposes

- **To allow Save-draft buttons to bypass validation**.

#### Syntax Rules and Structure

```html
<input type="submit" value="Save draft" formnovalidate>
```

**Constraints**: Only on submit controls; still submit normally.

#### Annotated Example

```html
<form action="/post" method="post">
  <input name="title" required>
  <input type="submit" value="Publish">
  <input type="submit" value="Save draft" formnovalidate formaction="/drafts">
</form>
```

**Expected output**: "Publish" enforces `required`; "Save draft" submits even when empty.

**Why this output occurs**: The submitter's formnovalidate suppresses validation.

#### Real-World Cases

- Draft saving, progressive workflows.

#### References

- MDN Web Docs — `formnovalidate` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#formnovalidate

---

### 3.25 `formtarget`

#### Definitions

**Core Definition**
`formtarget` overrides `target` for a specific submit button.

**Technical Definition**
Applicable to submit controls; value is a browsing-context name.

**Beginner-Friendly Explanation**
Lets a specific button open the result in a different tab.

#### Purposes

- **To choose per-submit response destination**.

#### Syntax Rules and Structure

```html
<input type="submit" formtarget="_blank" value="Open in new tab">
```

**Constraints**: Only on submit controls.

#### Annotated Example

```html
<form action="/report" method="get">
  <input name="q">
  <input type="submit" value="Search">
  <input type="submit" value="Open in new tab" formtarget="_blank">
</form>
```

**Expected output**: First button replaces the page; second opens a new tab.

**Why this output occurs**: formtarget overrides form.target.

#### Real-World Cases

- "Compare" or "Open report" secondary actions.

#### References

- MDN Web Docs — `formtarget` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#formtarget

---

### 3.26 `dirname`

#### Definitions

**Core Definition**
`dirname` submits the text direction of a field as an extra name/value pair.

**Technical Definition**
Applies to text/search inputs and to `<textarea>`. The submitted key is the `dirname` value; the value is `ltr` or `rtl` reflecting the element's computed directionality.

**Beginner-Friendly Explanation**
Also submits "this was typed left-to-right" or "right-to-left".

#### Purposes

- **To preserve directionality** for multilingual content.

#### Syntax Rules and Structure

```html
<input type="text" name="comment" dirname="comment.dir">
```

**Constraints**: Submits an extra entry `comment.dir=ltr|rtl`.

#### Annotated Example

```html
<form action="/save" method="post">
  <input type="text" name="comment" dirname="comment.dir" value="مرحبا">
  <button>Save</button>
</form>
```

**Expected output**: POST includes `comment=مرحبا&comment.dir=rtl`.

**Why this output occurs**: The UA computes directionality and appends it.

#### Real-World Cases

- Multilingual comment systems, chat.

#### References

- MDN Web Docs — `dirname` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input#dirname

---

## 4. Input Best Practices

### Definitions

**Core Definition**
Input best practices are guidelines for producing inputs that are accessible, usable, safe, and maintainable beyond raw specification conformance.

**Technical Definition**
They combine HTML conformance with WCAG success criteria, platform conventions (mobile keyboards, autofill), and security guidance (server validation, no secrets in hidden fields, file-type validation). They also cover programmatic hooks (`checkValidity`, `setCustomValidity`, `formdata`).

**Beginner-Friendly Explanation**
The habits that make fields easy for everyone to use and safe for the server.

---

### Purposes

- **To maximise accessibility and mobile usability**.
- **To make errors obvious and recoverable**.
- **To improve autofill reliability and reduce typing**.
- **To prevent security issues** (client-trusted data, file-type bypass, secret leakage).
- **To enable progressive enhancement**.

---

### Syntax Rules and Structure

Best practices are principles; the table summarises the core rules.

| Practice | Rule |
|---|---|
| **Always label inputs** | `<label for>`, wrapping label, `aria-label`, or `aria-labelledby`. |
| **Pick the right type** | Use `email`, `tel`, `url`, `date`, `number`, etc. to bring UI and validation. |
| **Add `autocomplete` tokens** | `email`, `current-password`, `new-password`, `one-time-code`, `cc-number`, etc. |
| **Group related controls** | `<fieldset>` + `<legend>` for radios/checkboxes. |
| **Associate errors** | `aria-describedby` + inline messages; focus the first invalid control. |
| **Validate on both sides** | Client-side for UX, server-side for security. |
| **Never store secrets in `hidden`** | Page source is readable; use server-side state. |
| **Constrain files correctly** | `accept` for UX, but validate type/size/extensions server-side. |
| **Avoid `readonly` for security** | Users can modify via dev tools; enforce on server. |
| **Prefer native UI** | `<input type="date">` beats hand-rolled pickers for a11y. |
| **Don't disable paste** | It breaks password managers and harms usability. |
| **Use `<button type="submit">`** | More styling/content options than `<input type="submit">`. |
| **Handle double submission** | Disable submit button or use idempotency keys server-side. |
| **Avoid autofocus overload** | Use only for single-field flows; never in multi-task pages. |

#### Syntax Rules

1. Every control must have an accessible name and, when relevant, a description.
2. Errors must be programmatically associated with their fields and surfaced non-visually (via `role="alert"` or `aria-live`).
3. Server-side validation must mirror every client-side constraint.
4. Inputs must work without JavaScript whenever the form does.

#### Constraints and Limitations

- Native UI varies by browser/OS; visual consistency requires CSS work.
- Autofill support varies; recognised tokens are essential.
- Some types (e.g. `datetime-local`, `week`) have weaker support in older browsers; provide fallbacks.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible, Mobile-Friendly Form

**Setup guide**: Save as `input-best-practices.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Input Best Practices</title>
  <style>
    label { display: block; margin-top: .75rem; font-weight: 600; }
    .error { color: #b00020; font-size: .9rem; }
    input { padding: .5rem; width: 100%; max-width: 24rem; box-sizing: border-box; }
  </style>
</head>
<body>
  <form action="/register" method="post" novalidate>
    <fieldset>
      <legend>Account</legend>

      <label for="email">Email</label>
      <input id="email" type="email" name="email"
             autocomplete="email" inputmode="email"
             aria-describedby="email-help email-err" required>
      <small id="email-help">We'll never share your email.</small>
      <p id="email-err" class="error" hidden></p>

      <label for="pw">Password</label>
      <input id="pw" type="password" name="password"
             autocomplete="new-password"
             minlength="8" aria-describedby="pw-err" required>
      <p id="pw-err" class="error" hidden></p>
    </fieldset>

    <button type="submit">Create account</button>
  </form>

  <script>
    const form = document.querySelector('form');
    // Inline error display on submit
    form.addEventListener('submit', (e) => {
      // Clear old errors
      form.querySelectorAll('.error').forEach(p => p.hidden = true);
      if (!form.checkValidity()) {
        e.preventDefault();
        form.querySelectorAll(':invalid').forEach(el => {
          const err = document.getElementById(el.id + '-err');
          if (err) { err.hidden = false; err.textContent = el.validationMessage; }
        });
        form.querySelector(':invalid')?.focus();
      }
    });
  </script>
</body>
</html>
```

**Expected output**: Submitting empty/invalid values reveals inline error messages and focuses the first invalid field. Valid values submit natively.

**Why this output occurs**: `checkValidity()` inspects all constraints; `validationMessage` provides a user-facing explanation; `:invalid` finds failing controls; focus management improves keyboard UX.

---

#### Example 2: Autofill-Optimised Checkout Fields

**Setup guide**: Save as `checkout.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Checkout</title>
</head>
<body>
  <form action="/pay" method="post" autocomplete="on">
    <label for="name">Cardholder name</label>
    <input id="name" name="ccname" autocomplete="cc-name" required>

    <label for="ccnum">Card number</label>
    <input id="ccnum" name="ccnumber" inputmode="numeric"
           autocomplete="cc-number" required>

    <label for="exp">Expiry</label>
    <input id="exp" name="ccexp" autocomplete="cc-exp" required>

    <label for="cvv">CVV</label>
    <input id="cvv" name="cvc" inputmode="numeric"
           autocomplete="cc-csc" required>

    <button type="submit">Pay</button>
  </form>
</body>
</html>
```

**Expected output**: Browsers and password managers can autofill card details from saved payment methods.

**Why this output occurs**: Standard `cc-*` tokens map fields to payment autofill entries; `inputmode="numeric"` selects numeric keyboards on mobile.

---

### Real-World Cases

1. **Banking and payments** — strict autocomplete tokens, HTTPS, server-side validation.
2. **Healthcare intake** — accessible labels, error associations, progressive enhancement.
3. **Public-sector services** — WCAG 2.2 AA requirements, mobile-first layouts.
4. **Developer tools** — search inputs with `type="search"` and sensible `autofocus`.

---

### References

- W3C WAI — Forms Tutorial — https://www.w3.org/WAI/tutorials/forms/
- MDN Web Docs — Input attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes
- WHATWG — The input element — https://html.spec.whatwg.org/multipage/input.html
- MDN Web Docs — Constraint validation — https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- OWASP — Input Validation Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html

---

## Summary Tables

### Input Types at a Glance

| Type | Purpose | Wire format |
|---|---|---|
| `text` | Free text | any string |
| `password` | Obscured text | any string |
| `email` | Email address(es) | `local@domain` |
| `url` | Absolute URL | `https://…` |
| `tel` | Phone number | any (pattern optional) |
| `number` | Numeric value | float |
| `range` | Numeric slider | float |
| `date` | Calendar date | `YYYY-MM-DD` |
| `time` | Time of day | `HH:MM[:SS]` |
| `datetime-local` | Date + time (no TZ) | `YYYY-MM-DDTHH:MM` |
| `month` | Year + month | `YYYY-MM` |
| `week` | ISO week | `YYYY-Www` |
| `color` | Colour | `#rrggbb` |
| `file` | Files | multipart parts |
| `checkbox` | Multi-select bool | value if checked |
| `radio` | Single-select | value if checked |
| `hidden` | Invisible payload | string |
| `submit` | Submits form | label |
| `reset` | Resets form | — |
| `button` | No default action | — |
| `image` | Image submit | coordinates |
| `search` | Search box | string |

### Attribute Applicability Quick Reference

| Attribute | Applies to |
|---|---|
| `placeholder` | text, search, url, tel, email, password, number |
| `required` | text, search, url, tel, email, password, number, checkbox, radio, file, and date/time types |
| `readonly` | text-like and date/time types (not checkbox/radio/file/buttons) |
| `multiple` | email, file, select |
| `accept` | file |
| `min`/`max`/`step` | number, range, date, time, datetime-local, month, week |
| `pattern` | text, search, url, tel, email, password |
| `minlength`/`maxlength` | text-like |
| `size` | text-like |
| `list` | text, search, url, tel, email, number, range, colour, date/time types |
| `dirname` | text, search, textarea |
| `form*` overrides | submit, image |
| `hidden` | all types (invisible) |
| `disabled` | all (non-submitting) |

---

## Final Notes on Deprecated, Version-Specific, and Security-Sensitive Features

- **`<input type="image">`** is legacy; prefer a CSS-styled `<button type="submit">` with an `<img>` child.
- **`type="range"`** has limited precision and accessibility; consider a number input paired with ARIA if precision matters.
- **`datetime-local`, `week`** have weaker browser support in legacy browsers; consider progressive enhancement.
- **`accept`** on file inputs is a picker hint only; always validate MIME type, extension, size, and content server-side.
- **`readonly`** is not a security mechanism; users can edit via dev tools.
- **`hidden`** inputs are visible in page source; never store secrets.
- **Autofill `off`** is often ignored for passwords; use `current-password`/`new-password` tokens.
- **`pattern`** regex syntax is the `v` (or `u`) flag flavour, not PCRE; avoid exotic features.
- **Client-side validation** is UX only; always re-validate server-side.
- **`inputmode`** (e.g. `numeric`, `email`, `tel`, `decimal`) is an extra hint not covered above but highly recommended alongside `type`.

---

This cheat sheet has covered `<input>` in full: the element itself, all 22 standard input types, all 26 input attributes, and production-grade best practices — with annotated, executable examples and authoritative references throughout. Use it as a reference and a teaching aid, and validate markup against the WHATWG HTML Living Standard and WCAG guidance.