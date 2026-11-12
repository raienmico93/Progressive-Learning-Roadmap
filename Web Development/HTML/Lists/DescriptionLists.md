# Description Lists — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Description lists are HTML structures that represent an association list of **name–value groups** — a set of terms (or names) paired with their corresponding descriptions (or values). They are created with the `<dl>` element, which contains one or more `<dt>` (description term) elements followed by one or more `<dd>` (description details) elements.

**Technical Definition**
In the WHATWG HTML Living Standard, `<dl>` belongs to the **flow content** category and, when its children include at least one name–value group, to the **palpable content** category. Its content model is zero or more groups, each consisting of **one or more `<dt>` elements followed by one or more `<dd>` elements**, optionally intermixed with script-supporting elements (`<script>`, `<template>`). In WHATWG HTML and W3C HTML 5.2 and later, the content model alternatively permits **one or more `<div>` elements** that each wrap a name–value group, a mechanism introduced to support styling and grouping without breaking semantics. The `<dt>` element specifies the term or name, and `<dd>` specifies the description or value. In the accessibility tree, `<dl>` maps to the ARIA role `list` (or no corresponding role, depending on the AT), `<dt>` maps to `term`, and `<dd>` maps to `definition`; screen readers announce the list and its term–description structure differently across platforms.

**Beginner-Friendly Explanation**
A description list is a set of term-and-description pairs. Think of a glossary: “HTML” is the term, and “the standard markup language for web pages” is the description. You wrap the whole glossary in `<dl>`, each term in `<dt>`, and each description in `<dd>`. Description lists are also perfect for metadata — key–value pairs like “Author: Jane Doe” or “Price: $29.99” — because they explicitly encode the relationship between a name and its value.

---

### Key Characteristics

- **Name–value association**: Every group pairs one or more terms (`<dt>`) with one or more descriptions (`<dd>`), preserving the semantic relationship.
- **Order flexibility**: A group may have multiple terms sharing one description, or one term with multiple descriptions, as long as the sequence (`<dt>`…`<dd>`) is maintained.
- **Not a glossary-only element**: The specification explicitly allows terms and definitions, metadata topics and values, questions and answers, or any other groups of name–value data.
- **`<div>` grouping**: In WHATWG HTML and HTML 5.2+, each name–value group may be wrapped in a `<div>` for styling, without changing the list semantics.
- **Accessibility support**: WCAG Technique H40 recognises description lists as a sufficient technique for providing descriptions of terms (related to SC 3.1.3 Unusual Words, when combined with G55).
- **No presentational attributes**: The `compact` attribute is obsolete; visual presentation is controlled entirely by CSS.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements and content models.
2. **Flow content**: understanding what may appear inside `<dt>` and `<dd>`.
3. **The difference between lists**: knowing when to use `<ul>`, `<ol>`, and `<dl>`.
4. **CSS fundamentals**: styling `dt` and `dd` with margins, floats, or grid.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | Technique H40 uses description lists for term–description relationships; `<dl>` maps to `list` and `<dt>`/`<dd>` to term/definition roles. |
| **Metadata / structured data** | Description lists are the standard HTML pattern for key–value metadata (author, date, price, specifications). |
| **CSS layout** | `dt`/`dd` are commonly styled with grid or flexbox for two-column definition layouts. |
| **Microdata / RDFa** | `itemprop` can be applied to `<dt>` and `<dd>` to build structured data from description lists. |
| **Glossaries and FAQs** | `<dl>` is the canonical markup for glossaries; FAQ accordions sometimes use `<dl>` with `<dt>` questions and `<dd>` answers. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<dl>`** — the description list container
2. **`<dt>`** — the description term
3. **`<dd>`** — the description details
4. **Terms and definitions** — the glossary use case
5. **Metadata-style lists** — the key–value use case
6. **Description list best practices** — consolidated guidelines

---

## 1. `<dl>` — The Description List Container

### Definitions

**Core Definition**
The `<dl>` element represents a description list — an association list consisting of zero or more name–value groups.

**Technical Definition**
`<dl>` belongs to the **flow content** and **palpable content** categories. Its content model is **zero or more groups**, each consisting of one or more `<dt>` elements followed by one or more `<dd>` elements, optionally intermixed with script-supporting elements. In WHATWG HTML and W3C HTML 5.2 and later, it may alternatively contain **one or more `<div>` elements**, each wrapping a name–value group. The `compact` attribute is obsolete. The DOM interface is `HTMLDListElement`.

**Beginner-Friendly Explanation**
`<dl>` is the container for a description list. You put terms in `<dt>` and their descriptions in `<dd>`, and the browser groups each term with its description.

---

### Purposes

- **To represent name–value pairs** — terms and definitions, metadata keys and values, questions and answers.
- **To provide a semantic grouping** that assistive technologies can interpret as a term–description relationship.
- **To serve as the structural base for glossaries and metadata blocks**.
- **To enable CSS-driven definition layouts** such as two-column term–description displays.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<dl>
  <dt>Term</dt>
  <dd>Description</dd>
  <dt>Another term</dt>
  <dd>Another description</dd>
</dl>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<dl>` | The description list container. |
| `<dt>` | One or more description terms. |
| `<dd>` | One or more description details, following the `<dt>` elements. |
| `<div>` (optional) | May wrap each name–value group in WHATWG HTML and HTML 5.2+. |

