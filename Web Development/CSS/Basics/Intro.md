# Introduction to CSS

CSS is the language that styles the web. While HTML provides structure and JavaScript provides behavior, CSS controls how content looks — its colors, spacing, typography, layout, and responsiveness. Understanding CSS is essential for building any modern web interface.

---

## 1. CSS Definition

### 1.1 Cascading Style Sheets

**CSS** stands for **Cascading Style Sheets**. It is a **stylesheet language** used to describe the **presentation** of a document written in a markup language such as HTML or XML.

The name captures two core ideas:

| Term | Meaning |
|---|---|
| **Cascading** | Multiple style sources combine, and conflicts are resolved by a defined priority system |
| **Style Sheets** | Collections of rules that describe how elements should be rendered |

### 1.2 Styling Language for HTML/XML-Based Documents

CSS is not limited to HTML. It can style any **XML-based document**, including:

- **HTML** — the primary use case
- **SVG** — scalable vector graphics
- **XHTML** — XML-serialized HTML
- **XML** — generic markup (with appropriate processing instructions)
- **MathML** — mathematical notation

**CSS separates content from presentation** — a foundational principle of web architecture.

### 1.3 Anatomy of a CSS Rule

```css
selector {
  property: value;
  property: value;
}
```

**Example:**

```css
h1 {
  color: navy;
  font-size: 2rem;
  margin-bottom: 1rem;
}
```

| Part | Meaning |
|---|---|
| `h1` | **Selector** — which elements to style |
| `color`, `font-size`, `margin-bottom` | **Properties** — what aspect to change |
| `navy`, `2rem`, `1rem` | **Values** — how to change it |
| `color: navy;` | **Declaration** — a single property-value pair |
| `{ ... }` | **Declaration block** — the set of declarations |

---

## 2. Purpose of CSS

CSS serves four broad purposes in web development.

### 2.1 Presentation

CSS controls the **visual appearance** of elements:

| Aspect | Examples |
|---|---|
| **Color** | `color`, `background-color`, `border-color` |
| **Typography** | `font-family`, `font-size`, `line-height`, `letter-spacing` |
| **Spacing** | `margin`, `padding`, `gap` |
| **Borders** | `border`, `border-radius`, `box-shadow` |
| **Backgrounds** | `background-image`, `background-size`, `gradient` |
| **Effects** | `opacity`, `filter`, `blend-mode`, `transform` |

```css
.card {
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  font-family: system-ui, sans-serif;
  color: #333;
}
```

### 2.2 Layout

CSS determines **where elements go** on the page:

| Technique | Purpose |
|---|---|
| **Normal flow** | Default block and inline behavior |
| **Flexbox** | One-dimensional layouts (row or column) |
| **Grid** | Two-dimensional layouts (rows and columns) |
| **Positioning** | `relative`, `absolute`, `fixed`, `sticky` |
| **Floats** | Legacy layout technique |
| **Multi-column** | Newspaper-style column flow |

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}
```

### 2.3 Responsive Adaptation

CSS enables layouts to **adapt to different screen sizes, orientations, and capabilities**:

- **Media queries** — `@media (max-width: 768px) { ... }`
- **Container queries** — style based on parent size
- **Relative units** — `%`, `em`, `rem`, `vw`, `vh`, `ch`
- **Fluid typography** — `clamp(1rem, 2.5vw, 2rem)`
- **Flexible images** — `max-width: 100%`
- **Responsive grid/flex** — `auto-fit`, `minmax()`

```css
/* Mobile-first: default styles for small screens */
.grid {
  display: grid;
  grid-template-columns: 1fr;
}

/* Tablet and up */
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

### 2.4 Visual Interaction

CSS provides **feedback and interaction** without JavaScript:

- **Hover states** — `:hover`
- **Focus states** — `:focus`, `:focus-visible`
- **Active states** — `:active`
- **Transitions** — `transition: all 0.3s ease`
- **Animations** — `@keyframes`, `animation`
- **Transforms** — `translate`, `rotate`, `scale`

```css
.button {
  background-color: #0066cc;
  color: white;
  transition: background-color 0.2s ease, transform 0.1s ease;
}

.button:hover {
  background-color: #0052a3;
}

.button:active {
  transform: scale(0.98);
}

.button:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

---

## 3. Relationship Between HTML, CSS, and JavaScript

The three core technologies of the web have distinct roles.

| Technology | Role | Analogy |
|---|---|---|
| **HTML** | Structure and content | Skeleton |
| **CSS** | Presentation and layout | Skin, clothing, styling |
| **JavaScript** | Behavior and dynamic logic | Muscles, nervous system |

### 3.1 HTML → Structure

HTML defines **what content exists** and its **semantic meaning**:

```html
<article class="post">
  <h1>Article Title</h1>
  <p>Article content...</p>
  <a href="/read-more">Read more</a>
