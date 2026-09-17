# CSS Font Properties: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
CSS font properties control the typographic appearance of text, including the typeface used, its size, weight, style, and other stylistic variations.

**Technical Definition**
According to the W3C CSS Fonts Module Level 4, this specification "defines modifications to the existing CSS Fonts 3 specification along with additional features for font feature control, font variations, and chromatic (color) fonts". The module defines properties that select and configure font resources, control font synthesis when a desired font face is unavailable, and enable access to advanced OpenType features. Font properties are applied to elements and inherited by descendants unless overridden.

**Beginner-Friendly Explanation**
Think of CSS font properties as the "styling tools" for your text. Just as a word processor lets you choose a typeface, make text bold or italic, and adjust its size, CSS font properties give you the same control over text on a web page. The `font-family` picks the typeface, `font-size` sets how big the text is, `font-weight` controls boldness, `font-style` handles italics, and so on. The `font` shorthand lets you set several of these at once.

---

### Key Characteristics

- **Inherited properties**: Most font properties are inherited by child elements, meaning you can set a base font on the `<body>` and all descendants will use it unless overridden.
- **Shorthand property**: The `font` shorthand combines `font-style`, `font-variant`, `font-weight`, `font-stretch`, `font-size`, `line-height`, and `font-family` into a single declaration.
- **Variable font support**: Modern font properties support variable fonts, which allow fine-grained control over weight, width, slant, and other axes.
- **Fallback mechanisms**: Font properties use fallback lists and synthesis algorithms to ensure text remains readable even when the desired font is unavailable.
- **Baseline widely available**: Core font properties have been available across browsers since July 2015.
- **Renaming in progress**: The `font-stretch` property is being renamed to `font-width` in the CSS Fonts specification, with `font-stretch` retained as a legacy alias for compatibility.

---

### Prerequisites

Before learning CSS font properties, you should understand:

1. **Basic CSS Syntax**: How properties and values are written in a rule set.
2. **CSS Selectors**: How to target elements for styling.
3. **CSS Units**: How to use length units like `px`, `em`, `rem`, and percentages.
4. **CSS Inheritance**: How properties pass from parent to child elements.
5. **Typography Basics**: Understanding of typefaces, font families, and typographic terminology.

---

### Related Programming Areas

- **CSS Text Properties**: `text-align`, `text-decoration`, `line-height`, and `letter-spacing`.
- **CSS Color**: Font color and text rendering.
- **Web Fonts**: `@font-face` for loading custom fonts.
- **Variable Fonts**: Modern font technology that provides continuous variation along multiple axes.
- **OpenType Features**: Advanced typographic features controlled by `font-variant` and `font-feature-settings`.
- **Responsive Typography**: Fluid font sizing using `clamp()`, viewport units, and relative units.

---

### Core Concepts / Features

The following core concepts are covered using the uniform structure required: **Definitions → Purposes → Syntax Rules and Structure → Multiple Annotated Code Examples → Real-World Cases**.

---

## 1. `font-family`

### Definitions

**Core Definition**
The `font-family` property specifies a prioritized list of font family names for the selected element.

**Technical Definition**
According to MDN Web Docs, the `font-family` CSS property "allows you to specify a prioritized list of font family names or generic family names for the selected element". The property accepts a comma-separated list of `<family-name>` and `<generic-name>` values. The browser selects the first font in the list that is available on the user's system or can be downloaded via `@font-face`.

**Beginner-Friendly Explanation**
`font-family` is how you tell the browser which typeface to use for your text. You provide a list of fonts in order of preference—if the first one is not available, the browser tries the next, and so on. It is always a good idea to end the list with a generic family like `sans-serif` or `serif` as a last resort.

---

### Purposes

- **To specify the typeface used for text** on an element.
- **To provide fallback fonts** in case the preferred font is unavailable.
- **To ensure readability** by including a generic family as the final fallback.
- **To support custom web fonts** loaded via `@font-face`.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font-family: <family-name> [, <family-name>]* | <generic-name>;
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `<family-name>` | The name of a specific font family (e.g., `'Open Sans'`, `Arial`) |
| `<generic-name>` | A generic fallback (e.g., `serif`, `sans-serif`, `monospace`, `cursive`, `fantasy`, `system-ui`) |

