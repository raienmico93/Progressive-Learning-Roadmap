# ARIA Fundamentals: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Accessible Rich Internet Applications (ARIA) is a W3C specification that provides a set of attributes for supplementing HTML with additional semantics, enabling assistive technologies to understand custom widgets, dynamic content, and interactive components that native HTML cannot fully express.

**Technical Definition**

ARIA is defined by the W3C Accessible Rich Internet Applications specification (WAI-ARIA). It provides a framework of roles, states, and properties that can be applied to HTML elements to convey semantic meaning to assistive technologies. The specification defines a taxonomy of roles (abstract, widget, document structure, landmark, live region, and window roles), global and widget-specific states and properties, and the mechanisms by which user agents map these to platform accessibility APIs. The HTML Accessibility API Mappings (HTML-AAM) and the Core Accessibility API Mappings (Core-AAM) define how ARIA attributes and native HTML elements are exposed to assistive technology. ARIA is not a replacement for HTML — it is a supplement for cases where native semantics are insufficient.

**Beginner-Friendly Explanation**

HTML has a limited vocabulary. There's a `<button>`, a `<nav>`, a `<table>` — but there's no `<tabpanel>` or `<treeview>` or `<combobox>`. ARIA fills that gap. It lets you tell screen readers "this `<div>` is actually a tab panel" or "this `<span>` is a dialog." But the golden rule is: if HTML already has an element for what you're building, use it. ARIA is a patch, not a foundation.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Supplement, not replacement** | ARIA adds semantics where native HTML falls short |
| **Three categories** | Roles (what it is), States (transient conditions), Properties (essential characteristics) |
| **Accessibility tree** | ARIA attributes populate the browser's accessibility tree, which screen readers consume |
| **No visual effect** | ARIA attributes have no default visual presentation |
| **No behaviour** | ARIA does not add keyboard support or interactivity — you must implement that yourself |
| **First Rule of ARIA** | Native HTML should always be preferred over ARIA-patched elements |
| **Error-prone** | Incorrect ARIA is often worse than no ARIA at all |

---

### Prerequisites

- Basic familiarity with HTML document structure and elements
- Understanding of semantic HTML and why it matters
- Awareness of how screen readers interpret web content
- Basic knowledge of the DOM and accessibility tree
- Familiarity with form controls and interactive elements

---

### Related Programming Areas

- **Web Accessibility (A11y)** – ARIA is a core tool in the accessibility toolkit
- **WCAG Compliance** – ARIA supports Success Criterion 4.1.2 (Name, Role, Value)
- **Semantic HTML** – Native semantics are always preferred over ARIA
- **Screen Reader Testing** – ARIA support varies; testing with real AT is essential
- **Component Libraries** – ARIA is essential for custom widgets (tabs, modals, comboboxes)
- **HTML-AAM and Core-AAM** – Specifications that define how ARIA maps to platform APIs

---

## Core Concepts / Features

---

### 1. Accessible Rich Internet Applications (ARIA)

#### Definitions

**Core Definition**

ARIA is a W3C specification that defines a set of attributes for adding semantic meaning to HTML elements, enabling assistive technologies to understand custom widgets and dynamic content.

**Technical Definition**

ARIA (Accessible Rich Internet Applications) is defined by the W3C WAI-ARIA specification. It is a technical specification that provides a framework to enhance the accessibility of web content and web applications by adding roles, states, and properties to HTML elements. ARIA attributes are consumed by the browser's accessibility tree, which is then exposed to assistive technologies through platform accessibility APIs (e.g., MSAA, IAccessible2, UIA, ATK/AT-SPI, macOS Accessibility API). The W3C's WAI-ARIA Authoring Practices Guide (APG) provides patterns for implementing common widgets with ARIA. ARIA has no effect on visual presentation or behaviour — it only changes what assistive technology perceives.

**Beginner-Friendly Explanation**

ARIA is a set of attributes you add to HTML to give screen readers more information. For example, if you build a custom dropdown menu with `<div>` elements, screen readers have no idea what it is. Adding `role="listbox"` and `role="option"` tells them it's a dropdown. But ARIA doesn't make it work — you still have to add keyboard support yourself. ARIA just changes what screen readers announce.

#### Purposes

