# Text-Level Semantics — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Text-level semantics are the HTML elements that describe the meaning, structure, or role of small runs of text — words, phrases, names, numbers, dates, and code. They operate at the **phrasing content** level, sitting inside paragraphs, headings, list items, and other phrasing containers, and they add semantic richness without breaking the flow of text.

**Technical Definition**
In the WHATWG HTML Living Standard, the relevant chapter is “Text-level semantics”, which defines `<a>`, `<em>`, `<strong>`, `<small>`, `<s>`, `<cite>`, `<q>`, `<dfn>`, `<abbr>`, `<ruby>`, `<rt>`, `<rp>`, `<data>`, `<time>`, `<code>`, `<var>`, `<samp>`, `<kbd>`, `<sub>`, `<sup>`, `<i>`, `<b>`, `<u>`, `<mark>`, `<bdi>`, `<bdo>`, `<span>`, `<br>`, and `<wbr>`. Most belong to the **phrasing content** and **palpable content** categories, and some (such as `<a>` and `<s>`) also belong to **flow content** or **interactive content** depending on their attributes and context. Their content models are generally phrasing content, with specific exceptions such as `<ruby>`, `<bdo>`, and `<a>`.

**Beginner-Friendly Explanation**
Text-level semantics are the “coloured pens” you use to annotate text inside a sentence. Instead of just plain words, you can mark a phrase as important (`<strong>`), as emphasised (`<em>`), as a citation (`<cite>`), as a code snippet (`<code>`), as an abbreviation (`<abbr>`), or as a highlighted search result (`<mark>`). They do not create new blocks of content; they enrich existing text.

---

### Key Characteristics

- **Phrasing content by default**: Nearly all text-level semantic elements belong to the phrasing content category and may appear inside paragraphs, headings, and other phrasing containers.
- **Meaning over appearance**: Elements such as `<em>`, `<strong>`, `<cite>`, `<dfn>`, and `<time>` are defined by their meaning, not by the visual style browsers give them.
- **Presentation sometimes differs from meaning**: `<b>`, `<i>`, and `<u>` are explicitly defined by the HTML standard as having *no* exact semantic meaning beyond a general indication (attention, alternate voice, unarticulated annotation). They are not interchangeable with `<strong>`, `<em>`, and semantic marking.
- **Machine-readable value support**: `<data>` and `<time>` provide machine-readable values via the `value` and `datetime` attributes.
- **Bidirectional support**: `<bdi>` and `<bdo>` handle bidirectional text flow, important for multilingual content.
- **Breaking helpers**: `<br>` and `<wbr>` control line breaks and break opportunities.
- **Accessibility relevance**: Correct text-level semantics improve screen-reader pronunciation, translation, and machine interpretation.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<html>`, `<head>`, `<body>`, and paragraphs (`<p>`).
2. **Phrasing content vs flow content**: knowing which elements can appear inside a paragraph.
3. **Introductory CSS**: understanding that browsers apply default styles to some elements (e.g. italics for `<em>`, bold for `<strong>`).
4. **Browser developer tools**: the Elements panel for inspecting the DOM and the Accessibility panel for observing semantics.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | Emphasis, strong importance, headings, abbreviations, and language marking improve pronunciation and comprehension. |
| **CSS typography** | Many text-level elements have default styling that authors override; knowing the semantic meaning prevents misuse. |
| **Internationalisation (i18n)** | `<bdi>`, `<bdo>`, `lang`, and `<ruby>` are essential for multilingual content. |
| **SEO** | `<strong>`, `<em>`, `<cite>`, and `<time>` provide machine-readable signals to search engines. |
| **Microformats and structured data** | `<time>`, `<data>`, and `<abbr>` are often used with `itemprop` for schema.org markup. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. `<a>` — hyperlink
2. `<em>` — emphasis
3. `<strong>` — strong importance
4. `<small>` — side comment / fine print
5. `<s>` — no longer accurate
6. `<cite>` — citation / title of a work
7. `<q>` — inline quotation
8. `<dfn>` — defining instance of a term
9. `<abbr>` — abbreviation or acronym
10. `<ruby>` — ruby annotation container
11. `<rt>` — ruby text
12. `<rp>` — ruby parenthesis
13. `<data>` — machine-readable data
14. `<time>` — machine-readable date/time
15. `<code>` — code fragment
16. `<var>` — variable
17. `<samp>` — sample output
18. `<kbd>` — keyboard input
19. `<sub>` — subscript
20. `<sup>` — superscript
21. `<i>` — idiomatic text
22. `<b>` — bring attention to
23. `<u>` — unarticulated annotation
24. `<mark>` — marked/highlighted text
25. `<bdi>` — bidirectional isolation
26. `<bdo>` — bidirectional override
27. `<span>` — generic phrasing container
28. `<br>` — line break
29. `<wbr>` — word break opportunity
30. **Text-level semantic best practices**

---

## 1. `<a>` — The Anchor (Hyperlink) Element

### Definitions

**Core Definition**
The `<a>` element, with its `href` attribute, creates a hyperlink to web pages, files, email addresses, locations within the same page, or anything else a URL can address.

**Technical Definition**
`<a>` belongs to the **flow content**, **phrasing content**, **interactive content** (when `href` is present), and **palpable content** categories. Its content model is **transparent**, but with the constraint that it must contain no interactive content descendants and no `<a>` element descendants. Key attributes include `href`, `target`, `rel`, `download`, `hreflang`, `type`, and `referrerpolicy`. When `href` is absent, the element becomes a placeholder link.

**Beginner-Friendly Explanation**
The `<a>` element is a link. With `href`, it takes you somewhere when you click it — another page, a file, an email address, or a section of the same page.

---

### Purposes

- **To create hyperlinks** to other resources.
- **To link to a section within the current page** using fragment identifiers.
- **To create email and phone links** using `mailto:` and `tel:` schemes.
- **To enable file downloads** using the `download` attribute.
- **To define link relationships** for SEO, security, and browser behaviour via `rel`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Visit Example
</a>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `href` | URL of the linked resource. |
| `target` | Browsing context (`_self`, `_blank`, `_parent`, `_top`). |
| `rel` | Relationship keywords (`noopener`, `noreferrer`, `nofollow`, `external`). |
| `download` | Prompts download instead of navigation; optional filename value. |
| `hreflang` | Language of the linked resource. |
| `type` | MIME type hint for the linked resource. |

#### Syntax Rules

1. **Content model is transparent** but may not contain interactive content or another `<a>`.
2. **Without `href`, `<a>` is a placeholder link** and is not interactive.
3. **`target="_blank"` should be paired with `rel="noopener"`** to prevent window.opener attacks.
4. **`mailto:` and `tel:` schemes** create email and phone links.
5. **`download` works only for same-origin URLs** in most browsers.

#### Constraints and Limitations

- The `<a>` element must not contain a `<button>` or another `<a>`.
- Fragments (`#id`) must match an existing element ID for in-page links.
- Link text must be meaningful out of context for accessibility; avoid "click here".

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: External Link and Email Link

**Setup guide**: Save as `a-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Anchor Example</title>
</head>
<body>
  <p>
    Visit
    <a href="https://developer.mozilla.org/" target="_blank" rel="noopener noreferrer">
      MDN Web Docs
    </a>
    for reference material.
  </p>

  <p>
    Contact:
    <a href="mailto:author@example.com">author@example.com</a>
  </p>
</body>
</html>
```

**Expected output**: A link reading “MDN Web Docs” that opens in a new tab when clicked, and an email link that opens the user's mail client.

**Why this output occurs**: The `href` attribute creates a navigable link. `target="_blank"` opens the destination in a new browsing context, and `rel="noopener noreferrer"` prevents the new page from accessing `window.opener` and strips the referrer. The `mailto:` scheme activates the mail client.

---

#### Example 2: In-Page Fragment Link

**Setup guide**: Save as `a-fragment.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Fragment Link</title>
</head>
<body>
  <nav>
    <a href="#section-two">Jump to Section Two</a>
  </nav>

  <section id="section-one">
    <h2>Section One</h2>
    <p>First section content.</p>
  </section>

  <section id="section-two">
    <h2>Section Two</h2>
    <p>Second section content.</p>
  </section>
</body>
</html>
```

**Expected output**: Clicking “Jump to Section Two” scrolls the page to the section with `id="section-two"`.

**Why this output occurs**: The fragment `#section-two` in the link’s `href` resolves to the element with `id="section-two"`. The browser scrolls to that element and updates the URL with the fragment.

---

### Real-World Cases

1. **Site navigation**: Every internal navigation link uses `<a>`.
2. **External references**: Links to documentation, social media, and sources.
3. **Email and phone CTAs**: `mailto:` and `tel:` links on contact pages.

---

### References

- WHATWG — The `a` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-a-element
- MDN Web Docs — `<a>`: The Anchor element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a
- W3C — HTML 5.2: The `a` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-a-element

---

## 2. `<em>` — The Emphasis Element

### Definitions

**Core Definition**
The `<em>` element marks text that has stress emphasis — text that would be pronounced differently or read with emphasis.

**Technical Definition**
`<em>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Its level of emphasis is determined by nesting: `<em>` inside `<em>` increases emphasis. Browsers typically render `<em>` in italics, but the italics are a presentation, not the meaning.

**Beginner-Friendly Explanation**
`<em>` marks a word or phrase that you would emphasise when speaking. It changes the meaning of the sentence depending on which word is emphasised.

---

### Purposes

- **To indicate stress emphasis** that changes the meaning or pronunciation of a sentence.
- **To allow nesting for increased emphasis**.
- **To signal to screen readers** that the text should be stressed.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>I <em>never</em> said that.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<em>` | The emphasis wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Nesting increases emphasis**: `<em><em>text</em></em>` is more emphasised.
3. **Do not use `<em>` merely for italic styling**; use CSS when the purpose is presentational.

#### Constraints and Limitations

- `<em>` is not interchangeable with `<i>`; `<i>` is for alternate voice or mood, not emphasis.
- Overuse dilutes meaning; use it sparingly where genuine emphasis exists.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Emphasis That Changes Meaning

**Setup guide**: Save as `em-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Emphasis</title>
</head>
<body>
  <p>Cats are <em>not</em> allowed on the counter.</p>
  <p><em>Cats</em> are not allowed on the counter — but dogs are.</p>
</body>
</html>
```

**Expected output**: In the first sentence, “not” is italicised. In the second, “Cats” is italicised.

**Why this output occurs**: The `<em>` element marks the emphasised word. The emphasis changes which word receives stress, and therefore changes the meaning of the sentence. The browser renders emphasis as italics by default.

---

#### Example 2: Nested Emphasis

**Setup guide**: Save as `em-nested.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Emphasis</title>
</head>
<body>
  <p>
    I said <em>you <em>really</em> need to be careful</em>.
  </p>
</body>
</html>
```

**Expected output**: The whole phrase is italicised, with “really” emphasised more strongly.

**Why this output occurs**: Nested `<em>` elements increase the level of emphasis. Browsers may render the deeper emphasis with stronger italics or additional stress markers.

---

### Real-World Cases

