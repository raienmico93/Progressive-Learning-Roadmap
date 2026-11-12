# RDFa — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
RDFa (Resource Description Framework in Attributes) is a W3C specification that provides a set of attributes for embedding structured data directly into HTML, XHTML, and other XML-based markup languages. It allows authors to annotate visible content with machine-readable metadata, expressing relationships between things using the RDF data model .

**Technical Definition**
RDFa Core 1.1 specifies a syntax for attributes that express structured data in any markup language, with the underlying abstract representation being RDF (Resource Description Framework) . RDFa uses a set of attributes — `vocab`, `typeof`, `property`, `resource`, `prefix`, and others such as `about`, `rel`, `rev`, `datatype`, `content`, and `inlist` — to create RDF triples from HTML markup . The RDFa Lite 1.1 profile defines a minimal subset of five attributes — `vocab`, `typeof`, `property`, `resource`, and `prefix` — that covers the majority of simple structured-data authoring needs and is fully upward-compatible with full RDFa 1.1 . RDFa is designed to be parsed without knowledge of the specific vocabulary being used, allowing terms from multiple independently-developed vocabularies to be freely intermixed .

**Beginner-Friendly Explanation**
RDFa is a way to label the content on your web page so that computers — search engines, browsers, and other tools — can understand what each piece of information means. Instead of just showing “Jane Doe” on a page, RDFa lets you say “this text is the name of a person.” It uses special HTML attributes that do not change how the page looks but add machine-readable meaning to what is already visible.

---

### Key Characteristics

- **Attribute-based syntax**: RDFa uses HTML attributes, so structured data is embedded directly in the markup, tightly coupled to the visible content .
- **RDF data model**: RDFa produces RDF triples (subject–predicate–object statements), making it compatible with the broader Linked Data ecosystem .
- **Vocabulary-agnostic**: RDFa specifies only a syntax; it relies on independent vocabularies (such as Schema.org) for the meaning of terms .
- **Multiple vocabularies**: Terms from different vocabularies can be freely intermixed on the same page .
- **Two profiles**: RDFa Core 1.1 (full attribute set) and RDFa Lite 1.1 (five-attribute subset for simpler authoring) .
- **W3C standard**: RDFa is a W3C Recommendation, with the core specification published in 2012 and revised through 2014 .

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, attributes, and the DOM.
2. **The concept of structured data**: understanding why machines need explicit labels for content.
3. **A vocabulary (e.g. Schema.org)**: knowing the types and properties you want to use.
4. **A structured-data testing tool**: such as Google’s Rich Results Test or an RDFa distiller.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Linked Data / Semantic Web** | RDFa is closely aligned with RDF and the Linked Data community . |
| **SEO / Rich results** | Google supports RDFa as one of three structured-data formats (alongside JSON-LD and microdata) . |
| **Schema.org** | RDFa Lite works well with Schema.org terms . |
| **Microdata** | A competing HTML-embedded structured-data syntax with different attribute names (`itemscope`, `itemtype`, `itemprop`) . |
| **JSON-LD** | A separate structured-data format that Google now recommends as the preferred approach . |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **RDFa** — the specification
2. `vocab` — default vocabulary declaration
3. `typeof` — item type declaration
4. `property` — property naming
5. `resource` — object resource specification
6. `prefix` — CURIE prefix mapping
7. **RDFa best practices**

---

## 1. RDFa — The Specification

### Definitions

**Core Definition**
RDFa is a W3C syntax for embedding RDF-based structured data in markup languages, using attributes that do not interfere with the rendered content .

**Technical Definition**
RDFa Core 1.1 defines the syntax and processing rules for RDFa in XML documents, while HTML+RDFa 1.1 extends this to HTML. The specification describes how attributes such as `vocab`, `typeof`, `property`, `resource`, `prefix`, `about`, `rel`, `rev`, `datatype`, `content`, and `inlist` are processed to produce RDF triples . RDFa Lite 1.1 is a five-attribute subset designed to cover simple authoring needs .

**Beginner-Friendly Explanation**
RDFa is the “labelling language” for adding meaning to web content using HTML attributes. It does not change what visitors see, but it tells machines what the content represents.

---

### Purposes

