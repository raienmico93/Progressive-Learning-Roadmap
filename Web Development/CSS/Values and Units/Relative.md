# CSS Relative Units — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS relative units are length measurements whose computed value depends on another quantity — such as the font size of an element, the dimensions of the viewport, or the size of a container. Unlike absolute units (which are fixed and anchored to physical or canonical measurements), relative units create relationships between elements, enabling scalable, responsive, and accessible designs.

**Technical Definition:** In formal CSS terms, relative length units are defined in the CSS Values and Units Module (Levels 3 and 4) as `<relative-length-unit>` values. Each relative unit specifies a length relative to another length property: font-relative units resolve against font metrics (font-size, x-height, line-height, etc.), viewport-percentage units resolve against the dimensions of the viewport, and container-relative units resolve against the dimensions of a query container. The computed value of a relative length is an absolute length resolved at computed-value time. Child elements do not inherit the specified relative values of their parent; they inherit the computed values. Relative units are recommended for creating scalable layouts that maintain vertical rhythm even when the user changes font size.

**Beginner-Friendly Explanation:** Think of relative units as "measurements that adapt." Instead of saying "this box is 200 pixels wide," you can say "this box is twice the width of the current font size" (using `em`), or "this box is half the viewport width" (using `vw`), or "this box is 30% of its container" (using `cqw`). This means the box automatically resizes when the font changes, the window resizes, or the container changes size. Relative units are the foundation of responsive web design and accessible typography — they let your layout bend instead of break.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Context-dependent** | The computed value depends on the element's font, the viewport, or a container's size. |
| **Cascade-aware** | Font-relative units resolve against the element's own computed font metrics; `rem` and `rlh` resolve against the root element. |
| **Computed, not inherited** | Child elements inherit the computed absolute length, not the relative expression. |
| **Scalable** | Layouts using relative units adapt automatically to changes in font size, viewport, or container dimensions. |
| **Accessible** | `rem` and `em` respect user font-size preferences; viewport units respect viewport resizing. |
| **Interpolation-friendly** | Relative units can be interpolated in animations and transitions. |
| **Self-contained resolution** | Each relative length resolves within its own declaration context; a child does not inherit the parent's reference frame. |

---

### Prerequisites

Before studying CSS Relative Units, you should understand:

- **CSS absolute units** — particularly `px`, the canonical anchor unit.
- **The CSS box model** — how `width`, `height`, `margin`, and `padding` interact.
- **CSS inheritance and computed values** — how values propagate and resolve.
- **The viewport concept** — the visible area of a web page in the browser.
- **CSS custom properties (`var()`)** — often used in conjunction with relative units.

---

### Related Programming Areas

- **Responsive Web Design** — relative units are the foundation of fluid layouts.
- **CSS Typography** — `em`, `rem`, `ex`, `ch`, and `lh` for scalable text.
- **CSS Layout** — `vw`, `vh`, and `cqi` for viewport- and container-based sizing.
- **CSS Container Queries** — `cqw`, `cqh`, etc. for component-level responsiveness.
- **Accessibility** — respecting user font-size preferences and browser zoom.
- **Design Systems** — relative units enable spacing scales that adapt across contexts.

---

### Core Concepts / Features

1. Font-Relative Units (`em`, `rem`, `ex`, `ch`, `cap`, `ic`, `lh`, `rlh`)
2. Viewport-Relative Units (`vw`, `vh`, `vmin`, `vmax`; `svw`, `svh`, `lvw`, `lvh`, `dvw`, `dvh`)
3. Container-Relative Units (`cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax`)

---

## 1. Font-Relative Units

### Definitions

**Core Definition:** Font-relative units are length measurements whose values are derived from the font metrics of either the element itself or the root element. They include `em`, `ex`, `ch`, `cap`, `ic`, `lh` (local font-relative) and `rem`, `rex`, `rch`, `rcap`, `ric`, `rlh` (root font-relative).

**Technical Definition:** Font-relative lengths define the `<length>` value in terms of the size of a particular character or font attribute in the font currently in effect in an element or its parent. The `em` unit represents the computed font-size of the element; when used on `font-size` itself, it represents the inherited font-size. The `rem` unit represents the font-size of the root element. The `ex` unit represents the x-height of the element's font; `ch` represents the advance measure of the glyph `0`; `cap` represents the cap-height; `ic` represents the advance measure of the CJK water ideograph `水`; `lh` represents the computed line-height of the element. When used in the value of any `font-*` property on the element they refer to, font-relative lengths resolve against the computed font metrics of the parent element.

**Beginner-Friendly Explanation:** Font-relative units are like saying "make this element's spacing proportional to its text size." If you set `font-size: 2em`, you are saying "make the font twice as big as the parent's font." If you set `padding: 1em`, you are saying "make the padding equal to one times the current font size." The `rem` unit is similar but always refers to the root element's font size (usually `<html>`), so it does not compound. The `ch` unit is especially useful for setting widths that match a specific number of characters — `width: 60ch` makes a text column that is approximately 60 characters wide.

---

### Purposes