1. **Editorial copy**: Emphasis on contrasting or correct-in-context words.
2. **Instructional text**: Highlighting words where mispronunciation changes meaning.
3. **UX writing**: Warning text where a specific word is critical.

---

### References

- WHATWG — The `em` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-em-element
- MDN Web Docs — `<em>`: The Emphasis element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/em
- W3C — HTML 5.2: The `em` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-em-element

---

## 3. `<strong>` — The Strong Importance Element

### Definitions

**Core Definition**
The `<strong>` element indicates that its contents have strong importance, seriousness, or urgency. Browsers typically render it in bold, but the bold is a presentation, not the meaning.

**Technical Definition**
`<strong>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The `strong` element conveys importance in a way similar to how `<em>` conveys emphasis; the two are often used together.

**Beginner-Friendly Explanation**
`<strong>` marks text that is important — warnings, critical instructions, or key takeaways. It changes how seriously you should take the content.

---

### Purposes

- **To indicate strong importance**, seriousness, or urgency.
- **To convey warnings and critical information** to users, including screen readers.
- **To signal key takeaways** in documentation and articles.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p><strong>Warning:</strong> Do not open this file.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<strong>` | The strong importance wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **`<strong>` is not merely bold styling**; use CSS when the purpose is presentational.
3. **`<strong>` may be nested** for higher importance levels.
4. **Combine with `<em>`** when a phrase is both emphasised and important.

#### Constraints and Limitations

- Do not use `<strong>` for headings; use heading elements.
- Do not use `<strong>` for decorative bold; use CSS.
- Overuse reduces impact; reserve it for genuinely important content.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Warning

**Setup guide**: Save as `strong-warning.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Strong Warning</title>
</head>
<body>
  <p><strong>Warning:</strong> This action cannot be undone.</p>
</body>
</html>
```

**Expected output**: “Warning:” appears in bold, drawing attention to the critical information that follows.

**Why this output occurs**: `<strong>` marks the word “Warning” as strongly important. The browser renders it in bold by default. Screen readers may announce it with emphasis.

---

#### Example 2: Combining Strong and Em

**Setup guide**: Save as `strong-em.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Strong and Em</title>
</head>
<body>
  <p>
    <strong>Do not <em>ever</em> share your password.</strong>
  </p>
</body>
</html>
```

**Expected output**: The whole sentence is bold, and “ever” is also italicised.

**Why this output occurs**: `<strong>` marks the sentence as strongly important, and the nested `<em>` marks “ever” as additionally emphasised. The two combine to convey urgency and stress.

---

### Real-World Cases

1. **Form validation messages**: Critical errors are marked `<strong>`.
2. **Documentation warnings**: Safety notes are `<strong>`.
3. **Key takeaways**: Article summaries use `<strong>` for key points.

---

### References

- WHATWG — The `strong` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-strong-element
- MDN Web Docs — `<strong>`: The Strong Importance element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/strong
- W3C — HTML 5.2: The `strong` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-strong-element

---

## 4. `<small>` — The Side Comment Element

### Definitions

**Core Definition**
The `<small>` element represents side comments and small print — content typically legally required, such as copyright notices, disclaimers, or attributions.

**Technical Definition**
`<small>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The element indicates small print and side comments; browsers render it in a smaller font by default, but the semantic meaning is “side comment”, not merely “smaller text”.

**Beginner-Friendly Explanation**
`<small>` is for fine print — the copyright line, terms of service links, legal disclaimers, and side notes that accompany main content but are not the main content.

---

### Purposes

- **To mark up legal fine print** such as copyright and disclaimer text.
- **To add side comments** to phrases or sentences.
- **To indicate attribution** in a visually smaller way.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>
  <small>© 2026 Example Company. All rights reserved.</small>
</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<small>` | The side-comment wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<small>` for side comments**, not for general small text.
3. **Do not use `<small>` for text that should be less important**; use `<strong>` or hierarchy instead.

#### Constraints and Limitations

- `<small>` should not be used to satisfy minimum contrast requirements by making text smaller in a way that fails accessibility.
- The element cannot be used to shrink text below legible size; CSS controls display size.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Copyright Line

**Setup guide**: Save as `small-copyright.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Small Print</title>
</head>
<body>
  <footer>
    <small>© 2026 Example Company. All rights reserved.</small>
  </footer>
</body>
</html>
```

**Expected output**: A footer containing a copyright notice rendered in smaller text.

**Why this output occurs**: `<small>` marks the copyright text as fine print. The browser renders it smaller by default, and screen readers may announce it with a “small” or “fine print” connotation.

---

#### Example 2: Inline Side Comment

**Setup guide**: Save as `small-inline.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline Side Comment</title>
</head>
<body>
  <p>
    Tickets are $50 <small>(service fees apply)</small>.
  </p>
</body>
</html>
```

**Expected output**: The main sentence is normal size, and “(service fees apply)” appears in smaller text.

**Why this output occurs**: The `<small>` element marks the parenthetical as a side comment — important information but distinct from the main statement. The smaller rendering conveys its supporting role.

---

### Real-World Cases

1. **Footers**: Copyright, trademark notices, and privacy disclaimers.
2. **Pricing**: “Taxes not included” or “Service fees apply”.
3. **Attributions**: Photo credit lines.

---

### References

- WHATWG — The `small` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-small-element
- MDN Web Docs — `<small>`: The side comment element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/small
- W3C — HTML 5.2: The `small` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-small-element

---

## 5. `<s>` — The Strikethrough Element

### Definitions

**Core Definition**
The `<s>` element represents contents that are no longer accurate or no longer relevant. Browsers typically render it with a strikethrough line.

**Technical Definition**
`<s>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. It indicates that the content is no longer accurate or relevant, not that a document has been edited (which is `<del>` and `<ins>`).

**Beginner-Friendly Explanation**
`<s>` marks text as outdated or no longer accurate — like a price that has changed or a piece of information that was corrected.

---

### Purposes

- **To indicate that text is no longer accurate** or relevant.
- **To show updated information** such as an old price.
- **To mark deprecated information** in announcements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p><s>$50.00</s> $35.00</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<s>` | The “no longer accurate” wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<s>` for outdated information**, not for document edits (use `<del>` for deletions).
3. **Do not use `<s>` for text you merely want to strike through for decoration**.

#### Constraints and Limitations

- `<s>` is not for spelling corrections or document revision; use `<del>` and `<ins>` for editing semantics.
- Screen readers may or may not announce `<s>` distinctly; do not rely on it for critical meaning.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Price Change

**Setup guide**: Save as `s-price.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Price Change</title>
</head>
<body>
  <p>Original price: <s>$50.00</s> — now $35.00.</p>
</body>
</html>
```

**Expected output**: The old price “$50.00” is shown with a line through it, and the new price “$35.00” is shown normally.

**Why this output occurs**: `<s>` marks the old price as no longer accurate. The browser applies strikethrough styling, and the semantic meaning is that the price is outdated.

---

#### Example 2: Corrected Information

**Setup guide**: Save as `s-corrected.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Corrected Information</title>
</head>
<body>
  <p>The event is at <s>3 PM</s> 4 PM.</p>
</body>
</html>
```

**Expected output**: “3 PM” is struck through and “4 PM” appears normally.

**Why this output occurs**: `<s>` marks the original time as no longer accurate, and the corrected time follows. The element conveys that the information changed.

---

### Real-World Cases

1. **E-commerce**: Prices before and after a sale.
2. **Announcements**: Updated meeting times or corrected information.
3. **Documentation**: Deprecated version numbers.

---

### References

- WHATWG — The `s` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-s-element
- MDN Web Docs — `<s>`: The Strikethrough element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/s
- W3C — HTML 5.2: The `s` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-s-element

---

## 6. `<cite>` — The Citation Element

### Definitions

**Core Definition**
The `<cite>` element represents the title of a creative work — a book, article, poem, song, screenplay, film, painting, sculpture, or similar — from which a quotation or reference is drawn.

**Technical Definition**
`<cite>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The element is for the *title* of a work, not for the author’s name or a URL. `cite` is often used alongside `<blockquote>` and `<q>`.

**Beginner-Friendly Explanation**
`<cite>` marks the title of a creative work — the name of a book, a movie, an article, or a song. It does not mark the author; it marks the work itself.

---

### Purposes

- **To identify the title of a referenced work**.
- **To distinguish work titles** from surrounding prose.
- **To support citation and attribution** in articles and essays.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>As <cite>The Great Gatsby</cite> shows…</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<cite>` | The citation wrapper. |
| Content | Phrasing content — the title of the work. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<cite>` only for titles of works**, not for author names or URLs.
3. **Browsers typically render `<cite>` in italics**, but italics are a presentation.

#### Constraints and Limitations

- Some guidance suggests `<cite>` can also be used for the *name* of a work’s author, but the WHATWG definition is limited to titles.
- Do not use `<cite>` for the URL of a source; use `<a>` with `href`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Citing a Book Title

**Setup guide**: Save as `cite-book.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Cite Example</title>
</head>
<body>
  <p>My favourite novel is <cite>The Left Hand of Darkness</cite>.</p>
</body>
</html>
```

**Expected output**: “The Left Hand of Darkness” is shown in italics.

**Why this output occurs**: `<cite>` marks the book title as the title of a work. Browsers render it in italics by default. The semantic meaning is more precise than `<i>` because it specifically identifies a creative work title.

---

#### Example 2: Citation with Quotation

**Setup guide**: Save as `cite-quote.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Cite with Quote</title>
</head>
<body>
  <blockquote cite="https://example.com/essay">
    <p>The only thing we have to fear is fear itself.</p>
  </blockquote>
  <p>— <cite>First Inaugural Address</cite></p>
</body>
</html>
```

**Expected output**: A quotation followed by the cited title in italics.

**Why this output occurs**: The `<cite>` element marks the title of the work being quoted. The attribution is outside the `<blockquote>`, as required by the specification.

---

### Real-World Cases

1. **Editorial content**: Book reviews and cultural commentary.
2. **Academic writing**: References to publications.
3. **Documentation**: Naming the specifications or standards referenced.

---

### References

- WHATWG — The `cite` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-cite-element
- MDN Web Docs — `<cite>`: The Citation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/cite
- W3C — HTML 5.2: The `cite` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-cite-element

---

## 7. `<q>` — The Inline Quotation Element

### Definitions

**Core Definition**
The `<q>` element indicates that the enclosed text is a short inline quotation. Browsers typically add quotation marks around the content, controlled by CSS.

**Technical Definition**
`<q>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. It has an optional `cite` attribute providing a URL for the source. When used, the browser generates quotation marks via `::before` and `::after` CSS pseudo-elements.

**Beginner-Friendly Explanation**
`<q>` is for a short quotation inside a sentence. Browsers automatically add quotation marks around it. For long quotations, use `<blockquote>`.

---

### Purposes

- **To mark short inline quotations**.
- **To allow the browser to add language-appropriate quotation marks** automatically.
- **To provide machine-readable source information** via the `cite` attribute.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>She said, <q cite="https://example.com/quote">Hello there</q>.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<q>` | The inline quotation wrapper. |
| `cite` | Optional URL of the quotation source. |
| Content | Phrasing content (the quotation). |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Do not manually add quotation marks** inside `<q>`; the browser inserts them.
3. **Use `<blockquote>` for long quotations**; `<q>` is for short inline ones.

#### Constraints and Limitations

- The `cite` attribute is not rendered visibly.
- CSS can change the quotation marks via `quotes` and `content`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline Quotation

**Setup guide**: Save as `q-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline Quote</title>
</head>
<body>
  <p>The guide says, <q>always test your code</q>.</p>
