# Microdata — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Microdata is a WHATWG HTML specification mechanism for nesting machine-readable metadata within existing visible content on web pages. It allows authors to annotate HTML elements with name-value pairs, called **items** and **properties**, that search engines and web crawlers can extract and process to provide richer browsing experiences .

**Technical Definition**
Microdata is an HTML extension that provides a declarative syntax for annotating DOM elements with structured data. It defines a data model of **items** (groups of name-value pairs) and **properties** (individual name-value pairs), represented by regular HTML elements carrying specific global attributes: `itemscope`, `itemtype`, `itemprop`, `itemid`, and `itemref`. These attributes are parsed by user agents and exposed through the `HTMLElement` microdata DOM API. Microdata is designed to be simpler than RDFa and classic microformats while remaining fully integrated into HTML .

**Beginner-Friendly Explanation**
Microdata lets you add hidden labels to the content people see on your page. You mark up a product name, a price, or an event date with attributes that tell search engines “this is the name,” “this is the price,” and “this is the date.” The visible content stays the same, but machines can now understand what each piece of information means.

---

### Key Characteristics

- **Visible-content annotation**: Microdata annotates existing, visible HTML content — it does not create hidden metadata blocks .
- **Vocabulary-driven**: Microdata uses external vocabularies (most commonly Schema.org) to define the meaning of item types and property names .
- **Attribute-based syntax**: The five microdata attributes — `itemscope`, `itemtype`, `itemprop`, `itemid`, and `itemref` — are global attributes usable on any HTML element.
- **Nested items**: Items can contain other items, allowing complex hierarchical data structures .
- **Global identifiers**: `itemid` provides a unique global identifier for an item, enabling cross-document references .
- **Non-descendant property association**: `itemref` allows properties located elsewhere in the document to be associated with an item .

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, attributes, and the DOM.
2. **The concept of structured data**: understanding why machines need explicit labels for content.
3. **A vocabulary (e.g. Schema.org)**: knowing the types and properties you want to use.
4. **An HTML validator or structured-data testing tool**: such as Google’s Rich Results Test.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **SEO / Rich results** | Major search engines use microdata to generate rich search results . |
| **Schema.org** | The dominant vocabulary for microdata markup, supported by Google, Microsoft, Yahoo, and Yandex . |
| **JSON-LD** | An alternative structured-data format that is now Google’s recommended approach . |
| **RDFa** | A more expressive but more complex structured-data syntax that competes with microdata . |
| **Accessibility** | Microdata does not directly affect accessibility; it is purely machine-oriented metadata. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Microdata** — the specification
2. `itemscope` — item creation
3. `itemtype` — vocabulary declaration
4. `itemprop` — property naming
5. `itemid` — global identifier
6. `itemref` — non-descendant property association
7. **Microdata best practices**

---

## 1. Microdata — The Specification

### Definitions

**Core Definition**
Microdata is a set of HTML attributes that allows authors to add machine-readable labels to visible content, enabling search engines and crawlers to extract structured information from web pages .

**Technical Definition**
Microdata is defined in the WHATWG HTML Living Standard. It provides a data model of items and properties, and a set of global attributes for encoding that model into HTML. The model supports nested items, typed items via vocabularies, global identifiers, and property association via references. Microdata is parsed by user agents and exposed through the `HTMLElement` microdata DOM API, though this API is not implemented in all browsers .

**Beginner-Friendly Explanation**
Microdata is the “labelling system” built into HTML. You use it to tell search engines what your content means — “this is a person,” “this is a book,” “this is the price” — without changing what visitors see.

---

### Purposes

- **To enable search engines to understand page content** and provide richer search results.
- **To annotate visible content** with machine-readable name-value pairs.
- **To support nested data structures** for complex items such as products with reviews.
- **To provide a declarative, HTML-native alternative** to RDFa and microformats.

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
| `itemscope` | Creates an item. |
| `itemtype` | Declares the vocabulary type. |
| `itemprop` | Adds a property to an item. |

#### Syntax Rules

1. **To create an item, use `itemscope`** on a container element.
2. **To add a property, use `itemprop`** on a descendant of the item.
3. **To declare the vocabulary, use `itemtype`** with an absolute URL.
4. **Properties not in the DOM subtree can be associated with `itemref`**.

#### Constraints and Limitations

- **Microdata does not change visual rendering**; it only adds machine-readable semantics.
- **Browser DOM API support is inconsistent**; the attributes still work for search engines.
- **JSON-LD is now Google’s recommended format**, though microdata remains supported .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Two Simple Items