- To create scalable typography that respects user font-size preferences.
- To maintain vertical rhythm and proportional spacing across an entire layout.
- To size elements based on the dimensions of their own text content.
- To provide bulletproof accessibility through root-relative sizing (`rem`).
- To match layout widths to typographic character counts using `ch`.
- To align elements with line-height dimensions using `lh` and `rlh`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
<number>em     /* Local font size */
<number>rem    /* Root font size */
<number>ex     /* Local x-height */
<number>ch     /* Local advance measure of "0" */
<number>cap    /* Local cap-height */
<number>ic     /* Local ideographic character advance */
<number>lh     /* Local line-height */
<number>rlh    /* Root line-height */
<number>rex    /* Root x-height */
<number>rch    /* Root advance measure of "0" */
<number>rcap   /* Root cap-height */
<number>ric    /* Root ideographic character advance */
```

#### Component Breakdown

| Unit | Relative To | Description |
|---|---|---|
| `em` | Element's computed `font-size` | 1em = current font size. On `font-size` property, uses inherited size. |
| `rem` | Root element's `font-size` | 1rem = root font size (typically 16px, but user-configurable). |
| `ex` | Element's font x-height | Approximately 0.5em in many fonts. |
| `ch` | Element's font "0" glyph width | Approximately 0.5em to 0.6em depending on the font. |
| `cap` | Element's font cap-height | Height of capital letters. |
| `ic` | Element's font ideographic advance | Width of the CJK water ideograph `水`. |
| `lh` | Element's computed `line-height` | Equal to the line-height in absolute length. |
| `rlh` | Root element's computed `line-height` | Root line-height. |
| `rex`, `rch`, `rcap`, `ric` | Root element's font metrics | Equivalent to `ex`, `ch`, `cap`, `ic` but relative to the root element. |

#### Syntax Rules

1. The unit identifier must immediately follow the number with no whitespace: `2em` is valid; `2 em` is invalid.
2. After a literal `0`, the unit may be omitted.
3. `em` and `rem` are the most widely supported; `cap`, `ic`, `lh`, `rlh`, and their root variants are newer and have varying browser support.
4. When `em` is used on the `font-size` property itself, it represents the **inherited** font-size of the element.
5. Font-relative units used in `font-*` properties resolve against the **parent** element's computed font metrics.
6. `lh` and `rlh` are based on the computed `line-height` and may differ from the actual line box size.
7. The `ch` unit is based on the advance measure of the `0` glyph; in fonts where this cannot be determined, it must be assumed to be 0.5em wide by 1em tall.

#### Constraints and Limitations

- **Compounding with `em`** — Nested elements using `em` for `font-size` compound multiplicatively, which can lead to unpredictable sizing.
- **Browser support** — `rem` is universally supported; `cap`, `ic`, `lh`, and `rlh` have more limited support (see references for current data).
- **Font availability** — `ex`, `ch`, `cap`, and `ic` depend on the actual font being used; if the font is not yet loaded, the browser may use fallback metrics.
- **Font downloads** — Font-relative units such as `ch` and `ic` can trigger font downloads if a required font is not yet loaded.
- **Line-height variance** — `lh` and `rlh` are based on the computed `line-height`, but actual line boxes may differ based on content.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `em` vs. `rem` — Compounding vs. Non-Compounding

**HTML File (`em-rem.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>em vs rem Demonstration</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="em-rem.css">
</head>
<body>
    <!-- Parent section with font-size in em -->
    <section class="parent-em">
        <h1>Parent with font-size: 1.5em</h1>
        <p>
            This parent section has a font-size of 1.5em, which resolves to
            1.5 × 16px = 24px.
        </p>
        <!-- Child section: font-size in em compounds -->
        <div class="child-em">
            <h2>Child with font-size: 1.5em (compounds to 36px)</h2>
            <p>
                This child inherits the parent's computed font-size (24px) and
                then applies its own 1.5em, resulting in 24px × 1.5 = 36px.
            </p>
        </div>
    </section>

    <!-- Section with font-size in rem: no compounding -->
    <section class="parent-rem">
        <h1>Parent with font-size: 1.5rem</h1>
        <p>
            This parent uses 1.5rem, which resolves to 1.5 × 16px = 24px
            (where 16px is the root font size).
        </p>
        <div class="child-rem">
            <h2>Child with font-size: 1.5rem (stays at 24px)</h2>
            <p>
                This child also uses 1.5rem, which resolves to the same 24px
                because it always refers to the root font size, not the parent.
            </p>
        </div>
    </section>
</body>
</html>
```

**CSS File (`em-rem.css`):**

```css
/* Root font size: the reference for all rem units */
html {
    font-size: 16px;   /* 16px is the common browser default, but users can change this */
}

/* Body base styling */
body {
    font-family: system-ui, sans-serif;
    margin: 20px;
    background-color: #fafafa;
}

/* ===== EM SECTION ===== */

/* Parent with em-based font-size */
.parent-em {
    font-size: 1.5em;    /* 1.5em = 1.5 × inherited font-size (16px from body) = 24px */
    background-color: #e8f4fd;
    padding: 1em;        /* 1em = 24px (the parent's computed font size) */
    border-left: 4px solid #3498db;
    margin-bottom: 20px;
}