**Syntax Rules**

- Font family names containing spaces must be quoted (e.g., `'Open Sans'`).
- Family names should be separated by commas.
- Generic family names are keywords and should not be quoted.
- The list is evaluated in order; the first available font is used.
- The browser may use font synthesis or fallback for individual characters if the chosen font lacks them.

**Constraints and Limitations**

- **System font availability varies**: A font available on one user's system may not be available on another's.
- **Generic families are not specific**: `sans-serif` resolves to different fonts on different operating systems.
- **Quoting is recommended for names with spaces or special characters**.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Basic Font Stack

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
    <title>Font Family Example</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Heading with Custom Font Stack</h1>
    <p>This paragraph uses the same font stack.</p>
</body>
</html>
```

**CSS (`styles.css`)**

```css
body {
    /* Font stack: try 'Open Sans', then Helvetica Neue, then Arial, finally sans-serif */
    font-family: 'Open Sans', 'Helvetica Neue', Arial, sans-serif;
    font-size: 16px;
    line-height: 1.6;
    color: #333;
}

h1 {
    /* Headings use a serif font stack */
    font-family: Georgia, 'Times New Roman', serif;
    color: #1a1a2e;
}
```

**Expected Output**

The body text uses Open Sans if available, falling back to Helvetica Neue, Arial, or the system's default sans-serif font. The heading uses Georgia if available, falling back to Times New Roman or a generic serif font.

**Why This Output Occurs**

The browser evaluates the font-family list from left to right. The first available font on the user's system is selected. If none of the specified fonts are available, the generic family (`sans-serif` or `serif`) is used as the final fallback.

---

### Real-World Cases

**Brand Consistency**
A company might specify its brand font first, with common system fonts as fallbacks, ensuring the closest possible match regardless of the user's system.

**Reading Optimization**
`font-family: Georgia, 'Times New Roman', serif;` for long-form articles leverages serif fonts, which are often considered more readable in print-like contexts.

**Code Blocks**
`font-family: 'Fira Code', 'Cascadia Code', Consolas, monospace;` provides a monospaced font stack for code, with popular developer fonts prioritized.

**System UI Fonts**
`font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;` matches the operating system's native UI font for a seamless native feel.

---

## 2. `font-size`

### Definitions

**Core Definition**
The `font-size` property sets the size of the font, which determines the size of the em unit and other relative length units.

**Technical Definition**
According to MDN Web Docs, the `font-size` CSS property "sets the size of the font (the size of the capital 'M')". Changing the font size also updates the sizes of other length units calculated with `em` and `ex` units. It can be specified using absolute-size keywords, relative-size keywords, `<length>` values, or `<percentage>` values.

**Beginner-Friendly Explanation**
`font-size` controls how big or small the text appears. You can use keywords like `small` or `large`, fixed units like `16px`, or relative units like `1.5em` (1.5 times the parent's font size) or `1rem` (relative to the root font size).

---

### Purposes

- **To set the size of text** for readability and visual hierarchy.
- **To establish a typographic scale** using relative units.
- **To create responsive typography** using viewport units or `clamp()`.
- **To control the size of em-based spacing** derived from font size.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font-size: xx-small | x-small | small | medium | large | x-large | xx-large | xxx-large;
font-size: larger | smaller;
font-size: <length>;
font-size: <percentage>;
font-size: math;  /* For MathML */
```

**Breakdown**

| Value Type | Example | Meaning |
|------------|---------|---------|
| Absolute keyword | `medium` | Maps to a browser-determined size (default 16px) |
| Relative keyword | `larger` | Scales relative to the parent's font size |
| `<length>` | `16px`, `1.5rem`, `2em` | Fixed or relative length |
| `<percentage>` | `120%` | Percentage of the parent's font size |

**Syntax Rules**

- `<length>` values must be positive.
- `em` units are relative to the parent element's font size; `rem` units are relative to the root element's font size.
- `<percentage>` values are relative to the parent element's font size.
- The default font size for most browsers is `16px` for the root element.

**Constraints and Limitations**

