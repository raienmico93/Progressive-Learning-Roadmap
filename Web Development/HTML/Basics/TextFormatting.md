# HTML Text Semantics: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML text semantics is the set of inline elements that convey meaning about the nature, purpose, or state of text content, rather than merely describing its visual appearance.

**Technical Definition**

Text-level semantics encompasses the phrasing-content elements defined in the WHATWG HTML Living Standard that annotate text with semantic information. These elements include `<strong>`, `<em>`, `<b>`, `<i>`, `<u>`, `<mark>`, `<small>`, `<del>`, `<ins>`, `<s>`, `<sub>`, and `<sup>`. Each element carries a specific semantic meaning that user agents and assistive technologies can interpret independently of any default visual presentation. The full list of text-level semantic elements also includes `<a>`, `<cite>`, `<q>`, `<dfn>`, `<abbr>`, `<ruby>`, `<data>`, `<time>`, `<code>`, `<var>`, `<samp>`, and `<kbd>`.

**Beginner-Friendly Explanation**

When you write text, you sometimes want to show that a word is important, emphasised, or has been deleted. HTML gives you special tags for each of these meanings. The key idea is that you pick a tag based on what the text *means*, not how you want it to *look*. For example, `<strong>` means “this is important,” while `<b>` just means “make this bold for attention.” Browsers often display them the same way (bold), but screen readers and search engines understand the difference.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Semantic meaning** | Each element carries specific meaning beyond its default rendering |
| **Phrasing content** | All these elements are inline-level and may appear inside paragraphs |
| **Global attributes only** | These elements accept only global attributes (plus `cite`/`datetime` on `<del>`/`<ins>`) |
| **CSS-controllable presentation** | Default visual styles can be overridden with CSS without changing semantics |
| **Accessibility relevance** | Some elements are announced by screen readers; others require CSS assistance |
| **Nesting allowed** | `<strong>` and `<em>` can be nested to indicate greater degrees of importance or emphasis |

---

### Prerequisites

- Basic familiarity with HTML document structure and elements
- Understanding of inline vs. block-level elements
- Awareness of the separation between content (HTML) and presentation (CSS)
- Basic knowledge of accessibility principles (helpful but not required)

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Semantic text elements are fundamental to screen reader comprehension
- **CSS Typography** – Visual presentation of text elements is controlled via CSS
- **Search Engine Optimization (SEO)** – Semantic markup helps search engines understand content importance
- **HTML Content Categories** – These elements belong to flow content, phrasing content, and palpable content categories
- **Web Standards and Specifications** – Governed by the WHATWG HTML Living Standard

---

### Core Concepts / Features

---

### 1. `<strong>` — Strong Importance

#### Definitions

**Core Definition**

The `<strong>` element indicates that its contents have strong importance, seriousness, or urgency.

**Technical Definition**

The `<strong>` element represents strong importance, seriousness, or urgency for its contents. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. The element accepts only global attributes and has a DOM interface of `HTMLElement` (or `HTMLSpanElement` up to Gecko 1.9.2 / Firefox 4 inclusive). The implicit ARIA role is `strong`, and any ARIA role is permitted.

**Beginner-Friendly Explanation**

Use `<strong>` when something in your text is really important — like a warning, a critical instruction, or a key point. Browsers usually show it in bold, but the bold look is not the point. The point is that the text *matters more* than the surrounding text.

#### Purposes

- To indicate that content has strong importance, seriousness, or urgency
- To mark warnings, alerts, or critical instructions
- To denote paragraphs or sentences of high importance within a document
- To provide a semantic alternative to purely visual bold styling

#### Syntax Rules and Structure

**General Syntax**

```html
<strong>Content of strong importance</strong>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<strong>` | Opening tag; indicates strong importance |
| `Content` | Phrasing content; the text to be marked as important |
| `</strong>` | Closing tag; required |

**Syntax Rules**

- `<strong>` must have both a start tag and an end tag
- It may contain only phrasing content
- It may be nested inside other phrasing-content elements
- It accepts only global attributes

**Constraints and Limitations**

- Do not use `<strong>` merely to apply bold styling; use CSS `font-weight` for that purpose
- Use `<b>` when you want to draw attention without indicating importance
- Use `<em>` when you want to indicate stress emphasis rather than importance

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Warning Message**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Strong Importance Demo</title>
</head>
<body>
    <!-- strong marks the critical warning -->
    <p>
        <strong>Warning:</strong> This action cannot be undone.
    </p>

    <!-- strong marks a high-importance instruction -->
    <p>
        Please <strong>read the terms and conditions</strong> before proceeding.
    </p>
</body>
</html>
```

**Expected Output**

The words “Warning:” and “read the terms and conditions” appear in bold. Screen readers may announce them with increased emphasis.

**Why This Output Occurs**

Browsers apply a default `font-weight: bold` to `<strong>` elements. The semantic meaning (strong importance) is conveyed to assistive technology independently of the visual rendering.

---

**Example 2: Nested Emphasis and Importance**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nested strong and em</title>
</head>
<body>
    <p>
        <em>Always</em> <strong>back up your data</strong> before
        <em>any</em> <strong>major update</strong>.
    </p>
</body>
</html>
```

**Expected Output**

“Always” and “any” appear in italic (stress emphasis). “Back up your data” and “major update” appear in bold (strong importance).

**Why This Output Occurs**

The `<em>` and `<strong>` elements can be nested to combine stress emphasis with strong importance. This is explicitly permitted by the HTML specification and creates a layered semantic meaning.

#### Real-World Cases

**Case 1: Form Validation Messages**

```html
<p><strong>Error:</strong> The email address you entered is not valid.</p>
```

