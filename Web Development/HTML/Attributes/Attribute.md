# Attribute Fundamentals — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
HTML attributes are additional values placed inside an element's start tag that configure the element or adjust its behaviour. They consist of a name-value pair and provide a way to customise how an element appears, behaves, or relates to other content.

**Technical Definition**
In the WHATWG HTML Living Standard, attributes are expressed inside an element's start tag. Each attribute consists of an **attribute name** followed by zero or more ASCII whitespace characters, followed by a single U+003D EQUALS SIGN (`=`), followed by an **attribute value**. The attribute name must be lowercase in HTML syntax, and the value may be quoted (single or double) or, under strict conditions, unquoted. Attributes are parsed into the DOM as `Attr` nodes, and their values are exposed as strings. The specification distinguishes between **boolean attributes** (whose presence represents `true` and absence represents `false`), **enumerated attributes** (which accept a finite set of keywords), and **string-valued attributes** (which accept arbitrary text).

**Beginner-Friendly Explanation**
Attributes are the “settings” you add to an HTML tag. They tell the browser extra information about the element — like where a link should go (`href`), what an image shows (`alt`), or whether an input is required (`required`). They live inside the opening tag, after the element name, and come in name-value pairs.

---

### Key Characteristics

- **Name-value structure**: Each attribute has a name and (usually) a value, written as `name="value"`.
- **Placement**: Attributes appear only inside the **start tag** of an element; they are not separate elements.
- **Global vs. element-specific**: **Global attributes** work on every HTML element (e.g. `class`, `id`, `lang`), while **element-specific attributes** apply only to certain elements (e.g. `href` on `<a>`, `src` on `<img>`).
- **Boolean presence semantics**: Boolean attributes such as `disabled`, `checked`, and `required` are true when present, regardless of their value — even `disabled="false"` still means disabled.
- **Enumerated keyword constraints**: Enumerated attributes such as `dir` and `contenteditable` accept only a predefined set of keywords; values outside that set may be ignored or defaulted.
- **Quoting is recommended**: While unquoted values are permitted under strict conditions, quoting all attribute values is a widely recommended best practice.
- **Case sensitivity**: Attribute names are case-insensitive in HTML (but lowercase is convention); attribute values are generally case-sensitive unless defined otherwise (e.g. enumerated keywords are ASCII case-insensitive).

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<html>`, `<head>`, `<body>`, and elements.
2. **The concept of start tags and end tags**: understanding how elements are written in HTML.
3. **Awareness of the DOM**: attributes become properties on DOM element objects.
4. **Basic CSS and JavaScript familiarity**: `class` and `id` attributes are common styling and scripting hooks.
5. **An HTML validator**: for checking conformance of attribute usage.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | `class`, `id`, and `style` attributes are primary styling hooks. |
| **JavaScript / DOM** | Attributes are accessible via `getAttribute()`, `setAttribute()`, and as DOM properties. |
| **Accessibility (ARIA)** | `role` and `aria-*` attributes convey semantic information to assistive technologies. |
| **Security** | `nonce`, `integrity`, and `crossorigin` attributes are critical for Content Security Policy and Subresource Integrity. |
| **Internationalisation** | `lang` and `dir` attributes declare language and text direction. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Attributes** — the general concept
2. **Global attributes** — attributes usable on all elements
3. **Element-specific attributes** — attributes tied to particular elements
4. **Boolean attributes** — presence-based true/false attributes
5. **Enumerated attributes** — keyword-constrained attributes
6. **Attribute values** — string, number, and other value types
7. **Attribute quoting** — single, double, and unquoted syntax
8. **Attribute best practices** — quality, accessibility, and security guidelines

---

## 1. Attributes — The General Concept

### Definitions

**Core Definition**
An attribute is a name-value pair inside an element's start tag that provides additional information about the element or modifies its behaviour.

**Technical Definition**
Attributes are part of the HTML syntax. They are written after the tag name in the start tag, separated by whitespace. Each attribute has an **attribute name** (which must be lowercase in HTML) and an **attribute value** (which may be omitted for boolean attributes). The HTML parser creates `Attr` nodes for each attribute, and the DOM exposes them through methods such as `getAttribute()` and `setAttribute()`. The specification defines several microsyntaxes for attribute values, including boolean, enumerated, string, and numeric types.

**Beginner-Friendly Explanation**
An attribute is an extra piece of information you attach to an HTML tag. For example, in `<a href="https://example.com">`, `href` is the attribute name and `https://example.com` is the attribute value.

---

### Purposes

