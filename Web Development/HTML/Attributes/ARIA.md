# ARIA Attributes — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Accessible Rich Internet Applications (ARIA) is a set of roles and attributes that define ways to make web content and web applications — especially those developed with JavaScript — more accessible to people with disabilities . ARIA attributes are HTML attributes that supplement native HTML semantics by exposing roles, states, and properties to assistive technologies when no native mechanism exists .

**Technical Definition**
ARIA is a W3C specification (currently WAI-ARIA 1.2) that provides an ontology of roles, states, and properties. ARIA attributes are written as `aria-*` attributes or as the `role` attribute on HTML elements. They are mapped by user agents to platform accessibility APIs (such as MSAA, UIA, and ATK), which assistive technologies consume. ARIA does **not** add behaviour, keyboard interaction, or focus management — it only provides semantics . The `role` attribute defines what an element **is**; `aria-*` states and properties supplement descriptive information .

**Beginner-Friendly Explanation**
ARIA attributes are extra labels you add to HTML elements to tell screen readers and other assistive tools what an element is and what it is doing. For example, if you build a custom button out of a `<div>`, you use `role="button"` to tell the screen reader “this is a button,” and `aria-pressed` to tell it whether the button is pressed. ARIA does not make the element work — you still have to write the JavaScript for clicks and keyboard support — but it makes the element understandable to users who cannot see it.

---

### Key Characteristics

- **Semantic-only layer**: ARIA provides roles, states, and properties; it does **not** provide behaviour, keyboard support, or focus management .
- **Supplement, not replacement**: ARIA is meant to fill gaps in native HTML semantics, not to replace native elements that already work .
- **Two main surfaces**: The `role` attribute (what an element is) and `aria-*` attributes (states and properties) .
- **High risk of harm**: Incorrect ARIA can make accessibility worse than no ARIA; WebAIM’s survey found that pages with ARIA averaged 41% more detected errors than those without .
- **Keyboard obligation**: Every interactive ARIA widget must be operable by keyboard .
- **Focus responsibility**: ARIA does not manage focus; developers must implement focus movement, focus traps, and focus restoration .
- **Naming and description are critical**: Every focusable, interactive element must have an accessible name; descriptions are optional supplements .

---

### Prerequisites

1. **Solid HTML semantics**: understanding native elements, attributes, and content models.
2. **The accessibility tree**: how browsers expose roles, names, states, and properties to assistive technologies.
3. **Keyboard interaction patterns**: Tab order, focus management, and standard widget keyboard conventions.
4. **ARIA Authoring Practices Guide (APG)**: the W3C patterns for widgets such as dialogs, tabs, and menus.
5. **Screen-reader testing**: hands-on experience with at least one screen reader (NVDA, JAWS, VoiceOver).

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Native HTML semantics** | The first rule of ARIA is to use native HTML whenever it provides the required semantics and behaviour . |
| **WCAG conformance** | ARIA is essential for WCAG 4.1.2 (Name, Role, Value) and 1.3.1 (Info and Relationships). |
| **JavaScript / DOM** | ARIA states must be updated dynamically via JavaScript as UI state changes. |
| **Focus management** | ARIA widgets require manual focus handling; ARIA does not provide it . |
| **Testing** | Automated accessibility checkers catch some ARIA errors, but manual screen-reader testing is essential. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **ARIA** — the specification
2. `role` — the element’s semantic identity
3. `aria-label` — string-based accessible name
4. `aria-labelledby` — referenced accessible name
5. `aria-describedby` — referenced accessible description
6. `aria-hidden` — removal from the accessibility tree
7. `aria-live` — live-region politeness
8. `aria-atomic` — live-region announcement granularity
9. `aria-busy` — live-region update suppression
10. `aria-checked` — checked state
11. `aria-disabled` — disabled state
12. `aria-expanded` — expanded/collapsed state
13. `aria-haspopup` — popup type indication
14. `aria-pressed` — toggle-button state
15. `aria-selected` — selected state
16. `aria-current` — current-item indicator
17. `aria-controls` — controlled-element relationship
18. `aria-owns` — ownership relationship
19. `aria-activedescendant` — active descendant within a composite widget
20. `aria-autocomplete` — autocomplete behaviour
21. `aria-multiline` — multi-line text input indication
22. `aria-multiselectable` — multi-select indication
23. `aria-orientation` — widget orientation
24. `aria-placeholder` — placeholder hint
25. `aria-readonly` — read-only state
26. `aria-required` — required state
27. `aria-sort` — sort direction
28. `aria-valuemax` — maximum value
29. `aria-valuemin` — minimum value
30. `aria-valuenow` — current value
31. `aria-valuetext` — human-readable value text
32. **ARIA attribute best practices**
33. **ARIA pitfalls**
34. **First rule of ARIA**

---

## 1. ARIA — The Specification

### Definitions

**Core Definition**
ARIA (Accessible Rich Internet Applications) is a W3C specification that provides roles, states, and properties to make web content and web applications more accessible to people with disabilities .

**Technical Definition**
WAI-ARIA 1.2 defines an ontology of abstract and concrete roles, supported states and properties, and the rules for their use in HTML. ARIA attributes are mapped to platform accessibility APIs by user agents and are consumed by assistive technologies. ARIA is designed to supplement HTML, not to replace it .

**Beginner-Friendly Explanation**
ARIA is the standard that lets you add accessibility information to elements that HTML does not fully describe. It is the “extra vocabulary” for screen readers.

---

### Purposes

- **To expose custom widget semantics** when no native HTML element exists.
- **To provide accessible names and descriptions** for elements.
- **To announce dynamic content changes** via live regions.
- **To convey state changes** (checked, expanded, selected, busy) to assistive technologies.
- **To describe relationships** between elements (controls, owns, labelledby, describedby).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<element role="role-name" aria-state="value" aria-property="value">
```

**Component breakdown**

| Component | Description |
|---|---|
| `role` | The element’s semantic identity. |
| `aria-*` state | A dynamic state such as `aria-expanded`, `aria-checked`. |
| `aria-*` property | A relatively static property such as `aria-label`, `aria-controls`. |

#### Syntax Rules

1. **ARIA attributes are global**; they can be used on any element.
2. **`role` takes a single concrete role name** (or a space-separated fallback list) .
3. **`aria-*` attributes take strings, tokens, booleans, or ID references** depending on the attribute.
4. **ARIA does not add behaviour**; keyboard and focus management must be implemented in JavaScript .

#### Constraints and Limitations

- **“No ARIA is better than bad ARIA”** : incorrect ARIA can make accessibility worse .
- **ARIA roles do not cause browsers to provide keyboard behaviours or styling** .
- **Redundant ARIA on native elements is discouraged**; native semantics already provide the role .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Custom Progress Bar with ARIA

**Setup Guide**: Save as `aria-progress.html` and inspect with a screen reader or accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ARIA Progress Bar</title>
</head>
<body>
  <div id="progress" role="progressbar"
       aria-valuemin="0" aria-valuemax="100" aria-valuenow="40">
    40%
  </div>

  <script>
    const bar = document.getElementById('progress');
    // Simulate progress update
    setTimeout(() => {
      bar.setAttribute('aria-valuenow', '70');
      bar.textContent = '70%';
    }, 1000);
  </script>
</body>
</html>
```

