# CSS Modern Selectors — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Modern Selectors are the enhanced selector features introduced in Selectors Level 4 that give authors more powerful, flexible, and forgiving ways to match elements. They include functional pseudo-classes like `:is()`, `:where()`, `:has()`, and an expanded `:not()`, as well as state-based selectors like `:focus-visible` and `:user-valid`.

**Technical Definition:** Selectors Level 4 introduces a set of functional pseudo-classes that accept selector lists as arguments, enabling more compact and expressive CSS. The `:is()` and `:where()` pseudo-classes accept a forgiving selector list and match any element that can be selected by one of the selectors in the list, with `:is()` adopting the specificity of its most specific argument while `:where()` always has zero specificity. The `:has()` pseudo-class is a relational pseudo-class that takes a relative selector list as an argument and represents an element if any of the relative selectors match at least one element when anchored against that element. The `:not()` pseudo-class has been modernized to accept a complex selector list, and its specificity is replaced by the specificity of the most specific selector in its argument. State-based pseudo-classes like `:focus-visible` and `:user-valid` provide more nuanced control over interactive states.

**Beginner-Friendly Explanation:** Modern CSS selectors are like a Swiss Army knife for matching elements. `:is()` lets you group multiple selectors together without repeating yourself. `:where()` does the same but with zero specificity, so it never wins fights with other styles. `:has()` is the long-awaited "parent selector" — it lets you style a parent based on what is inside it. `:not()` now accepts complex selectors, so you can exclude more precisely. And selectors like `:focus-visible` and `:user-valid` let you style elements based on how the user is interacting with them, in a way that respects their input device and interaction history.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Forgiving parsing** | `:is()` and `:where()` accept forgiving selector lists; invalid selectors are ignored, not the whole list. |
| **Specificity control** | `:is()` takes the highest specificity of its arguments; `:where()` has zero specificity. |
| **Relational matching** | `:has()` selects elements based on their descendants or siblings. |
| **Complex negation** | `:not()` accepts complex selector lists, not just simple selectors. |
| **State awareness** | `:focus-visible` and `:user-valid` provide nuanced state-based styling. |
| **Compact syntax** | These selectors significantly reduce repetition in complex selector chains. |
| **Browser support** | Baseline widely available since 2021–2023. |

---

### Prerequisites

Before studying CSS Modern Selectors, you should understand:

- **CSS Selectors** — element, class, ID, attribute, and combinator selectors.
- **CSS Specificity** — how the cascade determines which declaration wins.
- **CSS Pseudo-classes** — `:hover`, `:focus`, `:active`, and their state-based nature.
- **CSS Selector Lists** — comma-separated groups of selectors.
- **The CSS Cascade** — how declarations are resolved.

---

### Related Programming Areas

- **CSS Cascade Layers** — `:where()` is often used to reduce specificity in layered architectures.
- **CSS Custom Properties** — modern selectors work alongside custom properties for theming.
- **Web Accessibility** — `:focus-visible` and `:user-valid` improve keyboard and form accessibility.
- **Component-Driven CSS** — `:has()` enables parent-based styling without JavaScript.
- **Design Systems** — modern selectors reduce specificity conflicts in large codebases.

---

### Core Concepts / Features

1. The Forgiving Selectors: `:is()` and `:where()`
2. Specificity Modification: `:is()` vs. `:where()`
3. The Relational Parent Selector: `:has()`
4. Advanced Negation: The Modernized `:not()`
5. State and Media Queries as Selectors

---

## 1. The Forgiving Selectors: `:is()` and `:where()`

### Definitions

**Core Definition:** `:is()` and `:where()` are functional pseudo-classes that accept a selector list as an argument and match any element that can be selected by one of the selectors in that list. They are "forgiving" because an invalid selector in the list does not invalidate the entire list.

**Technical Definition:** The `:is()` CSS pseudo-class function takes a selector list as its argument and selects any element that can be selected by one of the selectors in that list. The `:where()` pseudo-class is functionally identical to `:is()` but always has zero specificity. The specification defines `:is()` and `:where()` as accepting a forgiving selector list. In CSS, when using a selector list, if any of the selectors are invalid, the whole list is deemed invalid. When using `:is()` or `:where()`, if a selector fails to parse, the incorrect or unsupported selector is ignored and the others are used, so the whole list is not invalidated. The list must not contain pseudo-elements, but any other simple, compound, and complex selectors are allowed.