- **To embed structured data** directly in HTML without duplicating content.
- **To enable search engines** to understand page content and provide richer results.
- **To support Linked Data** by producing RDF triples that can be consumed by semantic-web tools.
- **To provide a flexible, vocabulary-agnostic syntax** that works with any RDF vocabulary.

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
| `vocab` | Sets the default vocabulary for the block. |
| `typeof` | Declares the type of the item. |
| `property` | Adds a property to the item. |

#### Syntax Rules

1. **RDFa attributes are allowed on all elements** in the HTML5 content model: `vocab`, `typeof`, `property`, `resource`, and `prefix` (RDFa Lite) .
2. **The full RDFa Core set** also includes `about`, `rel`, `rev`, `datatype`, `content`, and `inlist` .
3. **RDFa attributes do not change visual rendering**; they only add machine-readable semantics.
4. **RDFa Lite is upward-compatible** with full RDFa 1.1 .

#### Constraints and Limitations

- **RDFa is more complex than microdata** for some use cases, though RDFa Lite simplifies it .
- **Google now recommends JSON-LD** as the preferred structured-data format, though RDFa remains supported .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Person in RDFa Lite

**Setup Guide**: Save as `rdfa-person.html` and test with a structured-data validator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>RDFa Person</title>
</head>
<body>
  <!-- vocab sets the default vocabulary; typeof declares the type -->
  <div vocab="https://schema.org/" typeof="Person">
    <!-- property adds a name property -->
    <span property="name">Jane Doe</span>
    <span property="jobTitle">Software Engineer</span>
  </div>
</body>
</html>
```

**Expected output**: The visible page shows “Jane Doe” and “Software Engineer”. An RDFa processor extracts a Person item with `name` and `jobTitle` properties.

**Why this output occurs**: `vocab` establishes Schema.org as the default vocabulary; `typeof="Person"` declares the item type; `property` attributes create the name-value pairs within that vocabulary.

---

### Real-World Cases

1. **Product pages**: Marking up product names, prices, and reviews with Schema.org vocabulary.
2. **Event listings**: Marking up event dates, locations, and organisers.
3. **Article metadata**: Marking up headlines, authors, and publication dates.

---

### References

- W3C — RDFa Core 1.1 — https://www.w3.org/TR/rdfa-core/
- W3C — RDFa Lite 1.1 — https://www.w3.org/TR/rdfa-lite/
- W3C — HTML+RDFa 1.1 — https://www.w3.org/TR/rdfa-in-html/
- W3C — RDFa 1.1 Primer — https://www.w3.org/TR/rdfa-primer/

---

## 2. `vocab` — Default Vocabulary Declaration

### Definitions

**Core Definition**
The `vocab` attribute establishes a default vocabulary (an IRI) for the current element and its descendants, allowing plain terms to be expanded against that vocabulary without requiring a prefix .

**Technical Definition**
`vocab` takes an absolute IRI (e.g. `https://schema.org/`). When specified, terms used in `typeof` and `property` attributes are resolved against this vocabulary. A default vocabulary can be defined by the host language as part of the initial context and can be overridden on any element and its descendants . The `vocab` attribute was introduced in RDFa 1.1 to simplify authoring .

**Beginner-Friendly Explanation**
`vocab` tells the processor “use this dictionary by default.” Once you set `vocab="https://schema.org/"`, you can write `typeof="Person"` and `property="name"` without spelling out the full URL every time.

---

### Purposes

- **To simplify authoring** by eliminating the need to write full vocabulary URLs for every term.
- **To establish a default vocabulary** for a block of markup.
- **To override the default vocabulary** inherited from an ancestor or host language.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div vocab="https://schema.org/" typeof="Person">
  <span property="name">Jane Doe</span>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `vocab="https://schema.org/"` | Absolute IRI of the default vocabulary. |
| Scope | Applies to the current element and descendants, unless overridden. |

#### Syntax Rules

1. **`vocab` supports an IRI** .
2. **The vocabulary applies to the element and its children** unless overridden.
3. **Plain terms in `typeof` and `property` resolve against the vocabulary**.

#### Constraints and Limitations

- **The vocabulary must define the terms used** in `typeof` and `property`.
- **Overriding `vocab` on a descendant** changes the default for that subtree.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Schema.org Vocabulary with Plain Terms

**Setup Guide**: Save as `vocab.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Vocab Example</title>
</head>
<body>
  <!-- vocab sets schema.org as the default vocabulary -->
  <div vocab="https://schema.org/" typeof="Product">
    <span property="name">Wireless Headphones</span>
    <span property="price">$99.99</span>
  </div>
</body>
</html>
```

