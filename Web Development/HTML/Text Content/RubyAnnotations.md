# Ruby Annotations — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Ruby annotations are small text annotations rendered above, below, or alongside base text, primarily used in East Asian typography to provide pronunciation guides, translations, or transliterations. HTML provides three elements for this: `<ruby>` (the container), `<rt>` (the annotation text), and `<rp>` (fallback parentheses for browsers that do not support ruby).

**Technical Definition**
In the WHATWG HTML Living Standard, `<ruby>` is a **phrasing-content** element that allows spans of phrasing content to be marked with ruby annotations. Its content model consists of **base text** segments (phrasing content without nested `<ruby>` elements) and **annotation** segments (one or more `<rt>` elements, optionally wrapped with `<rp>` fallback parentheses). The `<rt>` element specifies the ruby text component (the annotation itself), while `<rp>` provides parentheses that are shown only by user agents that do not support ruby rendering. All three elements accept only global attributes.

**Beginner-Friendly Explanation**
Ruby annotations are the small readings printed above East Asian characters — like furigana above Japanese kanji. HTML lets you mark up the base text (the characters) and the ruby text (the reading) so browsers can display the reading in a small font alongside or above the base. The `<rp>` element is a safety net: it provides parentheses that appear only if the browser cannot render ruby, so the reading still makes sense inline.

---

### Key Characteristics

- **Three complementary elements**: `<ruby>` creates the annotation container; `<rt>` holds the reading or annotation; `<rp>` provides fallback parentheses.
- **Content-model driven**: `<ruby>` contains base text and one or more `<rt>` elements, optionally with `<rp>` fallbacks in a specific sequence (`rp, rt, rp, rt, ...`).
- **Baseline widely available**: All three elements have been supported across browsers since July 2015.
- **Accessibility-critical**: WCAG Technique H62 recognises ruby annotation as a sufficient technique for Success Criterion 3.1.6 (Pronunciation).
- **Fallback mechanism**: `<rp>` ensures that annotations remain readable even in browsers that do not support ruby.
- **Multiple annotation types**: Ruby can express pronunciation (mono-ruby, jukugo-ruby), meaning (group ruby), or both (double-sided ruby).

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, tags, and attributes.
2. **Phrasing vs. flow content**: understanding which elements can appear inside paragraphs.
3. **East Asian typography basics**: awareness that Japanese, Chinese, and Korean use characters that may benefit from pronunciation guides.
4. **Accessibility awareness**: understanding that pronunciation information is essential for some users with disabilities.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | Ruby annotations satisfy WCAG 3.1.6 (Pronunciation) via Technique H62. |
| **Internationalisation (i18n)** | Ruby is essential for East Asian language support, especially Japanese furigana. |
| **CSS Ruby** | CSS provides properties such as `ruby-position` and `ruby-align` for controlling ruby rendering. |
| **Screen readers** | Screen readers may read the `<rt>` content instead of the base text to convey pronunciation. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<ruby>`** — the annotation container
2. **`<rt>`** — the ruby text (annotation)
3. **`<rp>`** — the fallback parentheses
4. **Ruby annotation best practices** — authoring guidelines

---

## 1. `<ruby>` — The Ruby Annotation Container

### Definitions

**Core Definition**
The `<ruby>` element represents small annotations that are rendered above, below, or next to base text, primarily used for showing the pronunciation of East Asian characters.

**Technical Definition**
`<ruby>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model consists of one or more **base text segments** followed by **annotation segments**. Base text is phrasing content without `<ruby>` descendants; annotation text is one or more `<rt>` elements, optionally interspersed with `<rp>` fallback elements in the sequence `rp, rt, rp, rt, ...`. Both start and end tags are mandatory.

**Beginner-Friendly Explanation**
`<ruby>` is the wrapper that tells the browser “this text has a small annotation alongside it.” You put the base text inside, followed by the annotation in `<rt>` elements.

---

### Purposes

- **To mark up base text with ruby annotations** for pronunciation, translation, or transliteration.
- **To provide pronunciation guides** (furigana) for Japanese kanji and other East Asian characters.
- **To support semantic annotations** such as translations or definitions alongside base text.
- **To satisfy WCAG 3.1.6 (Pronunciation)** by making pronunciation information programmatically available.

---

### Syntax Rules and Structure

#### Complete General Syntax (Simple)

```html
<ruby>base text<rt>annotation</rt></ruby>
```

#### Complete General Syntax (with `<rp>` fallback)

```html
<ruby>base text<rp>(</rp><rt>annotation</rt><rp>)</rp></ruby>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<ruby>` | The annotation container. |
| Base text | Phrasing content without nested `<ruby>` elements. |
| `<rt>` | The annotation (ruby text). |
| `<rp>` | Fallback parentheses (optional). |