**Beginner-Friendly Explanation:** Imagine you want to style all headings (`h1` through `h6`) inside any of several sectioning elements (`section`, `article`, `aside`, `nav`). Without `:is()`, you would have to write out every combination — a very long selector list. With `:is()`, you write `:is(section, article, aside, nav) :is(h1, h2, h3, h4, h5, h6)`. The "forgiving" part means that if you include a selector the browser does not understand (like a new pseudo-class), the browser will just ignore that one and still apply the rest, instead of throwing out the whole rule.

---

### Purposes

- To dramatically shorten complex, repetitive selector lists.
- To group multiple selectors without repeating the surrounding context.
- To provide forgiving parsing so that one unsupported selector does not break the entire rule.
- To enable more maintainable and readable CSS.
- To serve as the foundation for specificity control via `:where()`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
:is(<forgiving-selector-list>) {
    /* ... */
}

:where(<forgiving-selector-list>) {
    /* ... */
}
```

#### Component Breakdown

| Function | Argument | Specificity | Forgiving? |
|---|---|---|---|
| `:is()` | Forgiving selector list | Highest of its arguments | Yes |
| `:where()` | Forgiving selector list | Zero | Yes |

#### Syntax Rules

1. `:is()` and `:where()` accept a comma-separated selector list as their argument.
2. The list must not contain pseudo-elements, but may contain simple, compound, and complex selectors.
3. The entire list is not invalidated if one selector fails to parse — the invalid selector is ignored and the others are used.
4. `:is()` adopts the specificity of the most specific selector in its argument list.
5. `:where()` always has zero specificity, regardless of its arguments.
6. Both are Baseline widely available since January 2021.

#### Constraints and Limitations

- **No pseudo-elements** — pseudo-elements like `::before` are not valid inside `:is()` or `:where()`.
- **Specificity of `:is()`** — the highest specificity in the argument list becomes the specificity of the entire `:is()` selector.
- **Nested `:is()`/`:where()`** — nesting is allowed but can make specificity hard to reason about.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simplifying Selector Lists with `:is()`

**HTML File (`is-basic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:is() Example</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="is-basic.css">
</head>
<body>
    <section>
        <h1>Section Heading</h1>
        <p>Content</p>
    </section>
    <article>
        <h2>Article Heading</h2>
        <p>Content</p>
    </article>
    <aside>
        <h3>Aside Heading</h3>
        <p>Content</p>
    </aside>
</body>
</html>
```

**CSS File (`is-basic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

/* Without :is() — repetitive */
section h1, section h2, section h3,
article h1, article h2, article h3,
aside h1, aside h2, aside h3 {
    color: #3498db;
}

