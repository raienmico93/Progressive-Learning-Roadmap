# CSS Syntax

CSS has a simple, declarative syntax. Understanding its structure — rule sets, selectors, declarations, properties, and values — is the foundation for writing stylesheets that browsers parse correctly and predictably.

---

## 1. Rule Sets

A **rule set** (often just called a **rule**) is the fundamental unit of CSS. It consists of a **selector** and a **declaration block**.

### General Form

```css
selector {
  property: value;
  property: value;
}
```

### Anatomy

```css
h1 {
  color: navy;
  font-size: 2rem;
}
```

| Part | Name | Purpose |
|---|---|---|
| `h1` | **Selector** | Targets which elements to style |
| `{ ... }` | **Declaration block** | Contains the styles to apply |
| `color: navy;` | **Declaration** | A single property-value pair |
| `color` | **Property** | What aspect to change |
| `navy` | **Value** | How to change it |

### Multiple Selectors, One Rule Set

Multiple selectors can share a declaration block using commas:

```css
h1, h2, h3 {
  font-family: Georgia, serif;
  color: #333;
}
```

This applies the same declarations to `h1`, `h2`, and `h3`.

### Rule Set vs. At-Rule

A **rule set** has a selector and declaration block. An **at-rule** begins with `@` and has different syntax:

```css
/* Rule set */
p { color: red; }

/* At-rules */
@media (min-width: 768px) { /* ... */ }
@import url("styles.css");
@font-face { /* ... */ }
@keyframes slide { /* ... */ }
@supports (display: grid) { /* ... */ }
@layer base, components;
```

At-rules are covered in a separate topic; here we focus on rule sets.

---

## 2. Selectors

A **selector** identifies which elements in the document a rule applies to.

### Basic Selector Types

| Selector | Syntax | Matches |
|---|---|---|
| **Universal** | `*` | All elements |
| **Type (element)** | `p` | All `<p>` elements |
| **Class** | `.button` | Elements with `class="button"` |
| **ID** | `#header` | Element with `id="header"` |
| **Attribute** | `[type="text"]` | Elements with matching attribute |
| **Pseudo-class** | `:hover` | Elements in a state |
| **Pseudo-element** | `::before` | Generated content |
| **Descendant** | `nav a` | `<a>` inside `<nav>` |
| **Child** | `ul > li` | Direct `<li>` children of `<ul>` |
| **Adjacent sibling** | `h1 + p` | `<p>` immediately after `<h1>` |
| **General sibling** | `h1 ~ p` | Any `<p>` after `<h1>` |

### Examples

```css
* { box-sizing: border-box; }              /* universal */
p { line-height: 1.6; }                    /* type */
.button { padding: 0.5rem 1rem; }          /* class */
#header { background: navy; }              /* ID */
input[type="email"] { border: 1px solid; } /* attribute */
a:hover { text-decoration: underline; }    /* pseudo-class */
p::first-line { font-weight: bold; }       /* pseudo-element */
nav a { color: white; }                    /* descendant */
ul > li { list-style: none; }              /* child */
h1 + p { margin-top: 0; }                  /* adjacent sibling */
h1 ~ p { color: gray; }                    /* general sibling */
```

### Selector Lists

Comma-separated selectors share a declaration block:

```css
h1, h2, h3, h4, h5, h6 {
  font-family: system-ui, sans-serif;
  margin-top: 0;
}
```

**Note:** A single invalid selector in a list invalidates the **entire rule** in older browsers. Modern browsers use **selector lists** with forgiveness in some contexts (e.g., `:is()`, `:where()`).

```css
/* If .foo is invalid, the whole rule is dropped in older browsers */
h1, .foo, h2 { color: red; }
```

### Specificity Preview

Selectors have **specificity** — a weight that determines which rule wins when multiple rules match:

| Selector | Specificity (a, b, c) |
|---|---|
| `*` | (0, 0, 0) |
| `p` | (0, 0, 1) |
| `.class` | (0, 1, 0) |
| `#id` | (1, 0, 0) |
| Inline style | (1, 0, 0, 0) |
| `!important` | Highest |

Specificity is covered in detail in the Cascade topic.

---

## 3. Declarations

A **declaration** is a single **property-value pair** inside a declaration block.

### Syntax

```css
property: value;
```

**Example:**

```css
color: navy;
```

- `color` — property
- `navy` — value
- `;` — semicolon (terminator)

### Declarations in Context