**Setup Guide**: Save as `microdata-basic.html` and test with Google’s Rich Results Test.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Microdata Basics</title>
</head>
<body>
  <!-- Item 1: a person -->
  <div itemscope>
    <p>My name is <span itemprop="name">Elizabeth</span>.</p>
  </div>

  <!-- Item 2: another person -->
  <div itemscope>
    <p>My name is <span itemprop="name">Daniel</span>.</p>
  </div>
</body>
</html>
```

**Expected output**: Two separate items, each with a `name` property (`Elizabeth` and `Daniel`). The visible page content is unchanged.

**Why this output occurs**: `itemscope` creates an item on each `<div>`. The `itemprop="name"` span inside each item adds a property whose value is the text content of the span .

---

#### Example 2: A Person with Multiple Properties

**Setup Guide**: Save as `microdata-person.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Person Microdata</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Person">
    <span itemprop="name">Jane Doe</span>
    <span itemprop="jobTitle">Software Engineer</span>
    <a itemprop="url" href="https://example.com/jane">Profile</a>
  </div>
</body>
</html>
```

**Expected output**: A Person item with three properties: `name`, `jobTitle`, and `url`.

**Why this output occurs**: `itemtype` declares the Schema.org Person vocabulary, and each `itemprop` adds a named property within that vocabulary .

---

### Real-World Cases

1. **Product pages**: Marking up product name, price, availability, and reviews for rich search results.
2. **Event listings**: Marking up event names, dates, and locations.
3. **Recipe pages**: Marking up ingredients, cooking times, and ratings.

---

### References

- MDN Web Docs — Using microdata in HTML — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Microdata
- WHATWG — Microdata — https://html.spec.whatwg.org/multipage/microdata.html
- Schema.org — Getting started with schema.org using Microdata — https://schema.org/docs/gs.html

---

## 2. `itemscope` — Item Creation

### Definitions

**Core Definition**
`itemscope` is a boolean global attribute that creates a new microdata item and defines the scope of the associated `itemtype` .

**Technical Definition**
`itemscope` is a boolean attribute. When specified, it indicates that the element is the container for an item — a group of name-value pairs. The item’s properties are provided by `itemprop` attributes on descendants (or on elements referenced by `itemref`). An element with `itemscope` may also carry `itemtype` to specify the vocabulary and `itemid` to provide a global identifier .

**Beginner-Friendly Explanation**
`itemscope` marks the beginning of a “thing” you are describing. Everything inside it (or referenced by `itemref`) belongs to that thing.

---

### Purposes

- **To create a microdata item** on an element.
- **To define the scope** within which `itemprop` properties belong to the item.
- **To enable nesting** of items within items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope>
  <span itemprop="name">Item name</span>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemscope` | Boolean attribute; presence creates an item. |

#### Syntax Rules

1. **`itemscope` is boolean**: presence means true.
2. **The item’s properties come from descendants with `itemprop`**.
3. **`itemscope` may be combined with `itemtype` and `itemid`**.

#### Constraints and Limitations

- **`itemref` can only be specified on elements that have `itemscope`** .
- **An element with `itemscope` and no `itemtype` must have an `itemref`** to provide properties .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Item

**Setup Guide**: Save as `itemscope.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemscope Example</title>
</head>
<body>
  <div itemscope>
    <p>My name is <span itemprop="name">Elizabeth</span>.</p>
  </div>
</body>
</html>
```

**Expected output**: An item with a single `name` property whose value is “Elizabeth”.

**Why this output occurs**: `itemscope` creates the item; the `itemprop` span adds a property within its scope .

---

### Real-World Cases

1. **Simple metadata blocks**: Describing a single entity on a page.
2. **Nested items**: A product item containing a nested brand item.
3. **Ad hoc vocabularies**: Using `itemscope` without `itemtype` for custom data.

---

### References

- MDN Web Docs — `itemscope` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemscope
- WHATWG — Items — https://html.spec.whatwg.org/multipage/microdata.html#items

---

## 3. `itemtype` — Vocabulary Declaration

### Definitions

**Core Definition**
`itemtype` specifies the URL of the vocabulary that defines the item’s type and the meaning of its `itemprop` property names .

**Technical Definition**
`itemtype` takes an absolute URL that identifies a vocabulary. When present, it must be specified on an element that also has `itemscope`. The vocabulary defines which property names are valid and what they mean. Multiple types are not allowed in the current specification, though some implementations tolerate space-separated values .

