# CSS Cascade

The cascade is the algorithm at the heart of CSS. It determines which style declarations "win" when multiple rules target the same element and property. Understanding the cascade is essential for writing predictable, maintainable stylesheets.

---

## 1. Meaning of Cascading

**Cascading** is the process by which the browser combines style declarations from multiple sources and resolves conflicts between them.

> "The cascade is an algorithm that defines how user agents combine property values originating from different sources. The cascade defines the origin and layer that takes precedence when declarations in more than one origin, cascade layer, or `@scope` block set a value for a property on an element."

The name **Cascading Style Sheets** reflects this core mechanism: styles "cascade" down through a series of priority levels, like water flowing through a series of pools.

### The Cascade as a Sorting Algorithm

When multiple declarations apply to the same element and property, the cascade sorts them according to a fixed set of criteria, in **descending order of priority**:

1. **Origin and importance** — where the declaration came from, and whether it is `!important`
2. **Cascade layers** — the layer the declaration belongs to (if any)
3. **Specificity** — the weight of the selector
4. **Source order** — the order in which declarations appear

The declaration that survives all filtering steps is the **winning declaration**.

> "The cascade lies at the core of CSS, as emphasized by the name: Cascading Style Sheets. When a selector matches an element, the property value from the origin with the highest precedence gets applied, even if the selector from a lower precedence origin or layer has greater specificity."

### Why It Matters

Without the cascade, every property would need exactly one rule. The cascade enables:

- **Layered authoring** — multiple stylesheets can coexist
- **User control** — users can override author styles for accessibility
- **Browser defaults** — the user-agent stylesheet provides a baseline
- **Theming** — base styles, component styles, and utilities can coexist predictably

---

## 2. Source Order

**Source order** is the final tiebreaker in the cascade. When two declarations have the same origin, importance, layer, and specificity, the one that appears **later** in the source wins.

> "Finally, sort by order specified: if two rules have the same weight, origin and specificity, the latter specified wins."

### How Source Order Works

| Source | Later Wins |
|---|---|
| **Within a stylesheet** | A rule lower in the file overrides one above it |
| **Between stylesheets** | A stylesheet linked later in the HTML overrides an earlier one |
| **Between internal and external** | A `<style>` after a `<link>` overrides the linked stylesheet |

### Example — Within a Stylesheet

```css
p {
  color: red;
}

p {
  color: blue;   /* wins — same specificity, later in source */
}
```

### Example — Between Stylesheets

```html
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="theme.css">   <!-- later in HTML → wins -->
```

### Example — `<style>` After `<link>`

```html
<link rel="stylesheet" href="external.css">
<style>
  p { color: green; }   /* wins — later than the <link> */
</style>
```

### Important Nuance — Imported Stylesheets

Imported stylesheets are ordered **as if their rules were substituted in place of the `@import` rule**:

```css
@import url("reset.css");    /* rules come first */
@import url("base.css");     /* then these */

p { color: navy; }           /* then these — win over imports */
```

> "Rules in imported style sheets are considered to be before any rules in the style sheet itself."

### Source Order in Modern `@layer`

Once cascade layer order is established, **source order within a layer** is still the tiebreaker for declarations in that layer.

---

## 3. Specificity

**Specificity** is a weight assigned to a selector based on its components. When two declarations have the same origin, importance, and layer, the one with **higher specificity** wins.

> "Sort by specificity of selector: more specific selectors will override more general ones. Pseudo-elements and pseudo-classes are counted as normal elements and classes, respectively."

### Specificity Calculation

Specificity is expressed as a three-part value **(A, B, C)**:

| Component | Counts | Examples |
|---|---|---|
| **A** | ID selectors | `#header`, `#main` |
| **B** | Class selectors, attribute selectors, pseudo-classes | `.button`, `[type="text"]`, `:hover` |
| **C** | Type selectors, pseudo-elements | `p`, `h1`, `::before` |

### Examples

| Selector | Specificity (A, B, C) |
|---|---|
| `*` | (0, 0, 0) |
| `p` | (0, 0, 1) |
| `p.intro` | (0, 1, 1) |
| `.intro` | (0, 1, 0) |
| `#header` | (1, 0, 0) |
| `#header .nav a` | (1, 1, 1) |
| `a:hover` | (0, 1, 1) |
| `p::first-line` | (0, 0, 2) |

### Comparison Rules

Compare **A** first; higher A wins. If A is equal, compare **B**; if B is equal, compare **C**. If all are equal, fall back to source order.