**Expected output**: The div is announced as a progress bar with a value between 0 and 100, initially at 40 and then at 70 after one second.

**Why this output occurs**: `role="progressbar"` gives the element its semantic identity; `aria-valuemin`, `aria-valuemax`, and `aria-valuenow` describe its range and current value . The JavaScript updates `aria-valuenow` to keep the state in sync.

---

### Real-World Cases

1. **Custom sliders and progress bars**: Native `<progress>` is limited; ARIA range attributes describe custom widgets.
2. **Live search results**: `aria-live` announces result counts as the user types.
3. **Disclosure widgets**: `aria-expanded` conveys whether a panel is open.

---

### References

- MDN Web Docs — ARIA — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA
- W3C — WAI-ARIA 1.2 — https://www.w3.org/TR/wai-aria-1.2/
- W3C — Using ARIA — https://w3c.github.io/using-aria/

---

## 2. `role` — The Element’s Semantic Identity

### Definitions

**Core Definition**
The `role` attribute defines the semantic meaning of an element — what kind of widget or structure it represents — for assistive technologies.

**Technical Definition**
`role` takes a space-separated list of role tokens; the first recognised role is used. ARIA roles are classified as abstract (not for author use) or concrete. Concrete roles include `button`, `dialog`, `navigation`, `progressbar`, `checkbox`, `tab`, and many others. A role is a **promise** about behaviour: if you use `role="button"`, you must also provide button-like keyboard and click behaviour .

**Beginner-Friendly Explanation**
`role` tells the screen reader “this element is a button” or “this element is a navigation region.” It is the most fundamental ARIA attribute.

---

### Purposes

- **To define the semantic identity** of a custom widget or structure.
- **To override or clarify** the implicit role of an element.
- **To enable assistive technologies** to present appropriate interaction affordances.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div role="button" tabindex="0">Click me</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `role="button"` | The concrete ARIA role. |
| `tabindex="0"` | Required for keyboard focusability (ARIA does not provide it). |

#### Syntax Rules

1. **Use a concrete role**, not an abstract role .
2. **Do not change native semantics unnecessarily** (second rule of ARIA) .
3. **A role is a promise**: provide the expected keyboard behaviour .
4. **Only one primary role** should be used per element.

#### Constraints and Limitations

- **Roles do not add behaviour**: you must implement keyboard and focus handling .
- **Redundant roles on native elements** are discouraged .
- **Incorrect roles can harm accessibility** more than no role .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Custom Button with Full Keyboard Support

**Setup Guide**: Save as `role-button.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ARIA Button</title>
</head>
<body>
  <div role="button" tabindex="0" id="custom-btn"
       onclick="activate()" onkeydown="if(event.key==='Enter'||event.key===' ') activate()">
    Save
  </div>

  <script>
    function activate() {
      alert('Saved!');
    }
  </script>
</body>
</html>
```

**Expected output**: The div is announced as a button, is focusable via Tab, and activates on click, Enter, or Space.

**Why this output occurs**: `role="button"` provides the semantic identity; `tabindex="0"` makes it focusable; the event handlers provide the expected keyboard and click behaviour. This follows the third rule of ARIA: all interactive ARIA controls must be usable with the keyboard .

---

### Real-World Cases

1. **Custom disclosure widgets**: `role="button"` with `aria-expanded`.
2. **Tabs and tab panels**: `role="tab"`, `role="tablist"`, `role="tabpanel"`.
3. **Dialogs**: `role="dialog"` with `aria-modal` and focus management.

---

### References

- MDN Web Docs — ARIA roles — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles
- W3C — Using ARIA: Second rule — https://w3c.github.io/using-aria/#second

---

## 3. `aria-label` — String-Based Accessible Name

### Definitions

**Core Definition**
`aria-label` provides a string that becomes the accessible name of an element when no visible label text exists.

**Technical Definition**
`aria-label` takes a free-form string. It is used when an element’s role permits naming and no visible text can serve as the name. It has lower priority than `aria-labelledby` in the accessible-name calculation .

**Beginner-Friendly Explanation**
Use `aria-label` when an element has no visible text label — for example, an icon-only button. It gives the button a name that screen readers can announce.

---

### Purposes

- **To name icon-only controls** (e.g. a magnifying-glass search button) .
- **To provide a name when no visible text exists** and `aria-labelledby` cannot be used.
- **To override a native label** in specific custom scenarios.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button aria-label="Search">🔍</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-label="Search"` | The accessible name string. |

#### Syntax Rules

1. **Use only when no visible label text exists** .
2. **Prefer `aria-labelledby`** when suitable visible text exists .
3. **Do not use `aria-label` on elements that do not support naming** from author attributes.

#### Constraints and Limitations

- **It does not create a visible label**; sighted users still see only the icon or content.
- **If visible text exists, use it**; `aria-label` overrides visible text, which can cause a mismatch.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Icon-Only Button

**Setup Guide**: Save as `aria-label.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-label Example</title>
</head>
<body>
  <button type="button" aria-label="Close dialog">✕</button>
</body>
</html>
```

**Expected output**: The button displays “✕” but is announced as “Close dialog, button” by a screen reader.

**Why this output occurs**: `aria-label` provides the accessible name; the visible “✕” is ignored as a naming source because `aria-label` takes priority.

---

### Real-World Cases

1. **Icon buttons**: Close, search, menu, and social-media icons.
2. **Search inputs**: Naming an input that has only a placeholder.
3. **Landmark regions**: Naming a `<nav>` when no visible heading exists.

---

### References

- MDN Web Docs — `aria-label` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-label
- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/

---

## 4. `aria-labelledby` — Referenced Accessible Name

### Definitions

**Core Definition**
`aria-labelledby` identifies the element (or elements) whose text content serves as the accessible name of the current element.

**Technical Definition**
`aria-labelledby` takes a space-separated list of IDs. The text content of the referenced elements is concatenated to form the accessible name. It has the highest priority in the accessible-name calculation and can reference hidden elements .

**Beginner-Friendly Explanation**
`aria-labelledby` points to another element on the page and says “use that element’s text as my name.” It is the preferred naming method when visible text exists.

---

### Purposes

- **To reuse visible text as the accessible name** (e.g. a section heading naming a region).
- **To combine multiple text sources** into a single name.
- **To name an element using text that is already visible** on the page .

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<h2 id="section-title">Account Settings</h2>
<section aria-labelledby="section-title">…</section>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-labelledby="section-title"` | Space-separated IDs of naming elements. |
| Referenced elements | Their text content forms the name. |

#### Syntax Rules

1. **Prefer `aria-labelledby` over `aria-label`** when visible text exists .
2. **Reference elements must exist in the DOM**.
3. **Hidden referenced elements are still used** for the name .

#### Constraints and Limitations

- **Referenced text must be meaningful** as a name.
- **Overly long references** can produce verbose names.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Naming a Region with a Heading

**Setup Guide**: Save as `aria-labelledby.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-labelledby Example</title>
</head>
<body>
  <h2 id="billing-heading">Billing Address</h2>
  <section aria-labelledby="billing-heading">
    <p>123 Main Street</p>
  </section>
</body>
</html>
```

**Expected output**: The section is announced as “Billing Address” region.

**Why this output occurs**: `aria-labelledby` references the heading, whose text becomes the section’s accessible name.

