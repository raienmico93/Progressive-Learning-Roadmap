# HTML Textarea — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
The `<textarea>` element is a multi-line plain-text form control that lets users enter substantial amounts of free-form text — messages, comments, descriptions, code snippets, and other multi-line content.

**Technical Definition**
`HTMLTextAreaElement` is a form-associated, labelable, submittable, resettable, interactive, phrasing-content, palpable element. Its content model is *text*, and the element is an *escapable raw text element*: character references are decoded but nested tags are not parsed as markup. Its default value is its child text content; its live value is the IDL `value` property. It participates in constraint validation via `required`, `minlength`, `maxlength`, and custom validity, and exposes `value`, `defaultValue`, `textLength`, `selectionStart`, `selectionEnd`, `selectionDirection`, `setSelectionRange()`, `select()`, and `setRangeText()`.

**Beginner-Friendly Explanation**
A textarea is a big text box where users can type multiple lines — like the message field in a contact form or the comment box under a blog post. Unlike a single-line `<input>`, it wraps long text across lines and supports Enter for new paragraphs.

---

### Key Characteristics

- **Multi-line text**: Displays and submits text that spans lines (newlines preserved as `\r\n` per HTML forms spec, though modern servers usually receive `\r\n`).
- **Escapable raw text element**: Tags inside `<textarea>` are shown as text, not parsed as HTML; `<` and `>` must be escaped when part of the intended value.
- **Default from content**: The text between the start and end tags is the element’s default value; programmatic changes set the live `value`.
- **Resizable**: Users can typically drag a resize handle (browser-dependent); CSS `resize` controls this.
- **Constraint-validated**: `required`, `minlength`, `maxlength`, `pattern` (via custom validation only), and custom validity all apply.
- **Form-associated and labelable**: Contributes its text under `name` on submission; must be labelled for accessibility.

---

### Prerequisites

1. **HTML form fundamentals** — `<form>`, `action`, `method`, submission.
2. **Input elements** — single-line inputs for contrast.
3. **Labels** — `<label for>` associations.
4. **Basic CSS** — sizing, `resize`, `min-height`, `font-family: monospace` for code areas.
5. **Basic JavaScript** — reading/writing `value`, selection APIs.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Forms & submission** | The textarea value is submitted under its `name` in the form entry list. |
| **Constraint validation** | `required`, `minlength`, `maxlength` and custom validity. |
| **Accessibility** | Labels, `aria-describedby`, character counters, error messaging. |
| **JavaScript/DOM** | `value`, `textLength`, `selectionStart`, `setSelectionRange()`, `setRangeText()`. |
| **Rich text** | A textarea is plain text; rich text requires `contenteditable` or an editor library. |
| **Security** | User-submitted text must be sanitised and escaped when rendered (XSS risk). |

---

### Core Concepts / Features

1. **`<textarea>`** (the element)
2. **Textarea attributes** — `name`, `rows`, `cols`, `placeholder`, `required`, `disabled`, `readonly`, `maxlength`, `minlength`, `wrap`
3. **Textarea best practices**

Each concept follows the same uniform structure: Definitions → Purposes → Syntax Rules and Structure → Annotated Code Examples → Real-World Cases → References.

---

## 1. `<textarea>`

### Definitions

**Core Definition**
`<textarea>` is the multi-line text input element of HTML. It displays a resizable text box where users type free-form text, and it submits that text as a single string.

**Technical Definition**
`HTMLTextAreaElement` has a content model of *text* and is classified as an *escapable raw text element*. Its default value is the concatenation of its child text nodes (a single leading newline immediately after the start tag is dropped). Its live value is managed by the *dirty value flag*, which becomes set once the user or a script modifies the value; after that, changes to the child text do not affect `value`. Its `wrap` attribute controls how line breaks are serialised on submission.

**Beginner-Friendly Explanation**
A textarea is the big multi-line text box. Type as much as you like; press Enter to start a new line. When the form is submitted, the whole thing (with newlines) goes as a single value.

---

### Purposes

- **To collect substantial free-form text** — messages, comments, feedback, notes.
- **To preserve user formatting** — paragraphs, lists (typed manually), code samples.
- **To support multi-line editing** with wrapping, scrolling, and resizing.
- **To provide a plain-text editor surface** for code, JSON, or structured text when appropriate.