#### Syntax Rules

1. **Content model**: One or more groups, each consisting of base text followed by annotation text.
2. **Base text**: Phrasing content with no `<ruby>` descendants, or a single nested `<ruby>` with no further `<ruby>` descendants.
3. **Annotation text**: One or more `<rt>` elements, or an `<rp>` element followed by one or more `<rt>` elements, each itself followed by an `<rp>` element (i.e. `rp, rt, rp, rt, ...`).
4. **Both start and end tags are mandatory**.

#### Constraints and Limitations

- **No nested `<ruby>` inside base text** (unless in the specific nested-ruby pattern for double-sided annotations).
- **`<rp>` elements are optional**; they provide fallback only.
- **Browser support is broad** (Baseline widely available since July 2015).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mono-Ruby for Individual Characters

**Setup Guide**: Save as `ruby-mono.html` and view in a browser that supports ruby (most modern browsers).

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Mono-Ruby</title>
</head>
<body>
  <!-- Each base character gets its own annotation -->
  <p>
    <ruby>漢<rt>kan</rt></ruby>
    <ruby>字<rt>ji</rt></ruby>
    は日本語です。
  </p>
</body>
</html>
```

**Expected output**: The characters 漢 and 字 each have their readings “kan” and “ji” rendered above them in a smaller font.

**Why this output occurs**: Each `<ruby>` element pairs one base character with one `<rt>` annotation. The browser renders the annotation as ruby text alongside the base.

---

#### Example 2: Group Ruby for a Compound Word

**Setup Guide**: Save as `ruby-group.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Group Ruby</title>
</head>
<body>
  <!-- One annotation for a multi-character base word -->
  <p>
    <ruby>紫陽花<rt>あじさい</rt></ruby>
  </p>
</body>
</html>
```

**Expected output**: The compound word 紫陽花 has the single reading “あじさい” rendered above it.

**Why this output occurs**: Group ruby associates one annotation with multiple base characters, which is appropriate when a one-to-one mapping would be difficult or misleading.

---

### Real-World Cases

1. **Japanese children’s books**: Furigana above kanji to help young readers.
2. **Language-learning materials**: Pronunciation guides for Chinese, Japanese, and Korean.
3. **Accessible educational content**: Satisfying WCAG 3.1.6 by providing pronunciation information for users who need it.

---

### References

- MDN Web Docs — `<ruby>`: The Ruby Annotation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ruby
- W3C — HTML5: The `ruby` element — https://www.w3.org/TR/2012/WD-html5-author-20121025/the-ruby-element.html
- W3C — Technique H62: Using the ruby element — https://www.w3.org/WAI/WCAG22/Techniques/html/H62.html

---

## 2. `<rt>` — The Ruby Text Element

### Definitions

**Core Definition**
The `<rt>` element specifies the ruby text component of a ruby annotation, which is used to provide pronunciation, translation, or transliteration information for East Asian typography.

**Technical Definition**
`<rt>` must always be contained within a `<ruby>` element. It belongs to the **phrasing content** category and accepts only global attributes. Its content is the annotation text itself — the reading, translation, or transliteration that accompanies the base text.

**Beginner-Friendly Explanation**
`<rt>` is the annotation part of a ruby annotation. If the base text is 漢, the `<rt>` contains “kan”. It is the small reading that appears alongside or above the base character.

---

### Purposes

- **To provide the pronunciation** of base text (e.g. furigana).
- **To provide a translation or transliteration** of base text.
- **To carry the annotation content** that is rendered as ruby text.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ruby>base text<rt>annotation</rt></ruby>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<rt>` | The annotation text. |
| Content | Phrasing content — the reading or annotation. |

#### Syntax Rules

1. **`<rt>` must be a child of `<ruby>`**.
2. **One or more `<rt>` elements may follow the base text** to provide multiple annotations.
3. **`<rt>` may be preceded and followed by `<rp>` fallback elements** in the sequence `rp, rt, rp`.

#### Constraints and Limitations

- **`<rt>` outside `<ruby>` is non-conforming**.
- **Screen readers may read `<rt>` content instead of base text** to convey pronunciation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Pronunciation Annotation

**Setup Guide**: Save as `rt-basic.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Ruby Text</title>
</head>
<body>
  <p>
    <ruby>明日<rt>Ashita</rt></ruby>
  </p>
</body>
</html>
```

**Expected output**: The base text 明日 has the reading “Ashita” rendered as ruby text above it.

**Why this output occurs**: The `<rt>` element provides the annotation content. The browser renders it in a smaller font alongside the base text.

---

#### Example 2: Multiple Annotations (Double-Sided Ruby)

