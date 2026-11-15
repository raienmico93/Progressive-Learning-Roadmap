# ARIA — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** ARIA (Accessible Rich Internet Applications) is a W3C specification that provides a set of HTML attributes to add semantic meaning to web content and custom user interface controls, enabling assistive technologies to understand and interact with them.

**Technical Definition:** WAI-ARIA defines an ontology of **roles** (what an element is), **states** (its dynamic condition), and **properties** (supplemental descriptive information) that authors apply to markup to expose user interface behavior and structural information to platform accessibility APIs. ARIA changes only how assistive technologies perceive an element — it does not add behavior, keyboard functionality, or focus management. It complements semantic HTML rather than replacing it, and should be used only when native HTML cannot express the required semantics.

**Beginner-Friendly Explanation:** ARIA is a set of special HTML attributes that tell screen readers what something is. If you build a custom dropdown from `<div>`s, ARIA lets you say "this is actually a menu" and "this option is selected." But ARIA only changes what is *announced* — it does not make anything clickable, focusable, or keyboard-operable. That is why the first rule of ARIA is: do not use ARIA if native HTML can do the job.

### Key Characteristics

- **Semantics-only:** ARIA changes the accessibility tree, not the DOM, behavior, or appearance.
- **Three declaration types:** Roles (what it is), states (dynamic conditions like expanded/checked), and properties (supplemental information like labels and relationships).
- **Native-first:** The first rule of ARIA is to use native HTML elements and attributes whenever they provide the required semantics and behavior.
- **Misuse is worse than omission:** Pages with ARIA averaged 59.1 detectable errors versus 42 on pages without ARIA, per the 2026 WebAIM Million report.
- **Keyboard is your responsibility:** ARIA declares semantics but does not add keyboard functionality — all interactive ARIA controls must be keyboard operable.

### Prerequisites

- Solid understanding of semantic HTML and the accessibility tree.
- Familiarity with screen readers (NVDA, JAWS, VoiceOver) and keyboard navigation.
- Awareness of WCAG success criteria (especially 4.1.2 Name, Role, Value).
- A code editor with ARIA linting (`eslint-plugin-jsx-a11y` for React).

### Related Programming Areas

- **Semantic HTML** — the foundation ARIA supplements.
- **Accessibility testing** — keyboard testing, screen reader testing, automated audits.
- **React accessibility** — applying ARIA correctly in JSX components.
- **WCAG conformance** — ARIA is required for custom widgets to meet 4.1.2.

### Core Concepts / Features

---

## 1. ARIA (WAI-ARIA)

### Definitions

**Core Definition:** WAI-ARIA is the W3C specification that defines roles, states, and properties for making dynamic web content and custom UI controls accessible.

**Technical Definition:** WAI-ARIA (Accessible Rich Internet Applications) is a technical specification published by the W3C as part of the Web Accessibility Initiative. It defines a framework of HTML attributes that tell assistive technologies about the roles, states, and properties of user interface elements. ARIA 1.3 is the current working draft, adding `header` and `sectionfooter` roles and the `aria-Notify` API, and preventing `aria-hidden="true"` on document root elements.

**Beginner-Friendly Explanation:** ARIA is the W3C's official system for labeling custom UI controls so screen readers can understand them. It fills the gaps that HTML left behind when developers started building complex widgets from generic `<div>`s.

### Purposes

- To provide semantics for custom widgets that native HTML cannot express (tabs, trees, carousels).
- To expose dynamic state changes (expanded, selected, checked) to assistive technology.
- To define page structure and landmarks for efficient screen reader navigation.
- To announce dynamic content updates via live regions.
- To enhance keyboard accessibility where native behavior is insufficient.

### Syntax Rules and Structure

ARIA attributes follow the `aria-*` naming convention and are applied as HTML attributes:

```html
<element role="role-name" aria-state="value" aria-property="value">
```

**Component Breakdown:**

| Category | Prefix | Purpose | Example |
|----------|--------|---------|---------|
| Role | `role` | Defines what the element is. | `role="button"` |
| State | `aria-*` | Dynamic condition that changes. | `aria-expanded="true"` |
| Property | `aria-*` | Supplemental information (often static). | `aria-label="Search"` |

**Constraints and Limitations:**

- ARIA does not change behavior — a `role="button"` on a `<div>` does not make it clickable or keyboard-operable.
- ARIA attributes must be valid for the element and role — invalid combinations are flagged by conformance tools.
- ARIA should be used only when native HTML cannot provide the required semantics.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Native HTML vs. ARIA Custom Button

```html
<!-- ✅ NATIVE: full accessibility for free -->
<button type="button" onclick="submitForm()">
  Submit
</button>

<!-- ❌ ARIA-ONLY: announces "button" but does nothing else -->
<div role="button" onclick="submitForm()">
  Submit
</div>
```

**Expected Output:** The native `<button>` is focusable, activates with Enter/Space, and is announced as "Submit, button." The `<div role="button">` is announced as a button but cannot be focused or activated by keyboard — it is a promise without functionality.

**Why this matters:** Fixing the ARIA version requires manually adding `tabindex="0"`, `onkeydown` handlers for Enter and Space, and focus styles — three extra steps that native `<button>` provides for free.

### Real-World Cases