- To add semantic meaning to custom widgets that native HTML cannot express
- To communicate dynamic state changes to assistive technology
- To define relationships between elements (e.g., a tab and its panel)
- To provide accessible names and descriptions when native labels are insufficient
- To enable accessible interactive components (tabs, modals, comboboxes, trees)

#### The Three Categories of ARIA

| Category | Description | Examples |
|---|---|---|
| **Roles** | Define what an element is or does | `role="dialog"`, `role="tablist"`, `role="alert"` |
| **States** | Transient, dynamic characteristics | `aria-expanded`, `aria-checked`, `aria-disabled` |
| **Properties** | Essential, rarely changed characteristics | `aria-label`, `aria-labelledby`, `aria-required` |

**Syntax Rules**

- ARIA attributes are written as HTML attributes (`aria-*` and `role`)
- Roles are defined with the `role` attribute
- States and properties are defined with `aria-*` attributes
- ARIA attributes have no visual effect; CSS must be used for styling
- ARIA attributes have no behavioural effect; JavaScript must be used for interaction
- ARIA attributes must be valid according to the specification

**Constraints and Limitations**

- Incorrect ARIA can make accessibility worse than no ARIA
- ARIA support varies across screen readers and browsers
- ARIA does not add keyboard support, focus management, or behaviour
- Some ARIA roles require specific child roles or properties
- Automated testing cannot fully validate ARIA implementations

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Custom Widget with ARIA**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>ARIA Custom Widget</title>
</head>
<body>
    <!-- A custom dropdown built with divs and ARIA -->
    <div class="dropdown">
        <button id="dropdown-btn"
                aria-haspopup="listbox"
                aria-expanded="false"
                aria-labelledby="dropdown-label">
            <span id="dropdown-label">Choose a fruit</span>
        </button>
        <ul role="listbox" id="dropdown-list" hidden
            aria-labelledby="dropdown-label">
            <li role="option" aria-selected="true">Apple</li>
            <li role="option" aria-selected="false">Banana</li>
            <li role="option" aria-selected="false">Cherry</li>
        </ul>
    </div>

    <script>
        const btn = document.getElementById('dropdown-btn');
        const list = document.getElementById('dropdown-list');

        btn.addEventListener('click', () => {
            const expanded = btn.getAttribute('aria-expanded') === 'true';
            btn.setAttribute('aria-expanded', !expanded);
            list.hidden = expanded;
        });

        list.addEventListener('click', (event) => {
            if (event.target.role === 'option') {
                [...list.children].forEach(li =>
                    li.setAttribute('aria-selected', li === event.target));
                btn.querySelector('#dropdown-label').textContent = event.target.textContent;
                list.hidden = true;
                btn.setAttribute('aria-expanded', 'false');
                btn.focus();
            }
        });
    </script>
