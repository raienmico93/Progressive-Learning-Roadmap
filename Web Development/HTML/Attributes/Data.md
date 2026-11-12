# Data Attributes (`data-*`) — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Data attributes (`data-*`) are a class of custom HTML attributes that allow developers to store proprietary, script-readable information directly on standard HTML elements. They provide a standards-based mechanism for attaching data to elements without resorting to non-standard attributes, hidden elements, or extra DOM properties.

**Technical Definition**
In the WHATWG HTML Living Standard and the W3C HTML 5.2 specification, a **custom data attribute** is defined as an attribute in no namespace whose name starts with the string `data-`, has at least one character after the hyphen, is XML-compatible, and contains no uppercase ASCII letters. Every HTML element may have any number of custom data attributes specified, with any value. The data stored in these attributes is intended to be **private to the page or application** — it is not a generic extension mechanism for publicly usable metadata. All `data-*` attributes on an element are exposed through the `HTMLElement.dataset` property, which returns a `DOMStringMap` object. The portion of the attribute name after `data-` is converted from kebab-case to camelCase for property access: `data-track-length` becomes `dataset.trackLength`.

**Beginner-Friendly Explanation**
Data attributes are custom “labels” you can attach to any HTML element to store extra information that your JavaScript can read later. They always start with `data-`. For example, `<li data-id="42">Item</li>` stores the ID `42` on the list item. The browser does not display this information, but your scripts can read it, update it, and use it to make decisions. Data attributes are the clean, standardised way to associate private data with an element without inventing your own attributes or abusing `class` names.

---

### Key Characteristics

- **Universal applicability**: Data attributes are **global attributes**; they can be used on any HTML element, including standard elements, custom elements, and SVG elements.
- **Namespaced by prefix**: Every data attribute name begins with `data-`, which acts as a namespace that prevents collisions with future standard HTML attributes.
- **Private to the page or application**: The specification explicitly states that data attributes are intended for the page’s own scripts, not for software independent of the site.
- **String-valued in the DOM**: Attribute values are stored as strings; numeric or boolean data must be parsed in JavaScript.
- **Script access via `dataset`**: The `HTMLElement.dataset` property provides a convenient `DOMStringMap` interface, automatically converting hyphenated attribute names to camelCase property names.
- **CSS selectability**: Data attributes can be targeted with CSS attribute selectors (`[data-state="open"]`) and used with the `attr()` function for generated content.
- **No accessibility semantics**: Screen readers and other assistive technologies do not interpret `data-*` attributes; they must not be used to convey information essential to understanding the page.
- **Not for sensitive data**: Because data attributes are visible in the DOM, they must never contain secrets, personal data, or security tokens.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, tags, and attributes.
2. **The concept of the DOM**: data attributes become properties on DOM element objects.
3. **JavaScript fundamentals**: variables, objects, property access, and type conversion (because `dataset` values are strings).
4. **CSS attribute selectors**: understanding `[attr]` and `[attr="value"]` syntax.
5. **Awareness of accessibility**: knowing that data attributes are invisible to assistive technologies.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **JavaScript / DOM scripting** | The primary consumer of data attributes; `dataset` provides read/write access. |
| **CSS** | Attribute selectors and the `attr()` function can read data attributes for styling and generated content. |
| **Web Components** | Custom elements frequently use `data-*` attributes to store state and configuration. |
| **Testing and automation** | `data-testid` and similar attributes are widely used as stable selectors for end-to-end tests. |
| **State management** | UI state such as `data-state="open"` or `data-loading` is commonly encoded in data attributes. |
| **Accessibility** | Data attributes are explicitly **not** an accessibility mechanism; ARIA and native semantics must be used instead. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`data-*`** — the attribute class
2. **Data attribute naming** — syntax and constraints
3. **Data attribute access** — `dataset` and `getAttribute()`
4. **Data attribute usage** — patterns and anti-patterns
5. **Data attribute best practices** — quality, security, and performance guidelines

---

## 1. `data-*` — The Custom Data Attribute Class

