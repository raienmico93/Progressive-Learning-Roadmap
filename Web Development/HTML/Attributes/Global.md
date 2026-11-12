# Global Attributes — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Global attributes are attributes that are common to all HTML elements. They can be used on every element — standard or non-standard — although they may have no effect on some elements in certain contexts. They provide a universal mechanism for identifying, classifying, styling, internationalising, and adding behaviour to elements without being tied to any specific element type.

**Technical Definition**
In the WHATWG HTML Living Standard, global attributes are defined in Section 3.2.5. The specification lists a core set of attributes that “may be specified on all HTML elements,” including `accesskey`, `autocapitalize`, `autofocus`, `class`, `contenteditable`, `dir`, `draggable`, `enterkeyhint`, `exportparts`, `hidden`, `id`, `inert`, `inputmode`, `is`, `itemid`, `itemprop`, `itemref`, `itemscope`, `itemtype`, `lang`, `nonce`, `part`, `popover`, `slot`, `spellcheck`, `style`, `tabindex`, `title`, `translate`, and `virtualkeyboardpolicy`. The ARIA `role` attribute and `aria-*` states/properties are also global in effect, though defined by the ARIA specification rather than HTML. Global attributes are parsed into the DOM as standard attributes and are accessible via the `Element` interface.

**Beginner-Friendly Explanation**
Global attributes are the “universal settings” you can attach to any HTML tag. Whether you are working with a `<div>`, an `<img>`, a `<button>`, or even a custom element, attributes like `class`, `id`, `lang`, and `hidden` are available to you. They let you identify elements, style them, hide them, make them editable, add keyboard shortcuts, and much more — without needing a special attribute for every different tag.

---

### Key Characteristics

- **Universal applicability**: Global attributes can be specified on every HTML element, including custom elements and even non-standard elements.
- **Mixed semantics**: Some global attributes are **enumerated** (e.g. `dir`, `contenteditable`, `draggable`, `translate`), some are **boolean** (e.g. `hidden`, `inert`, `itemscope`), and others accept **free-form strings or integers** (e.g. `class`, `id`, `tabindex`).
- **No effect guaranteed**: An element may accept a global attribute without the attribute having any functional effect on that element. For example, `hidden` works on all elements, but `spellcheck` only has meaning on editable content.
- **Accessibility-critical subset**: `lang`, `role`, `aria-*`, and `tabindex` are central to making content usable by assistive technologies.
- **Security-related subset**: `nonce` is used with Content Security Policy to authorise inline scripts and styles.
- **Web Components subset**: `is`, `slot`, `part`, and `exportparts` enable custom element and Shadow DOM integration.
- **Microdata subset**: `itemid`, `itemprop`, `itemref`, `itemscope`, and `itemtype` enable machine-readable structured data.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, tags, and attributes.
2. **The concept of the DOM**: global attributes become properties and attributes on DOM elements.
3. **Introductory CSS and JavaScript**: `class` and `id` are the primary hooks for styling and scripting.
4. **Awareness of accessibility**: `lang`, `role`, and `tabindex` directly affect screen-reader and keyboard users.
5. **An HTML validator**: for checking conformance and catching misuse of global attributes.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | `class`, `style`, `part`, and `exportparts` are styling hooks, including for Shadow DOM. |
| **JavaScript / DOM** | `id`, `class`, `dataset` (via `data-*`), and `is` are accessible through DOM APIs. |
| **Accessibility (ARIA)** | `role`, `aria-*`, `lang`, and `tabindex` determine how assistive technologies interpret content. |
| **Internationalisation** | `lang`, `dir`, and `translate` control language, direction, and translation behaviour. |
| **Security** | `nonce` supports Content Security Policy for inline scripts and styles. |
| **Web Components** | `is`, `slot`, `part`, and `exportparts` integrate custom elements with Shadow DOM. |
| **Structured data** | `itemscope`, `itemtype`, `itemprop`, `itemid`, and `itemref` enable microdata. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. `accesskey` — keyboard shortcut hint
2. `autocapitalize` — automatic capitalisation control
3. `autocorrect` — automatic spelling correction control
4. `autofocus` — initial focus control
5. `class` — space-separated class list
6. `contenteditable` — editable content control
7. `dir` — text directionality
8. `draggable` — drag-and-drop enablement
9. `enterkeyhint` — virtual keyboard enter-key label hint
10. `exportparts` — Shadow DOM part re-export
11. `hidden` — hiding control
12. `id` — unique identifier
13. `inert` — interaction and accessibility-tree removal
14. `inputmode` — virtual keyboard type hint
15. `is` — customised built-in element reference
16. `itemid` — microdata item identifier
17. `itemprop` — microdata property name
18. `itemref` — microdata property association
19. `itemscope` — microdata item scope
20. `itemtype` — microdata vocabulary URL
21. `lang` — content language
22. `nonce` — CSP cryptographic nonce
23. `part` — Shadow DOM part name
24. `popover` — popover element designation
25. `role` — ARIA role
26. `slot` — Shadow DOM slot assignment
27. `spellcheck` — spell-checking control
28. `style` — inline CSS declarations
29. `tabindex` — focusability and tab order
30. `title` — advisory information
31. `translate` — translation control
32. `virtualkeyboardpolicy` — virtual keyboard visibility control
33. **Global attribute best practices**

---

## 1. `accesskey` — The Keyboard Shortcut Hint

### Definitions

**Core Definition**
The `accesskey` attribute provides a hint for generating a keyboard shortcut for the current element. It consists of a space-separated list of characters; the browser uses the first one that exists on the user’s keyboard layout.

**Technical Definition**
`accesskey` is a global attribute whose value is a space-separated token list of characters. The browser maps the first available character to a platform-specific shortcut (e.g. Alt+key on Windows, Ctrl+Alt+key on macOS). If multiple elements declare the same `accesskey`, browser behaviour is implementation-dependent.

**Beginner-Friendly Explanation**
`accesskey` lets you suggest a keyboard shortcut for a button or link. For example, `accesskey="s"` might let users press Alt+S to activate a “Save” button.

---

### Purposes

- **To provide a keyboard shortcut hint** for frequently used controls.
- **To improve efficiency** for power users who prefer keyboard navigation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button accesskey="s">Save</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `accesskey="s"` | A space-separated list of candidate shortcut characters. |
| Browser behaviour | Selects the first character available on the keyboard layout. |

#### Syntax Rules

1. **The value is a space-separated list of characters**.
2. **The browser chooses the first available character**.
3. **Activation requires a modifier key**, which varies by platform and browser.

#### Constraints and Limitations

- **Discouraged by MDN and many accessibility experts**: `accesskey` conflicts with screen-reader and browser shortcuts, and the required modifier keys vary unpredictably across platforms.
- Shortcut conflicts are common and can prevent users from activating the intended control.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accesskey on a Button

**Setup Guide**: Save as `accesskey.html` and try the platform-specific shortcut.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accesskey Example</title>
</head>
<body>
  <button accesskey="s" onclick="alert('Saved!')">Save (Alt+S)</button>
  <button accesskey="c" onclick="alert('Cancelled!')">Cancel (Alt+C)</button>
</body>
</html>
```

**Expected output**: The buttons can be activated via the platform’s access-key shortcut (e.g. Alt+S on Windows).

**Why this output occurs**: The `accesskey` attribute registers the character as a potential shortcut. The browser combines it with the platform’s modifier key.

---

### Real-World Cases

- **Legacy enterprise applications**: Access keys are sometimes used in intranet applications.
- **Modern practice**: Rarely recommended due to accessibility conflicts.

---

### References

- MDN Web Docs — `accesskey` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/accesskey
- MDN Web Docs — Global attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes

---

## 2. `autocapitalize` — The Automatic Capitalisation Control

### Definitions

**Core Definition**
The `autocapitalize` attribute controls whether inputted text is automatically capitalised and, if so, in what manner.

**Technical Definition**
`autocapitalize` is an enumerated global attribute that applies to editable content. Valid values include `none`/`off`, `sentences`/`on`, `words`, and `characters`. The attribute is a hint to the browser’s virtual keyboard and input method editor (IME).

**Beginner-Friendly Explanation**
`autocapitalize` tells the browser whether to automatically capitalise letters as you type. You can disable it for usernames or enable it for sentences.

---

### Purposes

- **To control automatic capitalisation** in text inputs and editable content.
- **To prevent unwanted capitalisation** in fields like usernames or codes.
- **To improve mobile typing experience** by matching the expected input format.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<input type="text" autocapitalize="none" name="username">
<textarea autocapitalize="sentences"></textarea>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `none` / `off` | No automatic capitalisation. |
| `sentences` / `on` | Capitalise the first letter of each sentence. |
| `words` | Capitalise the first letter of each word. |
| `characters` | Capitalise every character. |

#### Syntax Rules

1. **Applies to editable content**: `<input>`, `<textarea>`, and `contenteditable` elements.
2. **Values are enumerated keywords**.
3. **Behaviour is a hint**; the browser and IME may not honour it in all cases.

#### Constraints and Limitations

- **Not supported on all platforms**; mobile browsers may implement it differently.
- It does not transform text server-side; it only affects input behaviour.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling Capitalisation for a Username

**Setup Guide**: Save as `autocapitalize.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Autocapitalize Example</title>
</head>
<body>
  <label for="username">Username (lowercase only):</label>
  <input type="text" id="username" name="username" autocapitalize="none">
