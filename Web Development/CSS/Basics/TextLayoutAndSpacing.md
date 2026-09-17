# CSS Spacing and Layout: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
CSS spacing and layout properties control the horizontal and vertical arrangement of text within block-level elements, including alignment, line spacing, character spacing, word spacing, indentation, whitespace handling, word wrapping, and text direction.

**Technical Definition**
According to the W3C CSS Text Module Level 3, these properties "define properties that control the layout of text, including line breaking, justification, alignment, and spacing". The module defines properties for controlling how text is aligned within its containing block, how lines are spaced vertically, how characters and words are spaced horizontally, how whitespace is collapsed or preserved, and how text direction is established. The `direction` property is defined in the CSS Writing Modes Module, which specifies the base direction of text and the direction of inline-level content.

**Beginner-Friendly Explanation**
Think of CSS spacing and layout properties as the "typesetting tools" for your text. Just as a newspaper editor decides whether text is left-aligned or justified, how much space goes between lines, and how words are spaced, these CSS properties give you the same control over text on a web page. They determine where text sits horizontally, how far apart lines are, how much space goes between letters and words, and what happens when text is too long to fit on one line.

---

### Key Characteristics

- **Most properties are inherited**: Properties like `text-align`, `line-height`, `letter-spacing`, `word-spacing`, `text-indent`, `white-space`, and `direction` are inherited by child elements.
- **Writing-mode aware**: Many of these properties adapt to vertical writing modes, where "horizontal" and "vertical" dimensions are swapped.
- **Interrelated behavior**: `white-space`, `word-break`, and `overflow-wrap` work together to control how text wraps and overflows.
- **Baseline widely available**: Core properties like `text-align`, `line-height`, `letter-spacing`, `word-spacing`, `text-indent`, `white-space`, and `direction` have been available across browsers since July 2015.
- **Newer properties**: `text-align-last` has been available since September 2022; `overflow-wrap` since October 2018; `word-break` since September 2015.
- **Shorthand relationships**: The `white-space` property is now a shorthand for `white-space-collapse` and `text-wrap-mode`.
- **Direction vs. `dir` attribute**: The `direction` CSS property is not inherited from table columns into table cells, unlike the HTML `dir` attribute.

---

### Prerequisites

Before learning CSS spacing and layout properties, you should understand:

1. **Basic CSS Syntax**: How properties and values are written in a rule set.
2. **CSS Selectors**: How to target elements for styling.
3. **CSS Units**: How to use length units like `px`, `em`, `rem`, and percentages.
4. **CSS Inheritance**: How properties pass from parent to child elements.
5. **The CSS Box Model**: How content, padding, border, and margin combine to form an element's total size.
6. **Block vs. Inline Elements**: How display types affect text layout.

---

### Related Programming Areas

- **CSS Font Properties**: `font-family`, `font-size`, and `font-weight` affect how text spacing properties are applied.
- **CSS Writing Modes**: `writing-mode` and `direction` together determine text flow.
- **CSS Text Decoration**: `text-decoration` and `text-emphasis` interact with text spacing.
- **Internationalization**: `direction`, `word-break`, and `text-align` have language-specific behaviors.
- **Accessibility**: Justified text, letter spacing, and line height all affect readability.
- **Responsive Typography**: `clamp()`, viewport units, and relative units are used with spacing properties for fluid layouts.

---

### Core Concepts / Features

The following core concepts are covered using the uniform structure required: **Definitions → Purposes → Syntax Rules and Structure → Multiple Annotated Complete Code Examples → Real-World Cases**.

---

## 1. `text-align`

### Definitions

**Core Definition**
The `text-align` property sets the horizontal alignment of inline-level content inside a block element or table-cell box.

**Technical Definition**
According to MDN Web Docs, the `text-align` CSS property "sets the horizontal alignment of the inline-level content inside a block element or table-cell box. This means it works like `vertical-align` but in the horizontal direction". It accepts keyword values including `start`, `end`, `left`, `right`, `center`, `justify`, `justify-all`, and `match-parent`, as well as a `<string>` value for character-based alignment.

**Beginner-Friendly Explanation**
`text-align` controls whether text is aligned to the left, right, center, or justified (stretched to fill the full width). It works like the alignment buttons in a word processor.

---

### Purposes

- **To align text to the left** for languages that read left-to-right.
- **To align text to the right** for right-to-left languages or stylistic effects.
- **To center text** within its container.
- **To justify text** so that both left and right edges are flush.
- **To align text based on the writing direction** using logical values `start` and `end`.
- **To align text to a specific character** (e.g., decimal points in numbers) using a string value.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
/* Keyword values */
text-align: start;
text-align: end;
text-align: left;
text-align: right;
text-align: center;
text-align: justify;
text-align: justify-all;   /* Experimental */
text-align: match-parent;

/* String value (character-based alignment) */
text-align: ".";