#### Syntax Rules

1. **Each group must consist of one or more `<dt>` elements followed by one or more `<dd>` elements**.
2. **A group may have multiple terms sharing one description**, or one term with multiple descriptions, provided the sequence is maintained.
3. **In WHATWG HTML and HTML 5.2+, a `<div>` may wrap each group** for styling.
4. **The `compact` attribute is obsolete**; use CSS instead.
5. **The `<dl>` element accepts only global attributes**.

#### Constraints and Limitations

- **A `<dd>` without a preceding `<dt>` is non-conformant**; the term must come first.
- **A `<dt>` without a following `<dd>` is non-conformant**; the description must follow.
- **Not every list of items belongs in a `<dl>`**: if items can stand alone without explanation, a `<ul>` is simpler and more appropriate.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Glossary Entry

**Setup Guide**: Save as `dl-basic.html` and open it in a browser. Inspect the accessibility tree to confirm the term–definition structure.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Description List</title>
</head>
<body>
  <!-- The dl groups each term with its description -->
  <dl>
    <dt>Firefox</dt>
    <dd>
      A free, open source, cross-platform, graphical web browser
      developed by the Mozilla Corporation and hundreds of volunteers.
    </dd>
  </dl>
</body>
</html>
```

**Expected output**: The term “Firefox” is displayed with its description indented or otherwise visually associated. A screen reader announces the term and its definition as a group.

**Why this output occurs**: The `<dl>` element creates the description list, and the `<dt>`/`<dd>` pair encodes the term–description relationship. Browsers apply default indentation to `<dd>`.

---

#### Example 2: A Description List with `<div>` Grouping

**Setup Guide**: Save as `dl-div.html`. This pattern is valid in WHATWG HTML and HTML 5.2+.

```html
<dl>
  <!-- Each group is wrapped in a div for styling -->
  <div>
    <dt>Name</dt>
    <dd>Godzilla</dd>
  </div>
  <div>
    <dt>Born</dt>
    <dd>1952</dd>
  </div>
  <div>
    <dt>Birthplace</dt>
    <dd>Japan</dd>
  </div>
</dl>
```

**Expected output**: The same term–description structure, but each group can now be targeted independently with CSS or scripted as a unit.

**Why this output occurs**: The `<div>` wrappers group each `<dt>`/`<dd>` pair without breaking the list semantics; the specification explicitly allows this pattern for styling and organisation.

---

### Real-World Cases

1. **Glossaries**: Terms and definitions in technical documentation.
2. **Metadata blocks**: Author, date, price, and specification tables.
3. **FAQ accordions**: Questions as `<dt>` and answers as `<dd>`.

---

### References

- W3C — HTML 5: The Markup Language: `dl` — https://w3c.github.io/html-reference/aria/dl.html
- MDN Web Docs — `<dl>`: The Description List element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dl
- WHATWG — The `dl` element — https://html.spec.whatwg.org/multipage/grouping-content.html#the-dl-element

---

## 2. `<dt>` — The Description Term

### Definitions

**Core Definition**
The `<dt>` element specifies a term or name in a description list. It must be contained within a `<dl>` element (or within a `<div>` that is a child of `<dl>`).

**Technical Definition**
`<dt>` has no content category of its own; its role is defined by its parent `<dl>`. Its content model is **flow content**, but with restrictions: it must not contain `<header>`, `<footer>`, sectioning content, or heading content descendants. Multiple consecutive `<dt>` elements may precede one or more `<dd>` elements, allowing multiple terms to share descriptions. In the accessibility tree, `<dt>` maps to the ARIA role `term`.

**Beginner-Friendly Explanation**
`<dt>` is the term in a description list. It comes before its description. If several terms mean the same thing, you can list them one after another before the shared description.

---

### Purposes

- **To specify the term or name** in a name–value group.
- **To allow multiple terms to share one or more descriptions**.
- **To provide the accessible term label** for the associated `<dd>` description.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<dl>
  <dt>Term</dt>
  <dd>Description</dd>
</dl>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<dt>` | The description term. |
| Content | Flow content, without headings, sectioning content, or header/footer descendants. |