/* Child with em-based font-size — COMPOUNDS */
.child-em {
    font-size: 1.5em;    /* 1.5em = 1.5 × inherited font-size (24px from parent) = 36px */
    background-color: #d5eaf5;
    padding: 1em;        /* 1em = 36px (the child's computed font size) */
    border-left: 3px solid #2980b9;
}

/* ===== REM SECTION ===== */

/* Parent with rem-based font-size */
.parent-rem {
    font-size: 1.5rem;   /* 1.5rem = 1.5 × root font-size (16px) = 24px */
    background-color: #fdeaea;
    padding: 1rem;       /* 1rem = 16px (always the root font size) */
    border-left: 4px solid #e74c3c;
    margin-bottom: 20px;
}

/* Child with rem-based font-size — DOES NOT COMPOUND */
.child-rem {
    font-size: 1.5rem;   /* 1.5rem = 1.5 × root font-size (16px) = 24px, NOT 36px */
    background-color: #fadbd8;
    padding: 1rem;       /* 1rem = 16px (still the root font size) */
    border-left: 3px solid #c0392b;
}

/* Shared heading and paragraph styling */
h1, h2 {
    margin: 0 0 0.5em 0;
    color: #2c3e50;
}

p {
    margin: 0;
    line-height: 1.6;
    color: #34495e;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML as `em-rem.html` and the CSS as `em-rem.css` in the same folder.
3. Open `em-rem.html` in a web browser.
4. Observe the blue section (em-based): the child heading is noticeably larger than the parent heading because the child's `1.5em` compounds on top of the parent's `1.5em`.
5. Observe the red section (rem-based): the parent and child headings are the same size because both use `1.5rem`, which always refers to the root's 16px.

**Expected Output:** A blue section with progressively larger text (16px → 24px → 36px) and a red section with consistent text at 24px for both parent and child. The padding in the em section also increases with the font size, while the padding in the rem section stays at 16px.

**Why This Works:** The `em` unit is relative to the element's own computed font-size. When the parent sets `font-size: 1.5em`, it computes to 24px. The child then inherits that 24px and applies its own `1.5em`, computing to 36px — this is compounding. The `rem` unit always refers to the root element's font-size (16px), regardless of the parent's font-size. So `1.5rem` on both parent and child always resolves to 24px, with no compounding.

---

#### Example 2: `ch` for Character-Count Width and `lh` for Line-Height Spacing

**HTML File (`ch-lh.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ch and lh Units</title>
    <link rel="stylesheet" href="ch-lh.css">
</head>
<body>
    <article class="article">
        <h1>Optimal Reading Width with ch</h1>
        <p>
            This article body uses <code>max-width: 65ch</code>, which means
            the text column is approximately 65 characters wide. This is
            widely considered an optimal measure for readability. The <code>ch</code>
            unit is based on the advance measure of the "0" glyph in the
            current font.
        </p>
        <p>
            Notice how the line breaks feel natural — the width is tied to
            the font's character dimensions, not an arbitrary pixel value.
        </p>
    </article>

    <div class="rhythm-example">
        <h2>Vertical Rhythm with lh</h2>
        <p>First paragraph. The margin below is exactly 1lh.</p>
        <p>Second paragraph. The margin below is exactly 1lh.</p>
        <p>Third paragraph. Notice the consistent spacing.</p>
    </div>
</body>
</html>
```

**CSS File (`ch-lh.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #fafafa;
    line-height: 1.6;
}

/* Article with ch-based max-width */
.article {
    /* 65ch = approximately 65 characters wide at the current font */
    max-width: 65ch;
    /* Center the article */
    margin: 0 auto 40px;
    /* Use rem for padding, independent of the font size */
    padding: 2rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.article h1 {
    font-size: 1.75rem;
    color: #2c3e50;
}

.article p {
    font-size: 1rem;
    line-height: 1.7;
    color: #34495e;
}

/* Vertical rhythm example using lh */
.rhythm-example {
    /* 65ch for consistency */
    max-width: 65ch;
    margin: 0 auto;
    padding: 2rem;
    background: #eafaf1;
    border-radius: 8px;
    border-left: 4px solid #27ae60;
}

.rhythm-example h2 {
    font-size: 1.25rem;
    color: #1e8449;
}

.rhythm-example p {
    /* 1lh = one line-height unit */
    margin-bottom: 1lh;
    color: #1e8449;
}

.rhythm-example p:last-child {
    margin-bottom: 0;
}

code {
    background-color: #ecf0f1;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: monospace;
    font-size: 0.9em;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `ch-lh.html` and the CSS as `ch-lh.css`.
2. Open `ch-lh.html` in a browser.
3. Observe the article text — the lines break at approximately 65 characters, creating a comfortable reading width.
4. Observe the green section — the spacing between paragraphs is exactly one line-height unit, maintaining a consistent vertical rhythm.
5. Resize the browser window and observe that the text column remains at 65ch, even as the viewport changes.

**Expected Output:** A white article card with a text column approximately 65 characters wide, and a green section below with three paragraphs separated by consistent 1lh margins.

**Why This Works:** The `ch` unit measures the advance width of the `0` glyph in the current font. `max-width: 65ch` creates a text column that fits approximately 65 characters, which is within the optimal reading range (45–75 characters). The `lh` unit equals the computed `line-height` of the element. `margin-bottom: 1lh` creates spacing exactly one line-height tall, ensuring that the gap between paragraphs matches the vertical rhythm of the text. Both units are font-relative, so they adapt automatically if the font or font size changes.

---

### Real-World Cases

- **Accessible base sizing:** Setting `html { font-size: 100%; }` and using `rem` for all component sizes, so that changing the browser's default font size scales the entire layout proportionally.
- **Component padding:** Using `padding: 1em` on buttons so the padding scales with the button's font size (e.g., a large button gets proportionally larger padding).
- **Optimal reading measure:** Using `max-width: 65ch` on article containers to ensure readable line lengths.
- **Vertical rhythm:** Using `margin-bottom: 1lh` on paragraphs to maintain consistent spacing aligned with line-height.
- **Icon sizing:** Using `width: 1em; height: 1em` on inline SVG icons so they scale with the surrounding text.

---

## 2. Viewport-Relative Units

### Definitions

**Core Definition:** Viewport-relative units are length measurements whose values are derived from the dimensions of the viewport — the visible area of a web page in the browser window. They include the legacy units `vw`, `vh`, `vmin`, `vmax` and the modern variants `svw`, `svh`, `lvw`, `lvh`, `dvw`, `dvh`.

**Technical Definition:** Viewport-percentage lengths define `<length>` values in percentage relative to the size of the initial containing block (the viewport). The `vw` unit equals 1% of the width of the UA-default viewport size; `vh` equals 1% of the height. The `vmin` unit equals the smaller of `vw` or `vh`; `vmax` equals the larger. The modern viewport variants account for the dynamic expansion and retraction of browser UI (such as address bars on mobile devices): `svw`/`svh` are based on the small viewport size (all UI visible), `lvw`/`lvh` are based on the large viewport size (all UI retracted), and `dvw`/`dvh` are based on the dynamic viewport size (the currently visible area). The `vi` and `vb` units correspond to the inline and block axes of the root element's writing mode.

**Beginner-Friendly Explanation:** Viewport units let you size things relative to the browser window. `100vw` means "100% of the viewport width" and `100vh` means "100% of the viewport height." A common use is `height: 100vh` for a hero section that fills the entire screen. But on mobile devices, the browser's address bar expands and retracts as you scroll, which changes the viewport height. The modern units solve this problem: `svh` is the smallest possible height (address bar visible), `lvh` is the largest (address bar hidden), and `dvh` is the current height (changes dynamically). For most use cases, `dvh` provides the most intuitive behaviour on mobile.

---

### Purposes

- To create full-screen layouts that fill the viewport.
- To size elements proportionally to the browser window, independent of their parent container.
- To build responsive typography that scales with viewport width (e.g., `font-size: 5vw`).
- To handle mobile browser UI dynamics gracefully using `svh`, `lvh`, and `dvh`.
- To create aspect-ratio-based layouts using `vmin` and `vmax`.
- To align elements with the inline or block axis using `vi` and `vb`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Legacy viewport units */
<number>vw     /* 1% of viewport width */
<number>vh     /* 1% of viewport height */
<number>vmin   /* 1% of smaller viewport dimension */
<number>vmax   /* 1% of larger viewport dimension */

/* Modern viewport variants */
<number>svw    /* 1% of small viewport width */
<number>svh    /* 1% of small viewport height */
<number>lvw    /* 1% of large viewport width */
<number>lvh    /* 1% of large viewport height */
<number>dvw    /* 1% of dynamic viewport width */
<number>dvh    /* 1% of dynamic viewport height */

/* Axis-relative units */
<number>vi     /* 1% of viewport inline size */
<number>vb     /* 1% of viewport block size */
```

#### Component Breakdown

| Unit | Relative To | Description |
|---|---|---|
| `vw` | Viewport width | 1% of the UA-default viewport width. |
| `vh` | Viewport height | 1% of the UA-default viewport height. |
| `vmin` | Smaller dimension | The smaller of `vw` or `vh`. |
| `vmax` | Larger dimension | The larger of `vw` or `vh`. |
| `svw`, `svh` | Small viewport | Assumes all browser UI is visible (smallest size). |
| `lvw`, `lvh` | Large viewport | Assumes all browser UI is retracted (largest size). |
| `dvw`, `dvh` | Dynamic viewport | Currently visible area; changes as UI expands/retracts. |
| `vi`, `vb` | Inline/block axis | 1% of the viewport's inline or block size based on writing mode. |

#### Syntax Rules

1. The unit identifier must immediately follow the number with no whitespace: `100vw` is valid; `100 vw` is invalid.
2. After a literal `0`, the unit may be omitted.
3. `vh` is equivalent to `lvh` in the specification's definition, representing the large viewport size.
4. The `vi` and `vb` units use the root element's writing mode to determine which axis they correspond to. In horizontal writing mode, `vi` = `vw` and `vb` = `vh`.
5. On mobile, `dvh` updates dynamically as the browser's UI expands and retracts; `svh` and `lvh` are fixed to the smallest and largest possible sizes.
6. Viewport units are not affected by the parent element's size — they always refer to the viewport.

#### Constraints and Limitations

- **Mobile browser UI** — the legacy `vh` unit can cause content to be hidden behind the browser's address bar on mobile devices; `dvh` solves this.
- **Scrollbar inconsistency** — `100vw` may include the scrollbar width on some browsers, causing horizontal overflow.
- **Performance** — `dvh` updates on every scroll frame, which can trigger layout recalculations; use sparingly in performance-critical contexts.
- **Browser support** — `svh`, `lvh`, and `dvh` have good but not universal support (see references).
- **Typographic scaling** — `font-size: 5vw` can make text unreadably small on narrow screens or too large on wide screens; `clamp()` is recommended.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full-Screen Hero Section with Modern Viewport Units

**HTML File (`viewport.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Viewport Units Demo</title>
    <link rel="stylesheet" href="viewport.css">
</head>
<body>
    <!-- Hero section using dvh for mobile-safe full-screen height -->
    <section class="hero">
        <h1>Full-Screen Hero</h1>
        <p>
            This section uses <code>height: 100dvh</code>, which means it
            always fills the currently visible viewport height — even on
            mobile devices where the browser's address bar expands and
            retracts as you scroll.
        </p>
    </section>

    <!-- Comparison section -->
    <section class="comparison">
        <h2>Viewport Unit Comparison</h2>
        <div class="box vh-box">100vh<br>(can overflow on mobile)</div>
        <div class="box svh-box">100svh<br>(smallest visible area)</div>
        <div class="box lvh-box">100lvh<br>(largest visible area)</div>
        <div class="box dvh-box">100dvh<br>(current visible area)</div>
    </section>
</body>
</html>
```

**CSS File (`viewport.css`):**

```css
/* Reset margins */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: system-ui, sans-serif;
    background-color: #f5f5f5;
}

/* Hero section: fills the dynamic viewport height */
.hero {
    /* 100dvh = 100% of the current visible viewport height */
    height: 100dvh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    /* vw-based horizontal padding for proportional spacing */
    padding: 5vw;
    background: linear-gradient(135deg, #2c3e50, #3498db);
    color: white;
    text-align: center;
}

.hero h1 {
    /* vw-based font size for responsive typography */
    font-size: 6vw;
    margin-bottom: 2vh;
}

.hero p {
    font-size: 2.5vw;
    max-width: 60ch;
    line-height: 1.6;
    opacity: 0.9;
}

/* Comparison section */
.comparison {
    padding: 5vw;
    background: #fafafa;
}

.comparison h2 {
    font-size: 3vw;
    margin-bottom: 3vh;
    color: #2c3e50;
}

.box {
    width: 100%;
    margin-bottom: 2vh;
    padding: 3vh 2vw;
    border-radius: 6px;
    color: white;
    font-weight: 500;
    text-align: center;
}

/* Each box uses a different viewport unit */
.vh-box {
    /* Legacy: may overflow on mobile */
    height: 20vh;
    background-color: #e74c3c;
}

.svh-box {
    /* Small viewport: smallest visible area */
    height: 20svh;
    background-color: #e67e22;
}

.lvh-box {
    /* Large viewport: largest visible area */
    height: 20lvh;
    background-color: #27ae60;
}

.dvh-box {
    /* Dynamic: current visible area */
    height: 20dvh;
    background-color: #2980b9;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `viewport.html` and the CSS as `viewport.css`.
2. Open `viewport.html` on a mobile device or in a mobile-emulated browser (Chrome DevTools → Toggle Device Toolbar).
3. Scroll down and observe that the hero section fills exactly the visible area — it does not extend behind the browser's address bar.
4. Observe the comparison boxes: on mobile, `20vh` may extend below the visible area, while `20dvh` adjusts dynamically.
5. Resize the browser window on desktop and observe the vw-based font sizes and padding scale proportionally.

**Expected Output:** A full-screen gradient hero section with large responsive text, followed by four coloured boxes demonstrating the differences between `vh`, `svh`, `lvh`, and `dvh`. On mobile, the `dvh` hero section fits exactly within the visible area.

**Why This Works:** The `dvh` unit is defined as 1% of the dynamic viewport size — the currently visible area. On mobile, as the browser's address bar retracts, the viewport grows, and `dvh` updates accordingly. The `svh` unit is fixed to the smallest possible viewport height (address bar visible), while `lvh` is fixed to the largest (address bar hidden). The `vh` unit is equivalent to `lvh` in the specification, which is why it can overflow the visible area on mobile.

---

#### Example 2: Responsive Typography with `clamp()` and Viewport Units

**HTML File (`clamp-vw.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Typography with clamp()</title>
    <link rel="stylesheet" href="clamp-vw.css">
</head>
<body>
    <h1 class="responsive-heading">
        Fluid Heading with clamp()
    </h1>
    <p class="responsive-body">
        This paragraph uses <code>font-size: clamp(1rem, 2.5vw, 1.5rem)</code>.
        The font size will never go below 1rem (16px), never exceed 1.5rem
        (24px), and scales fluidly with the viewport width in between.
    </p>
    <div class="spacer"></div>
</body>
</html>
```

**CSS File (`clamp-vw.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 5vw;
    background-color: #fafafa;
}

/* Fluid heading: scales with viewport but is bounded */
.responsive-heading {
    /* clamp(min, preferred, max):
       - Never smaller than 1.5rem (24px)
       - Ideally 4vw (4% of viewport width)
       - Never larger than 3rem (48px) */
    font-size: clamp(1.5rem, 4vw, 3rem);
    color: #2c3e50;
    margin-bottom: 3vh;
    line-height: 1.2;
}

/* Fluid body text */
.responsive-body {
    /* Never smaller than 1rem, ideally 2.5vw, never larger than 1.5rem */
    font-size: clamp(1rem, 2.5vw, 1.5rem);
    line-height: 1.7;
    color: #34495e;
    max-width: 65ch;
}

code {
    background-color: #ecf0f1;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: monospace;
    font-size: 0.9em;
}

.spacer {
    height: 50vh;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `clamp-vw.html` and the CSS as `clamp-vw.css`.
2. Open `clamp-vw.html` in a browser.
3. Resize the browser window from very narrow (mobile) to very wide (desktop).
4. Observe that the heading and body text grow smoothly with the viewport but never exceed their bounds.

**Expected Output:** A heading that is 24px on narrow screens, grows to 4% of the viewport width, and caps at 48px on wide screens. Body text follows a similar pattern between 16px and 24px.

**Why This Works:** `clamp(min, preferred, max)` ensures the font size never goes below `min` or above `max`, while the `preferred` value (`4vw` or `2.5vw`) provides fluid scaling. On narrow screens, the `vw` value is small, so `clamp()` returns the minimum. On wide screens, the `vw` value exceeds the maximum, so `clamp()` returns the maximum. In between, the `vw` value is used directly, creating smooth scaling.

---

### Real-World Cases

- **Hero sections:** Using `height: 100dvh` for full-screen hero sections that fit the visible area on mobile.
- **Full-screen modals:** Using `height: 100svh` to ensure modals fit within the smallest viewport.
- **Responsive typography:** Using `font-size: clamp(1rem, 2.5vw, 1.5rem)` for headings that scale with the viewport.
- **Aspect-ratio boxes:** Using `width: 50vmin; height: 50vmin` for square elements that maintain their aspect ratio regardless of orientation.
- **Spacing systems:** Using `margin: 5vh 3vw` for responsive vertical and horizontal spacing.

---

## 3. Container-Relative Units

### Definitions

**Core Definition:** Container-relative units are length measurements whose values are derived from the dimensions of a **query container** — an ancestor element that has been designated as a container via `container-type`. They include `cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, and `cqmax`.

**Technical Definition:** Container query length units specify a length relative to the dimensions of a query container. The `cqw` unit equals 1% of the query container's width; `cqh` equals 1% of its height; `cqi` equals 1% of its inline size; `cqb` equals 1% of its block size; `cqmin` equals the smaller of `cqi` or `cqb`; `cqmax` equals the larger. The query container is the nearest ancestor element that has a `container-type` of `size` or `inline-size`. If no ancestor is designated as a container, the units resolve against the viewport (acting as the initial container).

**Beginner-Friendly Explanation:** Container-relative units are like viewport units, but they measure against a **container element** instead of the browser window. This means you can make a component's text, padding, and layout adapt to the size of the card or panel it is placed inside — not just the overall screen size. For example, if a product card is 300px wide in a sidebar, `font-size: 5cqi` makes the text 5% of the card's width. If the same card is placed in a 600px-wide main area, the text automatically becomes twice as large. This is the foundation of component-level responsive design.

---

### Purposes

- To create self-contained, responsive components that adapt to their container's size.
- To scale typography and spacing based on the available space within a component.
- To build micro-layouts that are independent of the browser window dimensions.
- To reduce the need for media queries by making components intrinsically responsive.
- To create design systems where components look correct in any container context.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
<number>cqw     /* 1% of query container's width */
<number>cqh     /* 1% of query container's height */
<number>cqi     /* 1% of query container's inline size */
<number>cqb     /* 1% of query container's block size */
<number>cqmin   /* 1% of smaller of cqi or cqb */
<number>cqmax   /* 1% of larger of cqi or cqb */
```

#### Component Breakdown

| Unit | Relative To | Description |
|---|---|---|
| `cqw` | Container width | 1% of the query container's width. |
| `cqh` | Container height | 1% of the query container's height. |
| `cqi` | Container inline size | 1% of the container's inline size (logical width). |
| `cqb` | Container block size | 1% of the container's block size (logical height). |
| `cqmin` | Smaller dimension | The smaller of `cqi` or `cqb`. |
| `cqmax` | Larger dimension | The larger of `cqi` or `cqb`. |

#### Syntax Rules

1. To use container-relative units, an ancestor element must be designated as a query container using `container-type: size` or `container-type: inline-size`.
2. The `cqw` and `cqh` units are physical (width/height) and are considered legacy; `cqi` and `cqb` are the logical equivalents and are preferred for writing-mode independence.
3. `cqmin` and `cqmax` are useful for maintaining aspect ratios and ensuring elements fit within both dimensions of the container.
4. If no ancestor is designated as a query container, container units resolve against the viewport (acting as the initial container).
5. Container units can be used in `@container` queries and in regular property declarations.
6. The `container-type: inline-size` designation makes the container's inline size available for container units, but not its block size.

#### Constraints and Limitations

- **Ancestor requirement** — container units require a designated query container ancestor; without it, they resolve against the viewport.
- **Browser support** — container query units have good support in modern browsers (Chrome 106+, Firefox 110+, Safari 16.6+) but are not supported in older browsers.
- **Performance** — container queries can trigger layout recalculations; use judiciously in performance-critical contexts.
- **Limited height queries** — `container-type: size` makes both dimensions available but can be more expensive; `inline-size` is recommended for most use cases.
- **Nested containers** — the nearest ancestor container is used; nested containers resolve to the innermost one.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Responsive Card Component with Container Units

**HTML File (`container.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Units Demo</title>
    <link rel="stylesheet" href="container.css">
</head>
<body>
    <!-- Main content area: wide container -->
    <main class="main-area">
        <h1>Main Content Area</h1>
        <!-- Card placed in wide container -->
        <div class="card-wrapper">
            <article class="card">
                <h2 class="card-title">Product Name</h2>
                <p class="card-text">
                    This card adapts to its container. In this wide layout,
                    the text and padding are proportionally larger.
                </p>
                <button class="card-button">Add to Cart</button>
            </article>
        </div>
    </main>

    <!-- Sidebar: narrow container -->
    <aside class="sidebar">
        <h1>Sidebar</h1>
        <!-- Same card markup, placed in narrow container -->
        <div class="card-wrapper">
            <article class="card">
                <h2 class="card-title">Product Name</h2>
                <p class="card-text">
                    The same card in a narrow sidebar. The text and padding
                    scale down proportionally to fit the container.
                </p>
                <button class="card-button">Add to Cart</button>
            </article>
        </div>
    </aside>
</body>
</html>
```

**CSS File (`container.css`):**

```css
/* Reset */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: system-ui, sans-serif;
    display: flex;
    gap: 2rem;
    padding: 2rem;
    background-color: #f5f5f5;
    min-height: 100vh;
}

/* Main area: wide layout */
.main-area {
    flex: 2;
}

/* Sidebar: narrow layout */
.sidebar {
    flex: 1;
    max-width: 300px;
}

/* Card wrapper: the query container */
.card-wrapper {
    /* Designate this element as a query container */
    container-type: inline-size;
    /* Assign a name for container queries (optional) */
    container-name: card;
    margin-top: 1rem;
}

/* Card component: uses container-relative units */
.card {
    /* Background and border */
    background: white;
    border-radius: 1cqi;  /* 1% of container inline size */
    border: 1px solid #e0e0e0;
    /* Padding scales with container size */
    padding: 4cqi;
    /* Overflow protection */
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

/* Card title: font size scales with container */
.card-title {
    /* 5cqi = 5% of the container's inline size */
    font-size: clamp(0.9rem, 5cqi, 1.5rem);
    color: #2c3e50;
    margin-bottom: 2cqi;
    line-height: 1.3;
}

/* Card text: scales with container */
.card-text {
    /* 3cqi = 3% of the container's inline size */
    font-size: clamp(0.75rem, 3cqi, 1rem);
    color: #555;
    line-height: 1.6;
    margin-bottom: 4cqi;
}

/* Card button: scales with container */
.card-button {
    /* Padding scales with container */
    padding: 2.5cqi 5cqi;
    font-size: clamp(0.75rem, 2.5cqi, 0.95rem);
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 0.5cqi;
    cursor: pointer;
    transition: background-color 0.2s;
}

.card-button:hover {
    background-color: #2980b9;
}

/* Sidebar-specific styling via container query */
@container card (max-width: 250px) {
    .card {
        padding: 3cqi;
    }
    .card-title {
        font-size: clamp(0.8rem, 6cqi, 1.1rem);
    }
    .card-text {
        font-size: clamp(0.7rem, 4cqi, 0.85rem);
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `container.html` and the CSS as `container.css`.
2. Open `container.html` in a modern browser (Chrome 106+, Firefox 110+, Safari 16.6+).
3. Observe the two cards: the one in the main area (wide) has larger text and padding, while the one in the sidebar (narrow) has proportionally smaller text and padding.
4. Resize the browser window and observe that the cards adapt to their container widths, not the viewport width.

**Expected Output:** Two product cards with identical HTML but different visual sizes. The wide card has larger text and padding; the narrow card has smaller text and padding, both scaled proportionally to their container width.

**Why This Works:** The `.card-wrapper` elements have `container-type: inline-size`, designating them as query containers. The card component uses `cqi` units, which resolve to 1% of the container's inline size. In the wide main area, the container is larger, so `5cqi` produces a larger font size. In the narrow sidebar, the container is smaller, so `5cqi` produces a smaller font size. The `clamp()` function ensures the text never becomes unreadably small or excessively large. This is component-level responsive design — the card adapts to its context, not the viewport.

---

#### Example 2: Micro-Layout with `cqmin` and `cqmax`

**HTML File (`micro-layout.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>cqmin and cqmax Demo</title>
    <link rel="stylesheet" href="micro-layout.css">
</head>
<body>
    <div class="container-wide">
        <h2>Wide Container</h2>
        <div class="responsive-circle">
            <span>cqmin</span>
        </div>
    </div>

    <div class="container-tall">
        <h2>Tall Container</h2>
        <div class="responsive-circle">
            <span>cqmin</span>
        </div>
    </div>
</body>
</html>
```

**CSS File (`micro-layout.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    display: flex;
    gap: 2rem;
    padding: 2rem;
    background-color: #f5f5f5;
    align-items: flex-start;
}

/* Wide container: query container with a wide aspect ratio */
.container-wide {
    /* Designate as a query container with both dimensions available */
    container-type: size;
    width: 400px;
    height: 200px;
    background-color: #e8f4fd;
    border-radius: 8px;
    padding: 1rem;
    overflow: hidden;
}

/* Tall container: query container with a tall aspect ratio */
.container-tall {
    container-type: size;
    width: 200px;
    height: 400px;
    background-color: #fdeaea;
    border-radius: 8px;
    padding: 1rem;
    overflow: hidden;
}

/* Responsive circle: uses cqmin to stay within both dimensions */
.responsive-circle {
    /* 50cqmin = 50% of the smaller container dimension */
    width: 50cqmin;
    height: 50cqmin;
    border-radius: 50%;
    background-color: #3498db;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: clamp(0.6rem, 8cqmin, 1.2rem);
    font-weight: bold;
    /* Center the circle in the container */
    margin: 0 auto;
}

/* In the wide container, the circle is sized by the smaller dimension (height) */
.container-wide .responsive-circle {
    background-color: #2980b9;
}

/* In the tall container, the circle is sized by the smaller dimension (width) */
.container-tall .responsive-circle {
    background-color: #e74c3c;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `micro-layout.html` and the CSS as `micro-layout.css`.
2. Open `micro-layout.html` in a browser.
3. Observe the two containers: a wide blue one and a tall red one.
4. Observe that in both containers, the circle is sized based on the smaller dimension (`cqmin`), so it always fits within the container without overflowing.
5. Resize the browser window and observe that the circles scale proportionally.

**Expected Output:** Two containers with circles of different sizes. In the wide container (400×200), the circle is 100px (50% of the height, the smaller dimension). In the tall container (200×400), the circle is also 100px (50% of the width, the smaller dimension).

**Why This Works:** The `cqmin` unit equals the smaller of `cqi` (inline size) and `cqb` (block size). In the wide container, the height (200px) is smaller than the width (400px), so `50cqmin` = 50% of 200px = 100px. In the tall container, the width (200px) is smaller than the height (400px), so `50cqmin` = 50% of 200px = 100px. This ensures the circle always fits within the container regardless of its aspect ratio.

---

### Real-World Cases

- **Product cards:** Using `cqi` units for card text and padding so cards look correct in both wide and narrow layouts.
- **Dashboard widgets:** Using `cqmin` for chart elements that need to fit within fixed-size widget containers.
- **Email templates:** Using container units for responsive email components that adapt to different email client widths.
- **Design systems:** Using container units in component libraries so components are intrinsically responsive without media queries.
- **Nested layouts:** Using container units in deeply nested components to ensure each component responds to its immediate context.

---

## References

- MDN Web Docs — CSS Values and Units - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Values_and_units
- MDN Web Docs — `<length>` - https://developer.mozilla.org/en-US/docs/Web/CSS/length
- MDN Web Docs — Font-relative units - https://developer.mozilla.org/en-US/docs/Web/CSS/length#relative_length_units_based_on_font
- MDN Web Docs — Viewport-relative units - https://developer.mozilla.org/en-US/docs/Web/CSS/length#viewport-percentage_length_units
- MDN Web Docs — Container query length units - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
- W3C — CSS Values and Units Module Level 3 - https://www.w3.org/TR/css-values-3/
- W3C — CSS Values and Units Module Level 4 - https://www.w3.org/TR/css-values-4/
- W3C — CSS Containment Module Level 3 - https://www.w3.org/TR/css-contain-3/
- web.dev — Container queries and units in action - https://web.dev/articles/baseline-in-action-container-queries
- web.dev — Sizing (CSS Learn) - https://web.dev/learn/css/sizing
- CSS-Tricks — The Large, Small, and Dynamic Viewports - https://css-tricks.com/the-large-small-and-dynamic-viewports/
- Can I Use — Container Query Units - https://caniuse.com/css-container-query-units
- LambdaTest — CSS Container Query Units Browser Compatibility - https://www.lambdatest.com/web-technologies/css-container-query-units