---

### Syntax Rules and Structure

**Complete general syntax**

```html
<label for="message">Message</label>
<textarea
  id="message"
  name="message"
  rows="6"
  cols="40"
  placeholder="Type your message…"
  maxlength="1000"
  required>
  Initial text content (optional default value).
</textarea>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<textarea` | Start tag. |
| `name="…"` | Submission key. |
| `rows` / `cols` | Initial dimensions in text rows/columns. |
| `placeholder` | Hint shown when empty. |
| `maxlength` / `minlength` | Length constraints in UTF-16 code units. |
| `required` | Enforces non-empty value on submission. |
| Text content | Default value; a leading newline immediately after `<textarea>` is ignored. |
| `</textarea>` | End tag; required. |

#### Syntax Rules

1. `<textarea>` requires a **closing tag**; it is not a void element.
2. Its content must be **text only** — no child elements; `<` and `&` in the intended value must be escaped (`&lt;`, `&amp;`).
3. A **single newline** immediately after the start tag is discarded (an artefact for readability of the markup).
4. Whitespace inside the textarea is preserved literally — do not indent the default value unless you want indentation in the value.
5. Programmatically setting `value` sets the live value but **does not** change the `defaultValue` unless you also set the child text.
6. The `wrap` attribute controls line-break insertion on submission.

#### Constraints and Limitations

- Cannot contain HTML children; there is no rich text formatting.
- Default `rows` and `cols` are 2 and 20 respectively; visual size depends on CSS, font, and browser.
- The resize handle is controlled by CSS `resize`; some developers disable it, harming usability — see best practices.
- Newlines are normalised to `\r\n` in submission on most platforms; servers should accept both.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Labelled Textarea

**Setup guide**: Save as `textarea-basic.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Basic Textarea</title>
</head>
<body>
  <form action="/contact" method="post">
    <label for="message">Your message</label>
    <!-- Default empty; user types into it -->
    <textarea id="message" name="message" rows="5" cols="40"
              placeholder="Type your message…" required></textarea>
    <button>Send</button>
  </form>
</body>
</html>
```

**Expected output**: A labelled 5-row text box with a placeholder. Submitting empty triggers the browser's "please fill out this field" bubble. Typing text and submitting sends `message=...` with newlines encoded as `%0D%0A`.

**Why this output occurs**: The `<textarea>` is a form control; its value is submitted under `name="message"`. `required` adds a `valueMissing` constraint. Newlines are percent-encoded in urlencoded submissions.

---

#### Example 2: Default Value from Content

**Setup guide**: Save as `textarea-default.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Textarea Default</title>
</head>
<body>
  <form action="/save" method="post">
    <label for="notes">Notes</label>
    <!-- The content between tags is the initial value -->
    <textarea id="notes" name="notes" rows="4" cols="50">Line one.
Line two.
Line three.</textarea>
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: The textarea shows three lines of pre-filled text exactly as typed.

**Why this output occurs**: The content between the start and end tags becomes the default value. Whitespace and newlines are preserved (no leading newline directly after the start tag, so line one is not padded).

---

#### Example 3: Escaping Special Characters in Default Value

**Setup guide**: Save as `textarea-escape.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Escaping</title></head>
<body>
  <form action="/save" method="post">
    <label for="code">Code snippet</label>
    <!-- '<' must be escaped as &lt; otherwise it can look like a tag -->
    <textarea id="code" name="code" rows="4" cols="50">&lt;div&gt;Hello&lt;/div&gt;
&lt;script&gt;alert('x')&lt;/script&gt;</textarea>
    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: The textarea displays `<div>Hello</div>` and `<script>alert('x')</script>` as literal text.

**Why this output occurs**: `<textarea>` is an escapable raw text element: character references are decoded, but `<` is not treated as the start of a tag. Failing to escape `<` can cause the parser to treat the content as a stray end tag.

---

### Real-World Cases

1. **Contact forms**: the message field is a textarea with a length limit.
2. **Blog comments**: multi-line textarea with character counting.
3. **Support tickets**: description field, often with markdown hint text.
4. **Code snippets**: monospaced textarea in developer tools.
5. **Admin notes**: internal notes on records.

---

### References