**Beginner-Friendly Explanation**
`itemtype` tells the parser which “dictionary” to use. For example, `itemtype="https://schema.org/Person"` says “use the Schema.org Person vocabulary.”

---

### Purposes

- **To declare the vocabulary** for an item.
- **To give meaning to `itemprop` names**.
- **To enable vocabulary-aware processing** by search engines.

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
| `itemtype="https://schema.org/Person"` | Absolute URL of the vocabulary. |

#### Syntax Rules

1. **`itemtype` requires `itemscope` on the same element**.
2. **The value must be an absolute URL**.
3. **The vocabulary must define the property names used in `itemprop`**.

#### Constraints and Limitations

- **Only one vocabulary URL is officially supported**; multiple types are not allowed in the current spec .

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

**Expected output**: A Book item with `name` and `author` properties.

**Why this output occurs**: `itemtype` declares the Schema.org Book vocabulary, and the `itemprop` names are interpreted within that vocabulary .

---

### Real-World Cases

1. **Product markup**: `itemtype="https://schema.org/Product"`.
2. **Review markup**: `itemtype="https://schema.org/Review"`.
3. **Event markup**: `itemtype="https://schema.org/Event"`.

---

### References

- MDN Web Docs — `itemtype` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemtype
- WHATWG — Typed items — https://html.spec.whatwg.org/multipage/microdata.html#typed-items

---

## 4. `itemprop` — Property Naming

### Definitions

**Core Definition**
`itemprop` adds a property to one or more items. It consists of a name-value pair, where the name is the attribute value and the value is derived from the element’s content or specific attributes .

**Technical Definition**
`itemprop` is a global attribute whose value is an unordered set of unique space-separated tokens representing property names. Each token must be either a valid absolute URL or, for typed items, a defined property name from the vocabulary. Property values are either strings or URLs; URL values are expressed using elements such as `<a>`, `<img>`, `<link>`, `<audio>`, `<video>`, `<iframe>`, `<embed>`, `<object>`, `<source>`, and `<track>` .

**Beginner-Friendly Explanation**
`itemprop` labels a piece of content as a specific property. For example, `<span itemprop="name">Jane</span>` says “this text is the name.”

---

### Purposes

- **To add a property** to an item.
- **To provide the name-value pair** that forms the core of microdata.
- **To associate values with vocabulary-defined property names**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<span itemprop="name">Jane Doe</span>
<a itemprop="url" href="https://example.com/jane">Profile</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemprop="name"` | The property name. |
| Value source | Element content (string) or specific attributes (URL). |

#### Syntax Rules

1. **The value is the property name** (or space-separated names).
2. **Must be on a descendant of an item** (or referenced via `itemref`).
3. **URL values use URL-bearing elements** (`<a>`, `<img>`, `<link>`, etc.) .
4. **String values use the element’s text content**.

#### Constraints and Limitations

- **Property names must be defined by the vocabulary** when `itemtype` is present.
- **The same property can appear multiple times**, and values are ordered .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: String and URL Properties

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
    <!-- String value -->
    <span itemprop="name">Jane Doe</span>

    <!-- URL value via <a href> -->
    <a itemprop="url" href="https://example.com/jane">Profile</a>

    <!-- URL value via <img src> -->
    <img itemprop="image" src="jane.jpg" alt="Jane Doe">
  </div>
</body>
</html>
```

**Expected output**: A Person item with `name` (string), `url` (URL from `href`), and `image` (URL from `src`).

**Why this output occurs**: `itemprop` on a `<span>` uses its text content as the value; `itemprop` on `<a>` and `<img>` uses their URL-bearing attributes as the value .

---

### Real-World Cases

1. **Product name and price**: `itemprop="name"` and `itemprop="price"`.
2. **Event dates**: `itemprop="startDate"` on a `<time>` element.
3. **Review ratings**: `itemprop="ratingValue"` on a `<span>`.

---

### References

- MDN Web Docs — `itemprop` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemprop
- WHATWG — Names: the itemprop attribute — https://html.spec.whatwg.org/multipage/microdata.html#names:-the-itemprop-attribute

---

## 5. `itemid` — Global Identifier

### Definitions

**Core Definition**
`itemid` provides a unique, global identifier for a microdata item. It can only be specified on elements that have both `itemscope` and `itemtype` attributes, and only when the vocabulary supports global identifiers .

**Technical Definition**
`itemid` is a global attribute whose value is an absolute URL (or, in practice, a URN is sometimes accepted). The exact meaning of the identifier is defined by the vocabulary. It enables the same item to be referenced across documents .