### Definitions

**Core Definition**
`data-*` is a global attribute pattern that allows any HTML element to carry custom, script-readable data. The asterisk represents the author-chosen portion of the attribute name, which must follow XML naming rules.

**Technical Definition**
A custom data attribute is an attribute in no namespace whose name starts with the string `data-`, has at least one character after the hyphen, is XML-compatible, and contains no uppercase ASCII letters. Every HTML element may have any number of custom data attributes specified, with any value. These attributes are exposed via `HTMLElement.dataset` as a `DOMStringMap`, and are also accessible via `getAttribute()` and CSS attribute selectors. The data is intended for use by the page’s own scripts and is not a public metadata extension mechanism.

**Beginner-Friendly Explanation**
`data-*` is the pattern you use to invent your own attributes. You write `data-` followed by a name you choose — like `data-user-id`, `data-price`, or `data-status`. The browser stores whatever value you give it, and your JavaScript can read it later.

---

### Purposes

- **To store private, script-readable data** on HTML elements without inventing non-standard attributes.
- **To provide a clean separation** between styling (`class`), structure (semantic elements), and application data.
- **To enable state encoding** on elements (e.g. `data-state="open"`) for CSS and JavaScript consumption.
- **To serve as stable test hooks** (`data-testid`) for automated testing frameworks.
- **To avoid abusing `class` or `id`** for data storage that has no presentational or identity meaning.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<any-element data-custom-name="value">
```

**Component breakdown**

| Component | Description |
|---|---|
| `data-` | The required prefix. |
| `custom-name` | The author-chosen name, following XML naming rules. |
| `="value"` | The string value; may be empty. |

#### Syntax Rules

1. **Every HTML element may have any number of data attributes**, with any value.
2. **Attribute names must be XML-compatible**, must not contain uppercase ASCII letters, must not start with `xml` (case-insensitive), and must not contain colons (`:`).
3. **The portion after `data-` must contain at least one character**.
4. **Values are always strings in the DOM**; JavaScript must convert them to numbers or booleans when needed.
5. **Data attributes are exposed via `dataset`** with camelCase conversion.

#### Constraints and Limitations

- **Not for public metadata**: data attributes are private to the page; external software should not rely on them.
- **Not for accessibility**: screen readers ignore them; use ARIA and native semantics for accessibility.
- **Not for sensitive data**: values are visible in the DOM and in page source.
- **Naming recommendations are not hard errors**: non-conforming names still work in browsers but are discouraged.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Storing Application Data on List Items

**Setup Guide**: Save as `data-basic.html` and open in a browser. Inspect the elements with developer tools.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Data Attribute Basics</title>
</head>
<body>
  <h1>Agents</h1>
  <ul>
    <!-- data-id stores a numeric identifier -->
    <li data-id="10784">Jason Walters, 003</li>
    <!-- data-id and data-clearance store multiple values -->
    <li data-id="97865" data-clearance="top-secret">Alex Trevelyan, 006</li>
    <!-- data-active stores a boolean-like state as a string -->
    <li data-id="45732" data-active="true">James Bond, 007</li>
  </ul>
</body>
</html>
```

**Expected output**: A list of three agents. The data attributes are not visible on the page but are present in the DOM and accessible via JavaScript.

**Why this output occurs**: The `data-*` attributes store private application data on each list item. The browser renders only the text content; the attributes remain in the DOM for scripts to read.

---

#### Example 2: Multiple Data Attributes on One Element

**Setup Guide**: Save as `data-multiple.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Multiple Data Attributes</title>
</head>
<body>
  <!-- An element can carry many data attributes -->
  <article
    id="electric-cars"
    data-columns="3"
    data-index-number="12314"
    data-parent="cars">
    <h2>Electric Cars</h2>
  </article>
</body>
</html>
```

**Expected output**: The article renders normally; its data attributes are available for scripts and CSS.

**Why this output occurs**: The specification allows any number of custom data attributes on any element, with any value.

---

### Real-World Cases

