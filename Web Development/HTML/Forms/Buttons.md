# HTML Buttons — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
The `<button>` element is an interactive control that users activate to trigger an action — most commonly submitting a form, resetting a form, or running a script. It can contain rich phrasing content (text, icons, inline markup) and is one of the primary interaction points of any web page.

**Technical Definition**
`HTMLButtonElement` is a form-associated, labelable, submittable, resettable, listable, interactive, phrasing-content, flow-content, and palpable element whose content model is *phrasing content but with no interactive descendants* (the content model is often described as *phrasing content minus interactive content*). Its `type` IDL attribute reflects an enumerated content attribute with keywords `submit`, `reset`, and `button`; the *missing value default* and *invalid value default* are both `submit`. It implements `form`, `formAction`, `formEnctype`, `formMethod`, `formNoValidate`, `formTarget`, `name`, `value`, `disabled`, `labels`, `willValidate`, `validity`, `validationMessage`, `checkValidity()`, `reportValidity()`, `setCustomValidity()`, and `type`.

**Beginner-Friendly Explanation**
A button is the thing you click to make something happen — send a form, clear it, or run an action. `<button>` is the most flexible way to create one because you can put text, icons, and other inline markup inside it.

---

### Key Characteristics

- **Three action types**: `submit` (submits the owning form), `reset` (resets the owning form), and `button` (no default action; scripts handle it).
- **Rich content**: Unlike `<input type="submit">`, `<button>` can contain phrasing content such as `<strong>`, `<span>`, `<svg>`, or `<img>`.
- **No nested interactive content**: A `<button>` must not contain another button, link, or other interactive element; the content model explicitly excludes interactive descendants.
- **Default type is `submit`**: Inside a form, a bare `<button>` submits the form — a common source of accidental submissions.
- **Form-associated**: Can be associated with a form via `form="id"` even when placed outside it; supports per-button overrides of the form's submission settings.
- **Constraint validation**: When type is `submit`, activation triggers validation unless `formnovalidate` or the form's `novalidate` suppresses it.
- **Labelled by default**: Its text content is its accessible name; icon-only buttons must provide a hidden label.

---

### Prerequisites

1. **HTML form fundamentals** — `<form>`, `action`, `method`, `enctype`, submission lifecycle.
2. **Input elements** — including button-like `<input>` types.
3. **Labels** — for form controls and for accessible names in general.
4. **Basic JavaScript** — event listeners, `preventDefault()`, `submit`/`click` events.
5. **Basic CSS** — sizing, focus styles, hover/active states.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Forms & submission** | `submit`/`reset` types interact with the form submission and reset algorithms. |
| **JavaScript/DOM** | Click, focus, keyboard activation; `disabled`; `formmethod` etc. |
| **Accessibility** | Accessible name, role=button (for custom widgets), focus visibility, keyboard activation (Enter/Space). |
| **Progressive enhancement** | Native buttons work without JS; script-based buttons degrade without it. |
| **Design systems** | Buttons are the core reusable component; ARIA and semantics are central. |
| **Security/UX** | Double-submission prevention, confirmation for destructive actions. |

---

### Core Concepts / Features

1. **`<button>`** (the element)
2. **Button types** — `submit`, `reset`, `button`
3. **Button attributes** — `type`, `name`, `value`, `disabled`, `form`, `formaction`, `formenctype`, `formmethod`, `formnovalidate`, `formtarget`
4. **Button best practices**

Each concept follows the same uniform structure: Definitions → Purposes → Syntax Rules and Structure → Annotated Code Examples → Real-World Cases → References.

---

## 1. `<button>`

### Definitions

**Core Definition**
`<button>` is an interactive element that represents a button whose activation performs an action — submitting, resetting, or running a script.

**Technical Definition**
`HTMLButtonElement` has a content model of *phrasing content, but with no interactive content descendants and no tabindex attribute*. It is a form-associated, labelable, submittable, resettable element. Its accessible name is its text content (or its `aria-label`/`aria-labelledby` if present). Its default `type` is `submit`. The default rendering is platform-dependent but typically a small rectangular button.

**Beginner-Friendly Explanation**
A button is the clickable rectangle that says something like "Save" or "Cancel". It can have text or even a little icon inside. Its job depends on its `type` — send, clear, or do something custom.

---

### Purposes

- **To submit forms** via the `submit` type (its default).
- **To reset forms** via the `reset` type.
- **To run scripted actions** via the `button` type with an event listener.
- **To provide accessible, keyboard-activatable controls** with proper role and semantics.
- **To include rich content** (icons, spans) inside the clickable area, unlike `<input>` button types.