**Expected output**: A Product item with `name` and `price` properties, without any prefix declarations.

**Why this output occurs**: `vocab` tells the processor to expand plain terms like `Product`, `name`, and `price` against `https://schema.org/` .

---

### Real-World Cases

1. **Schema.org markup**: Using `vocab="https://schema.org/"` to write plain term names.
2. **Custom vocabularies**: Setting `vocab` to a proprietary vocabulary for internal data.
3. **Mixed vocabularies**: Overriding `vocab` on a nested element to use a different vocabulary for a sub-item.

---

### References

- W3C — RDFa Core 1.1: The `vocab` attribute — https://www.w3.org/TR/rdfa-core/#a-vocab
- W3C — RDFa Lite 1.1: `vocab` — https://www.w3.org/TR/rdfa-lite/#vocab

---

## 3. `typeof` — Item Type Declaration

### Definitions

**Core Definition**
The `typeof` attribute declares the type (or types) of the item being described. It indicates that the element represents an instance of a class such as Person, Product, or Event .

**Technical Definition**
`typeof` takes a space-separated list of terms, CURIEs, or absolute IRIs that identify the item’s type. When used with `vocab`, plain terms are resolved against the default vocabulary. The `typeof` attribute can be used to declare new data items and to type resources being described .

**Beginner-Friendly Explanation**
`typeof` says “this thing is a Person” or “this thing is a Product.” It is how you declare what kind of entity you are describing.

---

### Purposes

- **To declare the item’s type** so processors know which vocabulary terms apply.
- **To create a new data item** that contains the properties declared within it.
- **To support nested items** by typing sub-items within a parent item.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div vocab="https://schema.org/" typeof="Person">
  <span property="name">Jane Doe</span>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `typeof="Person"` | The type of the item (resolved against `vocab` or a prefix). |
| Multiple types | Space-separated list is permitted. |

#### Syntax Rules

1. **`typeof` takes a term, CURIE, or absolute IRI**.
2. **When `vocab` is present, plain terms resolve against it**.
3. **`typeof` can be used on any element** that creates an item.

#### Constraints and Limitations

- **The type must be defined in the vocabulary** for meaningful processing.
- **Multiple types can be declared**, though this is less common.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Product Type with Nested Organisation

**Setup Guide**: Save as `typeof.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Typeof Example</title>
</head>
<body>
  <!-- Product item -->
  <div vocab="https://schema.org/" typeof="Product">
    <span property="name">Wireless Headphones</span>

    <!-- Nested Organisation item for the brand -->
    <div property="brand" typeof="Organization">
      <span property="name">Acme Audio</span>
    </div>
  </div>
</body>
</html>
```

**Expected output**: A Product item with a `name` and a `brand` property whose value is a nested Organization item with its own `name`.

**Why this output occurs**: `typeof="Product"` declares the outer item; `typeof="Organization"` declares the nested item; `property="brand"` links them .

---

### Real-World Cases

1. **Product markup**: `typeof="Product"` for e-commerce items.
2. **Person markup**: `typeof="Person"` for author or profile pages.
3. **Event markup**: `typeof="Event"` for event listings.

---

### References

- W3C — RDFa Core 1.1: The `typeof` attribute — https://www.w3.org/TR/rdfa-core/#a-typeof
- W3C — RDFa Lite 1.1: `typeof` — https://www.w3.org/TR/rdfa-lite/#typeof

---

## 4. `property` — Property Naming

### Definitions

**Core Definition**
The `property` attribute adds a property (a predicate) to the current item, expressing a relationship between the subject and either a literal value or a resource .

**Technical Definition**
`property` takes a space-separated list of CURIEs, terms, or absolute IRIs. When the element also has a `resource`, `href`, or `src` attribute, the property’s object is a resource (URI). Otherwise, the object is a literal whose value is the element’s text content or its `content` attribute . The `property` attribute does not induce chaining in the same way as `rel`; it is used primarily for literal-valued properties and direct resource relationships .

**Beginner-Friendly Explanation**
`property` says “this piece of content is the name” or “this link is the URL.” It connects a property name to the value shown in or referenced by the element.

---

### Purposes

- **To add a property to an item** with a literal or resource value.
- **To express name-value pairs** for the structured data.
- **To support both text values** (via element content) and **resource values** (via `resource`, `href`, or `src`).