</body>
</html>
```

**Expected output**: The sentence displays with quotation marks around “always test your code”, added automatically by the browser.

**Why this output occurs**: The `<q>` element generates language-appropriate quotation marks through CSS defaults. The quotation text is wrapped in the appropriate quote characters for the document’s language.

---

#### Example 2: Nested Quotations

**Setup guide**: Save as `q-nested.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Quotes</title>
</head>
<body>
  <p>
    He said, <q>She told me, <q>It is done</q>, and left.</q>
  </p>
</body>
</html>
```

**Expected output**: Nested quotation marks — outer double quotes and inner single quotes.

**Why this output occurs**: The browser applies alternate quote styles for nested `<q>` elements, following the `quotes` CSS property defaults, producing a visually correct nested quotation.

---

### Real-World Cases

1. **Editorial writing**: Quoting speech within sentences.
2. **Reviews**: Quoting lines from works.
3. **Interviews**: Quoting short answers inline.

---

### References

- WHATWG — The `q` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-q-element
- MDN Web Docs — `<q>`: The Inline Quotation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/q
- W3C — HTML 5.2: The `q` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-q-element

---

## 8. `<dfn>` — The Definition Element

### Definitions

**Core Definition**
The `<dfn>` element indicates the defining instance of a term in a document — the point at which the term is being defined.

**Technical Definition**
`<dfn>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content, but with restrictions: no `<dfn>` descendants. The defining instance is the `<dfn>` element inside the nearest ancestor sectioning content or sectioning root, provided it is the first `<dfn>` for that term.

**Beginner-Friendly Explanation**
`<dfn>` marks the first time a term is defined in a document. It is used when introducing a new technical term or concept.

---

### Purposes

- **To mark the defining instance of a term**.
- **To allow other elements to reference the definition** (e.g. via `href` or `aria-labelledby`).
- **To provide machine-readable definition context** for glossaries and documentation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p><dfn>HTML</dfn> is the standard markup language for web pages.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<dfn>` | The defining-instance wrapper. |
| Content | Phrasing content (the term being defined). |

#### Syntax Rules

1. **Content model is phrasing content**, with no `<dfn>` descendants.
2. **The `<dfn>` element should be the first occurrence of the term** in the document.
3. **The definition should follow the term** in nearby content.

#### Constraints and Limitations

- Do not use `<dfn>` for every occurrence of a term; only the defining instance.
- `<dfn>` should not contain interactive content or other definitional markup.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Defining a Term

**Setup guide**: Save as `dfn-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Definition</title>
</head>
<body>
  <p>
    <dfn id="def-html">HTML</dfn> is the standard markup language for web pages.
    Once you learn HTML, you can write HTML by hand.
  </p>
</body>
</html>
```

**Expected output**: The first occurrence of “HTML” appears in italics (default styling), and the term is defined by the sentence.

**Why this output occurs**: `<dfn>` marks the defining instance. Browsers default to italics. The `id` allows other elements to link back to this definition.

---

#### Example 2: Abbreviation with Definition

**Setup guide**: Save as `dfn-abbr.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>DFN with ABBR</title>
</head>
<body>
  <p>
    <dfn id="def-html"><abbr title="HyperText Markup Language">HTML</abbr></dfn>
    is the standard markup language for web pages.
  </p>
</body>
</html>
```

**Expected output**: “HTML” is italicised and, on hover, shows the expansion “HyperText Markup Language”.

**Why this output occurs**: The `<dfn>` marks the term as the defining instance, and the `<abbr>` provides the expansion. This is a common pattern for introducing acronyms.

---

### Real-World Cases

1. **Technical documentation**: Defining new terms on first use.
2. **Glossaries**: Anchoring definitions for linking.
3. **Educational content**: Introducing new concepts.

---

### References

- WHATWG — The `dfn` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-dfn-element
- MDN Web Docs — `<dfn>`: The Definition element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dfn
- W3C — HTML 5.2: The `dfn` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-dfn-element

---

## 9. `<abbr>` — The Abbreviation Element

### Definitions

**Core Definition**
The `<abbr>` element represents an abbreviation or acronym, optionally with its expansion provided in the `title` attribute.

**Technical Definition**
`<abbr>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The optional `title` attribute contains the expansion. Browsers may show the expansion as a tooltip on hover.

**Beginner-Friendly Explanation**
`<abbr>` marks an abbreviation or acronym. You can give it a `title` attribute containing the full form, so users can hover to see the expansion.

---

### Purposes

- **To mark abbreviations and acronyms**.
- **To provide the expansion** via the `title` attribute.
- **To make terms machine-readable** for translation and search.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<abbr title="HyperText Markup Language">HTML</abbr>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `title` | Full expansion of the abbreviation. |
| Content | The abbreviation itself. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **The `title` attribute is optional** but recommended when the expansion is not obvious.
3. **Do not use `<abbr>` for every shortened word**; reserve it for genuine abbreviations.

#### Constraints and Limitations

- Tooltips on `title` are inaccessible on touch devices; consider spelling out the expansion at first use.
- Screen readers may not consistently announce `title`; provide inline expansion where possible.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Abbreviation with Expansion

**Setup guide**: Save as `abbr-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Abbreviation</title>
</head>
<body>
  <p>
    <abbr title="HyperText Markup Language">HTML</abbr> is the backbone of the web.
  </p>
</body>
</html>
```

**Expected output**: “HTML” is marked as an abbreviation. Hovering over it shows the tooltip “HyperText Markup Language”.

**Why this output occurs**: `<abbr>` indicates the text is an abbreviation, and the `title` attribute provides its expansion. Browsers may display a dotted underline and a hover tooltip.

---

#### Example 2: Acronym with CSS Styling

**Setup guide**: Save as `abbr-styled.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Styled Abbreviation</title>
  <style>
    abbr[title] { border-bottom: 1px dotted; cursor: help; }
  </style>
</head>
<body>
  <p>
    The <abbr title="World Wide Web Consortium">W3C</abbr> sets web standards.
  </p>
</body>
</html>
```

**Expected output**: “W3C” appears with a dotted underline and a help cursor, indicating that hovering reveals the expansion.

**Why this output occurs**: The CSS rule styles `<abbr>` elements that have a `title`, providing visual cues that more information is available. The `title` attribute supplies the expansion.

---

### Real-World Cases

1. **Technical writing**: Acronyms in specifications and guides.
2. **Legal and medical text**: Abbreviations with first-use expansions.
3. **Multilingual content**: Marking abbreviations that may be unfamiliar.

---

### References

- WHATWG — The `abbr` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-abbr-element
- MDN Web Docs — `<abbr>`: The Abbreviation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/abbr
- W3C — HTML 5.2: The `abbr` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-abbr-element

---

## 10. `<ruby>` — The Ruby Annotation Element

### Definitions

**Core Definition**
The `<ruby>` element represents ruby annotations — small annotations rendered alongside base text, commonly used in East Asian typography to show pronunciation.

**Technical Definition**
`<ruby>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content, but it is designed to contain ruby base text and ruby text elements (`<rt>` and `<rp>`). Browsers render the ruby text above the base text.

**Beginner-Friendly Explanation**
`<ruby>` is used in Japanese, Chinese, and Korean text to show pronunciation above characters. The base characters and their pronunciation are placed inside `<ruby>`, with `<rt>` giving the pronunciation.

---

### Purposes

- **To provide pronunciation guides** above East Asian characters.
- **To annotate base text** with small guiding text.
- **To support multilingual typography**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ruby>
  漢 <rp>(</rp><rt>kan</rt><rp>)</rp>
  字 <rp>(</rp><rt>ji</rt><rp>)</rp>
</ruby>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<ruby>` | The ruby container. |
| Base text | The main characters to annotate. |
| `<rt>` | Ruby text — pronunciation or annotation. |
| `<rp>` | Ruby parenthesis — fallback parentheses for browsers that do not support ruby. |

#### Syntax Rules

1. **Content model is phrasing content**, but contains ruby base text and `<rt>`/`<rp>` children.
2. **Base text is not wrapped** in a dedicated element; it sits directly inside `<ruby>`.
3. **`<rt>` and `<rp>` must be children of `<ruby>`**.

#### Constraints and Limitations

- Ruby support varies; `<rp>` provides fallback parentheses for unsupported browsers.
- Ruby is mainly used in East Asian typography; for other languages, alternative notation is often clearer.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Japanese Ruby Annotation

**Setup guide**: Save as `ruby-japanese.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Ruby Example</title>
</head>
<body>
  <p>
    <ruby>
      漢 <rp>(</rp><rt>kan</rt><rp>)</rp>
      字 <rp>(</rp><rt>ji</rt><rp>)</rp>
    </ruby>
    は日本語です。
  </p>
</body>
</html>
```

**Expected output**: The characters 漢字 appear with small “kan” and “ji” above them (ruby annotation). In browsers without ruby support, the parentheses and readings appear inline as fallback.

**Why this output occurs**: The `<ruby>` element marks the base text and its reading. The `<rt>` elements contain the pronunciations, which the browser renders above the base characters. The `<rp>` elements provide parentheses as fallback.

---

#### Example 2: Chinese Ruby Annotation

**Setup guide**: Save as `ruby-chinese.html`.

```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>Chinese Ruby</title>
</head>
<body>
  <p>
    <ruby>
      北京 <rp>(</rp><rt>Běijīng</rt><rp>)</rp>
    </ruby>
    是中国的首都。
  </p>
</body>
</html>
```

**Expected output**: “北京” is annotated with “Běijīng” above the characters.

**Why this output occurs**: The ruby base text is “北京”, and the `<rt>` provides the pinyin reading. The `<rp>` provides fallback parentheses.

---

### Real-World Cases

1. **Japanese learning materials**: Furigana above kanji.
2. **Chinese textbooks**: Pinyin above hanzi.
3. **Korean content**: Hanja readings above hangul.

---

### References

- WHATWG — The `ruby` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-ruby-element
- MDN Web Docs — `<ruby>`: The Ruby Annotation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ruby
- W3C — HTML Ruby Markup Extensions — https://www.w3.org/TR/html-ruby-extensions/

---

## 11. `<rt>` — The Ruby Text Element

### Definitions

**Core Definition**
The `<rt>` element specifies the ruby text component of a ruby annotation — the small text, typically a pronunciation or translation, rendered alongside the base text.

**Technical Definition**
`<rt>` belongs to the **phrasing content** category. It must be a child of `<ruby>`. Its content model is phrasing content. It provides the annotation for the preceding ruby base text.

**Beginner-Friendly Explanation**
`<rt>` is the pronunciation that sits above a character inside a `<ruby>` element. It is the “reading” of the base character.

---

### Purposes