---

### Real-World Cases

1. **Landmark regions**: Naming `<nav>` and `<section>` elements by their headings.
2. **Form groups**: Naming a fieldset by a visible heading.
3. **Dialogs**: Naming a dialog by its title.

---

### References

- MDN Web Docs — `aria-labelledby` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-labelledby
- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/

---

## 5. `aria-describedby` — Referenced Accessible Description

### Definitions

**Core Definition**
`aria-describedby` identifies the element (or elements) whose text content serves as the accessible description of the current element.

**Technical Definition**
`aria-describedby` takes a space-separated list of IDs. The referenced text becomes the accessible description, which is announced after the name and role. Unlike `aria-labelledby`, it does **not** provide the name .

**Beginner-Friendly Explanation**
`aria-describedby` gives extra information about an element — like instructions or error messages — that screen readers announce after the name.

---

### Purposes

- **To provide supplemental instructions** for form fields .
- **To associate error messages** with their inputs.
- **To describe an element** with additional context that is too long for a name.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<label for="pw">Password</label>
<input type="password" id="pw" aria-describedby="pw-help">
<p id="pw-help">Must be at least 12 characters.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-describedby="pw-help"` | IDs of describing elements. |
| Referenced elements | Their text becomes the description. |

#### Syntax Rules

1. **Use for supplemental information**, not for the primary name .
2. **Reference existing elements** in the DOM.
3. **Descriptions are announced after the name and role** .

#### Constraints and Limitations

- **Descriptions are optional** and may be skipped by some screen readers by default .
- **Do not duplicate information** that is already in the name.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Form Field with Instructions

**Setup Guide**: Save as `aria-describedby.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-describedby Example</title>
</head>
<body>
  <label for="email">Email</label>
  <input type="email" id="email" aria-describedby="email-hint">
  <p id="email-hint">We will never share your email.</p>
</body>
</html>
```

**Expected output**: The email input is announced as “Email, edit text” followed by “We will never share your email.”

**Why this output occurs**: `aria-describedby` references the hint paragraph, which becomes the accessible description.

---

### Real-World Cases

1. **Password requirements**: Describing format rules.
2. **Error messages**: Associating validation errors with inputs.
3. **Help text**: Additional context for complex fields.

---

### References

- MDN Web Docs — `aria-describedby` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-describedby
- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/

---

## 6. `aria-hidden` — Removal from the Accessibility Tree

### Definitions

**Core Definition**
`aria-hidden` indicates whether an element is exposed to assistive technologies. `aria-hidden="true"` removes the element and its subtree from the accessibility tree.

**Technical Definition**
`aria-hidden` takes `true`, `false`, or `undefined`. When `true`, the element is hidden from assistive technologies but remains visible on screen. It must **not** be used on focusable elements or elements containing focusable descendants .

**Beginner-Friendly Explanation**
`aria-hidden="true"` tells screen readers to ignore an element. It is used for decorative content that adds no meaning.

---

### Purposes

- **To hide decorative content** (icons, illustrations) from assistive technologies.
- **To remove duplicate content** that is already described elsewhere.
- **To simplify the accessibility tree** in complex UIs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<span aria-hidden="true">★</span>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-hidden="true"` | Removes the element from the accessibility tree. |

#### Syntax Rules

1. **Use only for content with no semantic value**.
2. **Do not use on focusable elements** or elements with focusable descendants .
3. **`aria-hidden` does not hide content visually**.

#### Constraints and Limitations

- **Focusable hidden elements create a “ghost focus” problem**: keyboard users can focus elements that screen-reader users cannot perceive .
- **It is not a substitute for `display: none` or `hidden`** when content should be removed entirely.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hiding a Decorative Icon

**Setup Guide**: Save as `aria-hidden.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-hidden Example</title>
</head>
<body>
  <button type="button">
    <span aria-hidden="true">★</span>
    Add to favourites
  </button>
</body>
</html>
```

**Expected output**: The star icon is not announced; the button is announced as “Add to favourites, button.”

**Why this output occurs**: `aria-hidden="true"` removes the star from the accessibility tree while leaving the visible text intact.

---

### Real-World Cases

1. **Decorative icons**: Hiding icons that duplicate adjacent text.
2. **Custom checkboxes**: Hiding the visual checkbox and exposing a styled element with proper ARIA.
3. **Complex visualisations**: Hiding purely visual elements that have a text alternative nearby.

---

### References

- MDN Web Docs — `aria-hidden` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden
- W3C — ARIA in HTML — https://www.w3.org/TR/html-aria/

---

## 7. `aria-live` — Live-Region Politeness

### Definitions

**Core Definition**
`aria-live` indicates that an element is a live region whose content updates should be announced by assistive technologies. Its value controls the politeness of announcements.

**Technical Definition**
`aria-live` accepts `off`, `polite`, or `assertive`. `polite` waits for the user to be idle before announcing; `assertive` interrupts immediately. Live regions are used for dynamic content such as status messages, chat logs, and search-result counts .

**Beginner-Friendly Explanation**
`aria-live` tells screen readers “this part of the page changes, so please announce updates.” Use `polite` for non-urgent updates and `assertive` for urgent ones.

---

### Purposes

- **To announce dynamic content changes** to screen-reader users.
- **To provide status messages** (e.g. “Item added to cart”).
- **To announce search-result counts** and other asynchronous updates.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div aria-live="polite" id="status"></div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `off` | No announcements (default). |
| `polite` | Announce when the user is idle. |
| `assertive` | Announce immediately, interrupting. |

#### Syntax Rules

1. **The live region must exist in the DOM before content changes**.
2. **Use `polite` for most updates**; reserve `assertive` for urgent alerts.
3. **Combine with `aria-atomic` and `aria-busy`** for finer control .

#### Constraints and Limitations

- **`assertive` can be disruptive** if overused.
- **Live regions must be present at page load**; dynamically inserting the live region itself may not announce content.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Polite Status Message

**Setup Guide**: Save as `aria-live.html`. Click the button to trigger an update.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-live Example</title>
</head>
<body>
  <button type="button" onclick="document.getElementById('status').textContent = 'Saved successfully!'">
    Save
  </button>
  <div aria-live="polite" id="status"></div>
</body>
</html>
```

**Expected output**: After clicking Save, a screen reader announces “Saved successfully!” when the user is idle.

**Why this output occurs**: The `aria-live="polite"` region announces its updated content after the user finishes their current interaction.

---

### Real-World Cases

1. **Form validation summaries**: Announcing errors after submission.
2. **Shopping carts**: “Item added” messages.
3. **Chat applications**: New-message announcements.

---

### References

- MDN Web Docs — `aria-live` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-live
- W3C — ARIA Live Regions — https://www.w3.org/WAI/ARIA/apg/practices/live-regions/

---

## 8. `aria-atomic` — Live-Region Announcement Granularity

### Definitions

**Core Definition**
`aria-atomic` indicates whether the entire live region or only the changed content should be announced when a change occurs.

**Technical Definition**
`aria-atomic` takes `true` or `false`. When `true`, the entire live region’s content is announced; when `false`, only the changed nodes are announced .

**Beginner-Friendly Explanation**
`aria-atomic="true"` means “read the whole region every time something changes.” `aria-atomic="false"` means “only read what changed.”

---

### Purposes

- **To announce the full context** when a small part of a message changes.
- **To avoid partial or confusing announcements**.
- **To control verbosity** in live regions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div aria-live="polite" aria-atomic="true">
  <span id="count">5</span> items in cart
</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Announce the entire region. |
| `false` | Announce only the changed portion (default). |

#### Syntax Rules

1. **Use `true` when the whole message must be understood together**.
2. **Use `false` when only the changed value is relevant**.

#### Constraints and Limitations

- **`true` can produce verbose announcements** for large regions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Atomic Cart Count

**Setup Guide**: Save as `aria-atomic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-atomic Example</title>
</head>
<body>
  <div aria-live="polite" aria-atomic="true">
    <span id="count">5</span> items in cart
  </div>

  <button type="button" onclick="document.getElementById('count').textContent = '6'">
    Add item
  </button>