- **Negative values are invalid**: Font sizes cannot be negative.
- **Accessibility**: Using fixed pixel sizes can prevent users from resizing text; relative units (`rem`, `em`, `%`) are more accessible.
- **Compounding with `em`**: Nested `em` values compound, which can lead to unexpected sizes.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Relative Font Sizing

**HTML**

```html
<div class="parent">
    <p>Parent text (1rem = 16px)</p>
    <div class="child">
        <p>Child text (1.5em = 24px)</p>
        <div class="grandchild">
            <p>Grandchild text (1.5em = 36px)</p>
        </div>
    </div>
</div>
```

**CSS**

```css
html {
    font-size: 16px;            /* Root font size */
}

.parent {
    font-size: 1rem;            /* 16px */
    background-color: #f8f9fa;
    padding: 16px;
}

.child {
    font-size: 1.5em;           /* 1.5 × 16px = 24px */
    background-color: #e9ecef;
    padding: 16px;
    margin-top: 8px;
}

.grandchild {
    font-size: 1.5em;           /* 1.5 × 24px = 36px */
    background-color: #dee2e6;
    padding: 16px;
    margin-top: 8px;
}
```

**Expected Output**

The parent text is 16px, the child text is 24px, and the grandchild text is 36px. Each level compounds the `em` value based on its parent's computed font size.

**Why This Output Occurs**

`em` units are relative to the parent element's computed font size. The child's `1.5em` is calculated from the parent's 16px, resulting in 24px. The grandchild's `1.5em` is calculated from the child's 24px, resulting in 36px. This compounding behavior is important to understand when using `em` units for nested elements.

---

#### Example 2: Responsive Font Size with `clamp()`

**HTML**

```html
<h1 class="responsive-heading">Responsive Heading</h1>
```

**CSS**

```css
.responsive-heading {
    /* clamp(minimum, preferred, maximum) */
    font-size: clamp(1.5rem, 4vw, 3rem);
    font-family: system-ui, sans-serif;
    color: #1a1a2e;
}
```

**Expected Output**

The heading's font size scales with the viewport width. It is at least 1.5rem, grows with 4vw of the viewport width, and is capped at 3rem.

**Why This Output Occurs**

`clamp()` returns the preferred value (`4vw`) as long as it is between the minimum (`1.5rem`) and maximum (`3rem`). On narrow screens, the heading is at least 1.5rem; on wide screens, it is capped at 3rem.

---

### Real-World Cases

**Fluid Typography**
`font-size: clamp(1rem, 2.5vw, 1.5rem);` creates text that scales smoothly with the viewport, improving readability across devices.

**Accessible Base Size**
`html { font-size: 100%; }` respects the user's browser font size settings, unlike setting a fixed pixel size.

**Typographic Scale**
`h1 { font-size: 2.5rem; }`, `h2 { font-size: 2rem; }`, etc., create a consistent hierarchy using rem units.

**Code Blocks**
`pre, code { font-size: 0.9em; }` slightly reduces code text size relative to surrounding text.

---

## 3. `font-weight`

### Definitions

**Core Definition**
The `font-weight` property sets the weight (boldness) of the font.

**Technical Definition**
According to MDN Web Docs, the `font-weight` CSS property "sets the weight (or boldness) of the font. The weights available depend on the `font-family` that is currently set". It accepts numeric values between 1 and 1000, as well as the keywords `normal` (400), `bold` (700), `lighter`, and `bolder`.

**Beginner-Friendly Explanation**
`font-weight` controls how bold or light the text appears. You can use `normal` for regular text, `bold` for bold text, or a number from 100 to 900 to access different weights if the font supports them. Variable fonts can use any number from 1 to 1000.

---

### Purposes

- **To emphasize text** using bold or heavier weights.
- **To de-emphasize text** using lighter weights.
- **To create typographic hierarchy** by varying weight across headings and body text.
- **To access multiple weights** from variable fonts.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font-weight: normal | bold | lighter | bolder | <number>;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `normal` | Normal weight (equivalent to 400) |
| `bold` | Bold weight (equivalent to 700) |
| `lighter` | One relative weight lighter than the parent |
| `bolder` | One relative weight heavier than the parent |
| `<number>` | Any value from 1 to 1000 (variable fonts); for non-variable fonts, values divisible by 100 map to common weight names |

**Common Weight Name Mapping**