</body>
</html>
```

**Expected Output**

Screen readers announce: "Choose a fruit, button, collapsed, has popup listbox." After opening: "Choose a fruit, button, expanded." Each option is announced as "option, selected/not selected."

**Why This Output Occurs**

`aria-haspopup="listbox"` tells the screen reader the button opens a listbox. `aria-expanded` communicates the open/closed state. `role="listbox"` and `role="option"` define the dropdown and its options. `aria-selected` communicates the selection state.

#### Real-World Cases

**Case 1: Custom Dropdowns**

Design systems that build custom dropdowns use ARIA to communicate the listbox pattern.

**Case 2: Modal Dialogs**

Modals use `role="dialog"` and `aria-modal="true"` to announce the dialog and trap focus.

**Case 3: Tab Interfaces**

Tabs use `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected`, and `aria-controls`.

---

### 2. Roles

#### Definitions

**Core Definition**

ARIA roles declare what a custom element is or does, providing assistive technology with a semantic identity for elements that lack native meaning.

**Technical Definition**

ARIA roles are defined by the `role` attribute, which accepts one or more role tokens (the first valid token wins). The WAI-ARIA specification defines six categories of roles: abstract roles (never used by authors), widget roles (interactive controls), document structure roles (content organisation), landmark roles (page regions), live region roles (dynamic content), and window roles (dialogs, alerts). Each role defines the required and supported states and properties (inherited states and properties), as well as required context roles and required owned elements. The HTML-AAM specifies which roles are permitted on which HTML elements (e.g., `role="button"` is not permitted on `<a href>`).

**Beginner-Friendly Explanation**

A role tells the screen reader what an element *is*. If you build a button out of a `<div>`, it has no role by default — screen readers just see a generic div. Adding `role="button"` tells the screen reader "this is a button." But roles don't make it work — you still need to add keyboard support and focus management.

#### Purposes

- To declare the semantic identity of custom elements
- To communicate widget types (button, dialog, tab, combobox)
- To define document structure (article, heading, list)
- To identify landmarks (navigation, main, banner, contentinfo)
- To create live regions (alert, status, log)
- To define window roles (dialog, alertdialog)

#### Common ARIA Roles

| Category | Role | Description |
|---|---|---|
| **Widget** | `button` | Clickable button |
| **Widget** | `checkbox` | Checkbox control |
| **Widget** | `dialog` | Modal or non-modal dialog |
| **Widget** | `tablist` / `tab` / `tabpanel` | Tab interface |
| **Widget** | `listbox` / `option` | Dropdown list |
| **Widget** | `combobox` | Input with popup |
| **Widget** | `slider` | Range slider |
| **Widget** | `switch` | On/off toggle |
| **Landmark** | `banner` | Site header |
| **Landmark** | `navigation` | Navigation region |
| **Landmark** | `main` | Main content |
| **Landmark** | `contentinfo` | Site footer |
| **Landmark** | `complementary` | Sidebar |
| **Live Region** | `alert` | Assertive announcement |
| **Live Region** | `status` | Polite announcement |
| **Document** | `article` | Self-contained composition |
| **Document** | `heading` | Heading |
| **Document** | `list` / `listitem` | List and items |

**Syntax Rules**

- The `role` attribute accepts one or more space-separated role tokens
- The first valid, non-abstract role is used
- Roles must be permitted on the element (per HTML-AAM)
- Some roles require specific child roles (e.g., `listbox` requires `option`)
- Some roles require specific states or properties (e.g., `checkbox` requires `aria-checked`)
- Do not use abstract roles (`role="widget"`, `role="input"`)

**Constraints and Limitations**

- Incorrect role usage can make content less accessible
- Roles do not add behaviour — keyboard support must be implemented
- Some roles are not permitted on certain elements
- Role support varies across screen readers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Custom Button with Role**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Role Demo</title>
</head>
<body>
    <!-- INCORRECT: Native button is preferred -->
    <div role="button" tabindex="0" onclick="alert('Clicked')"
         onkeydown="if(event.key==='Enter'||event.key===' ')alert('Clicked')">
        Click Me
    </div>

    <!-- CORRECT: Native button -->
    <button type="button" onclick="alert('Clicked')">Click Me</button>
</body>
</html>
```

**Expected Output**

Both work with screen readers, but the native `<button>` is simpler, more reliable, and requires no extra code.

**Why This Output Occurs**

The `<div role="button">` requires `tabindex`, `onclick`, and `onkeydown` to replicate native behaviour. The native `<button>` provides all of this automatically.

---

**Example 2: Tab Interface with Roles**

```html
<div role="tablist" aria-label="Settings">
    <button role="tab" id="tab-1" aria-selected="true" aria-controls="panel-1">
        General
    </button>
    <button role="tab" id="tab-2" aria-selected="false" aria-controls="panel-2"
            tabindex="-1">
        Privacy
    </button>
</div>

<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
    <h2>General Settings</h2>
</div>
<div role="tabpanel" id="panel-2" aria-labelledby="tab-2" hidden>
    <h2>Privacy Settings</h2>
</div>
```

**Expected Output**

Screen readers announce: "Settings, tab list. General, tab, selected. Privacy, tab." When Privacy is activated, its panel is shown and the tab is selected.

**Why This Output Occurs**

`role="tablist"`, `role="tab"`, and `role="tabpanel"` define the tab interface. `aria-selected` communicates the active tab. `aria-controls` links each tab to its panel. `aria-labelledby` links each panel back to its tab.

#### Real-World Cases

**Case 1: Design Systems**

Material Design, Bootstrap, and other design systems use ARIA roles for custom widgets.

**Case 2: Single-Page Applications**

SPAs use ARIA roles to communicate routing and view changes.

**Case 3: Interactive Dashboards**

Dashboards use ARIA roles for tabs, dialogs, sliders, and other widgets.