- **Custom dropdowns and comboboxes:** Native `<select>` cannot be styled sufficiently; ARIA roles and states describe the custom widget.
- **Tab interfaces:** `role="tablist"`, `role="tab"`, `role="tabpanel"` with `aria-selected`.
- **Modal dialogs:** `role="dialog"`, `aria-modal="true"`, and focus management.

### References

- W3C — WAI-ARIA 1.3 Specification — https://www.w3.org/TR/wai-aria-1.3/
- MDN — WAI-ARIA Basics — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/WAI-ARIA_basics
- Level Access — WAI-ARIA Guidance: Best Practices — https://www.levelaccess.com/blog/wai-aria-guidance-best-practices-for-accessible-web-interfaces/

---

## 2. ARIA Roles

### Definitions

**Core Definition:** An ARIA role defines what an element is — its type or purpose in the user interface — so assistive technologies can announce it appropriately.

**Technical Definition:** Roles are defined by the `role` attribute and fall into six categories: **Abstract** (foundation roles not for author use), **Widget** (interactive controls like button, slider, tab), **Document Structure** (content organization like article, list, table), **Landmark** (page regions like banner, navigation, main), **Live Region** (dynamic content areas like alert, log, status), and **Window** (browser or application windows like dialog, alertdialog).

**Beginner-Friendly Explanation:** A role tells the screen reader "this thing is a button" or "this is a navigation menu." It is the most fundamental ARIA attribute — without a role, a custom widget has no identity.

### Purposes

- To define the type of custom widget for assistive technology.
- To expose page structure and landmarks for navigation.
- To identify live regions and document structures.
- To provide the semantic foundation for associated states and properties.

### Syntax Rules and Structure

```html
<element role="role-name">
```

**Role categories and common roles:**

| Category | Examples | Purpose |
|----------|----------|---------|
| **Widget** | `button`, `checkbox`, `slider`, `tab`, `tablist`, `tabpanel`, `combobox`, `dialog` | Interactive controls. |
| **Landmark** | `banner`, `navigation`, `main`, `complementary`, `contentinfo`, `search`, `form` | Page regions. |
| **Document structure** | `article`, `list`, `listitem`, `table`, `heading` | Content organization. |
| **Live region** | `alert`, `log`, `status`, `timer`, `marquee` | Dynamic content areas. |
| **Window** | `dialog`, `alertdialog` | Windows and dialogs. |

**Constraints and Limitations:**

- Abstract roles (e.g., `widget`, `landmark`, `input`) must not be used by authors — they are for the specification only.
- A `role` must be valid for the element it is applied to — do not override native semantics without strong justification.
- Roles do not add behavior — keyboard interaction must be implemented separately.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Tab Interface

```html
<div role="tablist" aria-label="Account settings">
  <button role="tab" aria-selected="true" aria-controls="panel-1" id="tab-1">
    Profile
  </button>
  <button role="tab" aria-selected="false" aria-controls="panel-2" id="tab-2" tabindex="-1">
    Security
  </button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
  <p>Profile settings...</p>
</div>
<div role="tabpanel" id="panel-2" aria-labelledby="tab-2" hidden>
  <p>Security settings...</p>
</div>
```

**Expected Output:** Screen readers announce "Account settings, tablist" and "Profile, tab, selected." Arrow-key navigation should be implemented in JavaScript to move between tabs.

### Real-World Cases

- **Design systems:** ARIA roles define the contract for custom components.
- **Data grids:** `role="grid"`, `role="row"`, `role="gridcell"` for spreadsheet-like interfaces.
- **Menus:** `role="menu"`, `role="menuitem"` for custom dropdown menus.

### References

- W3C — ARIA Roles — https://www.w3.org/TR/wai-aria-1.3/#role_definitions
- MDN — ARIA Roles — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles

---

## 3. ARIA States

### Definitions

**Core Definition:** ARIA states describe the current, dynamic condition of an element — such as whether it is expanded, selected, checked, or disabled.

**Technical Definition:** States are `aria-*` attributes that can change throughout the lifecycle of an application, typically updated programmatically via JavaScript. They differ from properties, which generally do not change. Common states include `aria-expanded`, `aria-selected`, `aria-checked`, `aria-disabled`, `aria-pressed`, `aria-hidden`, `aria-invalid`, and `aria-busy`.

**Beginner-Friendly Explanation:** A state is the current condition of a widget — is the dropdown open or closed? Is the checkbox checked? Is the tab selected? States change as the user interacts.

### Purposes

- To communicate the current condition of a widget to assistive technology.
- To keep the screen reader's understanding in sync with the visual UI.
- To enable correct announcements of toggles, selections, and disabled states.
- To support dynamic widgets like accordions, tabs, and checkboxes.

### Syntax Rules and Structure

```html
<element aria-state="value">
```

**Common states:**

| State | Values | Used on | Meaning |
|-------|--------|---------|---------|
| `aria-expanded` | `true` / `false` | Buttons, disclosure widgets | Expanded or collapsed. |
| `aria-selected` | `true` / `false` | Tabs, options, grid cells | Selected or not. |
| `aria-checked` | `true` / `false` / `mixed` | Checkboxes, switches, radios | Checked, unchecked, or indeterminate. |
| `aria-disabled` | `true` / `false` | Interactive elements | Disabled or enabled. |
| `aria-pressed` | `true` / `false` / `mixed` | Toggle buttons | Pressed or not. |
| `aria-hidden` | `true` / `false` | Any element | Hidden from assistive tech. |
| `aria-invalid` | `true` / `false` / `grammar` / `spelling` | Form controls | Invalid input. |
| `aria-busy` | `true` / `false` | Live regions | Content is being updated. |