| Numeric Value | Common Name |
|---------------|-------------|
| 100 | Thin / Hairline |
| 200 | Extra Light / Ultra Light |
| 300 | Light |
| 400 | Normal / Regular |
| 500 | Medium |
| 600 | Semi Bold / Demi Bold |
| 700 | Bold |
| 800 | Extra Bold / Ultra Bold |
| 900 | Black / Heavy |

**Syntax Rules**

- The `lighter` and `bolder` keywords are calculated relative to the parent's font weight, considering only four weights: 100, 400, 700, and 900.
- For non-variable fonts, if the exact weight is unavailable, a fallback algorithm selects the closest available weight.
- Variable fonts support any numeric value from 1 to 1000.

**Constraints and Limitations**

- **Font support varies**: Not all fonts offer all weights; the browser uses fallback rules to select the closest available weight.
- **Relative keywords are limited**: `lighter` and `bolder` only consider four weight levels, which can lead to unexpected results.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Weight Hierarchy

**HTML**

```html
<h1 class="black-weight">Black (900)</h1>
<h2 class="bold-weight">Bold (700)</h2>
<p class="regular-weight">Regular (400)</p>
<p class="light-weight">Light (300)</p>
<p class="thin-weight">Thin (100)</p>
```

**CSS**

```css
body {
    font-family: 'Inter', system-ui, sans-serif;
}

.black-weight {
    font-weight: 900;
}

.bold-weight {
    font-weight: 700;
}

.regular-weight {
    font-weight: 400;
}

.light-weight {
    font-weight: 300;
}

.thin-weight {
    font-weight: 100;
}
```

**Expected Output**

Each heading and paragraph displays at its specified weight, creating a clear typographic hierarchy. If the font does not support a weight, the browser selects the closest available.

**Why This Output Occurs**

The `font-weight` property requests a specific weight from the font family. If the font is a variable font, all weights are available. If it is a static font with limited weights, the browser's fallback algorithm selects the closest match.

---

#### Example 2: Variable Font Weight

**HTML**

```html
<p class="variable-weight">Variable font weight: 550</p>
```

**CSS**

```css
.variable-weight {
    font-family: 'Inter Variable', 'Inter', sans-serif;
    font-weight: 550;           /* Custom weight available in variable fonts */
}
```

**Expected Output**

If the variable font is loaded and supports weight 550, the text renders at that exact weight.

**Why This Output Occurs**

Variable fonts support a continuous range of weights. Unlike static fonts, which may only offer 400 and 700, variable fonts allow any value within their supported range, providing precise typographic control.

---

### Real-World Cases

**Design Systems**
A design system might define `--font-weight-light: 300`, `--font-weight-regular: 400`, `--font-weight-medium: 500`, and `--font-weight-bold: 700` as tokens.

**Emphasis in Prose**
`strong { font-weight: 600; }` provides a subtler emphasis than the browser default `bold` (700).

**Headings**
`h1, h2, h3 { font-weight: 800; }` creates bold, impactful headings.

**Variable Font Animation**
`font-weight` can be animated when using variable fonts, creating smooth weight transitions on hover.

---

## 4. `font-style`

### Definitions

**Core Definition**
The `font-style` property sets whether text is rendered in a normal, italic, or oblique face.

**Technical Definition**
The `font-style` CSS property sets the font style, which can be `normal`, `italic`, or `oblique`. The `oblique` value optionally accepts an angle to specify the slant. Italic and oblique are distinct: italic uses a separate font face designed with cursive characteristics, while oblique is a slanted version of the normal face.

**Beginner-Friendly Explanation**
`font-style` is how you make text italic. You can use `normal` for upright text, `italic` for true italics (which often have different letter shapes), or `oblique` for a slanted version of the normal font.

---

### Purposes

- **To emphasize text** with italics or obliques.
- **To distinguish quotations**, titles, and foreign words.
- **To create typographic contrast** in headings or pull quotes.
- **To access true italic faces** when available.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font-style: normal;
font-style: italic;
font-style: oblique;
font-style: oblique <angle>;   /* e.g., oblique 15deg */
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `normal` | Upright text (default) |
| `italic` | True italic face if available, otherwise oblique |
| `oblique` | Slanted version of the normal face |
| `oblique <angle>` | Slanted at a specific angle (variable fonts) |