1. **E-commerce product listings**: `data-product-id`, `data-price`, and `data-category` store product metadata for client-side filtering and cart logic.
2. **Interactive UI state**: `data-state="open"` on accordions and `data-loading` on buttons drive CSS and JavaScript behaviour.
3. **Analytics and tracking**: `data-track="cta-click"` labels elements for event tracking without adding extra classes or IDs.

---

### References

- MDN Web Docs — `data-*` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*
- W3C — HTML 5.2: Embedding custom non-visible data — https://www.w3.org/TR/html52/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes
- WHATWG — Custom data attributes — https://html.spec.whatwg.org/multipage/dom.html#custom-data-attribute

---

## 2. Data Attribute Naming

### Definitions

**Core Definition**
Data attribute naming refers to the rules and recommendations for the portion of the attribute name that follows the `data-` prefix. Names must be XML-compatible and contain no uppercase ASCII letters.

**Technical Definition**
A custom data attribute name must start with `data-`, have at least one character after the hyphen, be XML-compatible, and contain no uppercase ASCII letters. The W3C and MDN further recommend that the name should not start with `xml` (case-insensitive, because it is reserved for future XML specifications), should not contain colon characters (because XML assigns meaning to them), and should not contain capital letters (because XML is all lowercase). These are recommendations rather than hard parsing errors, but following them ensures interoperability and avoids surprises with `dataset`.

**Beginner-Friendly Explanation**
The name you choose after `data-` should be all lowercase, should not start with “xml”, and should not contain colons. Hyphens are allowed and are converted to camelCase when you access the attribute through `dataset`. For example, `data-user-name` becomes `dataset.userName`.

---

### Purposes

- **To prevent collisions** with standard HTML attributes and future specifications.
- **To ensure `dataset` compatibility** through predictable camelCase conversion.
- **To maintain XML compatibility** for documents processed by XML tools.
- **To support CSS attribute selectors** reliably.

---

### Syntax Rules and Structure

#### Complete General Syntax

```
data-[a-z][a-z0-9-]*
```

**Component breakdown**

| Rule | Requirement |
|---|---|
| Prefix | Must be `data-`. |
| First character after hyphen | Must exist. |
| Case | No uppercase ASCII letters (recommended and spec-required for `dataset` compatibility). |
| Reserved prefix | Must not start with `xml` (case-insensitive). |
| Colon | Must not contain `:` (recommended). |
| XML compatibility | Must follow XML `Name` production rules. |

#### Syntax Rules

1. **No uppercase ASCII letters**: `data-userId` is discouraged; `data-user-id` is correct.
2. **No leading `xml`**: `data-xml-data` is discouraged because `xml` is reserved.
3. **No colons**: `data-foo:bar` is discouraged because XML assigns meaning to colons.
4. **Hyphens are converted to camelCase in `dataset`**: `data-user-name` → `dataset.userName`.
5. **Avoid digits immediately after a hyphen if `dataset` access is needed**: `data-test-1` is not reliably accessible via `dataset` because the hyphen is followed by a digit, not a letter.

#### Constraints and Limitations

- **Non-conforming names still work** in browsers and CSS, but may behave unpredictably with `dataset`.
- **The portion after `data-` must be a valid JavaScript property name segment** if `dataset` access is required (letters, digits, `_`, `$`, and hyphens that convert to camelCase).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recommended vs Discouraged Naming

**Setup Guide**: Save as `naming-rules.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Data Attribute Naming</title>
</head>
<body>
  <!-- RECOMMENDED: lowercase, hyphenated -->
  <div data-user-name="Alice" data-account-level="admin"></div>

  <!-- DISCOURAGED: uppercase and leading xml -->
  <div data-userName="Bob" data-xml-data="true"></div>
</body>
</html>
```

**Expected output**: The first div’s attributes are accessible as `dataset.userName` and `dataset.accountLevel`. The second div’s `data-userName` is accessible via `dataset.userName` (because attribute names are lowercased in HTML) but is non-conforming; `data-xml-data` is discouraged but still parsed.