Screen readers announce the error with the `strong` role, making it stand out.

**Case 2: Legal Disclaimers**

```html
<p><strong>Important:</strong> By using this service, you agree to our terms.</p>
```

The strong importance conveys the seriousness of the legal agreement.

**Case 3: Product Warnings**

```html
<p><strong>Do not</strong> use this product near water.</p>
```

The warning is semantically marked as critical, not just visually bold.

---

### 2. `<em>` — Stress Emphasis

#### Definitions

**Core Definition**

The `<em>` element marks text that has stress emphasis, which changes the meaning of the sentence when spoken.

**Technical Definition**

The `<em>` element marks text that has stress emphasis. It can be nested, with each level of nesting indicating a greater degree of emphasis. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. It accepts only global attributes and has an implicit ARIA role of “no corresponding role.” The DOM interface is `HTMLElement` (or `HTMLSpanElement` up to Firefox 4).

**Beginner-Friendly Explanation**

Use `<em>` when you want to stress a word or phrase — like when you say “I *didn‘t* take your pen” with extra emphasis on “didn’t.” The emphasis changes the meaning of the sentence. Browsers usually show it in italics, but the italics are just the visual cue, not the purpose.

#### Purposes

- To indicate stress emphasis that affects the meaning of a sentence
- To mark words or phrases that would be stressed when spoken aloud
- To nest for progressively greater degrees of emphasis
- To provide a semantic alternative to purely visual italic styling

#### Syntax Rules and Structure

**General Syntax**

```html
<em>Emphasised content</em>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<em>` | Opening tag; indicates stress emphasis |
| `Content` | Phrasing content; the text to be emphasised |
| `</em>` | Closing tag; required |

**Syntax Rules**

- `<em>` must have both a start tag and an end tag
- It may contain only phrasing content
- It may be nested, with each level indicating greater emphasis
- It accepts only global attributes

**Constraints and Limitations**

- Do not use `<em>` merely to apply italic styling; use CSS `font-style` for that purpose
- Use `<i>` for text in an alternate voice or mood (scientific names, foreign words, etc.)
- Use `<cite>` for titles of works (books, plays, songs)

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Stress Emphasis in a Sentence**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Emphasis Demo</title>
</head>
<body>
    <!-- The emphasis is on "now" -->
    <p>I need you to submit the report <em>now</em>, not tomorrow.</p>

    <!-- The emphasis is on "I" -->
    <p><em>I</em> didn't say that — she did.</p>
</body>
</html>
```

**Expected Output**

“Now” and “I” appear in italic. Screen readers may change prosody to indicate emphasis.

**Why This Output Occurs**

The `<em>` element marks the words that carry the stress emphasis. The italic rendering is a default browser style, but the semantic meaning is that those words are stressed in the sentence.

---

**Example 2: Nested Emphasis**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nested em</title>
</head>
<body>
    <p>
        <em>I</em> <strong>really</strong> <em>mean</em> <em><em>exactly</em></em> what I said.
    </p>
</body>
</html>
```

**Expected Output**

“Exactly” appears with the greatest degree of emphasis (nested `<em>` elements).

**Why This Output Occurs**

Each level of nested `<em>` indicates a greater degree of emphasis. The specification explicitly allows nesting of `<em>` elements.

#### Real-World Cases

**Case 1: Dialogue in Fiction**

```html
<p>“You <em>promised</em> you would come,” she said.</p>
```

The emphasis on “promised” reflects the character‘s tone.

**Case 2: User Instructions**

```html
<p>Press <em>Ctrl</em> + <em>S</em> to save your work.</p>
```

The keys are emphasised because they are the critical actions.

**Case 3: Contrastive Statements**

```html
<p>The <em>first</em> option is free; the <em>second</em> costs money.</p>
```

The contrastive emphasis helps readers distinguish between the options.

---

### 3. `<b>` — Bring Attention To

#### Definitions

**Core Definition**

The `<b>` element draws attention to text for utilitarian purposes without conveying any extra importance or emphasis.

**Technical Definition**

The `<b>` element represents a span of text offset from its surrounding content without conveying any extra emphasis or importance, and for which the conventional typographic presentation is bold text. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. It accepts only global attributes.

**Beginner-Friendly Explanation**

Use `<b>` when you want to make text bold just to catch the eye — like keywords in a summary or product names in a review — but the text is not actually more important or emphasised. It‘s a purely visual offset with no semantic weight.

#### Purposes

- To draw attention to text without implying importance or emphasis
- To mark keywords in a document summary or abstract
- To highlight product names in a review or commentary
- To provide a semantic alternative to bold styling where no stronger element applies

#### Syntax Rules and Structure

**General Syntax**

```html
<b>Content to bring attention to</b>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<b>` | Opening tag; draws attention without semantic weight |
| `Content` | Phrasing content |
| `</b>` | Closing tag; required |

**Syntax Rules**

- `<b>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- Use it as a last resort when no other element is more appropriate

**Constraints and Limitations**

- Do not use `<b>` for headings; use `<h1>`–`<h6>`
- Do not use `<b>` for emphasised text; use `<em>`
- Do not use `<b>` for important text; use `<strong>`
- Do not use `<b>` for marked/highlighted text; use `<mark>`
- The WHATWG specification says: “The `b` element should be used as a last resort when no other element is more appropriate”

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Keywords in a Summary**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>b Element Demo</title>
</head>
<body>
    <h1>Article Summary</h1>
    <p>
        This study examines the effects of <b>climate change</b> on
        <b>coral reef</b> ecosystems in the Pacific Ocean.
    </p>
</body>
</html>
```

**Expected Output**

“Climate change” and “coral reef” appear in bold. Screen readers do not announce any special role.