**Syntax Rules**

- The `italic` keyword selects a true italic face if the font provides one; otherwise, the browser may synthesize an oblique.
- The `oblique` keyword selects a slanted face; an optional angle can specify the exact slant.
- Variable fonts can use any angle between `-90deg` and `90deg` for oblique.

**Constraints and Limitations**

- **Synthesis**: If the font does not have a true italic face, the browser may synthesize an oblique by slanting the normal face, which can look less refined.
- **Angle support**: The `oblique <angle>` syntax is only supported by variable fonts.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Normal, Italic, and Oblique

**HTML**

```html
<p class="normal-style">This is normal text.</p>
<p class="italic-style">This is italic text.</p>
<p class="oblique-style">This is oblique text.</p>
```

**CSS**

```css
body {
    font-family: Georgia, serif;
    font-size: 18px;
}

.normal-style {
    font-style: normal;
}

.italic-style {
    font-style: italic;
}

.oblique-style {
    font-style: oblique;
}
```

**Expected Output**

The first paragraph is upright, the second uses the italic face of Georgia, and the third uses an oblique (slanted) version of the normal face.

**Why This Output Occurs**

`italic` selects the font's designed italic face, which has distinct letterforms. `oblique` slants the normal face without changing the letterforms. If Georgia has a true italic, the difference is visible.

---

#### Example 2: Custom Oblique Angle

**HTML**

```html
<p class="custom-oblique">Custom oblique: 20 degrees</p>
```

**CSS**

```css
.custom-oblique {
    font-family: 'Recursive', sans-serif;
    font-style: oblique 20deg;  /* Variable font with slant axis */
}
```

**Expected Output**

If the variable font supports a slant axis, the text is slanted at exactly 20 degrees.

**Why This Output Occurs**

Variable fonts can expose a slant axis (`slnt`), allowing precise control over the oblique angle. The `oblique <angle>` syntax accesses this axis.

---

### Real-World Cases

**Blockquotes**
`blockquote { font-style: italic; }` is a classic way to distinguish quoted material.

**Book Titles**
`cite { font-style: italic; }` renders titles of books, films, and other works in italics.

**Emphasis**
`em { font-style: italic; }` provides semantic emphasis, though the browser default already applies italic.

**Variable Font Slant**
Variable fonts can animate `font-style: oblique` from 0 to a negative angle for interactive effects.

---

## 5. `font-variant`

### Definitions

**Core Definition**
The `font-variant` property is a shorthand that controls typographic variants such as small-caps, ligatures, and stylistic alternates.

**Technical Definition**
According to MDN Web Docs, the `font-variant` CSS shorthand property "allows you to set all the font variants for a font. You can also set the `<font-variant-css2>` values of `font-variant` defined in CSS Level 2.1, (that is, `normal` or `small-caps`), by using the `font` shorthand". It is a shorthand for `font-variant-caps`, `font-variant-numeric`, `font-variant-ligatures`, `font-variant-east-asian`, `font-variant-position`, and other sub-properties.

**Beginner-Friendly Explanation**
`font-variant` lets you access special glyph variations in a font, like small-caps (where lowercase letters appear as smaller capitals), old-style figures, or ligatures (where certain letter combinations like "fi" are combined into a single glyph).

---

### Purposes