#### Syntax Rules

1. **`<dt>` must be a child of `<dl>`** (or a `<div>` child of `<dl>`).
2. **Content model is flow content**, but restricted: no `<header>`, `<footer>`, sectioning content, or heading content.
3. **Multiple `<dt>` elements may precede `<dd>` elements** to allow multiple terms to share descriptions.
4. **Within a single `<dl>`, there should not be more than one `<dt>` for each name**.

#### Constraints and Limitations

- **`<dt>` cannot contain headings or sectioning elements**.
- **A `<dt>` without a following `<dd>` is non-conformant**.
- **Using `<dt>` outside `<dl>` is non-conforming**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Terms Sharing One Description

**Setup Guide**: Save as `dt-multiple.html`.

```html
<dl>
  <!-- Three terms share one description -->
  <dt>Firefox</dt>
  <dt>Mozilla Firefox</dt>
  <dt>Fx</dt>
  <dd>
    A free, open source, cross-platform, graphical web browser
    developed by the Mozilla Corporation and hundreds of volunteers.
  </dd>
</dl>
```

**Expected output**: The three terms are listed, and the single description applies to all of them.

**Why this output occurs**: The specification explicitly permits multiple `<dt>` elements to precede one or more `<dd>` elements, allowing synonyms or alternative names to share a definition.

---

#### Example 2: A Term with a Definition and a Footnote

**Setup Guide**: Save as `dt-definition.html`.

```html
<dl>
  <dt lang="en-US"><dfn>color</dfn></dt>
  <dt lang="en-GB"><dfn>colour</dfn></dt>
  <dd>
    A sensation derived from the ability of the eye to distinguish
    different wavelengths of light.
  </dd>
</dl>
```

**Expected output**: Two terms (“color” and “colour”) share a single definition, with language attributes marking the regional variants.

**Why this output occurs**: The `<dt>` elements provide the terms, and the `<dd>` provides the shared description. The `lang` attributes give pronunciation context.

---

### Real-World Cases

1. **Glossaries**: Each term is a `<dt>`.
2. **Bilingual terminology**: Multiple language variants of a term share a definition.
3. **Metadata labels**: Labels such as “Author” or “Version” are `<dt>` elements.

---

### References

- MDN Web Docs — `<dt>`: The Description Term element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dt
- W3C — HTML 5: The Markup Language: `dt` — https://w3c.github.io/html-reference/aria/dt.html

---

## 3. `<dd>` — The Description Details

### Definitions

**Core Definition**
The `<dd>` element provides the description, definition, or value for the preceding term or terms in a description list.

**Technical Definition**
`<dd>` has no content category of its own; its role is defined by its parent `<dl>`. Its content model is **flow content**. One or more `<dd>` elements may follow one or more `<dt>` elements, allowing a term or group of terms to have multiple descriptions. In the accessibility tree, `<dd>` maps to the ARIA role `definition`.

**Beginner-Friendly Explanation**
`<dd>` is the description or value that follows a term in a description list. It tells you what the term means or what value the label holds.

---

### Purposes

- **To provide the description or value** for a term or group of terms.
- **To allow multiple descriptions** for a term or group of terms.
- **To complete the name–value pairing** of the description list group.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<dl>
  <dt>Term</dt>
  <dd>Description</dd>
