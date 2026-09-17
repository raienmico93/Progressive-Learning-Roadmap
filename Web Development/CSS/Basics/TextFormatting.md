# CSS Text Formatting and Decoration: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
CSS text formatting and decoration properties control the visual presentation of text content, including its color, decorative lines, capitalization, shadows, and emphasis marks.

**Technical Definition**
According to the W3C CSS Text Decoration Module Level 3, these properties "describe ways of decorating text, such as underlines, overlines, and line-throughs, as well as other text-related effects like shadows and emphasis marks." The module defines properties for specifying the appearance of decorative lines, their color, style, and thickness, as well as properties for text shadows and emphasis marks used in East Asian typography. The `color` property is defined in the CSS Color Module and sets the foreground color of text and its decorations.

**Beginner-Friendly Explanation**
Think of CSS text properties as the "makeup kit" for your text. You can change its color, add lines under or through it, make it all uppercase or capitalize each word, give it a shadow for depth, or add small marks above characters (like the dots used in some Asian languages). These properties let you control exactly how text looks on a page.

---

### Key Characteristics

- **Color is foundational**: The `color` property sets the foreground color for text and its decorations, and also establishes the `currentColor` value that other properties can inherit.
- **Decoration inherits across descendants**: Text decorations are applied across all descendant text elements. A child element cannot remove a decoration applied by a parent, though it can add its own.
- **Shadow applies to decorations**: The `text-shadow` property applies to text and any of its text decorations, including underlines and overlines.
- **Emphasis marks are inherited**: Unlike `text-decoration`, the `text-emphasis` property is inherited, meaning descendants can change emphasis marks independently.
- **Baseline widely available**: Core properties like `color`, `text-decoration`, `text-transform`, and `text-shadow` have been available across browsers since July 2015. `text-emphasis` has been available since March 2022.
- **Language-aware transformations**: The `text-transform` property takes into account language-specific case mapping rules, such as the Turkish dotted/dotless "i" and the German "ß" becoming "SS" in uppercase.
- **Renaming and evolution**: The `text-decoration` shorthand now includes `text-decoration-thickness`, and properties like `text-underline-offset` and `text-decoration-skip-ink` provide fine-grained control over underline appearance.

---

### Prerequisites

Before learning CSS text formatting and decoration, you should understand:

1. **Basic CSS Syntax**: How properties and values are written in a rule set.
2. **CSS Selectors**: How to target elements for styling.
3. **CSS Color Values**: How to specify colors using hex, RGB, HSL, named colors, etc.
4. **CSS Units**: How to use length units like `px`, `em`, `rem`, and percentages.
5. **CSS Inheritance**: How properties pass from parent to child elements.

---

### Related Programming Areas

- **CSS Font Properties**: `font-family`, `font-size`, `font-weight`, and `font-style` control the typeface and size of text.
- **CSS Color Module**: Defines the `<color>` data type and color functions used by the `color` property.
- **CSS Text Module**: Defines text layout properties like `text-align`, `line-height`, and `letter-spacing`.
- **CSS Writing Modes**: Affects how text decorations and emphasis marks are positioned.
- **Accessibility**: Color contrast, text decoration removal, and emphasis marks all have accessibility implications.
- **Internationalization**: `text-transform` and `text-emphasis` have language-specific behaviors that are essential for multilingual content.

---

### Core Concepts / Features

The following core concepts are covered using the uniform structure required: **Definitions → Purposes → Syntax Rules and Structure → Multiple Annotated Code Examples → Real-World Cases**.

---

## 1. `color`

### Definitions

**Core Definition**
The `color` property sets the foreground color of an element's text and text decorations.

**Technical Definition**
According to MDN Web Docs, the `color` CSS property "sets the foreground color value of an element's text and text decorations, and sets the `currentcolor` value. `currentcolor` may be used as an indirect value on other properties and is the default for other color properties, such as `border-color`". The value must be a uniform color; gradients are not allowed because they are a type of `<image>`.

**Beginner-Friendly Explanation**
`color` is the most fundamental text property—it sets what color the text appears in. It also sets the "current color" that other properties like borders can automatically inherit from.