- **To display text in small-caps** for stylistic or typographic effect.
- **To enable or disable ligatures** for improved readability or aesthetics.
- **To access numeric variants** like old-style figures or tabular figures.
- **To use stylistic alternates** provided by the font designer.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font-variant: normal | none | [ <common-lig-values> || <discretionary-lig-values> || ... ];
```

**Common Values**

| Value | Meaning |
|-------|---------|
| `normal` | Resets all variant sub-properties to normal |
| `none` | Disables ligatures and resets other sub-properties |
| `small-caps` | Renders lowercase letters as small capitals |
| `all-small-caps` | Renders all letters as small capitals |
| `petite-caps` | Renders as petite capitals |
| `unicase` | Mixed uppercase and lowercase forms |
| `tabular-nums` | Monospaced (tabular) figures |
| `oldstyle-nums` | Old-style figures |
| `common-ligatures` | Enables common ligatures (on by default) |
| `no-common-ligatures` | Disables common ligatures |

**Syntax Rules**

- The `font-variant` shorthand can set multiple sub-properties at once.
- The `font` shorthand only supports `normal` and `small-caps` for `font-variant`.
- Sub-properties like `font-variant-caps`, `font-variant-numeric`, and `font-variant-ligatures` provide more granular control.

**Constraints and Limitations**

- **Font support varies**: Not all fonts support all variant features; unsupported variants are ignored.
- **Synthesis**: Small-caps may be synthesized by the browser if the font lacks a true small-caps face.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Small-Caps

**HTML**

```html
<p class="small-caps">This text uses small-caps.</p>
<p class="normal-caps">This text is normal.</p>
```

**CSS**

```css
.small-caps {
    font-variant: small-caps;
    font-size: 20px;
}

.normal-caps {
    font-size: 20px;
}
```

**Expected Output**

The first paragraph renders lowercase letters as smaller capitals, while uppercase letters remain full size. The second paragraph is normal.

**Why This Output Occurs**

`font-variant: small-caps` activates the font's small-caps feature. If the font does not have a true small-caps face, the browser may synthesize it by scaling down uppercase letters.

---

#### Example 2: Numeric Variants

**HTML**

```html
<p class="oldstyle">Old-style figures: 1234567890</p>
<p class="tabular">Tabular figures: 1234567890</p>
```

**CSS**

```css
.oldstyle {
    font-variant-numeric: oldstyle-nums;
}

.tabular {
    font-variant-numeric: tabular-nums;
}
```

**Expected Output**

The old-style figures have varying heights (like lowercase letters), while tabular figures are monospaced and align vertically.

**Why This Output Occurs**

`oldstyle-nums` activates the font's old-style figure feature, where numbers have descenders and varying heights. `tabular-nums` activates monospaced figures, useful for tables where numbers need to align.

---

### Real-World Cases

**Small-Caps Headings**
`h2 { font-variant: small-caps; }` creates elegant section headings without using all uppercase, which can be harder to read.

**Tabular Data**
`table { font-variant-numeric: tabular-nums; }` ensures numbers align in table columns.

**Code Ligatures**
`code { font-variant-ligatures: common-ligatures; }` enables ligatures in programming fonts like Fira Code.

**Old-Style Figures in Prose**
`body { font-variant-numeric: oldstyle-nums; }` gives body text a classic, book-like appearance.

---

## 6. `font-stretch` (Renamed to `font-width`)

### Definitions

**Core Definition**
The `font-stretch` property selects a normal, condensed, or expanded face from a font family.

**Technical Definition**
According to MDN Web Docs, the `font-stretch` CSS property "selects a normal, condensed, or expanded face from a font". Note that the property was renamed to `font-width` in the CSS Fonts specification, with `font-stretch` retained as a legacy alias for compatibility. It accepts keyword values from `ultra-condensed` to `ultra-expanded`, as well as percentage values from 50% to 200%.

**Beginner-Friendly Explanation**
`font-stretch` (or `font-width`) lets you choose a narrower or wider version of a font, if the font family provides such variations. It does not stretch the letters geometrically—it selects a different designed face.

---

### Purposes

- **To select condensed or expanded font faces** for space-constrained or expansive layouts.
- **To create typographic variety** within a font family.
- **To access variable font width axes** for precise control.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font-stretch: ultra-condensed | extra-condensed | condensed | semi-condensed | normal | semi-expanded | expanded | extra-expanded | ultra-expanded | <percentage>;
```

**Keyword to Percentage Mapping**

| Keyword | Percentage |
|---------|------------|
| `ultra-condensed` | 50% |
| `extra-condensed` | 62.5% |
| `condensed` | 75% |
| `semi-condensed` | 87.5% |
| `normal` | 100% |
| `semi-expanded` | 112.5% |
| `expanded` | 125% |
| `extra-expanded` | 150% |
| `ultra-expanded` | 200% |

**Syntax Rules**

- Percentage values must be between 50% and 200% inclusive.
- Negative values are invalid.
- If the font family does not have the requested width face, the browser selects the closest available face.

