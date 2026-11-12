# Bidirectional Text — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Bidirectional text (bidi) refers to text that contains sequences of characters running in opposite directions — most commonly left-to-right (LTR) scripts such as Latin, and right-to-left (RTL) scripts such as Arabic or Hebrew. HTML provides three mechanisms for controlling bidirectional formatting: the `<bdi>` element for isolation, the `<bdo>` element for override, and the `dir` global attribute for setting base direction.

**Technical Definition**
The Unicode Bidirectional Algorithm (UBA) determines the display order of mixed-direction text. HTML's bidirectional mechanisms interact with the UBA as follows: the `dir` global attribute is an enumerated attribute that sets an element's **base direction** and **directionally isolates** its contents from surrounding text. The `<bdi>` element (Bidirectional Isolate) tells the browser's bidirectional algorithm to treat its contents in isolation from surrounding text, and is especially useful when dynamically inserting text of unknown directionality. The `<bdo>` element (Bidirectional Override) overrides the current directionality of text so that the text within is rendered in a different direction. The HTML specification encourages authors to use these markup mechanisms rather than manually inserting bidirectional formatting characters (such as U+202A–U+202E), because those characters interact poorly with CSS.

**Beginner-Friendly Explanation**
Imagine writing a sentence in English that includes an Arabic word. The browser has to figure out which way each part should read. Most of the time the browser gets it right on its own. But when it gets confused — especially with usernames, product codes, or text from a database — HTML gives you three tools to help: `dir` to tell the browser which way the text should generally run, `<bdi>` to protect a piece of text so it does not mess up the text around it, and `<bdo>` to force a piece of text to run in a specific direction.

---

### Key Characteristics

- **Three complementary mechanisms**: `dir` sets base direction and isolates; `<bdi>` isolates without requiring a known direction; `<bdo>` forces a direction override.
- **Isolation is the default for `dir`**: Modern HTML rendering rules apply `unicode-bidi: isolate` to elements with `dir`, and `isolate-override` to `<bdo dir>`, matching the behaviour already used by `<bdi>`.
- **`dir="auto"` detects direction from content**: The user agent determines direction from the first strong directional character, skipping `<bdi>`, `<script>`, `<style>`, `<textarea>`, and elements with valid `dir` attributes.
- **`<bdi>` always behaves as if `dir="auto"` is set**: It determines its direction from its content.
- **`<bdo>` requires a `dir` attribute**: Unlike other elements, `<bdo>` must have `dir` to function.
- **Markup is preferred over Unicode control characters**: The specification explicitly encourages using `dir`, `<bdo>`, and `<bdi>` rather than maintaining bidirectional formatting characters manually.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, attributes, and the DOM.
2. **The concept of text directionality**: understanding that not all languages read left-to-right.
3. **Phrasing vs. flow content**: understanding which elements can appear inside paragraphs.
4. **Unicode basics**: awareness that characters have inherent directionality properties.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Internationalisation (i18n)** | Bidirectional text is essential for supporting RTL languages such as Arabic, Hebrew, and Persian. |
| **Accessibility (WCAG)** | Correct directionality ensures screen readers pronounce and navigate text correctly. |
| **CSS** | `unicode-bidi` and `direction` properties provide complementary styling, but HTML `dir` is the semantic mechanism. |
| **User-generated content** | `<bdi>` and `dir="auto"` are critical for safely embedding text of unknown direction. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<bdi>`** — bidirectional isolation
2. **`<bdo>`** — bidirectional override
3. **`dir` attribute** — base direction and isolation
4. **Bidirectional text best practices** — authoring guidelines

---

## 1. `<bdi>` — The Bidirectional Isolate Element

### Definitions

**Core Definition**
The `<bdi>` element tells the browser's bidirectional algorithm to treat the text it contains in isolation from its surrounding text. It is especially useful when dynamically inserting text whose directionality is unknown.

**Technical Definition**
`<bdi>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. It supports global attributes, including `dir`, but even without `dir`, a `<bdi>` element determines its direction from its content as if `dir="auto"` were set. The element has an implicit ARIA role of no corresponding role. It is widely supported across all major browsers (Baseline widely available since January 2020).