</article>
```

HTML alone renders as plain, unstyled content.

### 3.2 CSS → Presentation and Layout

CSS defines **how the content looks**:

```css
.post {
  max-width: 60ch;
  margin: 2rem auto;
  padding: 1rem;
  font-family: Georgia, serif;
}

.post h1 {
  font-size: 2rem;
  color: #222;
}

.post a {
  color: #0066cc;
  text-decoration: none;
}
```

### 3.3 JavaScript → Behavior and Dynamic Logic

JavaScript defines **how the page behaves** and responds to user input:

```javascript
document.querySelector('.post a').addEventListener('click', (e) => {
  e.preventDefault();
  document.querySelector('.post').classList.toggle('expanded');
});
```

### 3.4 Separation of Concerns

The **separation of concerns** principle holds that each technology should focus on its own domain:

| Principle | Benefit |
|---|---|
| **HTML for content** | Accessible, semantic, SEO-friendly |
| **CSS for presentation** | Themeable, maintainable, responsive |
| **JavaScript for behavior** | Progressive enhancement, non-blocking |

**Anti-pattern (mixing concerns):**

```html
<!-- ✗ Inline styles mix content with presentation -->
<p style="color: red; font-size: 14px;">Warning</p>

<!-- ✗ Inline handlers mix content with behavior -->
<button onclick="doSomething()">Click</button>
```

**Best practice:**

```html
<!-- HTML: structure only -->
<p class="warning">Warning</p>
<button class="action-btn">Click</button>
```

```css
/* CSS: presentation only */
.warning { color: red; font-size: 14px; }
.action-btn { /* ... */ }
```

```javascript
// JavaScript: behavior only
document.querySelector('.action-btn').addEventListener('click', doSomething);
```

### 3.5 Progressive Enhancement

Each layer enhances the previous one:

```
Layer 1: HTML         → content is accessible
Layer 2: HTML + CSS   → content is styled
Layer 3: HTML + CSS + JS → content is interactive
```

If CSS fails to load, content remains readable. If JavaScript fails, content remains styled.

---

## 4. CSS Evolution

CSS has evolved significantly since its introduction in 1996.

### 4.1 CSS1 (1996)

The **first official CSS specification**, published by the W3C in December 1996.

**Key features:**

- Basic typography — fonts, colors, text alignment
- Basic box model — margin, padding, border
- Basic selectors — element, class, ID
- Simple positioning

**Limitations:**

- No layout system beyond basic block/inline
- No media queries
- Limited selectors
- Inconsistent browser implementation

### 4.2 CSS2 (1998)

Published in May 1998, CSS2 added substantial capabilities.

**Key features:**

- **Positioning** — `relative`, `absolute`, `fixed`
- **Media types** — `@media screen`, `@media print`
- **Z-index** — stacking order
- **Generated content** — `:before`, `:after`
- **Bidirectional text** — `direction`, `unicode-bidi`
- **Aural stylesheets** — for screen readers
- **Font properties** — `@font-face` (limited support)

**Limitations:**

- No true layout system (relied on floats and tables)
- No responsive design
- Inconsistent implementation

### 4.3 CSS2.1 (2004–2011)

CSS 2.1 was a **revision of CSS2** that:

- Clarified ambiguities
- Removed poorly supported features
- Corrected errors in CSS2
- Became the **de facto baseline** for browsers

CSS 2.1 was the **most widely implemented** specification for many years — a browser that supported CSS 2.1 well was considered modern.

### 4.4 CSS3 (1999 onward)

CSS3 was a **major shift** in how CSS was specified: instead of a single monolithic specification, CSS3 was **divided into independent modules**.

**Why modularization?**

| Benefit | Explanation |
|---|---|
| **Independent evolution** | Modules can advance at different paces |
| **Faster standardization** | Smaller specs are easier to agree on |
| **Faster implementation** | Browsers can adopt modules incrementally |
| **Clearer scope** | Each module has a focused purpose |

**Key CSS3 modules:**

| Module | Features |
|---|---|
| **Selectors Level 3** | `:nth-child()`, `:not()`, attribute selectors |
| **Colors** | `rgba()`, `hsl()`, `hsla()`, opacity |
| **Backgrounds & Borders** | `border-radius`, `box-shadow`, gradients |
| **Transitions** | Animate property changes |
| **Animations** | `@keyframes`, keyframe animations |
| **Transforms** | 2D and 3D transformations |
| **Flexbox** | One-dimensional layout |
| **Grid** | Two-dimensional layout |
| **Media Queries** | Responsive design |
| **Fonts** | `@font-face` for web fonts |
| **Multi-column** | Column layouts |

CSS3 is **not a single specification** — it is a collection of dozens of independent modules, each at its own level of maturity.

### 4.5 Modern CSS Modules

CSS continues to evolve through active W3C and WHATWG work. Notable modern developments:

| Module | Status | Purpose |
|---|---|---|
| **CSS Grid Layout** | Stable | 2D layout |
| **CSS Flexbox** | Stable | 1D layout |
| **CSS Custom Properties** | Stable | Variables |
| **CSS Nesting** | Stable (2023) | Nested selectors |
| **Container Queries** | Stable (2023) | Style based on parent size |
| **CSS Cascade Layers** | Stable | `@layer` for cascade control |
| **`:has()` selector** | Stable | Parent selection |
| **Logical Properties** | Stable | Writing-mode-aware properties |
| **CSS Color Module Level 4/5** | Evolving | `lab()`, `lch()`, `oklch()`, `color-mix()` |
| **CSS Subgrid** | Stable | Nested grid alignment |
| **View Transitions** | Evolving | Page transition animations |
| **Scroll-driven Animations** | Evolving | Animations tied to scroll |
| **CSS Anchor Positioning** | Evolving | Position elements relative to anchors |
| **`@scope`** | Evolving | Scoped styles |

**Example — modern CSS features:**

```css
:root {
  --brand: oklch(60% 0.2 250);
}