---

### Purposes

- **To set the foreground color of text** for readability and branding.
- **To establish the `currentColor` value** that other properties can reference automatically.
- **To create visual hierarchy** by differentiating text colors for headings, body text, and links.
- **To ensure sufficient contrast** between text and background for accessibility.
- **To support theming** by changing the color at the root level.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
color: <color>;
color: currentColor;
color: inherit;
color: initial;
color: unset;
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `<color>` | Any valid CSS color value (named color, hex, RGB, HSL, HWB, etc.) |
| `currentColor` | Uses the computed value of the `color` property from the parent |
| Global values | `inherit`, `initial`, `revert`, `revert-layer`, `unset` |

**Syntax Rules**

- The `color` property is inherited by default.
- When `currentColor` is used on the `color` property itself, it is treated as `inherit`.
- The value must be a single uniform color, not a gradient.
- The initial value is `CanvasText` (typically black in light mode).

**Constraints and Limitations**

- **Gradients are not allowed**: You cannot use a gradient as a `color` value.
- **No animation of `currentColor`**: The `currentColor` keyword cannot be animated between different color contexts.
- **Accessibility**: Low contrast between text color and background color can make content unreadable for users with visual impairments.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Basic Text Color and currentColor

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
    <title>Color Property Example</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <article class="card">
        <h2>Article Title</h2>
        <p>This is the article body text.</p>
        <a href="#">Read more</a>
    </article>
</body>
</html>
```

**CSS (`styles.css`)**

```css
/* Set the base text color on the card */
.card {
    color: #2c3e50;             /* Dark slate blue text */
    border: 2px solid;          /* Border color defaults to currentColor */
    padding: 16px;
    border-radius: 8px;
    background-color: #f8f9fa;
}

/* Heading inherits color from .card */
.card h2 {
    color: inherit;             /* Explicitly inherit from parent */
    margin-top: 0;
}

/* Link uses currentColor for its underline */
.card a {
    color: #e94560;             /* Override with a red-pink */
    text-decoration: underline wavy currentColor; /* Wavy underline matches link color */
}
```

**Expected Output**

- The card has dark slate blue text and a border that matches the text color (because `border-color` defaults to `currentColor`).
- The heading text is the same dark slate blue.
- The link is red-pink with a wavy underline in the same red-pink color.

**Why This Output Occurs**

The `.card` rule sets `color: #2c3e50`. Because `border-color` defaults to `currentColor`, the border uses the same dark slate blue. The heading explicitly inherits the color. The link overrides `color` to `#e94560`, and `text-decoration` uses `currentColor` to match the underline to the link color.

---

### Real-World Cases

**Design System Tokens**
A design system defines color tokens as CSS custom properties (`--color-primary`, `--color-text`) and applies them to the `color` property across components.

**Dark Mode Support**
Using `color-scheme` and `light-dark()`, a site can automatically switch text colors based on the user's preferred color scheme.

**Accessible Text**
Ensuring that `color` values meet WCAG contrast ratios (4.5:1 for body text, 3:1 for large text) makes content readable for users with visual impairments.

---

## 2. `text-decoration`

### Definitions

**Core Definition**
The `text-decoration` shorthand property sets the appearance of decorative lines on text, including underlines, overlines, and line-throughs.

**Technical Definition**
According to MDN Web Docs, the `text-decoration` shorthand CSS property "sets the appearance of decorative lines on text. It is a shorthand for `text-decoration-line`, `text-decoration-color`, `text-decoration-style`, and the newer `text-decoration-thickness` property". Text decorations are propagated to all descendant text elements; a child element cannot remove a decoration applied by an ancestor, though it can add its own.

**Beginner-Friendly Explanation**
`text-decoration` is how you add lines to text—underline, overline (line above), or line-through (strikethrough). You can also control the line's color, style (solid, dashed, dotted, wavy), and thickness.

---

### Purposes