- **To supply the pronunciation or annotation** for ruby base text.
- **To provide translations or glosses** for characters.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ruby>漢<rt>kan</rt></ruby>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<rt>` | The ruby text. |
| Content | The annotation text (pronunciation, translation). |

#### Syntax Rules

1. **`<rt>` must be a child of `<ruby>`**.
2. **Content model is phrasing content**.
3. **Each `<rt>` applies to the preceding ruby base text**.

#### Constraints and Limitations

- `<rt>` must always follow the base text it annotates.
- Browsers without ruby support will ignore `<rt>` unless `<rp>` fallback is also present.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Ruby Text

**Setup guide**: Save as `rt-basic.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Ruby Text</title>
</head>
<body>
  <p><ruby>漢<rt>kan</rt></ruby></p>
</body>
</html>
```

**Expected output**: “漢” appears with “kan” rendered above it.

**Why this output occurs**: The `<rt>` provides the pronunciation annotation for the base text “漢”. The browser renders it as ruby text above the character.

---

#### Example 2: Ruby Text for a Phrase

**Setup guide**: Save as `rt-phrase.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Phrase Ruby</title>
</head>
<body>
  <p><ruby>東京<rt>とうきょう</rt></ruby>は日本の首都です。</p>
</body>
</html>
```

**Expected output**: The characters 東京 have “とうきょう” (Tōkyō in hiragana) above them.

**Why this output occurs**: The ruby base is “東京” and the `<rt>` provides its hiragana reading. The browser renders the small reading above the base characters.

---

### Real-World Cases

1. **Japanese furigana**: Kanji readings for learners.
2. **Chinese pinyin**: Pronunciation of hanzi.
3. **Children’s books**: Pronunciation guides above difficult characters.

---

### References

- WHATWG — The `rt` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-rt-element
- MDN Web Docs — `<rt>`: The Ruby Text element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/rt
- W3C — HTML 5.2: The `rt` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-rt-element

---

## 12. `<rp>` — The Ruby Parenthesis Element

### Definitions

**Core Definition**
The `<rp>` element provides fallback parentheses for ruby annotations in browsers that do not support `<ruby>`.

**Technical Definition**
`<rp>` belongs to the **phrasing content** category and must be a child of `<ruby>`. It is rendered only when the browser does not support ruby annotation; in supporting browsers, it is hidden by the ruby rendering rules.

**Beginner-Friendly Explanation**
`<rp>` is the fallback parentheses around ruby text. In browsers that support ruby, it is invisible. In browsers that do not, it shows parentheses around the pronunciation so the annotation still makes sense inline.

---

### Purposes

- **To provide parentheses** around ruby text for non-supporting browsers.
- **To preserve the readability** of ruby annotations in older environments.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ruby>漢<rp>(</rp><rt>kan</rt><rp>)</rp></ruby>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<rp>` | The fallback parenthesis. |
| Content | Typically a parenthesis character. |

#### Syntax Rules

1. **`<rp>` must be a child of `<ruby>`**.
2. **Content is usually a parenthesis or similar marker**.
3. **In ruby-supporting browsers, `<rp>` content is hidden**.

#### Constraints and Limitations

- `<rp>` content is not rendered in browsers with full ruby support.
- Without `<rp>`, the fallback for older browsers will show `<rt>` directly after the base text without parentheses, which may be harder to read.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fallback Parentheses

**Setup guide**: Save as `rp-basic.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Ruby Fallback</title>
</head>
<body>
  <p>
    <ruby>
      漢 <rp>(</rp><rt>kan</rt><rp>)</rp>
      字 <rp>(</rp><rt>ji</rt><rp>)</rp>
    </ruby>
  </p>
</body>
</html>
```

**Expected output**: In modern browsers, the pronunciation “kan” and “ji” appear above 漢 and 字, with no parentheses visible. In older browsers without ruby support, the text appears as “漢 (kan) 字 (ji)”.

**Why this output occurs**: `<rp>` provides fallback parentheses. Ruby-supporting browsers hide `<rp>` content. Non-supporting browsers display it, so the annotation reads naturally inline.

---

### Real-World Cases

1. **Japanese textbooks**: Ensuring readability in older e-readers.
2. **Cross-platform publishing**: Publishing systems that render in varied environments.

---

### References

- WHATWG — The `rp` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-rp-element
- MDN Web Docs — `<rp>`: The Ruby Fallback Parenthesis element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/rp
- W3C — HTML 5.2: The `rp` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-rp-element

---

## 13. `<data>` — The Machine-Readable Data Element

### Definitions

**Core Definition**
The `<data>` element links a piece of content to a machine-readable translation via its `value` attribute. It is useful when content is human-readable but requires a machine-readable equivalent.

**Technical Definition**
`<data>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. It has a required `value` attribute containing the machine-readable form. The element’s content is the human-readable form; the `value` attribute holds the machine-readable form.

**Beginner-Friendly Explanation**
`<data>` pairs human-readable content with machine-readable data. For example, a product name shown to a user could have a product code attached for scripts and search engines.

---

### Purposes

- **To provide machine-readable values** for human-readable content.
- **To support scripts and structured data** that need numeric or coded values.
- **To maintain both a display form and a data form** without duplication.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<data value="UPC-123456">Product Name</data>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `value` | **Required.** The machine-readable value. |
| Content | The human-readable content. |

#### Syntax Rules

1. **`value` is required**.
2. **Content model is phrasing content**.
3. **Use `<data>` when the human-readable form differs from the machine-readable form**; otherwise use the text directly.

#### Constraints and Limitations

- Do not use `<data>` when the display text is the same as the value; it is redundant.
- For dates and times, use `<time>` instead.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Product Code

**Setup guide**: Save as `data-product.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Data Element</title>
</head>
<body>
  <ul>
    <li><data value="398">Mini Ketchup</data></li>
    <li><data value="399">Jumbo Ketchup</data></li>
  </ul>
</body>
</html>
```

**Expected output**: A list of product names, each with a machine-readable value in the DOM.

**Why this output occurs**: The `<data>` element stores the product code in the `value` attribute while displaying the human-readable name. Scripts can read `element.value` to obtain the code.

---

#### Example 2: Currency with Machine Value

**Setup guide**: Save as `data-currency.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Currency Data</title>
</head>
<body>
  <p>
    Price: <data value="29.99">$29.99</data>
  </p>
</body>
</html>
```

**Expected output**: The price displays as “$29.99”, and the machine-readable value is “29.99” (without the currency symbol).

**Why this output occurs**: The `<data>` element separates the human-readable display from the machine-readable numeric value. Scripts can parse the numeric value directly.

---

### Real-World Cases

1. **E-commerce**: Product codes and prices.
2. **Catalogs**: Unique identifiers for items.
3. **Dashboards**: Machine-readable numeric values for charts.

---

### References

- WHATWG — The `data` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-data-element
- MDN Web Docs — `<data>`: The Data element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/data
- W3C — HTML 5.2: The `data` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-data-element

---

## 14. `<time>` — The Machine-Readable Time Element

### Definitions

**Core Definition**
The `<time>` element represents a specific period in time — a date, a time, or a combined date-time. Its `datetime` attribute provides a machine-readable equivalent.

**Technical Definition**
`<time>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The optional `datetime` attribute must be a valid date or time string in the formats defined by the HTML specification (e.g. `2026-11-06`, `10:30`, `2026-11-06T10:30`, `2h 30m`).

**Beginner-Friendly Explanation**
`<time>` marks dates and times in a way machines can understand. You can display “6 November 2026” but provide “2026-11-06” in the `datetime` attribute for search engines and scripts.

---

### Purposes

- **To mark up dates and times machine-readably**.
- **To support search-engine and browser parsing** of dates (e.g. for calendars, event listings).
- **To distinguish human-readable time formats** from machine-readable ones.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<time datetime="2026-11-06">6 November 2026</time>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `datetime` | Valid date or time string, if the content is not already machine-readable. |
| Content | Human-readable date/time text. |

#### Syntax Rules

1. **The `datetime` attribute is optional**; if the text content is already a valid date string, it may be omitted.
2. **Valid formats include** `YYYY-MM-DD`, `HH:MM`, `YYYY-MM-DDTHH:MM`, duration strings.
3. **Content model is phrasing content**.

#### Constraints and Limitations

- `datetime` must match the specification’s valid time formats.
- Without `datetime`, the element’s content itself must be a valid date string.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Human-Readable Date

**Setup guide**: Save as `time-date.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Time Example</title>
</head>
<body>
  <p>The event is on <time datetime="2026-11-06">6 November 2026</time>.</p>
</body>
</html>
```

**Expected output**: The text displays “6 November 2026”, and the machine-readable value is “2026-11-06”.

**Why this output occurs**: The `<time>` element provides a machine-readable date via the `datetime` attribute, while the content displays a human-readable version. Search engines and scripts can parse the `datetime` value.

---

#### Example 2: Time in a Log

**Setup guide**: Save as `time-log.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Log Entry</title>
</head>
<body>
  <p>Server started at <time>10:30</time>.</p>
</body>
</html>
```

**Expected output**: “10:30” is marked as a machine-readable time.

**Why this output occurs**: The content “10:30” is itself a valid time string, so the `datetime` attribute is not required. Browsers and parsers interpret it as a time.

---

### Real-World Cases

1. **Event listings**: Dates and times of events.
2. **Blog posts**: Publication and update timestamps.
3. **Server logs**: Machine-readable timestamps.

---

### References

- WHATWG — The `time` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-time-element
- MDN Web Docs — `<time>`: The (Date) Time element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/time
- W3C — HTML 5.2: The `time` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-time-element

---

## 15. `<code>` — The Code Element

### Definitions

**Core Definition**
The `<code>` element represents a fragment of computer code — a function call, variable name, HTML tag, or similar.

**Technical Definition**
`<code>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Browsers typically render it in a monospaced font, but the semantics are “code”, not “monospace styling”.

**Beginner-Friendly Explanation**
`<code>` marks a piece of code inside text — a variable name, function, or command. It is rendered in monospaced font by default and is often used alongside `<pre>` for code blocks.

---

### Purposes

- **To mark inline code snippets**.
- **To support syntax highlighting** by giving a target for stylesheets and scripts.
- **To improve accessibility** by signalling that the text is code.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Use <code>console.log()</code> to print output.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<code>` | The code wrapper. |
| Content | The code fragment. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **For multi-line code**, wrap `<code>` inside `<pre>`.
3. **Escape `<` and `>` characters** as `&lt;` and `&gt;` when showing HTML code.

#### Constraints and Limitations

- `<code>` is a semantic marker, not a syntax highlighter; styling must be applied with CSS or JavaScript.
- It does not preserve whitespace on its own; use `<pre>` for that.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline Code

**Setup guide**: Save as `code-inline.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Code Example</title>
</head>
<body>
  <p>The <code>querySelector</code> method selects elements.</p>
</body>
</html>
```

**Expected output**: “querySelector” appears in monospaced font.

**Why this output occurs**: `<code>` marks the term as code. The browser applies a monospaced font by default and screen readers may announce it as code.

---

#### Example 2: Code Block with Pre

**Setup guide**: Save as `code-block.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Code Block</title>
</head>
<body>
  <pre><code>for (let i = 0; i &lt; 3; i++) {
  console.log(i);
}</code></pre>
</body>
</html>
```