</body>
</html>
```

**Expected output**: After clicking “Add item,” the screen reader announces “6 items in cart” — the entire region, not just “6.”

**Why this output occurs**: `aria-atomic="true"` causes the full text to be announced when any part changes.

---

### Real-World Cases

1. **Cart and counter updates**: Announcing “6 items in cart” rather than just “6.”
2. **Status summaries**: Announcing a full status message when a timestamp changes.

---

### References

- MDN Web Docs — `aria-atomic` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-atomic

---

## 9. `aria-busy` — Live-Region Update Suppression

### Definitions

**Core Definition**
`aria-busy` indicates that an element is being updated and that assistive technologies should wait until the update is complete before announcing changes.

**Technical Definition**
`aria-busy` takes `true` or `false`. When `true`, changes inside the element are not added to the live-region announcement queue until the attribute is cleared .

**Beginner-Friendly Explanation**
`aria-busy="true"` tells screen readers “hold on, I’m still loading — don’t announce anything yet.” When the update is finished, set it to `false` or remove it.

---

### Purposes

- **To suppress announcements during loading**.
- **To prevent rapid-fire updates from overwhelming users**.
- **To batch multiple changes** into a single announcement.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div aria-live="polite" aria-busy="true">Loading…</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Suppress announcements. |
| `false` | Resume announcements (default). |

#### Syntax Rules

1. **Set `aria-busy="true"` before beginning a bulk update**.
2. **Clear it (set to `false` or remove) when the update is complete** .

#### Constraints and Limitations

- **Leaving `aria-busy="true"` permanently** prevents all announcements.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Suppressing Announcements During Load

**Setup Guide**: Save as `aria-busy.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-busy Example</title>
</head>
<body>
  <div aria-live="polite" aria-busy="true" id="results">Loading…</div>

  <script>
    setTimeout(() => {
      const results = document.getElementById('results');
      results.textContent = '10 results found';
      results.setAttribute('aria-busy', 'false'); // announce now
    }, 2000);
  </script>
</body>
</html>
```

**Expected output**: After two seconds, the screen reader announces “10 results found” once — not “Loading…” and then “10 results found.”

**Why this output occurs**: `aria-busy="true"` suppresses the intermediate “Loading…” announcement; clearing it releases the final content for announcement.

---

### Real-World Cases

1. **Search result updates**: Suppressing intermediate loading messages.
2. **Chat message bursts**: Batching multiple rapid messages.
3. **Data refreshes**: Announcing the final result once.

---

### References

- MDN Web Docs — `aria-busy` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-busy

---

## 10. `aria-checked` — Checked State

### Definitions

**Core Definition**
`aria-checked` indicates the current checked state of checkboxes, radio buttons, and other widgets that can be checked.

**Technical Definition**
`aria-checked` accepts `true`, `false`, `mixed`, or `undefined`. `mixed` represents a tri-state checkbox. It is used on roles such as `checkbox`, `radio`, `menuitemcheckbox`, and `switch` .

**Beginner-Friendly Explanation**
`aria-checked` tells the screen reader whether a custom checkbox is checked, unchecked, or partially checked.

---

### Purposes

- **To convey checked state** for custom checkbox and radio widgets.
- **To support tri-state checkboxes** with `mixed`.
- **To keep custom widgets in sync** with visual state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div role="checkbox" aria-checked="false" tabindex="0">Subscribe</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Checked. |
| `false` | Unchecked. |
| `mixed` | Partially checked (tri-state). |

#### Syntax Rules

1. **Use with roles that support checking** (`checkbox`, `radio`, `switch`).
2. **Update the value in JavaScript** when the visual state changes.
3. **Use `mixed` only for tri-state widgets**.

#### Constraints and Limitations

- **`aria-checked` does not change the visual state**; CSS and JavaScript must handle that.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Checkbox

**Setup Guide**: Save as `aria-checked.html`. Click to toggle.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-checked Example</title>
</head>
<body>
  <div role="checkbox" aria-checked="false" tabindex="0" id="cb">
    Subscribe to newsletter
  </div>

  <script>
    const cb = document.getElementById('cb');
    cb.addEventListener('click', () => {
      const checked = cb.getAttribute('aria-checked') === 'true';
      cb.setAttribute('aria-checked', String(!checked));
    });
    cb.addEventListener('keydown', (e) => {
      if (e.key === ' ') { e.preventDefault(); cb.click(); }
    });
  </script>
</body>
</html>
```

**Expected output**: The div is announced as a checkbox, initially unchecked, and toggles between checked and unchecked on click or Space.

**Why this output occurs**: `aria-checked` conveys the state; JavaScript toggles it on interaction, and `role="checkbox"` provides the semantic identity.

---

### Real-World Cases

1. **Custom-styled checkboxes**: Replacing native checkboxes with styled elements.
2. **Toggle switches**: `role="switch"` with `aria-checked`.
3. **Tri-state tree checkboxes**: Parent nodes with `mixed` state.

---

### References

- MDN Web Docs — `aria-checked` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-checked

---

## 11. `aria-disabled` — Disabled State

### Definitions

**Core Definition**
`aria-disabled` indicates that an element is perceivable but disabled, so it is not editable or otherwise operable .

**Technical Definition**
`aria-disabled` takes `true` or `false`. It is used on roles that support a disabled state. Unlike the native `disabled` attribute, it does not automatically prevent interaction; developers must also handle behaviour.

**Beginner-Friendly Explanation**
`aria-disabled="true"` tells the screen reader that an element is disabled. You must also prevent clicks and focus in JavaScript.

---

### Purposes

- **To convey disabled state** for custom widgets.
- **To keep an element in the accessibility tree** while marking it unavailable.
- **To support elements that cannot use the native `disabled` attribute**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button aria-disabled="true">Submit</button>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Disabled. |
| `false` | Enabled (default). |

#### Syntax Rules

1. **`aria-disabled` does not prevent interaction**; handle click and focus in JavaScript.
2. **Use native `disabled` when possible**.
3. **Keep the element focusable** if users need to discover why it is disabled.

#### Constraints and Limitations

- **Screen readers may still allow focus** on `aria-disabled` elements.
- **It does not visually style the element**; CSS must handle appearance.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabled Custom Button

**Setup Guide**: Save as `aria-disabled.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-disabled Example</title>
</head>
<body>
  <div role="button" tabindex="0" aria-disabled="true" id="btn">
    Submit
  </div>

  <script>
    const btn = document.getElementById('btn');
    btn.addEventListener('click', (e) => {
      if (btn.getAttribute('aria-disabled') === 'true') {
        e.preventDefault();
        alert('This button is disabled.');
      }
    });
  </script>
</body>
</html>
```