**Why this output occurs**: HTML lowercases attribute names during parsing, so `data-userName` becomes `data-username`, which maps to `dataset.username` — not `dataset.userName` as the author might expect. Following the lowercase recommendation avoids this confusion.

---

#### Example 2: Hyphen-to-CamelCase Conversion

**Setup Guide**: Save as `naming-camel.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CamelCase Conversion</title>
</head>
<body>
  <div data-track-length="2m11s"></div>

  <script>
    const el = document.querySelector('[data-track-length]');
    // dataset.trackLength is the camelCase equivalent
    console.log(el.dataset.trackLength); // "2m11s"
    console.log(el.getAttribute('data-track-length')); // "2m11s"
  </script>
</body>
</html>
```

**Expected output**: The console logs `"2m11s"` for both the `dataset` access and the `getAttribute` access.

**Why this output occurs**: `dataset` converts `data-track-length` to `trackLength` by removing hyphens and capitalising the following letter.

---

### Real-World Cases

1. **Design systems**: Consistent naming conventions like `data-state`, `data-variant`, and `data-size` across components.
2. **Testing**: `data-testid` is a widely adopted convention for stable test selectors.
3. **Analytics**: `data-track-category` and `data-track-action` follow predictable naming for event systems.

---

### References

- MDN Web Docs — `data-*`: Naming recommendations — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*#naming_recommendations
- W3C — XML Name production — https://www.w3.org/TR/xml/#NT-Name
- W3C — HTML 5.2: Custom data attributes — https://www.w3.org/TR/html52/dom.html#custom-data-attribute

---

## 3. Data Attribute Access

### Definitions

**Core Definition**
Data attribute access refers to the mechanisms by which JavaScript and CSS can read, write, and select elements based on their `data-*` attributes. The primary JavaScript interface is `HTMLElement.dataset`; the primary CSS mechanism is attribute selectors.

**Technical Definition**
The `HTMLElement.dataset` property returns a `DOMStringMap` object that exposes all `data-*` attributes on the element. For each attribute whose name begins with `data-` and whose remaining characters contain no uppercase ASCII letters, `dataset` adds a name-value pair; hyphens followed by lowercase letters are removed and the following letter is uppercased (camelCase conversion). Setting a `dataset` property reflects back to the corresponding `data-*` attribute; deleting a `dataset` property removes the attribute. Alternatively, `getAttribute()` and `setAttribute()` can be used with the full attribute name. CSS attribute selectors (`[data-foo]`, `[data-foo="bar"]`) and the `attr()` function provide declarative access.

**Beginner-Friendly Explanation**
You read data attributes in JavaScript using `element.dataset`. For `data-user-name`, you write `element.dataset.userName`. You can also use `element.getAttribute('data-user-name')`. In CSS, you can select elements with `[data-user-name]` or `[data-user-name="Alice"]`.

---

### Purposes

- **To read stored data** from elements in JavaScript.
- **To write or update data** on elements dynamically.
- **To select elements** based on their data values in CSS and JavaScript.
- **To provide a stable testing interface** that is decoupled from styling classes.

---

### Syntax Rules and Structure

#### Complete General Syntax (JavaScript via `dataset`)

```javascript
const value = element.dataset.camelCaseName;
element.dataset.camelCaseName = 'new value';
```

#### Complete General Syntax (JavaScript via `getAttribute`)

```javascript
const value = element.getAttribute('data-kebab-case-name');
element.setAttribute('data-kebab-case-name', 'new value');
```

#### Complete General Syntax (CSS)

```css
[data-state="open"] { display: block; }
[data-state]::before { content: attr(data-state); }
```

**Component breakdown**

| Access method | Description |
|---|---|
| `dataset.camelCaseName` | CamelCase property access; reflects to the attribute. |
| `getAttribute('data-name')` | Full attribute name access; returns a string. |
| `setAttribute('data-name', value)` | Sets the attribute value. |
| `[data-name]` | CSS attribute existence selector. |
| `[data-name="value"]` | CSS attribute value selector. |
| `attr(data-name)` | CSS function that reads the attribute value. |