- **To add underlines** to links or emphasized text.
- **To add strikethrough** to indicate deleted or completed content.
- **To add overlines** for decorative or mathematical notation.
- **To control the color, style, and thickness** of decorative lines for design flexibility.
- **To remove default decorations** (e.g., link underlines) using `text-decoration: none`.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
text-decoration: <text-decoration-line> || <text-decoration-style> || <text-decoration-color> || <text-decoration-thickness>;
```

**Breakdown**

| Component | Meaning | Values |
|-----------|---------|--------|
| `text-decoration-line` | Type of line | `none`, `underline`, `overline`, `line-through`, `blink` (deprecated) |
| `text-decoration-style` | Style of line | `solid`, `double`, `dotted`, `dashed`, `wavy` |
| `text-decoration-color` | Color of line | Any `<color>` value |
| `text-decoration-thickness` | Thickness of line | `auto`, `from-font`, `<length>`, `<percentage>` |

**Syntax Rules**

- Values can appear in any order.
- Multiple line values can be combined: `text-decoration: underline overline;` adds both.
- The shorthand resets any omitted sub-properties to their initial values.
- `text-decoration: none` removes all decorations.

**Constraints and Limitations**

- **Decoration cannot be removed by descendants**: If a parent has `text-decoration: underline`, a child cannot remove it with `text-decoration: none`.
- **`blink` is deprecated**: The `blink` value is deprecated and not supported in modern browsers.
- **Thickness and offset**: `text-decoration-thickness` and `text-underline-offset` are newer properties with varying browser support.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Common Decoration Styles

**HTML**

```html
<p class="underline">Underlined text</p>
<p class="overline">Overlined text</p>
<p class="line-through">Struck-through text</p>
<p class="wavy">Wavy underline</p>
<p class="none">No decoration (link)</p>
```

**CSS**

```css
.underline {
    text-decoration: underline;
}

.overline {
    text-decoration: overline;
}

.line-through {
    text-decoration: line-through;
}

.wavy {
    text-decoration: underline wavy #e94560;
}

.none {
    text-decoration: none;
    color: #0d6efd;
}
```

**Expected Output**

- "Underlined text" has a solid underline.
- "Overlined text" has a line above it.
- "Struck-through text" has a line through the middle.
- "Wavy underline" has a red-pink wavy underline.
- "No decoration (link)" has no underline.

**Why This Output Occurs**

Each class sets a different `text-decoration` value. The `wavy` class combines `underline`, `wavy`, and a color in a single declaration. The `none` class removes all decorations.

---

#### Example 2: Decoration Propagation

**HTML**

```html
<p class="parent-decoration">
    This paragraph has an underline.
    <em>This emphasized text cannot remove the underline.</em>
    <span class="own-decoration">This span adds an overline.</span>
</p>
```

**CSS**

```css
.parent-decoration {
    text-decoration: underline;
}

.parent-decoration em {
    text-decoration: none;      /* Has no effect — cannot remove parent's underline */
}