</dl>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<dd>` | The description details. |
| Content | Flow content. |

#### Syntax Rules

1. **`<dd>` must be a child of `<dl>`** (or a `<div>` child of `<dl>`) and must follow one or more `<dt>` elements.
2. **Content model is flow content**.
3. **Multiple `<dd>` elements may follow a `<dt>`** to provide multiple descriptions.

#### Constraints and Limitations

- **`<dd>` must not appear before any `<dt>` in a group**.
- **Using `<dd>` outside `<dl>` is non-conforming**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Descriptions for One Term

**Setup Guide**: Save as `dd-multiple.html`.

```html
<dl>
  <dt>Firefox</dt>
  <!-- Two descriptions for the same term -->
  <dd>
    A free, open source, cross-platform, graphical web browser
    developed by the Mozilla Corporation and hundreds of volunteers.
  </dd>
  <dd>
    The Red Panda, also known as the Lesser Panda, Wah, Bear Cat or Firefox,
    is a mostly herbivorous mammal, slightly larger than a domestic cat.
  </dd>
</dl>
```

**Expected output**: The term “Firefox” has two descriptions, one for the browser and one for the red panda.

**Why this output occurs**: The specification permits one term to have multiple descriptions, as long as the `<dd>` elements follow the `<dt>` in sequence.

---

#### Example 2: Metadata Values with Multiple Labels

**Setup Guide**: Save as `dd-metadata.html`. This example is based on the W3C metadata example.

```html
<dl>
  <dt>Authors</dt>
  <dt>Editors</dt>
  <dd>Robert Rothman</dd>
  <dd>Daniel Jackson</dd>
</dl>
```

**Expected output**: Two labels (“Authors” and “Editors”) share two values (“Robert Rothman” and “Daniel Jackson”).

**Why this output occurs**: The specification explicitly provides this example to show that multiple `<dt>` elements may share multiple `<dd>` elements, representing alternative name–value assignments.

---

### Real-World Cases

1. **Glossary definitions**: Each `<dd>` provides the meaning of its term.
2. **Multiple values for a label**: “Authors” may have multiple `<dd>` values.
3. **Metadata values**: The value side of a metadata key–value pair.

---

### References

- MDN Web Docs — `<dd>`: The Description Details element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dd
- W3C — HTML 5: The Markup Language: `dd` — https://w3c.github.io/html-reference/aria/dd.html

---

## 4. Terms and Definitions — The Glossary Use Case

### Definitions

**Core Definition**
The terms-and-definitions use case is the most common application of description lists: a set of terms paired with their definitions, as in a glossary or dictionary.

**Technical Definition**
When `<dl>` is used for terms and definitions, each `<dt>` contains the term (often marked with `<dfn>` to indicate the defining instance), and each `<dd>` contains the definition. WCAG Technique H40 states that the objective is to “provide the description of names or terms by presenting them in a description list”, and that “usage of description lists ensures that terms and their descriptions are semantically related even as presentation format changes, as well as ensuring that these terms and descriptions are semantically grouped as a unit”.

**Beginner-Friendly Explanation**
A glossary is a list of terms and what they mean. Description lists are the correct HTML element for a glossary because they explicitly pair each term with its definition.

---

### Purposes

- **To provide a semantic glossary** that assistive technologies can interpret as term–description pairs.
- **To support WCAG 3.1.3 (Unusual Words)** when combined with a mechanism for linking to definitions.
- **To preserve the term–definition relationship** when the visual presentation changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<dl>
  <dt><dfn>Term</dfn></dt>
  <dd>Definition of the term.</dd>
</dl>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<dt>` | The term; may contain `<dfn>` to mark the defining instance. |
| `<dd>` | The definition. |

#### Syntax Rules

1. **Use `<dfn>` inside `<dt>`** to mark the term being defined.
2. **Each `<dt>` should be followed by one or more `<dd>` definitions**.
3. **Description lists are easiest to use when the descriptions are ordered alphabetically** (though this is a recommendation, not a requirement).

#### Constraints and Limitations

- **A glossary is not the only use of `<dl>`**; metadata and other name–value pairs are also valid.
- **Do not use `<dl>` for lists where items can stand alone**; use `<ul>` in that case.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Nautical Glossary (WCAG H40 Example)

**Setup Guide**: Save as `glossary.html`. This is the W3C Technique H40 example.