- MDN Web Docs — `<textarea>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea
- WHATWG — The textarea element — https://html.spec.whatwg.org/multipage/form-elements.html#the-textarea-element
- MDN Web Docs — HTMLTextAreaElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLTextAreaElement

---

## 2. Textarea Attributes

Each attribute below follows the same uniform structure.

---

### 2.1 `name`

#### Definitions

**Core Definition**
`name` is the key under which the textarea's value is submitted.

**Technical Definition**
Content attribute reflected to `HTMLTextAreaElement.name`. A textarea without a `name` is not a *successful control* and never contributes to the form's entry list.

**Beginner-Friendly Explanation**
The label the server receives alongside the typed text.

#### Purposes

- **To identify the submitted value** on the server.
- **To group related textareas** in multi-step or repeatable forms.

#### Syntax Rules and Structure

```html
<textarea name="message">…</textarea>
```

**Constraints and Limitations**: Unnamed textareas submit nothing. Duplicate names across textareas are valid but produce repeated entries (useful for arrays).

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>name</title></head>
<body>
  <form action="/send" method="post">
    <label for="msg">Message</label>
    <textarea id="msg" name="message" rows="3" cols="40">Hello!</textarea>
    <button>Send</button>
  </form>
</body>
</html>
```

**Expected output**: Submitting sends `message=Hello%21` (with newlines encoded if present).

**Why this output occurs**: The `name` attribute makes the control successful and provides the submission key.

#### Real-World Cases

- Every production textarea in a form.

#### References

- MDN Web Docs — `name` on `<textarea>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea#name

---

### 2.2 `rows`

#### Definitions

**Core Definition**
`rows` sets the initial visible height of the textarea in text lines.

**Technical Definition**
Positive integer content attribute; defaults to 2. It is a presentational hint — it determines the element's *intrinsic height* measured in lines of the current font. CSS `height` overrides it; users can typically drag to resize.

**Beginner-Friendly Explanation**
How many lines tall the textarea starts out.

#### Purposes

- **To set a sensible initial height** matched to expected input size.
- **To reduce the need for CSS sizing** in simple pages.

#### Syntax Rules and Structure

```html
<textarea rows="6">…</textarea>
```

**Constraints and Limitations**: `rows` is a hint; CSS `height` / `min-height` / `max-height` govern the rendered size. On mobile, resizing behaves differently.

#### Annotated Example

```html
<label for="feedback">Feedback</label>
<textarea id="feedback" name="feedback" rows="8" cols="50"></textarea>
```

**Expected output**: A textarea roughly 8 lines tall (depending on font and UA).

**Why this output occurs**: The `rows` attribute seeds the intrinsic size; CSS can override.

#### Real-World Cases

- Contact forms (4–6 rows), code snippets (10–20 rows).

#### References

- MDN Web Docs — `rows` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea#rows

---

### 2.3 `cols`

#### Definitions

**Core Definition**
`cols` sets the initial visible width of the textarea in average character widths.

**Technical Definition**
Positive integer content attribute; defaults to 20. Presentational hint measured in *average character widths* (like CSS `ch` units approximately). CSS `width` overrides it.

**Beginner-Friendly Explanation**
How wide the textarea starts out, roughly in characters.

#### Purposes

- **To set an initial width** before CSS is applied.
- **To make textareas usable** even without a stylesheet.

#### Syntax Rules and Structure

```html
<textarea rows="4" cols="60">…</textarea>
```

**Constraints and Limitations**: `cols` is a hint; CSS width wins. On narrow viewports, a fixed `cols` may overflow — use responsive CSS (`width: 100%`).

#### Annotated Example

```html
<textarea rows="4" cols="80" name="msg"></textarea>
```

**Expected output**: A textarea about 80 characters wide by 4 rows tall on desktop.

**Why this output occurs**: `cols` seeds intrinsic width; large values overflow small screens without CSS.

#### Real-World Cases

- Responsive forms typically override with `width: 100%` and rely less on `cols`.

#### References

- MDN Web Docs — `cols` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea#cols

---

### 2.4 `placeholder`

#### Definitions

**Core Definition**
`placeholder` is a hint shown inside the textarea when it is empty.

**Technical Definition**
Content attribute; not a substitute for a `<label>`; not announced reliably on all ATs; and it disappears once the user types.

**Beginner-Friendly Explanation**
Faint grey example text that disappears when you type.

#### Purposes

- **To give format or content hints** without cluttering the UI.

#### Syntax Rules and Structure

```html
<textarea placeholder="Describe the issue in as much detail as possible…"></textarea>
```

**Constraints and Limitations**: Do not use as the sole label; contrast of placeholder text must meet WCAG (many browsers default to low-contrast grey).

#### Annotated Example

```html
<label for="issue">Issue</label>
<textarea id="issue" name="issue" rows="4"
          placeholder="Steps to reproduce, expected vs actual…"></textarea>