#### Syntax Rules

1. **`dataset` converts kebab-case to camelCase**: `data-foo-bar` → `dataset.fooBar`.
2. **`dataset` values are strings**; type conversion is the author’s responsibility.
3. **`getAttribute()` returns the raw string** and works with any attribute name.
4. **CSS attribute selectors are case-sensitive for values** but case-insensitive for attribute names in HTML.
5. **`dataset` does not expose attributes whose names do not follow the camelCase conversion rules** (e.g. `data-test-1`).

#### Constraints and Limitations

- **`dataset` camelCase conversion can surprise authors** who expect the raw attribute name.
- **`getAttribute()` is more verbose** but works for all attribute names.
- **CSS attribute selectors may be slower** than class or ID selectors in some browsers, though the difference is usually negligible.
- **Data attributes are not accessible to screen readers**; they must not carry essential information.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading and Writing with `dataset`

**Setup Guide**: Save as `access-dataset.html`. Open the console.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Dataset Access</title>
</head>
<body>
  <article id="electric-cars"
           data-columns="3"
           data-index-number="12314"
           data-parent="cars">
    Electric Cars
  </article>

  <script>
    const article = document.querySelector('#electric-cars');

    // Read via dataset (camelCase)
    console.log(article.dataset.columns);      // "3"
    console.log(article.dataset.indexNumber);  // "12314"
    console.log(article.dataset.parent);       // "cars"

    // Write via dataset (reflects to the attribute)
    article.dataset.columns = 5;
    console.log(article.getAttribute('data-columns')); // "5"

    // Read via getAttribute (full name)
    console.log(article.getAttribute('data-index-number')); // "12314"
  </script>
</body>
</html>
```

**Expected output**: The console logs `"3"`, `"12314"`, `"cars"`, then `"5"` after the write, and finally `"12314"` again.

**Why this output occurs**: `dataset.columns` reads `data-columns`; `dataset.indexNumber` reads `data-index-number` through camelCase conversion; setting `dataset.columns` updates the underlying attribute.

---

#### Example 2: CSS Attribute Selectors and `attr()`

**Setup Guide**: Save as `access-css.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Access to Data Attributes</title>
  <style>
    /* Show the data-value in generated content */
    .metric::after {
      content: " (" attr(data-value) ")";
      color: gray;
    }

    /* Style elements based on a data-state value */
    [data-state="active"] {
      font-weight: bold;
      color: green;
    }
  </style>
</head>
<body>
  <p class="metric" data-value="42">Answer</p>
  <p data-state="active">Active item</p>
  <p data-state="inactive">Inactive item</p>