```html
<dl title="Nautical Terms">
  <dt>Knot</dt>
  <dd>
    A <i>knot</i> is a unit of speed equaling 1 nautical mile per hour
    (1.15 miles per hour or 1.852 kilometers per hour).
  </dd>
  <dt>Port</dt>
  <dd>
    <i>Port</i> is the nautical term (used on boats and ships) that refers
    to the left side of a ship, as perceived by a person facing towards the
    bow (the front of the vessel).
  </dd>
  <dt>Starboard</dt>
  <dd>
    <i>Starboard</i> is the nautical term (used on boats and ships) that
    refers to the right side of a vessel, as perceived by a person facing
    towards the bow (the front of the vessel).
  </dd>
</dl>
```

**Expected output**: A glossary of nautical terms, with each term and its definition grouped semantically.

**Why this output occurs**: The `<dl>` element groups each `<dt>` term with its `<dd>` definition, satisfying Technique H40 and preserving the term–definition relationship.

---

#### Example 2: A Glossary with `<dfn>` and Abbreviations

**Setup Guide**: Save as `glossary-dfn.html`.

```html
<dl>
  <dt><dfn id="def-html">HTML</dfn></dt>
  <dd>
    HyperText Markup Language — the standard markup language for
    creating web pages.
  </dd>
  <dt><dfn id="def-css">CSS</dfn></dt>
  <dd>
    Cascading Style Sheets — a style sheet language used for describing
    the presentation of a document written in HTML.
  </dd>
</dl>
```

**Expected output**: A glossary with each term marked as the defining instance via `<dfn>`, allowing other parts of the document to link to `#def-html` and `#def-css`.

**Why this output occurs**: The `<dfn>` element identifies the defining instance of the term, and the `<dd>` provides the definition. The `id` attributes enable cross-references.

---

### Real-World Cases

1. **Technical documentation**: Glossaries of API terms and concepts.
2. **Educational content**: Vocabulary lists in language-learning materials.
3. **Legal and medical glossaries**: Definitions of specialised terms.

---

### References

- W3C — Technique H40: Using description lists — https://www.w3.org/WAI/WCAG22/Techniques/html/H40
- W3C — HTML 5: The `dl` element (glossary example) — https://www.w3.org/TR/2011/WD-html5-author-20110809/the-dl-element.html

---

## 5. Metadata-Style Lists — The Key–Value Use Case

### Definitions

**Core Definition**
A metadata-style description list uses `<dl>` to display metadata as a list of key–value pairs — for example, “Name: Godzilla”, “Born: 1952”, “Birthplace: Japan”.

**Technical Definition**
The WHATWG specification explicitly states that name–value groups may be “metadata topics and values”, and provides an example with “Last modified time”, “Recommended update interval”, “Authors”, and “Editors” as labels paired with their values. MDN similarly notes that “description lists are useful for displaying metadata as a list of key–value pairs”. This use case does not require definitions in the dictionary sense; it requires a label–value association.

**Beginner-Friendly Explanation**
Metadata lists are everywhere: product specifications, author information, event details, and API parameters. A description list is the correct HTML element for these key–value pairs because it explicitly associates each label with its value.

---

### Purposes

- **To display metadata as key–value pairs** — product specs, author details, event information.
- **To provide a semantic association** between a label and its value that assistive technologies can interpret.
- **To support CSS two-column layouts** for label–value tables.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<dl>
  <dt>Name</dt>
  <dd>Godzilla</dd>
  <dt>Born</dt>
  <dd>1952</dd>
  <dt>Birthplace</dt>
  <dd>Japan</dd>
</dl>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<dt>` | The metadata label or key. |
| `<dd>` | The metadata value. |

#### Syntax Rules

1. **Each `<dt>`–`<dd>` pair is a key–value group**.
2. **Multiple values for one key** are permitted via multiple `<dd>` elements.
3. **Multiple keys sharing the same values** are permitted via multiple `<dt>` elements.

#### Constraints and Limitations

- **Do not use `<dl>` for arbitrary content**; the key–value relationship must be genuine.
- **If the items can stand alone without labels**, use `<ul>` instead.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Product Metadata

**Setup Guide**: Save as `metadata-product.html`.

```html
<dl>
  <dt>Name</dt>
  <dd>Wireless Noise-Cancelling Headphones</dd>
  <dt>Price</dt>
  <dd>$299.99</dd>
  <dt>Battery life</dt>
  <dd>30 hours</dd>
  <dt>Weight</dt>
  <dd>250 g</dd>
