# CSS Web Fonts — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Web Fonts refers to the set of CSS mechanisms that allow authors to load and use custom typefaces hosted on a server, rather than relying solely on fonts installed on the user's device. The central mechanism is the `@font-face` at-rule, which defines a custom font family and specifies its source files. This system encompasses font file formats (WOFF2, WOFF, TTF, OTF, EOT), font loading strategies (via the `font-display` descriptor), and fallback font mechanisms that ensure text remains readable while custom fonts are loading or if they fail to load entirely.

**Technical Definition:** In formal CSS terms, `@font-face` is an at-rule that establishes a font face within a document's font family namespace. It accepts descriptors including `font-family` (the name used to reference the font), `src` (a prioritised list of font sources, each optionally qualified by `format()` and `tech()`), `font-weight`, `font-style`, `font-stretch`, `unicode-range`, and `font-display`. The `src` descriptor follows a comma-separated fallback model similar to `font-family`, where the user agent iterates through sources until it finds one it can successfully load and use. The `font-display` descriptor controls the font's rendering behaviour during the download period, defining block, swap, and failure periods. The font loading process involves the CSS Font Loading API (Level 3) and the browser's font matching algorithm, which resolves characters to glyphs using the loaded font or a fallback. Web font files are typically served over HTTP and must be CORS-compliant when loaded cross-origin.

**Beginner-Friendly Explanation:** Normally, when you set a font like `Arial` or `Georgia` in CSS, you are relying on the user's computer already having that font installed. But what if you want to use a font that most people do not have — like a beautiful custom typeface you bought or downloaded? That is where CSS Web Fonts come in. Using the `@font-face` rule, you can tell the browser: "Download this font file from my server and use it to display my text." You put the font file on your website, write a little CSS to point to it, and the browser fetches it and uses it. The tricky part is that downloading a font takes time, so you need to decide what should happen while the font is loading — should the text be invisible? Should it show a fallback font? Should it swap later? That is what `font-display` controls.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **At-rule syntax** | `@font-face` is not a selector-based rule; it is an at-rule that declares a font face globally within the stylesheet. |
| **Descriptor-based** | Unlike properties applied to selectors, `@font-face` uses descriptors (e.g., `font-family`, `src`, `font-display`) that describe the font face itself. |
| **Not declared within selectors** | `@font-face` cannot be nested inside a CSS rule; it must be at the top level of a stylesheet or within a conditional group at-rule. |
| **Self-contained** | Each `@font-face` rule defines one specific font face (one weight, one style). Multiple rules are needed for a family with multiple weights. |
| **Format-aware loading** | The `src` descriptor uses `format()` hints so browsers can skip sources they do not support, avoiding unnecessary downloads. |
| **Progressive enhancement** | Web fonts are additive; a complete `font-family` stack with generic fallbacks ensures text remains readable even if the web font fails. |
| **CORS-restricted** | Font files loaded cross-origin require proper CORS headers (`Access-Control-Allow-Origin`). |
| **Variable font support** | Modern `@font-face` can declare variable fonts, allowing a single file to cover a range of weights, widths, and styles. |

---

### Prerequisites

Before studying CSS Web Fonts, you should understand:

- **Basic CSS syntax** — selectors, properties, values, at-rules, and the cascade.
- **The `font-family` property** — how font stacks and fallbacks work, and the self-contained nature of `font-family` declarations.
- **HTML document structure** — how CSS is linked and applied.
- **HTTP basics** — requests, responses, caching, and CORS.
- **Font terminology** — the distinction between typeface, font family, font face, weight, and style.

---

### Related Programming Areas

- **Web Performance** — font loading, FOIT/FOUT, CLS, preloading, subsetting.
- **Accessibility** — ensuring text remains readable during font loading, respecting user font preferences.
- **Internationalisation (i18n)** — `unicode-range` and script-specific font loading.
- **Typography** — variable fonts, OpenType features, `font-variation-settings`.
- **Design Systems** — managing branded typography across a component library.

---

### Core Concepts / Features

1. `@font-face`
2. Font File Formats
3. Font Loading
4. Fallback Fonts
5. `font-display` Strategies

---

## 1. `@font-face`

### Definitions

**Core Definition:** The `@font-face` at-rule defines a custom font family that can be used in `font-family` declarations, specifying the source files from which the font should be loaded and descriptors that control its rendering behaviour.

**Technical Definition:** `@font-face` is a CSS at-rule that associates a font family name with one or more font resources. It accepts the descriptors `font-family` (required), `src` (required), `font-style`, `font-weight`, `font-stretch`, `font-display`, `unicode-range`, `font-feature-settings`, `font-variation-settings`, and others. The rule can be declared at the top level of a stylesheet or within a conditional group at-rule. Multiple `@font-face` rules with the same `font-family` name create a font family with multiple faces (e.g., regular and bold). The `src` descriptor is a comma-separated list of `url()` and `local()` sources, each optionally qualified with `format()` and `tech()`. The user agent iterates through the list until it finds a source it can load. If no source in a given `@font-face` rule can be loaded, the next rule (if any) for that family is tried, ultimately falling back to the `font-family` stack.

**Beginner-Friendly Explanation:** The `@font-face` rule is like a recipe card for the browser. It says: "Here is a font family called 'MyFont'. Here is where you can find the file. Here is what weight and style it represents." You write one card for each weight and style you want to use (regular, bold, italic, etc.). Once the browser reads these cards, you can use `font-family: 'MyFont', sans-serif;` anywhere in your CSS, just as if 'MyFont' were a normal installed font.