```

**Expected output**: Hint visible when empty; disappears on typing; label remains visible above.

**Why this output occurs**: `placeholder` renders only when the value is empty.

#### Real-World Cases

- Formatting hints for support tickets and code inputs.

#### References

- MDN Web Docs — `placeholder` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/placeholder

---

### 2.5 `required`

#### Definitions

**Core Definition**
`required` marks the textarea as mandatory.

**Technical Definition**
Boolean constraint attribute; adds the `valueMissing` validity state when the value is the empty string. Note: **whitespace-only input counts as non-empty** and passes `required`; trim and validate server-side.

**Beginner-Friendly Explanation**
The box must not be empty to submit.

#### Purposes

- **To enforce that some text is provided**.

#### Syntax Rules and Structure

```html
<textarea name="comment" required></textarea>
```

**Constraints and Limitations**: Whitespace-only values pass. Combine with server-side trimming and, if strict, a `pattern` via `setCustomValidity()`.

#### Annotated Example

```html
<form action="/post" method="post">
  <label for="body">Post body</label>
  <textarea id="body" name="body" rows="6" required></textarea>
  <button>Publish</button>
</form>
```

**Expected output**: Empty submission triggers `valueMissing`; submitting spaces passes (and should be handled server-side).

**Why this output occurs**: `required` only checks the empty string.

#### Real-World Cases

- Required comments, descriptions, and message bodies.

#### References

- MDN Web Docs — `required` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/required

---

### 2.6 `disabled`

#### Definitions

**Core Definition**
`disabled` disables the textarea entirely: no interaction, no focus, no submission.

**Technical Definition**
Boolean attribute; a disabled textarea is barred from interaction, is not focusable, is barred from constraint validation, and is not a successful control. Its value is not included in the form entry list.

**Beginner-Friendly Explanation**
Greys out the box and stops anything from being typed or sent.

#### Purposes

- **To reflect unavailable state** while preserving visual context.

#### Syntax Rules and Structure

```html
<textarea name="notes" disabled>Read-only preview.</textarea>
```

**Constraints and Limitations**: Disabled values do not submit; if you must submit, use `readonly` (for the value) or a hidden input.

#### Annotated Example

```html
<textarea name="draft" disabled>Locked while saving…</textarea>
<input type="hidden" name="draft" value="Locked while saving…">
```

**Expected output**: The box is greyed out and unfocusable, but the hidden input still sends the value.

**Why this output occurs**: Disabled controls are not successful; the hidden input carries the value.

#### Real-World Cases

- Locked content during processing, or read-only archive views.

#### References

- MDN Web Docs — `disabled` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/disabled

---

### 2.7 `readonly`

#### Definitions

**Core Definition**
`readonly` makes the textarea's value non-editable but still focusable, selectable, and submitted.

**Technical Definition**
Boolean attribute; a read-only textarea is barred from the user *modifying* its value, but the value is a successful control and is included in submission. `required` has no effect on a read-only textarea (its value is exempt from the constraint). Users can still focus and copy.

**Beginner-Friendly Explanation**
A box you can read and copy from, but not edit; its content is still sent with the form.

#### Purposes

- **To display a value that shouldn't be edited** but should be submitted.
- **To allow copy-to-clipboard flows**.

#### Syntax Rules and Structure

```html
<textarea name="order_notes" readonly>Deliver before 5pm.</textarea>
```

**Constraints and Limitations**: Read-only does not prevent scripts from changing the value; do not use as a security control. `required` is ignored for read-only controls.

#### Annotated Example

```html
<label for="summary">Order summary</label>
<textarea id="summary" name="summary" rows="3" readonly>3 items, total $42.00.</textarea>
```

**Expected output**: Text visible, selectable, and submitted as `summary=3+items%2C+total+%2442.00.`.

**Why this output occurs**: `readonly` blocks editing only; the control still submits.

#### Real-World Cases

- Server-generated summaries, pre-filled legal text the user must accept.

#### References

- MDN Web Docs — `readonly` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/readonly

---

### 2.8 `maxlength`

#### Definitions

**Core Definition**
`maxlength` caps the value's length in characters (technically UTF-16 code units).

**Technical Definition**
Non-negative integer content attribute; participates in the `tooLong` validity check. UAs typically prevent further typing beyond the limit but allow paste to overflow with validation failure depending on the browser. Measured in **UTF-16 code units**, so emoji with surrogate pairs count as 2, and complex graphemes can count as many.

**Beginner-Friendly Explanation**
The maximum number of characters allowed.

#### Purposes

- **To limit length** for storage, layout, or business rules.

#### Syntax Rules and Structure

```html
<textarea maxlength="280"></textarea>
```

**Constraints and Limitations**: Enforced mostly client-side; server must also enforce. Counts code units not user-perceived characters — mismatches are common.

#### Annotated Example

```html
<label for="bio">Bio (max 280 chars)</label>
<textarea id="bio" name="bio" rows="4" maxlength="280"></textarea>
<p><span id="count">0</span>/280</p>
<script>
  const bio = document.getElementById('bio');
  const count = document.getElementById('count');
  bio.addEventListener('input', () => {
    count.textContent = bio.value.length; // UTF-16 code units
  });