- **To configure element behaviour**: such as specifying a link destination (`href`) or an image source (`src`).
- **To provide metadata**: such as `alt` text for images or `title` for tooltips.
- **To enable accessibility**: such as `lang` for pronunciation and `role` for semantic meaning.
- **To support styling and scripting**: such as `class` and `id`.
- **To control form behaviour**: such as `required`, `disabled`, and `type`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<element attribute-name="attribute-value">
```

**Component breakdown**

| Component | Description |
|---|---|
| `attribute-name` | The name of the attribute; must be lowercase in HTML. |
| `=` | The equals sign separating name from value (omitted for boolean attributes). |
| `"attribute-value"` | The value, preferably wrapped in double quotes. |
| Whitespace | Attributes must be separated from the tag name and from each other by whitespace. |

#### Syntax Rules

1. **Attributes appear only in the start tag**.
2. **Attribute names must not contain whitespace, `"`, `'`, `>`, `/`, or `=`**.
3. **Attribute names are unique per element**; duplicate attribute names are non-conforming (the first occurrence is used).
4. **Attribute values should be quoted**; unquoted values are permitted only under strict conditions.
5. **Boolean attributes may omit the value entirely**.

#### Constraints and Limitations

- Duplicate attributes produce a parse error; only the first is retained.
- Attribute names are case-insensitive but conventionally lowercase.
- Some attributes are **enumerated** and accept only specific keywords; others are **boolean** and ignore their values.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Attributes in Practice

**Setup Guide**: Save as `attributes-basic.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Basic Attributes</title>
</head>
<body>
  <!-- href is an attribute of <a>; class is a global attribute -->
  <a href="https://example.com" class="external-link" target="_blank" rel="noopener">
    Visit Example
  </a>

  <!-- src and alt are attributes of <img> -->
  <img src="photo.jpg" alt="A mountain landscape" width="400" height="300">
</body>
</html>
```

**Expected output**: A link to `https://example.com` that opens in a new tab, and an image with descriptive alternative text.

**Why this output occurs**: The `href` attribute provides the link destination, `target="_blank"` opens it in a new browsing context, `rel="noopener"` prevents access to `window.opener`, and `src`/`alt` provide the image resource and its textual alternative.

---

#### Example 2: Duplicate Attribute Parsing

**Setup Guide**: Save as `duplicate-attr.html` and inspect the DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Duplicate Attribute</title>
</head>
<body>
  <!-- Duplicate class attributes: only the first is used -->
  <p class="first" class="second">Which class applies?</p>
</body>
</html>
```

**Expected output**: The paragraph is styled by `.first`; the second `class` attribute is ignored, and a validator reports a duplicate-attribute error.

**Why this output occurs**: The HTML parser retains only the first occurrence of a duplicate attribute. The second is discarded as a parse error.

---

### Real-World Cases

1. **Links**: `href`, `target`, and `rel` configure hyperlinks.
2. **Images**: `src`, `alt`, `width`, and `height` configure images.
3. **Forms**: `type`, `name`, `required`, and `placeholder` configure form controls.

---

### References

- MDN Web Docs — HTML attribute reference — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes
- WHATWG — Attributes — https://html.spec.whatwg.org/multipage/dom.html#attributes

---

## 2. Global Attributes

### Definitions

**Core Definition**
Global attributes are attributes that are common to all HTML elements and can be used on every element, though they may have no effect on some.

**Technical Definition**
Global attributes are defined in the WHATWG standard as attributes that may be specified on all HTML elements, including non-standard elements. They include `accesskey`, `autocapitalize`, `autofocus`, `class`, `contenteditable`, `dir`, `draggable`, `enterkeyhint`, `hidden`, `id`, `inert`, `inputmode`, `is`, `itemid`, `itemprop`, `itemref`, `itemscope`, `itemtype`, `lang`, `nonce`, `part`, `popover`, `slot`, `spellcheck`, `style`, `tabindex`, `title`, `translate`, and the event handler attributes. The `role` attribute and `aria-*` attributes are also global, though they are defined by ARIA rather than HTML.

**Beginner-Friendly Explanation**
Global attributes are the “universal settings” you can use on any HTML tag. For example, `class`, `id`, `lang`, and `hidden` work on `<div>`, `<p>`, `<img>`, and almost every other element.

---

### Purposes

- **To identify elements uniquely**: `id`.
- **To classify elements for styling and scripting**: `class`.
- **To declare language and direction**: `lang` and `dir`.
- **To hide content**: `hidden` and `inert`.
- **To make elements focusable**: `tabindex` and `autofocus`.
- **To provide tooltips**: `title`.
- **To support accessibility**: `role` and `aria-*` attributes.
- **To enable drag-and-drop**: `draggable`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<any-element class="my-class" id="unique-id" lang="en" hidden>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `class` | Space-separated list of classes for CSS/JavaScript. |
| `id` | Unique identifier within the document. |
| `lang` | Language of the element’s content (BCP 47). |
| `hidden` | Boolean; hides the element. |

#### Syntax Rules

1. **Global attributes may be used on any HTML element**.
2. **`id` values must be unique** within the document.
3. **`class` values are space-separated**; multiple classes are allowed.
4. **`hidden` is a boolean attribute**; its presence hides the element.
5. **`lang` must be a valid BCP 47 language tag** for reliable pronunciation.

#### Constraints and Limitations

- `id` duplicates are non-conforming and can break scripting and fragment links.
- `hidden` removes the element from rendering but does not remove it from the DOM.
- `accesskey` is discouraged because shortcuts conflict with screen-reader and browser shortcuts.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Using Global Attributes for Styling and Language

**Setup Guide**: Save as `global-attrs.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Global Attributes</title>
  <style>
    .highlight { background: yellow; }
    #main-heading { color: navy; }
  </style>