/* Global values */
text-align: inherit;
text-align: initial;
text-align: revert;
text-align: revert-layer;
text-align: unset;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `start` | Same as `left` if direction is left-to-right, `right` if right-to-left |
| `end` | Same as `right` if direction is left-to-right, `left` if right-to-left |
| `left` | Aligns inline content to the left edge of the line box |
| `right` | Aligns inline content to the right edge of the line box |
| `center` | Centers inline content within the line box |
| `justify` | Spaces text to line up left and right edges, except for the last line |
| `justify-all` | Same as `justify`, but also forces the last line to be justified |
| `match-parent` | Similar to `inherit`, but `start` and `end` are calculated according to the parent's `direction` |
| `<string>` | Aligns text to the specified character (e.g., `"."` for decimal alignment) |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `start`.
- The `justify-all` value is experimental and may not be supported in all browsers.
- The string value aligns all instances of the specified character in the line.

**Constraints and Limitations**

- **Justified text accessibility**: The inconsistent spacing between words created by justified text can be problematic for people with cognitive concerns such as dyslexia.
- **Last line with `justify`**: The last line of a justified block is not justified by default; use `text-align-last` to control it.
- **String alignment browser support**: The `<string>` value is experimental and has limited browser support.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Basic Alignment Values

**Setup Guide**

1. Create an HTML file named `index.html`.
2. Create a CSS file named `styles.css` and link it in the HTML `<head>`.
3. Paste the code below.

**HTML (`index.html`)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Text Align Example</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <p class="align-left">This text is left-aligned. The quick brown fox jumps over the lazy dog.</p>
    <p class="align-center">This text is center-aligned. The quick brown fox jumps over the lazy dog.</p>
    <p class="align-right">This text is right-aligned. The quick brown fox jumps over the lazy dog.</p>
    <p class="align-justify">This text is justified. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs. How vexingly quick daft zebras jump!</p>