---

### Syntax Rules and Structure

#### Complete General Syntax (Literal Value)

```html
<span property="name">Jane Doe</span>
```

#### Complete General Syntax (Resource Value)

```html
<a property="url" href="https://example.com/jane">Profile</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| `property="name"` | The property name. |
| Value source | Text content (literal) or `resource`/`href`/`src` (URI). |

#### Syntax Rules

1. **`property` takes a term, CURIE, or absolute IRI**.
2. **Literal values come from the element’s content** or `content` attribute.
3. **Resource values come from `resource`, `href`, or `src`** .
4. **`property` does not chain** in the way `rel` does .

#### Constraints and Limitations

- **The property name must be defined by the vocabulary**.
- **Multiple properties can be specified**, though this is less common.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Literal and Resource Properties

**Setup Guide**: Save as `property.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Property Example</title>
</head>
<body>
  <div vocab="https://schema.org/" typeof="Person">
    <!-- Literal value from text content -->
    <span property="name">Jane Doe</span>

    <!-- Resource value from href -->
    <a property="url" href="https://example.com/jane">Profile</a>

    <!-- Resource value from src -->
    <img property="image" src="jane.jpg" alt="Jane Doe">
  </div>
</body>
</html>
```

**Expected output**: A Person item with `name` (literal “Jane Doe”), `url` (URI from `href`), and `image` (URI from `src`).

**Why this output occurs**: `property` on `<span>` uses text content as a literal; `property` on `<a>` and `<img>` uses their URL-bearing attributes as resource values .

---

### Real-World Cases

1. **Product name and price**: `property="name"` and `property="price"`.
2. **Author URLs**: `property="url"` with `href`.
3. **Images**: `property="image"` with `src`.

---

### References

- W3C — RDFa Core 1.1: The `property` attribute — https://www.w3.org/TR/rdfa-core/#a-property
- W3C — RDFa Lite 1.1: `property` — https://www.w3.org/TR/rdfa-lite/#property

---

## 5. `resource` — Object Resource Specification

### Definitions

**Core Definition**
The `resource` attribute specifies the object (target) of a relationship when that object is a resource (URI) rather than a literal text value .

**Technical Definition**
`resource` takes a URI or SafeCURIE/CURIE. It can serve two roles: when used on an element with `property`, it sets the object (target) of the property relationship; when used on an element without a property, it can set the context (subject) for subsequent statements . The primer explains that `resource` combined with `property` sets the target, much like `href` .

**Beginner-Friendly Explanation**
`resource` tells the processor “the thing this property points to is at this URL.” It is used when the value of a property is a link or identifier rather than plain text.

---

### Purposes

- **To set the object resource** of a property relationship.
- **To specify a URI value** for a property without making the element a hyperlink.
- **To set the context** (subject) for subsequent statements when used alone.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<h3 property="creator" resource="#me">Alice</h3>
```

**Component breakdown**

| Component | Description |
|---|---|
| `resource="#me"` | The URI or CURIE of the object resource. |
| Combined with `property` | Sets the target of the property. |

#### Syntax Rules

1. **`resource` takes a URI or CURIE** .
2. **When combined with `property`, it sets the object resource** .
3. **When used alone, it can set the context** for subsequent statements .

#### Constraints and Limitations

- **`resource` does not create a hyperlink**; use `href` for clickable links.
- **The distinction between `resource` and `about`** is that `about` only sets the context, while `resource` can set both context and object depending on context .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Property with Resource Object

**Setup Guide**: Save as `resource.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Resource Example</title>
</head>
<body>
  <div vocab="https://schema.org/" typeof="BlogPosting">
    <h2 property="headline">The Trouble with Bob</h2>

    <!-- resource sets the object of the creator property to #me -->
    <h3 property="creator" resource="#me">Alice</h3>
  </div>
</body>
</html>
```

**Expected output**: A BlogPosting item with a `headline` and a `creator` whose object is the URI `#me` (resolving to `https://example.com/page.html#me`).

**Why this output occurs**: `resource="#me"` sets the object of the `creator` property to a resource URI rather than a literal string .

---

### Real-World Cases

1. **Author references**: `resource="#author"` pointing to an author entity.
2. **Product brand references**: `resource="#brand"` pointing to a brand entity.
3. **Context setting**: Using `resource` alone to set the subject for a block of statements.