**Why This Output Occurs**

The `<b>` element applies bold styling by default but carries no semantic meaning. It is appropriate here because the terms are keywords for scanning, not important or emphasised in a semantic sense.

---

**Example 2: Product Names in a Review**

```html
<p>
    I compared the <b>UltraBook Pro</b> with the <b>PowerLaptop X</b>
    and found the <b>UltraBook Pro</b> had better battery life.
</p>
```

**Expected Output**

The product names appear in bold, making them easy to scan.

**Why This Output Occurs**

Product names in a review are drawn to attention for utilitarian purposes. They are not more important than the surrounding text, nor are they emphasised. The `<b>` element is appropriate here.

#### Real-World Cases

**Case 1: Document Abstracts**

Academic papers use `<b>` for keywords in abstracts to help readers scan.

**Case 2: Shopping Comparison Tables**

Product comparison pages use `<b>` for product names to distinguish them visually.

**Case 3: Chat Interfaces**

Some chat applications use `<b>` for usernames to make them stand out in a conversation.

---

### 4. `<i>` — Idiomatic Text

#### Definitions

**Core Definition**

The `<i>` element represents text in an alternate voice or mood, or text otherwise offset from normal prose, typically rendered in italics.

**Technical Definition**

The `<i>` element represents a span of text offset from its surrounding content without conveying any extra emphasis or importance, and for which the conventional typographic presentation is italic text. It is categorised as flow content and phrasing content. Its permitted content is phrasing content. It accepts only global attributes.

**Beginner-Friendly Explanation**

Use `<i>` for text that is different from the normal flow — like scientific names, foreign words, technical terms, or a character’s inner thoughts. It‘s not emphasised or important; it’s just “in a different voice.” Browsers usually show it in italics.

#### Purposes

- To mark text in an alternate voice or mood
- To denote taxonomic designations (e.g., *Homo sapiens*)
- To mark foreign-language idioms (e.g., *et cetera*)
- To indicate technical terms, transliterations, or ship names
- To represent a character‘s thoughts in narrative text

#### Syntax Rules and Structure

**General Syntax**

```html
<i>Idiomatic content</i>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<i>` | Opening tag; indicates alternate voice or mood |
| `Content` | Phrasing content |
| `</i>` | Closing tag; required |

**Syntax Rules**

- `<i>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- Use the `lang` attribute to identify the language of foreign words

**Constraints and Limitations**

- Do not use `<i>` for emphasis; use `<em>`
- Do not use `<i>` for important text; use `<strong>`
- Do not use `<i>` merely for italic styling; use CSS `font-style`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Scientific Name**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>i Element Demo</title>
</head>
<body>
    <p>
        The <i>Musa</i> genus includes bananas and plantains.
    </p>
    <p>
        The term <i>bandwidth</i> describes how much data can pass
        through a connection.
    </p>
</body>
</html>
```

**Expected Output**

“Musa” and “bandwidth” appear in italic. No emphasis or importance is announced.

**Why This Output Occurs**

Scientific names are conventionally italicised and represent an alternate voice (taxonomic nomenclature). The `<i>` element is the correct semantic choice.

---

**Example 2: Foreign Words and Thoughts**

```html
<p>I looked at the painting and thought, <i>this can't be real!</i></p>
<p>She said <i lang="fr">c'est la vie</i> and walked away.</p>
```

**Expected Output**

“This can‘t be real!” and “c’est la vie” appear in italic.

**Why This Output Occurs**

The character‘s thought is in an alternate voice, and the French phrase is a foreign-language idiom. The `lang` attribute on the French phrase helps screen readers pronounce it correctly.

#### Real-World Cases

**Case 1: Biological Taxonomy**

Biology textbooks use `<i>` for genus and species names: *Escherichia coli*, *Canis lupus*.

**Case 2: Technical Documentation**

Technical writers use `<i>` for variable names or terms being defined: “The <i>timeout</i> parameter controls how long the request waits.”

**Case 3: Literary Fiction**

Novelists use `<i>` for internal monologue: “<i>I should have known better</i>, she thought.”

---

### 5. `<u>` — Unarticulated Annotation (Underline)

#### Definitions

**Core Definition**

The `<u>` element represents inline text that has a non-textual annotation, typically rendered as an underline.

**Technical Definition**

The `<u>` element represents a span of inline text which should be rendered in a way that indicates that it has a non-textual annotation. This is rendered by default as a single solid underline but may be altered using CSS. It was deprecated in HTML 4 but restored in HTML 5 with a new semantic meaning: to mark text as having some form of non-textual annotation applied.

**Beginner-Friendly Explanation**

Use `<u>` when text has some kind of marking that isn’t part of the actual words — like a spelling error annotation. It shows as an underline by default, but you can change that with CSS. Do **not** use `<u>` just to underline text for decoration, and never use it in a way that could be confused with a hyperlink.

#### Purposes

- To indicate spelling or grammatical errors in text
- To apply proper-name marks (e.g., in Chinese text)
- To represent other non-textual annotations
- To provide a semantic container for annotation styling via CSS

#### Syntax Rules and Structure

**General Syntax**

```html
<u>Annotated content</u>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<u>` | Opening tag; indicates non-textual annotation |
| `Content` | Phrasing content |
| `</u>` | Closing tag; required |

**Syntax Rules**

- `<u>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- Use CSS `text-decoration` to control the visual style

**Constraints and Limitations**

- Avoid the default underline style in contexts where it could be confused with a hyperlink
- Do not use `<u>` to underline text for presentation; use CSS
- Do not use `<u>` to denote titles of books; use `<cite>`
- Valid use cases are limited to spelling errors, proper-name marks, and similar annotations

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Spelling Error Annotation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>u Element Demo</title>
    <style>
        u.spelling {
            text-decoration: red wavy underline;
        }
    </style>
</head>
<body>
    <p>
        This paragraph includes a <u class="spelling">wrnogly</u>
        spelled word.
    </p>
</body>
</html>
```