```css
h1 {
  color: navy;
  font-size: 2rem;
  margin-bottom: 1rem;
}
```

This block contains **three declarations**.

### Declaration Rules

| Rule | Example |
|---|---|
| Property and value separated by `:` | `color: navy` |
| Terminated by `;` | `color: navy;` |
| Whitespace around `:` is optional | `color:navy` and `color : navy` |
| Multiple declarations separated by `;` | `color: navy; font-size: 2rem;` |
| Final `;` is optional but recommended | `color: navy` works alone |

### Shorthand vs. Longhand Declarations

**Longhand** — one property per declaration:

```css
margin-top: 1rem;
margin-right: 2rem;
margin-bottom: 1rem;
margin-left: 2rem;
```

**Shorthand** — one property for multiple values:

```css
margin: 1rem 2rem;
```

Shorthands are concise but can inadvertently reset values you didn't intend to change.

---

## 4. Properties

A **property** is the aspect of an element's presentation that you want to change.

### Categories of Properties

| Category | Examples |
|---|---|
| **Color** | `color`, `background-color`, `border-color` |
| **Typography** | `font-family`, `font-size`, `line-height`, `letter-spacing` |
| **Box model** | `width`, `height`, `margin`, `padding`, `border` |
| **Layout** | `display`, `position`, `float`, `flex`, `grid-template-columns` |
| **Backgrounds** | `background`, `background-image`, `background-size` |
| **Borders** | `border`, `border-radius`, `border-style` |
| **Effects** | `opacity`, `box-shadow`, `filter`, `transform` |
| **Animation** | `transition`, `animation`, `@keyframes` |
| **Custom properties** | `--brand-color`, `--spacing` |

### Property Naming

- **Lowercase** by convention
- **Hyphenated** (kebab-case) — `font-size`, `background-color`
- **No spaces** in property names
- **Case-insensitive** — `COLOR`, `Color`, and `color` are equivalent

### Vendor-Prefixed Properties

Historical and experimental properties use vendor prefixes:

```css
.box {
  -webkit-border-radius: 8px;
  -moz-border-radius: 8px;
  border-radius: 8px;
}
```

| Prefix | Vendor |
|---|---|
| `-webkit-` | Chrome, Safari, Edge |
| `-moz-` | Firefox |
| `-ms-` | Internet Explorer / legacy Edge |
| `-o-` | Opera (pre-Blink) |

Modern practice: use standard properties; let Autoprefixer add prefixes based on your browser targets.

### Custom Properties (CSS Variables)

Custom properties begin with `--` and can hold any value:

```css
:root {
  --brand-color: #0066cc;
  --spacing-unit: 0.5rem;
}

.button {
  background-color: var(--brand-color);
  padding: var(--spacing-unit) calc(var(--spacing-unit) * 2);
}
```

Custom properties are **case-sensitive** (unlike standard properties).

---

## 5. Values

A **value** is what you assign to a property.

### Types of Values

| Type | Examples |
|---|---|
| **Keywords** | `auto`, `inherit`, `initial`, `none`, `block` |
| **Lengths** | `10px`, `1.5rem`, `2em`, `50%`, `100vh`, `10ch` |
| **Colors** | `red`, `#ff0000`, `rgb(255, 0, 0)`, `oklch(60% 0.2 250)` |
| **Numbers** | `1`, `0.5`, `-2` |
| **Strings** | `"Open Sans"`, `'Arial'` |
| **URLs** | `url("image.png")`, `url(image.png)` |
| **Functions** | `calc(100% - 2rem)`, `min(2rem, 5vw)`, `clamp(1rem, 2vw, 2rem)` |
| **Lists** | `1px solid black`, `Arial, Helvetica, sans-serif` |
| **Multiple values** | `margin: 1rem 2rem` |

### Value Syntax Forms

| Form | Example | Meaning |
|---|---|---|
| **Single value** | `color: red` | One value |
| **Space-separated** | `margin: 1rem 2rem` | Multiple values in one declaration |
| **Comma-separated** | `font-family: Arial, sans-serif` | Alternative values |
| **Slash-separated** | `font: 16px/1.5 Arial` | Shorthand with ratio |

### Units

**Absolute units:**

| Unit | Meaning |
|---|---|
| `px` | Pixels |
| `cm`, `mm`, `in`, `pt`, `pc` | Physical units (rarely used in screen design) |

**Relative units:**