**Constraints and Limitations**

- **Font support is limited**: Many font families do not offer condensed or expanded faces.
- **Renaming in progress**: The property is being renamed to `font-width`; use `font-stretch` for current compatibility.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Condensed and Expanded

**HTML**

```html
<p class="condensed">Condensed text (75%)</p>
<p class="normal">Normal text (100%)</p>
<p class="expanded">Expanded text (125%)</p>
```

**CSS**

```css
body {
    font-family: 'Roboto', sans-serif;
    font-size: 18px;
}

.condensed {
    font-stretch: condensed;    /* Or font-width: 75% */
}

.normal {
    font-stretch: normal;
}

.expanded {
    font-stretch: expanded;     /* Or font-width: 125% */
}
```

**Expected Output**

The condensed text is narrower, the expanded text is wider, and the normal text is standard width—provided the font family has these faces.

**Why This Output Occurs**

`font-stretch` selects the corresponding width face from the font family. If the font is a variable font with a width axis, the browser interpolates to the requested width.

---

### Real-World Cases

**Space-Constrained Layouts**
`font-stretch: condensed;` fits more text into narrow columns without reducing font size.

**Variable Font Width Animation**
`font-width` can be animated in variable fonts for dynamic text effects.

**Headline Impact**
`font-stretch: expanded;` creates wide, impactful headlines.

---

## 7. The `font` Shorthand Property

### Definitions

**Core Definition**
The `font` shorthand property sets multiple font-related properties in a single declaration.

**Technical Definition**
According to MDN Web Docs, the `font` CSS shorthand property "sets all the different properties of an element's font" and can also set the element's font to a system font. It is a shorthand for `font-style`, `font-variant`, `font-weight`, `font-stretch`, `font-size`, `line-height`, and `font-family`.

**Beginner-Friendly Explanation**
The `font` shorthand is a time-saver. Instead of writing five or six separate lines for style, weight, size, and family, you can write one line. For example: `font: italic bold 16px/1.5 Arial, sans-serif;`.

---

### Purposes