**Expected Output**

“wrnogly” appears with a red wavy underline, the conventional annotation for a spelling error.

**Why This Output Occurs**

The `<u>` element marks the text as having a non-textual annotation. The CSS class `.spelling` overrides the default solid underline with a red wavy underline, matching the conventional presentation of spelling errors.

---

**Example 2: Proper-Name Mark (Chinese Text)**

```html
<p lang="zh-Hans">
    他叫<u>王小明</u>。
</p>
```

**Expected Output**

The name “王小明” appears with an underline, which is the proper-name mark in Chinese typography.

**Why This Output Occurs**

The `<u>` element represents a non-textual annotation — in this case, a proper-name mark that indicates the underlined text is a personal name.

#### Real-World Cases

**Case 1: Spell-Check Interfaces**

Web-based text editors use `<u>` to highlight misspelled words with a red wavy underline.

**Case 2: Chinese Typography**

Chinese-language websites use `<u>` for proper-name marks in certain contexts.

**Case 3: Annotation Systems**

Annotation platforms use `<u>` to mark text that has an associated note or comment.

---

### 6. `<mark>` — Marked/Highlighted Text

#### Definitions

**Core Definition**

The `<mark>` element represents text marked or highlighted for reference or notation purposes due to its relevance in the enclosing context.

**Technical Definition**

The `<mark>` element represents text which is marked or highlighted for reference or notation purposes due to the marked passage‘s relevance in the enclosing context. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. It accepts only global attributes.

**Beginner-Friendly Explanation**

Use `<mark>` when you want to highlight text like a highlighter pen — for example, the search terms that matched your query, or a passage in a quote that is especially relevant. It’s about *relevance* in context, not importance.

#### Purposes

- To highlight search terms that matched a user‘s query
- To mark passages of special interest within a quotation
- To indicate content relevant to the user’s current activity
- To provide a semantic alternative to purely visual highlighting

#### Syntax Rules and Structure

**General Syntax**

```html
<mark>Marked content</mark>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<mark>` | Opening tag; indicates marked/highlighted text |
| `Content` | Phrasing content |
| `</mark>` | Closing tag; required |

**Syntax Rules**

- `<mark>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- Use CSS to customise the highlight colour

**Constraints and Limitations**

- Do not use `<mark>` for syntax highlighting; use `<span>` with CSS
- Do not confuse `<mark>` with `<strong>`: `<mark>` indicates relevance, `<strong>` indicates importance
- The presence of `<mark>` is not announced by most screen readers by default; CSS `content` can be used to add announcements

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Search Results Highlighting**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>mark Element Demo</title>
    <style>
        mark {
            background-color: #ffeb3b;
            padding: 0 2px;
        }
    </style>
</head>
<body>
    <p>Search results for "salamander":</p>
    <hr>
    <p>
        Several species of <mark>salamander</mark> inhabit the
        temperate rainforest of the Pacific Northwest.
    </p>
    <p>
        Most <mark>salamander</mark>s are nocturnal, and hunt for
        insects, worms, and other small creatures.
    </p>
</body>
</html>
```

**Expected Output**

Each occurrence of “salamander” is highlighted with a yellow background.

**Why This Output Occurs**

The `<mark>` element marks the search terms as relevant to the user‘s current activity (searching for “salamander”). The CSS provides a visual highlight that mimics the default browser style.

---

**Example 2: Marking Text in a Quote**

```html
<blockquote>
    <p>
        It is a truth universally acknowledged, that a single man
        in possession of a good fortune, must be in want of a wife.
    </p>
    <p>
        However little known the feelings or views of such a man
        may be on his first entering a neighbourhood, this truth is
        so well fixed in the minds of the surrounding families, that
        he is considered as the rightful property of some one or other
        of their daughters.
    </p>
    <p>
        <mark>“My dear Mr. Bennet,”</mark> said his lady to him one day,
        “have you heard that Netherfield Park is let at last?”
    </p>
</blockquote>
```

**Expected Output**

The opening dialogue “My dear Mr. Bennet” is highlighted, drawing the reader‘s attention to a passage of special interest within the quote.

**Why This Output Occurs**

When `<mark>` is used inside a quotation, it indicates text that is of special interest but is not marked in the original source material — like using a highlighter pen in a book.

#### Real-World Cases

**Case 1: Search Engines**

Google and other search engines highlight matched query terms with `<mark>` in search result snippets.

**Case 2: E-Readers**

E-reader applications use `<mark>` to show passages the user has highlighted.

**Case 3: Documentation Search**

Documentation platforms highlight search terms within article text using `<mark>`.

---

### 7. `<small>` — Side Comment

#### Definitions

**Core Definition**

The `<small>` element represents side-comments and small print, such as copyright notices and legal disclaimers.

**Technical Definition**

The `<small>` element represents side-comments and small print, like copyright and legal text, independent of its styled presentation. By default, it renders text within it one font-size smaller. It is categorised as flow content and phrasing content. Its permitted content is phrasing content. Its implicit ARIA role is `generic`.

**Beginner-Friendly Explanation**

Use `<small>` for the “fine print” — copyright notices, legal disclaimers, caveats, and side comments. It usually shows in smaller text, but the size is not the point; the point is that it’s ancillary information.

#### Purposes