---

### Syntax Rules and Structure

**Complete general syntax**

```html
<button
  type="submit|reset|button"
  name="action"
  value="value-sent"
  form="form-id"
  disabled>
  Button label or inline content
</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<button` | Start tag. |
| `type` | `submit` (default), `reset`, or `button`. |
| `name` / `value` | If the button submits the form, `name=value` is included in the entry list. |
| `form` | Optional; associates with an external form by id. |
| `disabled` | Boolean; disables the button. |
| Content | Phrasing content only; no interactive descendants. |
| `</button>` | End tag; required. |

#### Syntax Rules

1. The content model is phrasing content *without* interactive descendants — do not put `<a>`, `<input>`, another `<button>`, or `contenteditable` inside.
2. The button must have an accessible name — its text content or an `aria-label`/`aria-labelledby`.
3. When a submit button is clicked, its `name`/`value` pair is included in the submission (if it has a `name`).
4. `disabled` prevents activation, focus, and submission; disabled buttons are not labelable targets for activation.
5. A `<button>` placed outside a form and without `form=` has no default action even with `type="submit"` (it has no form owner to submit).

#### Constraints and Limitations

- Nesting interactive elements inside a button is non-conforming and creates ambiguous activation.
- The browser reserves Space and Enter for activation; a script that intercepts these must preserve native behaviour.
- `disabled` buttons are removed from the accessibility tree for keyboard navigation — do not disable the submit button to prevent double-submission without a fallback UX (e.g. loading state with the button still focusable).
- Inside a `<form>`, a bare `<button>` defaults to `submit` — always specify `type="button"` for non-submitting actions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Submit Button

**Setup guide**: Save as `button-basic.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Basic Button</title>
</head>
<body>
  <form action="/save" method="post">
    <label for="title">Title</label>
    <input id="title" name="title" required>
    <!-- Default type is submit; button submits the form -->
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: A "Save" button that submits the form when clicked or activated with Enter/Space; an empty title triggers the browser's validation bubble.

**Why this output occurs**: With no `type` attribute, the button defaults to `submit`. Activation triggers the submission algorithm, which runs constraint validation first.

---

#### Example 2: A Rich-Content Button

**Setup guide**: Save as `button-rich.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Rich Button</title></head>
<body>
  <form action="/save" method="post">
    <label for="note">Note</label>
    <textarea id="note" name="note" rows="3"></textarea>
    <!-- <button> can contain phrasing content such as spans and svg icons -->
    <button type="submit">
      <svg width="16" height="16" aria-hidden="true" focusable="false">
        <circle cx="8" cy="8" r="7" fill="currentColor"></circle>
      </svg>
      Save note
    </button>
  </form>
</body>
</html>
```

**Expected output**: A button with a small circle icon and the text "Save note"; clicking submits the form; a screen reader announces "Save note, button".

**Why this output occurs**: `<button>` accepts phrasing content; the `<svg>` is decorative and hidden from AT via `aria-hidden="true"` and `focusable="false"`. The accessible name comes from the text content.

---

### Real-World Cases

1. **Form submission buttons** with icon + label combos.
2. **Card/modal actions** (Open, Close, Dismiss).
3. **Toolbar buttons** in editors (Bold, Italic) with icon content.

---

### References

- MDN Web Docs — `<button>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button
- WHATWG — The button element — https://html.spec.whatwg.org/multipage/form-elements.html#the-button-element
- MDN Web Docs — HTMLButtonElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement

---

## 2. Button Types

Each type is a distinct behaviour of `<button>`. The `type` attribute keyword selects between them.

---

### 2.1 `submit`

#### Definitions

**Core Definition**
A `submit` button submits its owning form when activated.

**Technical Definition**
The *submit button* state is the missing-value and invalid-value default of the `type` attribute. Its activation behaviour is the form submission algorithm, which fires the `submit` event (cancellable), runs interactive constraint validation unless `novalidate`/`formnovalidate` are present, and then navigates or fetches. The submitter's `name`/`value` becomes part of the entry list when it has a `name`.

**Beginner-Friendly Explanation**
The "Send" button — clicking it sends the form.

#### Purposes

- **To trigger form submission**.
- **To provide per-submit overrides** via `form*` attributes (e.g. Publish vs Save draft).

#### Syntax Rules and Structure

```html
<button type="submit" name="action" value="save">Save</button>
```

**Constraints**: If the button has no form owner (no ancestor `<form>` and no `form=`), activation does nothing. Without an explicit `type`, a `<button>` is a `submit` button — a frequent source of accidental submissions.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>submit</title></head>
<body>
  <form action="/submit" method="post">
    <label for="name">Name</label>
    <input id="name" name="name" required>
    <button type="submit">Send</button>
  </form>
