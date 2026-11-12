# Deprecated HTML Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Deprecated HTML elements are elements that were defined in earlier versions of HTML — primarily HTML4 and its predecessors — but have been removed from the current WHATWG HTML Living Standard. They are classified as **non-conforming features** and must not be used by authors in new documents.

**Technical Definition**
In the WHATWG HTML Living Standard, obsolete features are divided into two categories: **obsolete but conforming features** (which trigger warnings in conformance checkers) and **non-conforming features** (which are entirely obsolete and must not be used). The elements covered in this cheat sheet belong to the latter category. The specification explicitly states: “Elements in the following list are entirely obsolete, and must not be used by authors”. These elements are retained in the specification only to define how user agents must handle them for backwards compatibility with legacy content.

**Beginner-Friendly Explanation**
Deprecated elements are old HTML tags that were once used to build web pages but are no longer part of the modern HTML language. Browsers may still display them for compatibility with old websites, but you should never use them in new projects. They have been replaced by better, more accessible, and more flexible alternatives — usually CSS or modern semantic HTML.

---

### Key Characteristics

- **Non-conforming**: They are explicitly forbidden in conforming HTML documents.
- **Presentational legacy**: Many deprecated elements (e.g. `<font>`, `<center>`, `<big>`, `<strike>`, `<tt>`) were purely presentational and are now replaced by CSS.
- **Structural legacy**: Some (e.g. `<frame>`, `<frameset>`, `<applet>`) represented obsolete architectural models that have been superseded by iframes, CSS layout, and modern embedding technologies.
- **Accessibility hazards**: Several (notably `<blink>`, `<marquee>`, and `<blink>`) violate WCAG success criteria due to distracting motion or blinking.
- **Browser compatibility**: Browsers generally still render them for legacy support, but behaviour is not guaranteed and may be inconsistent.
- **Alternatives are well-established**: The WHATWG specification and MDN provide explicit replacement guidance for every obsolete element.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<html>`, `<head>`, `<body>`, and modern content categories.
2. **CSS fundamentals**: understanding selectors, the box model, and properties such as `text-align`, `font-family`, and `margin`.
3. **Awareness of modern HTML5 semantics**: sectioning elements, text-level semantics, and embedded content.
4. **Browser developer tools**: for inspecting legacy markup and testing replacements.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | Most presentational deprecated elements are replaced by CSS properties. |
| **Accessibility (WCAG)** | `<blink>`, `<marquee>`, and `<frame>` have specific accessibility failure modes. |
| **Legacy browser support** | Some obsolete elements are still partially supported for backwards compatibility. |
| **HTML validation** | Conformance checkers warn or error on deprecated elements. |
| **Web Components / modern embedding** | `<applet>`, `<frame>`, and plugin-based elements are replaced by iframes and native HTML elements. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. `<applet>` — embedded Java applet
2. `<acronym>` — acronym
3. `<bgsound>` — background sound
4. `<big>` — larger text
5. `<blink>` — blinking text
6. `<center>` — centered text
7. `<dir>` — directory list
8. `<font>` — font styling
9. `<frame>` — individual frame
10. `<frameset>` — frame layout container
11. `<isindex>` — single-line prompt
12. `<keygen>` — key pair generator
13. `<marquee>` — scrolling text
14. `<menuitem>` — context menu item
15. `<nobr>` — no line break
16. `<noembed>` — fallback for embed
17. `<noframes>` — fallback for frames
18. `<plaintext>` — plain text
19. `<rb>` — ruby base
20. `<rtc>` — ruby text container
21. `<spacer>` — layout spacer
22. `<strike>` — struck-through text
23. `<tt>` — teletype text
24. `<xmp>` — example text
25. **Deprecated element alternatives**
26. **Deprecated element best practices**

---

## 1. `<applet>` — The Embedded Java Applet Element

### Definitions

**Core Definition**
The `<applet>` element was used to embed a Java applet into an HTML document. It has been entirely obsolete since HTML5.

**Technical Definition**
`<applet>` was a non-conforming element. The WHATWG specification states: “Use `embed` or `object` instead”. It was replaced because Java applets required a browser plugin (the Java Plugin), which has been discontinued in all major browsers due to security and performance concerns.

**Beginner-Friendly Explanation**
The `<applet>` element put a small Java program into a web page. It is completely obsolete because browsers no longer support Java plugins.

---

### Purposes

- **Historically: to embed interactive Java applets** for games, visualisations, and enterprise applications.
- **Modern guidance: do not use**; use `<object>` or `<embed>` for generic embedding, or native HTML/CSS/JavaScript for interactivity.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<applet code="MyApplet.class" width="300" height="200">
  <param name="param1" value="value1">
  Alternative content for browsers without Java.
</applet>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `code` | The Java class file to load. |
| `width` / `height` | Dimensions. |
| `codebase` | Base URL for the applet’s classes. |
| `<param>` | Configuration parameters. |

#### Constraints and Limitations

- Requires a Java plugin, which has been removed from all modern browsers.
- Non-conforming; must not be used in new documents.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Historical Applet Usage (For Reference Only)

**Setup guide**: This code will not run in any modern browser. It is shown only for historical context.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Historical Applet</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <applet code="HelloWorld.class" width="200" height="100">
    <param name="message" value="Hello">
    Your browser does not support Java applets.
  </applet>
</body>
</html>
```

**Expected output**: Nothing is rendered in modern browsers. The fallback text may be shown, but the applet itself will not load.

**Why this output occurs**: The `<applet>` element is obsolete, and browser support for Java plugins has been removed.

---

#### Example 2: Modern Replacement with `<object>`

**Setup guide**: Save as `object-replacement.html`. This replaces the applet with a generic object embedding.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Object Replacement</title>
</head>
<body>
  <!-- MODERN: Use object for generic embedding -->
  <object data="interactive.svg" type="image/svg+xml" width="300" height="200">
    <p>Your browser cannot display the SVG. <a href="interactive.svg">Download it</a>.</p>
  </object>
</body>
</html>
```

**Expected output**: The SVG is embedded and rendered. If the browser cannot display it, the fallback paragraph and download link are shown.

**Why this output occurs**: The `<object>` element provides a standards-based way to embed external resources with fallback content, replacing the obsolete `<applet>`.

---

### Real-World Cases

1. **Legacy enterprise applications**: Some old intranet systems may still contain `<applet>` markup, but the applets no longer run.
2. **Migration**: Modern equivalents use `<object>`, `<iframe>`, or JavaScript-based interactive components.

---

### References

- WHATWG — Obsolete features: `applet` — https://html.spec.whatwg.org/multipage/obsolete.html#applet
- MDN Web Docs — `<applet>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/applet

---

## 2. `<acronym>` — The Acronym Element

### Definitions

**Core Definition**
The `<acronym>` element was used to mark up an acronym. It has been replaced by the `<abbr>` element.

**Technical Definition**
`<acronym>` is a non-conforming element. The WHATWG specification states: “Use `abbr` instead”. The `<abbr>` element serves the same purpose but is more general — it marks both abbreviations and acronyms.

**Beginner-Friendly Explanation**
`<acronym>` marked acronyms like “HTML” or “NASA”. It has been replaced by `<abbr>`, which does the same job but is part of modern HTML.

---

### Purposes

- **Historically: to mark acronyms**.
- **Modern guidance: use `<abbr>`** with the `title` attribute for the expansion.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<acronym title="HyperText Markup Language">HTML</acronym>
```

#### Modern Replacement

```html
<abbr title="HyperText Markup Language">HTML</abbr>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `title` | The expansion of the abbreviation or acronym. |

#### Constraints and Limitations

- `<acronym>` is non-conforming; use `<abbr>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete Acronym Markup

**Setup guide**: Save as `acronym-obsolete.html`. This will trigger validation warnings.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Acronym</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p>The <acronym title="World Wide Web Consortium">W3C</acronym> sets web standards.</p>
</body>
</html>
```

**Expected output**: The browser may render the text normally, but a validator will report that `<acronym>` is obsolete.

**Why this output occurs**: The element is non-conforming, though browsers may still apply default styling.

---

#### Example 2: Modern `<abbr>` Replacement