</body>
</html>
```

**Expected output**: The first paragraph displays “Answer (42)”; the second paragraph is bold and green; the third is normal.

**Why this output occurs**: `attr(data-value)` reads the attribute value into generated content, and `[data-state="active"]` selects the element by its data attribute value.

---

### Real-World Cases

1. **UI state toggling**: `element.dataset.state = 'open'` updates both the attribute and CSS styling.
2. **Form validation feedback**: `data-error="true"` triggers error styles via `[data-error="true"]`.
3. **Testing**: `element.dataset.testid` provides a stable selector hook independent of CSS classes.

---

### References

- MDN Web Docs — `HTMLElement.dataset` — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset
- MDN Web Docs — Using data attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/Use_data_attributes
- W3C — HTML 5.1: `dataset` IDL attribute — https://www.w3.org/TR/html51/dom.html#dom-dataset

---

## 4. Data Attribute Usage

### Definitions

**Core Definition**
Data attribute usage refers to the patterns, conventions, and anti-patterns associated with storing and consuming data in `data-*` attributes. Correct usage respects the specification’s intent that data attributes hold private, non-visible, script-oriented data.

**Technical Definition**
The specification defines data attributes as a mechanism for storing custom data **private to the page or application, for which there are no more appropriate attributes or elements**. They are **not intended for use by software that is independent of the site**. A music site could annotate tracks with `data-length="2m11s"` and use that data to sort or filter, but generic external software should not rely on those attributes as public metadata. Data attributes are also not a substitute for semantic HTML, ARIA, or visible content when the information is needed by users or assistive technologies.

**Beginner-Friendly Explanation**
Use data attributes for “behind-the-scenes” data that your own JavaScript needs. Do not use them for information that users should see, for accessibility labels, or for secrets. Think of them as private notes you write on an element for your scripts to read later.

---

### Purposes

- **To store application state** on elements without polluting `class` or `id`.
- **To provide configuration** for JavaScript widgets and components.
- **To enable data-driven styling** through CSS attribute selectors.
- **To expose stable test hooks** for automated testing.
- **To avoid overloading semantic attributes** with unrelated data.

---

### Syntax Rules and Structure

#### Complete General Syntax (State Encoding)

```html
<button data-state="idle" data-action="save">Save</button>
```

#### Complete General Syntax (Configuration)

```html
<div data-config='{"theme":"dark","pageSize":20}'></div>
```

**Component breakdown**

| Pattern | Use case |
|---|---|
| `data-state="value"` | Encode UI state for CSS and JavaScript. |
| `data-action="value"` | Identify the action an element triggers. |
| `data-config='{"…"}'` | Store JSON configuration as a string. |
| `data-testid="value"` | Provide a stable testing selector. |

#### Syntax Rules

1. **Use data attributes for private, non-visible data** that scripts consume.
2. **Do not use data attributes for accessibility information**; use ARIA and native semantics.
3. **Do not store sensitive data**; attributes are visible in the DOM and page source.
4. **Keep values small**; data attributes are not a substitute for a database or state management library.
5. **Use JSON strings for complex data**, but be aware that parsing errors must be handled.

#### Constraints and Limitations

- **No accessibility semantics**: screen readers ignore data attributes, so they cannot replace `aria-*` or visible labels.
- **No public metadata contract**: external software should not depend on your data attribute names; use microdata or JSON-LD for public structured data.
- **Performance overhead**: large numbers of attributes or very long values can increase DOM size and memory usage.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: State-Driven UI with Data Attributes

**Setup Guide**: Save as `usage-state.html`. Click the button to toggle the state.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>State with Data Attributes</title>
  <style>
    .panel { display: none; }
    [data-state="open"] .panel { display: block; }
    [data-state="open"] .toggle::after { content: " (open)"; }
  </style>
</head>
<body>
  <div id="widget" data-state="closed">
    <button class="toggle" type="button">Toggle</button>
    <div class="panel">Panel content.</div>
  </div>

  <script>
    const widget = document.getElementById('widget');
    widget.querySelector('.toggle').addEventListener('click', () => {
      widget.dataset.state = widget.dataset.state === 'open' ? 'closed' : 'open';
    });
  </script>
</body>
</html>
```

**Expected output**: Clicking the button toggles the panel between visible and hidden, and the button label gains “ (open)” when the state is open.

**Why this output occurs**: The `data-state` attribute drives CSS selectors; JavaScript updates `dataset.state`, which reflects to the attribute and triggers the CSS changes.

---

#### Example 2: Storing JSON Configuration

**Setup Guide**: Save as `usage-json.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>JSON Configuration</title>
</head>
<body>
  <div id="chart" data-config='{"type":"bar","series":[1,2,3]}'></div>

  <script>
    const el = document.getElementById('chart');
    try {
      const config = JSON.parse(el.dataset.config);
      console.log(config.type);    // "bar"
      console.log(config.series);  // [1, 2, 3]
    } catch (e) {
      console.error('Invalid config:', e);
    }
  </script>
</body>
</html>
```

**Expected output**: The console logs `"bar"` and `[1, 2, 3]`.

**Why this output occurs**: The JSON string is stored in `data-config`, read via `dataset.config`, and parsed with `JSON.parse`. Error handling prevents crashes from malformed JSON.

---

### Real-World Cases