</body>
</html>
```

**CSS (`styles.css`)**

```css
body {
    font-family: Georgia, serif;
    font-size: 16px;
    line-height: 1.5;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

.align-left {
    text-align: left;
}

.align-center {
    text-align: center;
}

.align-right {
    text-align: right;
}

.align-justify {
    text-align: justify;
}
```

**Expected Output**

- The first paragraph is flush with the left edge.
- The second paragraph is centered.
- The third paragraph is flush with the right edge.
- The fourth paragraph has flush left and right edges, with extra space distributed between words. The last line remains left-aligned.

**Why This Output Occurs**

Each class applies a different `text-align` value. `justify` stretches all lines except the last to fill the container width. The last line remains left-aligned by default, which is why `text-align-last` is needed to change that behavior.

---

### Real-World Cases

**Headings and Titles**
`h1 { text-align: center; }` centers page titles for a formal look.

**Article Body Text**
`article p { text-align: justify; }` creates clean, newspaper-like columns in long-form articles.

**Numeric Data in Tables**
`td.number { text-align: right; }` aligns numbers to the right for easier comparison in tables.

**RTL Language Support**
`p { text-align: start; }` automatically aligns text to the correct side based on the document's direction.

---

## 2. `line-height`

### Definitions

**Core Definition**
The `line-height` property sets the height of a line box in horizontal writing modes, controlling the vertical space between lines of text.

**Technical Definition**
According to MDN Web Docs, the `line-height` CSS property "sets the height of a line box in horizontal writing modes. In vertical writing modes, it sets the width of a line box". For block-level elements, it specifies the minimum height of line boxes within the element. For non-replaced inline elements, it specifies the height used in calculating the line box height.

**Beginner-Friendly Explanation**
`line-height` controls how much vertical space each line of text takes up. A larger value creates more space between lines (looser text), while a smaller value makes lines closer together (tighter text). It is one of the most important properties for readable text.

---

### Purposes

- **To control the vertical rhythm** of text for readability.
- **To increase space between lines** for a more airy, open feel.
- **To decrease space between lines** for a denser, more compact layout.
- **To vertically center single lines of text** in a container.
- **To create consistent spacing** across all text elements.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
line-height: normal;
line-height: <number>;
line-height: <length>;
line-height: <percentage>;
line-height: inherit;
line-height: initial;
line-height: unset;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `normal` | Depends on the user agent; typically around 1.2 |
| `<number>` | The used value is the number multiplied by the element's font size |
| `<length>` | A fixed length (e.g., `24px`, `1.5em`) |
| `<percentage>` | Relative to the element's own font size |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `normal`.
- Unitless numbers are recommended because they are inherited as numbers, not computed values, and recalculate based on each element's font size.
- Length and percentage values may produce unexpected results when inherited because they are inherited as computed values.

**Constraints and Limitations**

- **`em` units can produce unpredictable results**: A value like `1.5em` is inherited as a fixed computed value, which does not adapt to a child's different font size.
- **Percentage values similar issue**: Percentages are inherited as computed values, which can lead to unexpected results.
- **Accessibility**: Very tight line heights (less than 1.0) can make text unreadable, especially for users with dyslexia.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Comparing Line-Height Values

**HTML**

```html
<p class="tight">Tight: line-height: 1.1. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.</p>
<p class="normal-lh">Normal: line-height: 1.5. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.</p>
<p class="loose">Loose: line-height: 2. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.</p>
```

**CSS**

```css
body {
    font-family: Georgia, serif;
    font-size: 16px;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

.tight {
    line-height: 1.1;
    background-color: #f8d7da;
}

.normal-lh {
    line-height: 1.5;
    background-color: #d4edda;
}

.loose {
    line-height: 2;
    background-color: #cce5ff;
}
```

**Expected Output**

- The tight paragraph has lines close together.
- The normal paragraph has comfortable spacing.
- The loose paragraph has generous spacing between lines.

**Why This Output Occurs**

The unitless `line-height` value is multiplied by the element's font size to determine the line box height. With a 16px font size, `1.1` gives approximately 17.6px line height, `1.5` gives 24px, and `2` gives 32px.

---

### Real-World Cases

**Body Text**
`body { line-height: 1.6; }` is a common choice for readable body text, providing comfortable spacing between lines.

**Headings**
`h1, h2, h3 { line-height: 1.2; }` keeps headings compact and impactful.

**Buttons**
`button { line-height: 1; }` combined with padding vertically centers button text without extra spacing.

**Code Blocks**
`pre, code { line-height: 1.4; }` gives code a slightly tighter line height than prose, keeping code blocks compact.

---

## 3. `letter-spacing`

### Definitions

**Core Definition**
The `letter-spacing` property sets the horizontal spacing between text characters, also known as tracking.

**Technical Definition**
According to MDN Web Docs, the `letter-spacing` CSS property "sets the spacing between text characters. This value is added to the natural spacing between characters while rendering the text. Positive values of `letter-spacing` spread characters further apart, while negative values of `letter-spacing` bring characters closer together". It accepts `normal` or a `<length-percentage>` value.

**Beginner-Friendly Explanation**
`letter-spacing` controls how much space goes between each letter in your text. Positive values spread letters apart, creating a more open look, while negative values bring them closer together.

---

### Purposes

- **To spread letters apart** for a more open, airy typographic feel.
- **To bring letters closer together** for a tighter, more compact look.
- **To improve readability** of uppercase text, which often benefits from extra spacing.
- **To create stylistic effects** like wide tracking for headings.
- **To adjust text to fit a specific width** without changing font size.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
letter-spacing: normal;
letter-spacing: <length-percentage>;
letter-spacing: inherit;
letter-spacing: initial;
letter-spacing: unset;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `normal` | The natural letter spacing for the current font (default) |
| `<length>` | Extra space added between characters (e.g., `2px`, `0.1em`) |
| `<percentage>` | Percentage of the width of the space character |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `normal`.
- Negative values are allowed but may be constrained by implementation limits.
- When `letter-spacing` is non-zero, user agents do not apply optional ligatures.

**Constraints and Limitations**

- **Accessibility**: A large positive or negative `letter-spacing` value will make text unreadable. Letters can become so far apart they appear disconnected, or so close they overlap.
- **Internationalization**: Languages that use connected scripts (such as Arabic) should not have letter spacing applied, as it can break the visual connection between letters.
- **Ligature suppression**: Non-zero letter spacing disables optional ligatures, which may affect the appearance of certain character combinations.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Positive and Negative Letter-Spacing

**HTML**

```html
<p class="wide">WIDE LETTER SPACING</p>
<p class="normal">Normal Letter Spacing</p>
<p class="tight">Tight letter spacing</p>
```

**CSS**

```css
body {
    font-family: system-ui, sans-serif;
    font-size: 20px;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

.wide {
    letter-spacing: 4px;
    font-weight: bold;
    text-transform: uppercase;
}

.normal {
    letter-spacing: normal;
}

.tight {
    letter-spacing: -1px;
}
```

**Expected Output**

- "WIDE LETTER SPACING" has letters spread far apart.
- "Normal Letter Spacing" has default spacing.
- "Tight letter spacing" has letters closer together.

**Why This Output Occurs**

`letter-spacing: 4px` adds 4 pixels of space between each character. `letter-spacing: -1px` removes 1 pixel of space between characters. The `normal` value uses the font's natural spacing.

---

### Real-World Cases

**Uppercase Headings**
`h1 { letter-spacing: 2px; text-transform: uppercase; }` creates impactful, spaced-out headings.

**Navigation Menus**
`nav a { letter-spacing: 1px; }` adds subtle tracking to navigation links for a polished look.

**Small Text**
`small { letter-spacing: 0.5px; }` improves readability of small text by adding slight spacing.

**Tight Display Text**
`.logo { letter-spacing: -2px; }` creates a compact, modern logo treatment.

---

## 4. `word-spacing`

### Definitions

**Core Definition**
The `word-spacing` property sets the spacing between words and between tags.

**Technical Definition**
According to MDN Web Docs, the `word-spacing` CSS property "sets the spacing between words and between tags". It accepts `normal` or a `<length-percentage>` value. Percentage values are calculated relative to the font-size of the text.

**Beginner-Friendly Explanation**
`word-spacing` controls how much space goes between words. Positive values spread words apart, while negative values bring them closer together.

---

### Purposes

- **To increase space between words** for a more open, airy feel.
- **To decrease space between words** for a denser text block.
- **To improve readability** of certain fonts or at certain sizes.
- **To create stylistic effects** in headings or display text.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
word-spacing: normal;
word-spacing: <length-percentage>;
word-spacing: inherit;
word-spacing: initial;
word-spacing: unset;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `normal` | The normal inter-word spacing as defined by the font and browser (default) |
| `<length>` | Extra spacing added to the font's intrinsic inter-word spacing |
| `<percentage>` | Percentage of the font size |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `normal`.
- Negative values are allowed.
- Percentage values are calculated relative to the font-size of the text.

**Constraints and Limitations**

- **Accessibility**: A large positive or negative `word-spacing` value will make text unreadable. Words can become so far apart they no longer appear connected, or so close they overlap.
- **Legibility varies by font**: Different font families have different character widths, so legible word spacing must be determined on a case-by-case basis.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Word Spacing Variations

**HTML**

```html
<p class="wide-words">This text has wide word spacing. The quick brown fox jumps over the lazy dog.</p>
<p class="normal-words">This text has normal word spacing. The quick brown fox jumps over the lazy dog.</p>
<p class="tight-words">This text has tight word spacing. The quick brown fox jumps over the lazy dog.</p>
```

**CSS**

```css
body {
    font-family: Georgia, serif;
    font-size: 16px;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

.wide-words {
    word-spacing: 8px;
}

.normal-words {
    word-spacing: normal;
}

.tight-words {
    word-spacing: -2px;
}
```

**Expected Output**

- "Wide word spacing" has noticeable gaps between words.
- "Normal word spacing" has default spacing.
- "Tight word spacing" has words closer together.

**Why This Output Occurs**

`word-spacing: 8px` adds 8 pixels of space between each word. `word-spacing: -2px` removes 2 pixels of space between words. The `normal` value uses the font's natural word spacing.

---

### Real-World Cases

**Justified Text Adjustment**
In justified text, `word-spacing` can be adjusted to control the amount of space distributed between words.

**Display Headings**
`h1 { word-spacing: 4px; }` creates a more open, dramatic heading effect.

**Small Text Readability**
`small { word-spacing: 1px; }` slightly increases word spacing for better readability at small sizes.

**Text Justification Fine-Tuning**
When `text-align: justify` creates too much or too little space, `word-spacing` can help balance the text.

---

## 5. `text-indent`

### Definitions

**Core Definition**
The `text-indent` property sets the length of empty space (indentation) that is put before lines of text in a block.

**Technical Definition**
According to MDN Web Docs, the `text-indent` CSS property "sets the length of empty space (indentation) that is put before lines of text in a block". The indentation occurs at the inline-start edge of the content box. It accepts `<length>` and `<percentage>` values, as well as the keywords `each-line` and `hanging`.

**Beginner-Friendly Explanation**
`text-indent` indents the first line of a paragraph, like the indentation you see in books and newspapers. You can also use it to create a hanging indent, where all lines except the first are indented.

---

### Purposes

- **To indent the first line of a paragraph** for a traditional book-like appearance.
- **To create a hanging indent** where subsequent lines are indented.
- **To indent each line after a forced line break** using the `each-line` keyword.
- **To create visual separation** between paragraphs without using margins.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
text-indent: <length>;
text-indent: <percentage>;
text-indent: <length> each-line;
text-indent: <length> hanging;
text-indent: <length> hanging each-line;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `<length>` | Indentation as an absolute length (negative values allowed) |
| `<percentage>` | Indentation as a percentage of the container's inner inline size |
| `each-line` | Indentation affects the first line and each line after a forced line break |
| `hanging` | Inverts which lines are indented; all lines except the first are indented |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `0`.
- Negative values are allowed, which can create a "hanging" effect.
- The `hanging` and `each-line` keywords are experimental and have varying browser support.

**Constraints and Limitations**

- **Keyword support varies**: The `hanging` and `each-line` keywords are experimental and may not be supported in all browsers.
- **Negative values**: Negative `text-indent` can cause text to extend outside the content box.

---

### Multiple Annotated Complete Code Examples

#### Example 1: First-Line Indent and Hanging Indent

**HTML**

```html
<p class="first-line-indent">This paragraph has a first-line indent. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs. How vexingly quick daft zebras jump!</p>
<p class="hanging-indent">This paragraph has a hanging indent. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs. How vexingly quick daft zebras jump!</p>
```

**CSS**

```css
body {
    font-family: Georgia, serif;
    font-size: 16px;
    line-height: 1.5;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

.first-line-indent {
    text-indent: 2em;
}

.hanging-indent {
    text-indent: 2em hanging;
}
```

**Expected Output**

- The first paragraph's first line is indented by 2em, and subsequent lines start at the left edge.
- The second paragraph's first line starts at the left edge, and all subsequent lines are indented by 2em.

**Why This Output Occurs**

`text-indent: 2em` indents only the first line by 2em. `text-indent: 2em hanging` inverts the indentation, so the first line is not indented and all subsequent lines are indented by 2em.

---

### Real-World Cases

**Book-Style Paragraphs**
`p { text-indent: 1.5em; margin-bottom: 0; }` creates traditional book-style paragraphs with indented first lines and no space between paragraphs.

**Hanging Indents for Citations**
`.citation { text-indent: -2em; padding-left: 2em; }` creates a hanging indent commonly used in academic citations.

**Code Blocks**
`pre { text-indent: 0; }` ensures code blocks have no first-line indent.

**Poetry**
` .poem p { text-indent: 0; }` removes indentation for poetry where line breaks are meaningful.

---

## 6. `white-space`

### Definitions

**Core Definition**
The `white-space` property sets how whitespace inside an element is handled, including whether sequences of whitespace are collapsed and whether text wraps.

**Technical Definition**
According to MDN Web Docs, the `white-space` CSS property "sets how white space inside an element is handled". The property values can be specified as one or two keywords representing the values for the `white-space-collapse` and `text-wrap-mode` properties, or as special keywords like `normal`, `nowrap`, `pre`, `pre-wrap`, and `pre-line`.

**Beginner-Friendly Explanation**
`white-space` controls what happens to spaces, tabs, and line breaks in your HTML. By default, browsers collapse multiple spaces into one and wrap text to fit the container. `white-space` lets you change that behavior—preserving spaces, preventing wrapping, or handling line breaks differently.

---

### Purposes

- **To preserve whitespace** exactly as written in the HTML source using `pre`.
- **To prevent text from wrapping** using `nowrap`.
- **To preserve whitespace but allow wrapping** using `pre-wrap`.
- **To collapse whitespace but preserve line breaks** using `pre-line`.
- **To control text wrapping behavior** independently of whitespace collapsing.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
white-space: normal;
white-space: nowrap;
white-space: pre;
white-space: pre-wrap;
white-space: pre-line;
white-space: break-spaces;

/* Two-value syntax (shorthand for white-space-collapse and text-wrap-mode) */
white-space: collapse wrap;
white-space: preserve nowrap;
```

**Breakdown**

| Value | Whitespace Handling | Line Wrapping |
|-------|---------------------|---------------|
| `normal` | Collapsed | Wraps |
| `nowrap` | Collapsed | Does not wrap |
| `pre` | Preserved | Wraps only at newlines and `<br>` |
| `pre-wrap` | Preserved | Wraps at newlines, `<br>`, and as needed |
| `pre-line` | Collapsed | Wraps at newlines, `<br>`, and as needed |
| `break-spaces` | Preserved (including trailing spaces) | Wraps at every opportunity |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `normal`.
- The `white-space` property is now a shorthand for `white-space-collapse` and `text-wrap-mode`.
- The two-value syntax is newer and has varying browser support.

**Constraints and Limitations**

- **`pre` does not wrap**: Text with `white-space: pre` will overflow its container if it is too long, unless `overflow` is set.
- **`break-spaces` is newer**: The `break-spaces` value has less browser support than the traditional keywords.
- **Two-value syntax support**: The two-value syntax is relatively new and may not work in older browsers.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Comparing White-Space Values

**HTML**

```html
<div class="normal-ws">This    text   has   multiple   spaces.
It also has a line break.</div>
<div class="nowrap-ws">This    text   has   multiple   spaces but will not wrap at the container edge. It will overflow.</div>
<div class="pre-ws">This    text   has   multiple   spaces.
It also has a line break.</div>
<div class="pre-wrap-ws">This    text   has   multiple   spaces and will wrap at the container edge. It also has a line break.</div>
<div class="pre-line-ws">This    text   has   multiple   spaces collapsed.
It also has a line break preserved.</div>
```

**CSS**

```css
body {
    font-family: monospace;
    font-size: 14px;
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
}

div {
    border: 1px solid #ccc;
    margin-bottom: 8px;
    padding: 8px;
    background-color: #f8f9fa;
}

.normal-ws {
    white-space: normal;
}

.nowrap-ws {
    white-space: nowrap;
}

.pre-ws {
    white-space: pre;
}

.pre-wrap-ws {
    white-space: pre-wrap;
}

.pre-line-ws {
    white-space: pre-line;
}
```

**Expected Output**

- `normal`: Multiple spaces collapse into one, line breaks are ignored, and text wraps.
- `nowrap`: Multiple spaces collapse, line breaks are ignored, and text does not wrap (overflows).
- `pre`: Multiple spaces are preserved, line breaks are preserved, and text does not wrap.
- `pre-wrap`: Multiple spaces are preserved, line breaks are preserved, and text wraps.
- `pre-line`: Multiple spaces collapse, line breaks are preserved, and text wraps.

**Why This Output Occurs**

Each value controls two things: whether whitespace is collapsed and whether text wraps. `pre` preserves both spaces and line breaks but does not wrap. `pre-wrap` preserves both and also wraps. `pre-line` collapses spaces but preserves line breaks and wraps.

---

### Real-World Cases

**Code Blocks**
`pre { white-space: pre; }` preserves indentation and line breaks in code blocks.

**Preformatted Text**
`pre, code { white-space: pre-wrap; }` preserves formatting while still allowing wrapping for long lines.

**Poetry**
` .poem { white-space: pre-line; }` preserves line breaks while collapsing spaces.

**No-Wrap Labels**
`label { white-space: nowrap; }` prevents form labels from wrapping onto multiple lines.

**Chat Messages**
`.chat-message { white-space: pre-wrap; }` preserves line breaks and spacing in user messages while still wrapping long lines.

---

## 7. `word-break` and `overflow-wrap`

### Definitions

**Core Definition**
`word-break` and `overflow-wrap` control how long words or unbreakable strings wrap across lines when they would otherwise overflow their container.

**Technical Definition**
According to MDN Web Docs, the `word-break` CSS property "sets whether line breaks appear wherever the text would otherwise overflow its content box". The `overflow-wrap` CSS property "applies to text, setting whether the browser should insert line breaks within an otherwise unbreakable string to prevent text from overflowing its line box". `overflow-wrap` was originally a nonstandard Microsoft extension called `word-wrap`.

**Beginner-Friendly Explanation**
When a very long word or URL is too wide to fit in its container, these properties control what happens. `overflow-wrap: break-word` breaks the word only when necessary to prevent overflow. `word-break: break-all` breaks words at any character, even if they could fit on a line of their own.

---

### Purposes

- **To prevent long words from overflowing** their container using `overflow-wrap: break-word`.
- **To break words at any character** using `word-break: break-all`.
- **To prevent word breaks in CJK text** using `word-break: keep-all`.
- **To allow breaking anywhere** with `overflow-wrap: anywhere`.
- **To improve word breaks in Southeast Asian languages** using `word-break: auto-phrase`.

---

### Syntax Rules and Structure

**`word-break` Values**

| Value | Meaning |
|-------|---------|
| `normal` | Default line break rules |
| `break-all` | Break between any two characters (except CJK) |
| `keep-all` | Do not break CJK text; non-CJK behaves as `normal` |
| `auto-phrase` | Language-specific analysis to avoid breaking in the middle of phrases |
| `break-word` | Same as `overflow-wrap: anywhere` with `word-break: normal` |

**`overflow-wrap` Values**

| Value | Meaning |
|-------|---------|
| `normal` | Break only at normal word break points |
| `anywhere` | Break anywhere to prevent overflow; affects min-content size |
| `break-word` | Same as `anywhere` but does not affect min-content size |

**Syntax Rules**

- Both properties are inherited by default.
- `overflow-wrap` was formerly known as `word-wrap` and is still supported under that name as an alias.
- `word-break: break-word` is deprecated; use `overflow-wrap: anywhere` instead.
- `word-break: break-all` creates a break at the exact place where text would otherwise overflow, even if putting the entire word on its own line would negate the need for a break.

**Constraints and Limitations**

- **`word-break: break-word` is deprecated**: Use `overflow-wrap: anywhere` or `overflow-wrap: break-word` instead.
- **`word-break: auto-phrase` is newer**: This value has limited browser support.
- **`overflow-wrap` vs. `word-break`**: `overflow-wrap` only breaks when necessary; `word-break: break-all` breaks at any character regardless of necessity.
- **CJK text differences**: `word-break: keep-all` prevents breaks in Chinese, Japanese, and Korean text, which normally allows breaks between characters.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Comparing Word-Break and Overflow-Wrap

**HTML**

```html
<div class="container">
    <p class="normal-break">Normal: Honorificabilitudinitatibus califragilisticexpialidocious</p>
    <p class="break-all">break-all: Honorificabilitudinitatibus califragilisticexpialidocious</p>
    <p class="overflow-break">overflow-wrap: break-word: Honorificabilitudinitatibus califragilisticexpialidocious</p>
    <p class="overflow-anywhere">overflow-wrap: anywhere: Honorificabilitudinitatibus califragilisticexpialidocious</p>
</div>
```

**CSS**

```css
.container {
    width: 250px;
    border: 2px solid #333;
    padding: 10px;
    font-family: monospace;
    font-size: 14px;
}

.container p {
    background-color: #f8f9fa;
    border: 1px solid #ccc;
    padding: 8px;
    margin-bottom: 8px;
}

.normal-break {
    word-break: normal;
}

.break-all {
    word-break: break-all;
}

.overflow-break {
    overflow-wrap: break-word;
}

.overflow-anywhere {
    overflow-wrap: anywhere;
}
```

**Expected Output**

- `normal`: The long word overflows the container.
- `break-all`: The long word breaks at any character, filling the line completely.
- `overflow-wrap: break-word`: The long word breaks only when necessary to prevent overflow.
- `overflow-wrap: anywhere`: Similar to `break-word`, but the browser considers the word's minimum content size differently.

**Why This Output Occurs**

`word-break: break-all` inserts breaks between any two characters, so the word fills the available width completely. `overflow-wrap: break-word` only breaks the word when it cannot fit on a line by itself. `overflow-wrap: anywhere` is similar but affects how the browser calculates the minimum content size of the element.

---

### Real-World Cases

**URLs in Text**
`a { overflow-wrap: break-word; }` prevents long URLs from overflowing their container.

**Code Comments**
`pre code { overflow-wrap: break-word; }` allows long identifiers in code to wrap.

**User-Generated Content**
`.comment { overflow-wrap: anywhere; }` handles user input that may contain long unbreakable strings.

**CJK Typography**
`p { word-break: keep-all; }` prevents word breaks in Japanese, Chinese, and Korean text, which normally allow breaks between characters.

**Language-Specific Breaking**
`p { word-break: auto-phrase; }` uses language-specific analysis to improve word breaks in Southeast Asian languages.

---

## 8. `text-align-last`

### Definitions

**Core Definition**
The `text-align-last` property sets how the last line of a block or a line right before a forced line break is aligned.

**Technical Definition**
According to MDN Web Docs, the `text-align-last` CSS property "sets how the last line of a block or a line, right before a forced line break, is aligned". It is particularly useful when `text-align` is set to `justify`, as it controls the alignment of the otherwise left-aligned last line.

**Beginner-Friendly Explanation**
When you use `text-align: justify`, the last line of a paragraph is not justified by default—it is left-aligned. `text-align-last` lets you change that, so the last line can be centered, right-aligned, or even justified like the rest of the text.

---

### Purposes

- **To center the last line** of a justified paragraph.
- **To right-align the last line** of a justified paragraph.
- **To justify the last line** so it spans the full width like the other lines.
- **To control alignment after forced line breaks** (`<br>` elements).

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
text-align-last: auto;
text-align-last: start;
text-align-last: end;
text-align-last: left;
text-align-last: right;
text-align-last: center;
text-align-last: justify;
text-align-last: inherit;
text-align-last: initial;
text-align-last: unset;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `auto` | Aligned per `text-align`, unless `text-align` is `justify`, in which case it acts as `start` |
| `start` | Same as `left` if direction is left-to-right, `right` if right-to-left |
| `end` | Same as `right` if direction is left-to-right, `left` if right-to-left |
| `left` | Aligns to the left edge |
| `right` | Aligns to the right edge |
| `center` | Centers the last line |
| `justify` | Justifies the last line |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `auto`.
- The property has been available across browsers since September 2022.

**Constraints and Limitations**

- **Browser support is recent**: Available since September 2022; older browsers may not support it.
- **Only affects the last line**: It does not affect other lines in the block.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Justified Text with Different Last-Line Alignments

**HTML**

```html
<p class="last-center">This paragraph is justified with a centered last line. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.</p>
<p class="last-right">This paragraph is justified with a right-aligned last line. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.</p>
<p class="last-justify">This paragraph is justified with a justified last line. The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.</p>
```

**CSS**

```css
body {
    font-family: Georgia, serif;
    font-size: 16px;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

p {
    text-align: justify;
    background-color: #f8f9fa;
    padding: 8px;
    margin-bottom: 12px;
}

.last-center {
    text-align-last: center;
}

.last-right {
    text-align-last: right;
}

.last-justify {
    text-align-last: justify;
}
```

**Expected Output**

- The first paragraph's last line is centered.
- The second paragraph's last line is right-aligned.
- The third paragraph's last line is justified, filling the full width.

**Why This Output Occurs**

`text-align-last` overrides the default alignment of the last line. With `center`, the last line is centered. With `right`, it is right-aligned. With `justify`, it is stretched to fill the container width.

---

### Real-World Cases

**Justified Articles**
`article p { text-align: justify; text-align-last: left; }` ensures the last line of each paragraph is left-aligned, which is more readable than justifying the last line.

**Poetry**
` .poem p { text-align: left; text-align-last: center; }` centers the last line of each stanza.

**Formal Documents**
` .document p { text-align: justify; text-align-last: justify; }` creates a fully justified block with no ragged last line.

**Headings**
`h1 { text-align: center; text-align-last: center; }` ensures headings are centered even when they are the last line of a block.

---

## 9. `direction`

### Definitions

**Core Definition**
The `direction` property sets the direction of text, table and grid columns, and horizontal overflow.

**Technical Definition**
According to MDN Web Docs, the `direction` CSS property "sets the direction of text, table and grid columns, and horizontal overflow. Use `rtl` for languages written from right to left (like Hebrew or Arabic), and `ltr` for those written from left to right (like English and most other languages)". The property also sets the default alignment of text and the direction that cells flow within a table or grid row.

**Beginner-Friendly Explanation**
`direction` tells the browser which way text flows—left-to-right (like English) or right-to-left (like Arabic and Hebrew). This affects not just text alignment but also the order of table columns and the direction of horizontal overflow.

---

### Purposes

- **To set text direction for right-to-left languages** like Arabic and Hebrew.
- **To set text direction for left-to-right languages** like English and most others.
- **To control the direction of table columns** and grid cells.
- **To control the direction of horizontal overflow**.
- **To establish the base direction for bidirectional text**.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
direction: ltr;
direction: rtl;
direction: inherit;
direction: initial;
direction: unset;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `ltr` | Text and other elements go from left to right (default) |
| `rtl` | Text and other elements go from right to left |

**Syntax Rules**

- The property is inherited by default.
- The initial value is `ltr`.
- For the `direction` property to have any effect on inline-level elements, the `unicode-bidi` property's value must be `embed` or `override`.
- The `direction` and `unicode-bidi` properties are the only two properties not affected by the `all` shorthand property.
- The property is not inherited from table columns into table cells, unlike the HTML `dir` attribute.

**Constraints and Limitations**

- **Prefer the HTML `dir` attribute**: MDN recommends using the HTML `dir` attribute instead of the `direction` CSS property where possible.
- **Not inherited into table cells**: CSS inheritance follows the document tree, and table cells are inside rows but not columns.
- **`unicode-bidi` required for inline elements**: Without `unicode-bidi: embed` or `override`, `direction` has no effect on inline-level elements.

---

### Multiple Annotated Complete Code Examples

#### Example 1: LTR and RTL Text Direction

**HTML**

```html
<p class="ltr-text">This text is left-to-right. The quick brown fox jumps over the lazy dog.</p>
<p class="rtl-text" dir="rtl">This text is right-to-left. The quick brown fox jumps over the lazy dog.</p>
```

**CSS**

```css
body {
    font-family: Georgia, serif;
    font-size: 16px;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
}

.ltr-text {
    direction: ltr;
    text-align: left;
    background-color: #d4edda;
    padding: 8px;
    margin-bottom: 12px;
}

.rtl-text {
    direction: rtl;
    text-align: right;
    background-color: #f8d7da;
    padding: 8px;
    margin-bottom: 12px;
}
```

**Expected Output**

- The LTR text flows from left to right and is left-aligned.
- The RTL text flows from right to left and is right-aligned. Punctuation appears at the left edge of the text.

**Why This Output Occurs**

`direction: rtl` changes the base direction of the text to right-to-left. This affects the order of characters, the placement of punctuation, and the default text alignment. The `dir="rtl"` HTML attribute is used alongside the CSS property for clarity and accessibility.

---

### Real-World Cases

**Multilingual Websites**
`html[dir="rtl"] body { direction: rtl; }` sets the text direction for Arabic and Hebrew locales.

**Bidirectional Text**
` .bidi-isolate { direction: ltr; unicode-bidi: isolate; }` isolates a left-to-right phrase within a right-to-left paragraph.

**Table Column Direction**
`table { direction: rtl; }` reverses the order of table columns, which is useful for RTL languages.

**Form Inputs**
`input[dir="rtl"] { direction: rtl; text-align: right; }` ensures form inputs accept RTL text correctly.

---

## Constraints, Limitations, and Version-Specific Notes

- **Justified text accessibility**: The inconsistent spacing between words created by justified text can be problematic for people with cognitive concerns such as dyslexia.
- **Line-height unitless numbers are recommended**: Using unitless numbers for `line-height` avoids inheritance issues that can occur with `em` and percentage values.
- **Letter-spacing accessibility**: Large positive or negative `letter-spacing` values can make text unreadable; legible spacing must be determined on a case-by-case basis.
- **Letter-spacing and ligatures**: Non-zero `letter-spacing` disables optional ligatures, which may affect the appearance of certain character combinations.
- **Letter-spacing in connected scripts**: Languages that use connected scripts (such as Arabic) should not have letter spacing applied, as it can break the visual connection between letters.
- **Word-spacing accessibility**: Large positive or negative `word-spacing` values can make text unreadable.
- **`white-space: pre` does not wrap**: Text with `white-space: pre` will overflow its container if it is too long, unless `overflow` is set.
- **`word-break: break-word` is deprecated**: Use `overflow-wrap: anywhere` or `overflow-wrap: break-word` instead.
- **`word-break: auto-phrase` is newer**: This value has limited browser support.
- **`text-align-last` browser support**: Available since September 2022; older browsers may not support it.
- **`text-align: justify-all` is experimental**: This value has limited browser support.
- **`text-align` string value is experimental**: The `<string>` value has limited browser support.
- **Prefer HTML `dir` attribute over `direction`**: MDN recommends using the HTML `dir` attribute instead of the `direction` CSS property where possible.
- **`direction` not inherited into table cells**: CSS inheritance follows the document tree, so `direction` is not inherited from table columns into table cells.
- **`unicode-bidi` required for inline elements**: Without `unicode-bidi: embed` or `override`, `direction` has no effect on inline-level elements.
- **`text-indent` keywords are experimental**: The `hanging` and `each-line` keywords have varying browser support.
- **`white-space` two-value syntax is newer**: The two-value syntax for `white-space` has varying browser support.

---

## References

- MDN Web Docs — `text-align` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-align
- MDN Web Docs — `line-height` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/line-height
- MDN Web Docs — `letter-spacing` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/letter-spacing
- MDN Web Docs — `word-spacing` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/word-spacing
- MDN Web Docs — `text-indent` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-indent
- MDN Web Docs — `white-space` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/white-space
- MDN Web Docs — `word-break` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/word-break
- MDN Web Docs — `overflow-wrap` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap
- MDN Web Docs — `text-align-last` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-align-last
- MDN Web Docs — `direction` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/direction
- W3C — CSS Text Module Level 3 - https://www.w3.org/TR/css-text-3/
- W3C — CSS Writing Modes Module Level 4 - https://www.w3.org/TR/css-writing-modes-4/
- W3C — CSS Text Decoration Module Level 3 - https://www.w3.org/TR/css-text-decor-3/
- MDN Web Docs — Fundamental Text and Font Styling - https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Text_styling/Fundamentals
- MDN Web Docs — `unicode-bidi` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/unicode-bidi
- MDN Web Docs — HTML `dir` Global Attribute - https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/dir

<!--
* Advanced and Modern Typography
   * writing-mode: Changes text flow direction to vertical or horizontal.
   * text-orientation: Sets character orientation inside vertical text.
   * hyphens: Automatically breaks words with hyphens (auto or manual).
   * Variable Fonts (font-variation-settings): Fine-tunes custom font axes like weight, width, and slant dynamically.
   * Custom bullets with CSS ::marker: Targets list markers directly to style custom bullets, numbers, and symbols. 
-->