**Expected output**: A multi-line code block with preserved whitespace and monospaced font.

**Why this output occurs**: `<pre>` preserves whitespace and line breaks, and `<code>` marks the text as code. The `<` and `>` are escaped as HTML entities so they render as literal characters.

---

### Real-World Cases

1. **API documentation**: Function names and parameters.
2. **Tutorials**: Sample code blocks.
3. **Technical blogs**: Inline code references.

---

### References

- WHATWG — The `code` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-code-element
- MDN Web Docs — `<code>`: The Inline Code element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/code
- W3C — HTML 5.2: The `code` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-code-element

---

## 16. `<var>` — The Variable Element

### Definitions

**Core Definition**
The `<var>` element represents the name of a variable in a mathematical or programming context.

**Technical Definition**
`<var>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Browsers often render it in italics, but the semantics are “variable”, not “italic”.

**Beginner-Friendly Explanation**
`<var>` marks a variable name in code or math — like `x` in an equation or `count` in a program. It is usually rendered in italics.

---

### Purposes

- **To mark variable names** in technical content.
- **To distinguish variables** from surrounding prose.
- **To support mathematical and programming notation**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Let <var>x</var> equal 5.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<var>` | The variable wrapper. |
| Content | The variable name. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<var>` for mathematical or programming variables**.
3. **Do not use `<var>` for arbitrary italic text**.

#### Constraints and Limitations

- `<var>` is not a generic emphasis element; use `<em>` for emphasis.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mathematical Variable

**Setup guide**: Save as `var-math.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Variable Example</title>
</head>
<body>
  <p>If <var>x</var> = 3 and <var>y</var> = 4, then <var>x</var> + <var>y</var> = 7.</p>
</body>
</html>
```

**Expected output**: The variables `x` and `y` appear in italics.

**Why this output occurs**: `<var>` marks the letters as variables. The browser renders them in italics by default. The semantic meaning is mathematical notation.

---

#### Example 2: Programming Variable

**Setup guide**: Save as `var-code.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Program Variable</title>
</head>
<body>
  <p>Set <var>count</var> to 0 before the loop.</p>
</body>
</html>
```

**Expected output**: “count” appears in italics.

**Why this output occurs**: `<var>` marks the programming variable name. Combined with `<code>` in surrounding examples, it distinguishes variables from function names.

---

### Real-World Cases

1. **Mathematics**: Equations and formulas.
2. **Programming tutorials**: Variable and parameter names.
3. **Scientific documentation**: Variables in formulas.

---

### References

- WHATWG — The `var` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-var-element
- MDN Web Docs — `<var>`: The Variable element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/var
- W3C — HTML 5.2: The `var` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-var-element

---

## 17. `<samp>` — The Sample Output Element

### Definitions

**Core Definition**
The `<samp>` element represents sample or quoted output from another program or computing system.

**Technical Definition**
`<samp>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Browsers typically render it in a monospaced font.

**Beginner-Friendly Explanation**
`<samp>` marks the output of a program — what you would see in a terminal or a console. It is often combined with `<kbd>` to show “type this, see that”.

---

### Purposes

- **To mark program output** as sample output.
- **To distinguish system responses** from surrounding prose.
- **To support technical documentation** with clear output examples.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>When you run the program, you will see: <samp>Hello, World!</samp></p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<samp>` | The sample-output wrapper. |
| Content | The output text. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<samp>` for program output**, not for arbitrary text.
3. **Combine with `<kbd>`** to show input and output together.

#### Constraints and Limitations

- `<samp>` should not be used for user input; that is `<kbd>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Program Output

**Setup guide**: Save as `samp-output.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sample Output</title>
</head>
<body>
  <p>The program will display: <samp>Hello, World!</samp></p>
</body>
</html>
```

**Expected output**: “Hello, World!” appears in monospaced font, signalling that it is program output.

**Why this output occurs**: `<samp>` indicates sample output. The browser renders it in a monospace font by default.

---

#### Example 2: Combined kbd and samp

**Setup guide**: Save as `samp-kbd.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Kbd and Samp</title>
</head>
<body>
  <p>
    Press <kbd>Enter</kbd> and the terminal will show
    <samp>Operation complete.</samp>
  </p>
</body>
</html>
```

**Expected output**: “Enter” is styled as keyboard input and “Operation complete.” as program output.

**Why this output occurs**: `<kbd>` marks user input and `<samp>` marks the system’s response. The browser applies monospaced styling to both, and each carries distinct semantic meaning.

---

### Real-World Cases

1. **Terminal documentation**: Showing output.
2. **Tutorials**: Displaying program results.
3. **CLI references**: Illustrating command output.

---

### References

- WHATWG — The `samp` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-samp-element
- MDN Web Docs — `<samp>`: The Sample Output element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/samp
- W3C — HTML 5.2: The `samp` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-samp-element

---

## 18. `<kbd>` — The Keyboard Input Element

### Definitions

**Core Definition**
The `<kbd>` element represents user input — typically keyboard input, but also voice commands or any other input from the user.

**Technical Definition**
`<kbd>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Nesting `<kbd>` inside `<kbd>` can represent more complex input sequences.

**Beginner-Friendly Explanation**
`<kbd>` marks what the user should type or press — a keyboard shortcut like `Ctrl+C`, or a command like `ls -la`.

---

### Purposes

- **To mark user input** in instructions.
- **To signal keyboard shortcuts**.
- **To style keyboard commands** in documentation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Press <kbd>Ctrl</kbd> + <kbd>C</kbd> to copy.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<kbd>` | The keyboard input wrapper. |
| Content | The key or command. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use nested `<kbd>` for multi-key sequences** when necessary.
3. **Combine with `<samp>`** for input/output pairs.

#### Constraints and Limitations

- `<kbd>` is for user input; do not use it for program output (that is `<samp>`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Keyboard Shortcut

**Setup guide**: Save as `kbd-shortcut.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Keyboard Input</title>
</head>
<body>
  <p>To save the file, press <kbd>Ctrl</kbd> + <kbd>S</kbd>.</p>
</body>
</html>
```

**Expected output**: “Ctrl” and “S” each appear in monospaced font with a border, indicating keyboard keys.

**Why this output occurs**: `<kbd>` marks the key names. The browser applies a monospaced font, and additional CSS can add key-cap styling.

---

#### Example 2: Command-Line Input

**Setup guide**: Save as `kbd-cli.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CLI Input</title>
</head>
<body>
  <p>Run <kbd>npm install</kbd> in the project directory.</p>
</body>
</html>
```

**Expected output**: “npm install” appears in monospaced font.

**Why this output occurs**: `<kbd>` marks the user-entered command. It signals to the reader that they should type that exact text.

---

### Real-World Cases

1. **Software documentation**: Keyboard shortcuts and commands.
2. **Tutorials**: Command-line instructions.
3. **Help systems**: Key combinations for actions.

---

### References

- WHATWG — The `kbd` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-kbd-element
- MDN Web Docs — `<kbd>`: The Keyboard Input element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/kbd
- W3C — HTML 5.2: The `kbd` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-kbd-element

---

## 19. `<sub>` — The Subscript Element

### Definitions

**Core Definition**
The `<sub>` element specifies inline text that should be displayed as subscript for typographical reasons. Subscripts are typically rendered lower and smaller than surrounding text.

**Technical Definition**
`<sub>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The element should be used only for typographical conventions, not for styling.

**Beginner-Friendly Explanation**
`<sub>` lowers a piece of text below the baseline — like the “2” in the chemical formula H₂O.

---

### Purposes

- **To represent subscripts** in chemical formulas, mathematical expressions, and footnotes.
- **To convey typographical conventions** with semantic markup.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Water is H<sub>2</sub>O.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<sub>` | The subscript wrapper. |
| Content | The subscript text. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<sub>` only for typographical subscript meaning**, not for visual effects.

#### Constraints and Limitations

- `<sub>` should not be used to position logos or decorative elements; CSS `vertical-align` is more appropriate for styling.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Chemical Formula

**Setup guide**: Save as `sub-formula.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Subscript</title>
</head>
<body>
  <p>Water is written as H<sub>2</sub>O.</p>
</body>
</html>
```

**Expected output**: The “2” in H₂O appears smaller and lower than the “H” and “O”.

**Why this output occurs**: `<sub>` lowers the content below the baseline and reduces its size, which is standard chemical notation.

---

#### Example 2: Footnote Reference

**Setup guide**: Save as `sub-footnote.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Footnote</title>
</head>
<body>
  <p>This claim is disputed.<sub><a href="#note-1">1</a></sub></p>
  <p id="note-1"><small>1. See the source list at the end.</small></p>
</body>
</html>
```

**Expected output**: A small “1” appears as a subscript after the sentence, linking to the footnote.

**Why this output occurs**: The `<sub>` lowers the footnote reference below the baseline. The `<a>` inside makes it a clickable link.

---

### Real-World Cases

1. **Chemistry**: H₂O, CO₂.
2. **Mathematics**: Logarithm bases like log₂.
3. **Publishing**: Footnote markers.

---

### References

- WHATWG — The `sub` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-sub-and-sup-elements
- MDN Web Docs — `<sub>`: The Subscript element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/sub
- W3C — HTML 5.2: The `sub` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-sub-and-sup-elements

---

## 20. `<sup>` — The Superscript Element

### Definitions

**Core Definition**
The `<sup>` element specifies inline text that should be displayed as superscript for typographical reasons. Superscripts are rendered higher and smaller than surrounding text.

**Technical Definition**
`<sup>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. It should be used for typographical conventions such as exponents, ordinals, and footnote markers.

**Beginner-Friendly Explanation**
`<sup>` raises a piece of text above the baseline — like the “2” in 10² or the “th” in 4th.

---

### Purposes

- **To represent superscripts** in exponents, ordinals, and footnote markers.
- **To convey typographical conventions** with semantic markup.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>10<sup>2</sup> equals 100.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<sup>` | The superscript wrapper. |
| Content | The superscript text. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<sup>` for typographical superscript**, not for decorative positioning.

#### Constraints and Limitations

- Superscripts can shrink text below accessible minimums; use CSS to control size carefully.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Exponent

**Setup guide**: Save as `sup-exponent.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Superscript</title>
</head>
<body>
  <p>The area is 5<sup>2</sup> = 25 square meters.</p>
</body>
</html>
```

**Expected output**: “2” appears smaller and higher, so it reads as “5 squared”.

**Why this output occurs**: `<sup>` raises the content above the baseline. The exponent is expressed semantically, not by font styling alone.

---

#### Example 2: Ordinal Suffix

**Setup guide**: Save as `sup-ordinal.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Ordinal</title>
</head>
<body>
  <p>She finished 4<sup>th</sup> in the race.</p>