---

### 3. States

#### Definitions

**Core Definition**

ARIA states are transient, dynamic characteristics of elements that change in response to user interaction, such as whether a menu is expanded, a checkbox is checked, or a control is disabled.

**Technical Definition**

ARIA states are a subset of ARIA attributes that describe the current condition of an element. They are defined with `aria-*` attributes and are typically updated dynamically by JavaScript. States differ from properties in that states are expected to change frequently (e.g., `aria-expanded` toggles when a menu opens and closes), while properties are relatively stable (e.g., `aria-label` rarely changes). The WAI-ARIA specification defines states and properties together as "ARIA attributes", distinguished primarily by convention and frequency of change. States must be updated to reflect the current condition; a stale state (e.g., `aria-expanded="true"` when the menu is closed) confuses assistive technology users.

**Beginner-Friendly Explanation**

States are the "right now" conditions of a widget. Is the menu open or closed? Is the checkbox checked? Is the button pressed? These change as the user interacts, so you need JavaScript to update them. If you forget to update a state, screen reader users get wrong information.

#### Purposes

- To communicate the current condition of interactive elements
- To reflect dynamic changes (expanded, checked, selected, pressed)
- To enable screen readers to announce state changes
- To indicate availability (disabled, busy)
- To support accessible custom widgets

#### Common ARIA States

| State | Values | Description |
|---|---|---|
| `aria-expanded` | `true`, `false` | Expanded/collapsed (menus, accordions) |
| `aria-checked` | `true`, `false`, `mixed` | Checked state (checkboxes, radios) |
| `aria-selected` | `true`, `false` | Selected state (tabs, options) |
| `aria-pressed` | `true`, `false`, `mixed` | Toggle button pressed state |
| `aria-disabled` | `true`, `false` | Disabled state |
| `aria-hidden` | `true`, `false` | Hidden from assistive technology |
| `aria-invalid` | `true`, `false`, `spelling`, `grammar` | Invalid input |
| `aria-busy` | `true`, `false` | Content is being updated |
| `aria-current` | `page`, `step`, `location`, `date`, `time`, `true`, `false` | Current item in a set |
| `aria-grabbed` | `true`, `false` | Deprecated (drag-and-drop) |

**Syntax Rules**

- States are written as `aria-*` attributes
- Values are typically strings (`"true"`, `"false"`, `"mixed"`)
- States must be updated dynamically with JavaScript
- States must be valid for the element's role
- Stale states confuse screen reader users; always update

**Constraints and Limitations**

- States have no visual effect; CSS must style the visual state
- States have no behavioural effect; JavaScript must implement the behaviour
- Some states require specific roles
- Support varies across screen readers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Accordion with `aria-expanded`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Accordion Demo</title>
</head>
<body>
    <h3>
        <button aria-expanded="false" aria-controls="section-1">
            Section 1
        </button>
    </h3>
    <div id="section-1" hidden>
        <p>Content of section 1.</p>
    </div>

    <script>
        const button = document.querySelector('[aria-controls="section-1"]');
        const panel = document.getElementById('section-1');

        button.addEventListener('click', () => {
            const expanded = button.getAttribute('aria-expanded') === 'true';
            button.setAttribute('aria-expanded', !expanded);
            panel.hidden = expanded;
        });
    </script>
</body>
</html>
```

**Expected Output**

Screen readers announce: "Section 1, button, collapsed." After clicking: "Section 1, button, expanded." The panel is shown.

**Why This Output Occurs**

`aria-expanded` communicates the state. `aria-controls` links the button to the panel. The `hidden` attribute hides the panel visually and from assistive technology.

---

**Example 2: Toggle Button with `aria-pressed`**

```html
<button type="button" aria-pressed="false" id="toggle">
    Bold
</button>

<script>
    const toggle = document.getElementById('toggle');

    toggle.addEventListener('click', () => {
        const pressed = toggle.getAttribute('aria-pressed') === 'true';
        toggle.setAttribute('aria-pressed', !pressed);
        toggle.style.fontWeight = pressed ? 'normal' : 'bold';
    });