**Expected output**: The div is announced as a disabled button; clicking it shows an explanation instead of submitting.

**Why this output occurs**: `aria-disabled="true"` conveys the disabled state; the JavaScript blocks the action and explains why.

---

### Real-World Cases

1. **Form submission buttons**: Disabled until validation passes.
2. **Custom menu items**: Disabled items in a custom menu.
3. **Inactive toolbar buttons**: Buttons that are contextually unavailable.

---

### References

- MDN Web Docs — `aria-disabled` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-disabled

---

## 12. `aria-expanded` — Expanded/Collapsed State

### Definitions

**Core Definition**
`aria-expanded` indicates whether an element, or another grouping element it controls, is currently expanded or collapsed .

**Technical Definition**
`aria-expanded` takes `true`, `false`, or `undefined`. It is used on disclosure buttons, tree items, and comboboxes to convey expansion state.

**Beginner-Friendly Explanation**
`aria-expanded` tells the screen reader whether a collapsible section is open or closed.

---

### Purposes

- **To convey disclosure state** for accordions and collapsible panels.
- **To indicate menu and tree-item expansion**.
- **To support combobox state**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button aria-expanded="false" aria-controls="panel">Toggle</button>
<div id="panel" hidden>Panel content</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Expanded. |
| `false` | Collapsed. |

#### Syntax Rules

1. **Update `aria-expanded` when toggling visibility**.
2. **Pair with `aria-controls`** to indicate the controlled element.
3. **Use `false` for initially collapsed content**.

#### Constraints and Limitations

- **Static `aria-expanded` values** that never change are a common pitfall .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accordion Toggle

**Setup Guide**: Save as `aria-expanded.html`. Click to toggle.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-expanded Example</title>
</head>
<body>
  <button aria-expanded="false" aria-controls="content" id="toggle">Show details</button>
  <div id="content" hidden>Details content.</div>

  <script>
    const btn = document.getElementById('toggle');
    const content = document.getElementById('content');
    btn.addEventListener('click', () => {
      const expanded = btn.getAttribute('aria-expanded') === 'true';
      btn.setAttribute('aria-expanded', String(!expanded));
      content.hidden = expanded;
    });
  </script>
</body>
</html>
```

**Expected output**: The button is announced as collapsed; clicking it expands the content and the button is announced as expanded.

**Why this output occurs**: `aria-expanded` reflects the toggle state, and `hidden` controls visibility.

---

### Real-World Cases

1. **Accordions**: FAQ and settings panels.
2. **Tree views**: Expanding and collapsing branches.
3. **Comboboxes**: Showing whether the listbox is open.

---

### References

- MDN Web Docs — `aria-expanded` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-expanded

---

## 13. `aria-haspopup` — Popup Type Indication

### Definitions

**Core Definition**
`aria-haspopup` indicates that an element triggers a popup menu, dialog, listbox, tree, or grid.

**Technical Definition**
`aria-haspopup` accepts `menu`, `listbox`, `tree`, `grid`, `dialog`, or `true` (equivalent to `menu`). It tells assistive technologies what kind of popup will appear.

**Beginner-Friendly Explanation**
`aria-haspopup` tells the screen reader “this button opens a popup” and what kind of popup it is.

---

### Purposes

- **To indicate that a control opens a popup**.
- **To specify the popup type** for better announcements.
- **To support menu buttons and comboboxes**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button aria-haspopup="menu" aria-expanded="false">Options</button>
```

**Component breakdown**

| Value | Popup type |
|---|---|
| `menu` | Menu. |
| `listbox` | Listbox. |
| `tree` | Tree. |
| `grid` | Grid. |
| `dialog` | Dialog. |
| `true` | Equivalent to `menu`. |

#### Syntax Rules

1. **Use the most specific popup type**.
2. **Pair with `aria-expanded`** to reflect open/closed state.

#### Constraints and Limitations

- **`aria-haspopup` does not create the popup**; JavaScript must implement it.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Menu Button

**Setup Guide**: Save as `aria-haspopup.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-haspopup Example</title>
</head>
<body>
  <button aria-haspopup="menu" aria-expanded="false" id="menu-btn">
    Actions
  </button>
  <ul role="menu" hidden id="menu">
    <li role="menuitem">Edit</li>
    <li role="menuitem">Delete</li>
  </ul>

  <script>
    const btn = document.getElementById('menu-btn');
    const menu = document.getElementById('menu');
    btn.addEventListener('click', () => {
      const expanded = btn.getAttribute('aria-expanded') === 'true';
      btn.setAttribute('aria-expanded', String(!expanded));
      menu.hidden = expanded;
    });
  </script>
</body>
</html>
```

**Expected output**: The button is announced as having a popup menu; clicking it opens the menu and updates `aria-expanded`.

**Why this output occurs**: `aria-haspopup="menu"` tells assistive technologies that a menu will appear; `aria-expanded` tracks whether it is open.

---

### Real-World Cases

1. **Menu buttons**: Dropdown action menus.
2. **Comboboxes**: Showing that a listbox will appear.
3. **Dialog triggers**: Buttons that open modal dialogs.

---

### References

- MDN Web Docs — `aria-haspopup` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-haspopup

---

## 14. `aria-pressed` — Toggle-Button State

### Definitions

**Core Definition**
`aria-pressed` indicates the current pressed state of a toggle button.

**Technical Definition**
`aria-pressed` accepts `true`, `false`, or `mixed`. It is used on `role="button"` to create a toggle button (e.g. a bold-formatting button that stays pressed).

**Beginner-Friendly Explanation**
`aria-pressed` tells the screen reader whether a toggle button is currently “on” or “off.”

---

### Purposes

- **To convey toggle-button state**.
- **To distinguish toggle buttons from regular buttons**.
- **To support formatting toolbars**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button aria-pressed="false" id="bold">Bold</button>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Pressed (on). |
| `false` | Not pressed (off). |
| `mixed` | Partially pressed. |

#### Syntax Rules

1. **Use on `role="button"` only**.
2. **Toggle the value on click**.
3. **Do not use `aria-pressed` on regular buttons**.

#### Constraints and Limitations

- **`aria-pressed` does not change the visual style**; CSS must reflect the state.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Bold Toggle Button

**Setup Guide**: Save as `aria-pressed.html`. Click to toggle.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-pressed Example</title>
</head>
<body>
  <button aria-pressed="false" id="bold-btn">Bold</button>

  <script>
    const btn = document.getElementById('bold-btn');
    btn.addEventListener('click', () => {
      const pressed = btn.getAttribute('aria-pressed') === 'true';
      btn.setAttribute('aria-pressed', String(!pressed));
    });
  </script>