| Unit | Relative To |
|---|---|
| `em` | Parent font size |
| `rem` | Root font size |
| `%` | Parent property |
| `vw`, `vh` | Viewport width/height |
| `vmin`, `vmax` | Smaller/larger viewport dimension |
| `ch` | Width of "0" character |
| `ex` | Height of "x" character |
| `lh` | Line height |
| `svh`, `lvh`, `dvh` | Small/large/dynamic viewport height |

### Colors

```css
/* Keyword */
color: red;

/* Hex */
color: #ff0000;
color: #f00;
color: #ff000080;  /* with alpha */

/* RGB / RGBA */
color: rgb(255, 0, 0);
color: rgba(255, 0, 0, 0.5);

/* HSL / HSLA */
color: hsl(0, 100%, 50%);
color: hsla(0, 100%, 50%, 0.5);

/* Modern color spaces */
color: oklch(60% 0.2 250);
color: lab(50% 40 30);
color: color(display-p3 1 0 0);
```

### Functions

```css
width: calc(100% - 2rem);
font-size: clamp(1rem, 2.5vw, 2rem);
padding: min(2rem, 5%);
background: linear-gradient(to right, red, blue);
transform: translateX(50%) rotate(45deg);
```

### Global Values

These keywords apply to **any** property:

| Value | Meaning |
|---|---|
| `inherit` | Take the parent's computed value |
| `initial` | Reset to the property's default value |
| `unset` | `inherit` if inherited, `initial` if not |
| `revert` | Revert to user-agent or user stylesheet value |
| `revert-layer` | Revert to the previous cascade layer |

```css
button {
  color: inherit;      /* use parent's color */
  margin: initial;     /* reset to default */
  all: unset;          /* reset every property */
}
```

---

## 6. Declaration Blocks

A **declaration block** is the `{ ... }` portion of a rule set, containing one or more declarations.

### Syntax

```css
selector {
  declaration;
  declaration;
  ...
}
```

### Structure

```css
.card {
  /* declaration block starts */
  background: white;
  border: 1px solid #ddd;
  padding: 1rem;
  /* declaration block ends */
}
```

### Block Rules

| Rule | Description |
|---|---|
| Enclosed in `{ }` | Required |
| Contains zero or more declarations | Empty blocks are valid but useless |
| Each declaration ends with `;` | Final `;` is optional |
| Whitespace is flexible | Newlines, spaces, tabs are ignored |
| Comments can appear inside | `/* ... */` |
| Nested blocks are **not** allowed | Except in `@media`, `@supports`, etc. |

### Empty Blocks

```css
.empty {
  /* valid, but does nothing */
}
```

### Nested At-Rules

CSS does not allow nested declaration blocks **inside a rule set** in the traditional syntax — but **CSS Nesting** (2023) now permits it:

```css
/* Traditional CSS — no nesting */
.card { background: white; }
.card .title { font-weight: bold; }

/* Modern CSS Nesting */
.card {
  background: white;
  .title {
    font-weight: bold;
  }
}
```

---

## 7. Comments

CSS supports **only one comment syntax**: `/* ... */`.

### Single-Line Comments

```css
/* This is a comment */
h1 { color: navy; }
```

### Multi-Line Comments

```css
/*
 * This is a multi-line comment.
 * It can span as many lines as needed.
 */
h1 { color: navy; }
```

### Inline Comments

```css
h1 {
  color: navy;      /* primary heading color */
  font-size: 2rem;  /* adjust for typography scale */
}
```

### Key Rules

| Rule | Description |
|---|---|
| Only `/* ... */` is valid | `//` is **not** a comment in CSS |
| Comments can span lines | Yes |
| Comments can be inside declaration blocks | Yes |
| Comments are stripped at parse time | Not sent to the browser after minification |
| Comments cannot be nested | `/* outer /* inner */ still outer? */` is invalid |

### What is NOT a Comment

```css
// This is NOT a comment in CSS — it's a syntax error
h1 { color: navy; }  // So is this
```

**But** some preprocessors (Sass, Less) accept `//` comments. **Plain CSS does not.**

### Common Comment Uses

```css
/* ========================================
   Layout
   ======================================== */

.container { /* ... */ }

/* ---- Header ---- */
.header { /* ... */ }

/* TODO: revisit after design review */
.sidebar { /* ... */ }
```

### Comments for Section Organization

```css
/* ==========================================================================
   1. Reset
   2. Base typography
   3. Layout
   4. Components
   5. Utilities
   ========================================================================== */
```

---

## 8. Whitespace

CSS ignores most whitespace — but not all.