**Beginner-Friendly Explanation**
`<bdi>` is a protective wrapper for a piece of text. It tells the browser: "Figure out this text's direction on its own, and do not let it affect the text around it." It is perfect for usernames, comments, or any text that comes from a database or user input, where you do not know whether the text will be left-to-right or right-to-left.

---

### Purposes

- **To isolate a span of text from its surroundings** for bidirectional formatting.
- **To safely embed user-generated content of unknown directionality** without breaking the surrounding layout.
- **To prevent a right-to-left phrase from reordering adjacent neutral characters** (spaces, punctuation, numbers).
- **To provide a semantic alternative to CSS-only isolation** that browsers can honour even when CSS is unavailable.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<bdi>User-generated text</bdi>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<bdi>` | The isolation wrapper. |
| Content | Phrasing content — the text to be isolated. |
| `dir` | Optional; `<bdi>` behaves as if `dir="auto"` regardless. |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **`<bdi>` isolates bidirectionally in both directions**: its directionality does not affect surrounding text, and surrounding text does not affect it.
3. **It determines direction from its content**, like `dir="auto"`.
4. **The `dir` attribute is permitted** but does not change the isolation behaviour.

#### Constraints and Limitations

- **`<bdi>` cannot resolve the "mismatched direction" problem by itself** when the surrounding text needs to know the isolate's direction; in those cases, `dir` on a tightly wrapped `<span>` with a known direction is more appropriate.
- **A `<bdi>` inside a paragraph does not create a block-level break**; it is an inline wrapper.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Usernames of Mixed Directionality

**Setup Guide**: Save as `bdi-usernames.html` and observe how each username renders correctly.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDI Usernames</title>
</head>
<body>
  <h1>World Wrestling Championships</h1>
  <ul>
    <!-- LTR usernames inside <bdi> -->
    <li><bdi class="name">Evil Steven</bdi>: 1st place</li>
    <li><bdi class="name">François fatale</bdi>: 2nd place</li>

    <!-- RTL username WITHOUT <bdi>: may break the layout -->
    <li><span class="name">سما</span>: 3rd place</li>

    <!-- RTL username WITH <bdi>: isolated correctly -->
    <li><bdi class="name">الرجل القوي إيان</bdi>: 4th place</li>

    <!-- Equivalent: span with dir="auto" -->
    <li><span class="name" dir="auto">سما</span>: 5th place</li>
  </ul>
</body>
</html>
```

**Expected output**: The usernames wrapped in `<bdi>` render in their correct direction without disrupting the “: N place” suffix. The un-wrapped RTL username may cause the “3rd” and the colon to be reordered incorrectly.

**Why this output occurs**: `<bdi>` isolates each username from the list marker and the “: N place” suffix. The RTL characters inside the isolate do not influence the direction of the neutral punctuation that follows. The `dir="auto"` span achieves the same visual result but with less semantic clarity.

---

#### Example 2: User Comments from a Database

**Setup Guide**: Save as `bdi-comments.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDI Comments</title>
</head>
<body>
  <p>
    New comment from
    <bdi>مرحبا</bdi>
    on your post.
  </p>
</body>
</html>
```

**Expected output**: The Arabic username “مرحبا” appears correctly, and the English phrase “on your post” is not reordered.

**Why this output occurs**: Without `<bdi>`, the Arabic text would influence the direction of the following English words and punctuation, potentially reversing the phrase. The `<bdi>` isolate prevents this cross-contamination.

---

### Real-World Cases

1. **User profiles and comments**: Usernames, display names, and comment bodies from international users.
2. **Product catalogs**: Product names or IDs that may contain RTL characters.
3. **Chat and messaging**: Messages in mixed languages where each message must render independently.