</head>
<body>
  <h1 id="main-heading" class="highlight">Global Attributes</h1>

  <!-- lang overrides the document language for this element -->
  <p lang="fr">Ce paragraphe est en français.</p>
</body>
</html>
```

**Expected output**: The heading is navy and highlighted yellow; the French paragraph is announced with French pronunciation by screen readers.

**Why this output occurs**: `id` provides a unique hook for CSS (`#main-heading`), `class` provides a reusable styling hook (`.highlight`), and `lang="fr"` overrides the document’s default language for that element.

---

#### Example 2: `hidden` vs `inert`

**Setup Guide**: Save as `hidden-inert.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hidden and Inert</title>
</head>
<body>
  <!-- hidden: not rendered, but still in the DOM -->
  <p hidden>This paragraph is hidden.</p>

  <!-- inert: rendered but non-interactive and hidden from assistive tech -->
  <div inert>
    <button type="button">Cannot click me</button>
  </div>
</body>
</html>
```

**Expected output**: The first paragraph is not displayed at all. The second block is displayed but its button cannot be clicked or focused, and it is removed from the accessibility tree.

**Why this output occurs**: `hidden` removes the element from rendering. `inert` keeps the element visible but removes it from the tab order and the accessibility tree, preventing interaction.

---

### Real-World Cases

1. **Styling hooks**: `class` is used universally by CSS frameworks and utility libraries.
2. **Fragment navigation**: `id` targets for in-page links (`#section`).
3. **Language declaration**: `lang` on `<html>` and on foreign-language phrases.
4. **Accessibility**: `role` and `aria-*` attributes on custom widgets.

---

### References

- MDN Web Docs — Global attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes
- WHATWG — Global attributes — https://html.spec.whatwg.org/multipage/dom.html#global-attributes

---

## 3. Element-Specific Attributes

### Definitions

**Core Definition**
Element-specific attributes are attributes defined for particular HTML elements and are not valid or meaningful on other elements.

**Technical Definition**
Each element has a defined set of attributes in the HTML specification. For example, `href` is specific to `<a>`, `<area>`, and `<link>`; `src` is specific to `<img>`, `<script>`, `<iframe>`, `<video>`, `<audio>`, `<source>`, and `<track>`; `alt` is specific to `<img>`, `<area>`, and `<input type="image">`. Using an element-specific attribute on an element that does not define it is non-conforming and may have no effect.

**Beginner-Friendly Explanation**
Some attributes only make sense on certain tags. For example, `href` is for links, and `src` is for resources like images and scripts. Putting `href` on an `<img>` does nothing useful.

---

### Purposes

- **To configure element-specific behaviour**: such as link targets, media sources, and form submission details.
- **To provide element-specific metadata**: such as image alternative text.
- **To enable element-specific functionality**: such as `controls` on media elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="url" target="_blank">Link</a>
<img src="image.jpg" alt="Description">
<input type="email" name="email" required>
```

**Component breakdown**

| Element | Element-specific attributes |
|---|---|
| `<a>` | `href`, `target`, `rel`, `download`, `hreflang` |
| `<img>` | `src`, `alt`, `width`, `height`, `srcset`, `sizes`, `loading` |
| `<input>` | `type`, `name`, `value`, `required`, `placeholder`, `min`, `max` |
| `<video>` | `src`, `controls`, `autoplay`, `muted`, `loop`, `poster` |

#### Syntax Rules

1. **Use element-specific attributes only on their defined elements**.
2. **Element-specific attributes may also be global attributes** on that element.
3. **Invalid attributes are ignored by the parser** but may trigger validation errors.

#### Constraints and Limitations

- Some attributes have been deprecated or removed; for example, `align` on `<div>` is obsolete.
- The same attribute name may have different semantics on different elements.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct Element-Specific Attributes

**Setup Guide**: Save as `specific-attrs.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Specific Attributes</title>
</head>
<body>
  <!-- href is specific to <a> -->
  <a href="https://example.com">Example</a>

  <!-- src and alt are specific to <img> -->
  <img src="photo.jpg" alt="A photo" width="300">

  <!-- type and required are specific to <input> -->
  <input type="email" name="email" required placeholder="you@example.com">