.own-decoration {
    text-decoration: overline;  /* Adds an overline on top of the parent's underline */
}
```

**Expected Output**

The entire paragraph has an underline. The `<em>` text still has the underline (the `none` has no effect). The `<span>` has both an underline (from the parent) and an overline (its own).

**Why This Output Occurs**

Text decorations propagate to all descendants and cannot be removed by a child. However, a child can add additional decorations. The `<em>` element's `text-decoration: none` does not remove the parent's underline because the decoration is drawn across the parent's entire content area.

---

### Real-World Cases

**Link Styling**
`a { text-decoration: none; }` removes the default underline, while `a:hover { text-decoration: underline; }` adds it back on hover for a common interactive pattern.

**Strikethrough Prices**
`.old-price { text-decoration: line-through; color: #999; }` indicates a discounted price.

**Accessible Links in Prose**
`text-decoration: underline; text-decoration-thickness: 2px; text-underline-offset: 3px;` creates clear, readable links that are distinguishable by more than just color.

**Grammar Error Indicators**
`text-decoration: underline wavy red;` mimics the wavy underline used by word processors to indicate spelling or grammar errors.

---

## 3. `text-transform`

### Definitions

**Core Definition**
The `text-transform` property controls the capitalization of text, allowing you to display text in uppercase, lowercase, or with each word capitalized.

**Technical Definition**
According to MDN Web Docs, the `text-transform` CSS property "specifies how to capitalize an element's text. It can be used to make text appear in all-uppercase or all-lowercase, or with each word capitalized. It also can help improve legibility for ruby". The property takes into account language-specific case mapping rules, such as the Turkish dotted/dotless "i" and the German "ß" becoming "SS" in uppercase.

**Beginner-Friendly Explanation**
`text-transform` changes how text is capitalized without changing the underlying HTML. You can make text all uppercase, all lowercase, or capitalize the first letter of each word.

---

### Purposes

- **To display text in all uppercase** for headings or labels.
- **To display text in all lowercase** for a stylistic effect.
- **To capitalize the first letter of each word** for titles or proper formatting.
- **To improve legibility** for ruby text in East Asian typography.
- **To handle language-specific capitalization rules** correctly.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
text-transform: none | capitalize | uppercase | lowercase | full-width | full-size-kana | math-auto;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `none` | No capitalization change (default) |
| `capitalize` | First letter of each word is uppercase |
| `uppercase` | All letters are uppercase |
| `lowercase` | All letters are lowercase |
| `full-width` | Converts to full-width forms (CJK) |
| `full-size-kana` | Converts small kana to full-size kana |
| `math-auto` | Automatic italicization for math |

**Syntax Rules**

- The property is inherited by default.
- Language-specific rules are applied based on the `lang` attribute.
- Support for language-specific cases varies between browsers.

**Constraints and Limitations**

- **Does not change the underlying data**: The text content in the HTML remains unchanged; only the visual rendering is affected.
- **Language-specific support varies**: Browser support for Turkish, German, Dutch, Greek, and Irish case rules varies.
- **`full-width` and `full-size-kana`**: These values are primarily for CJK typography.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Basic Transformations

**HTML**

```html
<p class="uppercase">this text will be uppercase</p>
<p class="lowercase">THIS TEXT WILL BE LOWERCASE</p>
<p class="capitalize">this text will be capitalized</p>
```

**CSS**

```css
.uppercase {
    text-transform: uppercase;
}

.lowercase {
    text-transform: lowercase;
}

.capitalize {
    text-transform: capitalize;
}
```

**Expected Output**

- "THIS TEXT WILL BE UPPERCASE"
- "this text will be lowercase"
- "This Text Will Be Capitalized"

**Why This Output Occurs**

Each class applies a different `text-transform` value. `uppercase` converts all letters to uppercase, `lowercase` converts all to lowercase, and `capitalize` converts the first letter of each word to uppercase.

---

#### Example 2: Language-Specific Transformations

**HTML**

```html
<p lang="tr" class="uppercase">istanbul</p>
<p lang="de" class="uppercase">straße</p>
```

**CSS**

```css
.uppercase {
    text-transform: uppercase;
}
```

**Expected Output**

- The Turkish text "istanbul" becomes "İSTANBUL" (with a dotted capital İ).
- The German text "straße" becomes "STRASSE" (with ß becoming SS).

**Why This Output Occurs**

The `lang` attribute tells the browser which language-specific case mapping rules to apply. Turkish has a dotted capital İ, and German ß becomes SS in uppercase. Browser support for these rules varies.

---

### Real-World Cases

**Headings and Labels**
`h1, h2, .label { text-transform: uppercase; }` creates consistent, impactful headings without typing in all caps.

**Navigation Menus**
`nav a { text-transform: uppercase; letter-spacing: 1px; }` gives navigation links a polished, structured look.

**Product Codes**
`.product-code { text-transform: uppercase; }` ensures product codes display consistently regardless of input.

**Stylistic Lowercase**
`.brand-name { text-transform: lowercase; }` creates a modern, minimalist brand aesthetic.

---

## 4. `text-shadow`

### Definitions

**Core Definition**
The `text-shadow` property adds one or more shadows to text, creating depth or decorative effects.

**Technical Definition**
According to MDN Web Docs, the `text-shadow` CSS property "adds shadows to text. It accepts a comma-separated list of shadows to be applied to the text and any of its text-decoration. Each shadow is described by some combination of X and Y offsets from the element, blur radius, and color". The first two `<length>` values are the `<offset-x>` and `<offset-y>`, the third optional value is the `<blur-radius>`, and the `<color>` value is optional.

**Beginner-Friendly Explanation**
`text-shadow` adds a shadow behind text, making it look like it is floating above the page. You can control the horizontal and vertical offset, how blurry the shadow is, and what color it is.

---

### Purposes

- **To add depth and dimension** to text.
- **To improve readability** of text over complex backgrounds.
- **To create decorative effects** like neon glows or retro shadows.
- **To add multiple layered shadows** for complex visual effects.
- **To create 3D text effects** using multiple offset shadows.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
text-shadow: <offset-x> <offset-y> <blur-radius>? <color>?;
text-shadow: none | <shadow>#;
```

**Breakdown**

| Component | Required? | Meaning |
|-----------|-----------|---------|
| `<offset-x>` | Required | Horizontal distance (negative = left) |
| `<offset-y>` | Required | Vertical distance (negative = up) |
| `<blur-radius>` | Optional | Blur amount (default 0) |
| `<color>` | Optional | Shadow color (default: user agent dependent) |

**Syntax Rules**

- At least two `<length>` values are required (offset-x and offset-y).
- The blur radius must be non-negative.
- Multiple shadows are comma-separated and applied front-to-back (first is on top).
- The `none` keyword removes all shadows.
- Applies to `::first-line` and `::first-letter` pseudo-elements.

**Constraints and Limitations**

- **Blur radius cannot be negative**: Negative blur values are invalid.
- **Color is user-agent dependent if omitted**: For consistency across browsers, always specify the color explicitly.
- **Performance**: Multiple large blur radii can impact rendering performance.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Basic Text Shadows

**HTML**

```html
<h1 class="simple-shadow">Simple Shadow</h1>
<h1 class="blur-shadow">Blurred Shadow</h1>
<h1 class="layered-shadow">Layered Shadow</h1>
```

**CSS**

```css
body {
    background-color: #f8f9fa;
    font-family: system-ui, sans-serif;
}

.simple-shadow {
    text-shadow: 2px 2px 0 #e94560;        /* Offset 2px right, 2px down, no blur */
}

.blur-shadow {
    text-shadow: 0 0 10px #0d6efd;          /* Centered, 10px blur, blue glow */
}

.layered-shadow {
    text-shadow:
        2px 2px 0 #155724,                  /* First shadow: dark green */
        4px 4px 0 #ffc107,                  /* Second shadow: amber */
        6px 6px 0 #e94560;                  /* Third shadow: red-pink */
}
```

**Expected Output**

- "Simple Shadow" has a sharp red-pink shadow offset to the right and down.
- "Blurred Shadow" has a blue glow surrounding the text.
- "Layered Shadow" has three stacked shadows in different colors, creating a retro 3D effect.

**Why This Output Occurs**

The `simple-shadow` uses two offset values and no blur, creating a sharp shadow. The `blur-shadow` uses `0 0` offsets (centered) with a 10px blur, creating a glow. The `layered-shadow` uses multiple comma-separated shadows, each with increasing offsets, creating a stacked effect.

---

#### Example 2: Text Shadow for Readability

**HTML**

```html
<div class="hero">
    <h1 class="hero-text">Text over a Busy Background</h1>
</div>
```

**CSS**

```css
.hero {
    background-image: url('https://mdn.github.io/shared-assets/images/examples/bubbles.png');
    background-size: cover;
    height: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.hero-text {
    color: white;
    font-size: 2.5rem;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5); /* Subtle shadow improves contrast */
}
```

**Expected Output**

The white text is readable over the busy bubble background because the dark shadow provides contrast.

**Why This Output Occurs**

The `text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5)` creates a semi-transparent dark shadow directly below the text. This shadow separates the text from the background, improving readability without being visually distracting.

---

### Real-World Cases

**Neon Text Effects**
`text-shadow: 0 0 5px #fff, 0 0 10px #fff, 0 0 20px #0d6efd;` creates a neon glow effect.