</script>
```

**Expected Output**

Screen readers announce: "Bold, toggle button, not pressed." After clicking: "Bold, toggle button, pressed."

**Why This Output Occurs**

`aria-pressed` communicates the toggle state. The visual style is updated separately with CSS.

#### Real-World Cases

**Case 1: Navigation Menus**

Dropdown menus use `aria-expanded` to communicate open/closed state.

**Case 2: Form Validation**

Forms use `aria-invalid` and `aria-describedby` to communicate errors.

**Case 3: Loading States**

Applications use `aria-busy` to indicate content is loading.

---

### 4. Properties

#### Definitions

**Core Definition**

ARIA properties are essential characteristics of elements that are rarely changed by interaction, such as an element's accessible name, description, or required status.

**Technical Definition**

ARIA properties are `aria-*` attributes that describe stable characteristics of an element. They are distinguished from states primarily by frequency of change. Properties include `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-required`, `aria-placeholder`, `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext`, `aria-orientation`, `aria-multiselectable`, `aria-autocomplete`, and many others. Properties are defined by the WAI-ARIA specification and mapped to platform accessibility APIs by the Core-AAM. Some properties are global (applicable to all roles), while others are role-specific.

**Beginner-Friendly Explanation**

Properties are the stable characteristics of an element — its name, its description, whether it's required, its value range. Unlike states, they don't change frequently. For example, `aria-label` gives an element its name; `aria-required` says a field is required; `aria-valuemin` and `aria-valuemax` define a slider's range.

#### Purposes

- To provide accessible names when native labels are insufficient
- To link descriptions and error messages to controls
- To define value ranges for sliders and progress bars
- To indicate required fields
- To define relationships between elements
- To communicate selection modes (single vs. multi-select)

#### Common ARIA Properties

| Property | Description |
|---|---|
| `aria-label` | Defines an accessible name (string) |
| `aria-labelledby` | References element(s) that label this element |
| `aria-describedby` | References element(s) that describe this element |
| `aria-required` | Indicates a required field |
| `aria-placeholder` | Defines a hint for input |
| `aria-valuemin` | Minimum value for a range |
| `aria-valuemax` | Maximum value for a range |
| `aria-valuenow` | Current value for a range |
| `aria-valuetext` | Human-readable value text |
| `aria-orientation` | Orientation (horizontal/vertical) |
| `aria-multiselectable` | Allows multiple selections |
| `aria-autocomplete` | Autocomplete behaviour |
| `aria-controls` | References elements controlled by this one |
| `aria-owns` | Defines a parent-child relationship |
| `aria-activedescendant` | Active descendant in a composite widget |
| `aria-haspopup` | Indicates a popup will appear |
| `aria-modal` | Indicates a modal dialog |
| `aria-live` | Live region politeness |
| `aria-atomic` | Whether the whole region is announced |
| `aria-relevant` | What changes are announced |

**Syntax Rules**

- Properties are written as `aria-*` attributes
- `aria-label` takes a string value
- `aria-labelledby` and `aria-describedby` take space-separated ID references
- `aria-valuemin`, `aria-valuemax`, and `aria-valuenow` take numeric values
- Properties must be valid for the element's role
- `aria-labelledby` takes precedence over `aria-label`, which takes precedence over native labels

**Constraints and Limitations**

- `aria-label` is not visible; use it only when a visible label is not possible
- `aria-labelledby` can reference multiple elements (concatenated)
- `aria-describedby` provides a description, not a name
- Properties have no visual or behavioural effect

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Icon Button with `aria-label`**

```html
<button type="button" aria-label="Close dialog">
    <svg aria-hidden="true" width="16" height="16">
        <!-- X icon -->
    </svg>
</button>
```

**Expected Output**

Screen readers announce: "Close dialog, button." The SVG is hidden from assistive technology.

**Why This Output Occurs**

`aria-label` provides the accessible name. `aria-hidden="true"` on the SVG prevents redundant announcement.

---

**Example 2: Slider with Value Properties**

```html
<div role="slider"
     aria-label="Volume"
     aria-valuemin="0"
     aria-valuemax="100"
     aria-valuenow="50"
     aria-valuetext="50%"
     tabindex="0"
     id="volume-slider">
</div>