</body>
</html>
```

**Expected output**: “th” appears smaller and higher after “4”, reading as “4th”.

**Why this output occurs**: The `<sup>` element raises “th” above the baseline. This is common in ordinal notation.

---

### Real-World Cases

1. **Mathematics**: Exponents.
2. **Publishing**: Ordinal suffixes and footnote markers.
3. **Science**: Isotope notation (e.g. ¹⁴C).

---

### References

- WHATWG — The `sup` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-sub-and-sup-elements
- MDN Web Docs — `<sup>`: The Superscript element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/sup
- W3C — HTML 5.2: The `sup` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-sub-and-sup-elements

---

## 21. `<i>` — The Idiomatic Text Element

### Definitions

**Core Definition**
The `<i>` element represents text that is in some way different from normal prose — an alternate voice or mood, a technical term, a phrase from another language, a thought, a ship name, or a similar idiomatic use.

**Technical Definition**
`<i>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The element has no fixed semantics beyond “alternate voice or mood” — it should not be used for emphasis (that is `<em>`) or for strong importance (`<strong>`).

**Beginner-Friendly Explanation**
`<i>` marks text that is different in tone, language, or style — foreign words, technical terms, thoughts, or ship names. Browsers render it in italics by default, but the meaning is “different from normal prose”.

---

### Purposes

- **To mark alternate voice or mood** in prose.
- **To distinguish foreign-language terms** from surrounding text.
- **To mark technical terms, ship names, and taxonomic designations**.
- **To provide a semantic alternative to `<em>`** when italics are needed without emphasis.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>The <i>RMS Titanic</i> was a British passenger liner.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<i>` | The idiomatic text wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Do not use `<i>` for emphasis**; use `<em>`.
3. **Do not use `<i>` for strong importance**; use `<strong>`.

#### Constraints and Limitations

- `<i>` has been described by the HTML standard as an “alternate voice or mood” element; using it for other purposes (like arbitrary italics) is less meaningful.
- Browsers default to italics, but CSS can override the presentation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Foreign-Language Term

**Setup guide**: Save as `i-foreign.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Idiomatic Text</title>
</head>
<body>
  <p>The French phrase <i lang="fr">déjà vu</i> means “already seen”.</p>
</body>
</html>
```

**Expected output**: “déjà vu” appears in italics and is marked as French.

**Why this output occurs**: `<i>` marks a foreign-language term as a distinct phrase. The `lang="fr"` attribute signals correct pronunciation to screen readers. Browsers italicise `<i>` by default.

---

#### Example 2: Ship Name

**Setup guide**: Save as `i-ship.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Ship Name</title>
</head>
<body>
  <p>The <i>Queen Mary 2</i> docked this morning.</p>
</body>
</html>
```

**Expected output**: “Queen Mary 2” appears in italics.

**Why this output occurs**: The `<i>` marks the ship name as an idiomatic use — a proper name in italics as is conventional for vessel names in some editorial styles.

---

### Real-World Cases

1. **Scientific writing**: Taxonomic names, such as *Homo sapiens*.
2. **Publishing**: Ship names, book titles (though `<cite>` is often preferred).
3. **Prose**: Foreign words and internal thoughts.

---

### References

- WHATWG — The `i` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-i-element
- MDN Web Docs — `<i>`: The Idiomatic Text element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/i
- W3C — HTML 5.2: The `i` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-i-element

---

## 22. `<b>` — The Bring Attention Element

### Definitions

**Core Definition**
The `<b>` element is used to draw attention to text without conveying extra importance — a stylistic alternative used for keywords, product names, or lede sentences.

**Technical Definition**
`<b>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The element conveys a general “bring attention to” meaning without any stronger semantic connotation. Browsers render it in bold, but bold is presentation.

**Beginner-Friendly Explanation**
`<b>` bolds text for attention without implying that it is more important. It is used for things like product names, key terms, or the first sentence of an article in a magazine style.

---

### Purposes

- **To draw attention** to text without implying importance.
- **To mark keywords** in summaries or abstracts.
- **To style product names, article ledes, or labels** stylistically.
- **To provide a semantic alternative to `<strong>`** when bold is needed without semantic importance.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>The <b>new model</b> ships next week.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<b>` | The attention wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Do not use `<b>` for emphasis or importance**; use `<em>` or `<strong>`.
3. **Use `<b>` only when bold is appropriate for stylistic reasons**.

#### Constraints and Limitations

- `<b>` is explicitly described by the specification as a stylistic element with no strong semantic meaning.
- Screen readers may not announce `<b>` distinctly.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Product Name

**Setup guide**: Save as `b-product.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Bring Attention</title>
</head>
<body>
  <p>The <b>UltraLite</b> is our newest model.</p>
</body>
</html>
```

**Expected output**: “UltraLite” appears in bold.

**Why this output occurs**: `<b>` brings attention to the product name without implying that the product is more important than anything else in the sentence. It is a stylistic choice.

---

#### Example 2: Article Lede

**Setup guide**: Save as `b-lede.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Article Lede</title>
</head>
<body>
  <p>
    <b>Researchers have discovered a new species of frog in the Amazon.</b>
    The frog, which was found in a remote region, is notable for its bright
    blue colouring.
  </p>
</body>
</html>
```

**Expected output**: The first sentence is bold, drawing attention to the headline information.

**Why this output occurs**: Magazine-style ledes often use bold for the first sentence without semantic importance. `<b>` is appropriate for this stylistic use.

---

### Real-World Cases

1. **Product marketing copy**: Bold product names.
2. **Summaries**: Key terms in abstracts.
3. **Journalistic writing**: Lede sentences in magazine-style articles.

---

### References

- WHATWG — The `b` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-b-element
- MDN Web Docs — `<b>`: The Bring Attention To element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/b
- W3C — HTML 5.2: The `b` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-b-element

---

## 23. `<u>` — The Unarticulated Annotation Element

### Definitions

**Core Definition**
The `<u>` element represents a span of text with an unarticulated, though explicitly rendered, non-textual annotation — such as a misspelled word or a proper name in Chinese.

**Technical Definition**
`<u>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Browsers render it with a text underline by default, but the semantics are “non-textual annotation”.

**Beginner-Friendly Explanation**
`<u>` marks text with a non-textual annotation — usually indicated by an underline. It is commonly used for misspelled words, but the HTML standard suggests using it only when the annotation is not otherwise conveyed by other markup.

---

### Purposes

- **To indicate a non-textual annotation** such as a spelling error or a proper name marking.
- **To distinguish text with an underline** that carries semantic meaning.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>This word is <u class="spelling">mispeled</u>.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<u>` | The annotation wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Do not use `<u>` for styling underlines**; use CSS for decorative underlines.
3. **Do not use `<u>` for emphasis**; use `<em>`.

#### Constraints and Limitations

- `<u>` should be used only when the annotation is meaningful and cannot be conveyed otherwise.
- Modern usage of `<u>` is controversial because underlines are commonly used for links; use sparingly.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Misspelled Word

**Setup guide**: Save as `u-spelling.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Unarticulated Annotation</title>
</head>
<body>
  <p>The word <u>mispeled</u> is underlined as a spelling error.</p>
</body>
</html>
```

**Expected output**: “mispeled” appears underlined.

**Why this output occurs**: `<u>` indicates a non-textual annotation — here, a spelling error. The browser renders the underline by default.

---

#### Example 2: Chinese Proper Name Mark

**Setup guide**: Save as `u-chinese.html`.

```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>Proper Name Mark</title>
</head>
<body>
  <p><u>孔子</u>是中国古代著名的思想家。</p>
</body>
</html>
```

**Expected output**: “孔子” appears underlined, marking it as a proper name.

**Why this output occurs**: In some Chinese editorial conventions, proper names are underlined. `<u>` provides the non-textual annotation semantically.

---

### Real-World Cases

1. **Spell check**: Underlining misspelled words.
2. **Chinese editorial style**: Marking proper names.
3. **Rarely used in mainstream web content**.

---

### References

- WHATWG — The `u` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-u-element
- MDN Web Docs — `<u>`: The Unarticulated Annotation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/u
- W3C — HTML 5.2: The `u` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-u-element

---

## 24. `<mark>` — The Mark Element

### Definitions

**Core Definition**
The `<mark>` element represents text that is marked or highlighted for reference or notation purposes, due to its relevance in the surrounding context.

**Technical Definition**
`<mark>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. Browsers render it with a yellow background by default, and the semantic meaning is “marked for relevance”.

**Beginner-Friendly Explanation**
`<mark>` highlights text that is relevant to the current context — like search results highlighted in a page, or passages highlighted by the user in an e-reader.

---

### Purposes

- **To highlight search-term matches** within a results page.
- **To indicate user-highlighted text** in reading apps.
- **To mark relevant text** in quotations or passages.
- **To draw attention to specific content** due to contextual relevance.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Search results for <mark>HTML</mark>:</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<mark>` | The marked wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<mark>` when the relevance is contextual**, not when importance is intrinsic.
3. **Do not confuse `<mark>` with `<strong>` or `<em>`** — the meanings are different.

#### Constraints and Limitations

- The default yellow background can fail contrast requirements; adjust with CSS if necessary.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Search Highlight

**Setup guide**: Save as `mark-search.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mark Example</title>
</head>
<body>
  <p>Search results for <mark>HTML</mark>:</p>
  <ul>
    <li>Introduction to <mark>HTML</mark></li>
    <li>Advanced <mark>HTML</mark> techniques</li>
  </ul>
</body>
</html>
```

**Expected output**: “HTML” is highlighted with a yellow background in each result.

**Why this output occurs**: `<mark>` marks the search term as relevant to the query. The default yellow background draws the reader’s eye.

---

#### Example 2: Highlighting a Quotation

**Setup guide**: Save as `mark-quote.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mark in Quote</title>
</head>
<body>
  <blockquote>
    <p>
      The most important thing about <mark>accessibility</mark> is that it
      benefits everyone.
    </p>
  </blockquote>
</body>
</html>
```

**Expected output**: The word “accessibility” is highlighted in the quotation.

**Why this output occurs**: `<mark>` marks text for reference within a quotation. It draws attention to a specific term without implying that it is more important than other text in the original source.

---

### Real-World Cases

1. **Search results**: Highlighting matched terms.
2. **Reading apps**: User-highlighted passages.
3. **Documentation**: Emphasising relevant passages in quoted material.

---

### References

- WHATWG — The `mark` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-mark-element
- MDN Web Docs — `<mark>`: The Mark Text element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/mark
- W3C — HTML 5.2: The `mark` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-mark-element

---

## 25. `<bdi>` — The Bidirectional Isolation Element

### Definitions

**Core Definition**
The `<bdi>` element isolates its content from the surrounding text for bidirectional text formatting — that is, it tells the browser to treat its content as a self-contained unit for determining text direction.

**Technical Definition**
`<bdi>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. It is useful for embedding user-provided content — such as usernames — that may contain text in a different direction from the surrounding text.

**Beginner-Friendly Explanation**
`<bdi>` tells the browser to “keep this piece of text separate” when deciding whether it reads left-to-right or right-to-left. It prevents user-generated content in one direction from messing up surrounding content in another direction.

---

### Purposes

- **To isolate user-generated text** from surrounding directionality.
- **To prevent bidirectional text from breaking layout**.
- **To mark usernames or other dynamic content** safely.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<bdi>User</bdi>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<bdi>` | The isolation wrapper. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **The element sets `unicode-bidi: isolate`** and derives its direction from its content.
3. **Use it for usernames, comments, and other dynamic text**.

#### Constraints and Limitations