**Setup Guide**: Save as `rt-multiple.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Multiple RT</title>
</head>
<body>
  <p>
    <ruby>
      東南
      <rt>とうなん</rt>
      <rt>たつみ</rt>
    </ruby>
    の方角
  </p>
</body>
</html>
```

**Expected output**: The base text 東南 has two annotations — “とうなん” and “たつみ” — rendered on separate sides.

**Why this output occurs**: Multiple `<rt>` elements provide multiple annotations for the same base text. Browsers may render them on different sides (above and below) or in sequence, depending on CSS ruby styling.

---

### Real-World Cases

1. **Japanese furigana**: Readings above kanji in books and educational materials.
2. **Chinese pinyin annotations**: Pronunciation above hanzi for language learners.
3. **Bilingual text**: Translation annotations above base text.

---

### References

- MDN Web Docs — `<rt>`: The Ruby Text element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/rt
- W3C — HTML5: The `rt` element — https://www.w3.org/TR/2012/WD-html5-author-20121025/the-rt-element.html

---

## 3. `<rp>` — The Ruby Parenthesis Element

### Definitions

**Core Definition**
The `<rp>` element is used to provide fallback parentheses for browsers that do not support display of ruby annotations using the `<ruby>` element.

**Technical Definition**
`<rp>` must be a child of `<ruby>`. Its content is typically a single parenthesis character (e.g. `(` or `)`). In browsers that support ruby, `<rp>` is not rendered; in browsers that do not, the `<rp>` content is displayed inline alongside the `<rt>` annotation, so the reading appears as `(annotation)` after the base text.

**Beginner-Friendly Explanation**
`<rp>` is the safety net for old browsers. If a browser cannot show ruby text above the characters, it will show the reading in parentheses after the base text instead. You put the opening parenthesis in one `<rp>` and the closing parenthesis in another.

---

### Purposes

- **To provide fallback parentheses** for non-ruby-supporting browsers.
- **To ensure readability** of annotations in legacy environments.
- **To satisfy the requirement that ruby annotations remain comprehensible even without ruby rendering**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ruby>base text<rp>(</rp><rt>annotation</rt><rp>)</rp></ruby>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<rp>(</rp>` | Opening parenthesis fallback. |
| `<rt>annotation</rt>` | The annotation. |
| `<rp>)</rp>` | Closing parenthesis fallback. |

#### Syntax Rules

1. **`<rp>` must be a child of `<ruby>`**.
2. **The `<rp>` element should contain the opening and closing parentheses** that wrap the `<rt>` element.
3. **In ruby-supporting browsers, `<rp>` content is not rendered**.

#### Constraints and Limitations

- **`<rp>` is optional**; modern browsers support ruby without it, but it is recommended for robustness.
- **The parentheses appear only when ruby rendering is unsupported**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fallback Parentheses for a Japanese Reading

**Setup Guide**: Save as `rp-basic.html`. Test in a modern browser (parentheses hidden) and an older browser (parentheses shown).

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
    は日本語です。
  </p>
</body>
</html>
```

**Expected output**: In a ruby-supporting browser, “kan” and “ji” appear above 漢 and 字, and the parentheses are hidden. In a non-supporting browser, the text reads “漢 (kan) 字 (ji) は日本語です。”

**Why this output occurs**: The `<rp>` elements provide the parentheses that are rendered only when ruby support is absent, ensuring the annotation is still readable inline.

---

### Real-World Cases

1. **Legacy browser support**: Ensuring readings remain readable in older or limited browsers.
2. **Email clients**: Some email clients do not support ruby; `<rp>` provides fallback.
3. **E-readers**: Older e-readers may lack ruby support; `<rp>` guarantees fallback readability.

---

### References

- MDN Web Docs — `<rp>`: The Ruby Fallback Parenthesis element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/rp
- W3C — HTML5: The `rp` element — https://dev.w3.org/html5/spec-author-view/the-rp-element.html

---

## 4. Ruby Annotation Best Practices

### Definitions

**Core Definition**
Ruby annotation best practices are guidelines for using `<ruby>`, `<rt>`, and `<rp>` correctly — choosing the appropriate annotation type, providing fallbacks, and ensuring accessibility for users who rely on pronunciation information.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG Technique H62, and East Asian typography guidance. They encompass using ruby for pronunciation where meaning depends on reading, providing `<rp>` fallbacks for non-supporting environments, using appropriate annotation granularity (mono-ruby vs. group ruby), and ensuring that ruby annotations are programmatically determinable for assistive technologies.

**Beginner-Friendly Explanation**
Best practices help you use ruby correctly: provide fallback parentheses for older browsers, use the right annotation style for the text, and make sure the pronunciation information is available to screen readers and other tools.

---

### Purposes