**Retro 3D Text**
`text-shadow: 3px 3px 0 #333, 6px 6px 0 #666;` creates a classic 3D layered effect.

**Readable Text Over Images**
A subtle `text-shadow` on hero text ensures readability over photographic backgrounds.

**Letterpress Effect**
`text-shadow: 0 1px 0 #fff;` on dark text creates a subtle letterpress or embossed effect.

---

## 5. `text-emphasis`

### Definitions

**Core Definition**
The `text-emphasis` property applies emphasis marks (such as dots or circles) above or beside text characters, commonly used in East Asian typography.

**Technical Definition**
According to MDN Web Docs, the `text-emphasis` CSS property "applies emphasis marks to text (except spaces and control characters). It is a shorthand for `text-emphasis-style` and `text-emphasis-color`". Unlike `text-decoration`, `text-emphasis` is inherited, meaning descendants can change emphasis marks independently. The size of the emphasis symbol is about 50% of the font size, and it may affect line height when the current leading is insufficient for the marks.

**Beginner-Friendly Explanation**
`text-emphasis` adds small marks—like dots, circles, or custom characters—above or beside text. This is similar to the "ruby" annotations used in Japanese and Chinese, where small characters are placed above or beside a main character for pronunciation or emphasis.

---

### Purposes

- **To add emphasis marks** to text characters in East Asian typography.
- **To create ruby-like annotations** for pronunciation or emphasis.
- **To use custom characters** as emphasis marks.
- **To control the color** of emphasis marks independently from the text color.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
text-emphasis: <text-emphasis-style> || <text-emphasis-color>;
```

**Breakdown**

| Component | Meaning | Values |
|-----------|---------|--------|
| `text-emphasis-style` | Shape and fill of the mark | `none`, `filled`, `open`, `dot`, `circle`, `double-circle`, `triangle`, `sesame`, `<string>` |
| `text-emphasis-color` | Color of the mark | Any `<color>` value (default: `currentColor`) |

**Syntax Rules**

- The `text-emphasis` shorthand does not reset `text-emphasis-position`; use that property separately to control the position of the marks.
- Emphasis marks are inherited by default.
- The size of the mark is about 50% of the font size.
- Custom characters can be specified as a string: `text-emphasis: "x";`.

**Constraints and Limitations**

- **Browser support**: `text-emphasis` has been available since March 2022, but custom string values may have varying support.
- **Position control**: The position of emphasis marks is controlled by `text-emphasis-position`, which is not part of the shorthand.
- **Line height impact**: Emphasis marks may increase line height if the current leading is insufficient.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Common Emphasis Marks

**HTML**

```html
<p class="dot-emphasis">Dot emphasis</p>
<p class="circle-emphasis">Circle emphasis</p>
<p class="sesame-emphasis">Sesame emphasis</p>
<p class="custom-emphasis">Custom emphasis</p>
```

**CSS**

```css
p {
    font-size: 24px;
    margin-bottom: 12px;
}