</body>
</html>
```

**Expected output**: On mobile devices, the virtual keyboard does not automatically capitalise the first letter.

**Why this output occurs**: `autocapitalize="none"` signals to the browser’s IME that no automatic capitalisation should occur.

---

### Real-World Cases

1. **Username and email fields**: Disabling capitalisation prevents typos.
2. **Sentence-style text areas**: Enabling sentence capitalisation improves mobile writing.
3. **Code or token inputs**: Disabling capitalisation prevents unwanted transformations.

---

### References

- MDN Web Docs — `autocapitalize` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/autocapitalize

---

## 3. `autocorrect` — The Automatic Spelling Correction Control

### Definitions

**Core Definition**
The `autocorrect` attribute controls whether input text is automatically corrected for spelling errors. It can be applied to elements with editable text, except `<input type="password">`, `<input type="email">`, and `<input type="url">`.

**Technical Definition**
`autocorrect` is an enumerated global attribute with values `on` and `off`. It is a hint to the browser’s autocorrection engine. Unlike `spellcheck`, which marks errors, `autocorrect` actively replaces text.

**Beginner-Friendly Explanation**
`autocorrect` tells the browser whether to automatically fix spelling mistakes as you type. Turn it off for fields where exact spelling matters, like codes or usernames.

---

### Purposes

- **To enable or disable automatic spelling correction** in editable fields.
- **To prevent unwanted corrections** in fields containing codes, names, or technical terms.
- **To improve mobile typing accuracy** for prose.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<textarea autocorrect="off">Exact text only</textarea>
<input type="text" autocorrect="on" name="message">
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `on` | Enable automatic correction. |
| `off` | Disable automatic correction. |

#### Syntax Rules

1. **Applies to editable text elements** (inputs, textareas, contenteditable).
2. **Values are `on` and `off`**.
3. **Not applicable to password, email, and URL inputs** in some implementations.

#### Constraints and Limitations

- **Browser and platform support varies**; it is primarily a mobile-input hint.
- Autocorrection behaviour is ultimately controlled by the user’s device settings.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling Autocorrect for a Code Field

**Setup Guide**: Save as `autocorrect.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Autocorrect Example</title>
</head>
<body>
  <label for="code">Enter code:</label>
  <input type="text" id="code" name="code" autocorrect="off">
</body>
</html>
```

**Expected output**: On supported mobile browsers, typing in the field does not trigger automatic spelling replacement.

**Why this output occurs**: `autocorrect="off"` instructs the browser’s autocorrection engine to leave the text unchanged.

---

### Real-World Cases

1. **Promo codes and coupon fields**: Preventing autocorrection preserves exact codes.
2. **Usernames and handles**: Avoiding unwanted capitalisation or replacement.
3. **Multilingual content**: Disabling autocorrection when typing in a non-default language.

---

### References

- MDN Web Docs — `autocorrect` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/autocorrect

---

## 4. `autofocus` — The Initial Focus Control

### Definitions

**Core Definition**
The `autofocus` attribute indicates that an element is to be focused on page load, or as soon as the `<dialog>` it is part of is displayed. It is a boolean attribute.

**Technical Definition**
`autofocus` is a boolean global attribute. Only one element per document (or per dialog) should have it. The browser focuses the element automatically when the document loads or when a dialog is shown.

**Beginner-Friendly Explanation**
`autofocus` makes an element (like a search box) automatically active when the page loads, so the user can start typing right away.

---

### Purposes

- **To focus an element automatically** on page load or dialog display.
- **To improve user efficiency** by placing the cursor in the most likely input.
- **To direct attention** to a primary action or message.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<input type="search" autofocus placeholder="Search…">
```

**Component breakdown**

| Component | Description |
|---|---|
| `autofocus` | Boolean attribute; presence activates automatic focus. |

#### Syntax Rules

1. **Boolean attribute**: presence means true.
2. **Only one element per document** should have `autofocus`.
3. **In dialogs**, autofocus applies when the dialog is shown.

#### Constraints and Limitations

- **Only one autofocus per document**; multiple elements with `autofocus` may cause unpredictable behaviour.
- **Can be disruptive** for screen-reader and keyboard users if focus jumps unexpectedly.
- **Mobile browsers may ignore it** or may not show the virtual keyboard automatically.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Autofocus on a Search Field

**Setup Guide**: Save as `autofocus.html`. Reload and observe the focus.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Autofocus Example</title>
</head>
<body>
  <form action="/search">
    <label for="q">Search:</label>
    <input type="search" id="q" name="q" autofocus>
    <button type="submit">Go</button>
  </form>