- **To ensure fallback readability** in browsers that do not support ruby.
- **To satisfy WCAG 3.1.6 (Pronunciation)** by making pronunciation information available.
- **To choose the appropriate annotation granularity** for the content.
- **To maintain semantic correctness** in ruby markup.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Provide `<rp>` fallbacks** | Wrap `<rt>` elements with `<rp>` parentheses for non-supporting browsers. |
| **Use mono-ruby for individual characters** | When each base character needs its own reading (e.g. kanji with individual furigana). |
| **Use group ruby for compound words** | When a single annotation applies to a multi-character base. |
| **Use double-sided ruby for multiple annotations** | When both pronunciation and meaning are needed. |
| **Ensure pronunciation information is accessible** | Ruby is a sufficient technique for WCAG 3.1.6. |
| **Do not use ruby for purely decorative purposes** | Ruby is for meaningful annotations, not visual effects. |

#### Syntax Rules

1. **Always provide `<rp>` fallbacks** when the content may be viewed in environments without ruby support.
2. **Choose the correct granularity**: mono-ruby for character-by-character readings; group ruby for whole-word annotations.
3. **Use `<rt>` for the annotation and `<rp>` for parentheses**; do not omit `<rt>`.
4. **Ensure the base text and annotation correspond** to the same linguistic content.

#### Constraints and Limitations

- **`<rb>` and `<rtc>` elements are not part of the WHATWG HTML Living Standard**; the standard uses only `<ruby>`, `<rt>`, and `<rp>`.
- **Complex ruby markup (tabular ruby) has limited support** and is not part of the core standard.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Complete Ruby Markup with Fallback

**Setup Guide**: Save as `ruby-best-practice.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Ruby Best Practices</title>
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

**Expected output**: In modern browsers, the readings appear above the characters with no parentheses. In non-supporting browsers, the readings appear in parentheses inline: “漢 (kan) 字 (ji) は日本語です。”

**Why this output occurs**: The `<rp>` elements provide the fallback parentheses, ensuring readability across environments. The `<rt>` elements carry the pronunciation information. This satisfies both visual presentation and accessibility requirements.

---

#### Example 2: WCAG-Compliant Pronunciation Annotation

**Setup Guide**: Save as `ruby-wcag.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WCAG Ruby</title>
</head>
<body>
  <p>
    When we talk about these guidelines, we often just call them
    <ruby>
      WCAG
      <rp>(</rp><rt>Wuh-KAG</rt><rp>)</rp>
    </ruby>.
  </p>
</body>
</html>
```

**Expected output**: The initialism “WCAG” has the pronunciation “Wuh-KAG” rendered as ruby text. In non-ruby browsers, it appears as “WCAG (Wuh-KAG)”.

**Why this output occurs**: This example follows WCAG Technique H62, using ruby to provide pronunciation information that makes the content accessible to users who can understand the language if pronunciation is provided.

---

### Real-World Cases

1. **Educational publishing**: Japanese textbooks with furigana on all kanji.
2. **Accessible web content**: Providing pronunciation for technical terms and initialisms.
3. **Bilingual dictionaries**: Annotations that provide translations alongside base text.

---

### References

- W3C — Technique H62: Using the ruby element — https://www.w3.org/WAI/WCAG22/Techniques/html/H62.html
- MDN Web Docs — `<ruby>`: The Ruby Annotation element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ruby
- W3C — HTML5: The `ruby` element — https://www.w3.org/TR/2012/WD-html5-author-20121025/the-ruby-element.html
- W3C — Requirements for Japanese Text Layout (JLREQ) — https://www.w3.org/TR/jlreq/

---

## Summary Table of Ruby Annotation Elements

| Element | Primary Role | Content Model | Required Parent |
|---|---|---|---|
| `<ruby>` | Annotation container | Base text + one or more `<rt>` (optionally with `<rp>`) | Any phrasing-content element |
| `<rt>` | Ruby text (annotation) | Phrasing content | `<ruby>` |
| `<rp>` | Fallback parentheses | Text (parenthesis characters) | `<ruby>` |

---

## Final Notes

- **Ruby is for pronunciation and meaning**: It is not a decorative styling tool; it conveys essential information for users who rely on readings.
- **`<rp>` fallbacks are best practice**: Even though modern browsers support ruby, `<rp>` ensures readability in all environments.
- **Choose the right granularity**: Mono-ruby for character-by-character annotations; group ruby for whole-word annotations.
- **`<rb>` and `<rtc>` are not in the WHATWG standard**: Use only `<ruby>`, `<rt>`, and `<rp>` for conforming markup.
- **Validate and test**: Test in both ruby-supporting and non-supporting environments to verify fallback behaviour.

---

This cheat sheet has covered ruby annotations — the `<ruby>`, `<rt>`, and `<rp>` elements — plus best practices, with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate ruby markup against the WHATWG HTML Living Standard and WCAG Technique H62.