<script>
    const slider = document.getElementById('volume-slider');

    slider.addEventListener('keydown', (event) => {
        let value = parseInt(slider.getAttribute('aria-valuenow'));
        if (event.key === 'ArrowRight' || event.key === 'ArrowUp') {
            value = Math.min(100, value + 5);
        } else if (event.key === 'ArrowLeft' || event.key === 'ArrowDown') {
            value = Math.max(0, value - 5);
        } else {
            return;
        }
        event.preventDefault();
        slider.setAttribute('aria-valuenow', value);
        slider.setAttribute('aria-valuetext', value + '%');
    });
</script>
```

**Expected Output**

Screen readers announce: "Volume, slider, 50%." Arrow keys change the value and the announcement updates.

**Why This Output Occurs**

`aria-valuemin`, `aria-valuemax`, and `aria-valuenow` define the range. `aria-valuetext` provides a human-readable value.

#### Real-World Cases

**Case 1: Form Fields**

Forms use `aria-describedby` for hints and errors, `aria-required` for required fields.

**Case 2: Sliders**

Sliders use `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, and `aria-valuetext`.

**Case 3: Comboboxes**

Comboboxes use `aria-expanded`, `aria-controls`, `aria-autocomplete`, and `aria-activedescendant`.

---

### 5. The First Rule of ARIA

#### Definitions

**Core Definition**

The First Rule of ARIA states: if you can use a native HTML element or attribute with the semantics and behaviour you require already built in, use it instead of repurposing an element and adding ARIA.

**Technical Definition**

The W3C WAI-ARIA specification defines five rules for ARIA use. The first rule states: "If you can use a native HTML element or attribute with the semantics and behaviour you require already built in, instead of re-purposing an element and adding an ARIA role, state or property to make it accessible, then do so." This rule reflects the fact that native HTML elements have built-in keyboard support, focus management, and accessibility semantics that ARIA-patched elements must replicate manually. The second rule states: "Do not change native semantics, unless you really have to." The third rule states: "All interactive ARIA controls must be usable with the keyboard." The fourth rule states: "Do not use role='presentation' or aria-hidden='true' on a focusable element." The fifth rule states: "All interactive elements must have an accessible name."

**Beginner-Friendly Explanation**

The First Rule of ARIA is simple: use HTML. If you need a button, use `<button>`. If you need a link, use `<a>`. If you need a checkbox, use `<input type="checkbox">`. Don't build a fake button out of a `<div>` and then add `role="button"`. Native elements come with keyboard support, focus management, and screen reader semantics for free. ARIA is a last resort.

#### Purposes

- To ensure the most robust, accessible implementation
- To reduce code complexity and maintenance burden
- To leverage built-in keyboard support and focus management
- To avoid the pitfalls of incorrect ARIA usage
- To comply with the WAI-ARIA specification's rules

#### Common Violations of the First Rule

| Violation | Correct Alternative |
|---|---|
| `<div role="button">` | `<button>` |
| `<div role="link">` | `<a href>` |
| `<div role="checkbox">` | `<input type="checkbox">` |
| `<div role="radio">` | `<input type="radio">` |
| `<div role="textbox">` | `<input type="text">` or `<textarea>` |
| `<div role="list">` | `<ul>` or `<ol>` |
| `<div role="listitem">` | `<li>` |
| `<div role="heading">` | `<h1>`–`<h6>` |
| `<div role="navigation">` | `<nav>` |
| `<div role="main">` | `<main>` |
| `<span role="checkbox">` | `<input type="checkbox">` |

**Syntax Rules**

- Always prefer native HTML elements for their intended purpose
- Use ARIA only when native HTML cannot express the required semantics
- Do not add redundant roles to native elements (e.g., `role="button"` on `<button>`)
- Do not change native semantics unless absolutely necessary
- All interactive elements must be keyboard-accessible

**Constraints and Limitations**

- Some widgets (tabs, comboboxes, trees) have no native HTML equivalent
- In these cases, ARIA is required, but keyboard support must be implemented manually
- The ARIA Authoring Practices Guide (APG) provides patterns for these cases

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Native vs. ARIA-Patched**

```html
<!-- INCORRECT: ARIA-patched div -->
<div role="button" tabindex="0"
     onclick="submitForm()"
     onkeydown="if(event.key==='Enter'||event.key===' ')submitForm()">
    Submit
</div>

<!-- CORRECT: Native button -->
<button type="button" onclick="submitForm()">Submit</button>
```

**Expected Output**