**Beginner-Friendly Explanation**
`itemid` gives an item a globally unique ID — like an ISBN for a book — so different pages can refer to the same thing.

---

### Purposes

- **To provide a unique global identifier** for an item.
- **To enable cross-document references** to the same entity.
- **To support vocabulary-specific identity requirements**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<dl itemscope itemtype="https://schema.org/Book" itemid="urn:isbn:0-374-22848-5">
  <dd itemprop="title">Owls of the Eastern Ice</dd>
</dl>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemid="urn:isbn:…"` | Unique global identifier. |
| Requires | `itemscope` and `itemtype`. |

#### Syntax Rules

1. **Only allowed with `itemscope` and `itemtype`**.
2. **The vocabulary must support global identifiers**.
3. **The value should be an absolute URL** (though URNs are commonly used) .

#### Constraints and Limitations

- **Not all vocabularies support `itemid`**.
- **The meaning of the identifier is vocabulary-defined**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Book with ISBN

**Setup Guide**: Save as `itemid.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemid Example</title>
</head>
<body>
  <dl itemscope itemtype="https://schema.org/Book" itemid="urn:isbn:0-374-22848-5">
    <dt>Title</dt>
    <dd itemprop="title">Owls of the Eastern Ice</dd>
    <dt>Author</dt>
    <dd itemprop="author">Jonathan C. Slaght</dd>
  </dl>
</body>
</html>
```

**Expected output**: A Book item with the global identifier `urn:isbn:0-374-22848-5` and `title` and `author` properties.

**Why this output occurs**: `itemid` provides the unique global identifier for the Book, enabling it to be referenced from other pages .

---

### Real-World Cases

1. **Books**: ISBN as `itemid`.
2. **Products**: SKU or GTIN as `itemid`.
3. **Organisations**: A canonical URL as `itemid`.

---

### References

- MDN Web Docs — `itemid` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemid
- WHATWG — Global identifiers for items — https://html.spec.whatwg.org/multipage/microdata.html#global-identifiers-for-items

---

## 6. `itemref` — Non-Descendant Property Association

### Definitions

**Core Definition**
`itemref` provides a list of element IDs elsewhere in the document whose `itemprop` descendants should be associated with the item, even though they are not DOM descendants of the `itemscope` element .

**Technical Definition**
`itemref` is a global attribute that can only be specified on elements with `itemscope`. Its value is a space-separated list of IDs. The referenced elements’ `itemprop` descendants become properties of the item. `itemref` is a syntactic aid, not part of the microdata data model .

**Beginner-Friendly Explanation**
`itemref` lets you include properties that live elsewhere on the page. You reference their IDs, and they become part of the item even though they are not inside it.

---

### Purposes

- **To associate non-descendant properties** with an item.
- **To support table layouts** where properties are in separate cells .
- **To keep structured data flexible** when the DOM structure does not match the data structure.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div itemscope itemref="a b"></div>
<p id="a"><span itemprop="name">Amanda</span></p>
<div id="b"><span itemprop="band">Jazz Band</span></div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `itemref="a b"` | Space-separated IDs of elements with additional properties. |

#### Syntax Rules

1. **Only allowed on elements with `itemscope`** .
2. **The value is a space-separated list of IDs**.
3. **Referenced elements must exist in the document**.

#### Constraints and Limitations

- **Circular references must be avoided**.
- **`itemref` is not part of the data model**; it is only a syntactic convenience .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Band Data with Separated Properties

**Setup Guide**: Save as `itemref.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Itemref Example</title>
</head>
<body>
  <div itemscope id="amanda" itemref="a b"></div>

  <p id="a">Name: <span itemprop="name">Amanda</span></p>

  <div id="b" itemprop="band" itemscope itemref="c"></div>

  <div id="c">
    <p>Band: <span itemprop="name">Jazz Band</span></p>
    <p>Size: <span itemprop="size">12</span> players</p>
  </div>
</body>
</html>
```

**Expected output**: An Amanda item with a `name` property (“Amanda”) and a `band` property (a nested item with `name` “Jazz Band” and `size` 12).

**Why this output occurs**: `itemref="a b"` pulls in properties from the referenced elements; the nested item on `#b` uses `itemref="c"` to pull in further properties .

---

### Real-World Cases

1. **Table layouts**: Properties in separate table cells.
2. **Multi-column layouts**: Data split across visual columns.
3. **Complex page structures**: Data not nested in a single container.

---

### References