1. **Component configuration**: Widgets read `data-options` as a JSON string to configure themselves.
2. **Analytics labelling**: `data-track` attributes identify elements for event tracking without adding extra classes.
3. **Progressive enhancement**: Server-rendered HTML includes `data-*` attributes that client-side scripts use to hydrate behaviour.

---

### References

- W3C — HTML 5.1: Custom data attributes — https://www.w3.org/TR/html51/dom.html#custom-data-attribute
- MDN Web Docs — Using data attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/Use_data_attributes
- WHATWG — Custom data attributes — https://html.spec.whatwg.org/multipage/dom.html#custom-data-attribute

---

## 5. Data Attribute Best Practices

### Definitions

**Core Definition**
Data attribute best practices are guidelines for using `data-*` attributes in ways that are semantically appropriate, accessible, secure, performant, and maintainable.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, the W3C HTML specification, accessibility guidance (WCAG and ARIA), and security principles. They encompass correct scoping of data attributes to private application data, avoiding accessibility-critical information, preventing sensitive data exposure, choosing appropriate access mechanisms (`dataset` vs `getAttribute`), and maintaining performance with CSS selectors and DOM size.

**Beginner-Friendly Explanation**
Best practices help you use data attributes for the right purposes and avoid common traps like putting secrets in the DOM, relying on data attributes for accessibility, or creating naming collisions.

---

### Purposes

- **To ensure data attributes are used for their intended purpose** — private script data.
- **To protect accessibility** by keeping essential information out of data attributes.
- **To protect security** by never storing secrets or sensitive personal data in the DOM.
- **To maintain performance** by keeping attribute counts and selector complexity reasonable.
- **To improve maintainability** through consistent naming and access patterns.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use data attributes only for private script data** | Do not use them for public metadata or accessibility semantics. |
| **Never store sensitive data** | Passwords, tokens, and personal data must not appear in the DOM. |
| **Use `data-testid` for test hooks** | Prefer dedicated test attributes over abusing `class` or `id`. |
| **Prefer `dataset` for camelCase access** | Use `getAttribute()` when you need the raw attribute name or broader compatibility. |
| **Validate and convert types explicitly** | `dataset` values are strings; convert to numbers or booleans with `parseInt`, `Number`, or explicit comparisons. |
| **Handle missing attributes** | Check `'prop' in element.dataset` or use optional chaining to avoid `undefined` errors. |
| **Keep values small and simple** | Avoid storing large JSON blobs or binary data in attributes. |
| **Avoid non-alphabetic characters after hyphens** | `data-test-1` is not reliably accessible via `dataset`. |
| **Use JSON sparingly and parse defensively** | Always wrap `JSON.parse` in `try…catch`. |
| **Do not replace ARIA with data attributes** | Use `aria-*` and native semantics for accessibility. |

#### Syntax Rules

1. **Scope**: Data attributes are for the page’s own scripts, not for external consumers.
2. **Accessibility**: Never rely on data attributes to convey information to assistive technologies.
3. **Security**: Treat every data attribute value as publicly visible.
4. **Naming**: Follow the lowercase, XML-compatible naming recommendations.
5. **Access**: Use `dataset` for camelCase convenience; use `getAttribute()` for exact names and maximum compatibility.

#### Constraints and Limitations

- **`dataset` does not expose all attribute names**; non-conforming names may be inaccessible.
- **CSS attribute selectors can be slower** than class or ID selectors, though the difference is usually negligible in practice.
- **Data attributes are not a state management solution**; complex application state should live in JavaScript memory or a dedicated store.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe and Accessible Data Attribute Usage

**Setup Guide**: Save as `best-practice-data.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Data Attributes</title>
</head>
<body>
  <!-- GOOD: data-testid for testing, data-state for UI state -->
  <button
    type="button"
    data-testid="submit-btn"
    data-state="idle"
    aria-label="Submit form">
    Submit
  </button>

  <!-- BAD: data-label used for accessible name instead of aria-label -->
  <!--
  <button type="button" data-label="Submit form">Submit</button>
  -->

  <!-- BAD: sensitive token in a data attribute -->
  <!--
  <div data-api-key="sk_live_12345"></div>
  -->
</body>
</html>
```