</body>
</html>
```

**Expected output**: Clicking "Send" with an empty name triggers validation; with a name, submits the form.

**Why this output occurs**: `type="submit"` selects the submit behaviour; the browser enforces `required` before submitting.

#### Real-World Cases

- The primary action button on login, registration, checkout, and comment forms.

#### References

- MDN Web Docs — `<button type="submit">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#type
- WHATWG — Submit button state — https://html.spec.whatwg.org/multipage/form-elements.html#submit-button-state-(type=submit)

---

### 2.2 `reset`

#### Definitions

**Core Definition**
A `reset` button resets its owning form to its initial values when activated.

**Technical Definition**
The *reset button* state fires a cancellable `reset` event; if not cancelled, the form's reset algorithm sets every resettable element back to its default value. Reset buttons do not contribute data to the form's entry list.

**Beginner-Friendly Explanation**
The "Undo all my typing" button — restores the form to how it looked when the page loaded.

#### Purposes

- **To let users clear a form** in one action.

#### Syntax Rules and Structure

```html
<button type="reset">Clear</button>
```

**Constraints**: Reset restores default values (from `value` attributes, `selected` options, `defaultValue` of textareas). It does not "empty" fields that had default content — it restores them.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>reset</title></head>
<body>
  <form action="/save" method="post">
    <label for="name">Name</label>
    <input id="name" name="name" value="Alice">
    <button type="reset">Reset</button>
    <button type="submit">Save</button>
  </form>
</body>
</html>
```

**Expected output**: Typing "Bob" and clicking Reset restores "Alice"; Save submits the current value.

**Why this output occurs**: Reset uses the element's default value (`value` attribute) as the target.

#### Real-World Cases

- Rarely recommended in modern UX; often replaced by a Cancel link.

#### References

- MDN Web Docs — `<button type="reset">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#type

---

### 2.3 `button`

#### Definitions

**Core Definition**
A `button` type button has no default action; scripts decide what happens on activation.

**Technical Definition**
The *button* state does not submit or reset. Activation fires `click` and `click`-derived events only. It is the correct type for script-driven toggles, dialogs, and custom interactions inside forms.

**Beginner-Friendly Explanation**
A "do whatever my script says" button — nothing happens unless you add JavaScript.

#### Purposes

- **To provide safe, non-submitting actions inside a form** (e.g. "Preview", "Toggle password visibility").
- **To trigger scripts** without interfering with form submission.

#### Syntax Rules and Structure

```html
<button type="button">Preview</button>
```

**Constraints**: Without a script, activation is a no-op; do not use `type="button"` for a submit action by mistake.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>type=button</title></head>
<body>
  <form action="/save" method="post">
    <label for="pw">Password</label>
    <input id="pw" type="password" name="password">
    <!-- type="button" does not submit; the script toggles visibility -->
    <button type="button" id="toggle">Show password</button>
    <button type="submit">Save</button>
  </form>
  <script>
    const pw = document.getElementById('pw');
    const toggle = document.getElementById('toggle');
    toggle.addEventListener('click', () => {
      const showing = pw.type === 'text';
      pw.type = showing ? 'password' : 'text';
      toggle.textContent = showing ? 'Show password' : 'Hide password';
    });
  </script>
</body>
</html>
```

**Expected output**: Clicking "Show password" toggles the field between hidden and visible; it does not submit the form. "Save" submits.

**Why this output occurs**: `type="button"` prevents default form submission; the listener toggles the input type.

#### Real-World Cases

- Password visibility toggles, "Preview" actions, sidebar toggles, dialog openers.

#### References

- MDN Web Docs — `<button type="button">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#type

---

## 3. Button Attributes

Each attribute below follows the same uniform structure.

---

### 3.1 `type`

#### Definitions

**Core Definition**
`type` selects the button's default action: submit, reset, or none.

**Technical Definition**
Enumerated content attribute with keywords `submit`, `reset`, `button`. Missing and invalid values default to `submit`.

**Beginner-Friendly Explanation**
Tells the button what to do when clicked: send, clear, or nothing.

#### Purposes

- **To choose the button's default behaviour**.
- **To avoid accidental form submissions** (use `type="button"` for scripts).

#### Syntax Rules and Structure

```html
<button type="button">Do it</button>
```

**Constraints**: If you work inside a form and don't specify a type, the button will submit — the single most common button bug.

#### Annotated Example