</body>
</html>
```

**Expected output**: The button is announced as “Bold, toggle button, not pressed” and toggles to “pressed” on click.

**Why this output occurs**: `aria-pressed` conveys the toggle state; `role="button"` (implicit on `<button>`) provides the identity.

---

### Real-World Cases

1. **Formatting toolbars**: Bold, italic, and underline toggles.
2. **Filter toggles**: Showing or hiding categories.
3. **Mute buttons**: Toggling audio mute.

---

### References

- MDN Web Docs — `aria-pressed` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-pressed

---

## 15. `aria-selected` — Selected State

### Definitions

**Core Definition**
`aria-selected` indicates the current selected state of a widget such as a tab, option, or grid cell.

**Technical Definition**
`aria-selected` accepts `true`, `false`, or `undefined`. It is used on roles such as `tab`, `option`, `gridcell`, and `row`.

**Beginner-Friendly Explanation**
`aria-selected` tells the screen reader which tab, option, or item is currently selected.

---

### Purposes

- **To convey selected state** in tabs, listboxes, and grids.
- **To distinguish selection from focus**.
- **To support multi-select widgets**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div role="tab" aria-selected="true">Tab 1</div>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `true` | Selected. |
| `false` | Not selected. |

#### Syntax Rules

1. **Update `aria-selected` when selection changes**.
2. **Use `aria-multiselectable` on the container** for multi-select.
3. **Do not confuse selection with focus**.

#### Constraints and Limitations

- **Selection state must be kept in sync with visual state**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Tab Selection

**Setup Guide**: Save as `aria-selected.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-selected Example</title>
</head>
<body>
  <div role="tablist">
    <div role="tab" aria-selected="true" tabindex="0">Tab 1</div>
    <div role="tab" aria-selected="false" tabindex="-1">Tab 2</div>
  </div>
</body>
</html>
```

**Expected output**: Tab 1 is announced as selected; Tab 2 is announced as not selected.

**Why this output occurs**: `aria-selected` conveys the selected state, and the tab roles provide the widget identity.

---

### Real-World Cases

1. **Tab interfaces**: Indicating the active tab.
2. **Listbox selection**: Single or multi-select lists.
3. **Grid cell selection**: Spreadsheet-like interfaces.

---

### References

- MDN Web Docs — `aria-selected` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-selected

---

## 16. `aria-current` — Current-Item Indicator

### Definitions

**Core Definition**
`aria-current` indicates the element that represents the current item within a set, such as the current page in navigation or the current step in a process.

**Technical Definition**
`aria-current` accepts `page`, `step`, `location`, `date`, `time`, `true`, or `false` (default). It was added in ARIA 1.1 .

**Beginner-Friendly Explanation**
`aria-current` tells the screen reader “this is the current page” or “this is the current step.” It is commonly used in navigation menus.

---

### Purposes

- **To indicate the current page** in navigation.
- **To indicate the current step** in a wizard.
- **To indicate the current location** or time.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="/about" aria-current="page">About</a>
```

**Component breakdown**

| Value | Meaning |
|---|---|
| `page` | Current page. |
| `step` | Current step. |
| `location` | Current location. |
| `true` | Generic current item. |

#### Syntax Rules

1. **Use `page` for navigation links**.
2. **Use `step` for multi-step processes**.
3. **Only one element per set should have `aria-current`**.

#### Constraints and Limitations

- **Overuse creates confusion**; use only for the current item.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Current Navigation Link

**Setup Guide**: Save as `aria-current.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-current Example</title>
</head>
<body>
  <nav>
    <a href="/">Home</a>
    <a href="/about" aria-current="page">About</a>
    <a href="/contact">Contact</a>
  </nav>
</body>
</html>
```

**Expected output**: The “About” link is announced as the current page.

**Why this output occurs**: `aria-current="page"` identifies the active navigation link.

---

### Real-World Cases

1. **Breadcrumb navigation**: Marking the current page.
2. **Wizard steps**: Indicating the active step.
3. **Pagination**: Marking the current page number.

---

### References

- MDN Web Docs — `aria-current` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-current

---

## 17. `aria-controls` — Controlled-Element Relationship

### Definitions

**Core Definition**
`aria-controls` identifies the element (or elements) whose contents or presence are controlled by the current element .

**Technical Definition**
`aria-controls` takes a space-separated list of IDs. It is used on disclosure buttons, tabs, and comboboxes to point to the element they control.

**Beginner-Friendly Explanation**
`aria-controls` tells the screen reader which element a control affects. For example, a tab’s `aria-controls` points to the tab panel it activates.

---

### Purposes

- **To establish a control–target relationship**.
- **To support tab and disclosure patterns**.
- **To help assistive technologies navigate** between a control and its target.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<button aria-controls="panel1" aria-expanded="false">Toggle</button>
<div id="panel1">Content</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-controls="panel1"` | ID of the controlled element. |

#### Syntax Rules

1. **Reference existing IDs**.
2. **Use on the controlling element**, not the controlled element.
3. **Combine with `aria-expanded`** for disclosure widgets.

#### Constraints and Limitations

- **Support varies across screen readers**; do not rely on it as the sole relationship indicator.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Tab with Controlled Panel

**Setup Guide**: Save as `aria-controls.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-controls Example</title>
</head>
<body>
  <div role="tab" aria-selected="true" aria-controls="panel1" tabindex="0">Tab 1</div>
  <div role="tabpanel" id="panel1">Panel 1 content</div>
</body>
</html>
```

**Expected output**: The tab is associated with the panel it controls.

**Why this output occurs**: `aria-controls` points to the panel’s ID, establishing the relationship.

---

### Real-World Cases

1. **Tabs**: Associating tabs with tab panels.
2. **Disclosure widgets**: Associating buttons with collapsible panels.
3. **Comboboxes**: Associating inputs with listboxes.

---

### References

- MDN Web Docs — `aria-controls` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-controls

---

## 18. `aria-owns` — Ownership Relationship

### Definitions

**Core Definition**
`aria-owns` establishes a parent–child relationship between elements that cannot be represented by the DOM hierarchy.

**Technical Definition**
`aria-owns` takes a space-separated list of IDs. It is used when a child element cannot be a DOM descendant of its logical parent, such as a listbox whose options are rendered in a separate popup.

**Beginner-Friendly Explanation**
`aria-owns` tells the screen reader “treat this element as if it were a child of that element,” even though it is not in the DOM tree.

---

### Purposes

- **To fix broken parent–child relationships** caused by DOM structure.
- **To support composite widgets** with popups.
- **To associate non-descendant elements**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div role="listbox" aria-owns="option1 option2"></div>
<div id="option1" role="option">Option 1</div>
<div id="option2" role="option">Option 2</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-owns="id1 id2"` | IDs of elements to treat as children. |

#### Syntax Rules

1. **Use only when DOM hierarchy cannot express the relationship**.
2. **Reference existing IDs**.
3. **Avoid circular ownership**.

#### Constraints and Limitations

- **Overuse can create confusing accessibility trees**.

---

### Real-World Cases

1. **Comboboxes with popups**: Owning options rendered outside the input’s DOM subtree.
2. **Menus**: Owning menu items in a portal.
3. **Custom tree views**: Owning non-descendant nodes.

---

### References

- MDN Web Docs — `aria-owns` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-owns

---

## 19. `aria-activedescendant` — Active Descendant

### Definitions

**Core Definition**
`aria-activedescendant` identifies the currently active element when DOM focus is on a composite widget, such as a listbox or grid .

**Technical Definition**
`aria-activedescendant` takes a single ID reference. It allows a composite widget to retain DOM focus while indicating which descendant is currently active.

**Beginner-Friendly Explanation**
`aria-activedescendant` tells the screen reader which item inside a widget is currently “active” while the widget itself keeps focus.