**Constraints and Limitations:**

- States must be kept in sync with the UI — static `aria-expanded="false"` on a button that does open is a lie.
- Do not use `aria-hidden="true"` on focusable elements — this creates a trap where keyboard users can focus something screen reader users cannot hear.
- `aria-checked="mixed"` is for tri-state checkboxes only.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Toggle Button with `aria-pressed`

```jsx
function ToggleButton({ isPressed, onToggle, children }) {
  return (
    <button
      type="button"
      aria-pressed={isPressed}   // State kept in sync with React state
      onClick={onToggle}
    >
      {children}
    </button>
  );
}
```

**Expected Output:** Screen readers announce "Toggle button, pressed" or "not pressed," matching the visual state.

#### Example 2: Accordion with `aria-expanded`

```html
<h3>
  <button aria-expanded="false" aria-controls="section-1">
    Section title
  </button>
</h3>
<div id="section-1" hidden>
  <p>Content...</p>
</div>
```

**Expected Output:** When the button is activated and `aria-expanded` becomes `"true"`, the screen reader announces "expanded" and the controlled region becomes visible.

### Real-World Cases

- **Accordions:** `aria-expanded` on the trigger button.
- **Checkboxes:** `aria-checked` on custom checkbox widgets.
- **Tabs:** `aria-selected` on tab buttons.
- **Disabled controls:** `aria-disabled` when native `disabled` is not used.

### References

- MDN — ARIA States and Properties — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes
- W3C — Definitions of States and Properties — https://www.w3.org/TR/wai-aria-1.3/#state_prop_def

---

## 4. ARIA Properties

### Definitions

**Core Definition:** ARIA properties are `aria-*` attributes that provide supplemental, generally static information about an element — such as its accessible name, description, relationships, or required status.

**Technical Definition:** Properties differ from states in that they typically do not change during the lifecycle of an application. They include naming properties (`aria-label`, `aria-labelledby`), descriptive properties (`aria-describedby`), relationship properties (`aria-controls`, `aria-owns`, `aria-activedescendant`), and status properties (`aria-required`, `aria-readonly`). Some properties, like `aria-required`, can change but are classified as properties because they describe an attribute of the element rather than its interactive state.

**Beginner-Friendly Explanation:** A property tells the screen reader something about an element that does not usually change — like its name, what it is required to do, or what other element it controls.

### Purposes

- To provide accessible names and descriptions for controls.
- To define relationships between elements (controller/controlled, owner/owned).
- To communicate required, read-only, and other status information.
- To associate error messages and instructions with form fields.

### Syntax Rules and Structure

```html
<element aria-property="value">
```

**Common properties:**

| Property | Purpose | Example |
|----------|---------|---------|
| `aria-label` | Defines an accessible name as a string. | `aria-label="Search"` |
| `aria-labelledby` | References another element as the name. | `aria-labelledby="heading-id"` |
| `aria-describedby` | References descriptive text. | `aria-describedby="hint-id"` |
| `aria-controls` | Identifies the element(s) this element controls. | `aria-controls="panel-id"` |
| `aria-owns` | Establishes a parent-child relationship not in the DOM. | `aria-owns="item-id"` |
| `aria-activedescendant` | Identifies the active descendant of a composite widget. | `aria-activedescendant="option-2"` |
| `aria-required` | Indicates a required field. | `aria-required="true"` |
| `aria-readonly` | Indicates a read-only field. | `aria-readonly="true"` |

**Constraints and Limitations:**

- Do not use `aria-label` on elements where naming is prohibited (e.g., `<div>` without a role).
- `aria-labelledby` takes precedence over `aria-label` when both are present — do not use both on the same element.
- Relationship properties reference IDs — those IDs must exist and be accessible.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Required and Read-Only Fields

```html
<label for="email">Email</label>
<input id="email" type="email" aria-required="true" aria-describedby="email-hint">
<p id="email-hint">We'll never share your email.</p>

<label for="username">Username</label>
<input id="username" type="text" aria-readonly="true" value="alice">
```

**Expected Output:** The email field is announced as "required" and its hint is announced as a description. The username field is announced as "read-only."

### Real-World Cases

- **Form fields:** `aria-required`, `aria-readonly`, `aria-describedby` for hints and errors.
- **Custom widgets:** `aria-controls` for disclosure buttons, `aria-activedescendant` for comboboxes.
- **Error handling:** `aria-invalid` and `aria-describedby` pointing to error messages.

### References

- W3C — ARIA States and Properties — https://www.w3.org/TR/wai-aria-1.3/#state_prop_def
- MDN — ARIA Attributes — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes

---

## 5. ARIA Labels

### Definitions

**Core Definition:** ARIA labels provide an accessible name for an element — a short string that identifies its purpose to assistive technology users.