```html
<button>Oops, this submits!</button>
<button type="button">Safe, this does nothing</button>
```

**Expected output**: The first button submits the surrounding form; the second does not.

**Why this output occurs**: The missing-value default of `type` is `submit`.

#### Real-World Cases

- Every `<button>` in a production app should set `type` explicitly.

#### References

- MDN Web Docs — `type` on `<button>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#type

---

### 3.2 `name`

#### Definitions

**Core Definition**
`name` names the button so its `value` can be submitted with the form.

**Technical Definition**
Content attribute reflected to `HTMLButtonElement.name`. When the button acts as a submitter and has both `name` and `value`, the pair is appended to the form's entry list.

**Beginner-Friendly Explanation**
Labels the button's submission contribution so the server can tell which button was pressed.

#### Purposes

- **To include the button's `value` in the submission** so the server knows which submit action was chosen.

#### Syntax Rules and Structure

```html
<button type="submit" name="action" value="delete">Delete</button>
```

**Constraints**: Only submit-type buttons contribute; without a `name`, nothing is submitted for the button.

#### Annotated Example

```html
<form action="/post" method="post">
  <textarea name="body"></textarea>
  <button type="submit" name="action" value="save">Save draft</button>
  <button type="submit" name="action" value="publish">Publish</button>
</form>
```

**Expected output**: Clicking "Save draft" submits `body=…&action=save`; clicking "Publish" submits `action=publish`.

**Why this output occurs**: The submitter's name/value pair is added to the entry list.

#### Real-World Cases

- Multi-button forms where the server dispatches on `action`.

#### References

- MDN Web Docs — `name` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#name

---

### 3.3 `value`

#### Definitions

**Core Definition**
`value` is the button's value, submitted alongside its `name` when the button submits the form.

**Technical Definition**
Content attribute reflected to `HTMLButtonElement.value`. Not displayed. Only included in the entry list when the button is the submitter and has a `name`.

**Beginner-Friendly Explanation**
The "answer" the button contributes when clicked.

#### Purposes

- **To distinguish submit buttons** on the server side.

#### Syntax Rules and Structure

```html
<button type="submit" name="action" value="publish">Publish</button>
```

**Constraints**: A `value` without a `name` submits nothing.

#### Annotated Example

```html
<button type="submit" name="action" value="like">👍</button>
<button type="submit" name="action" value="dislike">👎</button>
```

**Expected output**: Clicking 👍 submits `action=like`; the emoji is the visible label.

**Why this output occurs**: `value` is the submitted string; text content is the label.

#### Real-World Cases

- Upvote/downvote, multiple-choice submit buttons.

#### References

- MDN Web Docs — `value` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#value

---

### 3.4 `disabled`

#### Definitions

**Core Definition**
`disabled` disables the button, preventing interaction, focus, and submission.

**Technical Definition**
Boolean attribute; a disabled button is not focusable, not activatable, barred from constraint validation, and does not contribute its name/value to submission. `:disabled` styling applies.

**Beginner-Friendly Explanation**
Greys out the button and stops users from clicking it.

#### Purposes

- **To reflect an unavailable action**.
- **To prevent double submission** (with care).

#### Syntax Rules and Structure

```html
<button type="submit" disabled>Saving…</button>
```

**Constraints**: Disabled buttons are removed from tab order — do not disable while focus is inside them without moving focus. Use `aria-disabled="true"` + script guard for "disabled but focusable" patterns where required.

#### Annotated Example

```html
<form action="/save" method="post" onsubmit="this.querySelector('button[type=submit]').disabled = true;">
  <input name="title" required>
  <button type="submit">Save</button>
</form>
```

**Expected output**: On submit, the button becomes disabled, preventing a second click while the request is in flight.

**Why this output occurs**: The inline `onsubmit` disables the button the moment submission starts.

#### Real-World Cases

- Preventing double submissions, reflecting permission states.

#### References

- MDN Web Docs — `disabled` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/disabled

---

### 3.5 `form`

#### Definitions

**Core Definition**
`form` associates the button with a form by id, even if the button is not a descendant of that form.

**Technical Definition**
Content attribute; when present, its value is used as the button's form owner. The `form` attribute is only honoured on form-associated elements (including `button`).

**Beginner-Friendly Explanation**
Links a button to a form even if the button isn't inside it.

#### Purposes

- **To place a button outside the form's markup** but still submit it.

#### Syntax Rules and Structure

```html
<button type="submit" form="myform">Save</button>
<form id="myform" action="/save" method="post">…</form>
```

**Constraints**: The referenced id must exist; otherwise the button has no owner.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>form attr</title></head>
<body>
  <!-- Button sits above the form; form= associates it -->
  <button type="submit" form="editor">Save</button>

  <form id="editor" action="/save" method="post">
    <textarea name="body"></textarea>
  </form>
</body>
</html>
```