---

### References

- MDN Web Docs — `<bdi>`: The Bidirectional Isolate element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/bdi
- W3C — HTML 5.3: The `bdi` element — https://www.w3.org/TR/html53/semantics-embedded-content.html#the-bdi-element
- W3C i18n — Inline markup and bidirectional text in HTML — https://www.w3.org/International/articles/inline-bidi-markup/

---

## 2. `<bdo>` — The Bidirectional Override Element

### Definitions

**Core Definition**
The `<bdo>` element overrides the current directionality of text, so that the text within is rendered in a different direction.

**Technical Definition**
`<bdo>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. Unlike other elements, `<bdo>` **requires a `dir` attribute** to function; the `dir` attribute specifies the direction of the override (`ltr` or `rtl`). Modern rendering rules apply `unicode-bidi: isolate-override` to `<bdo dir>`, meaning the element both isolates its content and overrides the bidirectional algorithm. The individual characters' orientation is not affected; the text is drawn from the starting point in the given direction.

**Beginner-Friendly Explanation**
`<bdo>` forces text to run in a direction you specify, regardless of what the Unicode algorithm would normally do. It is like telling the browser: "No matter what, display this text right-to-left" (or left-to-right). It is useful for demonstrating reversed text or forcing a specific display for a controlled string.

---

### Purposes

- **To force a specific text direction** regardless of the content's inherent directionality.
- **To demonstrate or test bidirectional rendering** by reversing text.
- **To override the bidirectional algorithm** for strings that must appear in a fixed visual order.
- **To satisfy specific rendering requirements** where `dir="auto"` or isolation is insufficient.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<bdo dir="rtl">Text to reverse</bdo>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<bdo>` | The override wrapper. |
| `dir` | **Required.** `ltr` or `rtl`. |
| Content | Phrasing content. |

#### Syntax Rules

1. **The `dir` attribute is required** on `<bdo>`; without it, the element has no defined behaviour.
2. **`dir` accepts `ltr` or `rtl`**; `auto` is not meaningful for `<bdo>`.
3. **The override applies to the entire subtree** of the element.
4. **Characters are not mirrored**; they are drawn in the specified direction from the starting point.

#### Constraints and Limitations

- **`<bdo>` is a blunt instrument**: it overrides the algorithm entirely, which can produce unintelligible output if used carelessly.
- **It does not isolate by default in older implementations**, but modern rendering rules apply `isolate-override`.
- **It should not be used as a substitute for correct `dir` usage** when the direction is known.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reversing Text Direction

**Setup Guide**: Save as `bdo-reverse.html` and observe the visual reversal.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDO Reversal</title>
</head>
<body>
  <!-- Normal left-to-right text -->
  <p>This text will go left to right.</p>

  <!-- Forced right-to-left override -->
  <p><bdo dir="rtl">This text will go right to left.</bdo></p>
</body>
</html>
```

**Expected output**: The first sentence reads normally left-to-right. The second sentence is drawn right-to-left, with the period appearing at the left end.

**Why this output occurs**: `<bdo dir="rtl">` overrides the bidirectional algorithm and forces the text to be laid out from right to left. The characters themselves are not mirrored; only their order is reversed.

---

#### Example 2: Demonstrating an RTL Override Inside LTR Context

**Setup Guide**: Save as `bdo-context.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BDO Inside Context</title>
</head>
<body>
  <p>
    The English word
    <bdo dir="rtl">Hello</bdo>
    is displayed with an RTL override.
  </p>