- To mark copyright notices and legal disclaimers
- To indicate caveats, conditions, or side comments
- To represent “fine print” or “small print” in legal documents
- To provide a semantic container for ancillary information

#### Syntax Rules and Structure

**General Syntax**

```html
<small>Fine print content</small>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<small>` | Opening tag; indicates side comment or small print |
| `Content` | Phrasing content |
| `</small>` | Closing tag; required |

**Syntax Rules**

- `<small>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- It does not “de-emphasise” content marked with `<em>` or `<strong>`

**Constraints and Limitations**

- Do not use `<small>` for entire paragraphs or sections of text
- It should be used for short, ancillary content only
- CSS can override the default smaller font size

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Copyright Notice**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>small Element Demo</title>
</head>
<body>
    <p>MDN Web Docs is a learning platform for Web technologies.</p>
    <hr>
    <p>
        <small>
            The content is licensed under a Creative Commons
            Attribution-ShareAlike 2.5 Generic License.
        </small>
    </p>
</body>
</html>
```

**Expected Output**

The copyright notice appears in smaller text below a horizontal rule.

**Why This Output Occurs**

The `<small>` element semantically marks the license text as a side comment or fine print. The default browser styling renders it one font-size smaller.

---

**Example 2: Side Comment Within a Sentence**

```html
<p>
    The event starts at 7pm.
    <small>(Doors open at 6:30pm.)</small>
</p>
```

**Expected Output**

“(Doors open at 6:30pm.)” appears in smaller text as a side comment within the paragraph.

**Why This Output Occurs**

The `<small>` element marks the parenthetical note as ancillary information that supplements the main sentence.

#### Real-World Cases

**Case 1: E-Commerce Footers**

Online stores use `<small>` for terms and conditions, return policies, and copyright notices.

**Case 2: Legal Documents**

Contracts use `<small>` for disclaimers and caveats that are legally required but not central to the main text.

**Case 3: News Articles**

News websites use `<small>` for bylines, publication dates, and correction notices.

---

### 8. `<del>` — Deleted Text

#### Definitions

**Core Definition**

The `<del>` element represents a range of text that has been deleted from a document.

**Technical Definition**

The `<del>` element represents a range of text that has been deleted from a document. It is often rendered with a strikethrough style. It is categorised as phrasing content and flow content. It supports the `cite` and `datetime` attributes in addition to global attributes. It is commonly paired with `<ins>` to show document edits.

**Beginner-Friendly Explanation**

Use `<del>` when you want to show text that was removed from a document — like in “track changes” mode in a word processor. It usually appears with a line through it.

#### Purposes

- To indicate text that has been deleted from a document
- To show edits in “track changes” or diff views
- To mark outdated or removed content in collaborative documents
- To provide a semantic alternative to strikethrough styling

#### Syntax Rules and Structure

**General Syntax**

```html
<del>Deleted content</del>
```

With optional attributes:

```html
<del cite="URL" datetime="date">Deleted content</del>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<del>` | Opening tag; indicates deleted text |
| `cite` | Optional; URI explaining the change |
| `datetime` | Optional; date and time of the change |
| `Content` | Phrasing content |
| `</del>` | Closing tag; required |

**Syntax Rules**

- `<del>` must have both a start tag and an end tag
- It may contain only phrasing content
- The `cite` attribute must be a valid URI
- The `datetime` attribute must be a valid date string

**Constraints and Limitations**

- The `<del>` element should not cross implied paragraph boundaries
- The presence of `<del>` is not announced by most screen readers by default
- Use `<s>` for content that is no longer accurate or relevant but not necessarily deleted

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Track Changes in a Document**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>del Element Demo</title>
    <style>
        del {
            background-color: #ffbbbb;
            text-decoration: line-through;
        }
    </style>
</head>
<body>
    <p>“You're late!”</p>
    <del>
        <p>“I apologize for the delay.”</p>
    </del>
    <ins cite="../how-to-be-a-wizard.html" datetime="2018-05">
        <p>“A wizard is never late…”</p>
    </ins>
</body>
</html>
```

**Expected Output**

The apology is shown with a strikethrough and a pink background (deleted), while the wizard quote is shown with an insertion marker and a green background (inserted).

**Why This Output Occurs**

The `<del>` element marks the removed text, and the `<ins>` element marks the added replacement. The CSS provides visual cues for the edit, and the `cite` and `datetime` attributes provide metadata about the change.

---

**Example 2: Price Change**

```html
<p>
    <del datetime="2026-01-01">Was: $49.99</del><br>
    Now: $39.99