**Expected output**: Clicking "Save" above submits the form below.

**Why this output occurs**: `form="editor"` gives the button that form owner.

#### Real-World Cases

- Sticky toolbars above long forms, "Save" buttons in headers.

#### References

- MDN Web Docs — `form` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#form

---

### 3.6 `formaction`

#### Definitions

**Core Definition**
`formaction` overrides the form's `action` for this specific submit.

**Technical Definition**
Applicable only to submit-capable controls. Content attribute; URL value resolved against the base URL.

**Beginner-Friendly Explanation**
Lets one button send to a different URL.

#### Purposes

- **To offer alternative submission endpoints** from a single form.

#### Syntax Rules and Structure

```html
<button type="submit" formaction="/drafts" formnovalidate>Save draft</button>
<button type="submit" formaction="/publish">Publish</button>
```

**Constraints**: Applies only to submit buttons.

#### Annotated Example

```html
<form action="/default" method="post">
  <input name="title" required>
  <button type="submit" formaction="/save">Save</button>
  <button type="submit" formaction="/preview">Preview</button>
</form>
```

**Expected output**: "Save" posts to `/save`; "Preview" posts to `/preview`.

**Why this output occurs**: The submitter's `formaction` overrides the form's action.

#### Real-World Cases

- Save vs Publish vs Delete buttons.

#### References

- MDN Web Docs — `formaction` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#formaction

---

### 3.7 `formenctype`

#### Definitions

**Core Definition**
`formenctype` overrides `enctype` for this specific submit.

**Technical Definition**
Applies only to submit-capable controls. Enumerated like `enctype`.

**Beginner-Friendly Explanation**
Lets one button use a different form encoding.

#### Purposes

- **To support a file-including vs file-excluding submit path**.

#### Syntax Rules and Structure

```html
<button type="submit" formenctype="multipart/form-data">Upload</button>
```

**Constraints**: Applies only to submit buttons.

#### Annotated Example

```html
<form action="/x" method="post" enctype="application/x-www-form-urlencoded">
  <input type="file" name="f">
  <button type="submit" formenctype="multipart/form-data">Upload</button>
  <button type="submit">Text only</button>
</form>
```

**Expected output**: "Upload" sends a multipart body; "Text only" sends urlencoded (file lost).

**Why this output occurs**: The submitter's `formenctype` overrides the form's enctype.

#### Real-World Cases

- Hybrid forms with optional file attachment.

#### References

- MDN Web Docs — `formenctype` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#formenctype

---

### 3.8 `formmethod`

#### Definitions

**Core Definition**
`formmethod` overrides `method` for this specific submit.

**Technical Definition**
Applies only to submit-capable controls. Values `get`, `post`, `dialog`.

**Beginner-Friendly Explanation**
Lets one button use GET while another uses POST.

#### Purposes

- **To mix idempotent preview with state-changing submit**.

#### Syntax Rules and Structure

```html
<button type="submit" formmethod="get" formaction="/preview">Preview</button>
```

**Constraints**: Only on submit buttons.

#### Annotated Example

```html
<form action="/editor" method="post">
  <textarea name="body">Hello</textarea>
  <button type="submit" formmethod="get" formaction="/preview">Preview</button>
  <button type="submit" formaction="/publish">Publish</button>
</form>
```

**Expected output**: "Preview" navigates with query string; "Publish" posts.

**Why this output occurs**: `formmethod` overrides `form.method` for that submission.

#### Real-World Cases

- Editor previews, read-only vs mutating operations.

#### References

- MDN Web Docs — `formmethod` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#formmethod

---

### 3.9 `formnovalidate`

#### Definitions

**Core Definition**
`formnovalidate` skips constraint validation for this specific submit.

**Technical Definition**
Boolean attribute applicable to submit controls; equivalent to `novalidate` on the form for this submission only.

**Beginner-Friendly Explanation**
Skips the browser's checks for this button only.

#### Purposes

- **To allow Save-draft buttons to bypass required fields**.

#### Syntax Rules and Structure

```html
<button type="submit" formnovalidate>Save draft</button>
```

**Constraints**: Only on submit buttons.

#### Annotated Example

```html
<form action="/post" method="post">
  <input name="title" required>
  <button type="submit">Publish</button>
  <button type="submit" formnovalidate formaction="/drafts">Save draft</button>
</form>
```