---

### Purposes

- To load custom typefaces from a server for use in web documents.
- To eliminate dependence on the limited set of fonts installed on user devices.
- To provide precise control over typography and branding.
- To support variable fonts that cover multiple weights and styles in a single file.
- To enable progressive enhancement with fallback fonts during loading.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
@font-face {
    font-family: <family-name>;
    src: [ <url> [ format( <font-format> ) ]? [ tech( <font-tech> ) ]? ]# | <font-face-name>#;
    font-style: <font-style>;
    font-weight: <font-weight>;
    font-stretch: <font-stretch>;
    font-display: <font-display>;
    unicode-range: <urange>#;
}
```

#### Component Breakdown

| Descriptor | Required | Description | Example |
|---|---|---|---|
| `font-family` | Yes | The name used to reference the font in `font-family` property values. | `font-family: "MyFont";` |
| `src` | Yes | A comma-separated list of sources. Each source is a `url()` or `local()` reference, optionally followed by `format()` and `tech()`. | `src: url("font.woff2") format("woff2");` |
| `font-style` | No | The style this font face represents. Can be a keyword or a range (CSS Fonts Level 4). | `font-style: italic;` or `font-style: normal italic;` |
| `font-weight` | No | The weight this font face represents. Can be a single value or a range (CSS Fonts Level 4). | `font-weight: 400;` or `font-weight: 400 700;` |
| `font-stretch` | No | The width this font face represents. | `font-stretch: condensed;` |
| `font-display` | No | Controls rendering during font loading. | `font-display: swap;` |
| `unicode-range` | No | Restricts the font face to specific Unicode codepoints. | `unicode-range: U+0000-00FF;` |

#### Syntax Rules

1. `@font-face` **cannot** be declared inside a CSS selector; it must be at the top level of a stylesheet or within a conditional group at-rule.
2. The `font-family` and `src` descriptors are **required**; all others are optional.
3. The `src` descriptor must be a comma-separated list of sources.
4. `local()` sources should be placed **before** `url()` sources so that locally installed fonts are preferred over downloads.
5. The `format()` hint tells the browser the file format, allowing it to skip unsupported formats without downloading them.
6. A given set of `@font-face` rules defines a set of fonts available for use within the documents that contain these rules.
7. All prior declarations for a descriptor are ignored if a descriptor is repeated within the same `@font-face` rule.

#### Constraints and Limitations

- **CORS requirement** — cross-origin font files require `Access-Control-Allow-Origin` headers.
- **MIME type requirement** — server must serve fonts with correct MIME types (e.g., `font/woff2`).
- **No wildcard family names** — `font-family` in `@font-face` defines a single name; you cannot define a family that matches multiple installed fonts.
- **One face per rule** — each `@font-face` rule defines one specific face; multiple rules are needed for bold, italic, etc.
- **Variable font support varies** — CSS Fonts Level 4 features (ranges for weight/style) have varying browser support.
- **Tech() hints** — `tech()` is a newer addition and may not be supported in older browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `@font-face` with WOFF2 and WOFF Fallbacks

**HTML File (`index.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basic @font-face Example</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <!-- Heading that will use the custom web font -->
    <h1 class="custom-font">Custom Web Font Demo</h1>
    <!-- Paragraph using the custom font with fallbacks -->
    <p class="custom-font">
        This paragraph uses a custom web font loaded via the @font-face rule.
        If the font file is unavailable, the browser will fall back to
        the system sans-serif font defined in the font stack.
    </p>
</body>
</html>
```

**CSS File (`styles.css`):**

```css
/* Define the custom font face */
@font-face {
    /* The name we will use to reference this font */
    font-family: "MyCustomFont";

    /* Priority list of sources: try WOFF2 first, then WOFF.
       The format() hint allows browsers to skip unsupported formats. */
    src: url("fonts/mycustomfont.woff2") format("woff2"),
         url("fonts/mycustomfont.woff") format("woff");

    /* This face represents the normal weight */
    font-weight: 400;

    /* This face represents the normal style */
    font-style: normal;

    /* Show fallback font immediately, swap when ready */
    font-display: swap;
}