```
(1, 0, 0)  >  (0, 5, 0)   ← one ID beats five classes
(0, 1, 0)  >  (0, 0, 10)  ← one class beats ten elements
```

### The `:is()`, `:where()`, and `:not()` Exceptions

| Function | Specificity |
|---|---|
| `:is()`, `:not()`, `:has()` | Specificity of the **most specific argument** |
| `:where()` | **Zero** — always (0, 0, 0) |

```css
:where(#header) p { }    /* specificity (0, 0, 1) — :where() adds nothing */
:is(#header) p { }       /* specificity (1, 0, 1) — :is() takes #header */
```

### Inline Styles

Inline styles have **higher specificity than any selector-based rule**:

| Source | Specificity |
|---|---|
| Inline `style` attribute | (1, 0, 0, 0) — conceptually |
| ID selector | (0, 1, 0, 0) |

Only `!important` declarations can override inline styles.

### Specificity and the Cascade

> "The cascade algorithm is applied before the specificity algorithm."

This means **origin and layer come first**. A rule with lower specificity can win if it comes from a higher-precedence origin or layer.

---

## 4. Importance

**Importance** is controlled by the `!important` flag. Important declarations change the normal cascade order.

### The `!important` Flag

```css
selector {
  property: value !important;
}
```

> "The `!important` flag changes the rules for selecting declarations in the cascade. Declarations not marked as important are called normal."

### Syntax Rules

| Rule | Example |
|---|---|
| Placed after the value, before `;` | `color: red !important;` |
| Must be the **last token** in the declaration | `color: red !important;` ✓ |
| Whitespace between `!` and `important` is allowed | `color: red ! important;` (not preferred) |
| No content after the flag (except whitespace/comments) | `color: red !important /* comment */;` ✓ |

> "The important flag must be the last token in the declaration. In other words, there can be whitespace and comments between the flag and the declaration's ending semicolon, but nothing else."

### What `!important` Does

`!important` **inverts the normal origin precedence order**.

| Priority (Normal) | Priority (Important) |
|---|---|
| 1. User-agent | 5. User-agent important |
| 2. User normal | 4. User important |
| 3. Author normal | 3. Author important |
| 4. Author important | 2. Author normal |
| 5. User important | 1. User normal |

**Normal order (lowest to highest):**

1. User-agent normal
2. User normal
3. Author normal
4. Author important
5. User important

**Important order (lowest to highest):**

1. User normal
2. Author normal
3. Author important
4. User important
5. User-agent important

> "When declarations are marked as important, the priority order is reversed."

### Why the Reversal Matters

The reversal exists for **accessibility and user control**:

> "Reversing the order of precedence for important declarations ensures that users with special needs (e.g., personalized color schemes or large fonts) can override author styles by marking certain declarations as important in their user stylesheet."

A user with low vision can write:

```css
/* User stylesheet */
* {
  font-size: 24px !important;
  background: black !important;
  color: white !important;
}
```

This overrides **any** author `!important` declarations, ensuring accessibility.

### `!important` and Specificity

> "Although `!important` is not part of determining specificity, it is related. Important declarations override all other declarations from the same origin and cascade layer."

**Within the same origin and layer**, `!important` beats normal declarations **regardless of specificity**.

```css
p { color: red !important; }      /* specificity (0,0,1) */
#main p { color: blue; }           /* specificity (1,0,1) */
/* Result: red — !important wins */
```

### `!important` and Inline Styles

| Declaration | Wins Over |
|---|---|
| Inline **normal** | All normal declarations, any origin |
| Inline **important** | All other important **author** declarations |
| User/UA **important** | Inline important declarations |
| **Transitions** | All `!important` declarations |

> "Inline normal styles take precedence over all normal declarations, regardless of origin. Inline important styles take precedence over all other important author styles, regardless of layer, but important styles from user or user-agent stylesheets and transitions override them."

### What Beats `!important`?

> "Is there anything higher than important declarations? Yes, transitions."

**Transitions** have the highest priority in the cascade. A property in the middle of a transition does not match any `!important` declaration.

```css
a {
  color: red !important;
  background-color: yellow;
  transition: all 2s linear;
}

a:hover {
  color: blue !important;
  background-color: orange !important;
}
```

The transition from red to blue and yellow to orange occurs **despite** the `!important` flags.

### `!important` in `@keyframes`

> "`!important` is invalid within `@keyframes` animation declarations."

Animations have their own priority level and cannot use `!important`.

### Best Practices