**Setup guide**: Save as `abbr-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Abbr Replacement</title>
</head>
<body>
  <p>The <abbr title="World Wide Web Consortium">W3C</abbr> sets web standards.</p>
</body>
</html>
```

**Expected output**: The text renders, and hovering over “W3C” shows the expansion as a tooltip.

**Why this output occurs**: `<abbr>` is the modern, conforming element for abbreviations and acronyms.

---

### Real-World Cases

- **Legacy documentation**: Some older pages still use `<acronym>`.
- **Modern replacement**: `<abbr>` is used universally for abbreviations and acronyms.

---

### References

- WHATWG — Obsolete features: `acronym` — https://html.spec.whatwg.org/multipage/obsolete.html#acronym
- MDN Web Docs — `<abbr>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/abbr

---

## 3. `<bgsound>` — The Background Sound Element

### Definitions

**Core Definition**
The `<bgsound>` element was used to play a background sound file when a page loaded. It has been replaced by `<audio>`.

**Technical Definition**
`<bgsound>` is a non-conforming void element. The WHATWG specification states: “Use `audio` instead”. It was an Internet Explorer–specific element that never became a standard.

**Beginner-Friendly Explanation**
`<bgsound>` played music in the background of a web page. It is obsolete; use `<audio>` instead.

---

### Purposes

- **Historically: to play background audio** automatically.
- **Modern guidance: use `<audio>`** with appropriate controls and user consent.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<bgsound src="music.mid" loop="infinite">
```

#### Modern Replacement

```html
<audio controls loop>
  <source src="music.mp3" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the audio file. |
| `loop` | Whether to repeat. |

#### Constraints and Limitations

- `<bgsound>` is non-conforming and non-standard.
- Autoplaying audio is discouraged for accessibility and user-experience reasons.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete Background Sound

**Setup guide**: Save as `bgsound-obsolete.html`. This will not play in modern browsers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete BGSound</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <bgsound src="background.mid" loop="infinite">
</body>
</html>
```

**Expected output**: No audio plays in modern browsers.

**Why this output occurs**: `<bgsound>` was never standardised and is not supported.

---

#### Example 2: Modern Audio Replacement

**Setup guide**: Save as `audio-replacement.html`. Provide an audio file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Audio Replacement</title>
</head>
<body>
  <audio controls loop>
    <source src="background.mp3" type="audio/mpeg">
    Your browser does not support the audio element.
  </audio>
</body>
</html>
```

**Expected output**: An audio player with controls and looping enabled.

**Why this output occurs**: `<audio>` is the standard, accessible element for audio playback.

---

### Real-World Cases

- **Legacy pages**: Some very old pages used `<bgsound>`.
- **Modern practice**: Background audio is generally discouraged; if used, `<audio>` with user controls is preferred.

---

### References

- WHATWG — Obsolete features: `bgsound` — https://html.spec.whatwg.org/multipage/obsolete.html#bgsound
- MDN Web Docs — `<audio>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/audio

---

## 4. `<big>` — The Big Text Element

### Definitions

**Core Definition**
The `<big>` element rendered text at one size larger than the surrounding text. It has been replaced by CSS `font-size`.

**Technical Definition**
`<big>` is a non-conforming presentational element. The WHATWG obsolete features list and multiple lint rules explicitly include `big` among obsolete elements, recommending CSS instead. It was purely presentational and had no semantic meaning.

**Beginner-Friendly Explanation**
`<big>` made text bigger. Use CSS `font-size` instead.

---

### Purposes

- **Historically: to increase text size** for emphasis or visual hierarchy.
- **Modern guidance: use CSS `font-size`** on a semantic element.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<p>This is <big>larger text</big>.</p>
```

#### Modern Replacement

```html
<p>This is <span style="font-size: larger;">larger text</span>.</p>
```

Or, preferably:

```css
.larger { font-size: 1.2em; }
```

#### Constraints and Limitations

- `<big>` is non-conforming; use CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<big>`

**Setup guide**: Save as `big-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Big</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p>This is <big>larger text</big>.</p>
</body>
</html>
```

**Expected output**: The text may render slightly larger in some browsers, but the markup is non-conforming.

**Why this output occurs**: Browsers may apply a default style for backwards compatibility, but the element is obsolete.

---

#### Example 2: Modern CSS Replacement

**Setup guide**: Save as `big-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Big Replacement</title>
  <style>
    .larger { font-size: 1.2em; }
  </style>
</head>
<body>
  <p>This is <span class="larger">larger text</span>.</p>
</body>
</html>
```

**Expected output**: The text renders at 1.2em, controlled entirely by CSS.

**Why this output occurs**: CSS provides precise, maintainable control over font size.

---

### Real-World Cases

- **Legacy pages**: `<big>` appears in old HTML.
- **Modern practice**: CSS `font-size` is used everywhere.

---

### References

