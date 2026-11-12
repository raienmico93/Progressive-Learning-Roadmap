# Custom Attributes — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Custom attributes in HTML are author-defined attributes that extend the standard attribute set to store application-specific data, metadata, or state on HTML elements. The most common and standards-compliant form is the `data-*` attribute family, which forms a class of attributes called **custom data attributes** that allow proprietary information to be exchanged between HTML and its DOM representation by scripts [6†L3-L9]. Beyond `data-*`, HTML also provides structured custom attribute vocabularies through **Microdata** (`itemscope`, `itemtype`, `itemprop`, `itemid`, `itemref`) and **RDFa** (`vocab`, `typeof`, `property`, `resource`, `prefix`), which serve specialised structured-data purposes.

**Technical Definition**
In the WHATWG HTML Living Standard, custom data attributes are defined as attributes in no namespace whose name starts with `data-`, has at least one character after the hyphen, is XML-compatible, and contains no uppercase ASCII letters [6†L23-L29]. Every HTML element may have any number of custom data attributes specified, with any value [0†L28-L33]. These attributes are exposed via the `HTMLElement.dataset` property, which returns a `DOMStringMap` object, and via standard `getAttribute()`/`setAttribute()` methods [6†L19-L23]. Microdata attributes are defined in the HTML Microdata specification and provide a data model of items and properties, while RDFa attributes are defined by the W3C RDFa Core 1.1 and RDFa Lite 1.1 specifications. All three mechanisms are **global attributes** — they may be specified on any HTML element.

**Beginner-Friendly Explanation**
Custom attributes are your own personal labels that you can stick onto any HTML element to store extra information. The browser will not display this information and will not do anything with it on its own — but your JavaScript can read it, update it, and use it to make decisions. The standard, safe way to create your own attributes is to start their names with `data-`. For example, `<li data-id="42">` stores the ID `42` on a list item. There are also specialised custom attribute systems called Microdata and RDFa that are used to tell search engines what your content means — like “this is a product” or “this is a person” — but those are for structured data rather than general-purpose application data.

---

### Key Characteristics

- **Extensible without breaking standards**: `data-*` attributes are the official, standards-compliant way to add custom attributes to HTML elements [6†L3-L9].
- **Namespaced by prefix**: The `data-` prefix acts as a namespace that prevents collisions with current and future standard HTML attributes [1†L36-L37].
- **String-valued in the DOM**: All custom data attributes are stored as strings; numeric or boolean data must be parsed in JavaScript [8†L26-L27].
- **Script access via `dataset`**: The `dataset` property provides convenient camelCase access to all `data-*` attributes on an element [6†L19-L23].
- **CSS selectability**: Data attributes can be targeted with CSS attribute selectors and read with the `attr()` function [8†L34-L45].
- **Multiple mechanisms**: HTML provides three structured custom-attribute systems — `data-*` for application data, Microdata for search-engine structured data, and RDFa for linked-data-oriented structured data.
- **No accessibility semantics**: Custom attributes are ignored by screen readers; they must not be used to convey information essential to understanding the page.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, attributes, and the DOM.
2. **JavaScript fundamentals**: variables, objects, property access, and type conversion (because `dataset` values are strings).
3. **CSS attribute selectors**: understanding `[attr]` and `[attr="value"]` syntax.
4. **Awareness of accessibility**: knowing that custom attributes are invisible to assistive technologies.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **JavaScript / DOM scripting** | The primary consumer of custom attributes; `dataset` provides read/write access. |
| **CSS** | Attribute selectors and the `attr()` function can read data attributes for styling and generated content. |
| **Structured data / SEO** | Microdata and RDFa use custom attributes to annotate content for search engines. |
| **Web Components** | Custom elements frequently use `data-*` attributes to store state and configuration. |
| **Testing and automation** | `data-testid` and similar attributes are widely used as stable selectors for end-to-end tests. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Custom attributes** — the general concept
2. **`data-*`** — the standard custom data attribute mechanism
3. **Microdata attributes** — `itemscope`, `itemtype`, `itemprop`, `itemid`, `itemref`
4. **RDFa attributes** — `vocab`, `typeof`, `property`, `resource`, `prefix`
5. **Attribute naming** — rules and recommendations
6. **Attribute usage** — access patterns and anti-patterns
7. **Custom attribute best practices** — quality, security, and performance guidelines