| Practice | Reason |
|---|---|
| **Avoid `!important`** | Breaks the natural cascade; hard to override |
| **Use for utilities** | `.hidden { display: none !important; }` is a legitimate pattern |
| **Use for accessibility overrides** | User stylesheets for special needs |
| **Never use for convenience** | If you need it, your specificity is likely wrong |
| **Document every use** | Future maintainers need to know why |

---

## 5. Inheritance

**Inheritance** controls what happens when **no value is specified** for a property on an element.

> "In CSS, inheritance controls what happens when no value is specified for a property on an element."

### Inherited vs. Non-Inherited Properties

| Type | Default Behavior | Examples |
|---|---|---|
| **Inherited** | Child gets parent's computed value | `color`, `font-family`, `font-size`, `line-height` |
| **Non-inherited** | Child gets the property's initial value | `border`, `margin`, `padding`, `background` |

### Inherited Properties — Example

```css
p { color: green; }
```

```html
<p>This paragraph has <em>emphasized text</em> in it.</p>
```

The `<em>` element inherits the `color: green` value from its parent `<p>`, because `color` is an inherited property. The word "emphasized text" appears green.

> "The words 'emphasized text' will be displayed in green because the `em` element inherits the value of the `color` property from the `p` element."

### Non-Inherited Properties — Example

```css
p { border: medium solid; }
```

```html
<p>This paragraph has <em>emphasized text</em> in it.</p>
```

The `<em>` element does **not** inherit the border, because `border` is a non-inherited property. The word "emphasized text" has no border of its own.

> "The words 'emphasized text' will not have another border (because the initial value of `border-style` is `none`)."

### The `inherit` Keyword

The `inherit` keyword explicitly forces inheritance, even for non-inherited properties:

```css
p { border: medium solid; }
em { border: inherit; }
```

Now the `<em>` element **does** have a border, because `border: inherit` explicitly pulls the parent's value.

> "The `inherit` keyword allows authors to explicitly specify inheritance. It works on both inherited and non-inherited properties."

### The `initial`, `unset`, and `revert` Keywords

| Keyword | Behavior |
|---|---|
| `inherit` | Use parent's computed value |
| `initial` | Use the property's initial value |
| `unset` | `inherit` if inherited, `initial` if not |
| `revert` | Revert to user-agent or user stylesheet value |
| `revert-layer` | Revert to previous cascade layer value |

```css
p {
  all: revert;        /* reset every property to UA default */
  font-size: 200%;    /* then apply these */
  font-weight: bold;
}
```

> "You can control inheritance for all properties at once using the `all` shorthand property, which applies its value to all properties."

### Inheritance and the Cascade

Inheritance is **not** the same as the cascade:

| Aspect | Cascade | Inheritance |
|---|---|---|
| Purpose | Resolve conflicts between declarations | Fill in missing values |
| Scope | Same element, same property | Parent → child |
| Triggered by | Multiple matching declarations | No declaration for a property |
| Precedence | Origin → layer → specificity → order | Only if no declaration applies |

Inheritance happens **after** the cascade — if the cascade produces a value, inheritance is not used for that property.

---

## 6. User-Agent Styles

**User-agent styles** (also called **browser default styles**) are the built-in stylesheets browsers apply to every document.

> "User-agents, or browsers, have basic stylesheets that give default styles to any document. These stylesheets are named user-agent stylesheets."

### Examples of User-Agent Styles

| Element | Default Style |
|---|---|
| `<h1>` | Large bold font, margins |
| `<p>` | Margins above and below |
| `<ul>` | Padding-left, list-style bullets |
| `<a>` | Blue color, underline |
| `<body>` | 8px margin |

### Specificity and Precedence

User-agent styles have the **lowest precedence** in the cascade:

> "All user and author rules have more weight than rules in the UA's default style sheet."

This means any author rule — even a low-specificity one — overrides user-agent styles.

> "Unless the user-agent stylesheet includes an `!important` next to a property, making it 'important', styles declared by author styles, including a reset stylesheet, take precedence over the user-agent styles."

### CSS Resets

Because user-agent styles vary between browsers, developers often use a **CSS reset** or **normalize** stylesheet:

> "To simplify the development process, Web developers may use a CSS reset stylesheet, such as `normalize.css`, which sets common property values to a known state for all browsers before beginning to make alterations to suit their specific needs."

---

## 7. Author Styles

**Author styles** are the styles written by the **web developer** — the CSS you write for a website.

> "The author specifies style sheets for a source document according to the conventions of the document language. For instance, in HTML, style sheets may be included in the document or linked externally."

### Sources of Author Styles

| Source | Example |
|---|---|
| **External stylesheet** | `<link rel="stylesheet" href="styles.css">` |
| **Internal stylesheet** | `<style> ... </style>` |
| **Inline style** | `<p style="color: red;">` |