@layer base, components, utilities;

@layer components {
  .card {
    container-type: inline-size;
    background: color-mix(in oklch, var(--brand) 10%, white);
  }

  .card:has(img) {
    padding: 0;
  }
}

@container (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

---

## 5. CSS Standards and Browser Implementation

### 5.1 W3C (World Wide Web Consortium)

The **W3C** is the primary standards body for CSS.

- Founded in **1994** by Tim Berners-Lee
- Develops and maintains **CSS specifications**
- Publishes **Recommendations** (standards), **Candidate Recommendations**, and **Working Drafts**
- Has a **CSS Working Group** that manages the CSS module process

**W3C specification maturity levels:**

| Level | Meaning |
|---|---|
| **Working Draft (WD)** | In development; subject to change |
| **Candidate Recommendation (CR)** | Feature-complete; gathering implementation experience |
| **Proposed Recommendation (PR)** | Awaiting final approval |
| **W3C Recommendation (REC)** | Final standard |
| **Editor's Draft** | Working document; not official |

### 5.2 WHATWG Ecosystem

The **WHATWG** (Web Hypertext Application Technology Working Group) maintains the **HTML Living Standard** and related web platform specs.

- Formed in **2004** by browser vendors (Apple, Mozilla, Opera)
- Focuses on **practical, implementation-driven standards**
- Maintains the **HTML Living Standard** — not versioned
- Collaborates with W3C on overlapping areas

**Relationship to CSS:**

- CSS is primarily a **W3C** domain
- HTML (which CSS styles) is a **WHATWG** domain
- Both groups coordinate on shared concerns
- Browser vendors are represented in both

### 5.3 Browser Engines

Each major browser uses a **rendering engine** that implements CSS.

| Browser | Engine | Vendor |
|---|---|---|
| **Chrome** | Blink | Google |
| **Edge** | Blink | Microsoft |
| **Firefox** | Gecko | Mozilla |
| **Safari** | WebKit | Apple |
| **Opera** | Blink | Opera |

**Engine lineage:**

```
KHTML (KDE)
  └── WebKit (Apple)
        ├── WebKit (Safari)
        └── Blink (Google, fork 2013)
              ├── Chrome
              ├── Edge (2019+)
              └── Opera (2013+)
```

**Why engines matter:**

- Each engine implements CSS independently
- Features may appear at different times
- Bugs differ between engines
- Testing across engines is essential

### 5.4 Compatibility Considerations

CSS implementation varies across browsers. Common compatibility concerns:

#### Vendor Prefixes

Historically, browsers used **vendor prefixes** for experimental features:

| Prefix | Browser |
|---|---|
| `-webkit-` | Chrome, Safari, Edge (Blink/WebKit) |
| `-moz-` | Firefox |
| `-ms-` | Internet Explorer, early Edge |
| `-o-` | Opera (pre-Blink) |

**Example (historical):**

```css
.box {
  -webkit-border-radius: 8px;
  -moz-border-radius: 8px;
  border-radius: 8px;
}
```

**Modern practice:** Prefixes are rarely needed for stable features. Use **Autoprefixer** to add them automatically based on browser support targets.

#### Feature Support

Not every browser supports every CSS feature:

| Feature | Support |
|---|---|
| Flexbox | Universal (modern) |
| Grid | Universal (modern) |
| Custom Properties | Universal (modern) |
| Container Queries | Modern browsers (2023+) |
| `:has()` | Modern browsers (2023+) |
| Nesting | Modern browsers (2023+) |
| `oklch()` | Modern browsers (2023+) |

**Checking support:**

- [Can I use](https://caniuse.com) — feature support tables
- [MDN Web Docs](https://developer.mozilla.org) — compatibility tables
- [CSS Feature Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_conditional_rules/Using_feature_queries) — `@supports`

#### Feature Detection with `@supports`

```css
@supports (display: grid) {
  .container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
}

@supports not (display: grid) {
  .container {
    display: flex;
    flex-wrap: wrap;
  }
}
```

#### Progressive Enhancement

Provide a **baseline experience** that works everywhere, then enhance:

```css
/* Baseline: works everywhere */
.grid {
  display: flex;
  flex-wrap: wrap;
}

/* Enhancement: modern browsers get better layout */
@supports (display: grid) {
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  }
}
```

#### Reset vs. Normalize

Browsers apply **default styles** inconsistently. Two approaches:

| Approach | Purpose |
|---|---|
| **CSS Reset** | Remove all default styles (`* { margin: 0; }`) |
| **Normalize.css** | Make defaults consistent across browsers |

Modern practice: use a **minimal reset** or **modern-normalize**, then define your own base styles.

```css
/* Simple reset */
*, *::before, *::after {
  box-sizing: border-box;
}

body, h1, h2, h3, p, figure {
  margin: 0;
}

img, picture, video {
  max-width: 100%;
  display: block;
}
```

### 5.5 Browser Adoption Timelines

New CSS features follow a **predictable path** from proposal to universal support:

```
1. Editor's Draft       — actively edited
2. Working Draft        — W3C WD published
3. Implementation       — one or more browsers ship it
4. Candidate Rec        — feature-complete, gathering data
5. W3C Recommendation   — final standard
6. Baseline             — supported in all modern browsers
```

**Baseline** (from web.dev) tracks feature availability:

| Baseline Status | Meaning |
|---|---|
| **Newly available** | Just shipped in all major browsers |
| **Widely available** | Available in browsers for 30+ months |
| **Limited availability** | Not yet in all browsers |

---

## Summary Table

| Topic | Key Points |
|---|---|
| **Definition** | Cascading Style Sheets — a stylesheet language for HTML/XML |
| **Purpose** | Presentation, layout, responsive adaptation, visual interaction |
| **HTML** | Structure and content |
| **CSS** | Presentation and layout |
| **JavaScript** | Behavior and dynamic logic |
| **CSS1 (1996)** | Basic typography, colors, box model |
| **CSS2 (1998)** | Positioning, media types, z-index |
| **CSS2.1 (2004–2011)** | Corrected baseline, widely implemented |
| **CSS3 (1999+)** | Modularized; selectors, transitions, flexbox, grid |
| **Modern modules** | Container queries, nesting, `:has()`, cascade layers, `oklch()` |
| **W3C** | Primary CSS standards body |
| **WHATWG** | HTML Living Standard; collaborates with W3C |
| **Browser engines** | Blink, Gecko, WebKit |
| **Compatibility** | Vendor prefixes (mostly historical), `@supports`, progressive enhancement |

---

## Key Takeaways

1. **CSS** stands for **Cascading Style Sheets** — a stylesheet language for styling HTML and XML documents.
2. CSS handles **presentation, layout, responsive adaptation, and visual interaction**.
3. **Separation of concerns** — HTML for structure, CSS for presentation, JavaScript for behavior.
4. CSS evolved through **CSS1 → CSS2 → CSS2.1 → CSS3 → modern modules**, shifting from a monolithic spec to **independent modules**.
5. **CSS3 is not a single specification** — it is dozens of modules, each evolving at its own pace.
6. Modern CSS includes **Grid, Flexbox, Custom Properties, Nesting, Container Queries, `:has()`, Cascade Layers, and modern color spaces**.
7. The **W3C** maintains CSS specifications; the **WHATWG** maintains HTML.
8. Browser engines (**Blink, Gecko, WebKit**) implement CSS independently, causing timing and behavior differences.
9. **Feature detection** (`@supports`), **progressive enhancement**, and **Baseline** are the modern approaches to compatibility.
10. **Vendor prefixes** are largely historical — prefer feature detection and graceful degradation.

---

Would you like me to continue with the next topic — **CSS Syntax and Selectors**, **CSS Box Model**, or **CSS Specificity and the Cascade**? I can format the next section in the same style.