---

## 1. Custom Attributes — The General Concept

### Definitions

**Core Definition**
A custom attribute is any author-defined HTML attribute that is not part of the standard HTML vocabulary. The standards-compliant form is the `data-*` attribute, which allows proprietary information to be stored on any element [6†L3-L9].

**Technical Definition**
HTML is designed with extensibility in mind for data that should be associated with a particular element but need not have any defined meaning [8†L3-L5]. The `data-*` global attributes form the official custom data attribute class. Attributes without the `data-` prefix that are not standardised are non-conforming, even though browsers may still parse them. Microdata and RDFa provide structured custom-attribute vocabularies with their own attribute names and semantics.

**Beginner-Friendly Explanation**
Custom attributes are the “extra fields” you add to HTML tags when the standard attributes (`id`, `class`, `href`, etc.) are not enough. For example, you might store a product ID, a user ID, or a state flag on an element so that your JavaScript can read it later.

---

### Purposes

- **To store application-specific data** on elements without inventing non-standard attributes [8†L3-L8].
- **To provide a clean separation** between styling (`class`), structure (semantic elements), and application data.
- **To enable state encoding** on elements (e.g. `data-state="open"`) for CSS and JavaScript consumption.
- **To serve as stable test hooks** (`data-testid`) for automated testing frameworks.
- **To avoid abusing `class` or `id`** for data storage that has no presentational or identity meaning [4†L8-L10].

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<any-element data-custom-name="value">
```

**Component breakdown**

| Component | Description |
|---|---|
| `data-` | The required prefix for standard custom data attributes. |
| `custom-name` | The author-chosen name, following XML naming rules. |
| `="value"` | The string value; may be empty. |

#### Syntax Rules

1. **Every HTML element may have any number of custom data attributes**, with any value [0†L28-L33].
2. **Attribute names must be XML-compatible**, must not contain uppercase ASCII letters, must not start with `xml` (case-insensitive), and must not contain colons [6†L23-L29].
3. **The portion after `data-` must contain at least one character**.
4. **Values are always strings in the DOM**; JavaScript must convert them when needed [8†L26-L27].
5. **Non-standard custom attributes** (without `data-`) are non-conforming and should be avoided [4†L13-L15].

#### Constraints and Limitations

- **Not for public metadata**: custom data attributes are private to the page; external software should not rely on them.
- **Not for accessibility**: screen readers ignore them; use ARIA and native semantics for accessibility.
- **Not for sensitive data**: values are visible in the DOM and in page source.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Storing Application Data on List Items

**Setup Guide**: Save as `custom-basic.html` and open in a browser. Inspect the elements with developer tools.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Custom Attribute Basics</title>
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

**Why this output occurs**: The `data-*` attributes store private application data on each list item. The browser renders only the text content; the attributes remain in the DOM for scripts to read [5†L13-L16].

---

#### Example 2: Multiple Custom Attributes on One Element

**Setup Guide**: Save as `custom-multiple.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Multiple Custom Attributes</title>
</head>
<body>
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

**Why this output occurs**: The specification allows any number of custom data attributes on any element, with any value [0†L28-L33].

---

### Real-World Cases

1. **E-commerce product listings**: `data-product-id`, `data-price`, and `data-category` store product metadata for client-side filtering and cart logic.
2. **Interactive UI state**: `data-state="open"` on accordions and `data-loading` on buttons drive CSS and JavaScript behaviour.
3. **Analytics and tracking**: `data-track="cta-click"` labels elements for event tracking without adding extra classes or IDs.

---

### References

- MDN Web Docs — `data-*` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*
- W3C — HTML 5.2: Custom data attributes — https://www.w3.org/TR/html52/dom.html#custom-data-attribute
- WHATWG — Custom data attributes — https://html.spec.whatwg.org/multipage/dom.html#custom-data-attribute

---

## 2. `data-*` — The Standard Custom Data Attribute Mechanism

### Definitions

**Core Definition**
The `data-*` global attributes form a class of attributes called **custom data attributes** that allow proprietary information to be exchanged between the HTML and its DOM representation by scripts [6†L3-L9].