### Precedence

By default, **author normal styles override user normal styles**:

> "By default, rules in author style sheets have more weight than rules in user style sheets."

This is intentional — the author designs the experience, but the user can override with `!important`.

### Author Important Styles

Author `!important` declarations are **stronger than author normal declarations** but **weaker than user important declarations**:

> "Both author and user style sheets may contain 'important' declarations, and user 'important' rules override author 'important' rules."

---

## 8. User Styles

**User styles** are styles written by the **user** (or reader) of a web page, typically via browser extensions, user stylesheets, or operating system preferences.

> "Users can specify styles using browser preferences, OS preferences, or browser extensions. Their important declarations take precedence over author (or web-developer-written) important declarations."

### Why User Styles Exist

User styles serve **accessibility** and **personalization**:

- Increasing font sizes for low vision
- High-contrast color schemes
- Hiding distracting elements
- Customizing site appearance

### Precedence

| Declaration Type | Priority |
|---|---|
| **User normal** | Lower than author normal |
| **User important** | **Higher than author important** |
| **User-agent important** | **Highest of all** |

> "The user's important declarations take precedence over the author's (or web developer's) important declarations."

### Example — User Stylesheet for Accessibility

```css
/* User stylesheet */
* {
  font-size: 20px !important;
  line-height: 1.8 !important;
  background: #000 !important;
  color: #fff !important;
}
```

This overrides **any** author styles, including author `!important` declarations.

### Tools for User Styles

| Tool | Platform |
|---|---|
| **Stylus** | Chrome, Firefox, Opera |
| **Stylish** | Chrome, Firefox |
| **User CSS** | Safari |
| **Browser DevTools** | All (temporary) |
| **OS high-contrast modes** | Windows, macOS |

---

## 9. `!important`

The `!important` flag is covered in detail in **Section 4 — Importance**. This section summarizes its role in the cascade.

### The Full Cascade Order

**Normal declarations (lowest to highest):**

1. User-agent normal
2. User normal
3. Author normal
4. Author `!important`
5. User `!important`
6. User-agent `!important`

**After importance, sort by:**

- **Layer** (layered vs. unlayered; layer order)
- **Specificity**
- **Source order**

### `!important` and Layers

> "In all three origins (author, user, and user-agent), normal declarations in unlayered styles override layered style declarations, where the last declared layer takes precedence over layers declared before it. Important declarations reverse the order of precedence: important declarations in the first layer take precedence over important declarations in the next layer, and so on. Also, all important declarations have precedence over any important declarations declared outside a layer."

**Key rules:**

| Declaration Type | Layered vs. Unlayered |
|---|---|
| **Normal** | Unlayered > layered (last layer wins among layers) |
| **Important** | Layered > unlayered (first layer wins among layers) |

### `!important` and `@keyframes`

> "`!important` is invalid within `@keyframes` animation declarations."

### `!important` and Transitions

Transitions have **higher priority** than `!important` declarations:

> "All important declarations take precedence over all animations."

---

## 10. Cascade Layers

**Cascade layers** (`@layer`) allow authors to explicitly control the precedence of style groups, independent of specificity.

> "Cascade layers are declared using the `@layer` at-rule, which can also be used to define the precedence order when multiple cascade layers are declared."

### Why Layers Exist

Before layers, controlling precedence meant manipulating specificity or using `!important`. Layers decouple **precedence** from **specificity**:

> "This makes it possible to use simpler CSS selectors, because you don't have to ensure selectors have high enough specificity to override conflicting rules; you just need to ensure it appears in a later layer."

### Declaring Layers

**Three ways to declare a layer:**

**1. Block `@layer` — named layer with rules:**

```css
@layer utilities {
  .padding-sm { padding: 0.5rem; }
  .padding-lg { padding: 0.8rem; }
}
```

**2. Statement `@layer` — layer order declaration:**

```css
@layer theme, layout, utilities;
```

**3. Anonymous `@layer` — unnamed layer:**

```css
@layer {
  /* rules here */
}
```

> "The `@layer` at-rule can create cascade layers in one of three ways."

### Layer Order

Layer order is determined by **first declaration order**:

> "The initial order of layer declaration indicates which layer has priority. As with declarations, if a declaration appears in multiple layers, the last layer in the list wins."

**Example:**

```css
@layer theme, layout, utilities;
```

Priority (lowest to highest): `theme` → `layout` → `utilities`.

A rule in `utilities` wins over a rule in `theme`, **even if the `theme` rule has higher specificity**.