Both are announced as "Submit, button," but the native button is simpler, more reliable, and requires no extra code.

**Why This Output Occurs**

The native `<button>` has built-in keyboard support (Enter and Space), focus management, and screen reader semantics. The ARIA-patched div must replicate all of this manually.

---

**Example 2: Redundant ARIA**

```html
<!-- INCORRECT: Redundant role on native element -->
<button role="button">Submit</button>
<nav role="navigation">...</nav>
<main role="main">...</main>

<!-- CORRECT: No redundant ARIA -->
<button>Submit</button>
<nav>...</nav>
<main>...</main>
```

**Expected Output**

Both work, but the redundant ARIA adds no value and can cause screen reader confusion.

**Why This Output Occurs**

Native elements already have implicit roles. Adding redundant roles is unnecessary and can cause some screen readers to announce the role twice.

#### Real-World Cases

**Case 1: Design Systems**

Well-designed component libraries use native elements whenever possible and ARIA only for custom widgets.

**Case 2: Framework Components**

React, Vue, and Angular components often use native HTML elements internally.

**Case 3: Accessibility Audits**

Automated tools like axe-core flag violations of the First Rule of ARIA.

---

### 6. Avoiding Unnecessary ARIA

#### Definitions

**Core Definition**

Avoiding unnecessary ARIA means not adding ARIA roles, states, or properties to elements that already have the correct native semantics, to prevent code bloat and potential screen reader confusion.

**Technical Definition**

The WAI-ARIA specification's second rule states: "Do not change native semantics, unless you really have to." Adding redundant ARIA (e.g., `role="button"` to a `<button>`, `role="navigation"` to a `<nav>`, `role="main"` to a `<main>`) creates no benefit and can cause problems. Some screen readers may announce the role twice, or the redundant ARIA may conflict with the native semantics. Additionally, ARIA attributes that are not valid for an element's role are ignored or cause errors. Automated accessibility tools flag redundant ARIA as a warning.

**Beginner-Friendly Explanation**

If an element already has the right meaning, don't add ARIA to it. A `<button>` is already a button — adding `role="button"` is redundant and can confuse screen readers. The same goes for `<nav role="navigation">` and `<main role="main">`. Only use ARIA when HTML doesn't have what you need.

#### Purposes

- To prevent code bloat and improve maintainability
- To avoid screen reader confusion from duplicate announcements
- To comply with the WAI-ARIA specification's rules
- To reduce the risk of ARIA errors
- To keep HTML clean and semantic

#### Common Redundant ARIA

| Redundant ARIA | Correct HTML |
|---|---|
| `<button role="button">` | `<button>` |
| `<a href role="link">` | `<a href>` |
| `<nav role="navigation">` | `<nav>` |
| `<main role="main">` | `<main>` |
| `<header role="banner">` | `<header>` (top-level) |
| `<footer role="contentinfo">` | `<footer>` (top-level) |
| `<aside role="complementary">` | `<aside>` |
| `<h1 role="heading" aria-level="1">` | `<h1>` |
| `<ul role="list">` | `<ul>` |
| `<li role="listitem">` | `<li>` |
| `<input type="checkbox" role="checkbox">` | `<input type="checkbox">` |

**Syntax Rules**

- Do not add `role` to elements that already have the correct implicit role
- Do not add `aria-*` attributes that duplicate native attributes
- Use the HTML-AAM to determine implicit roles
- Use automated tools to detect redundant ARIA
- When in doubt, leave it out

**Constraints and Limitations**

- Some redundant ARIA is harmless but still unnecessary
- Some ARIA attributes are required for custom widgets (not redundant)
- The line between "redundant" and "supplementary" can be subtle

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Redundant vs. Necessary ARIA**

```html
<!-- REDUNDANT: Native elements already have these roles -->
<button role="button">Submit</button>
<nav role="navigation">...</nav>
<main role="main">...</main>
<ul role="list">
    <li role="listitem">Item</li>
</ul>

<!-- NECESSARY: Custom widgets need ARIA -->
<div role="tablist">
    <button role="tab" aria-selected="true">Tab 1</button>
</div>
<div role="tabpanel">...</div>
```

**Expected Output**

The redundant ARIA has no effect (or causes duplicate announcements). The necessary ARIA enables the tab interface.

**Why This Output Occurs**