**Expected output**: "Publish" enforces `required`; "Save draft" submits even when empty.

**Why this output occurs**: The submitter's `formnovalidate` suppresses validation.

#### Real-World Cases

- Draft saving, "skip for now" workflows.

#### References

- MDN Web Docs — `formnovalidate` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#formnovalidate

---

### 3.10 `formtarget`

#### Definitions

**Core Definition**
`formtarget` overrides the form's `target` for this specific submit.

**Technical Definition**
Applicable only to submit controls; value is a browsing-context name (`_self`, `_blank`, `_parent`, `_top`, or a name).

**Beginner-Friendly Explanation**
Opens this button's result in a different tab or frame.

#### Purposes

- **To choose the response's browsing context per button**.

#### Syntax Rules and Structure

```html
<button type="submit" formtarget="_blank">Open in new tab</button>
```

**Constraints**: Only on submit buttons.

#### Annotated Example

```html
<form action="/report" method="get">
  <input name="q">
  <button type="submit">Search</button>
  <button type="submit" formtarget="_blank">Search in new tab</button>
</form>
```

**Expected output**: The first button navigates in place; the second opens a new tab.

**Why this output occurs**: `formtarget` overrides `form.target` for that submission.

#### Real-World Cases

- "Open report in new tab" secondary actions.

#### References

- MDN Web Docs — `formtarget` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button#formtarget

---

## 4. Button Best Practices

### Definitions

**Core Definition**
Button best practices are guidelines that produce buttons that are accessible, usable, and safe in production, going beyond markup conformance.

**Technical Definition**
They combine HTML conformance with WCAG success criteria (2.1.1 Keyboard, 2.4.7 Focus Visible, 2.5.5 Target Size, 3.3.2 Labels or Instructions, 4.1.2 Name/Role/Value), usability principles (clear labels, sensible placement, loading states), and security/robustness guidance (server-side validation, double-submission prevention, safe defaults).

**Beginner-Friendly Explanation**
The habits that make buttons obvious, easy to click, easy to use with a keyboard, and hard to misuse in code.

---

### Purposes

- **To ensure accessibility** for keyboard, screen-reader, and switch-control users.
- **To make the primary action obvious** and reduce user errors.
- **To prevent accidental or duplicate submissions**.
- **To keep behaviour predictable** across browsers, frameworks, and user agents.

---

### Syntax Rules and Structure

Best practices are principles; the table summarises the core rules.

| Practice | Rule |
|---|---|
| **Always specify `type`** | `submit`, `reset`, or `button` — never rely on the default inside a form. |
| **Use `<button>` over `<input>`** | Richer content, easier styling; `<a>` should not be used for actions. |
| **Use `<a>` for navigation, `<button>` for actions** | Semantics matter for assistive tech and users. |
| **Give an accessible name** | Text content or `aria-label` for icon-only buttons; never leave blank. |
| **Visible focus styles** | Never `outline: none` without an equivalent replacement. |
| **Prevent double submission** | Disable on submit, show a loading state, or use idempotency keys server-side. |
| **Set minimum target size** | WCAG 2.5.5 recommends 44×44 CSS px minimum for touch. |
| **Keep disabled discoverable** | If disabled, provide context on why; consider `aria-disabled` + script guards for long-running actions. |
| **Confirm destructive actions** | Use a confirmation dialog or undo pattern, not silent execution. |
| **Don't nest interactive content** | No links, buttons, or inputs inside `<button>`. |
| **Decorative icons** | `aria-hidden="true"` and `focusable="false"` on SVG; provide text label. |
| **Loading states** | Replace label with "Saving…" and `aria-busy="true"` on the button. |
| **Don't use `role="button"` unless necessary** | Native `<button>` has built-in semantics and keyboard behaviour. |

#### Syntax Rules

1. Every button has an accessible name.
2. Inside a form, set `type` explicitly.
3. Focus indicator must be visible (WCAG 2.4.7).
4. Never disable the *only* submit button before submission without a fallback (e.g. show an alternate action).
5. Use `aria-disabled="true"` when the button should remain focusable (e.g. so screen readers can still announce it).

#### Constraints and Limitations

- `disabled` removes the button from the accessibility tree and tab order — not always the right UX.
- Native buttons inside a `<form>` default to submit — this is a frequent production bug.
- Buttons styled to look like links (or vice versa) confuse users; keep visual affordance consistent with behaviour.
- Custom button-like `<div>`s require tabindex, role, keyboard handlers, and ARIA — prefer `<button>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible, Loading-Aware Submit

**Setup guide**: Save as `button-best.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Button Best Practices</title>
  <style>
    button {
      min-height: 44px;
      min-width: 44px;
      padding: .5rem 1rem;
      font: inherit;
      cursor: pointer;
      border: 1px solid #0077cc;
      border-radius: 4px;
      background: #0077cc;
      color: #fff;
    }
    button:focus-visible {
      outline: 3px solid #ffcc00;
      outline-offset: 2px;
    }
    button[aria-disabled="true"] { opacity: .6; cursor: progress; }
  </style>