---

### References

- W3C — RDFa Core 1.1: The `resource` attribute — https://www.w3.org/TR/rdfa-core/#a-resource
- W3C — RDFa 1.1 Primer: Setting the context — https://www.w3.org/TR/rdfa-primer/#setting-the-context

---

## 6. `prefix` — CURIE Prefix Mapping

### Definitions

**Core Definition**
The `prefix` attribute declares prefix-to-IRI mappings that allow Compact URI Expressions (CURIEs) to be used as shorthand for full IRIs in RDFa attributes .

**Technical Definition**
`prefix` takes a whitespace-separated list of prefix–IRI pairs in the form `prefix: IRI`. The declared prefixes apply to the current element and its descendants, with the innermost mapping taking precedence . RDFa 1.1 deprecates the use of `xmlns:` for prefix declarations in favour of the `prefix` attribute . CURIEs consist of a prefix and a reference separated by a colon, and they are expanded to full IRIs during processing .

**Beginner-Friendly Explanation**
`prefix` lets you create short aliases for long vocabulary URLs. For example, `prefix="foaf: http://xmlns.com/foaf/0.1/"` lets you write `foaf:name` instead of the full URL. It is useful when you are mixing terms from multiple vocabularies.

---

### Purposes

- **To enable CURIE shorthand** for vocabulary terms.
- **To mix multiple vocabularies** on the same page.
- **To replace the deprecated `xmlns:` mechanism** from RDFa 1.0 .

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<html prefix="foaf: http://xmlns.com/foaf/0.1/ dc: http://purl.org/dc/terms/">
```

**Component breakdown**

| Component | Description |
|---|---|
| `prefix="foaf: http://xmlns.com/foaf/0.1/"` | Prefix–IRI pair. |
| Multiple pairs | Whitespace-separated. |

#### Syntax Rules

1. **The value is a whitespace-separated list of `prefix: IRI` pairs**.
2. **Prefixes apply to the current element and descendants** .
3. **The innermost mapping takes precedence** .
4. **RDFa 1.1 deprecates `xmlns:`** in favour of `prefix` .

#### Constraints and Limitations

- **Prefixes must be declared before use** in the document order.
- **Conflicting prefix declarations** follow innermost-precedence rules.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Prefixes for Mixed Vocabularies

**Setup Guide**: Save as `prefix.html`.

```html
<!DOCTYPE html>
<html lang="en" prefix="foaf: http://xmlns.com/foaf/0.1/ dc: http://purl.org/dc/terms/">
<head>
  <meta charset="UTF-8">
  <title>Prefix Example</title>
</head>
<body>
  <div typeof="foaf:Person">
    <span property="foaf:name">Jane Doe</span>
    <span property="dc:title">Senior Engineer</span>
  </div>