</p>
```

**Expected Output**

“Was: $49.99” appears with a strikethrough, while “Now: $39.99” appears normally.

**Why This Output Occurs**

The `<del>` element semantically marks the old price as deleted, and the `datetime` attribute records when the deletion occurred.

#### Real-World Cases

**Case 1: Collaborative Editing Platforms**

Google Docs and similar platforms use `<del>` (or equivalent markup) to show tracked deletions.

**Case 2: E-Commerce Price Changes**

Online stores use `<del>` to show original prices that have been reduced.

**Case 3: Version Control Interfaces**

GitHub and similar tools use `<del>` (or equivalent) to show deleted lines in diff views.

---

### 9. `<ins>` — Inserted Text

#### Definitions

**Core Definition**

The `<ins>` element represents a range of text that has been added to a document.

**Technical Definition**

The `<ins>` element represents a range of text that has been added to a document. It is used alongside `<del>` to show document edits. It supports the `cite` and `datetime` attributes. It is categorised as flow content and phrasing content.

**Beginner-Friendly Explanation**

Use `<ins>` when you want to show text that was added to a document — the opposite of `<del>`. In track-changes views, insertions are typically underlined or highlighted.

#### Purposes

- To indicate text that has been added to a document
- To show insertions in “track changes” or diff views
- To mark new content in collaborative documents
- To provide a semantic alternative to underline styling for additions

#### Syntax Rules and Structure

**General Syntax**

```html
<ins>Inserted content</ins>
```

With optional attributes:

```html
<ins cite="URL" datetime="date">Inserted content</ins>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<ins>` | Opening tag; indicates inserted text |
| `cite` | Optional; URI explaining the change |
| `datetime` | Optional; date and time of the change |
| `Content` | Phrasing content |
| `</ins>` | Closing tag; required |

**Syntax Rules**

- `<ins>` must have both a start tag and an end tag
- It may contain only phrasing content
- The `cite` attribute must be a valid URI
- The `datetime` attribute must be a valid date string

**Constraints and Limitations**

- The presence of `<ins>` is not announced by most screen readers by default
- It should not cross implied paragraph boundaries
- Use `<u>` for non-textual annotations, not for insertions

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Insertion in Track Changes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ins Element Demo</title>
    <style>
        ins {
            background-color: #d4fcbc;
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <p>“You're late!”</p>
    <del>
        <p>“I apologize for the delay.”</p>
    </del>
    <ins cite="../how-to-be-a-wizard.html" datetime="2018-05">
        <p>“A wizard is never late…”</p>
    </ins>
</body>
</html>
```

**Expected Output**

The wizard quote appears with a green background and underline, indicating it is an insertion.

**Why This Output Occurs**

The `<ins>` element marks the added text, and the CSS provides visual cues for the insertion. The `cite` and `datetime` attributes provide metadata about the change.

---

**Example 2: Adding a Correction**

```html
<p>
    The meeting is on <del>Tuesday</del> <ins>Wednesday</ins>.
</p>
```

**Expected Output**

“Tuesday” appears with a strikethrough, and “Wednesday” appears underlined.

**Why This Output Occurs**

The `<del>` and `<ins>` elements together show a correction: the original date was deleted and a new date was inserted.

#### Real-World Cases

**Case 1: Legal Document Revisions**

Law firms use `<ins>` and `<del>` to show proposed changes in contract negotiations.

**Case 2: Wikipedia Edit History**

Wikipedia uses equivalent markup to show inserted and deleted text in article diffs.

**Case 3: Collaborative Writing Tools**

Writing platforms use `<ins>` to show additions made by co-authors.

---

### 10. `<s>` — Strikethrough

#### Definitions

**Core Definition**

The `<s>` element renders text with a strikethrough, representing content that is no longer accurate or no longer relevant.

**Technical Definition**

The `<s>` element represents contents that are no longer accurate or no longer relevant. It is categorised as phrasing content and flow content. Its permitted content is phrasing content. It accepts only global attributes.

**Beginner-Friendly Explanation**

Use `<s>` for things that used to be true but aren‘t anymore — like a sold-out item at a store or an outdated price. It shows a line through the text. Do **not** use `<s>` for document edits; use `<del>` for that.

#### Purposes

- To indicate content that is no longer accurate
- To mark items that are no longer available or relevant
- To show outdated information that should not be removed
- To provide a semantic alternative to strikethrough styling

#### Syntax Rules and Structure

**General Syntax**

```html
<s>Outdated content</s>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<s>` | Opening tag; indicates no-longer-accurate content |
| `Content` | Phrasing content |
| `</s>` | Closing tag; required |

**Syntax Rules**

- `<s>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes

**Constraints and Limitations**

- Do not use `<s>` for document edits; use `<del>` and `<ins>`
- Do not use `<s>` for content that has been deleted; use `<del>`
- The `<strike>` element is obsolete and must not be used

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Sold-Out Item**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>s Element Demo</title>
</head>
<body>
    <p>
        <s>Today's Special: Salmon</s> SOLD OUT
    </p>
    <p>
        <s>Was: $49.99</s> Now: $39.99
    </p>
</body>
</html>
```

**Expected Output**

“Today‘s Special: Salmon” and “Was: $49.99” appear with a strikethrough.

**Why This Output Occurs**

The `<s>` element indicates that the content is no longer accurate or relevant — the special is sold out, and the old price is no longer valid.

---

**Example 2: Outdated Information**

```html
<p>
    <s>Office hours: Monday–Friday, 9am–5pm</s><br>
    New office hours: Monday–Thursday, 8am–4pm
</p>
```

**Expected Output**

The old office hours appear with a strikethrough, while the new hours appear normally.

**Why This Output Occurs**

The `<s>` element marks the old information as no longer accurate, while preserving it for reference.

#### Real-World Cases

**Case 1: E-Commerce Availability**

Online stores use `<s>` to mark products that are sold out or discontinued.

**Case 2: Price Comparisons**

Price comparison sites use `<s>` to show original prices that have changed.

**Case 3: Policy Updates**

Organizations use `<s>` to strike through outdated policies while showing the new version.

---

### 11. `<sub>` — Subscript

#### Definitions

**Core Definition**

The `<sub>` element specifies inline text that should be displayed as subscript for typographical reasons.

**Technical Definition**

The `<sub>` element represents a subscript. Subscripts are typically rendered with a lowered baseline using smaller text. It is categorised as phrasing content and flow content. It accepts only global attributes.

**Beginner-Friendly Explanation**

Use `<sub>` when you need text to appear below the normal baseline — like the “2” in the chemical formula H₂O or footnote numbers. It’s for typographical reasons, not for styling.

#### Purposes

- To mark footnote or endnote reference numbers
- To indicate subscripts in mathematical variable families
- To show the number of atoms in a chemical formula
- To provide a semantic alternative to lowered-baseline styling