</head>
<body>
  <form id="f" action="/subscribe" method="post">
    <label for="email">Email</label>
    <input id="email" name="email" type="email" autocomplete="email" required>

    <!-- Submit button; explicit type; loading state uses aria-disabled + aria-busy -->
    <button type="submit" id="go">Subscribe</button>
    <p id="status" role="status" aria-live="polite"></p>
  </form>

  <script>
    const form = document.getElementById('f');
    const go = document.getElementById('go');
    const status = document.getElementById('status');

    form.addEventListener('submit', async (e) => {
      if (!form.checkValidity()) return; // native validation handles UI
      e.preventDefault();
      // Lock button but keep it focusable so screen readers announce state
      go.setAttribute('aria-disabled', 'true');
      go.setAttribute('aria-busy', 'true');
      go.textContent = 'Subscribing…';
      try {
        const res = await fetch(form.action, {
          method: 'POST',
          body: new FormData(form),
          credentials: 'same-origin'
        });
        if (!res.ok) throw new Error('HTTP ' + res.status);
        status.textContent = 'Subscribed!';
        form.reset();
      } catch (err) {
        status.textContent = 'Failed. Please try again.';
      } finally {
        go.removeAttribute('aria-disabled');
        go.removeAttribute('aria-busy');
        go.textContent = 'Subscribe';
      }
    });
  </script>
</body>
</html>
```

**Expected output**: Clicking "Subscribe" shows "Subscribing…" while the fetch is in flight; on success, "Subscribed!" appears and fields reset. Focus styles are visible; the button remains focusable during loading.

**Why this output occurs**: `aria-disabled` + `aria-busy` communicate the state without removing the button from tab order; disabling via attribute (rather than `disabled` property) keeps it announced. The `finally` resets state.

---

#### Example 2: Multi-Action Form with `formaction` and `formnovalidate`

**Setup guide**: Save as `button-multi.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Multi-Action Buttons</title></head>
<body>
  <form action="/default" method="post">
    <label for="title">Title</label>
    <input id="title" name="title" required>
    <label for="body">Body</label>
    <textarea id="body" name="body" rows="4"></textarea>

    <!-- Publish enforces validation; draft skips it -->
    <button type="submit" formaction="/publish">Publish</button>
    <button type="submit" formaction="/drafts" formnovalidate>Save draft</button>
  </form>
</body>
</html>
```

**Expected output**: With an empty title, "Publish" triggers validation; "Save draft" submits anyway.

**Why this output occurs**: `formnovalidate` on the draft button suppresses validation for that submission; `formaction` overrides the endpoint.

---

#### Example 3: Icon-Only Button with Accessible Name

**Setup guide**: Save as `button-icon.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Icon Button</title>
  <style>
    .close {
      display: inline-flex;
      align-items: center; justify-content: center;
      width: 44px; height: 44px;
      border: 1px solid #ccc; background: #fff; border-radius: 8px;
      cursor: pointer;
    }
    .visually-hidden {
      position: absolute; width: 1px; height: 1px;
      overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap;
    }
  </style>
</head>
<body>
  <button type="button" class="close" aria-label="Close dialog">
    <svg aria-hidden="true" focusable="false" width="16" height="16" viewBox="0 0 16 16">
      <path d="M2 2 L14 14 M14 2 L2 14" stroke="currentColor" stroke-width="2"/>
    </svg>
  </button>
</body>
</html>
```

**Expected output**: A 44×44 close button with an X icon; screen readers announce "Close dialog, button".

**Why this output occurs**: `aria-label` provides the accessible name; `aria-hidden` hides the decorative SVG; the sizing meets WCAG 2.5.5's touch-target guidance.

---

### Real-World Cases

1. **Checkout flows**: primary submit button with loading state; secondary "Continue shopping" as a link (not a button, because it navigates).
2. **Admin dashboards**: destructive "Delete" button styled in red with a confirmation dialog; `type="button"` to avoid form submission unless intended.
3. **Editor toolbars**: `type="button"` on format toggles so they never submit the editor's form.
4. **Cookie banners**: `<button>` for Accept/Reject, `<a>` for the Cookie Policy page.

---

### References

- W3C — Web Content Accessibility Guidelines (WCAG) 2.2 — https://www.w3.org/TR/WCAG22/
- W3C WAI — Forms Tutorial — https://www.w3.org/WAI/tutorials/forms/
- MDN Web Docs — `<button>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button
- MDN Web Docs — HTMLButtonElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement
- WHATWG — The button element — https://html.spec.whatwg.org/multipage/form-elements.html#the-button-element
- WHATWG — Form submission algorithm — https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#form-submission-algorithm