</script>
```

**Expected output**: Typing stops at 280 characters; the counter updates live.

**Why this output occurs**: `maxlength` caps input; the script reads `value.length`.

#### Real-World Cases

- Twitter-like limits, database column caps, SMS length.

#### References

- MDN Web Docs — `maxlength` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/maxlength

---

### 2.9 `minlength`

#### Definitions

**Core Definition**
`minlength` requires the value to be at least N characters.

**Technical Definition**
Non-negative integer content attribute; participates in `tooShort` when the value is non-empty. Empty values pass `minlength` but are caught by `required`. Measured in UTF-16 code units.

**Beginner-Friendly Explanation**
The minimum number of characters required.

#### Purposes

- **To enforce minimum length** (e.g. meaningful feedback).

#### Syntax Rules and Structure

```html
<textarea minlength="20" required></textarea>
```

**Constraints and Limitations**: Empty values pass `minlength` alone; pair with `required`. A leading/trailing-whitespace-only value may fail `minlength` based on raw length; trim server-side.

#### Annotated Example

```html
<form action="/submit" method="post">
  <label for="review">Review (20–500 chars)</label>
  <textarea id="review" name="review" rows="5"
            minlength="20" maxlength="500" required></textarea>
  <button>Post</button>
</form>
```

**Expected output**: "Too short" fails until 20 chars are typed; over 500 is blocked.

**Why this output occurs**: `tooShort` triggers when non-empty value length is below `minlength`.

#### Real-World Cases

- Minimum-length reviews, feedback, and error reports.

#### References

- MDN Web Docs — `minlength` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/minlength

---

### 2.10 `wrap`

#### Definitions

**Core Definition**
`wrap` controls whether and how the textarea's line wrapping affects the submitted value.

**Technical Definition**
Enumerated attribute with keywords `soft` (default) and `hard`. With `soft`, visual soft-wraps are not serialised; the value retains the user's actual newlines. With `hard`, soft-wrap line breaks are inserted as `\r\n` in the submitted value; `hard` requires `cols` to be specified.

**Beginner-Friendly Explanation**
`soft` (default): submit exactly what the user typed. `hard`: also include the line breaks the browser inserted visually at the edge of the box.

#### Purposes

- **To control whether visual wrapping becomes part of the value**.
- **To standardise message lengths** for systems that need line-limited input.

#### Syntax Rules and Structure

```html
<textarea wrap="soft" cols="40"></textarea>
<textarea wrap="hard" cols="40"></textarea>
```

**Constraints and Limitations**: `wrap="hard"` requires `cols`. Different browsers may wrap at slightly different points based on font; `hard` therefore introduces formatting the user did not type.

#### Annotated Example

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>wrap</title></head>
<body>
  <form action="/save" method="post">
    <label for="a">Soft wrap (default)</label>
    <textarea id="a" name="a" rows="3" cols="30" wrap="soft"></textarea>

    <label for="b">Hard wrap</label>
    <textarea id="b" name="b" rows="3" cols="30" wrap="hard"></textarea>

    <button>Save</button>
  </form>
</body>
</html>
```