</dl>
```

**Expected output**: A product specification list with each label paired with its value.

**Why this output occurs**: The `<dt>` elements provide the metadata keys, and the `<dd>` elements provide their values. The `<dl>` groups them semantically.

---

#### Example 2: The W3C Metadata Example

**Setup Guide**: Save as `metadata-w3c.html`. This is the W3C specification’s metadata example.

```html
<dl>
  <dt>Last modified time</dt>
  <dd>2004-12-23T23:33Z</dd>
  <dt>Recommended update interval</dt>
  <dd>60s</dd>
  <dt>Authors</dt>
  <dt>Editors</dt>
  <dd>Robert Rothman</dd>
  <dd>Daniel Jackson</dd>
</dl>
```

**Expected output**: A metadata block where “Authors” and “Editors” share the values “Robert Rothman” and “Daniel Jackson”.

**Why this output occurs**: The specification explicitly permits multiple `<dt>` elements to share multiple `<dd>` elements, representing alternative name–value assignments in metadata.

---

### Real-World Cases

1. **Product specifications**: Dimensions, weight, battery life, price.
2. **Event details**: Date, time, location, price.
3. **Author metadata**: Name, email, affiliation.

---

### References

- MDN Web Docs — `<dl>`: Metadata — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dl#metadata
- W3C — HTML 5: The `dl` element (metadata example) — https://www.w3.org/TR/2011/WD-html5-author-20110809/the-dl-element.html

---

## 6. Description List Best Practices

### Definitions

**Core Definition**
Description list best practices are guidelines for choosing `<dl>` appropriately, structuring term–description groups correctly, handling multiple terms and descriptions, and preserving semantics when styling — so that the name–value relationship is conveyed accurately to assistive technologies and search engines.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG 1.3.1 (Info and Relationships) and Technique H40, and the specification’s explicit content model rules. They encompass using `<dl>` only for genuine name–value pairs, maintaining the `<dt>`…`<dd>` sequence, allowing multiple terms and descriptions when semantically appropriate, grouping with `<div>` only for styling, and avoiding the obsolete `compact` attribute.

**Beginner-Friendly Explanation**
Best practices help you use description lists correctly: use `<dl>` only when you have a real term–description or key–value relationship, put every term in `<dt>` and every description in `<dd>`, keep the term before the description, and use `<div>` wrappers only for styling, not to change the semantics.

---

### Purposes

- **To ensure semantic correctness** and valid markup.
- **To preserve the term–description relationship** for assistive technologies.
- **To support WCAG 3.1.3 (Unusual Words)** through Technique H40.
- **To maintain visual and structural consistency** across a site.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `<dl>` only for name–value pairs** | Terms/definitions, metadata keys/values, questions/answers. |
| **Every term must be a `<dt>` and every description a `<dd>`** | Do not use `<div>` or `<p>` as substitutes. |
| **The `<dt>` must precede the `<dd>`** | A `<dd>` without a preceding `<dt>` is non-conformant. |
| **Multiple `<dt>` elements may share `<dd>` elements** | Allowed when terms are synonyms or alternative names. |
| **Multiple `<dd>` elements may follow one `<dt>`** | Allowed when a term has multiple descriptions. |
| **Use `<div>` wrappers only for styling** | Valid in WHATWG HTML and HTML 5.2+; does not change semantics. |
| **Do not use the obsolete `compact` attribute** | Use CSS instead. |
| **Do not use `<dl>` when items can stand alone** | Use `<ul>` when each item is independent. |

#### Syntax Rules

1. **Semantic correctness**: `<dl>` for name–value pairs; `<ul>` for independent items; `<ol>` for sequences.
2. **Structural correctness**: `<dt>` before `<dd>`; one or more of each per group.
3. **Grouping discipline**: `<div>` wrappers are for styling and organisation, not for replacing `<dt>`/`<dd>`.

#### Constraints and Limitations

- **A `<dl>` that does not contain genuine name–value pairs misrepresents the content**.
- **Headings or sectioning content inside `<dt>` violate the content model**.
- **`<div>` wrappers are not permitted in all HTML versions**; check the target specification.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct vs Incorrect Description List Structure

**Setup Guide**: Save as `dl-best-practice.html`. Run an HTML validator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Description List Best Practices</title>
</head>
<body>
  <!-- GOOD: dt followed by dd, multiple terms allowed -->
  <dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language.</dd>
    <dt>CSS</dt>
    <dt>Cascading Style Sheets</dt>
    <dd>A style sheet language for describing presentation.</dd>
  </dl>

  <!-- BAD: dd without a preceding dt -->
  <!--
  <dl>
    <dd>Orphaned description</dd>
  </dl>
  -->

  <!-- BAD: div as a substitute for dt/dd -->
  <!--
  <dl>
    <div>Term</div>
    <div>Description</div>
  </dl>
  -->
</body>
</html>
```