/* With :is() — compact */
:is(section, article, aside) :is(h1, h2, h3) {
    color: #006064;
    border-bottom: 2px solid #006064;
    padding-bottom: 8px;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `is-basic.html`.
3. Save the CSS code as `is-basic.css` in the same folder.
4. Open `is-basic.html` in a web browser.
5. Observe that all headings inside sectioning elements have the teal colour and border.

**Expected Output:** Headings inside `<section>`, `<article>`, and `<aside>` are styled with teal colour and a bottom border. The `:is()` selector replaces the 9-line selector list with a single compact rule.

**Why This Works:** The `:is(section, article, aside)` part matches any of the three container elements, and the `:is(h1, h2, h3)` part matches any of the three heading levels. The combination matches all nine possible combinations without writing them out.

---

#### Example 2: Forgiving Parsing with `:is()` and `:where()`

**HTML File (`forgiving.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Forgiving Selector List</title>
    <link rel="stylesheet" href="forgiving.css">
</head>
<body>
    <input type="text" placeholder="Text input">
    <input type="email" placeholder="Email input">
    <p>Some text content.</p>
</body>
</html>
```

**CSS File (`forgiving.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

/* :is() forgives invalid selectors — the unsupported pseudo-class is ignored */
:is(:valid, :unsupported-pseudo, :focus) {
    border: 2px solid #27ae60;
    outline: none;
}

/* :where() forgives the same way */
:where(:valid, :unsupported-pseudo, :focus) {
    padding: 8px;
}

p {
    background-color: #e0f7fa;
    padding: 12px;
    border-radius: 6px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `forgiving.html` and CSS as `forgiving.css`.
2. Open in a browser.
3. Observe that the inputs are styled with a green border and padding — the `:unsupported-pseudo` is ignored, but `:valid` and `:focus` still work.

**Expected Output:** Input fields have a green border and padding. The unsupported pseudo-class does not break the rule because `:is()` and `:where()` are forgiving.

**Why This Works:** The specification defines `:is()` and `:where()` as accepting a forgiving selector list. When one selector in the list cannot be parsed, the browser ignores just that selector and applies the rest. This is particularly useful when using newer pseudo-classes that may not be supported in all browsers.

---

### Real-World Cases

- **Design systems:** `:is(h1, h2, h3, h4, h5, h6)` for consistent heading styles.
- **Progressive enhancement:** `:is(:focus, :focus-visible)` for focus styles that work in older browsers.
- **Form styling:** `:is(input, select, textarea)` for shared form control styles.
- **Navigation styling:** `:is(nav, aside) :is(a, button)` for shared interactive element styles.

---

## 2. Specificity Modification: `:is()` vs. `:where()`

### Definitions

**Core Definition:** `:is()` and `:where()` differ in a single critical way: `:is()` adopts the specificity of the most specific selector in its argument list, while `:where()` always has zero specificity.

**Technical Definition:** The difference between `:is()` and `:where()` is that `:is()` counts towards the specificity of the overall selector (it takes the specificity of its most specific argument), whereas `:where()` has a specificity value of 0. The `:where()` pseudo-class is a specificity-adjustment pseudo-class that behaves exactly the same as `:is()` except it always has zero specificity. This makes `:where()` ideal for base styles, resets, and utility classes where you never want the styles to accidentally win over more specific rules. `:is()`, by contrast, is ideal when you want the specificity to be determined by the most specific selector in the group — which can be used intentionally to "boost" specificity.

**Beginner-Friendly Explanation:** Think of `:is()` as a selector that inherits the "weight" of its heaviest member. If you write `:is(h1, #id)`, the whole selector has the specificity of an ID selector, even when it matches an `h1`. `:where()` is like a weightless selector — it has zero specificity no matter what you put inside it. Use `:is()` when you want the group to compete normally in the cascade. Use `:where()` when you want the styles to be easily overridden — for example, in a reset or base stylesheet.

---

### Purposes

- To choose between inheriting the highest specificity in a group (`:is()`) or dropping specificity to zero (`:where()`).
- To use `:where()` for base styles, resets, and utility classes that should never win specificity battles.
- To use `:is()` for grouping selectors while maintaining their natural specificity weight.
- To intentionally "boost" specificity by including a high-specificity selector in `:is()`.
- To work effectively with cascade layers, where `:where()` reduces specificity conflicts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* :is() adopts the highest specificity in its argument list */
:is(<selector-list>) {
    /* specificity = highest of arguments */
}

/* :where() always has zero specificity */
:where(<selector-list>) {
    /* specificity = 0,0,0 */
}
```

#### Component Breakdown

| Function | Specificity Behaviour | Best For |
|---|---|---|
| `:is()` | Highest of its arguments | Grouping selectors that should compete normally |
| `:where()` | Always 0 | Base styles, resets, low-specificity utilities |

#### Specificity Examples

| Selector | Specificity |
|---|---|
| `:is(h1, h2, h3)` | 0,0,1 |
| `:is(h1, h2, h3, #id)` | 1,0,0 |
| `:where(h1, h2, h3)` | 0,0,0 |
| `:where(h1, h2, h3, #id)` | 0,0,0 |
| `:is(.class, :where(#id))` | 0,1,0 |

#### Syntax Rules

1. `:is()` takes the specificity of the most specific selector in its argument list.
2. `:where()` always has zero specificity, regardless of its arguments.
3. Both are forgiving selector lists — invalid selectors are ignored.
4. `:where()` is particularly useful in cascade layers for base styles that should not conflict with component styles.
5. You can nest `:where()` inside `:is()` to selectively drop specificity for part of the selector.
6. The specificity of `:not()` and `:has()` is also determined by the most specific selector in their arguments, following the same pattern as `:is()`.

#### Constraints and Limitations

- **Specificity is not always intuitive** — the highest specificity in an `:is()` list applies to the entire selector, even when matching lower-specificity elements.
- **`:where()` can be "too weak"** — styles in `:where()` are easily overridden, which may be undesirable for critical component styles.
- **Browser support** — both are Baseline widely available since January 2021.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `:is()` vs. `:where()` Specificity Comparison

**HTML File (`specificity.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:is() vs :where()</title>
    <link rel="stylesheet" href="specificity.css">
</head>
<body>
    <div class="card">
        <h1 class="title">Heading 1</h1>
        <h2 class="title">Heading 2</h2>
        <p class="title">Paragraph</p>
    </div>
</body>
</html>
```

**CSS File (`specificity.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card {
    background: white;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

/* :where() has zero specificity — easily overridden */
:where(h1, h2, h3, p) {
    color: #999;
    font-weight: normal;
}

/* :is() takes the specificity of the most specific selector (h1 = 0,0,1) */
:is(h1, h2, h3) {
    color: #006064;
    font-weight: bold;
    border-bottom: 2px solid #006064;
    padding-bottom: 8px;
}

/* A class selector (0,1,0) beats :where() (0,0,0) and :is(h1) (0,0,1) */
.title {
    color: #e74c3c;
    font-size: 1.5rem;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `specificity.html` and CSS as `specificity.css`.
2. Open in a browser.
3. Observe:
   - The paragraph (`.title`) is red because `.title` (0,1,0) beats `:where(p)` (0,0,0).
   - The headings are also red because `.title` (0,1,0) beats `:is(h1, h2, h3)` (0,0,1).
   - The `:where()` rule is completely overridden by `.title`.

**Expected Output:** All three elements are red and bold (from `.title`). The `:where()` rule has no effect because its specificity is zero. The `:is()` rule would have styled the headings if `.title` had not overridden them.

**Why This Works:** The `.title` class selector has specificity (0,1,0). The `:where()` selector has specificity (0,0,0), so `.title` always wins. The `:is(h1, h2, h3)` selector has specificity (0,0,1) — the highest in its list is `h1` (0,0,1) — and `.title` (0,1,0) still beats it. This demonstrates the specificity difference clearly.

---

#### Example 2: Using `:where()` for a Reset

**HTML File (`where-reset.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:where() Reset</title>
    <link rel="stylesheet" href="where-reset.css">
</head>
<body>
    <h1>Heading</h1>
    <p>Paragraph text.</p>
    <ul>
        <li>List item 1</li>
        <li>List item 2</li>
    </ul>
</body>
</html>
```

**CSS File (`where-reset.css`):**

```css
/* :where() reset — zero specificity, easily overridden */
:where(h1, h2, h3, p, ul, ol) {
    margin-block: 0;
    padding: 0;
}

:where(ul, ol) {
    list-style: none;
}

/* Component styles override the reset without specificity issues */
body {
    font-family: system-ui, sans-serif;
    padding: 40px;
    background-color: #f5f5f5;
}

h1 {
    color: #006064;
    margin-bottom: 16px;
}

p {
    color: #333;
    margin-bottom: 12px;
    line-height: 1.6;
}

ul {
    padding-left: 20px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `where-reset.html` and CSS as `where-reset.css`.
2. Open in a browser.
3. Observe that the reset styles are applied without any specificity conflicts — the component styles override them freely.

**Expected Output:** A cleanly styled page where the `:where()` reset removes default margins and list styles, and the component styles apply their own margins and colours without needing `!important` or higher specificity.

**Why This Works:** The `:where()` selector has zero specificity, so any subsequent rule (element, class, or ID) automatically overrides it. This makes it ideal for resets and base styles — you can write them first and forget about them.

---

### Real-World Cases

- **CSS resets:** `:where(*)` for universal margin/padding removal with zero specificity.
- **Design systems:** `:where()` for base component styles that consumers can easily override.
- **Utility classes:** `:where(.sr-only)` for screen-reader-only utilities that never conflict.
- **Cascade layers:** Combining `:where()` with `@layer` for maximum control over specificity.

---

## 3. The Relational Parent Selector: `:has()`

### Definitions

**Core Definition:** The `:has()` pseudo-class is a relational pseudo-class that selects an element if any of the relative selectors passed as arguments match at least one element when anchored against that element. It is often called the "parent selector" because it enables selecting a parent based on its children.

**Technical Definition:** The `:has()` CSS pseudo-class represents an element if any of the relative selectors that are passed as an argument match at least one element when anchored against this element. This pseudo-class presents a way of selecting a parent element or a previous sibling element. The `:has()` pseudo-class takes a relative selector list as an argument. Its specificity is determined by the most specific selector in its argument, following the same pattern as `:is()` and `:not()`. The `:has()` pseudo-class cannot be nested within another `:has()`, and pseudo-elements are not valid selectors inside it.

**Beginner-Friendly Explanation:** CSS has always worked "downward" — you select a parent, then its children. `:has()` flips that: you select a parent based on what its children are. For example, `section:has(.featured)` selects any `<section>` that contains an element with the class `featured`. You can also use it to style a heading differently when it is immediately followed by a paragraph: `h1:has(+ p)`. This unlocks a whole range of styling possibilities that previously required JavaScript.

---

### Purposes

- To select a parent element based on its descendants.
- To style elements based on their previous siblings.
- To create complex structural state queries without JavaScript.
- To enable conditional styling based on form input states (e.g., `:has(input:checked)`).
- To reduce the need for JavaScript-based class toggling.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
:has(<relative-selector-list>) {
    /* ... */
}
```

#### Component Breakdown

| Selector | Description | Example |
|---|---|---|
| `parent:has(child)` | Selects any parent containing the child. | `section:has(.featured)` |
| `parent:has(> child)` | Selects a direct parent containing the child. | `section:has(> .featured)` |
| `element:has(+ sibling)` | Selects an element immediately followed by the sibling. | `h1:has(+ p)` |
| `element:has(~ sibling)` | Selects an element followed by the sibling anywhere after. | `h1:has(~ p)` |
| `form:has(:checked)` | Selects a form containing a checked input. | `form:has(input:checked)` |

#### Syntax Rules

1. `:has()` accepts a relative selector list as its argument.
2. The relative selector is anchored against the element being matched.
3. `:has()` cannot be nested inside another `:has()`.
4. Pseudo-elements are not valid inside `:has()` and cannot be anchored by it.
5. The specificity of `:has()` is determined by the most specific selector in its argument.
6. `:has()` is Baseline newly available since December 2023.
7. If a browser does not support `:has()`, the entire selector block is invalidated unless it is wrapped in a forgiving selector list like `:is()` or `:where()`.

#### Constraints and Limitations

- **No nesting** — `:has()` cannot be nested inside another `:has()`.
- **No pseudo-elements** — pseudo-elements are not valid selectors inside `:has()`.
- **Performance considerations** — `:has()` can be expensive to evaluate because the browser must check descendants for every potential match.
- **Browser support** — Baseline since December 2023; older browsers do not support it.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Selecting a Parent Based on a Child

**HTML File (`has-parent.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:has() Parent Selection</title>
    <link rel="stylesheet" href="has-parent.css">
</head>
<body>
    <section>
        <article class="featured">Featured content</article>
        <article>Regular content</article>
    </section>
    <section>
        <article>Regular content</article>
    </section>
</body>
</html>
```

**CSS File (`has-parent.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

section {
    background: white;
    padding: 20px;
    border-radius: 12px;
    margin-bottom: 20px;
    border: 2px solid transparent;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

/* Select any section that contains a .featured element */
section:has(.featured) {
    border-color: #3498db;
    background: #f0f7ff;
}

article {
    padding: 12px;
    border-radius: 8px;
    margin-bottom: 8px;
}

.featured {
    background: #3498db;
    color: white;
    font-weight: bold;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `has-parent.html` and CSS as `has-parent.css`.
2. Open in a browser that supports `:has()` (all modern browsers).
3. Observe that the first section has a blue border and light blue background, while the second section does not.

**Expected Output:** The first section (containing the featured article) has a blue border and light blue background. The second section (without a featured article) has the default styling. This demonstrates selecting a parent based on its content.

**Why This Works:** The `section:has(.featured)` selector matches any `<section>` element that contains an element with the class `featured`. The relative selector `.featured` is anchored against the section, so only sections with a descendant `.featured` are matched.

---

#### Example 2: Styling Based on Sibling Relationships

**HTML File (`has-sibling.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:has() Sibling Selection</title>
    <link rel="stylesheet" href="has-sibling.css">
</head>
<body>
    <article>
        <h1>Morning Times</h1>
        <p>Content follows the heading.</p>
    </article>
    <article>
        <h1>Afternoon Times</h1>
    </article>
</body>
</html>
```

**CSS File (`has-sibling.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

article {
    background: white;
    padding: 20px;
    border-radius: 12px;
    margin-bottom: 20px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

h1 {
    margin-top: 0;
}

/* Style an h1 differently when immediately followed by a p */
h1:has(+ p) {
    margin-bottom: 0;
    color: #006064;
    border-bottom: 2px solid #006064;
    padding-bottom: 8px;
}

p {
    margin-top: 12px;
    color: #555;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `has-sibling.html` and CSS as `has-sibling.css`.
2. Open in a browser.
3. Observe that the first `h1` (followed by a `p`) has a teal colour and border, while the second `h1` (no following `p`) does not.

**Expected Output:** The first heading is teal with a bottom border because it is immediately followed by a paragraph. The second heading retains its default styling. This demonstrates using `:has()` with the adjacent sibling combinator to style an element based on what comes after it.

**Why This Works:** The `h1:has(+ p)` selector matches any `<h1>` element that is immediately followed by a `<p>` element. The relative selector `+ p` is anchored against the `h1`, checking whether the next sibling is a paragraph.

---

### Real-World Cases

- **Form validation styling:** `form:has(:user-invalid)` for styling forms with invalid inputs.
- **E-commerce cards:** `.card:has(img)` for cards with images vs. cards without.
- **Navigation states:** `html:has(#dark-mode:checked)` for a CSS-only dark mode toggle.
- **Content-aware layouts:** `figure:has(> figcaption)` for figures with captions.

---

## 4. Advanced Negation: The Modernized `:not()`

### Definitions

**Core Definition:** The `:not()` pseudo-class represents elements that do not match a list of selectors. In Selectors Level 4, it has been modernized to accept a complex selector list instead of only simple selectors.

**Technical Definition:** The `:not()` CSS pseudo-class represents elements that do not match a list of selectors. Since it prevents specific items from being selected, it is known as the negation pseudo-class. In Selectors Level 3, `:not()` only allowed a single simple selector. Selectors Level 4 allows `:not()` to accept a list of selectors, including complex selectors. The specificity of the `:not()` pseudo-class is replaced by the specificity of the most specific selector in its comma-separated argument of selectors, providing the same specificity as if it had been written `:not(:is(argument))`.

**Beginner-Friendly Explanation:** `:not()` lets you say "style everything except this." In the old days, you could only exclude one simple selector at a time — `:not(.foo)` or `:not(#bar)`. Now you can exclude whole groups and even complex selectors: `:not(.foo, .bar, #baz)` or `:not(table a)`. The specificity of the `:not()` part is determined by the most specific selector inside it, just like `:is()`.

---

### Purposes

- To exclude multiple selectors in a single rule.
- To exclude complex selectors, not just simple ones.
- To write cleaner, more precise CSS without multiple `:not()` chains.
- To use `:not()` with modern selectors like `:has()` for advanced exclusion logic.
- To understand how `:not()` specificity works in Selectors Level 4.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
:not(<complex-selector-list>) {
    /* ... */
}
```

#### Component Breakdown

| Version | Argument | Example |
|---|---|---|
| Selectors Level 3 | Single simple selector | `:not(.foo)` |
| Selectors Level 4 | Complex selector list | `:not(.foo, .bar, #baz)` |

#### Syntax Rules

1. In Selectors Level 4, `:not()` accepts a complex selector list.
2. The list must not contain another negation selector or a pseudo-element, but any other simple, compound, and complex selectors are allowed.
3. The specificity of `:not()` is replaced by the specificity of the most specific selector in its argument list.
4. `:not(.foo)` matches anything that is not `.foo`, including `<html>` and `<body>`.
5. `:not(.foo, .bar)` is equivalent to `:not(.foo):not(.bar)`.
6. The `:not()` pseudo-class is Baseline widely available.

#### Constraints and Limitations

- **No nested negation** — `:not()` cannot contain another `:not()`.
- **No pseudo-elements** — pseudo-elements are not valid inside `:not()`.
- **Specificity can increase** — `:not(#foo)` has the specificity of an ID selector, which can be surprising.
- **Descendant combinator traps** — `body :not(table) a` still matches links inside a table, because `<tr>`, `<td>`, etc. can match `:not(table)`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Complex Negation with `:not()`

**HTML File (`not-basic.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Modern :not()</title>
    <link rel="stylesheet" href="not-basic.css">
</head>
<body>
    <nav>
        <a href="#" class="nav-link">Home</a>
        <a href="#" class="nav-link active">About</a>
        <a href="#" class="nav-link">Services</a>
        <a href="#" class="nav-link external">Contact</a>
    </nav>
</body>
</html>
```

**CSS File (`not-basic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

nav {
    display: flex;
    gap: 16px;
    background: white;
    padding: 16px 24px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.nav-link {
    color: #333;
    text-decoration: none;
    font-weight: bold;
    padding: 8px 16px;
    border-radius: 8px;
    transition: background-color 200ms;
}

/* Style all nav links EXCEPT .active and .external */
.nav-link:not(.active, .external) {
    background-color: #f0f0f0;
}

.nav-link.active {
    background-color: #3498db;
    color: white;
}

.nav-link.external {
    border: 2px solid #e74c3c;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `not-basic.html` and CSS as `not-basic.css`.
2. Open in a browser.
3. Observe that "Home" and "Services" have a grey background, while "About" is blue and "Contact" has a red border.

**Expected Output:** The `.active` and `.external` links are excluded from the grey background styling. The `:not(.active, .external)` selector matches all `.nav-link` elements except those two classes.

**Why This Works:** In Selectors Level 4, `:not()` accepts a comma-separated list of selectors. The rule `.nav-link:not(.active, .external)` matches any element with class `nav-link` that does not also have class `active` or `external`.

---

### Real-World Cases

- **Form styling:** `input:not([type="submit"], [type="reset"])` for styling form inputs while excluding buttons.
- **Navigation:** `a:not([href^="mailto:"])` for styling links that are not email links.
- **Layout:** `:not(:last-child)` for applying margins to all but the last child.
- **Component variants:** `.button:not(.button--primary, .button--danger)` for default button styles.

---

## 5. State and Media Queries as Selectors

### Definitions

**Core Definition:** State-based pseudo-classes like `:focus-visible` and `:user-valid` allow authors to style elements based on nuanced interaction states, while container query evaluation states allow styling based on scroll and layout states.

**Technical Definition:** The `:focus-visible` pseudo-class matches a focused element only when the user agent determines, through heuristics, that the focus should be visibly indicated — typically when the user is navigating via keyboard rather than pointing device. The `:user-valid` pseudo-class represents any validated form element whose value validates correctly based on its validation constraints, but unlike `:valid`, it only matches after the user has interacted with the control. Container query evaluation states, such as scroll-state queries, allow styling descendants based on whether a container is stuck, snapped, or partially scrolled, using `container-type: sticky` and `@container scroll-state(...)` queries.

**Beginner-Friendly Explanation:** Modern CSS lets you style elements based on how the user is actually interacting with them. `:focus-visible` shows focus rings when someone tabs with a keyboard, but not when they click with a mouse. `:user-valid` shows a green checkmark only after the user has typed something valid — not immediately on page load. And with scroll-state container queries, you can style a sticky header differently when it is actually stuck to the top, or style a snapped carousel item differently when it is snapped into view.

---

### Purposes

- To provide keyboard-only focus indicators with `:focus-visible`.
- To validate forms only after user interaction with `:user-valid`.
- To style sticky elements based on whether they are stuck.
- To style scroll-snapped elements based on whether they are snapped.
- To create more nuanced, context-aware styling without JavaScript.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* :focus-visible */
:focus-visible {
    outline: 3px solid #3498db;
}

/* :user-valid */
input:user-valid {
    border-color: #27ae60;
}

/* Container scroll-state queries */
.container {
    container-type: sticky;
    container-name: my-menu;
    position: sticky;
    top: 0;
}
@container my-menu scroll-state(stuck: top) {
    .header { background: #006064; }
}

/* Snap state */
.carousel {
    container-type: scroll-state;
}
@container scroll-state(snapped: inline) {
    .slide { opacity: 1; }
}
```

#### Component Breakdown

| Selector / Query | Description | Baseline |
|---|---|---|
| `:focus-visible` | Focused element that should visibly indicate focus. | Widely available |
| `:user-valid` | Valid form element after user interaction. | Newly available (2023) |
| `:user-invalid` | Invalid form element after user interaction. | Newly available (2023) |
| `scroll-state(stuck: top)` | Container is stuck to the top. | Emerging |
| `scroll-state(snapped: inline)` | Container is snapped along the inline axis. | Emerging |

#### Syntax Rules

1. `:focus-visible` matches a focused element only when the UA determines focus should be visible.
2. `:user-valid` matches only after the user has interacted with the form control (changed value, attempted submit, etc.).
3. `:user-invalid` is the inverse — matches after interaction and validation failure.
4. Container scroll-state queries require `container-type: sticky` or `container-type: scroll-state`.
5. The `scroll-state()` function accepts conditions like `stuck`, `snapped`, and `overflowing`.
6. Browser support for scroll-state queries is emerging; check compatibility.

#### Constraints and Limitations

- **`:focus-visible` heuristics** — the exact conditions for matching vary by browser.
- **`:user-valid` interaction requirement** — the pseudo-class does not match until the user interacts, which may delay validation feedback.
- **Scroll-state queries** — still emerging; not yet Baseline.
- **Performance** — scroll-state queries introduce additional rendering update complexity.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Keyboard-Only Focus with `:focus-visible`

**HTML File (`focus-visible.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:focus-visible</title>
    <link rel="stylesheet" href="focus-visible.css">
</head>
<body>
    <button class="btn">Click or Tab to me</button>
    <a href="#" class="link">A link</a>
</body>
</html>
```

**CSS File (`focus-visible.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    gap: 20px;
    align-items: center;
}

.btn {
    padding: 12px 24px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
}

.link {
    color: #006064;
    font-weight: bold;
}

/* Remove default focus outline */
:focus {
    outline: none;
}

/* Show focus ring only for keyboard navigation */
:focus-visible {
    outline: 3px solid #e74c3c;
    outline-offset: 2px;
    border-radius: 4px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `focus-visible.html` and CSS as `focus-visible.css`.
2. Open in a browser.
3. Click the button with a mouse — no focus ring appears.
4. Press Tab to focus the button — a red focus ring appears.

**Expected Output:** The focus ring appears only when navigating via keyboard, not when clicking with the mouse. This provides a better experience for both mouse and keyboard users.

**Why This Works:** The `:focus-visible` pseudo-class matches only when the browser determines that visible focus indication is helpful — typically when the user is navigating via keyboard. The default `:focus` outline is removed, and the `:focus-visible` outline provides a clear indicator for keyboard users.

---

#### Example 2: Form Validation with `:user-valid`

**HTML File (`user-valid.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>:user-valid</title>
    <link rel="stylesheet" href="user-valid.css">
</head>
<body>
    <form>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required value="user@example.com">
        <span class="status"></span>
    </form>
</body>
</html>
```

**CSS File (`user-valid.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

form {
    background: white;
    padding: 24px;
    border-radius: 12px;
    max-width: 400px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

label {
    display: block;
    font-weight: bold;
    margin-bottom: 8px;
}

input {
    width: 100%;
    padding: 10px 12px;
    border: 2px solid #ddd;
    border-radius: 8px;
    font-size: 1rem;
    box-sizing: border-box;
    transition: border-color 200ms;
}

/* Green border and checkmark only after user interaction */
input:user-valid {
    border-color: #27ae60;
    background-color: #f0fff4;
}

input:user-valid + .status::before {
    content: "✓ Valid email";
    color: #27ae60;
    font-weight: bold;
    display: block;
    margin-top: 8px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `user-valid.html` and CSS as `user-valid.css`.
2. Open in a browser.
3. Initially, the input has a grey border (the value is valid but the user has not interacted).
4. Type in the input and then click elsewhere — the border turns green and the checkmark appears.

**Expected Output:** The green border and checkmark appear only after the user has interacted with the field and the value is valid. Before interaction, the field looks neutral, even if the prefilled value is valid.

**Why This Works:** The `:user-valid` pseudo-class matches only after the user has interacted with the form control. This prevents the "false positive" of showing a valid state before the user has done anything, which is a common UX improvement over `:valid`.

---

### Real-World Cases

- **Accessibility:** `:focus-visible` for keyboard-only focus indicators.
- **Form UX:** `:user-valid` and `:user-invalid` for post-interaction validation feedback.
- **Sticky headers:** `scroll-state(stuck: top)` for styling headers when they are stuck.
- **Carousels:** `scroll-state(snapped: inline)` for styling the active slide.

---

## References

- MDN Web Docs — `:is()` - https://developer.mozilla.org/en-US/docs/Web/CSS/:is
- MDN Web Docs — `:where()` - https://developer.mozilla.org/en-US/docs/Web/CSS/:where
- MDN Web Docs — `:has()` - https://developer.mozilla.org/en-US/docs/Web/CSS/:has
- MDN Web Docs — `:not()` - https://developer.mozilla.org/en-US/docs/Web/CSS/:not
- MDN Web Docs — `:focus-visible` - https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible
- MDN Web Docs — `:user-valid` - https://developer.mozilla.org/en-US/docs/Web/CSS/:user-valid
- W3C — Selectors Level 4 - https://www.w3.org/TR/selectors-4/
- W3C — CSS Conditional Rules Level 5 (Scroll State Container Queries) - https://drafts.csswg.org/css-conditional-5/
- Chrome for Developers — `:has()` Case Study - https://developer.chrome.com/blog/css-ui-ecommerce-has
- CSS-Tricks — Quick Reminder That `:is()` and `:where()` Are Basically the Same With One Key Difference - https://css-tricks.com/quick-reminder-that-is-and-where-are-basically-the-same-with-one-key-difference/
- Can I Use — `:is()` and `:where()` - https://caniuse.com/css-is-where
- Can I Use — `:has()` - https://caniuse.com/css-has
- Can I Use — `:focus-visible` - https://caniuse.com/css-focus-visible
- Can I Use — `:user-valid` - https://caniuse.com/mdn-css_selectors_user-valid
- Web.dev — New CSS functional pseudo-class selectors `:is()` and `:where()` - https://web.dev/articles/css-is-and-where
- Web.dev — 5 CSS snippets every front-end developer should know in 2024 - https://web.dev/articles/5-css-snippets-every-front-end-developer-should-know-in-2024