---

### Purposes

- **To support composite widgets** that manage internal focus.
- **To indicate the active option** in a listbox or combobox.
- **To avoid moving DOM focus** between many elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<div role="listbox" tabindex="0" aria-activedescendant="opt2">
  <div id="opt1" role="option">Option 1</div>
  <div id="opt2" role="option">Option 2</div>
</div>
```

**Component breakdown**

| Component | Description |
|---|---|
| `aria-activedescendant="opt2"` | ID of the active descendant. |

#### Syntax Rules

1. **The container must be focusable** (`tabindex="0"`).
2. **The active descendant must be a DOM descendant** or owned via `aria-owns`.
3. **Update the ID as the active item changes**.

#### Constraints and Limitations

- **The active descendant must exist and be valid**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Listbox with Active Descendant

**Setup Guide**: Save as `aria-activedescendant.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-activedescendant Example</title>
</head>
<body>
  <div role="listbox" tabindex="0" aria-activedescendant="item2" id="listbox">
    <div id="item1" role="option">Apple</div>
    <div id="item2" role="option">Banana</div>
    <div id="item3" role="option">Cherry</div>
  </div>
</body>
</html>
```

**Expected output**: The listbox is focusable; the screen reader announces “Banana” as the active option while focus remains on the listbox.

**Why this output occurs**: `aria-activedescendant` points to the active option, allowing the container to manage focus internally.

---

### Real-World Cases

1. **Comboboxes**: Indicating the highlighted option while the input retains focus.
2. **Listboxes**: Arrow-key navigation without moving DOM focus.
3. **Grids and trees**: Composite widget navigation.

---

### References

- MDN Web Docs — `aria-activedescendant` — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-activedescendant

---

## 20–31. Additional ARIA Attributes

The following attributes are covered with their key definitions, purposes, and syntax rules.

### `aria-autocomplete`

- **Definition**: Indicates whether input completion suggestions are provided and how they are presented.
- **Values**: `none`, `inline`, `list`, `both`.
- **Purpose**: To tell assistive technologies that autocomplete suggestions are available.
- **Syntax**: `aria-autocomplete="list"` on a text input or combobox.
- **Example**: A search input with a suggestion list uses `aria-autocomplete="list"`.

### `aria-multiline`

- **Definition**: Indicates whether a text box accepts multiple lines of input.
- **Values**: `true` or `false`.
- **Purpose**: To convey multi-line text-input behaviour.
- **Syntax**: `aria-multiline="true"` on `role="textbox"`.

### `aria-multiselectable`

- **Definition**: Indicates that the user may select more than one item from the current selectable descendants.
- **Values**: `true` or `false`.
- **Purpose**: To support multi-select listboxes and grids.
- **Syntax**: `aria-multiselectable="true"` on the container role.

### `aria-orientation`

- **Definition**: Indicates whether a widget is oriented horizontally or vertically.
- **Values**: `horizontal`, `vertical`, `undefined`.
- **Purpose**: To convey orientation for sliders, scrollbars, and toolbars.
- **Syntax**: `aria-orientation="vertical"`.

### `aria-placeholder`

- **Definition**: Defines a short hint intended to aid the user with data entry when the control has no value.
- **Values**: A string.
- **Purpose**: To provide a placeholder for custom text inputs.
- **Syntax**: `aria-placeholder="Enter your name"` on `role="textbox"`.
- **Note**: This is not a replacement for a visible label; it disappears once text is entered.

### `aria-readonly`

- **Definition**: Indicates that the element is not editable but is otherwise operable.
- **Values**: `true` or `false`.
- **Purpose**: To convey read-only state for custom text inputs.
- **Syntax**: `aria-readonly="true"` on `role="textbox"` or `role="grid"`.

### `aria-required`

- **Definition**: Indicates that user input is required on the element before a form can be submitted.
- **Values**: `true` or `false`.
- **Purpose**: To convey required state for custom form controls.
- **Syntax**: `aria-required="true"` on `role="textbox"`, `role="combobox"`, etc.
- **Note**: Place it on the input element, not on a associated button .

### `aria-sort`

- **Definition**: Indicates whether items in a table or grid are sorted in ascending or descending order.
- **Values**: `none`, `ascending`, `descending`, `other`.
- **Purpose**: To convey sort direction on column headers.
- **Syntax**: `aria-sort="ascending"` on a `<th>` element .

### `aria-valuemax`, `aria-valuemin`, `aria-valuenow`

- **Definition**: Define the maximum, minimum, and current value for a range widget .
- **Values**: Numbers (or strings that parse as numbers).
- **Purpose**: To describe slider, progress bar, and spinbutton values.
- **Syntax**:

```html
<div role="slider" aria-valuemin="0" aria-valuemax="100" aria-valuenow="50"
     aria-valuetext="50 percent">50%</div>
```

- **Note**: `aria-valuenow` must be kept up to date.

### `aria-valuetext`

- **Definition**: Defines the human-readable text alternative of `aria-valuenow` for a range widget .
- **Values**: A string.
- **Purpose**: To provide readable values when the numeric value is not meaningful (e.g. “small”, “medium”, “large”).
- **Syntax**: `aria-valuetext="Medium"` alongside `aria-valuenow`.

---

## 32. ARIA Attribute Best Practices

### Definitions

**Core Definition**
ARIA best practices are guidelines for using ARIA attributes correctly, sparingly, and in harmony with native HTML semantics.

**Technical Definition**
Best practices are informed by the W3C Using ARIA document, the ARIA Authoring Practices Guide (APG), and the HTML-ARIA specification. They encompass the four rules of ARIA, the priority of native semantics, and the requirements for naming, keyboard support, and focus management.

**Beginner-Friendly Explanation**
Best practices help you use ARIA only when necessary and avoid the common traps that make accessibility worse.

---

### Purposes

- **To ensure ARIA supplements rather than replaces native HTML** .
- **To prevent accessibility failures** caused by incorrect or redundant ARIA .
- **To enforce keyboard and focus requirements** for interactive widgets .
- **To prioritise accessible names and descriptions** correctly .

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use native HTML first** | The first rule of ARIA: if a native element exists, use it . |
| **Do not change native semantics unnecessarily** | The second rule: preserve existing semantics . |
| **Ensure keyboard operability** | The third rule: all interactive ARIA controls must be keyboard-operable . |
| **Do not use `aria-hidden` on focusable elements** | The fourth rule: hidden content must not be focusable . |
| **Prefer `aria-labelledby` over `aria-label`** when visible text exists . |
| **Use `aria-describedby` only for supplemental descriptions** . |
| **Update states dynamically** | `aria-expanded`, `aria-checked`, and `aria-valuenow` must reflect current state. |
| **Test with screen readers** | Automated tools catch some errors, but manual testing is essential. |

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Native HTML vs ARIA

**Setup Guide**: Save as `best-practice-native.html`.

```html
<!-- GOOD: native button -->
<button type="button">Save</button>

<!-- BAD: div with ARIA role and manual keyboard handling -->
<!--
<div role="button" tabindex="0" onclick="save()"
     onkeydown="if(event.key==='Enter') save()">Save</div>