**Expected output**: The button is accessible via `aria-label`, testable via `data-testid`, and its UI state is encoded in `data-state`. No sensitive data is exposed.

**Why this output occurs**: `aria-label` provides the accessible name; `data-testid` is a private test hook; `data-state` is private UI state. The anti-patterns show common mistakes: using a data attribute for an accessible name and storing a secret in the DOM.

---

#### Example 2: Type Conversion and Missing Attribute Handling

**Setup Guide**: Save as `best-practice-conversion.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Data Attribute Conversions</title>
</head>
<body>
  <div id="product" data-price="29.99" data-in-stock="true"></div>

  <script>
    const product = document.getElementById('product');

    // WRONG: treating strings as numbers
    // const total = product.dataset.price * 2; // "29.992" (string concatenation)

    // CORRECT: explicit numeric conversion
    const price = parseFloat(product.dataset.price);
    console.log(price * 2); // 59.98

    // CORRECT: explicit boolean conversion
    const inStock = product.dataset.inStock === 'true';
    console.log(inStock); // true

    // CORRECT: checking for missing attributes
    if ('discount' in product.dataset) {
      console.log(product.dataset.discount);
    } else {
      console.log('No discount attribute');
    }
  </script>
</body>
</html>
```

**Expected output**: The console logs `59.98`, `true`, and `"No discount attribute"`.

**Why this output occurs**: `dataset` values are strings; explicit conversion prevents type-coercion bugs. The `'in'` operator safely checks for attribute existence.

---

### Real-World Cases

1. **E-commerce**: `data-price` values are converted with `parseFloat` before arithmetic; `data-in-stock` is compared to the string `"true"`.
2. **Testing**: `data-testid` provides stable selectors that do not change when CSS classes are refactored.
3. **Accessibility audits**: Teams ensure that no critical information lives exclusively in `data-*` attributes.

---

### References

- MDN Web Docs — `data-*` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*
- MDN Web Docs — Using data attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/Use_data_attributes
- W3C — HTML 5.1: Custom data attributes — https://www.w3.org/TR/html51/dom.html#custom-data-attribute
- Oxylabs — How to Get Data Attribute Value in JavaScript (best practices) — https://oxylabs.io/resources/web-scraping-faq/javascript/data-attribute
- W3C — WCAG 2.2 — https://www.w3.org/WAI/standards-guidelines/wcag/

---

## Summary Table of Data Attribute Concepts

| Concept | Key Takeaway |
|---|---|
| **`data-*`** | Custom attributes for private, script-readable data on any element. |
| **Naming** | Lowercase, XML-compatible, no leading `xml`, no colons; hyphens convert to camelCase in `dataset`. |
| **Access** | `element.dataset.camelCaseName` for convenience; `getAttribute()` for raw names; CSS `[data-*]` selectors for styling. |
| **Usage** | State encoding, configuration, test hooks; never for accessibility or sensitive data. |
| **Best practices** | Scope to private data, convert types explicitly, handle missing attributes, avoid secrets, respect accessibility. |

---

## Final Notes

- **Specification status**: Data attributes are a stable, widely supported feature of HTML, defined in the WHATWG Living Standard and W3C HTML 5.2/5.1.
- **Private contract**: Data attributes are intended for the page’s own scripts, not for external software or public metadata.
- **Accessibility boundary**: Screen readers ignore `data-*`; ARIA and native semantics remain mandatory for accessibility.
- **Security boundary**: Everything in a data attribute is visible in the DOM; never store secrets.
- **Performance**: While data attribute selectors can be marginally slower than class or ID selectors, the difference is rarely significant; prefer clarity and maintainability.

---

This cheat sheet has covered `data-*` attributes — the attribute class, naming rules, access mechanisms, usage patterns, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate data attribute usage against the WHATWG HTML Living Standard, W3C specifications, and accessibility guidelines.