**Expected output**: The good example produces valid, accessible markup. The bad examples violate the content model and lose the term–description association.

**Why this output occurs**: The specification requires one or more `<dt>` elements followed by one or more `<dd>` elements; a `<dd>` without a `<dt>` and a `<div>` substituted for `<dt>`/`<dd>` are both non-conformant.

---

#### Example 2: Styled Description List Without Breaking Semantics

**Setup Guide**: Save as `dl-styled.html`. Use CSS grid for a two-column layout.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Styled Description List</title>
  <style>
    dl {
      display: grid;
      grid-template-columns: max-content 1fr;
      gap: 0.5rem 1rem;
    }
    dt { font-weight: bold; }
    dd { margin: 0; }
  </style>
</head>
<body>
  <dl>
    <dt>Name</dt>
    <dd>Godzilla</dd>
    <dt>Born</dt>
    <dd>1952</dd>
    <dt>Birthplace</dt>
    <dd>Japan</dd>
  </dl>
</body>
</html>
```

**Expected output**: A two-column metadata table with bold labels and aligned values, while the `<dl>`, `<dt>`, and `<dd>` semantics remain intact.

**Why this output occurs**: CSS grid changes the visual presentation without altering the HTML structure; the term–description association is preserved for assistive technologies.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.3.1 compliance requires semantic name–value markup; audits check for `<div>`-based fake metadata lists.
2. **E-commerce**: Product specifications use `<dl>` with `<dt>` labels and `<dd>` values.
3. **Documentation sites**: Glossaries use `<dl>` with `<dfn>` terms and `<dd>` definitions.

---

### References

- W3C — Technique H40: Using description lists — https://www.w3.org/WAI/WCAG22/Techniques/html/H40
- MDN Web Docs — `<dl>`: Accessibility — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dl#accessibility
- WHATWG — The `dl` element — https://html.spec.whatwg.org/multipage/grouping-content.html#the-dl-element
- W3C — HTML 5: The `dl` element — https://www.w3.org/TR/2011/WD-html5-author-20110809/the-dl-element.html

---

## Summary Table of Description List Concepts

| Concept | Key Takeaway |
|---|---|
| **`<dl>`** | Container for name–value groups; content model is one or more `<dt>` followed by one or more `<dd>`; `<div>` wrappers permitted in WHATWG HTML and HTML 5.2+. |
| **`<dt>`** | The term or name; flow content without headings or sectioning content; multiple `<dt>` elements may share descriptions. |
| **`<dd>`** | The description or value; flow content; multiple `<dd>` elements may follow a `<dt>`. |
| **Terms and definitions** | Glossary use case; WCAG Technique H40; `<dfn>` marks the defining instance. |
| **Metadata-style lists** | Key–value pairs for product specs, author info, and event details. |
| **Best practices** | Use `<dl>` only for genuine name–value pairs; maintain the `<dt>`…`<dd>` sequence; use `<div>` only for styling. |

---

## Final Notes

- **Description lists are for name–value pairs**: If the items can stand alone without labels, use `<ul>` instead.
- **`<dt>` must precede `<dd>`**: A `<dd>` without a preceding `<dt>` is non-conformant.
- **Multiple terms and multiple descriptions are allowed**: Use them when the semantics genuinely call for synonyms or alternative values.
- **`<div>` wrappers are for styling, not semantics**: Valid in WHATWG HTML and HTML 5.2+, but the `<dl>`–`<dt>`–`<dd>` structure must remain intact.
- **WCAG Technique H40 recognises description lists** as a sufficient technique for providing term descriptions, related to SC 3.1.3.
- **Validate and test**: Use HTML validators and accessibility checkers to catch orphaned `<dd>` elements and non-conformant grouping.

---

This cheat sheet has covered description lists — the `<dl>` container, `<dt>` terms, `<dd>` descriptions, the terms-and-definitions glossary use case, the metadata key–value use case, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate description list markup against the WHATWG HTML Living Standard and WCAG 1.3.1 (Info and Relationships) with Technique H40.