- Not needed when the directionality is already known or set explicitly.
- Should not be used for large blocks of content; it is for inline text.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Isolating a Username

**Setup guide**: Save as `bdi-username.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDI Example</title>
</head>
<body>
  <ul>
    <li>User <bdi>John</bdi>: 5 posts</li>
    <li>User <bdi>إيما</bdi>: 12 posts</li>
    <li>User <bdi>田中</bdi>: 3 posts</li>
  </ul>
</body>
</html>
```

**Expected output**: Each username is displayed in its own direction, and the “: N posts” remains in the list’s direction.

**Why this output occurs**: `<bdi>` isolates each username so the browser can determine its directionality independently. The Arabic and Japanese names do not flip the surrounding text.

---

### Real-World Cases

1. **Social media feeds**: Usernames in different languages.
2. **Comment systems**: User-provided content mixed with UI text.
3. **Chat applications**: Messages in various languages.

---

### References

- WHATWG — The `bdi` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-bdi-element
- MDN Web Docs — `<bdi>`: The Bidirectional Isolate element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/bdi
- W3C — HTML 5.2: The `bdi` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-bdi-element

---

## 26. `<bdo>` — The Bidirectional Override Element

### Definitions

**Core Definition**
The `<bdo>` element overrides the current directionality of text — it forces the text to render in the specified direction using the `dir` attribute.

**Technical Definition**
`<bdo>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. The `dir` attribute is **required** and must be either `ltr` or `rtl`.

**Beginner-Friendly Explanation**
`<bdo>` overrides the direction of text. If the surrounding text is left-to-right, `<bdo dir="rtl">` makes its content render right-to-left.

---

### Purposes

- **To override the default direction** of a specific phrase.
- **To render text in an explicitly opposite direction**.
- **To support bidirectional text rendering** in demos or testing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<bdo dir="rtl">This text is reversed</bdo>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `dir` | Required; `ltr` or `rtl`. |
| Content | Phrasing content. |

#### Syntax Rules

1. **`dir` is required** and must be `ltr` or `rtl`.
2. **Content model is phrasing content**.
3. **Use only when the override is necessary**; `<bdi>` is usually better for isolation.

#### Constraints and Limitations

- Overuse can harm readability; use `<bdi>` for isolation and `<bdo>` only when overriding is needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reversing Text Direction

**Setup guide**: Save as `bdo-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDO Example</title>
</head>
<body>
  <p>This text is normal. <bdo dir="rtl">This text is reversed.</bdo></p>
</body>
</html>
```

**Expected output**: The first sentence reads normally; the second renders with right-to-left direction (the period moves to the left).

**Why this output occurs**: `<bdo dir="rtl">` forces the content to render in right-to-left order, reversing the visual order of the text.

---

#### Example 2: Hebrew in English Context

**Setup guide**: Save as `bdo-hebrew.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDO Hebrew</title>
</head>
<body>
  <p>The Hebrew word for peace is <bdo dir="rtl">שלום</bdo>.</p>
</body>
</html>
```

**Expected output**: The Hebrew word appears correctly in RTL order inside the English sentence.

**Why this output occurs**: `<bdo>` forces the Hebrew word to render right-to-left regardless of the surrounding English direction.

---

### Real-World Cases

1. **Language learning tools**: Demonstrating text in reverse direction.
2. **Bidirectional text demos**: Testing how text renders.
3. **Rarely used in everyday content** — `<bdi>` is usually the better choice.

---

### References

- WHATWG — The `bdo` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-bdo-element
- MDN Web Docs — `<bdo>`: The Bidirectional Text Override element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/bdo
- W3C — HTML 5.2: The `bdo` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-bdo-element

---

## 27. `<span>` — The Generic Inline Container

### Definitions

**Core Definition**
The `<span>` element is a generic inline container for phrasing content that has no inherent semantic meaning. It is used to style or script a specific portion of text when no other semantic element fits.

**Technical Definition**
`<span>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is phrasing content. It is the phrasing-level counterpart of `<div>`.

**Beginner-Friendly Explanation**
`<span>` is a plain inline box. Use it when you need to style or target a piece of text but no semantic element fits.

---

### Purposes

- **To provide a styling hook** for phrasing content.
- **To provide a scripting hook** for a portion of text.
- **To wrap text when no semantic element is appropriate**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<span class="highlight">text</span>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<span>` | The generic wrapper. |
| `class` / `id` / `lang` | Styling, scripting, and language hooks. |
| Content | Phrasing content. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **`<span>` has no semantic meaning**; use only when no specific element fits.
3. **The `lang` attribute can be used** for language marking.

#### Constraints and Limitations

- Overuse of `<span>` instead of semantic elements (e.g. `<em>`, `<strong>`, `<mark>`, `<cite>`) reduces the semantic richness of the document.
- Empty `<span>` elements are not palpable and should be avoided.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Styling a Specific Phrase

**Setup guide**: Save as `span-style.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Span Example</title>
  <style>
    .highlight { background-color: yellow; }
  </style>
</head>
<body>
  <p>The <span class="highlight">important phrase</span> appears here.</p>
</body>
</html>
```

**Expected output**: “important phrase” has a yellow background.

**Why this output occurs**: The `<span>` provides a styling hook via the `class` attribute. The `<p>` content flow is preserved, and the span does not add block-level formatting.

---

#### Example 2: Language Marking with Span

**Setup guide**: Save as `span-lang.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Span Lang</title>
</head>
<body>
  <p>The word <span lang="fr">bonjour</span> means “hello” in French.</p>
</body>
</html>
```

**Expected output**: The word “bonjour” renders normally, but a screen reader using language switching will pronounce it in French.

**Why this output occurs**: The `<span>` has no semantic meaning, but the `lang="fr"` attribute overrides the language for its content. This is useful when no semantic element is available.

---

### Real-World Cases

1. **CSS styling**: Wrapping phrases for visual effects.
2. **JavaScript targeting**: Selecting specific text spans.
3. **Language marking**: Overriding the language for a phrase when no element fits.

---

### References

- WHATWG — The `span` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-span-element
- MDN Web Docs — `<span>`: The Content Span element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/span
- W3C — HTML 5.2: The `span` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-span-element

---

## 28. `<br>` — The Line Break Element

### Definitions

**Core Definition**
The `<br>` element produces a line break in text (carriage return). It is a **void element** and must not have an end tag.

**Technical Definition**
`<br>` belongs to the **flow content** and **phrasing content** categories. It has no content model. The element represents a line break, but using it excessively for layout spacing is discouraged.

**Beginner-Friendly Explanation**
`<br>` starts a new line. It is useful in poetry or addresses where line breaks are part of the content, but not for spacing paragraphs (that is `<p>` with CSS).

---

### Purposes

- **To insert a line break** within a paragraph or phrase.
- **To format poetry** where line breaks are part of the content.
- **To preserve address formatting** where lines are semantically distinct.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<br>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<br>` | A void element; no end tag, no content. |
| Attributes | Global attributes; `clear` was removed. |

#### Syntax Rules

1. **`<br>` is a void element**.
2. **Use `<br>` for content-level line breaks**, not for spacing.
3. **Do not use multiple `<br>` elements for vertical spacing**; use CSS margins.

#### Constraints and Limitations

- `<br>` has no accessible alternative for spacing; screen readers may not announce it.
- In `<pre>` content, line breaks are preserved by the element itself, not by `<br>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Address Line Breaks

**Setup guide**: Save as `br-address.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Line Break</title>
</head>
<body>
  <p>
    John Smith<br>
    123 Main Street<br>
    Anytown, USA
  </p>
</body>
</html>
```

**Expected output**: The three lines display on separate rows.

**Why this output occurs**: Each `<br>` inserts a line break, so the address renders as three distinct lines within a single paragraph.

---

#### Example 2: Poetry

**Setup guide**: Save as `br-poem.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Poem</title>
</head>
<body>
  <p>
    Roses are red,<br>
    Violets are blue,<br>
    Sugar is sweet,<br>
    And so are you.
  </p>
</body>
</html>
```

**Expected output**: Each line of the poem appears on its own line.

**Why this output occurs**: `<br>` preserves the poem’s line structure. The line breaks are semantic content, not spacing.

---

### Real-World Cases

1. **Poetry**: Preserving line breaks in verse.
2. **Addresses**: Formatting multi-line addresses.
3. **Song lyrics**: Line-by-line display.

---

### References

- WHATWG — The `br` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-br-element
- MDN Web Docs — `<br>`: The Line Break element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/br
- W3C — HTML 5.2: The `br` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-br-element

---

## 29. `<wbr>` — The Word Break Opportunity Element

### Definitions

**Core Definition**
The `<wbr>` element represents a word break opportunity — a position within text where the browser may optionally break a line if needed.

**Technical Definition**
`<wbr>` is a **void element** belonging to the **flow content** and **phrasing content** categories. It has no content model. Unlike `<br>`, which forces a line break, `<wbr>` only suggests where a break is allowed.

**Beginner-Friendly Explanation**
`<wbr>` tells the browser “you may break the line here if you need to”. It is useful for long URLs or words that would otherwise overflow the container.

---

### Purposes

- **To allow optional line breaks** in long unbroken strings.
- **To prevent overflow** of long URLs and identifiers.
- **To suggest break points** without forcing them.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>https://example.com/<wbr>very<wbr>long<wbr>path</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<wbr>` | A void element; no end tag. |
| Attributes | Global attributes. |

#### Syntax Rules

1. **`<wbr>` is a void element**.
2. **It allows, but does not force, a line break**.
3. **Use it for long URLs, code identifiers, and unbreakable strings**.

#### Constraints and Limitations