### Where Whitespace Is Flexible

| Location | Whitespace Behavior |
|---|---|
| Between declarations | Ignored |
| Around `{` and `}` | Ignored |
| Around `:` | Ignored (mostly) |
| Between selector parts | **Significant in some contexts** |
| Inside values | **Significant in some contexts** |

### Equivalent Examples

These are all equivalent:

```css
h1{color:navy;font-size:2rem}
```

```css
h1 { color: navy; font-size: 2rem; }
```

```css
h1
{
  color:
    navy
  ;
  font-size
    :
    2rem
  ;
}
```

### Where Whitespace Matters

**1. Descendant combinator** — space is a combinator:

```css
nav a { }      /* a inside nav */
nav a         /* different from */
nav>a { }     /* direct child */
```

**2. Values with multiple parts** — space-separated values:

```css
margin: 1rem 2rem;     /* two values */
margin: 1rem2rem;      /* one value — likely invalid */
```

**3. `!important`** — space before `!` is optional but conventional:

```css
color: red !important;
color: red!important;   /* also valid */
```

**4. Between property and colon** — space is optional:

```css
color: red;     /* conventional */
color : red;    /* valid but unusual */
color:red;      /* valid */
```

**5. Inside `calc()`** — **spaces are required around `+` and `-`**:

```css
width: calc(100% - 2rem);   /* ✓ spaces required */
width: calc(100%-2rem);     /* ✗ invalid */
width: calc(100% + 2rem);   /* ✓ */
width: calc(100%/3);        /* ✓ — no spaces needed for * and / */
```

### Whitespace in Selectors

```css
/* Space = descendant combinator */
div p { }        /* p inside div */

/* No space = compound selector */
div.p { }        /* div with class "p" */

/* Space around > is optional */
ul > li { }
ul>li { }
ul >li { }
ul> li { }
```

All four child combinator forms are valid.

### Minification

Minifiers remove unnecessary whitespace, comments, and final semicolons:

```css
/* Before */
h1 {
  color: navy;
  font-size: 2rem;
}

/* After minification */
h1{color:navy;font-size:2rem}
```

---

## 9. Semicolons

The **semicolon** (`;`) terminates each declaration.

### Rules

| Rule | Description |
|---|---|
| Terminates a declaration | `color: navy;` |
| Optional after **last** declaration in a block | `{ color: navy }` is valid |
| Required between declarations | `{ color: navy font-size: 2rem }` is invalid |
| Whitespace around it is optional | `color: navy ;` and `color: navy;` are equivalent |
| Extra semicolons are ignored | `color: navy;;;` is valid |

### Missing Semicolon — Common Bug

```css
/* ✗ Missing semicolon after "navy" */
h1 {
  color: navy
  font-size: 2rem;
}
```

The browser tries to parse `color: navy font-size: 2rem;` as one declaration. `navy font-size: 2rem` is not a valid value for `color`, so the **entire declaration is dropped**.

**Result:** Both `color` and `font-size` fail to apply.

### Why the Final Semicolon Matters

The final semicolon is **optional**:

```css
/* Valid */
h1 {
  color: navy;
  font-size: 2rem
}
```

But it's **strongly recommended** because:

1. **Easier to add new declarations** — no need to remember the previous line lacked `;`
2. **Reduces bugs during editing** — pasting declarations is safer
3. **Matches most style guides** (Google, Airbnb, Standard)
4. **Prevents parse errors** — a missing semicolon in the middle is a common mistake

### Semicolon in `@import` and At-Rules

Some at-rules end with `;`:

```css
@import url("reset.css");
@charset "UTF-8";
```

Others use blocks:

```css
@media (min-width: 768px) { /* ... */ }
@font-face { /* ... */ }
```

### Semicolon After Rule Sets

A semicolon after a rule set's closing brace is **ignored**:

```css
h1 { color: navy; };   /* the trailing ; is harmless but unnecessary */
```

Some developers omit it; others use it for consistency with preprocessor syntax.

---

## 10. Invalid Declarations

CSS is **forgiving**: an invalid declaration is **dropped**, but the rest of the rule continues to apply.

### Error Recovery Model

When the parser encounters an invalid declaration:

1. It discards the invalid declaration
2. It continues parsing the remaining declarations
3. The rest of the rule set is preserved

**Example:**

```css
h1 {
  color: navy;
  font-size: 2rem;      /* ✓ valid */
  colour: red;          /* ✗ typo — invalid property */
  margin: 1rem;
}
```