#### Syntax Rules and Structure

**General Syntax**

```html
<sub>Subscript content</sub>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<sub>` | Opening tag; indicates subscript |
| `Content` | Phrasing content |
| `</sub>` | Closing tag; required |

**Syntax Rules**

- `<sub>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- The `<sub>` and `<sup>` elements cannot be used simultaneously

**Constraints and Limitations**

- Use only for typographical reasons, not for visual effects
- For complex mathematical formulas with both subscripts and superscripts, use MathML
- Do not use `<sub>` for styling; use CSS `vertical-align`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Chemical Formula**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>sub Element Demo</title>
</head>
<body>
    <p>
        The chemical formula for caffeine is
        C<sub>8</sub>H<sub>10</sub>N<sub>4</sub>O<sub>2</sub>.
    </p>
</body>
</html>
```

**Expected Output**

The formula appears as C₈H₁₀N₄O₂, with the numbers lowered and reduced in size.

**Why This Output Occurs**

The `<sub>` elements mark the atom counts as subscripts, following the typographical convention for chemical formulas.

---

**Example 2: Footnote Reference**

```html
<p>
    This is a statement that needs a citation.<sub>1</sub>
</p>
<p>
    <small><sub>1</sub> Source: Journal of Examples, 2026.</small>
</p>
```

**Expected Output**

The footnote number “1” appears as a subscript after the sentence, and again at the bottom of the page.

**Why This Output Occurs**

The `<sub>` element marks the footnote reference number as a subscript, following standard typographical convention.

#### Real-World Cases

**Case 1: Academic Papers**

Academic papers use `<sub>` for footnote and endnote reference numbers.

**Case 2: Chemistry Textbooks**

Chemistry content uses `<sub>` for atom counts in chemical formulas.

**Case 3: Mathematical Notation**

Mathematical texts use `<sub>` for variable subscripts: x₁, x₂, x₃.

---

### 12. `<sup>` — Superscript

#### Definitions

**Core Definition**

The `<sup>` element specifies inline text that should be displayed as superscript for typographical reasons.

**Technical Definition**

The `<sup>` element represents a superscript. Superscripts are usually rendered with a raised baseline using smaller text. It is categorised as flow content, phrasing content, and palpable content. It accepts only global attributes.

**Beginner-Friendly Explanation**

Use `<sup>` when you need text to appear above the normal baseline — like the “3” in x³ or the “th” in “4th.” It‘s for typographical reasons, not for styling.

#### Purposes

- To display exponents and mathematical powers
- To show ordinal suffixes (e.g., 1st, 2nd, 3rd)
- To mark superior lettering in abbreviations (e.g., Mlle)
- To provide a semantic alternative to raised-baseline styling

#### Syntax Rules and Structure

**General Syntax**

```html
<sup>Superscript content</sup>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<sup>` | Opening tag; indicates superscript |
| `Content` | Phrasing content |
| `</sup>` | Closing tag; required |

**Syntax Rules**

- `<sup>` must have both a start tag and an end tag
- It may contain only phrasing content
- It accepts only global attributes
- The `<sub>` and `<sup>` elements cannot be used simultaneously

**Constraints and Limitations**

- Use only for typographical reasons, not for visual effects
- For complex mathematical formulas, use MathML
- Do not use `<sup>` for styling; use CSS `vertical-align`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Exponent**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>sup Element Demo</title>
</head>
<body>
    <p>
        The area of a square with side length x is x<sup>2</sup>.
    </p>
    <p>
        The volume of a cube with side length x is x<sup>3</sup>.
    </p>
</body>
</html>
```

**Expected Output**

The exponents “2” and “3” appear raised above the baseline and in smaller text.

**Why This Output Occurs**

The `<sup>` elements mark the exponents as superscripts, following the typographical convention for mathematical notation.

---

**Example 2: Ordinal Suffix**

```html
<p>
    She finished in 1<sup>st</sup> place, he finished in 2<sup>nd</sup> place.