---

## Summary Tables

### Button Types

| Type | Default | Action | Submits value? |
|---|---|---|---|
| `submit` | ✔ (default) | Submits the owning form | Yes (name+value) |
| `reset` | — | Resets the form to default values | No |
| `button` | — | None (scripts handle it) | No |

### Button Attributes at a Glance

| Attribute | Applies to | Effect |
|---|---|---|
| `type` | all | `submit` / `reset` / `button`; default `submit` |
| `name` | all | Submission key (with `value`) when acting as submitter |
| `value` | all | Submitted with `name` on submit |
| `disabled` | all | Blocks activation, focus, submission |
| `form` | all | Associates with an external form by id |
| `formaction` | submit | Overrides form `action` |
| `formenctype` | submit | Overrides form `enctype` |
| `formmethod` | submit | Overrides form `method` |
| `formnovalidate` | submit | Skips validation for this submission |
| `formtarget` | submit | Overrides form `target` |

### `<button>` vs `<input type="…">` vs `<a>`

| Aspect | `<button>` | `<input type="submit">` | `<a href>` |
|---|---|---|---|
| Rich content | Yes (phrasing) | No (label only) | Yes |
| Form submission | Yes (type submit) | Yes | No (navigates) |
| Semantic role | button | button | link |
| Correct for navigation | No | No | Yes |
| Correct for actions | Yes | Yes | No |

### Common Pitfalls

| Pitfall | Consequence | Fix |
|---|---|---|
| Missing `type` inside a form | Accidental submit | Always set `type` explicitly |
| Using `<a>` for actions | Wrong semantics; breaks keyboard/AT | Use `<button>` |
| Icon-only with no label | Unnamed button | `aria-label` or visually hidden text |
| Removing focus outline | Accessibility failure | Provide `:focus-visible` styles |
| `disabled` on submit during load | Lost focus/announcement | `aria-disabled` + `aria-busy` |
| Nesting `<a>`/`<input>` in `<button>` | Non-conforming; ambiguous activation | Move interactive content out |
| Duplicate submissions | Server double-processing | Disable, loading state, idempotency keys |
| Using `role="button"` on `<div>` | Missing keyboard/Enter/Space support | Use native `<button>` |

---

## Final Notes on Deprecated, Version-Specific, and Security-Sensitive Features

- **Bare `<button>` inside a `<form>` defaults to `type="submit"`** — this is the single most common button bug; always set `type`.
- **`<button>` content model forbids interactive descendants** — nesting an `<a>` or `<input>` inside a `<button>` is non-conforming and produces ambiguous activation; move them out.
- **`disabled` removes the button from tab order and the accessibility tree** — do not use it to lock a submit during async work if you want screen-reader users to know; use `aria-disabled` + `aria-busy` instead.
- **`form*` override attributes** (`formaction`, `formenctype`, `formmethod`, `formnovalidate`, `formtarget`) apply only to submit-capable buttons; `type="button"` and `type="reset"` ignore them.
- **`<input type="image">`** is a legacy image submit; prefer a `<button type="submit">` with an `<img>` or `<svg>` child.
- **Native `<button>` gives you Enter/Space activation, focus, and role=button for free** — do not replace it with a `<div>` unless absolutely necessary, and then you must implement all behaviours (tabindex, keydown for Space/Enter, `role="button"`, and focus styles).
- **Client-side double-submission prevention is a UX courtesy**; servers must still deduplicate via idempotency keys or one-time tokens.
- **`aria-pressed`, `aria-expanded`, `aria-controls`** are the right ARIA for toggle and disclosure buttons — but use them only where the native button alone does not communicate state.

---

This cheat sheet has covered `<button>` in full: the element, its three types (`submit`, `reset`, `button`), all ten attributes (`type`, `name`, `value`, `disabled`, `form`, `formaction`, `formenctype`, `formmethod`, `formnovalidate`, `formtarget`), and production-grade best practices — with annotated, executable examples and authoritative references throughout. Use it as a reference and a teaching aid, and validate markup against the WHATWG HTML Living Standard and WCAG guidance.