**Expected output**: Typing a long line in "soft" submits one line; in "hard" the same visual wrap is submitted as real newlines.

**Why this output occurs**: `hard` causes the UA to insert the visual line breaks into the submitted value.

#### Real-World Cases

- Email-composer style fields requiring wrapped text; mostly legacy patterns.

#### References

- MDN Web Docs — `wrap` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea#wrap
- WHATWG — The wrap attribute — https://html.spec.whatwg.org/multipage/form-elements.html#attr-textarea-wrap

---

## 3. Textarea Best Practices

### Definitions

**Core Definition**
Textarea best practices are guidelines that produce textareas that are accessible, usable, secure, and maintainable beyond specification conformance.

**Technical Definition**
They combine HTML conformance with WCAG success criteria (labels, contrast, error association, target size), usability principles (visible labels, character counters, sensible heights, allowing resize), security guidance (server-side length limits and output escaping to prevent XSS), and maintainability principles (using `value`/`defaultValue` correctly, using `setSelectionRange` for programmatic insertion).

**Beginner-Friendly Explanation**
The habits that make a text box easy to use and safe for the site.

---

### Purposes

- **To ensure accessibility** for keyboard and screen-reader users.
- **To improve UX** with sizing, resize behaviour, counters, and clear errors.
- **To prevent XSS and abuse** by validating length server-side and escaping output.
- **To keep behaviour predictable** across browsers and frameworks.

---

### Syntax Rules and Structure

Best practices are principles; the table summarises the core rules.

| Practice | Rule |
|---|---|
| **Always label** | `<label for>` or an ARIA name; placeholder is not a label. |
| **Do not disable resize** | Allow users to grow the box; use `resize: vertical` at most. |
| **Responsive width** | Use `width: 100%` (with `box-sizing: border-box`) instead of relying on `cols`. |
| **Sensible default height** | Match expected input length (e.g. 4–6 rows for messages). |
| **Add character counters** | Live `input` listener reflecting `maxlength`; announce politely. |
| **Trim on server** | Reject whitespace-only for `required`. |
| **Escape on render** | Never inject user text with `innerHTML` without sanitising. |
| **Bound length on server** | Enforce `maxlength` server-side too. |
| **Associate errors** | `aria-describedby`, `aria-invalid`, focus-first-invalid. |
| **Use monospace for code** | `font-family: ui-monospace, monospace` when appropriate. |
| **Avoid replacing with contenteditable** | A real textarea is accessible, form-associated, and predictable; use a rich editor only when rich text is a requirement. |
| **Set default value via content or `defaultValue`** | Prefer `defaultValue = '…'` in JS over setting `innerHTML`. |

#### Syntax Rules

1. Every textarea needs an accessible name.
2. `placeholder` must supplement, not replace, the label.
3. Any client-side `maxlength`/`minlength` must be duplicated server-side.
4. Auto-resize scripts must not break Ctrl/Cmd-Z or scroll position.

#### Constraints and Limitations

- Unicode length mismatches between code units and perceived characters require a `graphemeLength` utility for exact counting.
- Auto-resizing textareas interact with CSS grid/flex layouts; test carefully.
- Rich text is *not* available in `<textarea>`; do not fake it with contenteditable unless you can implement full keyboard and ARIA behaviour.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible Comment Box with Character Counter and Errors