Native elements have implicit roles. Adding redundant roles is unnecessary. Custom widgets (tabs) have no native equivalent, so ARIA is required.

---

**Example 2: Using `role="list"` When `list-style: none` Removes List Semantics**

```html
<!-- Some screen readers remove list semantics when list-style: none is applied -->
<ul style="list-style: none;" role="list">
    <li>Item 1</li>
    <li>Item 2</li>
</ul>
```

**Expected Output**

The `role="list"` explicitly preserves the list semantics when `list-style: none` removes them in some screen readers (e.g., VoiceOver on Safari).

**Why This Output Occurs**

This is one of the few cases where adding a redundant role is justified: it compensates for a browser/screen reader behaviour that removes the implicit role.

#### Real-World Cases

**Case 1: Navigation Menus**

Navigation menus often have `role="navigation"` added redundantly to `<nav>` elements.

**Case 2: Buttons**

Buttons often have `role="button"` added redundantly to `<button>` elements.

**Case 3: Landmarks**

Landmarks often have redundant ARIA roles added to semantic HTML elements.

---

### 7. Choosing the Right ARIA Approach

#### Definitions

**Core Definition**

Choosing the right ARIA approach means determining when ARIA is necessary, which roles, states, and properties to use, and how to implement them without violating the First Rule of ARIA.

**Technical Definition**

The choice depends on whether native HTML can express the required semantics. If it can, use HTML. If it cannot (e.g., for tabs, comboboxes, trees, sliders, custom dialogs), use ARIA following the WAI-ARIA Authoring Practices Guide. Always implement keyboard support, focus management, and state updates. Test with real assistive technology.

#### Decision Guide

| Requirement | Native HTML? | ARIA? |
|---|---|---|
| Button | `<button>` | No ARIA needed |
| Link | `<a href>` | No ARIA needed |
| Checkbox | `<input type="checkbox">` | No ARIA needed |
| Radio button | `<input type="radio">` | No ARIA needed |
| Text input | `<input type="text">` | No ARIA needed |
| Dropdown | `<select>` | No ARIA needed |
| Tabs | No native equivalent | `role="tablist"`, `role="tab"`, `role="tabpanel"` |
| Accordion | `<details>`/`<summary>` | Or ARIA `aria-expanded` |
| Modal dialog | `<dialog>` | Or ARIA `role="dialog"` |
| Combobox | `<datalist>` (limited) | ARIA `role="combobox"` |
| Tree view | No native equivalent | ARIA `role="tree"`, `role="treeitem"` |
| Slider | `<input type="range">` | Or ARIA `role="slider"` |
| Tooltip | `title` (limited) | ARIA `role="tooltip"` |
| Live region | No native equivalent | `aria-live`, `role="status"`, `role="alert"` |

---

## References

- W3C – WAI-ARIA Overview – https://www.w3.org/WAI/standards-guidelines/aria/
- W3C – Accessible Rich Internet Applications (WAI-ARIA) 1.2 – https://www.w3.org/TR/wai-aria-1.2/
- W3C – WAI-ARIA Authoring Practices Guide (APG) – https://www.w3.org/WAI/ARIA/apg/
- W3C – ARIA in HTML – https://www.w3.org/TR/html-aria/
- W3C – Using ARIA: Roles, states, and properties – https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/
- W3C – HTML Accessibility API Mappings (HTML-AAM) – https://w3c.github.io/html-aam/
- W3C – Core Accessibility API Mappings (Core-AAM) – https://www.w3.org/TR/core-aam-1.2/
- MDN Web Docs – ARIA – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA
- MDN Web Docs – ARIA roles – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles
- MDN Web Docs – ARIA states and properties – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes
- MDN Web Docs – ARIA: First Rule of ARIA – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques
- WebAIM – Introduction to ARIA – https://webaim.org/techniques/aria/
- WebAIM – ARIA: The Good, the Bad, and the Ugly – https://webaim.org/blog/aria-good-bad-ugly/
- Deque – ARIA Spec – https://www.deque.com/aria/
- The A11Y Project – ARIA – https://www.a11yproject.com/posts/aria/
- W3C – WCAG 2.1 Understanding Success Criterion 4.1.2: Name, Role, Value – https://www.w3.org/WAI/WCAG21/Understanding/name-role-value.html