</body>
</html>
```

**Expected output**: A link, an image, and an email input with a placeholder and required validation.

**Why this output occurs**: Each attribute is used on the element for which it is defined. The browser applies the link destination, loads the image, and enables email validation.

---

#### Example 2: Misplaced Attribute (Non-Conforming)

**Setup Guide**: Save as `misplaced-attr.html`. A validator will report an error.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Misplaced Attribute</title>
</head>
<body>
  <!-- href is not a valid attribute of <img> -->
  <img src="photo.jpg" alt="A photo" href="https://example.com">
</body>
</html>
```

**Expected output**: The image loads, but the `href` attribute has no effect and the markup is non-conforming.

**Why this output occurs**: `href` is not defined for `<img>`. The parser retains the attribute in the DOM but does not use it for navigation.

---

### Real-World Cases

1. **Hyperlinks**: `href` on `<a>` and `<area>`.
2. **Media**: `src` and `controls` on `<video>` and `<audio>`.
3. **Forms**: `type`, `name`, and `required` on `<input>`.

---

### References

- MDN Web Docs — HTML attribute reference (element-specific attributes) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes
- WHATWG — Attributes — https://html.spec.whatwg.org/multipage/dom.html#attributes

---

## 4. Boolean Attributes

### Definitions

**Core Definition**
A boolean attribute is an attribute whose presence on an element represents the `true` value, and whose absence represents the `false` value. The attribute’s value, if present, is ignored.

**Technical Definition**
In the WHATWG standard, a number of attributes are boolean attributes. The presence of a boolean attribute on an element represents the true value, and the absence represents the false value. If the attribute is present, its value must either be the empty string or a value that is an ASCII case-insensitive match for the attribute’s canonical name, with no leading or trailing whitespace. The values `"true"` and `"false"` are not allowed on boolean attributes; to represent a false value, the attribute must be omitted entirely. Common boolean attributes include `disabled`, `checked`, `required`, `readonly`, `multiple`, `selected`, `autofocus`, `inert`, `hidden`, `loop`, `autoplay`, `controls`, `muted`, and `reversed`.

**Beginner-Friendly Explanation**
Boolean attributes are on/off switches. If the attribute is there, it is on (true); if it is not there, it is off (false). Writing `disabled="false"` does not turn it off — the attribute is still present, so it is still true. To turn it off, you must remove the attribute entirely.

---

### Purposes

- **To enable or disable a feature** without needing a value (`disabled`, `required`).
- **To set a state** that is inherently binary (`checked`, `selected`).
- **To simplify markup** by omitting unnecessary values.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<input type="checkbox" checked>
<button type="button" disabled>Cannot click</button>
```

**Component breakdown**

| Form | Meaning |
|---|---|
| `checked` | True (present). |
| `checked=""` | True (empty value). |
| `checked="checked"` | True (canonical name as value). |
| `checked="false"` | True (presence alone means true; `"false"` is not a valid boolean value). |
| (attribute absent) | False. |

#### Syntax Rules

1. **Presence means true; absence means false**.
2. **The value is ignored**; `"true"` and `"false"` are not valid boolean values.
3. **The attribute may be written with no value, an empty value, or its canonical name**.
4. **Boolean attributes must not be written with a value of `"false"`** expecting false behaviour.

#### Constraints and Limitations

- Writing `disabled="false"` is a common mistake; it still disables the element.
- Some attributes that look boolean (e.g. `contenteditable`) are actually enumerated, not boolean.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Boolean Attributes in Forms

**Setup Guide**: Save as `boolean-form.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Boolean Attributes</title>
</head>
<body>
  <form>
    <!-- required: the field must be filled -->
    <label for="name">Name</label>
    <input type="text" id="name" name="name" required>

    <!-- disabled: the button cannot be clicked -->
    <button type="submit" disabled>Submit (disabled)</button>

    <!-- checked: the checkbox is checked by default -->
    <label>
      <input type="checkbox" name="agree" checked>
      I agree
    </label>
  </form>
</body>
</html>
```

**Expected output**: The name field is required; the submit button is disabled and cannot be clicked; the checkbox is checked by default.

**Why this output occurs**: `required`, `disabled`, and `checked` are boolean attributes. Their presence activates the corresponding behaviour.

---

#### Example 2: The `disabled="false"` Trap

**Setup Guide**: Save as `disabled-false.html`. Try clicking the button.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Disabled False Trap</title>
</head>
<body>
  <!-- WRONG: disabled="false" still disables the button -->
  <button type="button" disabled="false">Still disabled</button>

  <!-- CORRECT: omit the attribute to enable the button -->
  <button type="button">Enabled</button>
</body>
</html>
```