- **To set multiple font properties in a single declaration**.
- **To reduce repetition** in stylesheets.
- **To quickly reset all font properties** to a known state.
- **To use system fonts** with a single keyword.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
font: [ <font-style> || <font-variant-css2> || <font-weight> || <font-stretch> ]? <font-size> [ / <line-height> ]? <font-family>;
```

**System Font Keywords**

```css
font: caption | icon | menu | message-box | small-caption | status-bar;
```

**Breakdown**

| Component | Required? | Meaning |
|-----------|-----------|---------|
| `font-style` | Optional | `normal`, `italic`, `oblique` |
| `font-variant-css2` | Optional | `normal` or `small-caps` |
| `font-weight` | Optional | `normal`, `bold`, or numeric weight |
| `font-stretch` | Optional | Width keywords or percentage |
| `font-size` | **Required** | Size of the font |
| `line-height` | Optional | Line height, preceded by `/` |
| `font-family` | **Required** | Font family list |

**Syntax Rules**

- `font-style`, `font-variant`, and `font-weight` must come before `font-size`.
- `font-size` and `font-family` are the only **required** values.
- `line-height` follows `font-size` separated by `/`.
- `font-family` must be specified last.
- Any omitted property is set to its initial value, overriding previously set values.
- System font keywords must be used alone (no other values).

**Constraints and Limitations**

- **Resets omitted properties**: The shorthand resets any font property not specified to its initial value, which can override previous declarations.
- **Only CSS 2.1 variant values**: The shorthand only supports `normal` and `small-caps` for `font-variant`; other variant values must be set separately.
- **`font-size` and `font-family` are mandatory**: The shorthand is invalid without both.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Complete Font Shorthand

**HTML**

```html
<p class="shorthand-font">This paragraph uses the font shorthand.</p>
```

**CSS**

```css
.shorthand-font {
    /* style weight size/line-height family */
    font: italic bold 18px/1.6 'Open Sans', Arial, sans-serif;
}
```

**Expected Output**

The paragraph is italic, bold, 18px in size, with a line-height of 1.6, using the Open Sans font stack.

**Why This Output Occurs**

The `font` shorthand combines all specified values into a single declaration. The browser parses the values in order: `italic` for style, `bold` for weight, `18px` for size, `1.6` for line-height, and the font stack for family.

---

#### Example 2: System Font

**HTML**

```html
<button class="system-btn">System Font Button</button>
```

**CSS**

```css
.system-btn {
    font: menu;                 /* Use the system menu font */
    padding: 8px 16px;
    border: 1px solid #ccc;
    border-radius: 4px;
}
```

**Expected Output**

The button uses the operating system's menu font, which varies by platform (e.g., San Francisco on macOS, Segoe UI on Windows).

**Why This Output Occurs**

The `menu` keyword is a system font keyword that resolves to the font used for menus in the user's operating system. This provides a native look and feel.

---

#### Example 3: Shorthand Reset Pitfall

**HTML**

```html
<p class="reset-demo">This text may not appear as expected.</p>
```

**CSS**

```css
.reset-demo {
    font-weight: bold;          /* Set bold first */
    font: 16px Arial, sans-serif; /* Shorthand resets weight to normal */
}
```

**Expected Output**

The text is **not bold**, because the `font` shorthand resets `font-weight` to its initial value (`normal`).

**Why This Output Occurs**

The `font` shorthand resets all font properties not explicitly specified to their initial values. Since `font-weight` is omitted from the shorthand, it is reset to `normal`, overriding the earlier `font-weight: bold` declaration.

---

### Real-World Cases

**Base Typography**
`body { font: 16px/1.6 system-ui, sans-serif; }` sets the base font in one line.

**System UI Components**
`button { font: menu; }` and `input { font: icon; }` create native-looking form controls.

**Reset Styles**
`* { font: inherit; }` resets all font properties to inherit from the parent, useful for form controls that do not inherit fonts by default.

**Component Fonts**
`.card-title { font: 600 1.25rem/1.3 'Inter', sans-serif; }` sets the complete font for a component in one declaration.

---

## Constraints, Limitations, and Version-Specific Notes

- **Font availability**: All font properties depend on the fonts available on the user's system or loaded via `@font-face`. If a specified font is unavailable, the browser uses fallback rules.
- **Synthesis**: When a font lacks a required face (e.g., italic, bold, small-caps), the browser may synthesize it, which can produce less refined results.
- **`font-stretch` renaming**: The property is being renamed to `font-width` in the CSS Fonts specification; `font-stretch` is retained as a legacy alias.
- **Shorthand resets**: The `font` shorthand resets omitted properties to their initial values, which can override previously set values.
- **Only CSS 2.1 variant values in shorthand**: The `font` shorthand only supports `normal` and `small-caps` for `font-variant`; other variant values require separate declarations.
- **Variable font support**: Numeric `font-weight` values from 1 to 1000 and `font-stretch` percentages require variable fonts for precise control.
- **Relative keyword limitations**: `lighter` and `bolder` for `font-weight` only consider four weight levels (100, 400, 700, 900), which can lead to unexpected results.
- **Accessibility**: Using relative units for `font-size` (rem, em, %) is more accessible than fixed pixels, as it respects user browser settings.
- **System font keywords**: System font keywords (`menu`, `icon`, etc.) must be used alone and resolve to different fonts on different operating systems.

---

## References

- MDN Web Docs — `font-family` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-family
- MDN Web Docs — `font-size` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-size
- MDN Web Docs — `font-weight` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-weight
- MDN Web Docs — `font-style` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-style
- MDN Web Docs — `font-variant` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-variant
- MDN Web Docs — `font-stretch` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-stretch
- MDN Web Docs — `font` Shorthand Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font
- MDN Web Docs — Fundamental Text and Font Styling - https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Text_styling/Fundamentals
- MDN Web Docs — CSS Fonts Guide - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts
- MDN Web Docs — Variable Fonts Guide - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_fonts/Variable_fonts_guide
- W3C — CSS Fonts Module Level 4 - https://www.w3.org/TR/css-fonts-4/
- W3C — CSS Fonts Module Level 3 - https://www.w3.org/TR/css-fonts-3/
- W3C — CSS Font Loading Module Level 3 - https://www.w3.org/TR/css-font-loading-3/
- MDN Web Docs — Shorthand Properties Guide - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Shorthand_properties