**Technical Definition**
`data-*` attributes are global attributes whose names begin with `data-` followed by at least one character. All such custom data are available via the `HTMLElement` interface of the element, specifically through the `HTMLElement.dataset` property, which exposes a `DOMStringMap` [6†L19-L23]. The portion of the attribute name after `data-` is converted from kebab-case to camelCase for `dataset` property access [6†L44-L46].

**Beginner-Friendly Explanation**
`data-*` is the official “custom attribute” system in HTML. You write `data-` followed by your own name, and the browser stores the value. Your JavaScript reads it through `element.dataset`.

---

### Purposes

- **To store private, script-readable data** on HTML elements without inventing non-standard attributes [8†L3-L8].
- **To provide a clean separation** between styling and application data [4†L8-L10].
- **To enable CSS-driven state** via attribute selectors [8†L34-L45].
- **To serve as stable testing hooks** (`data-testid`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<any-element data-custom-name="value">
```

#### Access Syntax (JavaScript)

```javascript
const value = element.dataset.camelCaseName;
element.dataset.camelCaseName = 'new value';
```

#### Access Syntax (CSS)

```css
[data-custom-name="value"] { /* styles */ }
```

**Component breakdown**

| Access method | Description |
|---|---|
| `dataset.camelCaseName` | CamelCase property access; reflects to the attribute [6†L44-L46]. |
| `getAttribute('data-custom-name')` | Full attribute name access; returns a string. |
| `[data-custom-name]` | CSS attribute existence selector. |

#### Syntax Rules

1. **`dataset` converts kebab-case to camelCase**: `data-foo-bar` → `dataset.fooBar` [6†L44-L46].
2. **`dataset` values are strings**; type conversion is the author’s responsibility [8†L26-L27].
3. **Avoid non-alphabetic characters following a hyphen** (e.g. `data-test-1`) if `dataset` access is needed [7†L40-L42].
4. **CSS attribute selectors are case-sensitive for values** but case-insensitive for attribute names.

#### Constraints and Limitations

- **`dataset` does not expose attributes whose names do not follow camelCase conversion rules**.
- **Data attributes are not accessible to screen readers**; they must not carry essential information.
- **Custom attributes without `data-` are non-conforming** [4†L13-L15].

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading and Writing with `dataset`

**Setup Guide**: Save as `data-dataset.html`. Open the console.

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
  </script>
</body>
</html>
```

**Expected output**: The console logs `"3"`, `"12314"`, `"cars"`, then `"5"` after the write.

**Why this output occurs**: `dataset.columns` reads `data-columns`; `dataset.indexNumber` reads `data-index-number` through camelCase conversion; setting `dataset.columns` updates the underlying attribute [6†L19-L23].

---

#### Example 2: CSS Attribute Selectors and `attr()`

**Setup Guide**: Save as `data-css.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Access to Data Attributes</title>
  <style>
    .metric::after {
      content: " (" attr(data-value) ")";
      color: gray;
    }
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

**Why this output occurs**: `attr(data-value)` reads the attribute value into generated content, and `[data-state="active"]` selects the element by its data attribute value [8†L34-L45].

---

### Real-World Cases

1. **UI state toggling**: `element.dataset.state = 'open'` updates both the attribute and CSS styling.
2. **Form validation feedback**: `data-error="true"` triggers error styles via `[data-error="true"]`.
3. **Testing**: `element.dataset.testid` provides a stable selector hook independent of CSS classes.

---

### References

- MDN Web Docs — `data-*` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*
- MDN Web Docs — `HTMLElement.dataset` — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset
- MDN Web Docs — Using data attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/Use_data_attributes

---

## 3. Microdata Attributes — `itemscope`, `itemtype`, `itemprop`, `itemid`, `itemref`

### Definitions

**Core Definition**
Microdata is a WHATWG HTML specification mechanism for nesting machine-readable metadata within existing visible content, using five global attributes: `itemscope`, `itemtype`, `itemprop`, `itemid`, and `itemref`. It allows authors to annotate content with vocabulary-defined items and properties for search engines and crawlers.

**Technical Definition**
Microdata provides a data model of **items** (groups of name-value pairs) and **properties** (individual name-value pairs) [0†L28-L33]. `itemscope` creates an item; `itemtype` declares the vocabulary; `itemprop` adds properties; `itemid` provides a global identifier; and `itemref` associates non-descendant properties. These attributes are defined in the WHATWG HTML Living Standard and are consumed by search engines supporting Schema.org vocabulary.

**Beginner-Friendly Explanation**
Microdata lets you label your visible content so search engines understand it. You mark a block as an item (`itemscope`), say what kind of thing it is (`itemtype`), and label its pieces (`itemprop`). It is mainly used for SEO and rich search results.

---

### Purposes

- **To annotate visible content** with machine-readable metadata for search engines.
- **To support rich search results** such as product ratings, event dates, and recipe details.
- **To provide a vocabulary-driven data model** using Schema.org types and properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemtype="https://schema.org/Person">
  <span itemprop="name">Jane Doe</span>
</div>
```

**Component breakdown**

| Attribute | Role |
|---|---|
| `itemscope` | Creates an item and defines property scope. |
| `itemtype` | Declares the vocabulary (absolute URL). |
| `itemprop` | Adds a property (name-value pair). |
| `itemid` | Provides a global identifier (requires `itemscope` and `itemtype`). |
| `itemref` | Associates non-descendant properties via IDs. |

#### Syntax Rules

1. **`itemscope` is boolean**: presence creates an item.
2. **`itemtype` requires `itemscope`** and takes an absolute URL.
3. **`itemprop` adds properties**; values come from text content or URL-bearing attributes.
4. **`itemid` requires both `itemscope` and `itemtype`**.
5. **`itemref` takes a space-separated list of IDs** and requires `itemscope`.

#### Constraints and Limitations

- **Microdata does not change visual rendering**; it only adds machine-readable semantics.
- **Browser DOM API support is inconsistent**; search engines still consume the attributes.
- **JSON-LD is now Google’s recommended format**, though microdata remains supported.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Person with Nested Organisation

**Setup Guide**: Save as `microdata.html` and test with Google’s Rich Results Test.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Microdata Example</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Person">
    <span itemprop="name">Jane Doe</span>
    <span itemprop="jobTitle">Software Engineer</span>

    <!-- Nested item for the employer -->
    <div itemprop="worksFor" itemscope itemtype="https://schema.org/Organization">
      <span itemprop="name">Acme Corp</span>
    </div>
  </div>
</body>
</html>
```

**Expected output**: A Person item with `name` and `jobTitle`, plus a nested Organization item for the employer.

**Why this output occurs**: `itemscope` creates the Person item; `itemtype` declares the vocabulary; `itemprop` adds properties; the nested `itemscope` creates a sub-item for the organisation.

---

### Real-World Cases

1. **Product pages**: Marking up product name, price, availability, and reviews.
2. **Event listings**: Marking up event names, dates, and locations.
3. **Recipe pages**: Marking up ingredients, cooking times, and ratings.

---

### References

- MDN Web Docs — Using microdata in HTML — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Microdata
- WHATWG — Microdata — https://html.spec.whatwg.org/multipage/microdata.html
- Schema.org — Getting started with schema.org using Microdata — https://schema.org/docs/gs.html

---

## 4. RDFa Attributes — `vocab`, `typeof`, `property`, `resource`, `prefix`

### Definitions

**Core Definition**
RDFa (Resource Description Framework in Attributes) is a W3C specification that embeds RDF-based structured data in HTML using attributes. The five RDFa Lite attributes — `vocab`, `typeof`, `property`, `resource`, and `prefix` — cover the majority of simple authoring needs.

**Technical Definition**
RDFa Core 1.1 defines the syntax for embedding RDF triples in markup [W3C RDFa Core]. RDFa Lite 1.1 is a five-attribute subset that is fully upward-compatible with full RDFa [W3C RDFa Lite]. `vocab` sets the default vocabulary; `typeof` declares the item type; `property` adds a property; `resource` specifies a resource-valued object; and `prefix` maps CURIE prefixes to IRIs.

**Beginner-Friendly Explanation**
RDFa is another way to add structured data to HTML. It is more flexible than microdata because it can mix vocabularies, but it is also more complex. RDFa Lite makes the common cases simple.

---

### Purposes

- **To embed structured data** directly in HTML for search engines and Linked Data tools.
- **To support multiple vocabularies** on the same page via `prefix`.
- **To provide a W3C-standard alternative** to microdata and JSON-LD.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div vocab="https://schema.org/" typeof="Person">
  <span property="name">Jane Doe</span>
</div>
```

**Component breakdown**

| Attribute | Role |
|---|---|
| `vocab` | Sets the default vocabulary for the element and descendants. |
| `typeof` | Declares the item type. |
| `property` | Adds a property with a literal or resource value. |
| `resource` | Specifies a resource-valued object. |
| `prefix` | Maps a prefix to an IRI for CURIE shorthand. |

#### Syntax Rules

1. **`vocab` takes an absolute IRI** and applies to descendants.
2. **`typeof` takes a term, CURIE, or IRI**.
3. **`property` takes a term, CURIE, or IRI**; values are literals or resources.
4. **`resource` takes a URI or CURIE** and sets the object of a property.
5. **`prefix` takes whitespace-separated `prefix: IRI` pairs**.

#### Constraints and Limitations

- **RDFa is more complex than microdata** for simple cases.
- **Google now recommends JSON-LD** as the preferred structured-data format.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: RDFa Lite Person

**Setup Guide**: Save as `rdfa.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>RDFa Example</title>
</head>
<body>
  <div vocab="https://schema.org/" typeof="Person">
    <span property="name">Jane Doe</span>
    <span property="jobTitle">Software Engineer</span>
  </div>
</body>
</html>
```

**Expected output**: A Person item with `name` and `jobTitle` properties, without any prefix declarations.

**Why this output occurs**: `vocab` establishes Schema.org as the default vocabulary; `typeof` declares the item type; `property` adds the name-value pairs.

---

### Real-World Cases

1. **Linked Data publishing**: Government and research sites using RDFa for semantic web data.
2. **Schema.org deployment**: RDFa Lite with Schema.org vocabulary for rich search results.
3. **Mixed-vocabulary markup**: Combining FOAF, Dublin Core, and Schema.org terms via `prefix`.

---

### References

- W3C — RDFa Core 1.1 — https://www.w3.org/TR/rdfa-core/
- W3C — RDFa Lite 1.1 — https://www.w3.org/TR/rdfa-lite/
- W3C — RDFa 1.1 Primer — https://www.w3.org/TR/rdfa-primer/

---

## 5. Attribute Naming

### Definitions

**Core Definition**
Attribute naming for custom attributes refers to the rules and recommendations for the portion of the attribute name that follows the `data-` prefix. Names must be XML-compatible and contain no uppercase ASCII letters [6†L23-L29].

**Technical Definition**
The name after `data-` must follow the production rule of XML names, which includes three key recommendations: it should not start with `xml` (case-insensitive, because it is reserved for future XML specifications), should not contain colon characters (`:`), and should not contain capital letters (because XML is all lowercase) [6†L23-L29]. These are recommendations rather than hard parsing errors, but following them ensures interoperability and `dataset` compatibility. If the name does not follow camelCase conversion rules (e.g. `data-test-1`), it will not be accessible via `dataset` [7†L40-L42].

**Beginner-Friendly Explanation**
The name you choose after `data-` should be all lowercase, should not start with “xml”, and should not contain colons. Hyphens are allowed and are converted to camelCase when you access the attribute through `dataset`. For example, `data-user-name` becomes `dataset.userName`.

---

### Purposes

- **To prevent collisions** with standard HTML attributes and future specifications [4†L36-L37].
- **To ensure `dataset` compatibility** through predictable camelCase conversion [6†L44-L46].
- **To maintain XML compatibility** for documents processed by XML tools [6†L23-L29].
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
| Case | No uppercase ASCII letters [6†L27-L29]. |
| Reserved prefix | Must not start with `xml` (case-insensitive) [6†L26-L27]. |
| Colon | Must not contain `:` [6†L27-L28]. |
| XML compatibility | Must follow XML `Name` production rules. |

#### Syntax Rules

1. **No uppercase ASCII letters**: `data-userId` is discouraged; `data-user-id` is correct.
2. **No leading `xml`**: `data-xml-data` is discouraged because `xml` is reserved.
3. **No colons**: `data-foo:bar` is discouraged because XML assigns meaning to colons.
4. **Hyphens are converted to camelCase in `dataset`**: `data-user-name` → `dataset.userName` [6†L44-L46].
5. **Avoid digits immediately after a hyphen if `dataset` access is needed**: `data-test-1` is not reliably accessible via `dataset` [7†L40-L42].

#### Constraints and Limitations

- **Non-conforming names still work** in browsers and CSS but may behave unpredictably with `dataset` [6†L30-L35].
- **The portion after `data-` must be a valid JavaScript property name segment** if `dataset` access is required [7†L32-L35].

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recommended vs Discouraged Naming

**Setup Guide**: Save as `naming-rules.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Custom Attribute Naming</title>
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

**Why this output occurs**: `dataset` converts `data-track-length` to `trackLength` by removing hyphens and capitalising the following letter [6†L44-L46].

---

### Real-World Cases

1. **Design systems**: Consistent naming conventions like `data-state`, `data-variant`, and `data-size` across components.
2. **Testing**: `data-testid` is a widely adopted convention for stable test selectors.
3. **Analytics**: `data-track-category` and `data-track-action` follow predictable naming for event systems.

---

### References

- MDN Web Docs — `data-*`: Naming recommendations — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/data-*
- W3C — XML Name production — https://www.w3.org/TR/xml/#NT-Name

---

## 6. Attribute Usage

### Definitions

**Core Definition**
Custom attribute usage refers to the patterns, conventions, and anti-patterns associated with storing and consuming data in custom attributes. Correct usage respects the specification’s intent that custom data attributes hold private, non-visible, script-oriented data [8†L3-L8].

**Technical Definition**
Custom attributes are accessed via `element.dataset` (for `data-*`), `getAttribute()`/`setAttribute()`, and CSS attribute selectors. They are intended for data private to the page or application, for which there are no more appropriate attributes or elements [0†L30-L32]. They are **not** intended for public metadata, accessibility semantics, or sensitive data. Microdata and RDFa attributes follow their own usage models for structured data.

**Beginner-Friendly Explanation**
Use custom attributes for “behind-the-scenes” data that your own JavaScript needs. Do not use them for information that users should see, for accessibility labels, or for secrets. Think of them as private notes you write on an element for your scripts to read later.

---

### Purposes

- **To store application state** on elements without polluting `class` or `id`.
- **To provide configuration** for JavaScript widgets and components.
- **To enable data-driven styling** through CSS attribute selectors [8†L34-L45].
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

1. **Use custom attributes for private, non-visible data** that scripts consume [8†L3-L8].
2. **Do not use custom attributes for accessibility information**; use ARIA and native semantics.
3. **Do not store sensitive data**; attributes are visible in the DOM and page source.
4. **Keep values small**; custom attributes are not a substitute for a database or state management library.
5. **Use JSON strings for complex data**, but parse defensively in JavaScript.

#### Constraints and Limitations

- **No accessibility semantics**: screen readers ignore custom attributes, so they cannot replace `aria-*` or visible labels.
- **No public metadata contract**: external software should not depend on your custom attribute names [0†L30-L32].
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

**Expected output**: Clicking the button toggles the panel between visible and hidden.

**Why this output occurs**: The `data-state` attribute drives CSS selectors; JavaScript updates `dataset.state`, which reflects to the attribute and triggers the CSS changes [8†L34-L45].

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

- MDN Web Docs — Using data attributes — https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/Use_data_attributes
- W3C — HTML 5.1: Custom data attributes — https://www.w3.org/TR/html51/dom.html#custom-data-attribute

---

## 7. Custom Attribute Best Practices

### Definitions

**Core Definition**
Custom attribute best practices are guidelines for using custom attributes in ways that are semantically appropriate, accessible, secure, performant, and maintainable.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, the W3C HTML specification, accessibility guidance (WCAG and ARIA), and security principles. They encompass correct scoping of custom attributes to private application data, avoiding accessibility-critical information, preventing sensitive data exposure, choosing appropriate access mechanisms (`dataset` vs `getAttribute()`), and maintaining performance with CSS selectors and DOM size.

**Beginner-Friendly Explanation**
Best practices help you use custom attributes for the right purposes and avoid common traps like putting secrets in the DOM, relying on custom attributes for accessibility, or creating naming collisions.

---

### Purposes

- **To ensure custom attributes are used for their intended purpose** — private script data [8†L3-L8].
- **To protect accessibility** by keeping essential information out of custom attributes.
- **To protect security** by never storing secrets or sensitive personal data in the DOM.
- **To maintain performance** by keeping attribute counts and selector complexity reasonable.
- **To improve maintainability** through consistent naming and access patterns [4†L8-L10].

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `data-*` for custom data** | The `data-` prefix is the standards-compliant way to create custom attributes [4†L13-L15]. |
| **Never store sensitive data** | Passwords, tokens, and personal data must not appear in the DOM. |
| **Use `data-testid` for test hooks** | Prefer dedicated test attributes over abusing `class` or `id`. |
| **Prefer `dataset` for camelCase access** | Use `getAttribute()` when you need the raw attribute name or broader compatibility [8†L15-L26]. |
| **Validate and convert types explicitly** | `dataset` values are strings; convert to numbers or booleans explicitly [8†L26-L27]. |
| **Handle missing attributes** | Check `'prop' in element.dataset` or use optional chaining. |
| **Keep values small and simple** | Avoid storing large JSON blobs or binary data in attributes. |
| **Avoid non-alphabetic characters after hyphens** | `data-test-1` is not reliably accessible via `dataset` [7†L40-L42]. |
| **Use JSON sparingly and parse defensively** | Always wrap `JSON.parse` in `try…catch`. |
| **Do not replace ARIA with custom attributes** | Use `aria-*` and native semantics for accessibility. |
| **Include library names in attribute names** | For reusable libraries, reduce clash risk by including the library name [1†L36-L41]. |

#### Syntax Rules

1. **Scope**: Custom attributes are for the page’s own scripts, not for external consumers [0†L30-L32].
2. **Accessibility**: Never rely on custom attributes to convey information to assistive technologies.
3. **Security**: Treat every custom attribute value as publicly visible.
4. **Naming**: Follow the lowercase, XML-compatible naming recommendations [6†L23-L29].
5. **Access**: Use `dataset` for camelCase convenience; use `getAttribute()` for exact names.

#### Constraints and Limitations

- **`dataset` does not expose all attribute names**; non-conforming names may be inaccessible [7†L40-L42].
- **CSS attribute selectors can be slower** than class or ID selectors, though the difference is usually negligible in practice [9†L13-L15].
- **Custom attributes are not a state management solution**; complex application state should live in JavaScript memory or a dedicated store.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe and Accessible Custom Attribute Usage

**Setup Guide**: Save as `best-practice-custom.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Custom Attributes</title>
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

**Why this output occurs**: `aria-label` provides the accessible name; `data-testid` is a private test hook; `data-state` is private UI state. The anti-patterns show common mistakes: using a custom attribute for an accessible name and storing a secret in the DOM.

---

#### Example 2: Type Conversion and Missing Attribute Handling

**Setup Guide**: Save as `best-practice-conversion.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Custom Attribute Conversions</title>
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
- W3C — WCAG 2.2 — https://www.w3.org/WAI/standards-guidelines/wcag/

---

## Summary Table of Custom Attribute Mechanisms

| Mechanism | Prefix / Attributes | Primary Use | Access |
|---|---|---|---|
| **`data-*`** | `data-` prefix | Private application data, state, configuration | `dataset`, `getAttribute()`, CSS selectors |
| **Microdata** | `itemscope`, `itemtype`, `itemprop`, `itemid`, `itemref` | Search-engine structured data | Parsers, DOM API (limited) |
| **RDFa** | `vocab`, `typeof`, `property`, `resource`, `prefix` | Linked Data, structured data | RDFa processors |

---

## Final Notes

- **`data-*` is the standard way to create custom attributes**: It is standards-compliant, widely supported, and integrates cleanly with the DOM and CSS [6†L3-L9].
- **Custom attributes without `data-` are non-conforming**: They may work in browsers but are not valid HTML and risk future collisions [4†L13-L15].
- **Microdata and RDFa are specialised structured-data mechanisms**: They serve SEO and Linked Data purposes rather than general application data.
- **Accessibility and security boundaries are critical**: Custom attributes must never carry accessibility semantics or sensitive data.
- **Validate and test**: Use HTML validators and accessibility testing tools to catch misuse of custom attributes.

---

This cheat sheet has covered custom attributes — the general concept, `data-*`, Microdata attributes, RDFa attributes, naming rules, usage patterns, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate custom attribute usage against the WHATWG HTML Living Standard, W3C specifications, and accessibility guidelines.