**Setup guide**: Save as `textarea-best.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Textarea Best Practices</title>
  <style>
    label { display: block; font-weight: 600; margin-bottom: .25rem; }
    textarea {
      width: 100%;
      max-width: 40rem;
      box-sizing: border-box;
      padding: .5rem;
      font: inherit;
      resize: vertical;         /* allow vertical resize, keep width stable */
      min-height: 6rem;
    }
    .meta { display: flex; justify-content: space-between; font-size: .85rem; color: #555; }
    .error { color: #b00020; }
  </style>
</head>
<body>
  <form action="/comment" method="post" novalidate>
    <label for="comment">Your comment</label>
    <textarea id="comment" name="comment"
              rows="6" minlength="10" maxlength="500" required
              aria-describedby="comment-hint comment-count comment-err"></textarea>

    <div class="meta">
      <span id="comment-hint">10–500 characters.</span>
      <span id="comment-count" aria-live="polite">0/500</span>
    </div>
    <p id="comment-err" class="error" role="alert" hidden></p>

    <button>Post</button>
  </form>

  <script>
    const form = document.querySelector('form');
    const comment = document.getElementById('comment');
    const count = document.getElementById('comment-count');
    const err = document.getElementById('comment-err');

    function updateCount() {
      count.textContent = `${comment.value.length}/500`;
    }
    comment.addEventListener('input', updateCount);
    updateCount();

    form.addEventListener('submit', (e) => {
      err.hidden = true;
      comment.setAttribute('aria-invalid', 'false');
      if (!form.checkValidity()) {
        e.preventDefault();
        let msg = '';
        if (comment.validity.valueMissing) msg = 'Please write a comment.';
        else if (comment.validity.tooShort) msg = 'Please write at least 10 characters.';
        else if (comment.validity.tooLong) msg = 'Please keep it under 500 characters.';
        err.textContent = msg;
        err.hidden = !msg;
        comment.setAttribute('aria-invalid', 'true');
        comment.focus();
      }
    });
  </script>
</body>
</html>
```

**Expected output**: A wide, vertically resizable comment box with a live counter. Submitting empty, too short, or too long reveals an inline error and focuses the textarea.

**Why this output occurs**: `aria-describedby` links hint/counter/error; `aria-live="polite"` announces counter updates; `checkValidity()` and `validity` surface the reason; `resize: vertical` preserves the width while allowing height growth.

---

#### Example 2: Auto-Resizing Textarea

**Setup guide**: Save as `textarea-autosize.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Auto-resizing Textarea</title>
  <style>
    textarea { width: 100%; max-width: 40rem; box-sizing: border-box; overflow: hidden; }
  </style>
</head>
<body>
  <label for="t">Notes</label>
  <textarea id="t" name="t" rows="3"></textarea>

  <script>
    const t = document.getElementById('t');
    function resize() {
      t.style.height = 'auto';          // reset to measure
      t.style.height = t.scrollHeight + 'px'; // fit content
    }
    t.addEventListener('input', resize);
    resize();
  </script>
</body>
</html>
```

**Expected output**: The textarea grows as you type and shrinks when you delete lines.

**Why this output occurs**: Setting `height: auto` then to `scrollHeight` measures content; the `input` listener keeps it updated.

---

#### Example 3: Programmatic Insertion at Caret

**Setup guide**: Save as `textarea-insert.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Insert at Caret</title></head>
<body>
  <label for="msg">Message</label>
  <textarea id="msg" name="msg" rows="4">Dear support,</textarea>
  <button type="button" id="insert">Insert template</button>

  <script>
    const msg = document.getElementById('msg');
    document.getElementById('insert').addEventListener('click', () => {
      // Insert at current caret position, replacing selection
      msg.setRangeText('\n\nEnvironment: production\nVersion: 1.2.3', msg.selectionStart, msg.selectionEnd, 'end');
      msg.focus();
    });
  </script>
</body>
</html>
```

**Expected output**: Clicking "Insert template" inserts the text at the caret; the cursor ends after the inserted text.

**Why this output occurs**: `setRangeText` replaces the selection (or inserts at the caret) and moves the cursor per the `'end'` argument, preserving undo integrity in most browsers.

---

### Real-World Cases

1. **Support tickets** — labeled description textarea with length limits and counters; markdown guidance as hint text.
2. **Code editors** — monospaced textarea with tab-key handling and auto-indent scripts.
3. **Chat/comment fields** — auto-resizing textarea with a counter and send shortcut (`Ctrl/Cmd+Enter`).
4. **Bulk-edit admin panels** — read-only textarea holding a preview of submitted JSON or text.

---

### References