</body>
</html>
```

**Expected output**: “Hello” is drawn right-to-left within the English sentence, so it appears as “olleH” visually.

**Why this output occurs**: The `<bdo dir="rtl">` forces the Latin characters to be ordered from right to left, producing a reversed visual string while leaving the surrounding text unaffected.

---

### Real-World Cases

1. **Demonstrating bidirectional behaviour**: Documentation and tutorials that show how direction overrides work.
2. **Forcing display order for fixed strings**: Serial numbers or codes that must appear in a specific visual order regardless of context.
3. **Testing bidi rendering**: Verifying that an application handles direction overrides correctly.

---

### References

- MDN Web Docs — `<bdo>`: The Bidirectional Text Override element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/bdo
- W3C — HTML 5.2: The `bdo` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-bdo-element

---

## 3. `dir` Attribute — Base Direction and Isolation

### Definitions

**Core Definition**
The `dir` global attribute is an enumerated attribute that sets the element's base text direction and directionally isolates its contents from surrounding text.

**Technical Definition**
`dir` accepts three values: `ltr` (left-to-right base direction), `rtl` (right-to-left base direction), and `auto` (user-agent-determined direction based on the first strong directional character). When `dir` is set, the element's contents are directionally isolated from the surrounding text. If `dir` is unspecified or invalid, direction is inherited from the parent element; if no ancestor sets a direction, the default is left-to-right. Exceptions include `<bdi>` (always behaves as `auto`) and `<input type="tel">` (always LTR). For `<textarea>` and `<pre>`, `dir="auto"` determines direction separately for each paragraph.

**Beginner-Friendly Explanation**
`dir` tells the browser which way the text should generally run. Use `dir="rtl"` on a page or section written in Arabic or Hebrew. Use `dir="auto"` when you do not know the direction — for example, for user comments — and let the browser figure it out from the first character.

---

### Purposes

- **To set the base direction** for an element and its descendants.
- **To directionally isolate** an element's contents from surrounding text.
- **To handle unknown directionality** automatically with `auto`.
- **To satisfy WCAG 1.3.2** (Meaningful Sequence) and related internationalisation requirements by providing programmatic directionality.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<element dir="ltr|rtl|auto">
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `ltr` | Left-to-right base direction. |
| `rtl` | Right-to-left base direction. |
| `auto` | User agent determines direction from the first strong directional character. |

#### Syntax Rules

1. **`dir` is an enumerated attribute** with the three values above.
2. **Setting `dir` isolates the element's contents** from surrounding text.
3. **`dir="auto"` skips `<bdi>`, `<script>`, `<style>`, `<textarea>`, and elements with valid `dir`** when determining direction.
4. **If `dir` is unspecified, direction is inherited** from the parent; the default is LTR.

#### Constraints and Limitations

- **Omitting `dir` does not enable automatic detection**; the direction is simply inherited.
- **`dir="auto"` should be used for unknown directionality**, not as a universal default.
- **`dir` on `<bdo>` must be `ltr` or `rtl`**, not `auto`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting Page-Level Direction

**Setup Guide**: Save as `dir-page.html`.

```html
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>RTL Page</title>
</head>
<body>
  <p>هذه فقرة باللغة العربية.</p>
  <p>This English paragraph inherits the RTL base direction but renders LTR characters correctly.</p>
</body>
</html>
```

**Expected output**: The page's base direction is right-to-left. Arabic text aligns and flows correctly; the English paragraph's characters run left-to-right, but the paragraph itself is positioned according to the RTL base direction.

**Why this output occurs**: `dir="rtl"` on `<html>` sets the base direction for the entire document and isolates it. Descendant English text still renders LTR because the characters are strongly LTR, but neutral elements (margins, alignment) follow the RTL base.

---

#### Example 2: Using `dir="auto"` for User Input

**Setup Guide**: Save as `dir-auto.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Dir Auto</title>
</head>
<body>
  <p>Comment: <span dir="auto">مرحبا بالعالم</span></p>
  <p>Comment: <span dir="auto">Hello world</span></p>