-->
```

**Expected output**: The native button is focusable, keyboard-activatable, and announced as a button without any ARIA.

**Why this output occurs**: The native `<button>` provides role, focusability, and keyboard activation for free. The ARIA version requires manual implementation and is more error-prone .

---

### References

- W3C — Using ARIA — https://w3c.github.io/using-aria/
- W3C — ARIA Authoring Practices Guide (APG) — https://www.w3.org/WAI/ARIA/apg/
- W3C — ARIA in HTML — https://www.w3.org/TR/html-aria/

---

## 33. ARIA Pitfalls

### Definitions

**Core Definition**
ARIA pitfalls are common mistakes in ARIA usage that can degrade accessibility rather than improve it.

**Technical Definition**
Pitfalls include adding roles without keyboard support, using `aria-hidden` on focusable elements, static state values, redundant ARIA on native elements, and missing focus management .

**Beginner-Friendly Explanation**
Pitfalls are the “gotchas” of ARIA. They are easy to fall into and can make your site harder to use.

---

### Purposes

- **To identify and avoid common ARIA mistakes**.
- **To prevent accessibility regressions** caused by incorrect ARIA.
- **To guide testing and review efforts**.

---

### Common Pitfalls

| Pitfall | Explanation |
|---|---|
| **Adding roles without keyboard support** | `role="button"` on a `<div>` does not add keyboard activation . |
| **`aria-hidden="true"` on focusable elements** | Creates “ghost focus” — keyboard users can focus invisible elements . |
| **Static `aria-expanded` values** | A value that never changes misrepresents the UI state . |
| **Redundant ARIA on native elements** | Adding `role="button"` to `<button>` is unnecessary and discouraged . |
| **Missing focus management** | ARIA does not manage focus; dialogs and menus need manual focus traps and restoration . |
| **Using ARIA for accessibility instead of visible labels** | `aria-label` without a visible label can create mismatches for sighted users. |
| **`aria-required` on the wrong element** | Placing it on a combobox button instead of the input is a common error . |
| **Assuming ARIA fixes broken HTML** | ARIA does not fix broken semantics . |

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Ghost Focus Pitfall

**Setup Guide**: Save as `pitfall-ghost.html`. Try tabbing to the hidden link.

```html
<!-- PITFALL: aria-hidden on a focusable element -->
<div aria-hidden="true">
  <a href="/secret">Hidden link</a>
</div>
```

**Expected output**: The link is invisible to screen readers but still reachable by keyboard — a confusing “ghost focus” experience.

**Why this output occurs**: `aria-hidden="true"` removes the element from the accessibility tree but does not remove it from the tab order. This violates the fourth rule of ARIA .

---

### References

- Accessibility.com — ARIA Myths That Cause Conformance Failures — https://www.accessibility.com/blog/aria-myths
- W3C — Developing a Keyboard Interface — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/

---

## 34. First Rule of ARIA

### Definitions

**Core Definition**
The first rule of ARIA states: “If you can use a native HTML element or attribute with the semantics and behaviour you require already built in, instead of re-purposing an element and adding an ARIA role, state or property to make it accessible, then do so” .

**Technical Definition**
The first rule prioritises native HTML semantics over ARIA because native elements come with built-in keyboard behaviour, focus management, and accessibility API mappings. ARIA should be used only when no native element provides the required semantics or when native support is incomplete .

**Beginner-Friendly Explanation**
The first rule of ARIA is simple: **do not use ARIA if native HTML can do the job**. A `<button>` is always better than a `<div role="button">`.

---

### Purposes

- **To prefer native semantics** whenever possible .
- **To reduce the risk of accessibility errors** from manual ARIA implementation.
- **To ensure keyboard and focus behaviour** is built in rather than hand-coded.

---

### Exceptions

The W3C identifies three exceptions where native HTML may not suffice :

1. **The feature is available in HTML but not implemented**, or its accessibility support is incomplete.
2. **Visual design constraints** rule out the native element.
3. **The feature is not currently available in HTML**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: First Rule in Practice

**Setup Guide**: Save as `first-rule.html`.

```html
<!-- FIRST RULE: use native HTML -->
<button type="button">Submit</button>

<!-- VIOLATION: re-purposing a div with ARIA -->
<!--
<div role="button" tabindex="0" onclick="submit()"
     onkeydown="if(event.key==='Enter') submit()">Submit</div>
-->
```

**Expected output**: The native button works immediately with keyboard, focus, and screen-reader support.

**Why this output occurs**: The native `<button>` already provides the semantics and behaviour required; adding ARIA to a `<div>` would duplicate and potentially break that behaviour .

---

### References

- W3C — Using ARIA: First Rule of ARIA Use — https://w3c.github.io/using-aria/#rule1
- MDN Web Docs — ARIA: Before using ARIA — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA

---

## Summary Table of ARIA Attributes

| Attribute | Category | Purpose |
|---|---|---|
| `role` | Identity | Defines the element’s semantic role. |
| `aria-label` | Naming | String-based accessible name. |
| `aria-labelledby` | Naming | Referenced accessible name. |
| `aria-describedby` | Description | Referenced accessible description. |
| `aria-hidden` | Visibility | Removes element from the accessibility tree. |
| `aria-live` | Live region | Controls announcement politeness. |
| `aria-atomic` | Live region | Controls full vs partial announcements. |
| `aria-busy` | Live region | Suppresses announcements during updates. |
| `aria-checked` | State | Checked/unchecked/mixed state. |
| `aria-disabled` | State | Disabled state. |
| `aria-expanded` | State | Expanded/collapsed state. |
| `aria-haspopup` | State | Popup type indication. |
| `aria-pressed` | State | Toggle-button state. |
| `aria-selected` | State | Selected state. |
| `aria-current` | State | Current-item indicator. |
| `aria-controls` | Relationship | Controlled-element relationship. |
| `aria-owns` | Relationship | Ownership relationship. |
| `aria-activedescendant` | Relationship | Active descendant within a composite widget. |
| `aria-autocomplete` | Property | Autocomplete behaviour. |
| `aria-multiline` | Property | Multi-line input indication. |
| `aria-multiselectable` | Property | Multi-select indication. |
| `aria-orientation` | Property | Widget orientation. |
| `aria-placeholder` | Property | Placeholder hint. |
| `aria-readonly` | State | Read-only state. |
| `aria-required` | State | Required state. |
| `aria-sort` | State | Sort direction. |
| `aria-valuemax` | Range | Maximum value. |
| `aria-valuemin` | Range | Minimum value. |
| `aria-valuenow` | Range | Current value. |
| `aria-valuetext` | Range | Human-readable value text. |

---

## Final Notes

- **ARIA is a supplement, not a replacement**: Native HTML is always the first choice .
- **ARIA does not add behaviour**: Keyboard support, focus management, and state updates must be implemented in JavaScript .
- **“No ARIA is better than bad ARIA”** : Incorrect ARIA can increase accessibility errors by 41% .
- **Test with real assistive technologies**: Automated checkers catch some issues, but screen-reader testing is essential.
- **Follow the four rules of ARIA**: Native first, preserve semantics, ensure keyboard operability, and never hide focusable content .

---

This cheat sheet has covered ARIA, the role attribute, naming and description attributes, live-region attributes, state attributes, relationship attributes, range attributes, best practices, pitfalls, and the first rule of ARIA — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate ARIA usage against the W3C WAI-ARIA specification and ARIA Authoring Practices Guide.