</body>
</html>
```

**Expected output**: On page load, the search input is focused and ready for typing.

**Why this output occurs**: The `autofocus` attribute instructs the browser to focus the element when the document loads.

---

### Real-World Cases

1. **Search pages**: Focusing the search box immediately.
2. **Login forms**: Focusing the username or email field.
3. **Dialog forms**: Focusing the first input when a modal opens.

---

### References

- MDN Web Docs — `autofocus` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/autofocus
- WHATWG — `autofocus` — https://html.spec.whatwg.org/multipage/interaction.html#the-autofocus-attribute

---

## 5. `class` — The Space-Separated Class List

### Definitions

**Core Definition**
The `class` attribute is a space-separated list of the classes of the element. Classes allow CSS and JavaScript to select and access specific elements via class selectors or functions like `Document.getElementsByClassName()`.

**Technical Definition**
`class` is a global attribute whose value is a set of space-separated tokens. In the DOM, it is exposed as `element.classList` (a `DOMTokenList`) and `element.className` (a string). Classes have no inherent semantics; they are author-defined hooks for styling and scripting.

**Beginner-Friendly Explanation**
`class` lets you label elements so you can style or select them later. For example, `class="card highlight"` gives an element two classes that CSS can target.

---

### Purposes

- **To provide styling hooks** for CSS class selectors.
- **To provide scripting hooks** for JavaScript selection and manipulation.
- **To group elements** logically without using IDs.
- **To apply utility classes** from CSS frameworks (e.g. Tailwind, Bootstrap).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div class="card highlight large">Content</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `class="card highlight"` | Space-separated list of class names. |
| Multiple classes | Allowed and common. |
| Case sensitivity | Class names are case-sensitive in selectors (except in quirks mode). |

#### Syntax Rules

1. **Values are space-separated tokens**.
2. **Classes may be reused** across any number of elements.
3. **Class names should be meaningful** and follow a consistent naming convention (e.g. BEM).

#### Constraints and Limitations

- **Classes have no semantic meaning**; they are purely author-defined.
- **Overly generic class names** can cause conflicts in large codebases.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Classes for Styling and Scripting

**Setup Guide**: Save as `class.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Class Example</title>
  <style>
    .card { border: 1px solid #ccc; padding: 16px; }
    .highlight { background: #fff3cd; }
  </style>
</head>
<body>
  <div class="card highlight" id="myCard">
    <p>This card has two classes.</p>
  </div>

  <script>
    const card = document.querySelector('.card.highlight');
    console.log(card.className); // "card highlight"
  </script>
</body>
</html>
```

**Expected output**: A bordered card with a yellow highlight background. The console logs the class string.

**Why this output occurs**: The `class` attribute provides both styling hooks for CSS and a selection hook for JavaScript.

---

### Real-World Cases

1. **CSS frameworks**: Utility classes like `text-center`, `mt-4`, and `btn-primary`.
2. **Component styling**: BEM classes like `card__title--large`.
3. **JavaScript selection**: `document.querySelectorAll('.js-toggle')`.

---

### References

- MDN Web Docs — `class` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/class

---

## 6. `contenteditable` — The Editable Content Control

### Definitions

**Core Definition**
The `contenteditable` attribute is an enumerated attribute indicating whether the element should be editable by the user. If so, the browser modifies its widget to allow editing.

**Technical Definition**
`contenteditable` is an enumerated global attribute with values `true` (or empty string) for editable, `false` for non-editable, and `plaintext-only` for raw-text-only editing. It is inherited by descendants unless overridden. The attribute is part of the HTML Editing APIs.

**Beginner-Friendly Explanation**
`contenteditable` makes any element editable directly in the browser, turning it into a mini text editor. You can type, delete, and paste inside it.

---

### Purposes

- **To make content editable in the browser** without a separate `<textarea>`.
- **To build rich-text editors** and inline editing interfaces.
- **To allow users to modify specific parts of a page**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div contenteditable="true">Edit me!</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` / empty string | Editable. |
| `false` | Not editable. |
| `plaintext-only` | Editable as plain text only (no rich formatting). |

#### Syntax Rules

1. **Inherited by descendants**; a child can override with `contenteditable="false"`.
2. **Values are enumerated keywords**.
3. **`plaintext-only` disables rich-text formatting**.

#### Constraints and Limitations

- **Rich-text behaviour varies across browsers**.
- **Does not provide a form control**; values are not automatically submitted.
- **Accessibility requires careful handling** of focus and announcements.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Editable Div with a Non-Editable Child

**Setup Guide**: Save as `contenteditable.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Contenteditable Example</title>
</head>
<body>
  <div contenteditable="true">
    <p>You can edit this paragraph.</p>
    <p contenteditable="false">But this paragraph is locked.</p>
  </div>
</body>
</html>
```

**Expected output**: The first paragraph is editable; the second is locked and cannot be modified.

**Why this output occurs**: `contenteditable="true"` on the parent makes its subtree editable, but the child’s `contenteditable="false"` overrides the inheritance.

---

### Real-World Cases

1. **Inline editors**: Click-to-edit headings and descriptions.
2. **Rich-text editors**: Building custom WYSIWYG editors.
3. **Note-taking apps**: Editable note surfaces.

---

### References

- MDN Web Docs — `contenteditable` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/contenteditable

---

## 7. `dir` — The Text Directionality Attribute

### Definitions

**Core Definition**
The `dir` attribute is an enumerated attribute indicating the directionality of the element’s text. It can have the values `ltr` (left-to-right), `rtl` (right-to-left), or `auto` (browser-determined).

**Technical Definition**
`dir` is an enumerated global attribute. The `auto` value uses a basic algorithm that scans the element’s text until it finds a character with strong directionality, then applies that direction to the whole element.

**Beginner-Friendly Explanation**
`dir` tells the browser which direction text should flow. Use `rtl` for languages like Arabic or Hebrew, and `auto` to let the browser figure it out.

---

### Purposes

- **To declare text direction** for right-to-left languages.
- **To automatically detect direction** for user-generated content.
- **To support bidirectional text** in multilingual documents.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p dir="rtl">مرحبا بالعالم</p>
<p dir="auto">User-generated text</p>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `ltr` | Left-to-right (default for most languages). |
| `rtl` | Right-to-left (Arabic, Hebrew, etc.). |
| `auto` | Browser detects direction from first strong character. |

#### Syntax Rules

1. **Values are enumerated keywords**.
2. **`dir="auto"` is useful for user-generated content**.
3. **The attribute is inherited** by descendants.

#### Constraints and Limitations

- **`auto` detection can be imperfect** for mixed-direction text.
- **`dir` affects layout and alignment**, not only text order.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Right-to-Left and Auto-Detected Text

**Setup Guide**: Save as `dir.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Dir Example</title>
</head>
<body>
  <p dir="rtl">This paragraph is rendered right-to-left.</p>
  <p dir="auto">مرحبا</p>
</body>
</html>
```

**Expected output**: The first paragraph is right-aligned and its text flows right-to-left. The second paragraph is automatically detected as right-to-left due to the Arabic text.

**Why this output occurs**: `dir="rtl"` explicitly sets right-to-left direction; `dir="auto"` scans the content and applies the detected direction.

---

### Real-World Cases

1. **Arabic and Hebrew websites**: Setting `dir="rtl"` on the `<html>` element.
2. **User comments and messages**: Using `dir="auto"` to respect the user’s language.
3. **Multilingual content**: Applying `dir` to individual phrases or sections.

---

### References

- MDN Web Docs — `dir` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/dir

---

## 8. `draggable` — The Drag-and-Drop Enablement

### Definitions

**Core Definition**
The `draggable` attribute is an enumerated attribute indicating whether the element can be dragged using the Drag and Drop API. It can have the values `true` or `false`.

**Technical Definition**
`draggable` is an enumerated global attribute. When `true`, the element becomes draggable and participates in the HTML Drag and Drop API. Images and links are draggable by default; other elements require `draggable="true"`.

**Beginner-Friendly Explanation**
`draggable` makes an element draggable. You can drag it with the mouse and drop it elsewhere, if you write the JavaScript to handle the drag-and-drop events.

---

### Purposes

- **To make an element draggable** for custom drag-and-drop interfaces.
- **To enable the HTML Drag and Drop API** on arbitrary elements.
- **To build sortable lists, kanban boards, and file-upload zones**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div draggable="true">Drag me</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Element is draggable. |
| `false` | Element is not draggable. |

#### Syntax Rules

1. **Values are enumerated**: `true` or `false`.
2. **Requires drag-and-drop event handlers** (`dragstart`, `dragover`, `drop`) to be useful.
3. **Images and links are draggable by default**.

#### Constraints and Limitations

- **`draggable` alone does not implement drag-and-drop**; JavaScript event handling is required.
- **Accessibility requires alternative input methods** for drag-and-drop operations.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Making a Div Draggable

**Setup Guide**: Save as `draggable.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Draggable Example</title>
</head>
<body>
  <div draggable="true" ondragstart="event.dataTransfer.setData('text/plain', 'Dragged!')">
    Drag this box
  </div>

  <div ondragover="event.preventDefault()" ondrop="event.preventDefault(); alert(event.dataTransfer.getData('text/plain'))">
    Drop here
  </div>
</body>
</html>
```

**Expected output**: The first box can be dragged and dropped onto the second box, which shows an alert with the dragged data.

**Why this output occurs**: `draggable="true"` makes the element draggable. The event handlers transfer data between the drag source and the drop target.

---

### Real-World Cases

1. **Sortable lists**: Reordering items by dragging.
2. **Kanban boards**: Moving cards between columns.
3. **File upload zones**: Dragging files onto a drop area.

---

### References

- MDN Web Docs — `draggable` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/draggable

---

## 9. `enterkeyhint` — The Virtual Keyboard Enter-Key Hint

### Definitions

**Core Definition**
The `enterkeyhint` attribute hints what action label (or icon) to present for the enter key on virtual keyboards.

**Technical Definition**
`enterkeyhint` is an enumerated global attribute that accepts keywords such as `enter`, `done`, `go`, `next`, `previous`, `search`, `send`, and `leave`. It is primarily used on `<input>` and `<textarea>` elements to customise the mobile keyboard’s action key.

**Beginner-Friendly Explanation**
`enterkeyhint` changes the label or icon on the mobile keyboard’s enter key. For example, it can say “Search” instead of “Enter” when typing in a search box.

---

### Purposes

- **To customise the virtual keyboard’s enter-key label** for the expected action.
- **To improve mobile form usability** by matching the action to the field’s purpose.
- **To guide users toward the next logical action**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<input type="search" enterkeyhint="search">
<input type="text" enterkeyhint="next">
```

**Component breakdown**

| Keyword | Suggested action |
|---|---|
| `enter` | Default enter action. |
| `done` | Finish input. |
| `go` | Navigate to the target. |
| `next` | Move to next field. |
| `previous` | Move to previous field. |
| `search` | Perform a search. |
| `send` | Send a message or form. |

#### Syntax Rules

1. **Applies mainly to input and editable elements**.
2. **Values are enumerated keywords**.
3. **Primarily affects mobile virtual keyboards**.

#### Constraints and Limitations

- **Support varies by browser and keyboard app**.
- **It is a hint**; the keyboard may not honour it.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Search Input with Enter-Key Hint

**Setup Guide**: Save as `enterkeyhint.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Enterkeyhint Example</title>
</head>
<body>
  <form action="/search">
    <label for="q">Search:</label>
    <input type="search" id="q" name="q" enterkeyhint="search">
    <button type="submit">Go</button>
  </form>
</body>
</html>
```

**Expected output**: On a mobile device, the virtual keyboard’s enter key displays “Search” instead of the generic “Enter”.

**Why this output occurs**: `enterkeyhint="search"` signals the keyboard to label the action key accordingly.

---

### Real-World Cases

1. **Search fields**: `enterkeyhint="search"`.
2. **Multi-step forms**: `enterkeyhint="next"` to move between fields.
3. **Messaging apps**: `enterkeyhint="send"` for message inputs.

---

### References

- MDN Web Docs — `enterkeyhint` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/enterkeyhint

---

## 10. `exportparts` — The Shadow DOM Part Re-Export

### Definitions

**Core Definition**
The `exportparts` attribute is used to transitively export shadow parts from a nested shadow tree into a containing light tree.

**Technical Definition**
`exportparts` is a global attribute that accepts a comma-separated list of part names (or `outer: inner` mappings). It allows a component that contains another component with a shadow tree to re-export the inner component’s parts so that an outer consumer can style them with `::part()`.

**Beginner-Friendly Explanation**
`exportparts` lets a component pass through the styleable “parts” of another component nested inside it. It is used with Shadow DOM to make nested internal elements styleable from outside.

---

### Purposes

- **To re-export shadow parts** from nested shadow trees.
- **To allow outer consumers to style deeply nested component internals**.
- **To compose styleable components** without breaking encapsulation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<my-outer exportparts="button, input: field">
  <my-inner></my-inner>
</my-outer>
```

**Component breakdown**

| Component | Description |
|---|---|
| `exportparts="button"` | Re-exports the part named `button`. |
| `exportparts="input: field"` | Re-exports `input` under the new name `field`. |
| Comma separation | Multiple parts can be exported. |

#### Syntax Rules

1. **Values are a comma-separated list** of part names or mappings.
2. **Used on host elements** that contain nested shadow trees.
3. **Works with the `::part()` CSS pseudo-element**.

#### Constraints and Limitations

- **Requires Shadow DOM support**.
- **Only parts explicitly named with `part` can be exported**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Re-Exporting a Nested Component’s Part

**Setup Guide**: Save as `exportparts.html`. This example illustrates the concept; full Shadow DOM setup is required.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Exportparts Example</title>
</head>
<body>
  <my-outer exportparts="label">
    <my-inner></my-inner>
  </my-outer>

  <!-- The inner component defines a part named "label" -->
  <!-- The outer component re-exports it -->
</body>
</html>
```

**Expected output**: An outer consumer can style the inner component’s label part via `my-outer::part(label)`.

**Why this output occurs**: `exportparts` bridges the shadow boundary, allowing the part to be targeted from the outer tree.

---

### Real-World Cases

1. **Design system composition**: Complex components re-exporting parts of nested components.
2. **Themable components**: Allowing consumers to style internal parts of nested widgets.
3. **Nested Web Components**: Composing components while preserving styleability.

---

### References

- MDN Web Docs — `exportparts` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/exportparts

---

## 11. `hidden` — The Hiding Control

### Definitions

**Core Definition**
The `hidden` attribute is an enumerated attribute indicating that the element is not yet, or is no longer, relevant. The browser will not render such elements.

**Technical Definition**
`hidden` is a global attribute. In HTML, it is an enumerated attribute with values `hidden`, `until-found`, and the empty string. In practice, its presence hides the element from rendering. The `until-found` value makes the element searchable and revealed when found.

**Beginner-Friendly Explanation**
`hidden` hides an element completely. It is not displayed on the page. Use it to hide content that is not yet relevant, like a login panel before the user signs in.

---

### Purposes

- **To hide an element from rendering**.
- **To remove content from view until it becomes relevant**.
- **To support search-and-reveal behaviour** with `hidden="until-found"`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p hidden>This paragraph is hidden.</p>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| (empty) / `hidden` | Element is hidden. |
| `until-found` | Element is hidden but searchable and revealed when found. |

#### Syntax Rules

1. **Presence hides the element**.
2. **`until-found` enables find-in-page discovery**.
3. **Do not use `hidden` for content that should be accessible to assistive technology**; use visually-hidden CSS techniques instead.

#### Constraints and Limitations

- **`hidden` removes the element from rendering but not from the DOM**.
- **`hidden="until-found"` support varies**.
- **CSS `display` can override `hidden`**, so do not rely on it for security.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hiding and Revealing Content

**Setup Guide**: Save as `hidden.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hidden Example</title>
</head>
<body>
  <p hidden id="secret">This content is hidden.</p>
  <button onclick="document.getElementById('secret').hidden = false">Show</button>
</body>
</html>
```

**Expected output**: The paragraph is hidden initially; clicking the button reveals it.

**Why this output occurs**: The `hidden` attribute hides the element; setting `hidden = false` removes the attribute and makes it visible.

---

### Real-World Cases

1. **Conditional UI**: Hiding panels until the user takes an action.
2. **Progressive disclosure**: Hiding advanced settings by default.
3. **Search-and-reveal**: Using `hidden="until-found"` for collapsible search results.

---

### References

- MDN Web Docs — `hidden` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/hidden

---

## 12. `id` — The Unique Identifier

### Definitions

**Core Definition**
The `id` attribute defines a unique identifier (ID) that must be unique within the whole document. Its purpose is to identify the element when linking (using a fragment identifier), scripting, or styling (with CSS).

**Technical Definition**
`id` is a global attribute whose value is a case-sensitive string that must be unique in the document. It is exposed as `element.id` and can be used with `getElementById()`, fragment navigation (`#id`), and CSS `#id` selectors.

**Beginner-Friendly Explanation**
`id` is a unique name for an element. You can use it to link directly to the element, select it with JavaScript, or style it with CSS.

---

### Purposes

- **To uniquely identify an element** in the document.
- **To enable fragment navigation** (`href="#id"`).
- **To provide a scripting hook** (`getElementById`).
- **To provide a CSS styling hook** (`#id` selector).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div id="unique-name">Content</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `id="unique-name"` | A unique, case-sensitive string. |
| Uniqueness | Must not be repeated in the document. |

#### Syntax Rules

1. **The value must be unique** within the document.
2. **Must not contain whitespace**.
3. **Should be meaningful** and stable.

#### Constraints and Limitations

- **Duplicate IDs are non-conforming** and break fragment links and scripting.
- **IDs are global**; they should not be used as a substitute for `class` when reuse is needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fragment Link and Styling with ID

**Setup Guide**: Save as `id.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ID Example</title>
  <style>
    #main-heading { color: navy; }
  </style>
</head>
<body>
  <a href="#main-heading">Jump to heading</a>
  <h1 id="main-heading">Main Heading</h1>

  <script>
    const heading = document.getElementById('main-heading');
    console.log(heading.textContent);
  </script>
</body>
</html>
```

**Expected output**: The link jumps to the heading; the heading is navy; the console logs its text.

**Why this output occurs**: The `id` provides a fragment target, a CSS selector hook, and a JavaScript lookup key.

---

### Real-World Cases

1. **Skip links**: `href="#main"` targeting `<main id="main">`.
2. **JavaScript components**: `document.getElementById('app')` for mounting.
3. **CSS specificity**: Using IDs for unique page sections.

---

### References

- MDN Web Docs — `id` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/id

---

## 13. `inert` — The Interaction and Accessibility Removal

### Definitions

**Core Definition**
The `inert` attribute is a boolean attribute that makes the browser disregard user input events for the element. It is useful when click events are present.

**Technical Definition**
`inert` is a boolean global attribute. When present, the element and its subtree are removed from the tab order, from the accessibility tree, and from find-in-page results. It also prevents click events from being dispatched to the element.

**Beginner-Friendly Explanation**
`inert` makes an element and its children completely non-interactive. They cannot be clicked, focused, or found by screen readers. It is like putting a glass pane over that part of the page.

---

### Purposes

- **To disable interaction** with a subtree without removing it from the DOM.
- **To remove content from the accessibility tree** while keeping it visible.
- **To prevent focus and clicks** on inactive regions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div inert>
  <button>Cannot click</button>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `inert` | Boolean attribute; presence makes the subtree inert. |

#### Syntax Rules

1. **Boolean attribute**: presence means true.
2. **Affects the entire subtree**.
3. **Removes elements from tab order and accessibility tree**.

#### Constraints and Limitations

- **`inert` does not hide the element visually**; it remains visible but non-interactive.
- **Support is broad but not universal** in very old browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inert Inactive Panel

**Setup Guide**: Save as `inert.html`. Try clicking and tabbing.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inert Example</title>
</head>
<body>
  <div inert>
    <p>This panel is inert.</p>
    <button type="button">Cannot click me</button>
  </div>

  <button type="button">This button works</button>
</body>
</html>
```

**Expected output**: The inert panel is visible, but its button cannot be clicked or focused, and screen readers skip it.

**Why this output occurs**: `inert` removes the subtree from interaction and the accessibility tree while keeping it rendered.

---

### Real-World Cases

1. **Modal backgrounds**: Making the page behind a modal inert.
2. **Disabled sections**: Preventing interaction with unavailable features.
3. **Off-screen content**: Removing hidden panels from tab order.

---

### References

- MDN Web Docs — `inert` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert

---

## 14. `inputmode` — The Virtual Keyboard Type Hint

### Definitions

**Core Definition**
The `inputmode` attribute provides a hint to browsers about the type of virtual keyboard configuration to use when editing this element or its contents.

**Technical Definition**
`inputmode` is an enumerated global attribute with values including `none`, `text`, `tel`, `url`, `email`, `numeric`, `decimal`, and `search`. It is primarily used on `<input>` elements but is usable on any element in `contenteditable` mode.

**Beginner-Friendly Explanation**
`inputmode` tells mobile browsers which keyboard layout to show. For example, `inputmode="numeric"` brings up a number pad instead of a full keyboard.

---

### Purposes

- **To request a specific virtual keyboard type**.
- **To improve mobile data entry** by showing the most relevant keys.
- **To reduce typing errors** on numeric or email fields.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<input type="text" inputmode="numeric" name="zip">
```

**Component breakdown**

| Value | Keyboard type |
|---|---|
| `none` | No virtual keyboard. |
| `text` | Standard text keyboard. |
| `tel` | Telephone keypad. |
| `url` | URL-optimised keyboard. |
| `email` | Email-optimised keyboard. |
| `numeric` | Numeric keypad. |
| `decimal` | Decimal numeric keypad. |
| `search` | Search-optimised keyboard. |

#### Syntax Rules

1. **Values are enumerated keywords**.
2. **Primarily affects mobile virtual keyboards**.
3. **Can be used on `contenteditable` elements**.

#### Constraints and Limitations

- **Support varies by browser and keyboard**.
- **It is a hint**, not a guarantee.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Numeric Input on Mobile

**Setup Guide**: Save as `inputmode.html`. Test on a mobile device.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inputmode Example</title>
</head>
<body>
  <label for="zip">ZIP code:</label>
  <input type="text" id="zip" name="zip" inputmode="numeric">
</body>
</html>
```

**Expected output**: On a mobile device, focusing the field shows a numeric keypad.

**Why this output occurs**: `inputmode="numeric"` hints to the browser to display a numeric keyboard.

---

### Real-World Cases

1. **ZIP and phone fields**: Numeric keypad.
2. **Email fields**: Email-optimised keyboard.
3. **Search bars**: Search key and layout.

---

### References

- MDN Web Docs — `inputmode` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inputmode

---

## 15. `is` — The Customised Built-In Element Reference

### Definitions

**Core Definition**
The `is` attribute allows you to specify that a standard HTML element should behave like a registered customised built-in element.

**Technical Definition**
`is` is a global attribute whose value must be the name of a registered customised built-in element. It is used with built-in tags such as `<button is="my-button">`. The custom element class must extend the corresponding built-in interface (e.g. `HTMLButtonElement`) and be registered with `{ extends: 'button' }`.

**Beginner-Friendly Explanation**
`is` lets a standard HTML tag (like `<button>`) use the behaviour of a custom element. It is how you upgrade built-in elements with custom functionality.

---

### Purposes

- **To upgrade a built-in element** with custom element behaviour.
- **To extend native elements** without inventing new tags.
- **To preserve native semantics** while adding custom functionality.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button is="my-button">Click me</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `is="my-button"` | The name of a registered customised built-in element. |
| Built-in tag | Must match the interface the custom class extends. |

#### Syntax Rules

1. **The value must be a registered custom element name**.
2. **The built-in tag must match the extended interface**.
3. **Used only with customised built-in elements**, not autonomous elements.

#### Constraints and Limitations

- **Browser support for customised built-ins varies**; Safari historically had limited support.
- **The custom class must extend the correct built-in interface**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Customised Built-In Button

**Setup Guide**: Save as `is.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Is Example</title>
</head>
<body>
  <button is="my-button">Click me</button>

  <script>
    class MyButton extends HTMLButtonElement {
      constructor() {
        super();
        this.addEventListener('click', () => alert('Custom!'));
      }
    }
    customElements.define('my-button', MyButton, { extends: 'button' });
  </script>
</body>
</html>
```

**Expected output**: The button behaves like a native button but shows a custom alert when clicked.

**Why this output occurs**: The `is` attribute associates the `<button>` with the `my-button` custom element, whose class extends `HTMLButtonElement`.

---

### Real-World Cases

1. **Enhanced buttons**: Adding ripple effects or analytics to native buttons.
2. **Custom inputs**: Extending `<input>` with validation or formatting behaviour.
3. **Progressive enhancement**: Upgrading native elements without changing the HTML tag.

---

### References

- MDN Web Docs — `is` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/is
- MDN Web Docs — Using custom elements — https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements

---

## 16. `itemid` — The Microdata Item Identifier

### Definitions

**Core Definition**
The `itemid` attribute is the unique, global identifier of a microdata item. It must be specified on elements that have both `itemscope` and `itemtype` attributes.

**Technical Definition**
`itemid` is a global attribute used in the HTML Microdata vocabulary. Its value must be an absolute URL. It provides a globally unique identifier for the item defined by `itemscope` and `itemtype`.

**Beginner-Friendly Explanation**
`itemid` gives a microdata item a globally unique ID, so different pages can refer to the same item.

---

### Purposes

- **To provide a unique identifier** for a microdata item.
- **To enable cross-document references** to structured data items.
- **To support vocabulary-specific identity requirements**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemtype="https://schema.org/Person" itemid="https://example.com/#jane">
  <span itemprop="name">Jane Doe</span>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemid="URL"` | An absolute URL identifying the item. |
| Requires | `itemscope` and `itemtype`. |

#### Syntax Rules

1. **Must be an absolute URL**.
2. **Only allowed on elements with `itemscope` and `itemtype`**.
3. **Uniquely identifies the item** globally.

#### Constraints and Limitations

- **Only meaningful in microdata contexts**.
- **Not all vocabularies require or use `itemid`**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Person with a Global ID

**Setup Guide**: Save as `itemid.html`. Inspect with a microdata parser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemid Example</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Person" itemid="https://example.com/#jane">
    <span itemprop="name">Jane Doe</span>
    <span itemprop="jobTitle">Engineer</span>
  </div>
</body>
</html>
```

**Expected output**: A microdata parser recognises the item with the global identifier `https://example.com/#jane`.

**Why this output occurs**: `itemid` provides the unique global identifier for the Person item.

---

### Real-World Cases

1. **Cross-referencing entities**: Linking the same person or organisation across pages.
2. **Knowledge graph data**: Providing stable identifiers for structured data.
3. **Product catalogs**: Using canonical product URLs as `itemid`.

---

### References

- MDN Web Docs — `itemid` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemid

---

## 17. `itemprop` — The Microdata Property Name

### Definitions

**Core Definition**
The `itemprop` attribute is used to add properties to an item. Every HTML element may have an `itemprop` attribute specified, where an `itemprop` consists of a name-value pair.

**Technical Definition**
`itemprop` is a global attribute used in microdata. Its value is the property name (or a space-separated list of names). The property value is derived from the element’s content, or from its `content`, `href`, `src`, or `datetime` attribute, depending on the element type.

**Beginner-Friendly Explanation**
`itemprop` labels a piece of data as a specific property of a microdata item — for example, marking a span as the `name` of a Person.

---

### Purposes

- **To declare a property of a microdata item**.
- **To connect a value to its vocabulary term**.
- **To build structured data for search engines and parsers**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemtype="https://schema.org/Person">
  <span itemprop="name">Jane Doe</span>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemprop="name"` | The property name. |
| Value source | Element content or specific attributes. |

#### Syntax Rules

1. **The value is the property name** (or space-separated names).
2. **Must be inside an item defined by `itemscope`**.
3. **The property value is determined by the element type**.

#### Constraints and Limitations

- **Only meaningful within a microdata item**.
- **Property names must match the vocabulary used in `itemtype`**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Person Properties

**Setup Guide**: Save as `itemprop.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemprop Example</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Person">
    <span itemprop="name">Jane Doe</span>
    <span itemprop="jobTitle">Engineer</span>
    <a itemprop="url" href="https://example.com/jane">Profile</a>
  </div>
</body>
</html>
```

**Expected output**: A microdata parser extracts `name`, `jobTitle`, and `url` properties for the Person item.

**Why this output occurs**: Each `itemprop` associates the element’s value with a property name in the schema.org Person vocabulary.

---

### Real-World Cases

1. **SEO structured data**: Marking up products, reviews, and events.
2. **Rich results**: Enabling enhanced search-result display.
3. **Data extraction**: Allowing parsers to consume structured content.

---

### References

- MDN Web Docs — `itemprop` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemprop

---

## 18. `itemref` — The Microdata Property Association

### Definitions

**Core Definition**
The `itemref` attribute provides a list of element IDs (not itemids) with additional properties elsewhere in the document, allowing properties that are not descendants of an `itemscope` element to be associated with the item.

**Technical Definition**
`itemref` is a global attribute used in microdata. Its value is a space-separated list of IDs. The referenced elements’ `itemprop` descendants become properties of the item, even though they are not DOM descendants.

**Beginner-Friendly Explanation**
`itemref` lets you include properties that live elsewhere on the page in a microdata item. You reference their IDs.

---

### Purposes

- **To associate non-descendant properties** with a microdata item.
- **To keep structured data flexible** when properties are scattered.
- **To avoid forcing all properties into a single DOM subtree**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemtype="https://schema.org/Person" itemref="extra-info">
  <span itemprop="name">Jane Doe</span>
</div>
<div id="extra-info">
  <span itemprop="jobTitle">Engineer</span>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemref="extra-info"` | Space-separated IDs of elements containing additional properties. |
| Referenced elements | Must contain `itemprop` descendants. |

#### Syntax Rules

1. **Value is a space-separated list of IDs**.
2. **Referenced elements must exist in the document**.
3. **Properties inside referenced elements are added to the item**.

#### Constraints and Limitations

- **Referenced elements must not create circular references**.
- **Only `itemprop` descendants are incorporated**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Associating Properties from Elsewhere

**Setup Guide**: Save as `itemref.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemref Example</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Person" itemref="job-section">
    <span itemprop="name">Jane Doe</span>
  </div>

  <div id="job-section">
    <span itemprop="jobTitle">Engineer</span>
  </div>
</body>
</html>
```

**Expected output**: The Person item includes both `name` (descendant) and `jobTitle` (referenced via `itemref`).

**Why this output occurs**: `itemref` pulls in the `jobTitle` property from the referenced element.

---

### Real-World Cases

1. **Scattered structured data**: Properties distributed across a page layout.
2. **Reusable property blocks**: Sharing property markup between items.
3. **CMS-generated pages**: Associating metadata that is not nested.

---

### References

- MDN Web Docs — `itemref` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemref

---

## 19. `itemscope` — The Microdata Item Scope

### Definitions

**Core Definition**
The `itemscope` attribute is a boolean attribute that creates a microdata item and defines the scope of the `itemtype` associated with it.

**Technical Definition**
`itemscope` is a boolean global attribute. When present, it declares that the element’s descendants (and any elements referenced by `itemref`) form a group of name-value pairs — a microdata item. It is usually used with `itemtype`.

**Beginner-Friendly Explanation**
`itemscope` marks the start of a microdata item. Everything inside it (or referenced by `itemref`) belongs to that item.

---

### Purposes

- **To declare a microdata item**.
- **To scope the properties** that belong to the item.
- **To enable structured data parsing**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemtype="https://schema.org/Person">
  …
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemscope` | Boolean attribute; presence creates an item. |
| `itemtype` | Optional vocabulary URL. |

#### Syntax Rules

1. **Boolean attribute**: presence creates an item.
2. **Properties are collected from descendants and `itemref` targets**.
3. **Usually paired with `itemtype`**.

#### Constraints and Limitations

- **`itemscope` without `itemtype` is allowed** but less useful.
- **Nested items are possible** but must be carefully scoped.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Microdata Item

**Setup Guide**: Save as `itemscope.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemscope Example</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Book">
    <span itemprop="name">The Great Gatsby</span>
    <span itemprop="author">F. Scott Fitzgerald</span>
  </div>
</body>
</html>
```

**Expected output**: A microdata parser recognises a Book item with `name` and `author` properties.

**Why this output occurs**: `itemscope` creates the item; `itemprop` descendants provide its properties.

---

### Real-World Cases

1. **Structured data markup**: Products, events, recipes, and reviews.
2. **Schema.org integration**: Powering rich search results.
3. **Data extraction**: Allowing machines to read page content.

---

### References

- MDN Web Docs — `itemscope` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemscope

---

## 20. `itemtype` — The Microdata Vocabulary URL

### Definitions

**Core Definition**
The `itemtype` attribute specifies the URL of the vocabulary that will be used to define `itemprop`s (item properties) in the data structure.

**Technical Definition**
`itemtype` is a global attribute used in microdata. Its value must be an absolute URL that identifies a vocabulary (e.g. `https://schema.org/Person`). It is used with `itemscope` to specify the item’s type.

**Beginner-Friendly Explanation**
`itemtype` tells the parser which vocabulary to use for an item — for example, schema.org’s Person or Book vocabulary.

---

### Purposes

- **To declare the vocabulary** for a microdata item.
- **To define the meaning of `itemprop` names**.
- **To enable vocabulary-aware parsing**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemtype="https://schema.org/Person">
  …
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemtype="https://schema.org/Person"` | Absolute URL of the vocabulary. |
| Requires | `itemscope` on the same element. |

#### Syntax Rules

1. **Must be an absolute URL**.
2. **Requires `itemscope` on the same element**.
3. **Multiple types may be specified** in some vocabularies.

#### Constraints and Limitations

- **The vocabulary must define the property names used in `itemprop`**.
- **Not all vocabularies are equally supported by consumers**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Schema.org Book

**Setup Guide**: Save as `itemtype.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemtype Example</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Book">
    <span itemprop="name">The Great Gatsby</span>
    <span itemprop="author">F. Scott Fitzgerald</span>
  </div>
</body>
</html>
```

**Expected output**: A parser recognises a schema.org Book item with the correct properties.

**Why this output occurs**: `itemtype` declares the vocabulary, and `itemprop` names are interpreted within that vocabulary.

---

### Real-World Cases

1. **Rich results**: Schema.org Product, Recipe, and Event markup.
2. **SEO**: Helping search engines understand page content.
3. **Data interoperability**: Using standard vocabularies for structured data.

---

### References

- MDN Web Docs — `itemtype` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemtype

---

## 21. `lang` — The Content Language

### Definitions

**Core Definition**
The `lang` attribute helps define the language of an element: the language that non-editable elements are in, or the language that editable elements should be written in by the user. It should contain a valid BCP 47 language tag.

**Technical Definition**
`lang` is a global attribute whose value is a BCP 47 language tag (e.g. `en`, `fr`, `ar`). It is inherited by descendants unless overridden. The `xml:lang` attribute has priority in XML contexts. It is critical for screen-reader pronunciation and translation tools.

**Beginner-Friendly Explanation**
`lang` tells the browser which language the content is in. This helps screen readers pronounce words correctly and helps translation tools.

---

### Purposes

- **To declare the language of content**.
- **To enable correct pronunciation** by screen readers.
- **To support translation and language-specific styling**.
- **To satisfy WCAG language requirements**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<html lang="en">
<p lang="fr">Ce paragraphe est en français.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `lang="en"` | BCP 47 language tag. |
| Inheritance | Applies to descendants unless overridden. |

#### Syntax Rules

1. **Value must be a valid BCP 47 language tag**.
2. **Use on `<html>` for the document language**.
3. **Override on elements containing foreign-language text**.

#### Constraints and Limitations

- **Invalid language tags may be ignored**.
- **The attribute does not translate content**; it only declares the language.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Document Language and Override

**Setup Guide**: Save as `lang.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Lang Example</title>
</head>
<body>
  <p>This is English.</p>
  <p lang="fr">Ceci est français.</p>
</body>
</html>
```

**Expected output**: Screen readers use English pronunciation for the first paragraph and French pronunciation for the second.

**Why this output occurs**: `lang="en"` on `<html>` sets the default; `lang="fr"` on the second paragraph overrides it.

---

### Real-World Cases

1. **Multilingual websites**: Declaring the page language and overriding for foreign phrases.
2. **Accessibility compliance**: WCAG requires a valid default language.
3. **Translation tools**: Identifying content language for translation.

---

### References

- MDN Web Docs — `lang` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/lang

---

## 22. `nonce` — The CSP Cryptographic Nonce

### Definitions

**Core Definition**
The `nonce` attribute is a cryptographic nonce (“number used once”) that can be used by Content Security Policy to determine whether or not a given fetch will be allowed to proceed.

**Technical Definition**
`nonce` is a global attribute whose value is an opaque string generated per response. It is used with the `script-src` and `style-src` CSP directives to authorise inline scripts and styles. The server must include the same nonce in the CSP header and in the attribute value.

**Beginner-Friendly Explanation**
`nonce` is a one-time code that tells the browser “this script is allowed to run.” It is used with Content Security Policy to block malicious inline scripts.

---

### Purposes

- **To authorise inline scripts and styles** under CSP.
- **To prevent cross-site scripting (XSS)** by blocking unauthorised inline code.
- **To support strict CSP policies** without `'unsafe-inline'`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<script nonce="random-value">console.log('Allowed');</script>
```

**Component breakdown**

| Component | Description |
|---|---|
| `nonce="random-value"` | An opaque, unpredictable string. |
| CSP header | Must include `'nonce-random-value'`. |

#### Syntax Rules

1. **The nonce must be unique per response**.
2. **The same value must appear in the CSP header and the attribute**.
3. **The value should be cryptographically random**.

#### Constraints and Limitations

- **Requires server-side support** to generate and embed the nonce.
- **The nonce is visible in the DOM**; it is not a secret after the page loads.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: CSP Nonce for Inline Script

**Setup Guide**: Save as `nonce.html`. Serve with a CSP header.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nonce Example</title>
  <script nonce="abc123">
    console.log('This inline script is allowed by CSP.');
  </script>
</head>
<body>
  <p>Check the console.</p>
</body>
</html>
```

**Expected output**: The inline script executes because its `nonce` matches the CSP header’s allowed nonce.

**Why this output occurs**: The `nonce` attribute authorises the inline script under the Content Security Policy.

---

### Real-World Cases

1. **Single-page applications**: Authorising framework-generated inline scripts.
2. **Strict CSP deployments**: Avoiding `'unsafe-inline'`.
3. **Security-hardened sites**: Mitigating XSS risks.

---

### References

- MDN Web Docs — `nonce` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/nonce
- W3C — Content Security Policy Level 3 — https://www.w3.org/TR/CSP3/

---

## 23. `part` — The Shadow DOM Part Name

### Definitions

**Core Definition**
The `part` attribute is a space-separated list of the part names of the element. Part names allow CSS to select and style specific elements in a shadow tree via the `::part` pseudo-element.

**Technical Definition**
`part` is a global attribute used inside a shadow tree. Its value is a space-separated list of part names. External stylesheets can target these parts using `::part(name)`, providing a controlled styling API across the shadow boundary.

**Beginner-Friendly Explanation**
`part` gives a name to an element inside a shadow DOM so that outside CSS can style it. It is like an official “style hook” exposed by a component.

---

### Purposes

- **To expose internal elements for styling** from outside a shadow tree.
- **To provide a controlled styling API** for Web Components.
- **To allow theme customisation** without breaking encapsulation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- Inside shadow DOM -->
<button part="button primary">Click</button>
```

```css
/* Outside, styling the exposed part */
my-component::part(button) { background: blue; }
```

**Component breakdown**

| Component | Description |
|---|---|
| `part="button"` | Space-separated part names. |
| `::part(button)` | External selector targeting the part. |

#### Syntax Rules

1. **Values are space-separated part names**.
2. **Only elements inside a shadow tree are styleable via `::part()`**.
3. **Parts can be re-exported with `exportparts`**.

#### Constraints and Limitations

- **Only explicitly named parts are styleable**.
- **`::part()` cannot style pseudo-elements of the part**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Exposing a Button Part

**Setup Guide**: Save as `part.html`. Full shadow DOM setup required.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Part Example</title>
  <style>
    my-button::part(button) { background: navy; color: white; }
  </style>
</head>
<body>
  <my-button></my-button>

  <script>
    class MyButton extends HTMLElement {
      constructor() {
        super();
        this.attachShadow({ mode: 'open' }).innerHTML =
          '<button part="button">Click</button>';
      }
    }
    customElements.define('my-button', MyButton);
  </script>
</body>
</html>
```

**Expected output**: The internal button is navy with white text, styled from outside the shadow tree.

**Why this output occurs**: The `part="button"` exposes the button for external styling via `::part(button)`.

---

### Real-World Cases

1. **Design system theming**: Exposing parts for consumer branding.
2. **Component customisation**: Allowing limited styling without full encapsulation breaks.
3. **Web Components libraries**: Standardising styleable parts.

---

### References

- MDN Web Docs — `part` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/part

---

## 24. `popover` — The Popover Element Designation

### Definitions

**Core Definition**
The `popover` attribute is used to designate an element as a popover element. Popover elements are hidden via `display: none` until opened via an invoking/control element or a `HTMLElement.showPopover()` call.

**Technical Definition**
`popover` is an enumerated global attribute with values `auto`, `manual`, and `hint`. It integrates with the Popover API. Popovers are rendered in the top layer and can be dismissed via light-dismiss or explicit controls.

**Beginner-Friendly Explanation**
`popover` turns an element into a popup that is hidden until triggered. It is a native way to create tooltips, menus, and dialogs.

---

### Purposes

- **To create native popovers** without custom JavaScript.
- **To render content in the top layer** above other content.
- **To support light-dismiss behaviour**.
- **To build menus, tooltips, and non-modal dialogs**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button popovertarget="my-popover">Open</button>
<div id="my-popover" popover>Popover content</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `popover` | Designates the element as a popover. |
| `popovertarget` | On a button, specifies the popover to toggle. |
| Values | `auto`, `manual`, `hint`. |

#### Syntax Rules

1. **Popovers are hidden by default**.
2. **`popovertarget` on a `<button>` opens/closes the popover**.
3. **`auto` popovers support light-dismiss**.

#### Constraints and Limitations

- **Browser support is modern** (Chrome 114+, Safari 17+, Firefox 125+).
- **Popovers are not modal** unless combined with other techniques.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Popover

**Setup Guide**: Save as `popover.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Popover Example</title>
</head>
<body>
  <button popovertarget="pop">Toggle Popover</button>
  <div id="pop" popover>This is a popover!</div>
</body>
</html>
```

**Expected output**: Clicking the button shows the popover near the button; clicking outside dismisses it.

**Why this output occurs**: `popover` makes the div a popover, and `popovertarget` connects the button to it.

---

### Real-World Cases

1. **Tooltips and hint bubbles**: Lightweight contextual information.
2. **Dropdown menus**: Native popover menus.
3. **Non-modal dialogs**: Popovers that don’t block the page.

---

### References

- MDN Web Docs — `popover` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/popover
- MDN Web Docs — Popover API — https://developer.mozilla.org/en-US/docs/Web/API/Popover_API

---

## 25. `role` — The ARIA Role

### Definitions

**Core Definition**
The `role` attribute defines the semantic meaning of content, allowing screen readers and other tools to present and support interaction with an object in a way that is consistent with user expectations.

**Technical Definition**
`role` is a global attribute defined by the ARIA specification. Its value is an ARIA role name (e.g. `button`, `navigation`, `dialog`). It overrides the implicit role of an element. Best practice is to prefer native semantic elements over `role` when possible.

**Beginner-Friendly Explanation**
`role` tells assistive technologies what an element is — for example, that a `<div>` should be treated as a button or a navigation region.

---

### Purposes

- **To define the semantic role** of an element for assistive technologies.
- **To provide missing semantics** when native HTML elements do not suffice.
- **To support custom widgets** with appropriate ARIA roles.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div role="button" tabindex="0">Click me</div>
<nav role="navigation">…</nav>
```

**Component breakdown**

| Component | Description |
|---|---|
| `role="button"` | ARIA role name. |
| Implicit role | Native elements have implicit roles; `role` overrides them. |

#### Syntax Rules

1. **Values must be valid ARIA role names**.
2. **The first role in a space-separated list is used**.
3. **Prefer native elements**; use `role` only when necessary.

#### Constraints and Limitations

- **Incorrect roles can harm accessibility** by misrepresenting content.
- **`role` does not add behaviour**; keyboard and event handling must still be implemented.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Button with ARIA Role

**Setup Guide**: Save as `role.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Role Example</title>
</head>
<body>
  <div role="button" tabindex="0" onclick="alert('Clicked!')"
       onkeydown="if(event.key==='Enter'||event.key===' ') alert('Clicked!')">
    Custom Button
  </div>
</body>
</html>
```

**Expected output**: The div is announced as a button, is focusable, and responds to click and keyboard activation.

**Why this output occurs**: `role="button"` provides the semantic role, and `tabindex="0"` makes it focusable. The event handlers provide behaviour.

---

### Real-World Cases

1. **Custom widgets**: Tabs, accordions, and sliders with ARIA roles.
2. **Landmark regions**: `role="navigation"`, `role="main"`, `role="complementary"`.
3. **Live regions**: `role="alert"`, `role="status"`.

---

### References

- MDN Web Docs — ARIA roles — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles
- MDN Web Docs — `role` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/role

---

## 26. `slot` — The Shadow DOM Slot Assignment

### Definitions

**Core Definition**
The `slot` attribute assigns a slot in a shadow DOM shadow tree to an element: an element with a `slot` attribute is assigned to the slot created by the `<slot>` element whose `name` attribute’s value matches that `slot` attribute’s value.

**Technical Definition**
`slot` is a global attribute used in light DOM to assign an element to a named slot in a shadow tree. It is a key part of Web Components composition, enabling content projection.

**Beginner-Friendly Explanation**
`slot` tells the browser which “hole” in a custom component an element should fill. It is how you plug your own content into a component’s shadow DOM.

---

### Purposes

- **To assign content to a named slot** in a shadow tree.
- **To enable content projection** in Web Components.
- **To support flexible component composition**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- Light DOM -->
<h2 slot="title">My Title</h2>

<!-- Shadow DOM -->
<slot name="title">Default title</slot>
```

**Component breakdown**

| Component | Description |
|---|---|
| `slot="title"` | The name of the slot to fill. |
| `<slot name="title">` | The insertion point in the shadow tree. |

#### Syntax Rules

1. **The `slot` value must match a `<slot name>`**.
2. **Unmatched content goes to the default (unnamed) slot**.
3. **Fallback content in the slot is shown when no assigned nodes exist**.

#### Constraints and Limitations

- **Slots only work within Shadow DOM**.
- **Slot names should be unique per shadow root**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Named Slot Projection

**Setup Guide**: Save as `slot-global.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Slot Example</title>
</head>
<body>
  <my-card>
    <h2 slot="title">Custom Title</h2>
    <p>Default slot content.</p>
  </my-card>

  <script>
    class MyCard extends HTMLElement {
      constructor() {
        super();
        this.attachShadow({ mode: 'open' }).innerHTML =
          '<div><slot name="title">Default</slot><slot></slot></div>';
      }
    }
    customElements.define('my-card', MyCard);
  </script>
</body>
</html>
```

**Expected output**: The card displays “Custom Title” in the title slot and “Default slot content.” in the default slot.

**Why this output occurs**: The `slot="title"` attribute assigns the heading to the named slot; the paragraph falls into the default slot.

---

### Real-World Cases

1. **Web Components composition**: Cards, dialogs, and layout components.
2. **Design systems**: Allowing consumers to inject content into predefined slots.
3. **Custom elements**: Any component that wraps consumer markup.

---

### References

- MDN Web Docs — `slot` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/slot

---

## 27. `spellcheck` — The Spell-Checking Control

### Definitions

**Core Definition**
The `spellcheck` attribute is an enumerated attribute that defines whether the element may be checked for spelling errors. It may have the values `true` (or empty string) or `false`.

**Technical Definition**
`spellcheck` is an enumerated global attribute. It hints to the browser whether spell-checking should apply to editable content. It is inherited by descendants.

**Beginner-Friendly Explanation**
`spellcheck` tells the browser whether to underline misspelled words in an editable field.

---

### Purposes

- **To enable or disable spell-checking** in editable content.
- **To prevent spell-check underlines** in code or username fields.
- **To improve the editing experience** for prose.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<textarea spellcheck="true"></textarea>
<input type="text" spellcheck="false" name="username">
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` / empty | Enable spell-checking. |
| `false` | Disable spell-checking. |

#### Syntax Rules

1. **Values are `true` or `false`** (empty string also means true).
2. **Inherited by descendants**.
3. **Applies to editable content**.

#### Constraints and Limitations

- **Behaviour depends on browser and OS settings**.
- **It is a hint, not a guarantee**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling Spellcheck for a Code Field

**Setup Guide**: Save as `spellcheck.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Spellcheck Example</title>
</head>
<body>
  <label for="code">Code:</label>
  <input type="text" id="code" name="code" spellcheck="false">
</body>
</html>
```

**Expected output**: The input does not show spelling-error underlines.

**Why this output occurs**: `spellcheck="false"` disables the browser’s spell-checking for that element.

---

### Real-World Cases

1. **Code and username fields**: Disabling spell-check.
2. **Prose text areas**: Enabling spell-check by default.
3. **Multilingual content**: Disabling when typing in a language not supported by the browser.

---

### References

- MDN Web Docs — `spellcheck` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/spellcheck

---

## 28. `style` — The Inline CSS Declarations

### Definitions

**Core Definition**
The `style` attribute contains CSS styling declarations to be applied to the element. It is recommended for styles to be defined in separate files; this attribute is mainly for quick styling and testing.

**Technical Definition**
`style` is a global attribute whose value is a semicolon-separated list of CSS declarations. It is exposed in the DOM as `element.style` (a `CSSStyleDeclaration` object) and has high specificity, overriding most stylesheet rules.

**Beginner-Friendly Explanation**
`style` lets you write CSS directly on an element. It is useful for quick changes but should be avoided for maintainable production code.

---

### Purposes

- **To apply quick inline styling**.
- **To test styles** during development.
- **To set dynamic styles** from JavaScript.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div style="color: red; font-size: 1.2em;">Styled text</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `style="property: value;"` | CSS declarations separated by semicolons. |
| Specificity | Inline styles have high specificity. |

#### Syntax Rules

1. **Values are CSS declarations**.
2. **Multiple declarations are separated by semicolons**.
3. **Inline styles override most stylesheet rules**.

#### Constraints and Limitations

- **Poor maintainability** compared with external stylesheets.
- **Cannot be overridden without `!important`** in some cases.
- **CSP `style-src` may restrict inline styles**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline Styling vs External CSS

**Setup Guide**: Save as `style.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Style Example</title>
</head>
<body>
  <p style="color: navy; font-weight: bold;">Inline styled text.</p>
  <p class="styled">Externally styled text.</p>

  <style>
    .styled { color: green; }
  </style>
</body>
</html>
```

**Expected output**: The first paragraph is navy and bold; the second is green.

**Why this output occurs**: The `style` attribute applies inline declarations; the `<style>` element applies a class rule.

---

### Real-World Cases

1. **Quick prototypes**: Inline styles for rapid experimentation.
2. **Dynamic styling**: JavaScript setting `element.style`.
3. **Email templates**: Inline styles for email-client compatibility.

---

### References

- MDN Web Docs — `style` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/style

---

## 29. `tabindex` — The Focusability and Tab Order Control

### Definitions

**Core Definition**
The `tabindex` attribute is an integer attribute indicating if the element can take input focus (is focusable), if it should participate in sequential keyboard navigation, and if so, at what position.

**Technical Definition**
`tabindex` is a global attribute that accepts an integer. Negative values make the element focusable but not reachable via sequential navigation. `0` makes it focusable in the natural document order. Positive values define a custom tab order, which is discouraged.

**Beginner-Friendly Explanation**
`tabindex` controls whether an element can be focused with the Tab key and in what order. `tabindex="0"` adds it to the natural tab order; `tabindex="-1"` makes it focusable only by script.

---

### Purposes

- **To make non-native elements focusable** (`tabindex="0"`).
- **To remove elements from the tab order** (`tabindex="-1"`).
- **To create custom tab orders** (positive values, discouraged).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div tabindex="0">Focusable div</div>
<div tabindex="-1">Programmatically focusable</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| Negative | Focusable via script, not via Tab. |
| `0` | Focusable in natural document order. |
| Positive | Custom tab order (discouraged). |

#### Syntax Rules

1. **`tabindex="0"` adds an element to the natural tab order**.
2. **`tabindex="-1"` allows programmatic focus but removes from Tab order**.
3. **Positive values should be avoided**; they disrupt natural order.

#### Constraints and Limitations

- **Positive `tabindex` values are widely discouraged** for accessibility.
- **Native interactive elements already have implicit tabindex**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Focusable Custom Widget

**Setup Guide**: Save as `tabindex.html`. Try tabbing through the page.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Tabindex Example</title>
</head>
<body>
  <button>Native button</button>
  <div tabindex="0">Focusable div</div>
  <div tabindex="-1">Script-focusable only</div>
</body>
</html>
```

**Expected output**: The native button and the `tabindex="0"` div are reachable via Tab. The `tabindex="-1"` div is not reachable via Tab but can be focused with `element.focus()`.

**Why this output occurs**: `tabindex="0"` adds the div to the natural tab order; `tabindex="-1"` makes it focusable only programmatically.

---

### Real-World Cases

1. **Custom widgets**: Making a `<div role="button">` focusable.
2. **Skip links**: Focusing the main content region.
3. **Modal focus trapping**: Managing focus within dialogs.

---

### References

- MDN Web Docs — `tabindex` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/tabindex

---

## 30. `title` — The Advisory Information

### Definitions

**Core Definition**
The `title` attribute contains a text representing advisory information related to the element it belongs to. Such information can typically, but not necessarily, be presented to the user as a tooltip.

**Technical Definition**
`title` is a global attribute whose value is free-form text. Browsers typically display it as a tooltip on hover. It is also used as the accessible name fallback for some elements when no other name is provided.

**Beginner-Friendly Explanation**
`title` shows a tooltip when you hover over an element. It can provide extra explanation or context.

---

### Purposes

- **To provide advisory information** as a tooltip.
- **To supplement link text** with additional context.
- **To provide fallback accessible names** in limited cases.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<abbr title="HyperText Markup Language">HTML</abbr>
```

**Component breakdown**

| Component | Description |
|---|---|
| `title="text"` | Advisory text. |
| Display | Typically a tooltip on hover. |

#### Syntax Rules

1. **Value is free-form text**.
2. **Tooltip display depends on the browser**.
3. **Not a substitute for visible labels** in forms.

#### Constraints and Limitations

- **Inaccessible on touch devices** (no hover).
- **Not announced consistently by screen readers**.
- **Do not rely on `title` for critical information**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Tooltip for an Abbreviation

**Setup Guide**: Save as `title.html`. Hover over the abbreviation.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title Example</title>
</head>
<body>
  <p>
    <abbr title="HyperText Markup Language">HTML</abbr> is the backbone of the web.
  </p>
</body>
</html>
```

**Expected output**: Hovering over “HTML” shows the tooltip “HyperText Markup Language”.

**Why this output occurs**: The `title` attribute provides the advisory text displayed as a tooltip.

---

### Real-World Cases

1. **Abbreviation expansions**: `title` on `<abbr>`.
2. **Supplementary link context**: Adding detail to ambiguous link text.
3. **Icon tooltips**: Brief descriptions for icon buttons.

---

### References

- MDN Web Docs — `title` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/title

---

## 31. `translate` — The Translation Control

### Definitions

**Core Definition**
The `translate` attribute is an enumerated attribute used to specify whether an element’s attribute values and the values of its text node children are to be translated when the page is localized, or whether to leave them unchanged.

**Technical Definition**
`translate` is an enumerated global attribute with values `yes` (or empty string) and `no`. It hints to translation tools whether content should be translated.

**Beginner-Friendly Explanation**
`translate` tells translation tools whether to translate an element’s content. Use `translate="no"` for brand names or code that should stay the same.

---

### Purposes

- **To prevent translation of proper nouns, code, or brand names**.
- **To allow translation of content** by default.
- **To improve localisation quality**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<span translate="no">BrandName™</span>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `yes` / empty | Translate (default). |
| `no` | Do not translate. |

#### Syntax Rules

1. **Values are `yes` and `no`**.
2. **Inherited by descendants**.
3. **Used by translation tools and browser features**.

#### Constraints and Limitations

- **Support depends on translation tools**.
- **Not a guarantee**; tools may ignore it.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Protecting a Brand Name

**Setup Guide**: Save as `translate.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Translate Example</title>
</head>
<body>
  <p>
    Welcome to <span translate="no">AcmeCorp</span>. We build things.
  </p>
</body>
</html>
```

**Expected output**: Translation tools leave “AcmeCorp” unchanged while translating the rest of the paragraph.

**Why this output occurs**: `translate="no"` signals that the brand name should not be translated.

---

### Real-World Cases

1. **Brand names and trademarks**: Preventing translation.
2. **Code snippets**: Keeping code in its original form.
3. **Legal names**: Preserving official names across languages.

---

### References

- MDN Web Docs — `translate` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/translate

---

## 32. `virtualkeyboardpolicy` — The Virtual Keyboard Visibility Control

### Definitions

**Core Definition**
The `virtualkeyboardpolicy` attribute is an enumerated attribute used to control the on-screen virtual keyboard behaviour on devices such as tablets, mobile phones, or other devices where a hardware keyboard may not be available for elements whose content is editable.

**Technical Definition**
`virtualkeyboardpolicy` is an enumerated global attribute with values `auto` (or empty string) and `manual`. `auto` shows the virtual keyboard when the element is focused; `manual` decouples focus and tap from the virtual keyboard’s state.

**Beginner-Friendly Explanation**
`virtualkeyboardpolicy` controls whether the mobile keyboard appears automatically when you focus an editable element. Use `manual` to control it yourself.

---

### Purposes

- **To control virtual keyboard visibility** on focus.
- **To support custom on-screen keyboard experiences**.
- **To prevent the keyboard from covering content** in certain layouts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div contenteditable="true" virtualkeyboardpolicy="manual">Editable</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `auto` / empty | Keyboard shows on focus. |
| `manual` | Keyboard visibility is controlled by script. |

#### Syntax Rules

1. **Applies to editable elements** (`<input>`, `<textarea>`, `contenteditable`).
2. **Values are `auto` and `manual`**.
3. **`manual` allows programmatic control**.

#### Constraints and Limitations

- **Support varies**; Safari currently does not support it.
- **Behaviour depends on the virtual keyboard implementation**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Manual Keyboard Control

**Setup Guide**: Save as `virtualkeyboardpolicy.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Virtual Keyboard Policy</title>
</head>
<body>
  <div contenteditable="true" virtualkeyboardpolicy="manual">
    Editable with manual keyboard control.
  </div>
</body>
</html>
```

**Expected output**: On supported devices, focusing the element does not automatically show the virtual keyboard; script must trigger it.

**Why this output occurs**: `virtualkeyboardpolicy="manual"` decouples focus from keyboard visibility.

---

### Real-World Cases

1. **Custom editors**: Controlling when the keyboard appears.
2. **Split-screen layouts**: Preventing the keyboard from obscuring content.
3. **Kiosk applications**: Managing keyboard visibility programmatically.

---

### References

- MDN Web Docs — `virtualkeyboardpolicy` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/virtualkeyboardpolicy

---

## Global Attribute Best Practices

### Definitions

**Core Definition**
Global attribute best practices are guidelines for using global attributes correctly, consistently, and securely across HTML documents.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG accessibility guidelines, and web platform security guidance. They encompass correct attribute selection, avoiding redundancy, ensuring accessibility, and applying security-related attributes appropriately.

**Beginner-Friendly Explanation**
Best practices help you use global attributes in ways that keep your HTML valid, accessible, and secure.

---

### Purposes

- **To ensure validity and consistency** across documents.
- **To maximise accessibility** with `lang`, `role`, `tabindex`, and proper use of `hidden`/`inert`.
- **To improve security** with `nonce` and proper CSP integration.
- **To enhance maintainability** with disciplined use of `class`, `id`, and `style`.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `id` uniquely** | Never duplicate IDs; use `class` for reusable hooks. |
| **Prefer native semantics over `role`** | Use native elements when possible; add `role` only when necessary. |
| **Use `lang` on `<html>` and foreign-language spans** | Required for correct pronunciation and WCAG compliance. |
| **Use `hidden` or `inert` appropriately** | `hidden` removes from rendering; `inert` removes from interaction and accessibility. |
| **Avoid `accesskey`** | Conflicts with assistive-technology shortcuts. |
| **Quote all attribute values** | Consistent, safe, and readable. |
| **Use `nonce` with CSP** | Generate cryptographically random nonces per response. |
| **Prefer `spellcheck="false"` for code fields** | Prevents unwanted underlines. |
| **Use `translate="no"` for brand names and code** | Prevents inappropriate translation. |
| **Do not use positive `tabindex` values** | Disrupts natural focus order. |
| **Use `autofocus` sparingly** | Only one per document; can disrupt users. |

#### Syntax Rules

1. **Accessibility first**: `lang`, `role`, and `tabindex` directly affect users.
2. **Security first**: `nonce` is critical for CSP; never hard-code it.
3. **Consistency**: Quote attributes, use lowercase names, and avoid duplication.
4. **Progressive enhancement**: Use global attributes that enhance without breaking.

#### Constraints and Limitations

- Some global attributes (`accesskey`, positive `tabindex`) are widely discouraged due to accessibility conflicts.
- `title` is inaccessible on touch devices and should not be relied upon for critical information.
- `hidden` can be overridden by CSS; do not use it for security.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible and Secure Global Attribute Usage

**Setup Guide**: Save as `best-practice-global.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Global Attributes</title>

  <!-- CSP nonce for inline script -->
  <script nonce="random-nonce">
    console.log('Authorised by CSP');
  </script>
</head>
<body>
  <!-- Unique ID for fragment navigation -->
  <main id="main-content">
    <h1>Global Attributes Done Right</h1>

    <!-- Class for styling -->
    <p class="highlight">This paragraph is styled with a class.</p>

    <!-- Lang override for foreign-language text -->
    <p lang="fr">Ce paragraphe est en français.</p>

    <!-- Inert inactive region -->
    <div inert>
      <button>Unavailable</button>
    </div>
  </main>
</body>
</html>
```

**Expected output**: The page is accessible, the inline script is authorised by the nonce, the French paragraph is pronounced correctly, and the inert button is non-interactive.

**Why this output occurs**: Each global attribute is used according to best practices: `id` is unique, `class` is reusable, `lang` is correctly overridden, `inert` removes interaction, and `nonce` secures the inline script.

---

#### Example 2: Avoiding Common Global Attribute Mistakes

**Setup Guide**: Save as `global-mistakes.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Global Attribute Mistakes</title>
</head>
<body>
  <!-- MISTAKE 1: duplicate ID -->
  <div id="dup"></div>
  <div id="dup"></div>

  <!-- MISTAKE 2: accesskey conflict -->
  <button accesskey="s">Save</button>
  <button accesskey="s">Send</button>

  <!-- MISTAKE 3: positive tabindex -->
  <a href="#" tabindex="1">First</a>
  <a href="#" tabindex="2">Second</a>

  <!-- MISTAKE 4: relying on title for critical info -->
  <span title="Important warning">⚠️</span>
</body>
</html>
```

**Expected output**: Duplicate IDs cause validation errors; accesskey conflicts cause unpredictable shortcuts; positive tabindex disrupts focus order; the title tooltip is inaccessible on touch devices.

**Why this output occurs**: These are common global attribute anti-patterns that harm accessibility, validity, and usability.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG compliance drives correct `lang`, `role`, and `tabindex` usage.
2. **Security-sensitive applications**: `nonce` is essential for strict CSP.
3. **Design systems**: Consistent `class` naming and unique `id` usage across components.

---

### References

- MDN Web Docs — Global attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes
- WHATWG — Global attributes — https://html.spec.whatwg.org/multipage/dom.html#global-attributes
- W3C — WCAG 2.2 — https://www.w3.org/WAI/standards-guidelines/wcag/
- W3C — Content Security Policy Level 3 — https://www.w3.org/TR/CSP3/

---

## Summary Table of Global Attributes

| Attribute | Type | Purpose |
|---|---|---|
| `accesskey` | Token list | Keyboard shortcut hint (discouraged). |
| `autocapitalize` | Enumerated | Automatic capitalisation control. |
| `autocorrect` | Enumerated | Automatic spelling correction control. |
| `autofocus` | Boolean | Initial focus on load or dialog display. |
| `class` | Token list | Styling and scripting hooks. |
| `contenteditable` | Enumerated | Editable content control. |
| `dir` | Enumerated | Text directionality. |
| `draggable` | Enumerated | Drag-and-drop enablement. |
| `enterkeyhint` | Enumerated | Virtual keyboard enter-key label hint. |
| `exportparts` | Comma-separated list | Shadow DOM part re-export. |
| `hidden` | Enumerated/Boolean | Hiding control. |
| `id` | String | Unique identifier. |
| `inert` | Boolean | Interaction and accessibility removal. |
| `inputmode` | Enumerated | Virtual keyboard type hint. |
| `is` | Custom element name | Customised built-in element reference. |
| `itemid` | URL | Microdata item identifier. |
| `itemprop` | String | Microdata property name. |
| `itemref` | ID list | Microdata property association. |
| `itemscope` | Boolean | Microdata item scope. |
| `itemtype` | URL | Microdata vocabulary URL. |
| `lang` | BCP 47 tag | Content language. |
| `nonce` | String | CSP cryptographic nonce. |
| `part` | Token list | Shadow DOM part name. |
| `popover` | Enumerated | Popover element designation. |
| `role` | ARIA role | Accessibility role. |
| `slot` | String | Shadow DOM slot assignment. |
| `spellcheck` | Enumerated | Spell-checking control. |
| `style` | CSS declarations | Inline CSS. |
| `tabindex` | Integer | Focusability and tab order. |
| `title` | String | Advisory information. |
| `translate` | Enumerated | Translation control. |
| `virtualkeyboardpolicy` | Enumerated | Virtual keyboard visibility control. |

---

## Final Notes

- **Global attributes are universal**: They work on every element, but not every attribute has an effect on every element.
- **Accessibility is paramount**: `lang`, `role`, `tabindex`, `hidden`, and `inert` directly determine whether content is usable by all users.
- **Security matters**: `nonce` is essential for strict Content Security Policy.
- **Modern Web Components**: `is`, `slot`, `part`, and `exportparts` enable advanced component composition.
- **Validate and test**: Use HTML validators and accessibility testing tools to catch misuse of global attributes.

---

This cheat sheet has covered all major global attributes — from `accesskey` to `virtualkeyboardpolicy` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate attribute usage against the WHATWG HTML Living Standard, WCAG accessibility guidelines, and Content Security Policy best practices.