.dot-emphasis {
    text-emphasis: filled dot #e94560;
}

.circle-emphasis {
    text-emphasis: open circle #0d6efd;
}

.sesame-emphasis {
    text-emphasis: filled sesame #155724;
}

.custom-emphasis {
    text-emphasis: "★" #ffc107;
}
```

**Expected Output**

- "Dot emphasis" has filled red-pink dots above each character.
- "Circle emphasis" has open blue circles above each character.
- "Sesame emphasis" has filled green sesame seed shapes above each character.
- "Custom emphasis" has amber stars above each character.

**Why This Output Occurs**

Each class applies a different `text-emphasis` value. The `filled` and `open` keywords control whether the mark is solid or hollow. The shape keywords (`dot`, `circle`, `sesame`) determine the mark's form. The custom string `"★"` uses a star character as the mark.

---

#### Example 2: Emphasis Position

**HTML**

```html
<p class="over-emphasis">Over position (default)</p>
<p class="under-emphasis">Under position</p>
```

**CSS**

```css
.over-emphasis {
    text-emphasis: filled dot #e94560;
    text-emphasis-position: over right;
}

.under-emphasis {
    text-emphasis: filled dot #e94560;
    text-emphasis-position: under right;
}
```

**Expected Output**

- "Over position" has dots above the text.
- "Under position" has dots below the text.

**Why This Output Occurs**

The `text-emphasis-position` property controls where the marks are drawn. The default is `over right` for horizontal writing modes. Setting it to `under right` moves the marks below the text.

---

### Real-World Cases

**Japanese and Chinese Typography**
`text-emphasis: filled dot;` adds emphasis marks to Japanese text, mimicking the traditional use of dots (圏点) for emphasis.

**Pronunciation Guides**
Ruby-like annotations using `text-emphasis` can provide pronunciation hints for East Asian characters.

**Stylistic Emphasis**
Custom characters like `"★"` or `"✓"` can be used for creative emphasis in marketing or editorial content.

**Educational Content**
`text-emphasis: open circle;` can highlight key vocabulary in language learning materials.

---

## Constraints, Limitations, and Version-Specific Notes

- **`text-decoration` cannot be removed by descendants**: If a parent applies a text decoration, a child element cannot remove it with `text-decoration: none`. Only additional decorations can be added.
- **`blink` value is deprecated**: The `blink` value for `text-decoration-line` is deprecated and not supported in modern browsers.
- **`text-shadow` color is user-agent dependent if omitted**: For consistent cross-browser results, always specify the shadow color explicitly.
- **`text-transform` language support varies**: Browser support for Turkish, German, Dutch, Greek, and Irish case rules varies; check compatibility tables.
- **`text-emphasis` browser support**: Available since March 2022; custom string values may have varying support.
- **`text-emphasis-position` is not in the shorthand**: The `text-emphasis` shorthand does not reset `text-emphasis-position`.
- **`text-decoration-thickness` and `text-underline-offset`**: These are newer properties with varying browser support; they were added to Safe to Release exceptions in CSS Snapshot 2025.
- **`text-decoration-skip-ink`**: The `all` value for `text-decoration-skip-ink` became newly available in May 2026; it forces interruptions in underlines where the line would cross a glyph.
- **Gradients cannot be used as `color` values**: The `color` property accepts only uniform colors, not gradients.
- **Accessibility**: Removing underlines from links can harm accessibility if links are not distinguishable by other means; ensure sufficient color contrast and additional visual cues.

---

## References

- MDN Web Docs — `color` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/color
- MDN Web Docs — `text-decoration` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration
- MDN Web Docs — `text-decoration-color` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration-color
- MDN Web Docs — `text-decoration-thickness` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration-thickness
- MDN Web Docs — `text-decoration-skip-ink` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration-skip-ink
- MDN Web Docs — `text-transform` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-transform
- MDN Web Docs — `text-shadow` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow
- MDN Web Docs — `text-emphasis` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-emphasis
- MDN Web Docs — `text-emphasis-style` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-emphasis-style
- MDN Web Docs — `text-emphasis-position` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/text-emphasis-position
- W3C — CSS Text Decoration Module Level 3 - https://www.w3.org/TR/css-text-decor-3/
- W3C — CSS Text Decoration Module Level 4 - https://drafts.csswg.org/css-text-decor-4/
- W3C — CSS Text Module Level 3 - https://www.w3.org/TR/css-text-3/
- W3C — CSS Color Module Level 4 - https://www.w3.org/TR/css-color-4/
- MDN Web Docs — Fundamental Text and Font Styling - https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Text_styling/Fundamentals
- Web Platform Features Explorer — `text-decoration-skip-ink: all` - https://web-platform-dx.github.io/web-features-explorer/features/text-decoration-skip-ink-all/
- W3C CSS Working Group — CSS Snapshot 2025 Safe to Release Exceptions - https://lists.w3.org/Archives/Public/public-css-archive/2025Sep/0354.html