- Browsers may or may not break at `<wbr>` opportunities depending on layout.
- Screen readers generally ignore `<wbr>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Breaking a Long URL

**Setup guide**: Save as `wbr-url.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Word Break</title>
  <style>
    .narrow { max-width: 200px; border: 1px solid #ccc; }
  </style>
</head>
<body>
  <p class="narrow">
    https://example.com/<wbr>articles/<wbr>html/<wbr>elements/<wbr>text-level
  </p>
</body>
</html>
```

**Expected output**: The URL wraps at the `<wbr>` points rather than overflowing the narrow container.

**Why this output occurs**: `<wbr>` signals allowed line-break points. Without it, the browser would be forced to overflow or break at an arbitrary position.

---

#### Example 2: Breaking a Long Identifier

**Setup guide**: Save as `wbr-identifier.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WBR Identifier</title>
</head>
<body>
  <p>
    The function name is
    <code>super<wbr>long<wbr>function<wbr>name<wbr>here</code>.
  </p>
</body>
</html>
```

**Expected output**: The identifier can wrap at the allowed points if the layout is narrow.

**Why this output occurs**: `<wbr>` inside `<code>` allows breaks at semantic points (camel-case boundaries) rather than at arbitrary positions, improving readability.

---

### Real-World Cases

1. **Long URLs**: Inline link text that can break.
2. **Code documentation**: Long function names.
3. **Technical content**: Long identifiers and hashes.

---

### References

- WHATWG — The `wbr` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-wbr-element
- MDN Web Docs — `<wbr>`: The Line Break Opportunity element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/wbr
- W3C — HTML 5.2: The `wbr` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-wbr-element

---

## 30. Text-Level Semantic Best Practices

### Definitions

**Core Definition**
Text-level semantic best practices are authoring guidelines for choosing, nesting, and presenting text-level elements so that their meaning is accurately conveyed to browsers, assistive technologies, and machine interpreters.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG techniques, and accessibility guidance. They encompass semantic element choice, avoidance of presentation-only elements when semantic ones exist, proper use of `lang` and `dir`, and the principle of progressive enhancement.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for marking up text. They ensure that the element you choose accurately describes what the text actually *is*, not just how it looks.

---

### Purposes

- **To maximise accessibility** through correct semantic meaning.
- **To preserve machine readability** for search engines and translation tools.
- **To avoid confusion** caused by misusing elements for styling.
- **To improve maintainability** by replacing visual hacks with semantic markup.

---

### Syntax Rules and Structure

Best practices are applied across elements. The following table summarises the core rules.

| Practice | Rule |
|---|---|
| **Match element to meaning** | Use `<em>` for emphasis, `<strong>` for importance, `<cite>` for work titles, `<q>` for inline quotations, `<blockquote>` for long quotations. |
| **Do not use `<b>`/`<i>` as substitutes for semantic elements** | Use `<strong>`, `<em>`, `<cite>`, `<mark>`, or `<dfn>` when the meaning matches. |
| **Mark language changes** | Use `lang` on elements that contain text in another language. |
| **Handle bidirectional text** | Use `<bdi>` for user-generated content and `<bdo>` only when override is needed. |
| **Use `<time>` and `<data>` for machine-readable values** | Do not embed dates or codes in plain text without markup. |
| **Do not use `<br>` for spacing** | Use CSS margins instead. |
| **Use `<wbr>` for breakable strings** | Prevent overflow of long URLs and identifiers. |
| **Combine `<pre>` and `<code>` for code blocks** | Use inline `<code>` for short snippets. |
| **Escape `<` and `>` in code samples** | Use `&lt;` and `&gt;` to avoid parser confusion. |
| **Avoid overusing `<span>`** | Use semantic elements when they fit; `<span>` is the fallback. |

#### Syntax Rules

1. **Semantic correctness**: Choose the element whose definition matches the content’s meaning.
2. **Accessibility first**: Use elements that assistive technologies can interpret correctly.
3. **Valid nesting**: Respect content models — no interactive content inside `<a>`, no `<dfn>` inside `<dfn>`, and so on.
4. **Language and direction**: Mark foreign-language text with `lang`, and use `<bdi>`/`<bdo>` for bidirectional isolation and override.
5. **Machine readability**: Use `<time>`, `<data>`, and `<abbr>` to provide machine-readable equivalents.

#### Constraints and Limitations

- Best practices can conflict with editorial style guides; alignment between technical and editorial teams is valuable.
- Some elements (`<b>`, `<i>`, `<u>`) are explicitly presentational but still valid for stylistic purposes when semantics are not implied.
- Support for advanced features (`<ruby>`, `<bdi>`) is now broad but may be limited in older environments.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Well-Structured Technical Passage

**Setup guide**: Save as `best-practice-text.html` and inspect the semantics in the accessibility panel.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Text Semantics Best Practices</title>
</head>
<body>
  <article>
    <h1>Understanding <abbr title="HyperText Markup Language">HTML</abbr> Text Semantics</h1>

    <p>
      The <dfn id="def-em">emphasis</dfn> element, <code>&lt;em&gt;</code>,
      marks text that changes the meaning of a sentence. By contrast,
      <strong>importance</strong> is conveyed by the <code>&lt;strong&gt;</code> element.
    </p>

    <p>
      For example, <cite>The Elements of Style</cite> notes that
      <q>the adjective has not been built that can pull a weak or inaccurate noun out of a tight place</q>.
    </p>

    <p>
      Published <time datetime="2026-11-12">12 November 2026</time>.
    </p>

    <pre><code>const answer = 42;
console.log(answer);</code></pre>
  </article>
</body>
</html>
```

**Expected output**: A technical article with an `<abbr>` expansion tooltip, a `<dfn>` defining “emphasis”, a `<strong>` importance marker, a `<cite>` for the book title, a `<q>` for the inline quotation, a `<time>` for the date, and a `<pre><code>` block for the code sample.

**Why this output occurs**: Every text-level element is used according to its semantic meaning. The `<abbr>` provides an expansion, the `<dfn>` marks a defining instance, the `<strong>` conveys importance, the `<cite>` names a work, the `<q>` marks an inline quotation, the `<time>` provides a machine-readable date, and the `<pre><code>` combination preserves and marks code.

---

#### Example 2: Avoiding Common Text Semantic Anti-Patterns

**Setup guide**: Save as `text-anti-patterns.html` and inspect the accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Text Anti-Patterns</title>
</head>
<body>
  <!-- ANTI-PATTERN 1: <i> used for emphasis instead of <em> -->
  <p>This is <i>really important</i>.</p>

  <!-- ANTI-PATTERN 2: <b> used for strong importance instead of <strong> -->
  <p><b>Warning:</b> Do not proceed.</p>

  <!-- ANTI-PATTERN 3: <br> for spacing -->
  <p>Paragraph one.<br><br><br>Paragraph two.</p>

  <!-- ANTI-PATTERN 4: <span> where semantic element applies -->
  <p>The book <span class="title">The Great Gatsby</span> is a classic.</p>

  <!-- ANTI-PATTERN 5: <a> containing a <button> -->
  <a href="/">Home <button type="button">Click</button></a>
</body>
</html>
```

**Expected output**: The page renders, but the semantic meaning is broken: `<i>` and `<b>` do not convey emphasis or importance the way `<em>` and `<strong>` do, multiple `<br>` elements add visual space with no semantic purpose, `<span>` fails to mark the book title as a citation, and the `<a>` containing a `<button>` is invalid interactive nesting.

**Why this output occurs**: These are common anti-patterns that sacrifice semantic clarity for presentation. The best-practice version uses `<em>`, `<strong>`, CSS margins, `<cite>`, and properly separated link and button elements.

---

### Real-World Cases

1. **Content management systems**: Templates that render content into semantic markup ensure that every page uses text-level semantics consistently.
2. **Accessibility audits**: Correct use of `<em>`, `<strong>`, `<abbr>`, `<mark>`, and `lang` are common checkpoints.
3. **Documentation generators**: Markdown-to-HTML converters map emphasis (`*text*`), strong (`**text**`), code (`` `code` ``), and abbreviations to the correct HTML elements.

---

### References

- WHATWG — HTML Living Standard: Text-level semantics — https://html.spec.whatwg.org/multipage/text-level-semantics.html
- MDN Web Docs — Inline text semantics — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements#inline_text_semantics
- W3C — WCAG 2.2: Info and Relationships — https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships
- W3C — HTML 5.2: Text-level semantics — https://www.w3.org/TR/html52/textlevel-semantics.html
- web.dev — Text content — https://web.dev/learn/html/text

---

## Summary Table of Text-Level Semantic Elements

| Element | Purpose | Category | Notable Attributes |
|---|---|---|---|
| `<a>` | Hyperlink | Flow, phrasing, interactive, palpable | `href`, `target`, `rel`, `download` |
| `<em>` | Stress emphasis | Flow, phrasing, palpable | global |
| `<strong>` | Strong importance | Flow, phrasing, palpable | global |
| `<small>` | Side comment / fine print | Flow, phrasing, palpable | global |
| `<s>` | No longer accurate | Flow, phrasing, palpable | global |
| `<cite>` | Title of a work | Flow, phrasing, palpable | global |
| `<q>` | Inline quotation | Flow, phrasing, palpable | `cite` |
| `<dfn>` | Defining instance | Flow, phrasing, palpable | global |
| `<abbr>` | Abbreviation | Flow, phrasing, palpable | `title` |
| `<ruby>` | Ruby container | Flow, phrasing, palpable | global |
| `<rt>` | Ruby text | Phrasing | global |
| `<rp>` | Ruby parenthesis | Phrasing | global |
| `<data>` | Machine-readable data | Flow, phrasing, palpable | `value` (required) |
| `<time>` | Machine-readable date/time | Flow, phrasing, palpable | `datetime` |
| `<code>` | Code fragment | Flow, phrasing, palpable | global |
| `<var>` | Variable | Flow, phrasing, palpable | global |
| `<samp>` | Sample output | Flow, phrasing, palpable | global |
| `<kbd>` | Keyboard input | Flow, phrasing, palpable | global |
| `<sub>` | Subscript | Flow, phrasing, palpable | global |
| `<sup>` | Superscript | Flow, phrasing, palpable | global |
| `<i>` | Idiomatic text | Flow, phrasing, palpable | `lang` |
| `<b>` | Bring attention to | Flow, phrasing, palpable | global |
| `<u>` | Unarticulated annotation | Flow, phrasing, palpable | global |
| `<mark>` | Marked/highlighted text | Flow, phrasing, palpable | global |
| `<bdi>` | Bidirectional isolation | Flow, phrasing, palpable | global |
| `<bdo>` | Bidirectional override | Flow, phrasing, palpable | `dir` (required) |
| `<span>` | Generic inline container | Flow, phrasing, palpable | `class`, `id`, `lang` |
| `<br>` | Line break | Flow, phrasing | global |
| `<wbr>` | Word break opportunity | Flow, phrasing | global |

---

## Final Notes on Deprecated, Obsolete, and Implementation-Dependent Features

- **`acronym` element**: Removed in HTML5; use `<abbr>` instead.
- **`big` element**: Obsolete; use CSS `font-size`.
- **`strike` element**: Obsolete; use `<s>` or `<del>` (or CSS `text-decoration`).
- **`tt` element**: Obsolete; use `<code>`, `<kbd>`, `<samp>`, or `<var>` depending on context, or CSS `font-family: monospace`.
- **`blink` element**: Obsolete and non-standard; never use it.
- **`marquee` element**: Obsolete but widely supported; avoid it in new code.
- **`<b>` and `<i>` semantics**: In HTML4, they were strictly presentational; in HTML5, they carry the weak semantics “bring attention” and “alternate voice”. They are not interchangeable with `<strong>` and `<em>`.
- **`<u>` semantics**: Historically presentational; in HTML5 it means “unarticulated annotation”.
- **`<rb>` and `<rtc>` elements**: Part of the HTML Ruby Markup Extensions but not part of the HTML5 specification; use `<ruby>`, `<rt>`, and `<rp>` for cross-browser safety.

---

This cheat sheet has covered the text-level semantic elements — `<a>`, `<em>`, `<strong>`, `<small>`, `<s>`, `<cite>`, `<q>`, `<dfn>`, `<abbr>`, `<ruby>`, `<rt>`, `<rp>`, `<data>`, `<time>`, `<code>`, `<var>`, `<samp>`, `<kbd>`, `<sub>`, `<sup>`, `<i>`, `<b>`, `<u>`, `<mark>`, `<bdi>`, `<bdo>`, `<span>`, `<br>`, and `<wbr>` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate text-level markup against the WHATWG HTML Living Standard and WCAG accessibility guidelines.