</body>
</html>
```

**Expected output**: The first comment renders right-to-left; the second renders left-to-right. Both are isolated from the surrounding English text.

**Why this output occurs**: `dir="auto"` detects the first strong directional character in each span and sets the direction accordingly, while isolating the span from the surrounding text.

---

### Real-World Cases

1. **RTL language sites**: Setting `dir="rtl"` on `<html>` for Arabic, Hebrew, or Persian sites.
2. **User-generated content**: Using `dir="auto"` on comments, usernames, and messages.
3. **Form inputs**: Setting `dir="auto"` on text inputs that accept mixed-direction input.

---

### References

- MDN Web Docs — `dir` global attribute — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/dir
- W3C — HTML 5.3: The `dir` attribute — https://www.w3.org/TR/html53/dom.html#the-dir-attribute

---

## 4. Bidirectional Text Best Practices

### Definitions

**Core Definition**
Bidirectional text best practices are guidelines for using `dir`, `<bdi>`, and `<bdo>` correctly — matching the mechanism to the situation, preferring markup over Unicode control characters, and ensuring accessibility and internationalisation.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, W3C i18n guidance, and accessibility standards. They encompass using `dir="auto"` and `<bdi>` for unknown directionality, using `dir` with a known value for tightly wrapped opposite-direction phrases, avoiding manual bidirectional formatting characters, and ensuring that directionality is programmatically determinable for assistive technologies.

**Beginner-Friendly Explanation**
Best practices help you choose the right tool: use `<bdi>` when you do not know the direction, use `dir` with a known value when you do, and avoid the old Unicode control characters because they cause problems with CSS.

---

### Purposes

- **To ensure correct rendering** of mixed-direction text.
- **To protect surrounding text** from direction contamination.
- **To satisfy internationalisation and accessibility requirements**.
- **To avoid fragile Unicode control characters** that interact poorly with CSS.

---

### Syntax Rules and Structure

| Situation | Recommended Mechanism |
|---|---|
| **Unknown directionality** (user input, database values) | `<bdi>` or `dir="auto"` on a tightly wrapped element. |
| **Known directionality** (e.g. a quoted Arabic phrase in English) | Tightly wrap the phrase in a `<span dir="rtl">` or similar element. |
| **Page-level direction** | `dir="rtl"` or `dir="ltr"` on `<html>`. |
| **Form inputs accepting mixed direction** | `dir="auto"` on the input. |
| **Forced direction override** (rare) | `<bdo dir="rtl">` or `<bdo dir="ltr">`. |
| **Unicode formatting characters** (U+202A–U+202E) | **Avoid**; use `dir`, `<bdi>`, and `<bdo>` instead. |

#### Syntax Rules

1. **Prefer markup over control characters**: Use `dir`, `<bdi>`, and `<bdo>` rather than U+202A–U+202E.
2. **Use `<bdi>` or `dir="auto"` for unknown directionality**: This is the safest default for user-generated content.
3. **Use `dir` with a known value for tightly wrapped opposite-direction phrases**: This allows surrounding text to flow correctly.
4. **Set `dir` on `<html>` for RTL pages**: This establishes the base direction for the entire document.
5. **Use `dir="auto"` on form inputs**: It handles mixed-direction user input gracefully.

#### Constraints and Limitations

- **`dir="auto"` is not a universal default**: It should only be used for content of unknown direction.
- **`<bdo>` should be used sparingly**: It overrides the algorithm entirely and can produce unintelligible results if misapplied.
- **Unicode control characters interact poorly with CSS**: The HTML specification explicitly discourages their manual use.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Known vs Unknown Direction

**Setup Guide**: Save as `best-practice-direction.html`.

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8">
  <title>Bidi Best Practices</title>
</head>
<body>
  <!-- KNOWN direction: tightly wrap the Arabic phrase -->
  <p>
    The Arabic word for peace is
    <span dir="rtl" lang="ar">سلام</span>
    and it is pronounced "salaam".
  </p>

  <!-- UNKNOWN direction: use <bdi> for user-generated content -->
  <p>
    Latest comment from
    <bdi>مستخدم</bdi>
    on your post.
  </p>

  <!-- UNKNOWN direction: dir="auto" on an input -->
  <label for="comment">Comment:</label>
  <input type="text" id="comment" name="comment" dir="auto">
</body>
</html>
```