> "The rule in `utilities` will be applied even if its specificity is lower than the rule in `theme`. This is because once layer order is established, specificity and order of appearance are ignored."

### Layer Precedence Summary

**Normal declarations:**

1. Unlayered normal (highest priority)
2. Last layer normal
3. ...
4. First layer normal (lowest priority)

**Important declarations:**

1. First layer important (highest priority)
2. ...
3. Last layer important
4. Unlayered important (lowest priority)

> "Important declarations reverse the priority order: important declarations in the first layer take precedence over important declarations in the next layer, and so on. Also, all important declarations have precedence over any important declarations declared outside a layer."

### Nested Layers

Layers can be nested:

```css
@layer framework {
  @layer base, components;
}

@layer framework.base {
  /* rules */
}
```

Nested layers have their own priority order within the parent layer.

### Use Cases for Layers

| Use Case | Example |
|---|---|
| **Third-party CSS** | Put vendor styles in a low-priority layer |
| **Design system** | `@layer base, components, utilities;` |
| **Tailwind** | Tailwind's utilities in a high-priority layer |
| **Overrides** | Put overrides in a layer that always wins |
| **Resets** | Put resets in the lowest layer |

### Example — Design System with Layers

```css
@layer reset, base, components, utilities, overrides;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  body { margin: 0; }
}

@layer base {
  body { font-family: system-ui, sans-serif; line-height: 1.6; }
  h1, h2, h3 { line-height: 1.2; }
}

@layer components {
  .card { padding: 1rem; border-radius: 8px; }
  .button { padding: 0.5rem 1rem; }
}

@layer utilities {
  .mt-0 { margin-top: 0 !important; }
  .text-center { text-align: center; }
}

@layer overrides {
  .card { border: 1px solid #ddd; }
}
```

### Layer Order and `!important`

Important declarations **reverse** the layer order:

| Layer | Normal Priority | Important Priority |
|---|---|---|
| `reset` | Lowest | Highest |
| `base` | Low | High |
| `components` | Medium | Medium |
| `utilities` | High | Low |
| `overrides` | Highest | Lowest |

This means `!important` in `reset` beats `!important` in `overrides` — a subtle but important behavior.

### Browser Support

> "Baseline Widely available — This feature is well established and works across many devices and browser versions. It's been available across browsers since March 2022."

Cascade layers are supported in all modern browsers (Chrome 99+, Firefox 97+, Safari 15.4+).

---

## Summary Table

| Concept | Definition | Priority Order |
|---|---|---|
| **Cascade** | Algorithm for resolving conflicting declarations | Origin → Layer → Specificity → Order |
| **Source order** | Later declarations win (all else equal) | Last wins |
| **Specificity** | Weight of a selector | Higher wins |
| **Importance** | `!important` flag | Important > normal |
| **Inheritance** | Child gets parent's value when no declaration exists | Applies only when no declaration matches |
| **User-agent styles** | Browser defaults | Lowest normal priority |
| **Author styles** | Developer-written CSS | Overrides user normal |
| **User styles** | User-written CSS | User important > author important |
| **`!important`** | Inverts origin order for important declarations | User-agent important > user important > author important |
| **`@layer`** | Explicit precedence groups | Unlayered normal > last layer; first layer important > unlayered important |

---

## Key Takeaways

1. **Cascading** is the algorithm that combines declarations from different sources and resolves conflicts.
2. **Source order** is the final tiebreaker: when all else is equal, the **last declaration wins**.
3. **Specificity** is a three-part weight (A, B, C) based on selector components; higher specificity wins within the same origin and layer.
4. **`!important`** inverts the normal origin precedence, giving user styles priority over author styles for accessibility.
5. **Inheritance** fills in missing values from parent elements — it applies only when no declaration matches.
6. **User-agent styles** are browser defaults with the lowest normal priority.
7. **Author styles** are developer-written CSS; author normal overrides user normal.
8. **User styles** can override author styles **only with `!important`** — this is a deliberate accessibility feature.
9. **`!important`** should be used sparingly; it breaks the natural cascade and makes debugging harder.
10. **Cascade layers (`@layer`)** decouple precedence from specificity, allowing simpler selectors and predictable style organization.
11. **Layer order is established by first declaration** — later layers win for normal declarations, but the order reverses for important declarations.
12. **Transitions** have the highest priority in the cascade — higher than any `!important` declaration.

---

Would you like me to continue with the next topic — **CSS Selectors in Depth**, **CSS Box Model**, or **CSS Units and Values**? I can format the next section in the same style.