**Expected output**: The first button is disabled despite `disabled="false"`. The second button is enabled.

**Why this output occurs**: Boolean attributes are true whenever present, regardless of value. The string `"false"` does not make the attribute false; it must be removed entirely.

---

### Real-World Cases

1. **Form validation**: `required` and `disabled` on inputs and buttons.
2. **Media playback**: `controls`, `autoplay`, and `loop` on `<video>` and `<audio>`.
3. **Details disclosure**: `open` on `<details>`.

---

### References

- WHATWG — Boolean attributes — https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#boolean-attributes
- MDN Web Docs — Boolean attributes (Glossary) — https://developer.mozilla.org/en-US/docs/Glossary/Boolean/HTML

---

## 5. Enumerated Attributes

### Definitions

**Core Definition**
An enumerated attribute is an attribute that accepts one of a finite, predefined set of keywords. Each keyword maps to a particular state.

**Technical Definition**
In the WHATWG standard, some attributes are defined as taking one of a finite set of keywords; these are called enumerated attributes. The keywords are each defined to map to a particular state (several keywords may map to the same state). If an enumerated attribute is specified, its value must be an ASCII case-insensitive match for one of the defined keywords; invalid values may be ignored or treated as a default state depending on the attribute’s definition. Examples include `dir` (values `ltr`, `rtl`, `auto`) and `contenteditable` (`true`, `false`, or an empty string).

**Beginner-Friendly Explanation**
Enumerated attributes give you a menu of allowed values. For example, `dir` can be `ltr`, `rtl`, or `auto` — you cannot invent your own value. If you use an invalid value, the browser may ignore it or fall back to a default.

---

### Purposes