**Expected output**: The known Arabic phrase is wrapped with `dir="rtl"` and renders correctly within the English sentence. The username in `<bdi>` is isolated and does not disturb the surrounding text. The input with `dir="auto"` detects the direction of the user's input.

**Why this output occurs**: The known-direction phrase uses `dir` with a specific value, allowing the surrounding text to flow correctly. The unknown-direction username uses `<bdi>` for safe isolation. The input uses `dir="auto"` for graceful handling of mixed-direction input.

---

#### Example 2: Avoiding Unicode Control Characters

**Setup Guide**: Save as `best-practice-no-control-chars.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>No Control Characters</title>
</head>
<body>
  <!-- GOOD: markup-based direction control -->
  <p>
    The title is
    <span dir="rtl">مدخل إلى C++</span>
    in Arabic.
  </p>

  <!-- BAD: manually inserted Unicode directional formatting characters -->
  <!--
  <p>
    The title is
    &#x202B;مدخل إلى C++&#x202C;
    in Arabic.
  </p>
  -->
</body>
</html>
```

**Expected output**: The markup-based version renders correctly and is maintainable. The control-character version may render but is fragile and interacts poorly with CSS.

**Why this output occurs**: The HTML specification encourages using `dir`, `<bdo>`, and `<bdi>` rather than maintaining bidirectional formatting characters manually, because those characters interact poorly with CSS.

---

### Real-World Cases

1. **Internationalised web applications**: Correct bidirectional handling is essential for applications serving RTL-language users.
2. **Social media platforms**: Usernames and comments from users worldwide require `<bdi>` or `dir="auto"` isolation.
3. **E-commerce**: Product names and descriptions in multiple languages must render correctly without breaking surrounding layout.

---

### References

- W3C i18n — Inline markup and bidirectional text in HTML — https://www.w3.org/International/articles/inline-bidi-markup/
- W3C — Internationalization Best Practices for Spec Developers — https://www.w3.org/TR/international-specs/
- WHATWG — HTML Living Standard: Bidirectional text — https://html.spec.whatwg.org/multipage/dom.html#bidirectional-text
- W3C — Strings on the Web: Language and Direction Metadata — https://www.w3.org/TR/string-meta/

---

## Summary Table of Bidirectional Text Mechanisms

| Mechanism | Primary Role | Direction Source | Use Case |
|---|---|---|---|
| **`dir="ltr"`** | Set LTR base direction and isolate | Author-specified | Known LTR content. |
| **`dir="rtl"`** | Set RTL base direction and isolate | Author-specified | Known RTL content (Arabic, Hebrew). |
| **`dir="auto"`** | Auto-detect direction and isolate | First strong directional character | Unknown direction (user input, database values). |
| **`<bdi>`** | Isolate without requiring a known direction | Content (as if `dir="auto"`) | Dynamic user-generated content. |
| **`<bdo dir="…">`** | Override direction entirely | Author-specified (`ltr` or `rtl`) | Forced display order, demonstrations. |

---

## Final Notes

- **Isolation is the default**: Modern HTML rendering applies isolation to elements with `dir` and `<bdi>`, preventing direction contamination.
- **`<bdi>` is the safest choice for unknown directionality**: It requires no configuration and always behaves as `dir="auto"`.
- **Avoid Unicode control characters**: The specification explicitly encourages markup-based direction control.
- **Test with real RTL content**: Bidirectional bugs are often subtle; test with Arabic, Hebrew, and mixed-direction strings.
- **Validate and inspect**: Use browser developer tools and accessibility checkers to verify that directionality is programmatically determinable.

---

This cheat sheet has covered bidirectional text in HTML — the `<bdi>` and `<bdo>` elements, the `dir` global attribute, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate bidirectional markup against the WHATWG HTML Living Standard, W3C i18n guidance, and accessibility requirements.