/* Apply the custom font with a complete fallback stack */
.custom-font {
    font-family: "MyCustomFont", "Helvetica Neue", Arial, sans-serif;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder with a `fonts/` subfolder.
2. Place your font files (`mycustomfont.woff2` and `mycustomfont.woff`) in the `fonts/` folder.
3. Save the HTML as `index.html` and the CSS as `styles.css`.
4. Open `index.html` in a modern browser.
5. Open DevTools → Network tab and filter by "Font" to see the font file being loaded.
6. The heading and paragraph should render in the custom font.

**Expected Output:** The `<h1>` and `<p>` elements render in "MyCustomFont". During loading, the fallback sans-serif font is shown briefly, then swapped to the custom font once it loads. If the font files are missing, the text remains in the fallback sans-serif font.

**Why This Works:** The `@font-face` rule defines the custom family "MyCustomFont" and tells the browser where to find the font files. The `src` list tries WOFF2 first (best compression), then WOFF (broader support). The `font-display: swap` ensures text is visible immediately using the fallback font, then swaps when the custom font is ready. The `font-family` declaration on `.custom-font` references the custom font and provides a complete fallback stack.

---

#### Example 2: Multiple Weights with Separate `@font-face` Rules

**HTML File (`weights.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multiple Weights with @font-face</title>
    <link rel="stylesheet" href="weights.css">
</head>
<body>
    <!-- Different weights of the same custom family -->
    <h1 class="light">Light Weight (300)</h1>
    <h1 class="regular">Regular Weight (400)</h1>
    <h1 class="bold">Bold Weight (700)</h1>
    <p>
        Each weight uses a separate @font-face rule with the same family name
        but a different font-weight descriptor and a different source file.
    </p>
</body>
</html>
```

**CSS File (`weights.css`):**

```css
/* Light weight (300) */
@font-face {
    font-family: "MyFont";
    src: url("fonts/myfont-light.woff2") format("woff2");
    font-weight: 300;
    font-style: normal;
    font-display: swap;
}

/* Regular weight (400) */
@font-face {
    font-family: "MyFont";
    src: url("fonts/myfont-regular.woff2") format("woff2");
    font-weight: 400;
    font-style: normal;
    font-display: swap;
}

/* Bold weight (700) */
@font-face {
    font-family: "MyFont";
    src: url("fonts/myfont-bold.woff2") format("woff2");
    font-weight: 700;
    font-style: normal;
    font-display: swap;
}

/* Apply the font family to the body */
body {
    font-family: "MyFont", Arial, sans-serif;
}

/* Each heading uses a different weight via font-weight */
.light {
    font-weight: 300;
}

.regular {
    font-weight: 400;
}

.bold {
    font-weight: 700;
}
```

**Step-by-Step Setup Guide:**

1. Place the three weight-specific font files in the `fonts/` folder.
2. Save the HTML as `weights.html` and CSS as `weights.css`.
3. Open `weights.html` in a browser.
4. Observe that each heading renders in a different weight of the same typeface.
5. Inspect the computed styles in DevTools to verify the correct font file is used for each weight.

**Expected Output:** Three headings in light, regular, and bold weights of "MyFont". The browser selects the correct `@font-face` rule based on the `font-weight` value applied to each element.

**Why This Works:** Each `@font-face` rule defines the same `font-family` name but a different `font-weight` and `src`. When the browser needs to render text at weight 300, it looks for a font face in the "MyFont" family with `font-weight: 300` and finds the first rule. This is the standard mechanism for defining multi-weight web font families.

---

#### Example 3: Variable Font with a Single `@font-face` Rule

**HTML File (`variable.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Variable Font Example</title>
    <link rel="stylesheet" href="variable.css">
</head>
<body>
    <h1 class="thin">Thin (100)</h1>
    <h1 class="regular">Regular (400)</h1>
    <h1 class="black">Black (900)</h1>
    <p>
        A single variable font file covers all weights from 100 to 900.
        The @font-face rule uses a weight range, and the browser
        interpolates the correct weight for each element.
    </p>
</body>
</html>
```

**CSS File (`variable.css`):**

```css
/* Single @font-face rule for a variable font */
@font-face {
    font-family: "MyVariableFont";
    /* One file covers the entire weight range */
    src: url("fonts/myvariablefont.woff2") format("woff2-variations");
    /* CSS Fonts Level 4: declare a range of supported weights */
    font-weight: 100 900;
    font-style: normal;
    font-display: swap;
}

body {
    font-family: "MyVariableFont", Arial, sans-serif;
}

/* Apply different weights — the variable font interpolates */
.thin { font-weight: 100; }
.regular { font-weight: 400; }
.black { font-weight: 900; }
```

**Step-by-Step Setup Guide:**

1. Download a variable font file (e.g., from Google Fonts) and place it in the `fonts/` folder.
2. Save the HTML as `variable.html` and CSS as `variable.css`.
3. Open in a browser that supports variable fonts (all modern browsers).
4. Observe the three headings render in thin, regular, and black weights from the same file.

**Expected Output:** Three headings with visibly different weights, all rendered from a single font file. The browser interpolates the weight axis based on the `font-weight` value applied to each element.

**Why This Works:** The `font-weight: 100 900` descriptor declares that this single font face supports the entire weight range. When the browser needs to render text at weight 100, 400, or 900, it uses the same file but applies the appropriate variation axis value. This eliminates the need for separate files per weight.

---

### Real-World Cases

- **Brand typography:** A company loads its proprietary typeface via `@font-face` to ensure consistent brand identity across all pages.
- **Google Fonts integration:** Google Fonts serves `@font-face` rules dynamically, allowing developers to use thousands of open-source fonts.
- **Variable font performance:** A news site uses a variable font to cover headlines, body text, and UI elements with a single file, reducing HTTP requests.
- **Icon fonts:** Some icon libraries (e.g., Font Awesome) use `@font-face` to load a font where glyphs are icons.

---

## 2. Font File Formats

### Definitions

**Core Definition:** Font file formats are the binary file specifications that encode typeface data (glyph outlines, metrics, kerning, OpenType features) for use in digital environments. For web use, several formats exist, each with different compression, browser support, and feature sets.

**Technical Definition:** Web font file formats include WOFF2 (Web Open Font Format 2), WOFF (Web Open Font Format 1), TTF (TrueType Font), OTF (OpenType Font), EOT (Embedded OpenType), and SVG fonts. WOFF2 and WOFF are wrapper formats that encapsulate either TrueType or OpenType font data, adding compression and metadata. WOFF2 uses Brotli compression and offers the best compression ratio. TTF and OTF are the underlying font formats, with TTF using quadratic Bézier curves and OTF using cubic Bézier curves (PostScript outlines). EOT is a legacy Microsoft format for older Internet Explorer. SVG fonts are XML-based and have been removed from most modern browsers. The `format()` hint in the `src` descriptor tells the user agent which format a given source provides.

**Beginner-Friendly Explanation:** Font files come in different formats, like different container types. TTF and OTF are the original "full-size" formats — they work but are large. WOFF is a compressed version designed specifically for the web. WOFF2 is the newer, even better compressed version and is what you should use today. EOT is an ancient format for old Internet Explorer, and SVG fonts are basically dead. The rule of thumb: use WOFF2 first, then WOFF as a fallback, and you will cover virtually all modern browsers.

---

### Purposes

- To encode typeface data in a format readable by browsers and operating systems.
- To provide compression for web delivery, reducing file size and load time.
- To encapsulate metadata (licensing, origin) within the font file.
- To support advanced typographic features (OpenType layout, variable axes).
- To maintain backward compatibility with older browsers through format fallbacks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
@font-face {
    font-family: "FontName";
    src: url("font.woff2") format("woff2"),
         url("font.woff") format("woff"),
         url("font.ttf") format("truetype");
}
```

#### Component Breakdown

| Format | `format()` Value | Description | Browser Support | Recommended? |
|---|---|---|---|---|
| WOFF2 | `"woff2"` | Best compression, modern standard. | All modern browsers. | ✅ Yes |
| WOFF | `"woff"` | Good compression, broad support. | All modern browsers, IE9+. | ✅ As fallback |
| TTF | `"truetype"` | Uncompressed TrueType. | Wide support, but large files. | ⚠️ Legacy only |
| OTF | `"opentype"` | Uncompressed OpenType. | Wide support, but large files. | ⚠️ Legacy only |
| EOT | `"embedded-opentype"` | Legacy Microsoft format. | IE8 and below only. | ❌ Deprecated |
| SVG | `"svg"` | XML-based font format. | Removed from most browsers. | ❌ Deprecated |

#### Syntax Rules

1. The `format()` hint is **optional** but strongly recommended — it allows browsers to skip unsupported formats.
2. Multiple sources are listed **comma-separated** in the `src` descriptor.
3. Sources are tried in **order**; the first supported and loadable format is used.
4. `local()` sources should be placed **before** `url()` sources.
5. The `format()` value must match the actual file format; an incorrect hint may cause the source to be skipped.
6. For variable fonts, use `format("woff2-variations")` or `format("woff2") tech(variations)`.

#### Constraints and Limitations

- **WOFF2 is not universally supported in very old browsers** — but this is negligible in practice.
- **TTF/OTF files are large** — using them without compression increases load times.
- **EOT and SVG are deprecated** — EOT is only needed for IE8 and below; SVG fonts are no longer supported.
- **MIME types matter** — the server must serve the correct `Content-Type` (e.g., `font/woff2`).
- **CORS** — cross-origin font files require proper CORS headers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Modern Format Stack (WOFF2 + WOFF)

**HTML File (`formats.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Font Format Stack</title>
    <link rel="stylesheet" href="formats.css">
</head>
<body>
    <h1>Modern Font Format Stack</h1>
    <p>
        This page uses a modern @font-face declaration that tries WOFF2 first,
        then WOFF. Both formats are supported by all modern browsers.
    </p>
</body>
</html>
```

**CSS File (`formats.css`):**

```css
@font-face {
    font-family: "ModernFont";
    /* Best format first: WOFF2 with Brotli compression */
    src: url("fonts/modernfont.woff2") format("woff2"),
         /* Fallback: WOFF with zlib compression */
         url("fonts/modernfont.woff") format("woff");
    font-weight: 400;
    font-style: normal;
    font-display: swap;
}

body {
    font-family: "ModernFont", Arial, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}
```

**Step-by-Step Setup Guide:**

1. Place `modernfont.woff2` and `modernfont.woff` in the `fonts/` folder.
2. Save the HTML and CSS files.
3. Open in a browser. Inspect the Network tab — only the WOFF2 file should be downloaded (the WOFF is skipped).
4. Test in an older browser (if available) — it should download the WOFF file.

**Expected Output:** The text renders in "ModernFont". In modern browsers, only the WOFF2 file is downloaded. In older browsers that do not support WOFF2, the WOFF file is downloaded instead.

**Why This Works:** The browser evaluates the `src` list. It checks if it supports the `woff2` format. If yes, it downloads and uses that file, ignoring the rest of the list. If not, it proceeds to the next source. This minimises bandwidth usage while maintaining broad compatibility.

---

#### Example 2: Legacy Format Stack (Full Compatibility)

**HTML File (`legacy.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Legacy Font Format Stack</title>
    <link rel="stylesheet" href="legacy.css">
</head>
<body>
    <h1>Legacy Font Format Stack</h1>
    <p>
        This page uses a comprehensive @font-face declaration that includes
        EOT, WOFF2, WOFF, TTF, and SVG formats. This is only necessary if you
        need to support very old browsers like IE8.
    </p>
</body>
</html>
```

**CSS File (`legacy.css`):**

```css
@font-face {
    font-family: "LegacyFont";
    /* EOT for IE8 and below */
    src: url("fonts/legacyfont.eot");
    /* EOT with ?#iefix to fix a rendering bug in IE8 */
    src: url("fonts/legacyfont.eot?#iefix") format("embedded-opentype"),
         /* WOFF2 for modern browsers */
         url("fonts/legacyfont.woff2") format("woff2"),
         /* WOFF for slightly older browsers */
         url("fonts/legacyfont.woff") format("woff"),
         /* TTF for very old browsers that don't support WOFF */
         url("fonts/legacyfont.ttf") format("truetype"),
         /* SVG for legacy iOS Safari */
         url("fonts/legacyfont.svg#LegacyFont") format("svg");
    font-weight: 400;
    font-style: normal;
    font-display: swap;
}

body {
    font-family: "LegacyFont", Arial, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}
```

**Step-by-Step Setup Guide:**

1. Place all five format files in the `fonts/` folder.
2. Save the HTML and CSS.
3. Open in a modern browser — only the WOFF2 file is downloaded.
4. Open in IE8 (if available) — the EOT file is downloaded.

**Expected Output:** The text renders in "LegacyFont" across all browsers, from IE8 to modern Chrome. Each browser downloads only the format it supports.

**Why This Works:** The `src` list starts with the EOT file (needed for IE8) and then provides progressively more modern formats. The browser iterates the list and uses the first format it supports. This approach is only necessary if you must support browsers older than IE9.

> **⚠️ Important:** The EOT and SVG formats are deprecated. Modern best practice is to use only WOFF2 and WOFF. The legacy stack shown above is included for completeness and historical reference.

---

### Real-World Cases

- **Google Fonts:** Serves WOFF2 to modern browsers and WOFF to older ones, automatically detecting format support.
- **Adobe Fonts (Typekit):** Uses a similar format stack, with WOFF2 as the primary format.
- **Self-hosted fonts:** A developer downloads fonts from Google Fonts and self-hosts them, using only WOFF2 and WOFF to keep the codebase simple.

---

## 3. Font Loading

### Definitions

**Core Definition:** Font loading is the process by which a browser fetches, parses, and makes available a web font file specified in an `@font-face` rule, and the set of strategies used to manage the user experience during that process.

**Technical Definition:** Font loading in CSS involves the browser's font matching algorithm, which resolves a required font face from the `@font-face` rules available in the document. When a font face is needed for rendering, the browser checks whether the corresponding font resource is already loaded. If not, it initiates a download (subject to CORS and caching). During the download period, the browser's rendering behaviour is governed by the `font-display` descriptor. The CSS Font Loading API provides programmatic control over font loading, exposing `FontFace` objects and the `document.fonts` interface. Font loading can be optimised through preloading (`<link rel="preload">`), subsetting, and caching. The terms FOIT (Flash of Invisible Text) and FOUT (Flash of Unstyled Text) describe the two main loading-related rendering phenomena.

**Beginner-Friendly Explanation:** When you use a web font, the browser has to download the font file before it can display your text in that font. This takes time — sometimes a fraction of a second, sometimes longer on slow connections. Font loading is all about what happens during that waiting period. If you do nothing, the browser might hide the text entirely until the font is ready (FOIT). Or it might show a fallback font and then swap (FOUT). The `font-display` property lets you choose which behaviour you prefer. You can also preload the font to start the download earlier, and you can subset the font to make the file smaller.

---

### Purposes

- To fetch and make available custom font resources for text rendering.
- To manage the user experience during the font download period.
- To minimise layout shifts caused by font swapping.
- To optimise font delivery through preloading, caching, and subsetting.
- To provide programmatic control over font loading via the CSS Font Loading API.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Basic @font-face with font-display */
@font-face {
    font-family: "FontName";
    src: url("font.woff2") format("woff2");
    font-display: swap;
}

/* Preload in HTML */
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
```

#### Component Breakdown

| Technique | Description | Implementation |
|---|---|---|
| `font-display` | Controls rendering during font loading. | CSS descriptor in `@font-face`. |
| Preload | Starts font download earlier in the page lifecycle. | `<link rel="preload" as="font">` in HTML. |
| `local()` source | Uses a locally installed font if available, avoiding download. | First entry in `src` descriptor. |
| Subsetting | Reduces font file size by including only needed characters. | External tools (e.g., `pyftsubset`). |
| CSS Font Loading API | Programmatic control over font loading. | `document.fonts.load()`, `FontFace` constructor. |

#### Syntax Rules

1. `font-display` is declared **inside** `@font-face`, not on a selector.
2. Preload links must include `as="font"`, `type`, and `crossorigin` attributes.
3. Preloading should be used sparingly — only for fonts critical to the initial render.
4. `local()` sources must be quoted family names.
5. The CSS Font Loading API uses promises and events to track loading progress.

#### Constraints and Limitations

- **Preloading too many fonts** can degrade performance.
- **Subsetting requires tooling** — it is not a CSS feature.
- **CORS** — preloaded fonts must be CORS-compliant.
- **Caching** — font files are cached by the browser, but cache-control headers control duration.
- **FOIT/FOUT trade-offs** — there is no perfect solution; the choice depends on content and design priorities.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preloading a Critical Font

**HTML File (`preload.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Font Preloading</title>
    <!-- Preload the critical font to start download early -->
    <link rel="preload"
          href="fonts/myfont.woff2"
          as="font"
          type="font/woff2"
          crossorigin>
    <link rel="stylesheet" href="preload.css">
</head>
<body>
    <h1>Preloaded Font Demo</h1>
    <p>
        The font used for this heading was preloaded, so the download started
        as soon as the HTML was parsed, before the CSS was even processed.
        This reduces the time the fallback font is visible.
    </p>
</body>
</html>
```

**CSS File (`preload.css`):**

```css
@font-face {
    font-family: "MyFont";
    src: url("fonts/myfont.woff2") format("woff2");
    font-weight: 400;
    font-style: normal;
    font-display: swap;
}

body {
    font-family: "MyFont", Arial, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}
```

**Step-by-Step Setup Guide:**

1. Place `myfont.woff2` in the `fonts/` folder.
2. Save the HTML and CSS.
3. Open in a browser and check the Network tab — the font download should start very early, alongside CSS and other critical resources.
4. Compare with a version without the preload link — the font download starts later.

**Expected Output:** The heading renders in "MyFont". Because of preloading, the font is available sooner, reducing the duration of the fallback font display.

**Why This Works:** The `<link rel="preload">` tag tells the browser to start downloading the font immediately, before it has even parsed the CSS that references it. This eliminates the discovery delay that would otherwise occur when the browser processes the `@font-face` rule.

---

### Real-World Cases

- **Landing pages:** Preloading the hero font to ensure the headline appears in the brand font as quickly as possible.
- **Progressive web apps (PWAs):** Using the CSS Font Loading API to load fonts based on user interaction or network conditions.
- **Performance-critical sites:** Subsetting fonts to include only the characters used on the page, reducing file size by 50–90%.

---

## 4. Fallback Fonts

### Definitions

**Core Definition:** Fallback fonts are the alternative typefaces specified in a `font-family` stack or in the `src` descriptor of `@font-face`, used when the preferred font is unavailable, still loading, or lacks a glyph for a particular character.

**Technical Definition:** Fallback fonts operate at two levels. First, within `@font-face`, the `src` descriptor lists multiple font sources; if the first source cannot be loaded, the next is tried. Second, within the `font-family` property, a comma-separated stack of family names and generic families is provided; the user agent iterates through the list until it finds an available font containing a glyph for the character to be rendered. Font fallback is **self-contained**: a `font-family` declaration on an element resolves entirely within that declaration and does not inherit the parent's fallback stack. If all fonts in a child element's declaration are unavailable, the browser falls back to its default font (typically Times New Roman), not to the parent's font. Character-by-character fallback occurs when a font lacks a glyph for a specific character; the browser proceeds to the next font in the stack for that character alone.

**Beginner-Friendly Explanation:** A fallback font is your backup plan. When you write `font-family: "MyCustomFont", Arial, sans-serif;`, you are saying: "Try MyCustomFont first. If that is not available, use Arial. If Arial is not available either, use any sans-serif font." The crucial thing to understand is that fallbacks are self-contained. If you set a custom font on a child element and forget to include fallbacks, the browser will not "look up" to the parent's font stack — it will use its own default (usually Times New Roman). This is why you should always write complete font stacks.

---

### Purposes

- To ensure text remains readable when a preferred font is unavailable.
- To maintain the stylistic category (serif, sans-serif, monospace) when a specific font fails.
- To provide a graceful degradation path during font loading.
- To handle characters not covered by the primary font (e.g., emoji, non-Latin scripts).
- To prevent the browser from falling back to an inappropriate default font.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Within @font-face src */
@font-face {
    font-family: "MyFont";
    src: local("MyFont"),
         url("myfont.woff2") format("woff2"),
         url("myfont.woff") format("woff");
}

/* Within font-family */
selector {
    font-family: "PreferredFont", "FallbackFont", generic-family;
}
```

#### Component Breakdown

| Level | Mechanism | Fallback Order |
|---|---|---|
| `src` | Comma-separated sources in `@font-face`. | `local()` first, then `url()` formats in priority order. |
| `font-family` | Comma-separated family names and generic families. | Specific fonts first, generic family last. |
| Character-level | If a font lacks a glyph, the next font in the stack is used for that character. | Automatic, based on `unicode-range` and glyph coverage. |

#### Syntax Rules

1. Always include a **generic family** at the end of a `font-family` stack.
2. `font-family` declarations are **self-contained** — child elements do not inherit parent fallback stacks.
3. When overriding `font-family` on a child element, **repeat the complete stack**.
4. `local()` sources in `@font-face` should come **before** `url()` sources.
5. Generic families should **not** be quoted.
6. Family names with spaces must be quoted.

#### Constraints and Limitations

- **No conditional fallback** — you cannot specify "use Font B only if Font A is unavailable and the user is on Windows."
- **Browser default risk** — a single-font stack with no fallback leads to the browser's default font (often Times New Roman).
- **Metric mismatches** — fallback fonts may have different metrics, causing layout shifts when the custom font swaps in.
- **Character coverage gaps** — even if a font loads, it may lack glyphs for certain characters.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Demonstrating Self-Contained Fallback

**HTML File (`fallback.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Self-Contained Fallback Demo</title>
    <link rel="stylesheet" href="fallback.css">
</head>
<body>
    <!-- This heading uses a single-font stack with no fallback -->
    <h1 class="broken">Broken Stack (No Fallback)</h1>
    <!-- This heading uses a complete stack with fallback -->
    <h1 class="fixed">Fixed Stack (With Fallback)</h1>
    <p>
        The first heading uses <code>font-family: "NonExistentFont";</code>
        with no fallback. The browser falls back to its default — usually
        Times New Roman — instead of inheriting the body's sans-serif stack.
    </p>
    <p>
        The second heading uses <code>font-family: "NonExistentFont", Arial, sans-serif;</code>
        so it always renders in a sans-serif font, even when the custom font
        is unavailable.
    </p>
</body>
</html>
```

**CSS File (`fallback.css`):**

```css
/* Body uses a system sans-serif stack */
body {
    font-family: system-ui, Arial, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
    color: #333;
}

/* BROKEN: single font, no fallback.
   If the font is unavailable, the browser uses its default,
   NOT the body's stack. */
.broken {
    font-family: "NonExistentFont";
    color: #c0392b;
}

/* FIXED: complete stack with generic fallback.
   Always renders in a sans-serif font. */
.fixed {
    font-family: "NonExistentFont", Arial, sans-serif;
    color: #27ae60;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML and CSS files.
2. Open in a browser.
3. Observe the first heading — it likely renders in Times New Roman (serif), despite the body being sans-serif.
4. Observe the second heading — it renders in Arial or the system sans-serif font.

**Expected Output:** The first heading appears in a serif font (Times New Roman) with red colour. The second heading appears in a sans-serif font with green colour. The surrounding paragraphs remain in the system sans-serif font.

**Why This Works:** The `font-family` declaration is self-contained. When `.broken` declares only `"NonExistentFont"`, the browser exhausts that declaration and falls back to its default (Times New Roman), not to the body's stack. The `.fixed` class includes `Arial, sans-serif` as fallbacks, so the browser has viable alternatives within the declaration itself.

---

### Real-World Cases

- **Design systems:** Defining custom properties like `--font-heading: "BrandFont", "Helvetica Neue", Arial, sans-serif;` to ensure complete stacks are reused consistently.
- **Multilingual sites:** Using `unicode-range` with `@font-face` to load different fonts for Latin, Cyrillic, and Greek characters, with appropriate fallbacks.
- **Email templates:** Using only web-safe fallbacks because email clients have limited web font support.

---

## 5. `font-display` Strategies

### Definitions

**Core Definition:** The `font-display` descriptor controls how a web font is rendered during the period in which it is being downloaded, balancing the visibility of text against the visual fidelity of the final font.

**Technical Definition:** `font-display` is a descriptor of the `@font-face` at-rule that defines a timeline consisting of a **block period**, a **swap period**, and a **failure period**. During the block period, if the font is not yet loaded, the text is rendered with an invisible fallback (FOIT). During the swap period, the fallback font is displayed and swapped when the custom font loads (FOUT). The five values are `auto` (browser default, often `block`), `block` (short block period, infinite swap period), `swap` (extremely small block period, infinite swap period), `fallback` (extremely small block period, short swap period), and `optional` (extremely small block period, no swap period). The `optional` value, when combined with preloading, can eliminate layout shifts caused by font swapping.

**Beginner-Friendly Explanation:** When a web font is downloading, you have to decide what the text looks like in the meantime. `font-display` gives you five options. `block` hides the text until the font is ready (bad for slow connections). `swap` shows a fallback font immediately and swaps when the font arrives (good for readability, but can cause a visible change). `fallback` is similar to `swap` but gives up after a short time. `optional` shows the fallback and only uses the web font if it is already cached — essentially eliminating layout shifts. The best choice depends on your content: for a hero headline, you might use `swap`; for body text, `optional` is often better for performance.

---

### Purposes

- To control the visibility of text during font loading.
- To minimise the negative impact of FOIT (invisible text) on user experience.
- To balance visual fidelity with performance and layout stability.
- To eliminate layout shifts (CLS) caused by font swapping.
- To provide a browser-agnostic strategy for font rendering behaviour.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
@font-face {
    font-family: "FontName";
    src: url("font.woff2") format("woff2");
    font-display: auto | block | swap | fallback | optional;
}
```

#### Component Breakdown

| Value | Block Period | Swap Period | Behaviour | Best For |
|---|---|---|---|---|
| `auto` | Browser-dependent | Browser-dependent | Uses the browser's default strategy (usually `block`). | Default; not recommended for production. |
| `block` | Short (~3s) | Infinite | Text invisible during block; fallback shown during swap. | Icons and symbol fonts where incorrect glyphs are worse than invisible text. |
| `swap` | Extremely small (~100ms) | Infinite | Fallback shown immediately; swapped when font loads. | Brand-critical text (headlines, logos). |
| `fallback` | Extremely small (~100ms) | Short (~3s) | Fallback shown; if font loads within 3s, swapped; otherwise, not swapped. | Body text where a swap after 3s is undesirable. |
| `optional` | Extremely small (~100ms) | None | Fallback shown; font used only if already cached (or loads within ~100ms). | Performance-critical sites where CLS must be zero. |

#### Syntax Rules

1. `font-display` is declared **inside** `@font-face`.
2. It applies to the entire font face, not per-element.
3. The block and swap periods are measured from when the font face is first needed.
4. `optional` gives the browser discretion to not use the font at all if it is not cached.
5. `swap` and `fallback` produce FOUT; `block` produces FOIT.

#### Constraints and Limitations

- **No perfect solution** — every value has trade-offs.
- **Browser default for `auto` varies** — some browsers use `block`, others may use a different strategy.
- **`optional` may skip the font entirely** on first visit — the font is only used if already cached.
- **Preloading + `optional`** is the only combination that eliminates CLS entirely, but requires careful implementation.
- **Variable fonts** — `font-display` applies to the entire font file, including all variation axes.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `font-display: swap` for a Hero Headline

**HTML File (`swap.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>font-display: swap</title>
    <link rel="stylesheet" href="swap.css">
</head>
<body>
    <!-- Hero headline using swap strategy -->
    <h1 class="hero">Welcome to Our Site</h1>
    <p>
        The headline above uses <code>font-display: swap</code>. While the
        custom font is loading, the fallback font is displayed immediately.
        When the custom font is ready, the text swaps to the new font.
    </p>
</body>
</html>
```

**CSS File (`swap.css`):**

```css
@font-face {
    font-family: "HeroFont";
    src: url("fonts/herofont.woff2") format("woff2");
    font-weight: 700;
    font-style: normal;
    /* Swap: show fallback immediately, swap when font loads */
    font-display: swap;
}

body {
    font-family: system-ui, Arial, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}

.hero {
    font-family: "HeroFont", "Helvetica Neue", Arial, sans-serif;
    font-size: 2.5rem;
    color: #1a1a1a;
}
```

**Step-by-Step Setup Guide:**

1. Place `herofont.woff2` in the `fonts/` folder.
2. Save the HTML and CSS.
3. Open in a browser with a throttled network connection (DevTools → Network → Slow 3G).
4. Observe the headline text: it is immediately visible in the fallback font, then swaps to "HeroFont" when the download completes.

**Expected Output:** The headline is readable from the moment the page loads. It initially appears in the system sans-serif font, then switches to "HeroFont" once the file is downloaded.

**Why This Works:** `font-display: swap` gives the font face an extremely small block period (text is invisible for at most ~100ms) and an infinite swap period. This means the fallback font is shown almost immediately, ensuring that the headline is readable even on slow connections. The trade-off is a visible font swap (FOUT) and potential layout shift if the fonts have different metrics.

---

#### Example 2: `font-display: optional` for Body Text

**HTML File (`optional.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>font-display: optional</title>
    <link rel="stylesheet" href="optional.css">
</head>
<body>
    <h1>Body Text with Optional Font</h1>
    <p>
        This paragraph uses <code>font-display: optional</code>. If the custom
        font is not cached and does not load almost instantly, the browser
        will use the fallback font and not swap. This eliminates layout shifts.
        On subsequent visits, the font is cached and used immediately.
    </p>
    <p>
        Reload the page to see the font switch to the custom font (cached).
    </p>
</body>
</html>
```

**CSS File (`optional.css`):**

```css
@font-face {
    font-family: "BodyFont";
    src: url("fonts/bodyfont.woff2") format("woff2");
    font-weight: 400;
    font-style: normal;
    /* Optional: use if cached, otherwise never swap */
    font-display: optional;
}

body {
    font-family: "BodyFont", Georgia, serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.7;
    color: #333;
}
```

**Step-by-Step Setup Guide:**

1. Place `bodyfont.woff2` in the `fonts/` folder.
2. Save the HTML and CSS.
3. Open in a browser with a throttled network connection.
4. On the first visit, the text renders in Georgia (fallback) and does not swap.
5. Reload the page — the font is now cached, and the text renders in "BodyFont" immediately.

**Expected Output:** On the first visit, the body text renders in Georgia (the fallback). No swap occurs, so there is no layout shift. On subsequent visits, the font is loaded from cache and the text renders in "BodyFont" from the start.

**Why This Works:** `font-display: optional` gives the font face an extremely small block period and **no swap period**. If the font is not already cached or does not load almost instantly, the browser uses the fallback font and does not swap. This eliminates CLS entirely but means the custom font may not be used on the first visit if the network is slow.

---

### Real-World Cases

- **News sites:** Using `font-display: swap` for headlines to ensure content is readable immediately, accepting a brief font swap.
- **E-commerce product pages:** Using `font-display: optional` for body text to eliminate layout shifts that could affect conversions.
- **Icon fonts:** Using `font-display: block` to avoid displaying incorrect fallback glyphs (e.g., showing "A" instead of a shopping cart icon).

---

## References

- MDN Web Docs — `@font-face` - https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face
- MDN Web Docs — `font-display` - https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display
- MDN Web Docs — CSS Font Loading API - https://developer.mozilla.org/en-US/docs/Web/API/CSS_Font_Loading_API
- W3C — CSS Fonts Module Level 4 - https://www.w3.org/TR/css-fonts-4/
- W3C — CSS Fonts Module Level 3 - https://www.w3.org/TR/css-fonts-3/
- CSS-Tricks — Understanding Web Fonts and Getting the Most Out of Them - https://css-tricks.com/understanding-web-fonts-getting/
- CSS Wizardry — `font-family` Doesn't Fall Back the Way You Think - https://csswizardry.com/2026/04/font-family-doesnt-fall-back-the-way-you-think/
- Chrome for Developers — Ensure text remains visible during webfont load - https://developer.chrome.com/docs/lighthouse/performance/font-display
- web.dev — Avoid invisible text during font loading - https://web.dev/articles/avoid-invisible-text
- web.dev — Preload optional fonts to prevent layout shifts and FOIT - https://web.dev/articles/preload-optional-fonts
- Can I Use — WOFF2 font format - https://caniuse.com/woff2
- Google Fonts — `font-display` parameter - https://developers.google.com/fonts/docs/getting_started#use_font-display