- WHATWG — Obsolete features: `big` (in non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features
- Siteimprove — Deprecated HTML elements — https://help.siteimprove.com/support/solutions/articles/80000448651-accessibility-rule-html-element-is-deprecated-or-obsolete-explained

---

## 5. `<blink>` — The Blinking Text Element

### Definitions

**Core Definition**
The `<blink>` element caused text to blink on and off. It is obsolete and fails WCAG accessibility criteria.

**Technical Definition**
`<blink>` was a non-standard element originally introduced by Netscape. It is explicitly listed as obsolete and has no modern browser support. W3C Technique F47 states that using the blink element fails WCAG Success Criterion 2.2.2 (Pause, Stop, Hide) because blinking can continue for more than three seconds.

**Beginner-Friendly Explanation**
`<blink>` made text flash on and off. It is terrible for accessibility and has been removed from HTML. Never use it.

---

### Purposes

- **Historically: to draw attention** through blinking.
- **Modern guidance: do not use**; use static emphasis (`<strong>`, `<em>`) or CSS animations with user controls.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<blink>Important text</blink>
```

#### Constraints and Limitations

- Non-standard, obsolete, and inaccessible.
- Violates WCAG 2.2.2.
- No modern browser support.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<blink>` (For Reference Only)

**Setup guide**: This will not blink in modern browsers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Blink</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p><blink>This text used to blink.</blink></p>
</body>
</html>
```

**Expected output**: The text renders normally in modern browsers; no blinking occurs.

**Why this output occurs**: Browser support for `<blink>` has been removed.

---

#### Example 2: Modern Accessible Alternative

**Setup guide**: Save as `blink-replacement.html`. If emphasis is needed, use `<strong>`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Blink Replacement</title>
</head>
<body>
  <p><strong>This text is emphasised without blinking.</strong></p>
</body>
</html>
```

**Expected output**: The text is bold and static.

**Why this output occurs**: `<strong>` conveys importance semantically without the accessibility problems of blinking.

---

### Real-World Cases

- **Legacy 1990s pages**: `<blink>` was used for attention.
- **Modern practice**: Never use blinking; use static emphasis or CSS animations with controls.

---

### References

- W3C — Technique F47: Failure of SC 2.2.2 due to using blink — https://www.w3.org/WAI/WCAG22/Techniques/failures/F47
- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features

---

## 6. `<center>` — The Centered Text Element

### Definitions

**Core Definition**
The `<center>` element was used to center text and block-level content horizontally. It has been replaced by CSS `text-align: center` and margin auto techniques.

**Technical Definition**
`<center>` is a non-conforming presentational element. MDN states that it was deprecated in HTML4 in favour of the CSS `text-align` property, which can be applied to a `<div>` or individual `<p>` elements. For centering block-level elements, CSS properties such as `margin-left` and `margin-right` set to `auto` (or `margin: 0 auto`) should be used.

**Beginner-Friendly Explanation**
`<center>` centered content. Use CSS instead: `text-align: center` for inline content and `margin: 0 auto` for block elements.

---

### Purposes

- **Historically: to center text and blocks**.
- **Modern guidance: use CSS** `text-align: center` and margin auto.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<center>This text is centered.</center>
```

#### Modern Replacement

```html
<div style="text-align: center;">This text is centered.</div>
```

For block centering:

```css
.centered-block {
  margin-left: auto;
  margin-right: auto;
  width: 80%;
}
```

#### Constraints and Limitations

- `<center>` is non-conforming; use CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<center>`

**Setup guide**: Save as `center-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Center</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <center>This text is centered.</center>
</body>
</html>
```

**Expected output**: The text is centered in most browsers, but the markup is non-conforming.

**Why this output occurs**: Browsers retain backwards-compatible styling for `<center>`.

---

#### Example 2: Modern CSS Centering

**Setup guide**: Save as `center-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Center Replacement</title>
  <style>
    .center-text { text-align: center; }
    .center-block { margin: 0 auto; width: 60%; background: #eee; }
  </style>
</head>
<body>
  <p class="center-text">This text is centered with CSS.</p>
  <div class="center-block">This block is centered with margin auto.</div>
</body>
</html>
```

**Expected output**: The text is centered, and the block is horizontally centered.

**Why this output occurs**: CSS provides precise, maintainable centering without presentational HTML.

---

### Real-World Cases

- **Legacy pages**: `<center>` is common in old HTML.
- **Modern practice**: CSS `text-align` and margin auto are standard.

---

### References

- MDN Web Docs — `<center>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/center
- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features

---

## 7. `<dir>` — The Directory List Element

### Definitions

**Core Definition**
The `<dir>` element was intended to represent a list of directory or folder names. It has been replaced by `<ul>`.

**Technical Definition**
`<dir>` is a non-conforming element. The WHATWG specification states: “Use `ul` instead”. It was similar to `<ul>` but intended for directory listings.

**Beginner-Friendly Explanation**
`<dir>` was an old way to make a list, originally for directories. Use `<ul>` instead.

---

### Purposes

- **Historically: to list directory contents**.
- **Modern guidance: use `<ul>`** for unordered lists.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<dir>
  <li>Folder 1</li>
  <li>Folder 2</li>
</dir>
```

#### Modern Replacement

```html
<ul>
  <li>Folder 1</li>
  <li>Folder 2</li>
</ul>
```

#### Constraints and Limitations

- `<dir>` is non-conforming; use `<ul>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<dir>`

**Setup guide**: Save as `dir-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Dir</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <dir>
    <li>Documents</li>
    <li>Downloads</li>
  </dir>
</body>
</html>
```

**Expected output**: The browser may render a bulleted list, but the markup is non-conforming.

**Why this output occurs**: Browsers may alias `<dir>` to `<ul>` for compatibility.

---

#### Example 2: Modern `<ul>` Replacement

**Setup guide**: Save as `ul-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>UL Replacement</title>
</head>
<body>
  <ul>
    <li>Documents</li>
    <li>Downloads</li>
  </ul>
</body>
</html>
```

**Expected output**: A standard bulleted list.

**Why this output occurs**: `<ul>` is the correct semantic element for unordered lists.

---

### Real-World Cases

- **Legacy pages**: `<dir>` appears in old HTML.
- **Modern practice**: `<ul>` is universally used.

---

### References

- WHATWG — Obsolete features: `dir` — https://html.spec.whatwg.org/multipage/obsolete.html#dir
- MDN Web Docs — `<dir>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dir

---

## 8. `<font>` — The Font Styling Element

### Definitions

**Core Definition**
The `<font>` element was used to set the font face, size, and color of text. It has been replaced by CSS font properties.

**Technical Definition**
`<font>` is a non-conforming presentational element. MDN states: “For all new web development, styling should be written using CSS only. The former behavior of the `<font>` element can be achieved, and even better controlled, by using CSS Fonts properties”. The CSS replacements include `font-family`, `font-size`, and `color`.

**Beginner-Friendly Explanation**
`<font>` let you change how text looked — its typeface, size, and colour. Use CSS instead: `font-family`, `font-size`, and `color`.

---

### Purposes

- **Historically: to style text** directly in HTML.
- **Modern guidance: use CSS** font and color properties.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<font face="Arial" size="4" color="red">Styled text</font>
```

#### Modern Replacement

```css
.styled {
  font-family: Arial, sans-serif;
  font-size: 1.2em;
  color: red;
}
```

```html
<span class="styled">Styled text</span>
```

#### Constraints and Limitations

- `<font>` is non-conforming; use CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<font>`

**Setup guide**: Save as `font-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Font</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p><font face="Arial" size="4" color="red">Styled text</font></p>
</body>
</html>
```

**Expected output**: The text may render with the specified font, size, and colour in some browsers, but the markup is non-conforming.

**Why this output occurs**: Browsers retain backwards-compatible styling for `<font>`.

---

#### Example 2: Modern CSS Replacement

**Setup guide**: Save as `font-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Font Replacement</title>
  <style>
    .styled {
      font-family: Arial, sans-serif;
      font-size: 1.2em;
      color: red;
    }
  </style>
</head>
<body>
  <p><span class="styled">Styled text</span></p>
</body>
</html>
```

**Expected output**: The text renders with Arial, larger size, and red colour.

**Why this output occurs**: CSS provides clean separation of presentation from structure.

---

### Real-World Cases

- **Legacy pages**: `<font>` was ubiquitous in 1990s and early 2000s HTML.
- **Modern practice**: CSS is the only appropriate way to style text.

---

### References

- MDN Web Docs — `<font>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/font
- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features

---

## 9. `<frame>` — The Frame Element

### Definitions

**Core Definition**
The `<frame>` element defined a single frame within a `<frameset>`. It has been replaced by `<iframe>` and CSS layout.

**Technical Definition**
`<frame>` is a non-conforming element. The WHATWG specification groups `frame`, `frameset`, and `noframes` together and states: “Either use `iframe` and CSS instead, or use server-side includes to generate complete pages with the various invariant parts merged in”. Framesets were used for page layout before CSS was capable of complex layouts.

**Beginner-Friendly Explanation**
`<frame>` was part of an old system for dividing a browser window into multiple independent panels. Use `<iframe>` for embedded documents and CSS Grid/Flexbox for layout instead.

---

### Purposes

- **Historically: to divide the browser window** into multiple scrollable frames.
- **Modern guidance: use `<iframe>`** for embedded documents and CSS layout for page structure.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<frameset cols="25%,75%">
  <frame src="menu.html" name="menu">
  <frame src="content.html" name="content">
</frameset>
```

#### Modern Replacement

```html
<div class="layout">
  <iframe src="menu.html" title="Menu"></iframe>
  <iframe src="content.html" title="Content"></iframe>
</div>
```

```css
.layout {
  display: grid;
  grid-template-columns: 25% 75%;
}
```

#### Constraints and Limitations

- `<frame>` and `<frameset>` are non-conforming; use `<iframe>` and CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete Frameset (For Reference Only)

**Setup guide**: This will not work in modern browsers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Frameset</title>
</head>
<frameset cols="25%,75%">
  <frame src="menu.html" name="menu">
  <frame src="content.html" name="content">
  <noframes>
    <p>Your browser does not support frames.</p>
  </noframes>
</frameset>
</html>
```

**Expected output**: Modern browsers may partially render the frames or ignore the frameset entirely.

**Why this output occurs**: Frames are obsolete; browser support is inconsistent.

---

#### Example 2: Modern Iframe and CSS Layout

**Setup guide**: Save as `iframe-layout.html`. Provide `menu.html` and `content.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Iframe Layout</title>
  <style>
    .layout {
      display: grid;
      grid-template-columns: 25% 75%;
      height: 100vh;
    }
    iframe { border: none; width: 100%; height: 100%; }
  </style>
</head>
<body>
  <div class="layout">
    <iframe src="menu.html" title="Navigation menu"></iframe>
    <iframe src="content.html" title="Main content"></iframe>
  </div>
</body>
</html>
```

**Expected output**: A two-column layout with each iframe occupying its grid cell.

**Why this output occurs**: CSS Grid provides the layout structure, and `<iframe>` provides the embedded documents. This is the modern, conforming replacement for framesets.

---

### Real-World Cases

- **Legacy sites**: Framesets were common in the late 1990s.
- **Modern practice**: CSS Grid/Flexbox for layout, `<iframe>` for embedding third-party content.

---

### References

- WHATWG — Obsolete features: `frame`, `frameset`, `noframes` — https://html.spec.whatwg.org/multipage/obsolete.html#frames
- MDN Web Docs — `<frame>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/frame

---

## 10. `<frameset>` — The Frameset Element

### Definitions

**Core Definition**
The `<frameset>` element was used to define a set of frames that divided the browser window. It has been replaced by CSS layout and `<iframe>`.

**Technical Definition**
`<frameset>` is a non-conforming element. The WHATWG specification states: “Use `iframe` and CSS instead, or use server-side includes”.

**Beginner-Friendly Explanation**
`<frameset>` was the container that held frames. Use CSS layout instead.

---

### Purposes

- **Historically: to define a grid of frames**.
- **Modern guidance: use CSS Grid/Flexbox** and `<iframe>` where embedding is needed.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<frameset rows="100,*">
  <frame src="header.html">
  <frame src="body.html">
</frameset>
```

#### Modern Replacement

```html
<header>…</header>
<main>…</main>
```

With CSS grid or flexbox for layout.

#### Constraints and Limitations

- Non-conforming; use CSS and `<iframe>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete Frameset

**Setup guide**: For reference only.

```html
<frameset rows="100,*">
  <frame src="header.html">
  <frame src="main.html">
</frameset>
```

**Expected output**: Not rendered in modern browsers.

**Why this output occurs**: Framesets are obsolete.

---

#### Example 2: Modern Layout Replacement

**Setup guide**: Save as `layout-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Modern Layout</title>
  <style>
    body { margin: 0; display: flex; flex-direction: column; height: 100vh; }
    header { height: 100px; background: #eee; }
    main { flex: 1; }
  </style>
</head>
<body>
  <header>Header content</header>
  <main>Main content</main>
</body>
</html>
```

**Expected output**: A header at the top and main content filling the rest of the page.

**Why this output occurs**: CSS Flexbox provides a clean, responsive layout without frames.

---

### Real-World Cases

- **Legacy pages**: Framesets are found in old intranet sites.
- **Modern practice**: CSS layout is standard.

---

### References

- WHATWG — Obsolete features: frames — https://html.spec.whatwg.org/multipage/obsolete.html#frames
- MDN Web Docs — `<frameset>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/frameset

---

## 11. `<isindex>` — The Isindex Element

### Definitions

**Core Definition**
The `<isindex>` element was used to create a single-line text input that submitted a query to the server. It has been replaced by explicit form controls.

**Technical Definition**
`<isindex>` is a non-conforming element. The WHATWG specification states: “Use an explicit `form` and text control combination instead”.

**Beginner-Friendly Explanation**
`<isindex>` was an old, limited way to create a search box. Use `<form>` with `<input>` and `<button>` instead.

---

### Purposes

- **Historically: to create a simple query prompt**.
- **Modern guidance: use `<form>` and `<input>`**.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<isindex prompt="Search: " action="/search">
```

#### Modern Replacement

```html
<form action="/search" method="get">
  <label for="q">Search:</label>
  <input type="search" id="q" name="q">
  <button type="submit">Go</button>
</form>
```

#### Constraints and Limitations

- Non-conforming; use explicit form controls.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<isindex>`

**Setup guide**: For reference only.

```html
<isindex prompt="Search: " action="/search">
```

**Expected output**: Not rendered in modern browsers.

**Why this output occurs**: `<isindex>` is obsolete.

---

#### Example 2: Modern Search Form

**Setup guide**: Save as `search-form.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Search Form</title>
</head>
<body>
  <form action="/search" method="get">
    <label for="q">Search:</label>
    <input type="search" id="q" name="q">
    <button type="submit">Go</button>
  </form>
</body>
</html>
```

**Expected output**: A labelled search input and submit button.

**Why this output occurs**: Explicit form controls provide full accessibility and customisation.

---

### Real-World Cases

- **Legacy pages**: `<isindex>` is very old and rare.
- **Modern practice**: Standard forms are universal.

---

### References

- WHATWG — Obsolete features: `isindex` — https://html.spec.whatwg.org/multipage/obsolete.html#isindex
- MDN Web Docs — `<isindex>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/isindex

---

## 12. `<keygen>` — The Key Pair Generator Element

### Definitions

**Core Definition**
The `<keygen>` element was used to generate a key pair for certificate enrollment. It has been replaced by the Web Cryptography API.

**Technical Definition**
`<keygen>` is a non-conforming element. The WHATWG specification states: “For enterprise device management use cases, use native on-device management capabilities. For certificate enrollment use cases, use the Web Cryptography API to generate a keypair for the certificate, and then export the certificate and key to allow the user to install them manually”. The modern replacement is `crypto.subtle.generateKey()`.

**Beginner-Friendly Explanation**
`<keygen>` generated encryption keys for security certificates. It is obsolete; use the Web Cryptography API in JavaScript instead.

---

### Purposes

- **Historically: to generate key pairs** for client certificates.
- **Modern guidance: use `crypto.subtle.generateKey()`** from the Web Cryptography API.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<keygen name="key" challenge="challenge-string" keytype="RSA">
```

#### Modern Replacement

```javascript
const keyPair = await crypto.subtle.generateKey(
  { name: "RSASSA-PKCS1-v1_5", modulusLength: 2048, publicExponent: new Uint8Array([1, 0, 1]), hash: "SHA-256" },
  true,
  ["sign", "verify"]
);
```

#### Constraints and Limitations

- `<keygen>` is non-conforming; use Web Crypto.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<keygen>`

**Setup guide**: For reference only.

```html
<form action="/enroll" method="post">
  <keygen name="key" challenge="abc123">
  <button type="submit">Enroll</button>
</form>
```

**Expected output**: Not supported in modern browsers.

**Why this output occurs**: `<keygen>` has been removed.

---

#### Example 2: Modern Web Crypto Key Generation

**Setup guide**: Save as `webcrypto-keygen.html`. Requires a secure context (HTTPS).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Web Crypto Key Generation</title>
</head>
<body>
  <button type="button" onclick="generateKey()">Generate Key Pair</button>
  <pre id="output"></pre>

  <script>
    async function generateKey() {
      try {
        const keyPair = await crypto.subtle.generateKey(
          { name: "RSASSA-PKCS1-v1_5", modulusLength: 2048, publicExponent: new Uint8Array([1, 0, 1]), hash: "SHA-256" },
          true,
          ["sign", "verify"]
        );
        document.getElementById('output').textContent = 'Key pair generated: ' + keyPair.privateKey.algorithm.name;
      } catch (e) {
        document.getElementById('output').textContent = 'Error: ' + e.message;
      }
    }
  </script>
</body>
</html>
```

**Expected output**: Clicking the button generates an RSA key pair and displays a confirmation message.

**Why this output occurs**: `crypto.subtle.generateKey()` is the standard, secure replacement for `<keygen>`.

---

### Real-World Cases

- **Legacy certificate enrollment**: Old enterprise systems used `<keygen>`.
- **Modern practice**: Web Crypto API is used for client-side key generation.

---

### References

- WHATWG — Obsolete features: `keygen` — https://html.spec.whatwg.org/multipage/obsolete.html#keygen
- MDN Web Docs — Web Crypto API — https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API

---

## 13. `<marquee>` — The Scrolling Text Element

### Definitions

**Core Definition**
The `<marquee>` element created scrolling text that moved across the page. It is obsolete and has significant accessibility problems.

**Technical Definition**
`<marquee>` is a non-conforming element. It is grouped with `<blink>` as a distracting element that can cause problems for users with attention disorders, vestibular disorders, or seizure disorders. WCAG Success Criterion 2.2.2 requires a mechanism to pause, stop, or hide moving content. The modern replacement is CSS animations with appropriate controls.

**Beginner-Friendly Explanation**
`<marquee>` made text scroll across the screen. It is obsolete and inaccessible. Use CSS animations if you need moving text, and always provide a way to pause or stop the motion.

---

### Purposes

- **Historically: to create scrolling or moving text**.
- **Modern guidance: use CSS animations** with user controls, or static content.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<marquee behavior="scroll" direction="left">Scrolling text</marquee>
```

#### Modern Replacement

```css
@keyframes scroll {
  from { transform: translateX(100%); }
  to { transform: translateX(-100%); }
}
.marquee {
  overflow: hidden;
  white-space: nowrap;
  animation: scroll 10s linear infinite;
}
```

#### Constraints and Limitations

- Non-conforming and inaccessible.
- Violates WCAG 2.2.2 if motion lasts more than three seconds without a pause mechanism.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<marquee>`

**Setup guide**: Save as `marquee-obsolete.html`. May still scroll in some browsers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Marquee</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <marquee behavior="scroll" direction="left">Scrolling text</marquee>
</body>
</html>
```

**Expected output**: Some browsers may still render scrolling text for compatibility, but the markup is non-conforming and inaccessible.

**Why this output occurs**: Browsers retain partial support for legacy reasons.

---

#### Example 2: Modern CSS Animation with Controls

**Setup guide**: Save as `marquee-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Moving Text</title>
  <style>
    @keyframes scroll {
      from { transform: translateX(100%); }
      to { transform: translateX(-100%); }
    }
    .marquee {
      overflow: hidden;
      white-space: nowrap;
      width: 100%;
    }
    .marquee span {
      display: inline-block;
      animation: scroll 10s linear infinite;
    }
    .paused span { animation-play-state: paused; }
  </style>
</head>
<body>
  <div class="marquee" id="marquee">
    <span>This text scrolls, but you can pause it.</span>
  </div>
  <button type="button" onclick="document.getElementById('marquee').classList.toggle('paused')">
    Pause / Resume
  </button>
</body>
</html>
```

**Expected output**: The text scrolls continuously, and the button allows the user to pause and resume the animation.

**Why this output occurs**: CSS animations provide the visual effect, and the pause/resume control satisfies WCAG 2.2.2.

---

### Real-World Cases

- **Legacy pages**: `<marquee>` was common in the 1990s and 2000s.
- **Modern practice**: CSS animations with controls, or static content that does not distract.

---

### References

- W3C — Technique F47 (blink) and SCR36 (scrolling text) — https://www.w3.org/WAI/WCAG22/Techniques/failures/F47
- WHATWG — Obsolete features (marquee in non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features

---

## 14. `<menuitem>` — The Menu Item Element

### Definitions

**Core Definition**
The `<menuitem>` element was intended to represent a command in a context menu or toolbar. It has been removed from HTML.

**Technical Definition**
`<menuitem>` is a non-conforming element. The WHATWG specification states: “To implement a custom context menu, use script to handle the `contextmenu` event”. The element was removed because the native context-menu API it was designed for was never fully implemented across browsers.

**Beginner-Friendly Explanation**
`<menuitem>` was supposed to create items in a right-click menu. It is obsolete; use JavaScript to build custom context menus if needed.

---

### Purposes

- **Historically: to define context menu items**.
- **Modern guidance: use JavaScript** and the `contextmenu` event, or standard HTML menus.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<menu type="context" id="myMenu">
  <menuitem label="Cut" command="cut"></menuitem>
  <menuitem label="Copy" command="copy"></menuitem>
</menu>
```

#### Modern Replacement

```javascript
document.addEventListener('contextmenu', function (e) {
  e.preventDefault();
  // Build and show a custom menu with DOM APIs
});
```

#### Constraints and Limitations

- Non-conforming; browser support was never complete.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<menuitem>`

**Setup guide**: For reference only.

```html
<menu type="context">
  <menuitem label="Refresh"></menuitem>
</menu>
```

**Expected output**: Not rendered as a context menu in modern browsers.

**Why this output occurs**: `<menuitem>` was never widely supported and is now obsolete.

---

#### Example 2: Modern Custom Context Menu

**Setup guide**: Save as `contextmenu-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Custom Context Menu</title>
  <style>
    #custom-menu {
      position: absolute;
      display: none;
      background: white;
      border: 1px solid #ccc;
      box-shadow: 2px 2px 6px rgba(0,0,0,0.2);
    }
    #custom-menu button { display: block; width: 100%; border: none; padding: 8px; text-align: left; }
  </style>
</head>
<body>
  <div id="custom-menu">
    <button type="button" onclick="alert('Refresh clicked')">Refresh</button>
    <button type="button" onclick="alert('Save clicked')">Save</button>
  </div>

  <script>
    document.addEventListener('contextmenu', function (e) {
      e.preventDefault();
      const menu = document.getElementById('custom-menu');
      menu.style.display = 'block';
      menu.style.left = e.pageX + 'px';
      menu.style.top = e.pageY + 'px';
    });
    document.addEventListener('click', function () {
      document.getElementById('custom-menu').style.display = 'none';
    });
  </script>
</body>
</html>
```

**Expected output**: Right-clicking shows a custom menu with “Refresh” and “Save” options.

**Why this output occurs**: JavaScript and DOM APIs provide the context-menu behaviour that `<menuitem>` was intended for.

---

### Real-World Cases

- **Legacy applications**: Rare instances of `<menuitem>` exist in old code.
- **Modern practice**: Custom context menus are built with JavaScript.

---

### References

- WHATWG — Obsolete features: `menuitem` — https://html.spec.whatwg.org/multipage/obsolete.html#menuitem
- MDN Web Docs — `<menuitem>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/menuitem

---

## 15. `<nobr>` — The No Line Break Element

### Definitions

**Core Definition**
The `<nobr>` element prevented text from wrapping to a new line. It has been replaced by CSS `white-space: nowrap`.

**Technical Definition**
`<nobr>` is a non-conforming element. It is listed among obsolete presentational elements, with CSS recommended as the replacement.

**Beginner-Friendly Explanation**
`<nobr>` stopped text from wrapping. Use CSS `white-space: nowrap` instead.

---

### Purposes

- **Historically: to prevent line wrapping**.
- **Modern guidance: use CSS `white-space: nowrap`**.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<nobr>This text will not wrap.</nobr>
```

#### Modern Replacement

```html
<span style="white-space: nowrap;">This text will not wrap.</span>
```

#### Constraints and Limitations

- Non-conforming; use CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<nobr>`

**Setup guide**: Save as `nobr-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Nobr</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p><nobr>This text will not wrap to the next line.</nobr></p>
</body>
</html>
```

**Expected output**: The text may not wrap in some browsers, but the markup is non-conforming.

**Why this output occurs**: Browsers may apply the legacy behaviour for compatibility.

---

#### Example 2: Modern CSS Replacement

**Setup guide**: Save as `nobr-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nobr Replacement</title>
  <style>
    .nowrap { white-space: nowrap; }
  </style>
</head>
<body>
  <p><span class="nowrap">This text will not wrap to the next line.</span></p>
</body>
</html>
```

**Expected output**: The text does not wrap, controlled by CSS.

**Why this output occurs**: `white-space: nowrap` is the standard, conforming way to prevent wrapping.

---

### Real-World Cases

- **Legacy pages**: `<nobr>` appears in old HTML.
- **Modern practice**: CSS `white-space` is standard.

---

### References

- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features
- Siteimprove — Deprecated HTML elements — https://help.siteimprove.com/support/solutions/articles/80000448651-accessibility-rule-html-element-is-deprecated-or-obsolete-explained

---

## 16. `<noembed>` — The Noembed Fallback Element

### Definitions

**Core Definition**
The `<noembed>` element provided fallback content for browsers that did not support the `<embed>` element. It has been replaced by `<object>`.

**Technical Definition**
`<noembed>` is a non-conforming element. The WHATWG specification states: “Use `object` instead of `embed` when fallback is necessary”.

**Beginner-Friendly Explanation**
`<noembed>` showed fallback content when `<embed>` was not supported. Use `<object>` instead, which has built-in fallback support.

---

### Purposes

- **Historically: to provide fallback for `<embed>`**.
- **Modern guidance: use `<object>`** with fallback content.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<embed src="file.swf" type="application/x-shockwave-flash">
<noembed>
  <p>Your browser does not support embedded content.</p>
</noembed>
```

#### Modern Replacement

```html
<object data="file.svg" type="image/svg+xml">
  <p>Your browser cannot display this SVG. <a href="file.svg">Download it</a>.</p>
</object>
```

#### Constraints and Limitations

- Non-conforming; use `<object>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<noembed>`

**Setup guide**: For reference only.

```html
<embed src="movie.swf" type="application/x-shockwave-flash">
<noembed>
  <p>Your browser does not support embedded content.</p>
</noembed>
```

**Expected output**: The fallback may or may not be shown depending on browser support.

**Why this output occurs**: `<noembed>` is obsolete and inconsistent.

---

#### Example 2: Modern `<object>` Fallback

**Setup guide**: Save as `object-fallback.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Object Fallback</title>
</head>
<body>
  <object data="diagram.svg" type="image/svg+xml">
    <p>Your browser cannot display the diagram. <a href="diagram.svg">Download it</a>.</p>
  </object>
</body>
</html>
```

**Expected output**: The SVG is displayed if possible; otherwise, the fallback paragraph is shown.

**Why this output occurs**: `<object>` natively supports fallback content.

---

### Real-World Cases

- **Legacy plugin content**: `<noembed>` was used with Flash and similar plugins.
- **Modern practice**: `<object>` provides reliable fallback.

---

### References

- WHATWG — Obsolete features: `noembed` — https://html.spec.whatwg.org/multipage/obsolete.html#noembed
- MDN Web Docs — `<object>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/object

---

## 17. `<noframes>` — The Noframes Fallback Element

### Definitions

**Core Definition**
The `<noframes>` element provided fallback content for browsers that did not support frames. It has been replaced by modern layout techniques.

**Technical Definition**
`<noframes>` is a non-conforming element. It is grouped with `<frame>` and `<frameset>` and replaced by iframes, CSS layout, or server-side includes.

**Beginner-Friendly Explanation**
`<noframes>` showed content when the browser did not support frames. Frames are obsolete, so `<noframes>` is obsolete too.

---

### Purposes

- **Historically: fallback for frameset-based pages**.
- **Modern guidance: use CSS layout and `<iframe>`**; no fallback element is needed.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<frameset cols="50%,50%">
  <frame src="left.html">
  <frame src="right.html">
  <noframes>
    <p>Your browser does not support frames.</p>
  </noframes>
</frameset>
```

#### Modern Replacement

Use CSS Grid or Flexbox for layout and `<iframe>` only for embedding external content. There is no direct replacement element because the entire frameset model is obsolete.

#### Constraints and Limitations

- Non-conforming; frames are obsolete.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<noframes>`

**Setup guide**: For reference only.

```html
<frameset cols="50%,50%">
  <frame src="a.html">
  <frame src="b.html">
  <noframes>
    <p>Frames are not supported.</p>
  </noframes>
</frameset>
```

**Expected output**: Not rendered in modern browsers.

**Why this output occurs**: The entire frameset model is obsolete.

---

#### Example 2: Modern Layout Without Frames

**Setup guide**: Save as `modern-layout.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Modern Layout</title>
  <style>
    .container { display: flex; }
    .column { flex: 1; padding: 16px; }
    .column:first-child { background: #f0f0f0; }
  </style>
</head>
<body>
  <div class="container">
    <div class="column">Left content</div>
    <div class="column">Right content</div>
  </div>
</body>
</html>
```

**Expected output**: Two side-by-side columns of content.

**Why this output occurs**: Flexbox provides the layout without frames.

---

### Real-World Cases

- **Legacy pages**: Framesets and `<noframes>` are found in very old sites.
- **Modern practice**: CSS layout is universal.

---

### References

- WHATWG — Obsolete features: frames — https://html.spec.whatwg.org/multipage/obsolete.html#frames
- MDN Web Docs — `<noframes>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/noframes

---

## 18. `<plaintext>` — The Plaintext Element

### Definitions

**Core Definition**
The `<plaintext>` element rendered all subsequent content as raw text, ignoring all HTML markup. It has been replaced by the `text/plain` MIME type.

**Technical Definition**
`<plaintext>` is a non-conforming element. The WHATWG specification states: “Use the `text/plain` MIME type instead”. It was a parsing directive rather than a content element.

**Beginner-Friendly Explanation**
`<plaintext>` stopped the browser from interpreting HTML after that point. Use a plain text file served with the `text/plain` MIME type instead.

---

### Purposes

- **Historically: to display raw text** without HTML parsing.
- **Modern guidance: serve text as `text/plain`** or use `<pre>` with escaped content.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<plaintext>
  <p>This tag and everything after it is shown as raw text.
```

#### Modern Replacement

Serve the file with HTTP header `Content-Type: text/plain`, or use `<pre>` with escaped characters.

#### Constraints and Limitations

- Non-conforming; use MIME types or `<pre>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<plaintext>`

**Setup guide**: For reference only.

```html
<plaintext>
<p>This will not be parsed as HTML.
```

**Expected output**: Everything after `<plaintext>` is shown as raw text in browsers that support it.

**Why this output occurs**: `<plaintext>` switches the parser to plain-text mode.

---

#### Example 2: Modern `<pre>` Replacement

**Setup guide**: Save as `plaintext-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Plaintext Replacement</title>
</head>
<body>
  <pre>
&lt;p&gt;This will be displayed as text, not parsed as HTML.&lt;/p&gt;
  </pre>
</body>
</html>
```

**Expected output**: The escaped HTML is displayed as literal text.

**Why this output occurs**: `<pre>` preserves whitespace, and escaping `<` and `>` prevents parsing.

---

### Real-World Cases

- **Legacy plain-text documents**: `<plaintext>` is extremely rare.
- **Modern practice**: Use `text/plain` or `<pre>` with escaping.

---

### References

- WHATWG — Obsolete features: `plaintext` — https://html.spec.whatwg.org/multipage/obsolete.html#plaintext
- MDN Web Docs — `<plaintext>` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/plaintext

---

## 19. `<rb>` — The Ruby Base Element

### Definitions

**Core Definition**
The `<rb>` element was intended to mark the base text component of a ruby annotation. It has been removed from HTML; ruby base text is now placed directly inside `<ruby>`.

**Technical Definition**
`<rb>` is a non-conforming element. The WHATWG specification states: “Providing the ruby base directly inside the `ruby` element or using nested `ruby` elements is sufficient”. It was part of an extended ruby model that was never widely implemented.

**Beginner-Friendly Explanation**
`<rb>` was an optional wrapper for the base characters in ruby annotations. It is no longer used; put the base text directly inside `<ruby>`.

---

### Purposes

- **Historically: to explicitly mark ruby base text**.
- **Modern guidance: place base text directly inside `<ruby>`**.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<ruby>
  <rb>漢</rb>
  <rt>kan</rt>
</ruby>
```

#### Modern Replacement

```html
<ruby>
  漢 <rt>kan</rt>
</ruby>
```

#### Constraints and Limitations

- Non-conforming; use direct ruby base text.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<rb>`

**Setup guide**: For reference only.

```html
<ruby>
  <rb>漢</rb><rt>kan</rt>
</ruby>
```

**Expected output**: May render similarly to modern ruby, but `<rb>` is obsolete.

**Why this output occurs**: Browsers may still parse `<rb>` for compatibility, but it is non-conforming.

---

#### Example 2: Modern Ruby Markup

**Setup guide**: Save as `ruby-modern.html`.

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Modern Ruby</title>
</head>
<body>
  <ruby>
    漢 <rt>kan</rt>
  </ruby>
</body>
</html>
```

**Expected output**: The base character with its ruby annotation above.

**Why this output occurs**: The modern ruby model places base text directly inside `<ruby>`.

---

### Real-World Cases

- **Legacy ruby markup**: `<rb>` is rare.
- **Modern practice**: Direct base text is standard.

---

### References

- WHATWG — Obsolete features: `rb`, `rtc` — https://html.spec.whatwg.org/multipage/obsolete.html#rb
- MDN Web Docs — `<ruby>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ruby

---

## 20. `<rtc>` — The Ruby Text Container Element

### Definitions

**Core Definition**
The `<rtc>` element was intended to group ruby text annotations. It has been removed from HTML.

**Technical Definition**
`<rtc>` is a non-conforming element, grouped with `<rb>` and replaced by simpler ruby markup.

**Beginner-Friendly Explanation**
`<rtc>` was a container for ruby annotations. It is obsolete; use `<ruby>` with `<rt>` directly.

---

### Purposes

- **Historically: to group multiple ruby annotations**.
- **Modern guidance: use direct `<rt>` inside `<ruby>`**.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<ruby>
  <rb>漢</rb>
  <rtc><rt>kan</rt></rtc>
</ruby>
```

#### Modern Replacement

```html
<ruby>
  漢 <rt>kan</rt>
</ruby>
```

#### Constraints and Limitations

- Non-conforming; use simpler ruby markup.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<rtc>`

**Setup guide**: For reference only.

```html
<ruby>
  <rb>漢</rb>
  <rtc><rt>kan</rt></rtc>
</ruby>
```

**Expected output**: May render, but non-conforming.

**Why this output occurs**: Browsers may parse it for compatibility.

---

#### Example 2: Modern Ruby

**Setup guide**: Save as `rtc-replacement.html`.

```html
<ruby>
  漢 <rt>kan</rt>
</ruby>
```

**Expected output**: Standard ruby annotation.

**Why this output occurs**: The modern model does not require `<rtc>`.

---

### Real-World Cases

- **Legacy ruby**: `<rtc>` is rare.
- **Modern practice**: Direct `<rt>` is standard.

---

### References

- WHATWG — Obsolete features: `rb`, `rtc` — https://html.spec.whatwg.org/multipage/obsolete.html#rb
- MDN Web Docs — `<ruby>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ruby

---

## 21. `<spacer>` — The Layout Spacer Element

### Definitions

**Core Definition**
The `<spacer>` element was a Netscape-specific element used to insert blank space into a layout. It has been replaced by CSS margin and padding.

**Technical Definition**
`<spacer>` is a non-conforming, non-standard element. W3C Technique C18 recommends using CSS margin and padding rules instead of spacer images or spacer elements for layout design.

**Beginner-Friendly Explanation**
`<spacer>` added empty space between elements. Use CSS `margin` or `padding` instead.

---

### Purposes

- **Historically: to insert spacing** in table-based or spacer-image layouts.
- **Modern guidance: use CSS `margin` and `padding`**.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<spacer type="horizontal" size="20">
```

#### Modern Replacement

```css
.spaced { margin-left: 20px; }
```

#### Constraints and Limitations

- Non-standard and non-conforming.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<spacer>`

**Setup guide**: For reference only.

```html
<p>Text <spacer type="horizontal" size="20"> more text.</p>
```

**Expected output**: The `<spacer>` is ignored in modern browsers.

**Why this output occurs**: It was never standard and is not supported.

---

#### Example 2: Modern CSS Spacing

**Setup guide**: Save as `spacing-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Spacing</title>
  <style>
    .spaced { margin-left: 20px; }
  </style>
</head>
<body>
  <p>Text <span class="spaced">more text.</span></p>
</body>
</html>
```

**Expected output**: The second part of the text is shifted 20px to the right.

**Why this output occurs**: CSS margin provides precise, maintainable spacing.

---

### Real-World Cases

- **Legacy Netscape-era pages**: `<spacer>` was used for layout.
- **Modern practice**: CSS margin and padding are standard.

---

### References

- W3C — Technique C18: Using CSS margin and padding — https://www.w3.org/WAI/WCAG22/Techniques/css/C18
- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features

---

## 22. `<strike>` — The Strikethrough Element

### Definitions

**Core Definition**
The `<strike>` element rendered text with a strikethrough line. It has been replaced by `<s>` (for no-longer-accurate content) and `<del>` (for deletions).

**Technical Definition**
`<strike>` is a non-conforming presentational element. HTML5 provides two semantic alternatives: `<s>` for content that is no longer accurate or relevant, and `<del>` for content that has been deleted from the document.

**Beginner-Friendly Explanation**
`<strike>` drew a line through text. Use `<s>` for outdated information or `<del>` for deleted content instead.

---

### Purposes

- **Historically: to strike through text** for emphasis or correction.
- **Modern guidance: use `<s>` or `<del>`** depending on meaning.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<strike>Old price</strike>
```

#### Modern Replacement

```html
<s>Old price</s> New price
```

Or for deletions:

```html
<del>Removed text</del>
```

#### Constraints and Limitations

- Non-conforming; use semantic alternatives.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<strike>`

**Setup guide**: Save as `strike-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete Strike</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p><strike>Old price: $50</strike> New price: $35</p>
</body>
</html>
```

**Expected output**: The old price is struck through, but the markup is non-conforming.

**Why this output occurs**: Browsers may render it for compatibility.

---

#### Example 2: Modern `<s>` Replacement

**Setup guide**: Save as `s-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>S Replacement</title>
</head>
<body>
  <p><s>Old price: $50</s> New price: $35</p>
</body>
</html>
```

**Expected output**: The old price is struck through, and the markup is conforming.

**Why this output occurs**: `<s>` is the correct semantic element for no-longer-accurate content.

---

### Real-World Cases

- **Legacy pages**: `<strike>` is common in old HTML.
- **Modern practice**: `<s>` and `<del>` are used based on meaning.

---

### References

- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features
- MDN Web Docs — `<s>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/s

---

## 23. `<tt>` — The Teletype Text Element

### Definitions

**Core Definition**
The `<tt>` element rendered text in a monospaced (teletype) font. It has been replaced by semantic elements such as `<code>`, `<kbd>`, `<samp>`, and `<var>`.

**Technical Definition**
`<tt>` is a non-conforming presentational element. MDN and other sources recommend replacing it with `<code>` for source code, `<kbd>` for keyboard input, `<samp>` for program output, and `<var>` for variables, depending on the meaning of the text.

**Beginner-Friendly Explanation**
`<tt>` made text look like typewriter output. Use `<code>`, `<kbd>`, `<samp>`, or `<var>` depending on what the text actually represents.

---

### Purposes

- **Historically: to render monospaced text**.
- **Modern guidance: use semantic elements** (`<code>`, `<kbd>`, `<samp>`, `<var>`) or CSS `font-family: monospace`.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<tt>Monospaced text</tt>
```

#### Modern Replacements

| Context | Replacement |
|---|---|
| Source code | `<code>` |
| Keyboard input | `<kbd>` |
| Program output | `<samp>` |
| Variable | `<var>` |
| Generic monospace | CSS `font-family: monospace` |

#### Constraints and Limitations

- Non-conforming; use semantic elements or CSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<tt>`

**Setup guide**: Save as `tt-obsolete.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Obsolete TT</title>
</head>
<body>
  <!-- OBSOLETE: Do not use -->
  <p><tt>console.log()</tt></p>
</body>
</html>
```

**Expected output**: Monospaced text, but non-conforming.

**Why this output occurs**: Browsers retain backwards-compatible styling.

---

#### Example 2: Modern `<code>` Replacement

**Setup guide**: Save as `code-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Code Replacement</title>
</head>
<body>
  <p><code>console.log()</code></p>
</body>
</html>
```

**Expected output**: Monospaced text with the correct semantic meaning.

**Why this output occurs**: `<code>` is the semantic element for code fragments.

---

### Real-World Cases

- **Legacy pages**: `<tt>` appears in old technical documentation.
- **Modern practice**: Semantic elements are used.

---

### References

- WHATWG — Obsolete features (non-conforming list) — https://html.spec.whatwg.org/multipage/obsolete.html#non-conforming-features
- MDN Web Docs — `<code>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/code

---

## 24. `<xmp>` — The Example Text Element

### Definitions

**Core Definition**
The `<xmp>` element rendered its content as raw text, similar to `<pre>` but without HTML parsing. It has been replaced by `<pre>` or `<code>` with escaped characters.

**Technical Definition**
`<xmp>` is a non-conforming element. The WHATWG specification states: “Use the `pre` element or, if semantically adequate, the `code` element instead. Note that you will need to escape the `<` character as `&lt;` and the `&` character as `&amp;` to make sure they are not interpreted as markup”.

**Beginner-Friendly Explanation**
`<xmp>` showed raw HTML code without rendering it. Use `<pre>` with escaped characters instead.

---

### Purposes

- **Historically: to display HTML source code** as text.
- **Modern guidance: use `<pre>` and `<code>`** with escaped characters.

---

### Syntax Rules and Structure

#### Historical General Syntax

```html
<xmp>
  <p>This is shown as text, not rendered.</p>
</xmp>
```

#### Modern Replacement

```html
<pre><code>
  &lt;p&gt;This is shown as text, not rendered.&lt;/p&gt;
</code></pre>
```

#### Constraints and Limitations

- Non-conforming; use `<pre>` and escaping.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Obsolete `<xmp>`

**Setup guide**: For reference only.

```html
<xmp>
  <div>Example</div>
</xmp>
```

**Expected output**: The markup is shown as text in some browsers.

**Why this output occurs**: `<xmp>` disables HTML parsing for its content.

---

#### Example 2: Modern `<pre><code>` Replacement

**Setup guide**: Save as `xmp-replacement.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Pre Code Replacement</title>
</head>
<body>
  <pre><code>&lt;div&gt;Example&lt;/div&gt;</code></pre>
</body>
</html>
```

**Expected output**: The escaped HTML is displayed as text.

**Why this output occurs**: `<pre>` preserves whitespace, and `<code>` marks the content as code, while escaping prevents parsing.

---

### Real-World Cases

- **Legacy code examples**: `<xmp>` was used in old tutorials.
- **Modern practice**: `<pre><code>` with escaping is standard.

---

### References

- WHATWG — Obsolete features: `xmp` — https://html.spec.whatwg.org/multipage/obsolete.html#xmp
- MDN Web Docs — `<pre>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/pre

---

## Deprecated Element Alternatives — Summary Table

| Deprecated Element | Modern Replacement |
|---|---|
| `<applet>` | `<object>` or `<embed>` |
| `<acronym>` | `<abbr>` |
| `<bgsound>` | `<audio>` |
| `<big>` | CSS `font-size` |
| `<blink>` | Static emphasis (`<strong>`, `<em>`) or CSS animations with controls |
| `<center>` | CSS `text-align: center` and `margin: auto` |
| `<dir>` | `<ul>` |
| `<font>` | CSS `font-family`, `font-size`, `color` |
| `<frame>`, `<frameset>`, `<noframes>` | `<iframe>` + CSS layout, or server-side includes |
| `<isindex>` | `<form>` + `<input>` + `<button>` |
| `<keygen>` | Web Cryptography API (`crypto.subtle.generateKey()`) |
| `<marquee>` | CSS animations with pause/stop controls |
| `<menuitem>` | JavaScript `contextmenu` event handling |
| `<nobr>` | CSS `white-space: nowrap` |
| `<noembed>` | `<object>` with fallback content |
| `<plaintext>` | `text/plain` MIME type |
| `<rb>`, `<rtc>` | Direct ruby base text inside `<ruby>` |
| `<spacer>` | CSS `margin` and `padding` |
| `<strike>` | `<s>` or `<del>` |
| `<tt>` | `<code>`, `<kbd>`, `<samp>`, `<var>`, or CSS monospace |
| `<xmp>` | `<pre>` or `<code>` with escaped characters |

---

## Deprecated Element Best Practices

### Definitions

**Core Definition**
Deprecated element best practices are the guidelines for identifying, avoiding, and replacing obsolete HTML elements with modern, standards-compliant alternatives.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, which explicitly forbids the use of non-conforming elements, and by WCAG accessibility guidelines, which identify specific failures caused by elements such as `<blink>`, `<marquee>`, and `<frame>`. The core principle is **separation of concerns**: HTML for structure and semantics, CSS for presentation, and JavaScript for behaviour.

**Beginner-Friendly Explanation**
Best practices are simple: do not use old tags. Use modern HTML for meaning, CSS for styling, and JavaScript for interactivity.

---

### Purposes

- **To ensure standards compliance** and avoid validation errors.
- **To maximise accessibility** by avoiding elements that violate WCAG.
- **To improve maintainability** by using current, well-supported techniques.
- **To future-proof documents** against browser changes.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Never use non-conforming elements** | The WHATWG specification explicitly states they “must not be used by authors”. |
| **Replace presentational elements with CSS** | `<font>`, `<center>`, `<big>`, `<strike>`, `<tt>`, `<nobr>`, `<spacer>` all have CSS replacements. |
| **Replace structural legacy elements with modern HTML** | `<frame>`/`<frameset>` → CSS layout + `<iframe>`; `<dir>` → `<ul>`; `<acronym>` → `<abbr>`. |
| **Replace plugin-based elements with native HTML** | `<applet>` → `<object>`/`<embed>`; `<bgsound>` → `<audio>`; `<keygen>` → Web Crypto. |
| **Avoid distracting elements entirely** | `<blink>` and `<marquee>` violate WCAG 2.2.2; use static content or controlled animations. |
| **Use semantic elements for text** | Replace `<tt>` with `<code>`, `<kbd>`, `<samp>`, or `<var>`; replace `<strike>` with `<s>` or `<del>`. |
| **Run conformance checkers** | Validators warn or error on deprecated elements. |

#### Constraints and Limitations

- Legacy content may still contain deprecated elements; incremental migration is acceptable.
- Some obsolete elements are still rendered by browsers for backwards compatibility, but this must not be relied upon.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Migrating a Legacy Page to Modern HTML

**Setup guide**: Save as `migration-example.html`. The “before” markup is shown alongside the “after” replacement.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Migration Example</title>
  <style>
    .center { text-align: center; }
    .large { font-size: 1.5em; }
    .strike { text-decoration: line-through; }
    .code { font-family: monospace; }
  </style>
</head>
<body>
  <!-- BEFORE (obsolete): -->
  <!--
  <center><font size="5" color="red">Welcome</font></center>
  <p><strike>Old info</strike></p>
  <p><tt>code()</tt></p>
  -->

  <!-- AFTER (modern): -->
  <h1 class="center large" style="color: red;">Welcome</h1>
  <p><s>Old info</s></p>
  <p><code>code()</code></p>
</body>
</html>
```

**Expected output**: The modern markup renders equivalent visual output without any obsolete elements.

**Why this output occurs**: CSS classes replace presentational attributes, and semantic elements (`<h1>`, `<s>`, `<code>`) replace obsolete tags.

---

#### Example 2: Checking for Deprecated Elements with a Linter

**Setup guide**: Most linters (ESLint with `html-eslint`, or online validators) flag deprecated elements. Example ESLint configuration:

```json
{
  "rules": {
    "html/no-obsolete-tags": "error"
  }
}
```

Running the linter on a file containing `<marquee>` will produce an error, prompting replacement.

**Expected output**: Linter errors identifying deprecated elements.

**Why this output occurs**: Automated tools enforce best practices and catch legacy markup.

---

### Real-World Cases

1. **Legacy site migration**: Organisations replacing framesets with CSS Grid and iframes, and `<font>` with CSS.
2. **Accessibility audits**: Removing `<marquee>` and `<blink>` to meet WCAG 2.2.2.
3. **Design system modernisation**: Replacing `<center>` and `<font>` with utility classes and semantic HTML.

---

### References

- WHATWG — Obsolete features — https://html.spec.whatwg.org/multipage/obsolete.html
- W3C — Technique F47: Failure of SC 2.2.2 due to blink — https://www.w3.org/WAI/WCAG22/Techniques/failures/F47
- W3C — Technique C18: Using CSS margin and padding — https://www.w3.org/WAI/WCAG22/Techniques/css/C18
- Siteimprove — Deprecated HTML elements — https://help.siteimprove.com/support/solutions/articles/80000448651-accessibility-rule-html-element-is-deprecated-or-obsolete-explained
- MDN Web Docs — Obsolete and deprecated elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements#obsolete_and_deprecated_elements

---

## Final Notes

- **Deprecated does not mean removed from browsers**: Browsers generally retain partial support for legacy compatibility, but this is not guaranteed and should never be relied upon.
- **Validation is your safety net**: Conformance checkers will flag every element listed in this cheat sheet.
- **Accessibility matters most**: `<blink>`, `<marquee>`, and `<frame>` are not merely outdated — they actively harm users with disabilities.
- **CSS and modern HTML are always the answer**: For every deprecated element, a standards-based replacement exists.

---

This cheat sheet has covered the deprecated HTML elements — `<applet>`, `<acronym>`, `<bgsound>`, `<big>`, `<blink>`, `<center>`, `<dir>`, `<font>`, `<frame>`, `<frameset>`, `<isindex>`, `<keygen>`, `<marquee>`, `<menuitem>`, `<nobr>`, `<noembed>`, `<noframes>`, `<plaintext>`, `<rb>`, `<rtc>`, `<spacer>`, `<strike>`, `<tt>`, and `<xmp>` — with uniform definitions, purposes, syntax rules, annotated examples, modern alternatives, and authoritative references. Use it as a migration guide and a teaching aid, and validate all markup against the WHATWG HTML Living Standard and WCAG accessibility guidelines.