- **To constrain attribute values** to a known set of states.
- **To provide predictable behaviour** across browsers.
- **To support accessibility and internationalisation** with standardised keywords (e.g. `dir="rtl"`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<element dir="ltr">
<element contenteditable="true">
```

**Component breakdown**

| Attribute | Valid keywords | Default/invalid behaviour |
|---|---|---|
| `dir` | `ltr`, `rtl`, `auto` | Invalid value → inherited direction. |
| `contenteditable` | `true`, `false`, empty string, `plaintext-only` | Invalid value → `inherit`. |
| `draggable` | `true`, `false` | Invalid value → `auto` (browser default). |
| `spellcheck` | `true`, `false`, empty string | Invalid value → default behaviour. |

#### Syntax Rules

1. **The value must be one of the predefined keywords** (case-insensitive).
2. **Invalid values may be ignored** or mapped to a default state.
3. **An empty string may be a valid keyword** for some enumerated attributes.
4. **Do not confuse enumerated attributes with boolean attributes**; `contenteditable="false"` is a valid enumerated value, not a boolean.

#### Constraints and Limitations

- Browser handling of invalid enumerated values varies; do not rely on fallback behaviour.
- Enumerated attributes are not the same as free-form string attributes (e.g. `class`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `dir` Enumerated Attribute

**Setup Guide**: Save as `enumerated-dir.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Enumerated Dir</title>
</head>
<body>
  <!-- dir="rtl" sets right-to-left direction -->
  <p dir="rtl">This text is right-to-left.</p>

  <!-- dir="auto" lets the browser detect direction -->
  <p dir="auto">مرحبا</p>
</body>
</html>
```

**Expected output**: The first paragraph is rendered right-to-left. The second paragraph’s direction is automatically detected as right-to-left based on the Arabic text.

**Why this output occurs**: `dir` is an enumerated attribute with valid keywords `ltr`, `rtl`, and `auto`. The browser applies the corresponding direction.

---

#### Example 2: `contenteditable` Enumerated Values

**Setup Guide**: Save as `contenteditable-enum.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Contenteditable</title>
</head>
<body>
  <!-- contenteditable="true" makes the element editable -->
  <div contenteditable="true">You can edit this text.</div>

  <!-- contenteditable="false" makes it non-editable even inside an editable parent -->
  <div contenteditable="true">
    Parent is editable.
    <span contenteditable="false">But this span is not.</span>
  </div>
</body>
</html>
```

**Expected output**: The first div is editable. In the second div, the parent text is editable but the span is locked and cannot be edited.

**Why this output occurs**: `contenteditable` is an enumerated attribute. `"true"` enables editing; `"false"` disables it for that subtree.

---

### Real-World Cases

1. **Internationalisation**: `dir="rtl"` for right-to-left languages.
2. **Rich text editing**: `contenteditable="true"` for inline editors.
3. **Drag-and-drop**: `draggable="true"` for custom drag interfaces.

---

### References

- WHATWG — Keywords and enumerated attributes — https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#keywords-and-enumerated-attributes
- MDN Web Docs — Enumerated (Glossary) — https://developer.mozilla.org/en-US/docs/Glossary/Enumerated

---

## 6. Attribute Values

### Definitions

**Core Definition**
Attribute values are the data assigned to an attribute. They may be strings, numbers, keywords, URLs, or other types depending on the attribute’s definition.

**Technical Definition**
Attribute values in HTML are fundamentally strings in the DOM, but the specification defines **microsyntaxes** for interpreting them: boolean attributes (presence/absence), enumerated attributes (keywords), integers (e.g. `width`, `height`), floating-point numbers (e.g. `step`), dates and times (e.g. `datetime`), URLs, and space-separated or comma-separated token lists (e.g. `class`, `rel`). The value may be quoted or, under strict conditions, unquoted.

**Beginner-Friendly Explanation**
Attribute values are the information you assign. Some are simple words (`class="button"`), some are numbers (`width="400"`), some are URLs (`href="https://example.com"`), and some are lists (`class="btn btn-primary"`). The browser knows how to interpret each type.

---

### Purposes

- **To provide data for element behaviour** (URLs, numbers, keywords).
- **To enable styling and scripting hooks** (classes, IDs).
- **To convey metadata** (alternative text, titles).
- **To control form validation** (minimum and maximum values, patterns).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<element attribute="value">
```

**Component breakdown**

| Value type | Example | Notes |
|---|---|---|
| String | `class="button primary"` | Space-separated tokens. |
| Integer | `width="400"` | Non-negative integer. |
| Float | `step="0.5"` | Floating-point number. |
| URL | `href="https://example.com"` | Absolute or relative URL. |
| Enumerated keyword | `dir="rtl"` | Must be a valid keyword. |
| Boolean | `disabled` | Presence only. |

#### Syntax Rules

1. **Attribute values are strings in the DOM**, but the specification defines how to interpret them.
2. **Numeric values must be valid** for the attribute’s microsyntax (e.g. `width` must be a non-negative integer).
3. **URL values** may be absolute or relative.
4. **Token-list values** are space-separated (e.g. `class`, `rel`).

#### Constraints and Limitations

- Invalid numeric values may be ignored or cause the attribute to be treated as absent.
- URL values are resolved against the document base URL.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Numeric and String Values

**Setup Guide**: Save as `value-types.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Value Types</title>
</head>
<body>
  <!-- Numeric values -->
  <img src="photo.jpg" alt="Photo" width="400" height="300">

  <!-- Space-separated token list -->
  <p class="text-large text-bold text-red">Styled text</p>

  <!-- URL value -->
  <a href="https://example.com/path?query=1">Link</a>
</body>
</html>
```

**Expected output**: The image is sized 400×300; the paragraph has three CSS classes applied; the link points to the specified URL.

**Why this output occurs**: `width` and `height` are interpreted as non-negative integers; `class` is a space-separated token list; `href` is interpreted as a URL.

---

### Real-World Cases

1. **Images**: Numeric `width` and `height`.
2. **Styling**: Token-list `class`.
3. **Links**: URL `href`.

---

### References

- WHATWG — Common microsyntaxes — https://html.spec.whatwg.org/multipage/common-microsyntaxes.html
- MDN Web Docs — HTML attribute reference — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes

---

## 7. Attribute Quoting

### Definitions

**Core Definition**
Attribute quoting is the practice of wrapping attribute values in quotation marks (double or single) to delimit them unambiguously.

**Technical Definition**
HTML defines three attribute-value syntaxes: **double-quoted** (`attr="value"`), **single-quoted** (`attr='value'`), and **unquoted** (`attr=value`). Unquoted values must not contain ASCII whitespace, `"`, `'`, `` ` ``, `=`, `<`, or `>`. Because of these restrictions, quoting all attribute values is strongly recommended for readability, safety, and consistency. The Google HTML/CSS Style Guide recommends double quotes for attribute values.

**Beginner-Friendly Explanation**
Quoting means putting quotation marks around attribute values. While HTML sometimes allows you to skip the quotes, it is always safer and clearer to include them. Double quotes are the most common convention.

---

### Purposes

- **To prevent parsing errors** when values contain spaces or special characters.
- **To improve readability** and consistency.
- **To avoid ambiguity** when values include characters that could be misinterpreted.
- **To comply with style guides** and linters.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- Double-quoted (recommended) -->
<element attr="value">

<!-- Single-quoted (valid) -->
<element attr='value'>

<!-- Unquoted (valid only under strict conditions) -->
<element attr=value>
```

**Component breakdown**

| Form | Allowed characters in value |
|---|---|
| Double-quoted | Anything except `"` (use `&quot;` for literal quotes). |
| Single-quoted | Anything except `'` (use `&#39;` for literal apostrophes). |
| Unquoted | No whitespace, `"`, `'`, `` ` ``, `=`, `<`, `>`. |

#### Syntax Rules

1. **Double quotes are the recommended default**.
2. **Single quotes are valid** and useful when the value contains double quotes.
3. **Unquoted values are permitted only when they contain no whitespace or special characters**.
4. **Always quote values containing spaces** (e.g. `class="btn primary"`).

#### Constraints and Limitations

- Unquoted values cannot contain spaces; `class=btn primary` is parsed as `class="btn"` and an invalid `primary` attribute.
- Quote characters inside values must be escaped as character references when using the same quote type.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Quoting Best Practice

**Setup Guide**: Save as `quoting.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Quoting</title>
</head>
<body>
  <!-- Recommended: double quotes -->
  <p class="highlight">Double-quoted class</p>

  <!-- Valid: single quotes (value contains a double quote) -->
  <p title='He said "hello"'>Single-quoted title</p>

  <!-- Unquoted: valid because the value has no spaces or special characters -->
  <input type="text" name=username>
</body>
</html>
```

**Expected output**: All attributes are parsed correctly; the single-quoted `title` preserves the double quotes inside the value.

**Why this output occurs**: Double quotes are used by convention; single quotes allow literal double quotes inside the value; the unquoted `name=username` is valid because it contains no prohibited characters.

---

#### Example 2: Unquoted Value Failure

**Setup Guide**: Save as `unquoted-fail.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Unquoted Failure</title>
</head>
<body>
  <!-- WRONG: unquoted value with a space -->
  <p class=btn primary>This will not parse as intended.</p>
</body>
</html>
```

**Expected output**: The `class` attribute receives only `btn`; `primary` becomes a separate, invalid attribute. The intended two-class styling fails.

**Why this output occurs**: Unquoted attribute values cannot contain spaces. The parser ends the value at the first whitespace, so `primary` is treated as a separate attribute.

---

### Real-World Cases

1. **All HTML authoring**: Quoting is the default convention in style guides.
2. **Template engines**: Quoting avoids parsing issues when interpolating dynamic values.
3. **Code formatters**: Prettier and similar tools enforce quoted attributes.

---

### References

- MDN Web Docs — Basic HTML syntax: Always include quotes — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Basic_HTML_syntax
- Google HTML/CSS Style Guide — HTML quotation marks — https://google.github.io/styleguide/htmlcssguide.html#HTML_Quotation_Marks
- WHATWG — Attribute value syntax — https://html.spec.whatwg.org/multipage/syntax.html#attributes-2

---

## 8. Attribute Best Practices

### Definitions

**Core Definition**
Attribute best practices are guidelines for using attributes correctly, consistently, and securely to ensure valid, accessible, and maintainable HTML.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG accessibility guidelines, and security frameworks. They encompass correct attribute selection, proper quoting, avoiding deprecated attributes, using security-related attributes (`nonce`, `integrity`, `crossorigin`), and preferring native attributes over custom JavaScript solutions. The web.dev guidance emphasises using standard HTML elements and attributes, and removing default or unnecessary attributes when they do not affect functionality or accessibility.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for attributes. They ensure your HTML is valid, your pages are accessible, and your code is safe and easy to maintain.

---

### Purposes

- **To ensure validity** and avoid parser errors.
- **To maximise accessibility** through correct `lang`, `role`, `aria-*`, and labelling attributes.
- **To improve security** with `nonce`, `integrity`, and `crossorigin`.
- **To enhance maintainability** through consistent quoting and naming.
- **To avoid deprecated attributes** and future compatibility issues.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Always quote attribute values** | Use double quotes by convention; single quotes when values contain double quotes. |
| **Use lowercase attribute names** | HTML is case-insensitive, but lowercase is the standard convention. |
| **Do not use duplicate attributes** | Only the first is retained; duplicates are non-conforming. |
| **Use boolean attributes correctly** | Presence means true; never write `disabled="false"`. |
| **Use enumerated attributes with valid keywords** | Invalid values may be ignored. |
| **Provide `alt` for images** | Required for accessibility. |
| **Use `lang` on `<html>` and foreign-language elements** | Improves pronunciation and translation. |
| **Use `rel="noopener"` with `target="_blank"`** | Prevents `window.opener` attacks. |
| **Use `nonce` or `integrity` for scripts** | Supports CSP and Subresource Integrity. |
| **Avoid deprecated presentational attributes** | Use CSS instead (e.g. `align`, `bgcolor`). |
| **Do not use `accesskey`** | Conflicts with screen-reader and browser shortcuts. |

#### Syntax Rules

1. **Quote all attribute values**.
2. **Use the correct attribute for the element and purpose**.
3. **Prefer semantic and accessible attributes** (`alt`, `lang`, `role`).
4. **Apply security attributes** (`nonce`, `integrity`, `crossorigin`) to external resources.
5. **Avoid deprecated and presentational attributes**.

#### Constraints and Limitations

- Overusing `aria-*` attributes on native semantic elements can cause redundant or conflicting announcements.
- `title` tooltips are inaccessible on touch devices; do not rely on them for critical information.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible and Secure Attribute Usage

**Setup Guide**: Save as `best-practice-attrs.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Attributes</title>

  <!-- Subresource Integrity and CORS for third-party script -->
  <script
    src="https://cdn.example.com/library.js"
    integrity="sha384-abc123..."
    crossorigin="anonymous"
    defer>
  </script>

  <!-- CSP nonce for inline script -->
  <script nonce="random-nonce-value">
    console.log('Allowed by CSP');
  </script>
</head>
<body>
  <!-- alt for image accessibility -->
  <img src="chart.png" alt="Sales increased 20% in Q4" width="600" height="400">

  <!-- rel="noopener" with target="_blank" -->
  <a href="https://example.com" target="_blank" rel="noopener noreferrer">
    External link
  </a>
</body>
</html>
```

**Expected output**: The third-party script is verified by SRI and loaded with CORS; the inline script is allowed by the CSP nonce; the image has descriptive alternative text; the external link is secure.

**Why this output occurs**: `integrity` and `crossorigin` verify the script’s authenticity; `nonce` authorises the inline script under CSP; `alt` provides an accessible name; `rel="noopener noreferrer"` prevents the new page from accessing `window.opener` and strips the referrer.

---

#### Example 2: Avoiding Common Attribute Mistakes

**Setup Guide**: Save as `attr-mistakes.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Mistakes</title>
</head>
<body>
  <!-- MISTAKE 1: unquoted value with space -->
  <p class=btn primary>Wrong</p>

  <!-- MISTAKE 2: disabled="false" still disables -->
  <button type="button" disabled="false">Disabled</button>

  <!-- MISTAKE 3: duplicate id -->
  <div id="dup"></div>
  <div id="dup"></div>
</body>
</html>
```

**Expected output**: The `class` only receives `btn`; the button is disabled; the duplicate `id` causes validation errors and unpredictable scripting behaviour.

**Why this output occurs**: Unquoted values cannot contain spaces; boolean attributes are true when present regardless of value; duplicate `id` values violate uniqueness.

---

### Real-World Cases

1. **Content Security Policy**: `nonce` attributes on scripts and styles.
2. **Subresource Integrity**: `integrity` and `crossorigin` on third-party resources.
3. **Accessibility compliance**: `alt`, `lang`, and `aria-*` attributes.
4. **Secure external links**: `rel="noopener noreferrer"` with `target="_blank"`.

---

### References

- WHATWG — Attributes — https://html.spec.whatwg.org/multipage/dom.html#attributes
- MDN Web Docs — HTML attribute reference — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes
- MDN Web Docs — Global attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes
- W3C — Content Security Policy Level 3 (nonce) — https://www.w3.org/TR/CSP3/
- web.dev — Attributes — https://web.dev/learn/html/attributes

---

## Summary Table of Attribute Categories

| Category | Key Examples | Semantics |
|---|---|---|
| **Global** | `class`, `id`, `lang`, `hidden`, `style` | Usable on all elements. |
| **Element-specific** | `href`, `src`, `alt`, `type` | Only valid on defined elements. |
| **Boolean** | `disabled`, `checked`, `required`, `hidden` | Presence = true; absence = false. |
| **Enumerated** | `dir`, `contenteditable`, `draggable` | Value must be a predefined keyword. |
| **String** | `class`, `title`, `placeholder` | Free-form text. |
| **Numeric** | `width`, `height`, `maxlength`, `step` | Parsed as numbers. |
| **URL** | `href`, `src`, `action` | Parsed as URLs. |
| **Security** | `nonce`, `integrity`, `crossorigin` | CSP and SRI. |
| **Accessibility** | `alt`, `lang`, `role`, `aria-*` | Assistive-technology semantics. |

---

## Final Notes on Deprecated, Unsafe, and Implementation-Dependent Attributes

- **Presentational attributes** such as `align`, `bgcolor`, `border`, and `valign` are obsolete; use CSS instead.
- **`accesskey`** is discouraged because shortcuts conflict with screen-reader and browser commands.
- **`http-equiv`** values such as `content-type` and `set-cookie` are obsolete or dangerous; use HTTP headers instead.
- **`on*` event handler attributes** are discouraged in favour of `addEventListener` and are incompatible with strict CSP.
- **`nonce` and `integrity`** are critical for modern security but require server-side support and HTTPS for full effectiveness.

---

This cheat sheet has covered HTML attribute fundamentals — attributes, global attributes, element-specific attributes, boolean attributes, enumerated attributes, attribute values, attribute quoting, and attribute best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate attribute usage against the WHATWG HTML Living Standard, WCAG accessibility guidelines, and Content Security Policy best practices.