**Result:**

```css
h1 {
  color: navy;
  font-size: 2rem;
  margin: 1rem;
}
```

The `colour: red;` declaration is silently dropped.

### Common Causes of Invalid Declarations

| Cause | Example |
|---|---|
| Typo in property name | `colr: red;` |
| Typo in value | `color: navyblue;` |
| Missing unit | `font-size: 16;` (invalid; needs unit) |
| Wrong unit | `width: 10deg;` |
| Missing semicolon | `color: navy font-size: 2rem;` |
| Invalid function | `width: calc(100% - );` |
| Invalid color | `color: #ff00;` (4 digits is invalid) |
| Unknown keyword | `display: flexbox;` |
| Unclosed string | `content: "unclosed;` |
| Unmatched bracket | `width: calc(100% - 2rem;` |

### Invalid Rules

Invalid **selectors** cause the entire rule to be dropped:

```css
/* Invalid selector — whole rule dropped */
h1 .. {
  color: red;
}
```

Modern browsers may drop only the invalid part in some contexts (e.g., `:is()` forgives invalid selectors).

### Invalid At-Rules

Unknown at-rules are ignored:

```css
@unknown-rule {
  /* ignored */
}
```

But `@media` queries with invalid syntax may be dropped.

### How Invalid Declarations Are Handled

**Without recovery:**
```css
h1 {
  color: navy;
  font-size: 16;        /* invalid — missing unit */
  margin: 1rem;
}
```

The `font-size: 16;` declaration is dropped. `color` and `margin` still apply.

**Entire rule dropped:**
```css
h1 {                     /* ← invalid selector */
  color: navy;
}
```

This drops the whole rule.

### Silent Failure — Why It's Dangerous

CSS **does not throw errors**. Invalid declarations fail silently. This means:

- Typos are easy to miss
- A single bad character can break a declaration
- Debugging requires DevTools

**Best practices:**

1. Use a **linter** (Stylelint)
2. Use **DevTools** to inspect computed styles
3. **Validate** with the W3C CSS Validator
4. Write **consistent, formatted** CSS

### Debugging Invalid Declarations

In browser DevTools, invalid declarations appear **struck through**:

```
color: navy;            ← applied
font-size: 16;          ← struck through (invalid)
margin: 1rem;           ← applied
```

This makes it easy to spot parse errors.

### `@supports` for Feature Detection

Instead of relying on invalid declarations, use `@supports`:

```css
/* ✗ Relies on invalid declaration being dropped */
.box {
  display: flex;
  display: grid;    /* overrides in supporting browsers */
}

/* ✓ Explicit feature detection */
.box {
  display: flex;
}

@supports (display: grid) {
  .box {
    display: grid;
  }
}
```

Both work, but `@supports` is more explicit and readable.

---

## 11. Multiple Declarations

A declaration block can contain any number of declarations.

### Syntax

```css
selector {
  property1: value1;
  property2: value2;
  property3: value3;
}
```

### Example

```css
.card {
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1.5rem;
  font-family: system-ui, sans-serif;
  font-size: 1rem;
  line-height: 1.5;
  color: #333;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

Nine declarations in one rule set.

### Order of Declarations

Declarations are applied in the order they appear, **but** order only matters when the same property is set twice:

```css
.box {
  color: red;
  color: blue;   /* wins — later declaration overrides earlier */
}
```

Different properties don't conflict:

```css
.box {
  color: red;
  font-size: 2rem;
  /* both apply */
}
```

### Shorthand and Longhand Order

When mixing shorthands and longhands, **order matters**:

```css
/* Longhand after shorthand — longhand wins */
.box {
  margin: 1rem;         /* sets all four sides to 1rem */
  margin-top: 2rem;     /* overrides top only */
}

/* Shorthand after longhand — shorthand wins */
.box {
  margin-top: 2rem;
  margin: 1rem;         /* overrides all four sides, including top */
}
```

**Rule:** Shorthands reset all their longhand sub-properties. Declare shorthands first, then longhands to override specific values.

### Overriding with `!important`

```css
.box {
  color: red !important;
  color: blue;          /* does NOT override — !important wins */
}
```

`!important` overrides normal declarations, regardless of order.

### Grouping with Selector Lists

Multiple selectors share the same declarations:

```css
h1, h2, h3 {
  font-family: Georgia, serif;
  color: #222;
  margin-bottom: 1rem;
}
```

### Multiple Rules for the Same Element

An element can be styled by **multiple rule sets**:

```css
p {
  color: black;
  line-height: 1.6;
}