- MDN Web Docs — `itemref` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/itemref
- WHATWG — Items — https://html.spec.whatwg.org/multipage/microdata.html#items

---

## 7. Microdata Best Practices

### Definitions

**Core Definition**
Microdata best practices are guidelines for using microdata correctly, accurately, and in a way that maximises its benefits for search engines while maintaining compliance with search-engine guidelines.

**Technical Definition**
Best practices are informed by Schema.org documentation, Google’s structured-data guidelines, and the WHATWG specification. They encompass marking up only visible content, using the most specific vocabulary type, matching structured data to visible content, filling required properties, and avoiding over-markup .

**Beginner-Friendly Explanation**
Best practices help you get the most from microdata without breaking search-engine rules or creating misleading data.

---

### Purposes

- **To ensure rich-result eligibility** by following search-engine guidelines.
- **To maintain data accuracy** by marking up only visible content.
- **To avoid penalties** for misleading or exaggerated markup.
- **To maximise interoperability** by using standard vocabularies.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Mark up only visible content** | Do not mark up hidden content; structured data must match what users see . |
| **Use the most specific vocabulary type** | Choose the most precise Schema.org type available . |
| **Fill all required properties** | Missing required properties can make rich results ineligible . |
| **Match structured data to visible content** | Every property must correspond to visible, human-readable content . |
| **Use standard vocabularies** | Schema.org is supported by Google, Microsoft, Yahoo, and Yandex . |
| **Avoid over-marking** | Excessive markup can reduce readability and negatively impact SEO . |
| **Test thoroughly** | Use Google’s Rich Results Test or Schema Markup Validator . |
| **Consider JSON-LD** | Google recommends JSON-LD as the preferred format . |

#### Constraints and Limitations

- **Microdata is less separable from visible markup than JSON-LD**, making maintenance more difficult.
- **Search-engine support for microdata may evolve** as JSON-LD becomes the preferred format.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct and Incorrect Microdata

**Setup Guide**: Save as `best-practice.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Microdata Best Practices</title>
</head>
<body>
  <!-- GOOD: markup matches visible content -->
  <div itemscope itemtype="https://schema.org/Product">
    <span itemprop="name">Wireless Headphones</span>
    <span itemprop="price">$99.99</span>
  </div>

  <!-- BAD: marking up hidden content or exaggerating -->
  <!--
  <div itemscope itemtype="https://schema.org/Product">
    <span itemprop="name">Wireless Headphones</span>
    <meta itemprop="price" content="$1.99"> <!-- price not visible -->
  </div>
  -->
</body>
</html>
```

**Expected output**: The good markup accurately represents visible product data; the bad markup includes a hidden price, which violates Google’s guidelines.

**Why this output occurs**: Google requires structured data to match visible content; hidden or exaggerated data can lead to manual actions .

---

### Real-World Cases

1. **E-commerce product pages**: Accurate price, availability, and review markup.
2. **Recipe blogs**: Marking up ingredients, cooking times, and ratings that match the visible recipe.
3. **Event pages**: Marking up dates and locations that are visible on the page.

---

### References

- Schema.org — Getting started with schema.org using Microdata — https://schema.org/docs/gs.html
- Google Search Central — General structured data guidelines — https://developers.google.com/search/docs/appearance/structured-data/sd-policies
- MDN Web Docs — Using microdata in HTML — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Microdata
- WHATWG — Microdata — https://html.spec.whatwg.org/multipage/microdata.html

---

## Summary Table of Microdata Attributes

| Attribute | Type | Purpose |
|---|---|---|
| `itemscope` | Boolean | Creates an item and defines property scope. |
| `itemtype` | URL | Declares the vocabulary for the item. |
| `itemprop` | Token list | Adds a property to an item. |
| `itemid` | URL/URN | Provides a unique global identifier for the item. |
| `itemref` | ID list | Associates non-descendant properties with the item. |

---

## Final Notes

- **Microdata is a WHATWG standard** and remains supported by major search engines, though JSON-LD is now Google’s recommended format .
- **All microdata attributes are global attributes** and can be used on any HTML element.
- **The microdata DOM API is not universally implemented**, but the attributes themselves are parsed and consumed by search engines.
- **Accuracy is paramount**: structured data must match visible content to avoid penalties and maintain user trust.
- **Always test** with a structured-data validator before deploying microdata markup.

---

This cheat sheet has covered Microdata and its five attributes — `itemscope`, `itemtype`, `itemprop`, `itemid`, and `itemref` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate microdata markup against the WHATWG HTML Living Standard and search-engine structured-data guidelines.