**Technical Definition:** An accessible name is a short string, typically 1–3 words, that authors associate with an element to provide users of assistive technologies with a label for the element. It can be provided via `aria-label` (a string attribute) or `aria-labelledby` (a reference to another element's content). Both WAI-ARIA and WCAG require all focusable, interactive elements to have an accessible name.

**Beginner-Friendly Explanation:** A label is what the screen reader calls an element — "Search," "Close," "Submit." Without a label, an icon button is announced as just "button," which tells the user nothing.

### Purposes

- To provide an accessible name for controls that lack visible text (icon buttons).
- To distinguish multiple similar controls (e.g., "Edit" buttons in a list).
- To satisfy WCAG 4.1.2 (Name, Role, Value) and 2.4.6 (Headings and Labels).
- To make custom widgets identifiable by screen readers.

### Syntax Rules and Structure

```html
<!-- String label -->
<element aria-label="accessible name">

<!-- Reference label -->
<element aria-labelledby="id-of-label-element">
```

**Labeling methods:**

| Method | Syntax | When to use |
|--------|--------|-------------|
| `aria-label` | `aria-label="Search"` | No visible text is available. |
| `aria-labelledby` | `aria-labelledby="heading-id"` | Visible text exists elsewhere and should be the name. |
| Native `<label>` | `<label for="id">` | Form controls. |
| Child content | Text inside the element | Buttons and links with visible text. |

**Constraints and Limitations:**

- `aria-label` must be concise — 1–3 words typically.
- Do not use `aria-label` on elements where naming is prohibited (e.g., generic `<div>`).
- `aria-labelledby` overrides `aria-label` and child content — do not mix them.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Icon Button with `aria-label`

```html
<button aria-label="Search">
  <svg aria-hidden="true">...</svg>
</button>
```

**Expected Output:** Screen readers announce "Search, button" instead of just "button."

#### Example 2: `aria-labelledby` for a Section

```html
<section aria-labelledby="pricing-heading">
  <h2 id="pricing-heading">Pricing</h2>
  <p>...</p>
</section>
```

**Expected Output:** The section is announced as a named region "Pricing," and the heading provides a navigable point.

### Real-World Cases

- **Icon buttons:** Menu, search, close, and social media icons.
- **Custom widgets:** Tabs, dialogs, and regions need names via `aria-labelledby`.
- **Data tables:** `aria-labelledby` referencing a caption or heading.

### References

- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/
- MDN — aria-label — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label

---

## 6. ARIA Descriptions

### Definitions

**Core Definition:** ARIA descriptions provide additional explanatory information about an element — such as instructions, format requirements, or error messages — beyond its accessible name.

**Technical Definition:** An accessible description is an author-provided string rendered by assistive technologies, typically via `aria-describedby` referencing another element's content. Descriptions are presented after the name and role, and are usually longer than names. Some screen readers do not announce descriptions by default, instead informing users of their presence so they can request them.

**Beginner-Friendly Explanation:** A description is extra information about a field — like "Password must be at least 8 characters" — that the screen reader reads after the label. It tells the user *how* to fill in the field, not just *what* it is.

### Purposes

- To associate instructions and format requirements with form fields.
- To provide error messages and validation feedback to screen reader users.
- To offer additional context for complex controls.
- To support WCAG 1.3.1 (Info and Relationships) and 3.3.2 (Labels or Instructions).

### Syntax Rules and Structure

```html
<element aria-describedby="id-of-description-element">
```

**Description sources:**

| Source | When to use |
|--------|-------------|
| `aria-describedby` | Reference to visible text elsewhere. |
| `title` attribute | Fallback description (less reliable). |
| `<caption>` | Tables and figures (becomes description when name comes from `aria-label`). |

**Constraints and Limitations:**

- Descriptions are optional — not every element needs one.
- Do not use `aria-describedby` to provide the accessible name — use `aria-labelledby` or `aria-label` for that.
- Descriptions should add information, not repeat the label.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Form Field with Hint and Error

```html
<label for="password">Password</label>
<input
  id="password"
  type="password"
  aria-describedby="password-hint password-error"
>
<p id="password-hint">Must be at least 8 characters.</p>
<p id="password-error" role="alert">Password is too short.</p>
```

**Expected Output:** The field is announced as "Password, edit text, Must be at least 8 characters. Password is too short." The `role="alert"` ensures the error is announced immediately.

### Real-World Cases

- **Form instructions:** Format requirements (dates, phone numbers, password rules).
- **Error messages:** Validation feedback associated with the invalid field.
- **Complex widgets:** Additional instructions for sliders, comboboxes, or data grids.

### References

- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/
- MDN — aria-describedby — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby

---

## 7. ARIA Live Regions

### Definitions

**Core Definition:** An ARIA live region is a mechanism for notifying assistive technologies when content changes dynamically, so screen reader users are informed of updates even when their focus is elsewhere.

**Technical Definition:** Live regions are perceivable regions of a web page that are typically updated as a result of an external event when user focus may be elsewhere. WAI-ARIA provides four attributes to control live region behavior: `aria-live` (off, polite, assertive), `aria-atomic` (true/false), `aria-relevant` (additions, removals, text, all), and `aria-busy` (true/false).

**Beginner-Friendly Explanation:** A live region is an area of the page that the screen reader watches. When something changes inside it — like a status message or an error — the screen reader announces the change automatically, without the user having to navigate to it.

### Purposes

- To announce dynamic content updates (search results, form errors, status messages).
- To inform users of changes in single-page applications where the URL does not change.
- To provide real-time feedback without disrupting the user's current task.
- To satisfy WCAG 4.1.3 (Status Messages).

### Syntax Rules and Structure

```html
<div aria-live="polite" aria-atomic="true">
  <!-- Dynamic content goes here -->
</div>
```

**Live region attributes:**

| Attribute | Values | Purpose |
|-----------|--------|---------|
| `aria-live` | `off`, `polite`, `assertive` | Priority of announcements. |
| `aria-atomic` | `true`, `false` | Announce the whole region or only changes. |
| `aria-relevant` | `additions`, `removals`, `text`, `all` | Types of changes to announce. |
| `aria-busy` | `true`, `false` | Indicates the region is being updated. |

**Pre-defined live region roles:**

| Role | Implicit `aria-live` | Use case |
|------|---------------------|----------|
| `alert` | `assertive` | Critical errors. |
| `status` | `polite` | Status messages. |
| `log` | `polite` | Chat logs, activity feeds. |
| `timer` | `off` | Countdown timers. |

**Constraints and Limitations:**

- `aria-live="assertive"` interrupts the user — use it only for critical errors.
- The live region must exist in the DOM **before** content is injected — dynamically creating the region and its content at the same time may not announce.
- Do not use `aria-live` on every dynamic element — reserve it for meaningful announcements.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Polite Status Message

```html
<div aria-live="polite" aria-atomic="true">
  <p id="status-message"></p>
</div>
```

```javascript
document.getElementById('status-message').textContent = 'Saved successfully.';
```

**Expected Output:** When the text is updated, the screen reader announces "Saved successfully" after the user finishes their current task.

#### Example 2: Assertive Error Alert

```html
<div role="alert">
  <p>Your session has expired. Please log in again.</p>
</div>
```

**Expected Output:** The error is announced immediately, interrupting the user's current activity.

### Real-World Cases

- **Form validation:** Announcing errors as the user submits.
- **Search results:** Announcing the number of results found.
- **Chat applications:** Announcing new messages in an activity feed.
- **Single-page applications:** Announcing route changes and page loads.

### References

- W3C — ARIA Live Regions — https://www.w3.org/WAI/ARIA/apg/practices/live-regions/
- MDN — ARIA Live Regions — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions
- Oracle — ARIA Live Regions — https://docs.oracle.com/en/middleware/developer-tools/jet/9.1/develop/developing-applications-oracle-jet.pdf

---

## 8. ARIA Relationships

### Definitions

**Core Definition:** ARIA relationships are properties that define connections between elements — such as which element controls another, which element owns another, or which element describes another.

**Technical Definition:** Relationship attributes indicate associations between elements that cannot be determined from the document structure alone. They include `aria-controls` (identifies the element(s) controlled), `aria-owns` (establishes a parent-child relationship not reflected in the DOM), `aria-activedescendant` (identifies the active descendant of a composite widget), `aria-flowto` (identifies the next element in an alternate reading order), `aria-labelledby` (name relationship), and `aria-describedby` (description relationship).

**Beginner-Friendly Explanation:** A relationship tells the screen reader "this button controls that panel" or "this input is described by that error message." It connects elements that are visually or functionally related but not structurally connected in the HTML.

### Purposes

- To define controller/controlled relationships (e.g., a button that opens a panel).
- To establish ownership when the DOM hierarchy does not reflect the logical structure.
- To identify the active item within a composite widget.
- To associate labels and descriptions across elements.

### Syntax Rules and Structure

```html
<element aria-relationship="id-reference">
```

**Common relationship properties:**

| Property | Purpose | Example |
|----------|---------|---------|
| `aria-controls` | Identifies the element(s) this element controls. | `aria-controls="panel-id"` |
| `aria-owns` | Establishes a parent-child relationship. | `aria-owns="item-id"` |
| `aria-activedescendant` | Identifies the active descendant. | `aria-activedescendant="option-2"` |
| `aria-flowto` | Defines alternate reading order. | `aria-flowto="next-section"` |
| `aria-labelledby` | References the labeling element. | `aria-labelledby="heading-id"` |
| `aria-describedby` | References the describing element. | `aria-describedby="hint-id"` |

**Constraints and Limitations:**

- Referenced IDs must exist and be accessible to the user.
- `aria-owns` can change the accessibility tree structure — use sparingly and only when the DOM cannot be restructured.
- `aria-activedescendant` is for composite widgets where focus stays on the container, not individual items.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Disclosure Button Controlling a Panel

```html
<button aria-expanded="false" aria-controls="details-panel">
  Show details
</button>
<div id="details-panel" hidden>
  <p>Details content...</p>
</div>
```

**Expected Output:** The button is announced as "Show details, button, collapsed, controls details-panel." When expanded, the panel becomes visible and the button state updates.

### Real-World Cases

- **Disclosure widgets:** `aria-controls` linking the trigger to the panel.
- **Comboboxes:** `aria-activedescendant` tracking the highlighted option.
- **Custom menus:** `aria-owns` when menu items are rendered in a portal.
- **Form errors:** `aria-describedby` linking the input to its error message.

### References

- W3C — ARIA Relationships — https://www.w3.org/TR/wai-aria-1.3/#relationships
- MDN — ARIA Relationship Attributes — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes

---

## 9. ARIA Best Practices

### Definitions

**Core Definition:** ARIA best practices are the guidelines that ensure ARIA is used correctly — improving accessibility rather than introducing new barriers.

**Technical Definition:** Best practices are governed by the **four rules of ARIA**: (1) use native HTML if it provides the required semantics and behavior; (2) do not change native semantics unless absolutely necessary; (3) all interactive ARIA controls must be keyboard operable; (4) do not put `role="presentation"` or `aria-hidden="true"` on focusable elements. Additionally, every interactive element must have an accessible name, ARIA states must be kept in sync with the UI, and ARIA should be tested with real assistive technology.

**Beginner-Friendly Explanation:** Best practices are the rules that keep ARIA from making things worse. Use native HTML first, do not lie about what an element is, make sure keyboard users can operate everything, and test with a real screen reader.

### Purposes

- To prevent ARIA from introducing new accessibility barriers.
- To ensure custom widgets are fully operable and understandable.
- To maintain consistency between visual and non-visual experiences.
- To satisfy WCAG conformance requirements.

### Syntax Rules and Structure

**The four rules of ARIA:**

| Rule | Meaning |
|------|---------|
| **1. Use native HTML** | If `<button>`, `<nav>`, `<input>` does the job, use it. |
| **2. Do not change native semantics** | Do not put `role="heading"` on a `<button>`. |
| **3. Keyboard operability** | Every interactive ARIA control must work with a keyboard. |
| **4. No hidden focusable elements** | Do not use `aria-hidden="true"` on focusable elements. |

**Additional best practices:**

| Practice | Description |
|----------|-------------|
| **Accessible names** | Every interactive element must have one. |
| **Sync states** | `aria-expanded` must match the actual UI state. |
| **Test with AT** | Use NVDA, JAWS, VoiceOver — not just simulators. |
| **No redundant ARIA** | Do not add `role="button"` to a `<button>`. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Best-Practice Custom Widget

```html
<!-- ✅ Uses native button, correct state, keyboard support -->
<button
  aria-expanded="false"
  aria-controls="menu-1"
  onclick="toggleMenu()"
>
  Menu
</button>
<ul id="menu-1" hidden>
  <li><a href="/home">Home</a></li>
</ul>
```

**Expected Output:** The button is focusable, activates with Enter/Space, announces "collapsed" / "expanded," and controls the menu. Keyboard users can operate it; screen reader users understand it.

### Real-World Cases

- **Design systems:** Every custom component follows ARIA best practices.
- **CI pipelines:** `eslint-plugin-jsx-a11y` and axe-core enforce rules automatically.
- **Accessibility audits:** ARIA misuse is a common finding in conformance reports.

### References

- WCAGrules — When to Use ARIA and When Not To — https://wcagrules.com/guides/aria-explained
- Level Access — WAI-ARIA Guidance: Best Practices — https://www.levelaccess.com/blog/wai-aria-guidance-best-practices-for-accessible-web-interfaces/

---

## 10. ARIA Pitfalls

### Definitions

**Core Definition:** ARIA pitfalls are common mistakes that make accessibility worse — incorrect roles, missing keyboard support, stale states, and redundant attributes.

**Technical Definition:** Common pitfalls include: adding `role="button"` to a `<div>` without keyboard support, setting `aria-expanded="false"` statically on a button that does toggle, using `aria-hidden="true"` on focusable elements, applying redundant ARIA to native elements, and using `aria-label` when visible text already exists (causing double announcements). Pages with ARIA averaged **59.1 detectable errors** versus 42 on pages without ARIA, per the 2026 WebAIM Million report.

**Beginner-Friendly Explanation:** ARIA pitfalls are the traps developers fall into — like putting `role="button"` on a div and forgetting to make it keyboard-accessible. The result is a screen reader that says "button" but does nothing when the user presses Enter.

### Common Pitfalls

| Pitfall | Why it is harmful | Correct approach |
|---------|-------------------|------------------|
| **`role="button"` on a `<div>` without keyboard support** | Announces a button that cannot be focused or activated. | Use `<button>` or add `tabindex`, `onkeydown`, and focus styles. |
| **Static `aria-expanded`** | The state never updates, so the screen reader reports the wrong condition. | Sync state with the UI via JavaScript/React state. |
| **`aria-hidden="true"` on focusable elements** | Keyboard users can focus something screen reader users cannot hear. | Never hide focusable elements. |
| **Redundant ARIA on native elements** | `aria-label` on a `<button>` with visible text causes double or unpredictable announcements. | Let native semantics and visible text do the work. |
| **`aria-label` and `aria-labelledby` together** | Conflicts — one overrides the other unpredictably. | Choose one naming method. |
| **ARIA without JavaScript behavior** | The role promises functionality that does not exist. | Implement the keyboard interaction the role implies. |
| **`aria-live` on dynamically created regions** | The screen reader may not detect the region if it is created at the same time as its content. | Render the live region empty first, then update its content. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: The Div Button Pitfall

```html
<!-- ❌ PITFALL -->
<div role="button" onclick="submit()">Submit</div>

<!-- ✅ CORRECT -->
<button type="button" onclick="submit()">Submit</button>
```

**Expected Output:** The `<div>` is announced as "button" but is not focusable and does not respond to Enter/Space. The `<button>` is fully operable.

### Real-World Cases

- **Legacy codebases:** Div buttons are common in older code.
- **CSS frameworks:** Some frameworks encourage div-based components.
- **Rapid prototyping:** Developers reach for `<div>` because it has no default styling.

### References

- Accessibility.com — ARIA Myths That Cause Conformance Failures — https://www.accessibility.com/blog/aria-myths-that-fail-conformance-checks
- Level Access — WAI-ARIA Guidance — https://www.levelaccess.com/blog/wai-aria-guidance-best-practices-for-accessible-web-interfaces/
- CSS-Tricks — I Learned The First Rule of ARIA the Hard Way — https://css-tricks.com/

---

## 11. When to Use ARIA

### Definitions

**Core Definition:** ARIA should be used only when native HTML cannot provide the required semantics or behavior for a custom widget or dynamic content update.

**Technical Definition:** ARIA is appropriate in four situations: (1) **signposts and landmarks** for custom page regions not covered by HTML; (2) **dynamic content updates** that need to be announced via live regions; (3) **enhancing keyboard accessibility** for elements that must receive focus but lack native support; and (4) **non-semantic controls** built from `<div>`s that need roles and properties to be understandable.

**Beginner-Friendly Explanation:** Use ARIA when you are building something HTML does not have — like a custom tab panel or a complex combobox — and you need to tell the screen reader what it is.

### When to Use ARIA

| Scenario | Why ARIA is needed |
|----------|-------------------|
| **Custom widgets** | Tabs, accordions, comboboxes, trees, data grids built from generic elements. |
| **Live regions** | Dynamic status messages, search results, chat logs. |
| **Landmarks for custom regions** | `role="search"`, `role="region"` for areas without native equivalents. |
| **Relationship attributes** | `aria-controls`, `aria-activedescendant` for composite widgets. |
| **State communication** | `aria-expanded`, `aria-selected`, `aria-checked` on custom controls. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Legitimate ARIA Use — Custom Combobox

```html
<div role="combobox" aria-expanded="false" aria-controls="listbox-1" aria-haspopup="listbox">
  <input type="text" aria-autocomplete="list" aria-activedescendant="option-1">
</div>
<ul id="listbox-1" role="listbox">
  <li id="option-1" role="option" aria-selected="true">Option 1</li>
  <li id="option-2" role="option" aria-selected="false">Option 2</li>
</ul>
```

**Why ARIA is justified:** There is no native `<combobox>` element that provides this exact behavior. ARIA roles and states tell the screen reader what the widget is and which option is active.

### Real-World Cases

- **Design systems:** Custom components that cannot be built with native HTML.
- **Data-heavy applications:** Data grids, trees, and carousels.
- **Single-page applications:** Live regions for route changes and dynamic updates.

### References

- MDN — WAI-ARIA Basics: When to Use — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/WAI-ARIA_basics
- WCAGrules — When to Use ARIA — https://wcagrules.com/guides/aria-explained

---

## 12. When Not to Use ARIA

### Definitions

**Core Definition:** ARIA should **not** be used when a native HTML element or attribute already provides the required semantics and behavior.

**Technical Definition:** The first rule of ARIA is: do not use ARIA if native HTML can do the job. Native elements like `<button>`, `<nav>`, `<input>`, `<select>`, and `<details>` come with built-in roles, keyboard behavior, and state management. Adding ARIA to these elements is redundant at best and harmful at worst — `aria-label` on a `<button>` with visible text can cause double announcements, and `role="heading"` on a `<button>` changes semantics without adding value.

**Beginner-Friendly Explanation:** Do not use ARIA when HTML already does the job. A `<button>` is already a button — you do not need to add `role="button"`. Adding ARIA to native elements only creates problems.

### When Not to Use ARIA

| Scenario | Correct approach |
|----------|------------------|
| **Native `<button>`** | Do not add `role="button"` — it is redundant. |
| **Native `<input type="checkbox">`** | Do not add `role="checkbox"` or `aria-checked`. |
| **Native `<nav>`** | Do not add `role="navigation"`. |
| **Visible text on a button** | Do not add `aria-label` — the visible text is already the name. |
| **Native `<select>`** | Do not rebuild it with ARIA unless absolutely necessary. |
| **`<div>` without a role** | Do not add `aria-label` — naming is prohibited on generic elements. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Redundant ARIA on Native Elements

```html
<!-- ❌ REDUNDANT AND HARMFUL -->
<button role="button" aria-label="Submit form">Submit</button>

<!-- ✅ CORRECT -->
<button type="submit">Submit</button>
```

**Expected Output:** The first version may cause the screen reader to announce "Submit form, button, Submit" — double naming. The second announces "Submit, button" cleanly.

### Real-World Cases

- **Design systems:** Native elements should be used before reaching for ARIA.
- **Form controls:** Native `<input>`, `<select>`, and `<textarea>` provide full accessibility.
- **Navigation:** `<nav>` already provides the navigation landmark.

### References

- WCAGrules — The First Rule of ARIA — https://wcagrules.com/guides/aria-explained
- Accessibility.com — ARIA Myths — https://www.accessibility.com/blog/aria-myths-that-fail-conformance-checks

---

## 13. Accessible React (ARIA in React)

### Definitions

**Core Definition:** Accessible React means applying ARIA correctly in React components — keeping ARIA states in sync with React state, using semantic HTML, and managing focus and live regions.

**Technical Definition:** React supports all ARIA attributes in JSX. The key challenge is that ARIA states must be **derived from React state** so they always reflect the current UI — `aria-expanded={isOpen}`, not a hardcoded string. React's dynamic rendering can disrupt focus order, so `useRef` and `useEffect` are used for focus management. `useId` provides SSR-safe unique IDs for `aria-labelledby` and `aria-describedby`. `eslint-plugin-jsx-a11y` catches ARIA violations at lint time.

**Beginner-Friendly Explanation:** In React, ARIA attributes should be driven by your component's state — if the dropdown is open, `aria-expanded` should be `true`. React's re-rendering makes this easy, but you have to remember to connect the ARIA state to the React state.

### Purposes

- To keep ARIA states synchronized with React's declarative state.
- To provide accessible names and descriptions in JSX components.
- To manage focus when React dynamically renders or removes elements.
- To announce dynamic updates with live regions in single-page applications.

### Syntax Rules and Structure

```jsx
<button
  aria-expanded={isOpen}              // State-driven, not hardcoded
  aria-controls={panelId}
  onClick={toggle}
>
  Menu
</button>
```

**React-specific ARIA rules:**

| Rule | Correct pattern |
|------|----------------|
| **State-driven ARIA** | `aria-expanded={isOpen}` — not `aria-expanded="false"`. |
| **Unique IDs** | `useId()` for `aria-labelledby` / `aria-describedby`. |
| **Focus management** | `useRef` + `useEffect` to move focus into modals. |
| **Live regions** | `role="alert"` or `aria-live` for async updates. |
| **Linting** | `eslint-plugin-jsx-a11y` catches violations. |

**Constraints and Limitations:**

- Do not hardcode ARIA states — they will become stale when the UI changes.
- React Fragments (`<>...</>`) do not render DOM elements — do not put ARIA on them.
- `dangerouslySetInnerHTML` can introduce inaccessible content.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible React Form Field with `useId`

```jsx
import { useId } from 'react';

function EmailField({ value, onChange, error }) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>Email address</label>
      <input
        id={id}
        type="email"
        value={value}
        onChange={onChange}
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
      />
      {error && <p id={errorId} role="alert">{error}</p>}
    </div>
  );
}
```

**Expected Output:** The label is associated with the input; when `error` is present, the input is marked invalid and the error message is announced via `role="alert"`.

#### Example 2: Accessible React Toggle Button

```jsx
function ToggleButton({ isPressed, onToggle, children }) {
  return (
    <button
      type="button"
      aria-pressed={isPressed}   // Synced with React state
      onClick={onToggle}
    >
      {children}
    </button>
  );
}
```

**Expected Output:** Screen readers announce the pressed state correctly as it changes.

### Real-World Cases

- **Design systems:** React component libraries use ARIA-driven props for accessibility.
- **Single-page applications:** Live regions announce route changes and data updates.
- **Modal dialogs:** Focus management with `useRef` and `useEffect`.

### References

- BrowserStack — React Accessibility: Complete Guide for Developers — https://enterprise.browserstack.com/accessibility-testing/techniques/react-accessibility
- React — Accessibility — https://react.dev/learn/accessibility
- eslint-plugin-jsx-a11y — https://github.com/jsx-eslint/eslint-plugin-jsx-a11y

---

## References

- W3C — WAI-ARIA 1.3 Specification — https://www.w3.org/TR/wai-aria-1.3/
- W3C — WAI-ARIA Overview — https://w3c.github.io/aria/
- W3C — ARIA Authoring Practices Guide — https://www.w3.org/WAI/ARIA/apg/
- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/
- W3C — ARIA Live Regions — https://www.w3.org/WAI/ARIA/apg/practices/live-regions/
- W3C — ARIA in HTML — https://www.w3.org/TR/html-aria/
- MDN — WAI-ARIA Basics — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/WAI-ARIA_basics
- MDN — ARIA Roles — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles
- MDN — ARIA Attributes — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes
- MDN — ARIA Live Regions — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions
- MDN — aria-label — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label
- MDN — aria-describedby — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby
- Level Access — WAI-ARIA Guidance: Best Practices — https://www.levelaccess.com/blog/wai-aria-guidance-best-practices-for-accessible-web-interfaces/
- WCAGrules — When to Use ARIA and When Not To — https://wcagrules.com/guides/aria-explained
- Accessibility.com — ARIA Myths That Cause Conformance Failures — https://www.accessibility.com/blog/aria-myths-that-fail-conformance-checks
- BrowserStack — React Accessibility: Complete Guide for Developers — https://enterprise.browserstack.com/accessibility-testing/techniques/react-accessibility
- Oracle — ARIA Live Regions — https://docs.oracle.com/en/middleware/developer-tools/jet/9.1/develop/developing-applications-oracle-jet.pdf
- CSS-Tricks — I Learned The First Rule of ARIA the Hard Way — https://css-tricks.com/
- React — Accessibility — https://react.dev/learn/accessibility
- eslint-plugin-jsx-a11y — https://github.com/jsx-eslint/eslint-plugin-jsx-a11y