.intro {
  font-size: 1.25rem;
}

#lead {
  color: navy;
}
```

```html
<p id="lead" class="intro">Hello</p>
```

All three rules apply. The final computed style is determined by **specificity and the cascade**:

- `color: navy` (from `#lead`) wins over `color: black` (from `p`)
- `line-height: 1.6` applies
- `font-size: 1.25rem` applies

### Performance Consideration

More declarations → more work for the browser. But:

- Modern browsers handle thousands of declarations efficiently
- **Selector complexity** matters more than declaration count
- **Layout-triggering properties** (e.g., `width`, `top`) are more expensive than paint-only properties (e.g., `color`)

**Optimization tip:** Move animated properties to `transform` and `opacity` — they can be handled by the compositor.

### Declaration Count Limits

There is **no practical limit** on the number of declarations. However, maintainability suffers with very large rule sets.

**Refactor when a rule set exceeds ~15–20 declarations** — consider splitting into smaller, focused components.

---

## Full Example — All Concepts Together

```css
/* ==========================================================================
   Typography
   ========================================================================== */

/* Multiple selectors, shared declarations */
h1, h2, h3 {
  font-family: Georgia, serif;
  color: #222;
  line-height: 1.2;
  margin-bottom: 0.5em;   /* comment after declaration */
}

/* Single selector, multiple declarations */
h1 {
  font-size: 2.5rem;
  font-weight: 700;
  letter-spacing: -0.02em;

  /* invalid declaration silently dropped */
  colr: red;

  /* valid — overrides previous if same property */
  color: navy;
}

/* Custom properties */
:root {
  --brand: #0066cc;
  --spacing: 1rem;
}

/* Rule with many declarations and a nested at-rule */
.button {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: var(--spacing) calc(var(--spacing) * 2);
  background: var(--brand);
  color: white;
  border: none;
  border-radius: 4px;
  font: inherit;
  cursor: pointer;
  transition: background-color 0.2s ease, transform 0.1s ease;
}

.button:hover {
  background: color-mix(in oklch, var(--brand) 80%, black);
}

.button:active {
  transform: scale(0.98);
}

/* Media query — at-rule wrapping rule sets */
@media (min-width: 768px) {
  h1 {
    font-size: 3rem;
  }
}
```

---

## Summary Table

| Concept | Definition | Example |
|---|---|---|
| **Rule set** | Selector + declaration block | `h1 { color: navy; }` |
| **Selector** | Targets elements | `h1`, `.class`, `#id` |
| **Declaration** | Property-value pair | `color: navy;` |
| **Property** | Aspect to change | `color`, `font-size` |
| **Value** | What to set it to | `navy`, `2rem` |
| **Declaration block** | `{ ... }` containing declarations | `{ color: navy; }` |
| **Comment** | Ignored by parser | `/* ... */` |
| **Whitespace** | Mostly insignificant | Flexible except in combinators and `calc()` |
| **Semicolon** | Terminates declarations | `color: navy;` |
| **Invalid declaration** | Dropped silently | `colr: red;` |
| **Multiple declarations** | Many in one block | `{ color: red; font-size: 2rem; }` |

---

## Key Takeaways

1. A **rule set** = selector + declaration block; the fundamental unit of CSS.
2. **Selectors** target elements; **declaration blocks** contain the styles.
3. A **declaration** is a single `property: value;` pair.
4. **Properties** are hyphenated, lowercase, and case-insensitive.
5. **Values** can be keywords, lengths, colors, functions, or lists.
6. **Declaration blocks** are enclosed in `{ }` and contain zero or more declarations.
7. **CSS comments** use `/* ... */` only — `//` is **not** a comment.
8. **Whitespace** is mostly flexible — but **matters** in combinators, multi-value lists, and inside `calc()` around `+` and `-`.
9. **Semicolons** terminate declarations; the final one is optional but recommended.
10. **Invalid declarations** are **silently dropped** — CSS never throws parse errors.
11. **Multiple declarations** are applied in order; later declarations override earlier ones for the same property.
12. **Shorthands reset longhands** — declare shorthands first, then longhands.
13. Use **DevTools**, **Stylelint**, and `@supports` to catch and manage invalid declarations.

---

Would you like me to continue with the next topic — **CSS Selectors in Depth**, **CSS Specificity and the Cascade**, or **CSS Box Model**? I can format the next section in the same style.