</p>
```

**Expected Output**

“st” and “nd” appear as superscripts after the numbers.

**Why This Output Occurs**

The `<sup>` elements mark the ordinal suffixes as superscripts, following standard English typographical convention.

#### Real-World Cases

**Case 1: Mathematical Documentation**

Technical documentation uses `<sup>` for exponents: E = mc².

**Case 2: French Abbreviations**

French text uses `<sup>` for superior lettering: M<sup>lle</sup> for “Mademoiselle.”

**Case 3: Footnotes and Endnotes**

Some citation styles use `<sup>` for reference numbers (an alternative to `<sub>`).

---

### 13. Choosing Semantic Elements Rather Than Purely Visual Elements

#### Definitions

**Core Definition**

Choosing semantic elements means selecting HTML elements based on the meaning of the content they contain, not on how they will look by default.

**Technical Definition**

Semantic HTML is the practice of using HTML elements according to their defined meaning and purpose as specified by the WHATWG HTML Living Standard, rather than using elements solely for their default visual presentation. This practice ensures that content is accessible to assistive technologies, understandable by search engines, and maintainable by developers.

**Beginner-Friendly Explanation**

Don’t pick a tag because it makes your text look the way you want. Pick a tag because it *means* the right thing. Then use CSS to make it look however you want. For example, if you want important text to be bold, use `<strong>` (which means important), not `<b>` (which just means bold). If you want italic text for a book title, use `<cite>`, not `<i>`.

#### Purposes

- To ensure content is accessible to screen readers and other assistive technologies
- To make HTML meaningful and understandable to search engines
- To separate content structure (HTML) from visual presentation (CSS)
- To improve code maintainability and consistency
- To comply with web standards and accessibility guidelines

#### Syntax Rules and Structure

**The Core Principle**

Use the most semantic element that fits your content. Only use generic or purely visual elements (`<b>`, `<i>`, `<div>`, `<span>`) when no more semantic element is appropriate.

**Decision Guide**

| If you want to… | Use… | Not… |
|---|---|---|
| Mark important text | `<strong>` | `<b>` |
| Mark stressed emphasis | `<em>` | `<i>` |
| Mark a book title | `<cite>` | `<i>` |
| Mark a foreign word | `<i lang="...">` | `<em>` |
| Highlight a search term | `<mark>` | `<span>` with background |
| Show deleted text | `<del>` | `<s>` |
| Show inserted text | `<ins>` | `<u>` |
| Show outdated content | `<s>` | `<del>` |
| Make text bold for no semantic reason | CSS `font-weight` | `<b>` |
| Make text italic for no semantic reason | CSS `font-style` | `<i>` |
| Underline text for decoration | CSS `text-decoration` | `<u>` |

**Syntax Rules**

- Always ask: “What does this text *mean*?” before choosing an element
- Use CSS for presentation, HTML for meaning
- Never choose an element based solely on its default visual appearance
- If no semantic element fits, use `<span>` or `<div>` with CSS classes

**Constraints and Limitations**

- Overusing generic elements (`<div>`, `<span>`) can harm accessibility
- Misusing semantic elements (e.g., `<em>` for italic styling) can confuse assistive technology
- The WHATWG specification notes that `<b>` and `<i>` are valid but should be used judiciously

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct vs. Incorrect Element Choice**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Semantic Choice Demo</title>
    <style>
        .bold-only {
            font-weight: bold;
        }
    </style>
</head>
<body>
    <!-- CORRECT: strong for importance -->
    <p>
        <strong>Warning:</strong> Do not delete this file.
    </p>

    <!-- CORRECT: b for attention without importance -->
    <p>
        The keywords in this summary are <b>climate change</b>
        and <b>coral reefs</b>.
    </p>

    <!-- CORRECT: em for stress emphasis -->
    <p>
        I said <em>tomorrow</em>, not today.
    </p>

    <!-- CORRECT: i for foreign words -->
    <p>
        She said <i lang="fr">c'est la vie</i>.
    </p>

    <!-- INCORRECT: using b for importance -->
    <p>
        <b>Warning:</b> This is wrong — b has no semantic importance.
    </p>

    <!-- CORRECT: CSS for purely visual bold -->
    <p>
        This text is <span class="bold-only">bold for no semantic reason</span>.
    </p>
</body>
</html>
```

**Expected Output**

All the bold text looks similar visually, but the semantic meaning differs. Screen readers will announce `<strong>` as important but will not announce `<b>` or `.bold-only`.

**Why This Output Occurs**

Browsers apply default styles to `<strong>` and `<b>` (both bold) and to `<em>` and `<i>` (both italic). However, only `<strong>` and `<em>` carry semantic meaning that assistive technology can interpret. The `.bold-only` class applies visual styling without any semantic meaning.

---

**Example 2: Book Title — `<cite>` vs. `<i>`**

```html
<!-- CORRECT: cite for book titles -->
<p>
    I am reading <cite>The Great Gatsby</cite>.
</p>

<!-- INCORRECT: i for book titles -->
<p>
    I am reading <i>The Great Gatsby</i>.
</p>
```

**Expected Output**

Both render in italic, but only `<cite>` semantically identifies the text as the title of a work.

**Why This Output Occurs**

The `<cite>` element is defined specifically for marking the title of a work. Using `<i>` for a book title loses the semantic information that the text is a citation.

#### Real-World Cases

**Case 1: News Websites**

Major news organisations use semantic elements to structure content. Headlines use `<h1>`, important terms use `<strong>`, book and film titles use `<cite>`, and foreign phrases use `<i lang="...">`.

**Case 2: E-Commerce Platforms**

Product pages use `<strong>` for critical information (price, availability), `<del>` for original prices, `<s>` for sold-out items, and `<mark>` for search-matched terms.

**Case 3: Government Accessibility Compliance**

Government websites must comply with WCAG guidelines that require semantic HTML. Using `<strong>` instead of `<b>`, and `<em>` instead of `<i>`, is part of meeting Success Criterion 1.3.1 (Info and Relationships).

---

## References

- MDN Web Docs – `<strong>`: The Strong Importance element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/strong
- MDN Web Docs – `<em>`: The Emphasis element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/em
- MDN Web Docs – `<b>`: The Bring Attention To element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/b
- MDN Web Docs – `<i>`: The Idiomatic Text element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/i
- MDN Web Docs – `<u>`: The Unarticulated Annotation element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/u
- MDN Web Docs – `<mark>`: The Mark Text element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/mark
- MDN Web Docs – `<small>`: The Side Comment element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/small
- MDN Web Docs – `<del>`: The Deleted Text element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/del
- MDN Web Docs – `<ins>`: The Inserted Text element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ins
- MDN Web Docs – `<s>`: The Strikethrough element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/s
- MDN Web Docs – `<sub>`: The Subscript element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/sub
- MDN Web Docs – `<sup>`: The Superscript element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/sup
- WHATWG HTML Living Standard – Text-level semantics – https://html.spec.whatwg.org/multipage/text-level-semantics.html
- W3C – Semantic structure – https://accessability.rgd.ca/web-accessibility/practicing-web-accessibility/semantic-structure-2/
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.6: Headings and Labels – https://www.w3.org/WAI/WCAG21/Understanding/headings-and-labels.html
- WHATWG – HTML Living Standard: Text-level semantics – https://html.spec.whatwg.org/multipage/text-level-semantics.html