- W3C WAI — Forms Tutorial: Multi-line text — https://www.w3.org/WAI/tutorials/forms/
- W3C — Web Content Accessibility Guidelines (WCAG) — https://www.w3.org/WAI/standards-guidelines/wcag/
- MDN Web Docs — `<textarea>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea
- MDN Web Docs — HTMLTextAreaElement — https://developer.mozilla.org/en-US/docs/Web/API/HTMLTextAreaElement
- WHATWG — The textarea element — https://html.spec.whatwg.org/multipage/form-elements.html#the-textarea-element
- OWASP — Cross Site Scripting Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html

---

## Summary Tables

### Textarea Attributes at a Glance

| Attribute | Type | Effect |
|---|---|---|
| `name` | string | Submission key; required to submit |
| `rows` | integer ≥ 1 | Initial visible height in lines (default 2) |
| `cols` | integer ≥ 1 | Initial visible width in characters (default 20) |
| `placeholder` | string | Hint shown when empty; not a label |
| `required` | boolean | Enforces non-empty value (whitespace counts as non-empty) |
| `disabled` | boolean | No interaction, no focus, no submission |
| `readonly` | boolean | Non-editable but focusable, selectable, submitted |
| `maxlength` | integer | Max length in UTF-16 code units; `tooLong` |
| `minlength` | integer | Min length; `tooShort` when non-empty |
| `wrap` | `soft` / `hard` | Whether soft-wrap is serialised on submit |

### `<input type="text">` vs `<textarea>`

| Aspect | `<input type="text">` | `<textarea>` |
|---|---|---|
| Lines | Single | Multiple |
| Content | Void (no children) | Text content |
| Default value source | `value` attribute | Child text content (or `defaultValue`) |
| Newlines | Not preserved (single-line) | Preserved |
| Resize | No | Usually yes (CSS `resize`) |
| Key attributes | `maxlength`, `pattern`, `size` | `rows`, `cols`, `wrap` |
| Rich content | No | No |

### Common Pitfalls

| Pitfall | Consequence | Fix |
|---|---|---|
| Missing `name` | Never submitted | Add `name` |
| Placeholder used as label | No accessible name; vanishes on typing | Add `<label>` |
| Not escaping `<`/`&` in default content | Parser may terminate the element | Use `&lt;`, `&amp;` |
| Setting `innerHTML` to change value | Destroys undo / breaks association | Set `.value` / `.defaultValue` |
| Whitespace-only passes `required` | Server receives junk | Trim server-side; use custom validity if strict |
| `disabled` when you meant "submit but not edit" | Value lost on submit | Use `readonly` |
| Disabling resize | Poor UX, especially on mobile | `resize: vertical` at minimum |
| Fixed `cols` breaking on mobile | Overflow / horizontal scroll | `width: 100%` + `box-sizing: border-box` |
| Trusting client-side `maxlength` | Server gets huge payloads | Enforce server-side |

---

## Final Notes on Deprecated, Version-Specific, and Security-Sensitive Features

- **`wrap="hard"`** requires `cols` and inserts line breaks the user did not type; it is uncommon and can surprise servers. Prefer the default `soft`.
- **Leading newline** immediately after `<textarea>` is ignored — but any subsequent whitespace is preserved. Indenting your markup shifts the default value.
- **`innerHTML`** should not be used to set or read user-editable textarea content; use `.value` / `.defaultValue` to keep undo and default state correct.
- **`maxlength`/`minlength` count UTF-16 code units**, so emoji and combined characters count inconsistently with user expectations. Use a grapheme-aware counter when correctness matters.
- **`required` does not reject whitespace-only values** — trim on the server and treat trim-empty as missing.
- **Rich text cannot be produced with `<textarea>`**; the correct path is a real rich-text editor library with proper ARIA and keyboard support — never a raw `contenteditable` div.
- **User text is potentially hostile** — always escape or sanitise on render (server-side HTML escaping or a strict sanitiser) to prevent XSS.
- **`form`, `dirname`, `autocomplete`, `autofocus`** also apply to `<textarea>` (via the shared form-associated control attributes) and behave similarly to `<input>` — use them per the corresponding cheat sheet sections.

---

This cheat sheet has covered the `<textarea>` element in full: its definition, all ten attributes (`name`, `rows`, `cols`, `placeholder`, `required`, `disabled`, `readonly`, `maxlength`, `minlength`, `wrap`), and production-grade best practices — with annotated, executable examples and authoritative references throughout. Use it as a reference and a teaching aid, and validate markup against the WHATWG HTML Living Standard and WCAG guidance.