</body>
</html>
```

**Expected output**: A Person item (from FOAF vocabulary) with `foaf:name` and `dc:title` properties from two different vocabularies.

**Why this output occurs**: The `prefix` attribute maps `foaf:` and `dc:` to their respective IRIs, allowing CURIEs to be expanded to full vocabulary terms .

---

### Real-World Cases

1. **Mixed-vocabulary markup**: Combining FOAF, Dublin Core, and Schema.org terms.
2. **Legacy migration**: Replacing `xmlns:` declarations with `prefix` in RDFa 1.1 documents .
3. **Enterprise vocabularies**: Using custom prefixes for proprietary vocabularies.

---

### References

- W3C — RDFa Core 1.1: The `prefix` attribute — https://www.w3.org/TR/rdfa-core/#a-prefix
- W3C — RDFa Lite 1.1: `prefix` — https://www.w3.org/TR/rdfa-lite/#prefix
- W3C — CURIE Syntax — https://www.w3.org/TR/curie/

---

## 7. RDFa Best Practices

### Definitions

**Core Definition**
RDFa best practices are guidelines for authoring RDFa markup that is accurate, maintainable, interoperable, and aligned with search-engine guidelines.

**Technical Definition**
Best practices are informed by the W3C RDFa specifications, Schema.org documentation, Google’s structured-data policies, and community authoring guides. They encompass using RDFa Lite for simple cases, ensuring markup matches visible content, validating output, and choosing the right structured-data format for the context .

**Beginner-Friendly Explanation**
Best practices help you get the most from RDFa without making mistakes that could hurt your search rankings or break your structured data.

---

### Purposes

- **To ensure accuracy** between visible content and structured data.
- **To maximise interoperability** with search engines and Linked Data tools.
- **To simplify maintenance** through consistent patterns.
- **To avoid common authoring errors** such as unescaped characters and empty element shortcuts .

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use RDFa Lite for simple cases** | The five-attribute subset covers most authoring needs . |
| **Match structured data to visible content** | Every property must correspond to visible, human-readable content . |
| **Use Schema.org vocabulary** | It is the most widely supported vocabulary for search engines . |
| **Validate output** | Use an RDFa distiller or structured-data testing tool. |
| **Escape special characters** | Use `&amp;` for `&` in attribute values and content . |
| **Avoid XML self-closing shortcuts** | Use explicit closing tags for `div` and `span`; some parsers mishandle `<div/>` . |
| **Use `prefix` instead of `xmlns:`** | RDFa 1.1 deprecates `xmlns:` for prefix declarations . |
| **Consider JSON-LD** | Google recommends JSON-LD as the preferred format for search-engine structured data . |

#### Constraints and Limitations

- **RDFa is more tightly coupled to markup** than JSON-LD, which can make maintenance harder when content changes .
- **Search-engine support for RDFa may evolve** as JSON-LD becomes the dominant format.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correctly Escaped RDFa

**Setup Guide**: Save as `best-practice-rdfa.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>RDFa Best Practices</title>
</head>
<body>
  <!-- GOOD: escaped ampersand in a resource URI -->
  <div vocab="https://schema.org/" typeof="Product">
    <span property="name">Wireless Headphones</span>
    <a property="url" href="https://example.com/product?id=1&amp;lang=en">Product page</a>
  </div>

  <!-- BAD: unescaped ampersand -->
  <!--
  <a property="url" href="https://example.com/product?id=1&lang=en">Product page</a>
  -->
</body>
</html>
```

**Expected output**: The good markup produces valid RDFa with a properly escaped URI. The bad markup may produce parsing errors or incorrect URI resolution .

**Why this output occurs**: The `&` character must be escaped as `&amp;` in HTML attribute values to avoid being interpreted as the start of a character reference .

---

### Real-World Cases

1. **E-commerce product pages**: Accurate price and availability markup that matches visible content.
2. **Government and public-sector sites**: RDFa used for Linked Data publishing with validation requirements .
3. **Schema.org deployment**: Using RDFa Lite with Schema.org vocabulary for rich search results .

---

### References

- W3C — RDFa 1.1 Primer — https://www.w3.org/TR/rdfa-primer/
- W3C — RDFa Lite 1.1 — https://www.w3.org/TR/rdfa-lite/
- Schema.org — Getting started with schema.org — https://schema.org/docs/gs.html
- Google Search Central — Structured data general guidelines — https://developers.google.com/search/docs/appearance/structured-data/sd-policies
- Tips for Authoring RDFa — http://ebusiness-unibw.org/wiki/RDFaAuthoring

---

## Summary Table of RDFa Lite Attributes

| Attribute | Role | Example |
|---|---|---|
| `vocab` | Sets the default vocabulary for a block. | `vocab="https://schema.org/"` |
| `typeof` | Declares the type of the item. | `typeof="Person"` |
| `property` | Adds a property with a literal or resource value. | `property="name"` |
| `resource` | Specifies the object resource of a relationship. | `resource="#me"` |
| `prefix` | Maps a prefix to an IRI for CURIE shorthand. | `prefix="foaf: http://xmlns.com/foaf/0.1/"` |

---

## Final Notes

- **RDFa is a W3C standard** with two profiles: RDFa Core 1.1 (full attribute set) and RDFa Lite 1.1 (five-attribute subset) .
- **RDFa produces RDF triples**, making it closely aligned with the Linked Data community .
- **Google supports RDFa** as one of three structured-data formats, alongside JSON-LD and microdata, though JSON-LD is now recommended .
- **RDFa Lite covers most simple authoring needs** and is fully upward-compatible with full RDFa 1.1 .
- **Always validate RDFa output** with a distiller or structured-data testing tool before deployment.

---

This cheat sheet has covered RDFa and its five Lite attributes — `vocab`, `typeof`, `property`, `resource`, and `prefix` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate RDFa markup against the W3C RDFa specifications and search-engine structured-data guidelines.