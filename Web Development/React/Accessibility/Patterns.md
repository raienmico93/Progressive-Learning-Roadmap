# Accessibility Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Accessibility patterns are reusable, standardized solutions for building interactive UI components — forms, buttons, modals, tabs, carousels — that work correctly for all users, including those who rely on keyboards, screen readers, and other assistive technologies.

**Technical Definition:** Accessibility patterns are implementation recipes defined by the **WAI-ARIA Authoring Practices Guide (APG)** and validated against WCAG 2.2. Each pattern specifies the required **roles, states, and properties** (ARIA), the **keyboard interaction model** (which keys do what), and the **focus management behavior** (where focus moves and when). These patterns exist because the first rule of ARIA is that **no ARIA is better than bad ARIA** — semantic HTML should be used wherever possible, and ARIA patterns should be applied only when native HTML cannot express the required behavior .

**Beginner-Friendly Explanation:** Accessibility patterns are the "correct ways" to build common UI components so that everyone can use them. A button should be a real `<button>`, a modal should trap focus and close with Escape, and a notification should be announced by screen readers. These patterns tell you exactly what to do — and what not to do.

### Key Characteristics

- **APG-aligned:** Each pattern follows the W3C's WAI-ARIA Authoring Practices, which define roles, states, and keyboard interaction for 30+ component types .
- **Native-first:** Semantic HTML is preferred; ARIA is used only to fill gaps that HTML cannot express .
- **Keyboard-complete:** Every interactive pattern must be fully operable with a keyboard — Tab, Shift+Tab, Enter, Space, Arrow keys, and Escape .
- **Focus-managed:** Patterns define where focus goes when a component opens, while it is open, and when it closes .
- **State-synchronized:** ARIA states (`aria-expanded`, `aria-selected`, `aria-checked`) must always reflect the actual UI state .

### Prerequisites

- Solid understanding of semantic HTML and the accessibility tree.
- Familiarity with ARIA roles, states, and properties.
- Keyboard navigation fluency and screen reader testing basics.
- Awareness of WCAG 2.2 success criteria (especially 4.1.2 Name, Role, Value).

### Related Programming Areas

- **Semantic HTML** — the foundation all patterns build on.
- **ARIA** — the supplement that makes custom widgets understandable.
- **Keyboard navigation** — the primary interaction model for all patterns.
- **Screen readers** — the primary consumers of pattern semantics.
- **Accessibility testing** — how patterns are validated.

---

## 1. Accessible Forms

### Definitions

**Core Definition:** An accessible form is a form where every control has a programmatically associated label, instructions and errors are announced to assistive technology, and the entire form can be completed with a keyboard.

**Technical Definition:** Accessible forms satisfy WCAG 1.3.1 (Info and Relationships) and 3.3.2 (Labels or Instructions). Every `<input>`, `<select>`, and `<textarea>` must have an accessible name via `<label for="...">` or `aria-label`/`aria-labelledby`. Help text and errors must be associated via `aria-describedby`. Invalid fields must be marked with `aria-invalid="true"`, and error messages should use `role="alert"` so they are announced immediately. Placeholder text must never be used as the only label .

**Beginner-Friendly Explanation:** Every form field needs a real, visible label — not just a placeholder. When something goes wrong, the error message should be connected to the field and announced to screen readers.

### Purposes

- To ensure every form control has a clear, programmatic label.
- To announce instructions and errors to assistive technology.
- To enable keyboard-only users to complete and submit the form.
- To prevent common failures (placeholder-as-label, color-only errors).

### Syntax Rules and Structure

**Accessible form field template:**

```html
<div>
  <label for="email">Email address</label>
  <input
    id="email"
    type="email"
    name="email"
    required
    aria-describedby="email-help email-error"
  />
  <p id="email-help">We'll never share your email.</p>
  <p id="email-error" role="alert" hidden>Please enter a valid email.</p>
</div>
```

**Key rules:**

| Rule | Why it matters |
|------|----------------|
| Use `<label for="...">` | Provides the accessible name and a larger click target. |
| Never use placeholder as label | Placeholders disappear on input, leaving the field unlabeled. |
| Use `aria-describedby` for help/error text | Associates instructions and errors with the field. |
| Use `aria-invalid="true"` for errors | Announces the field as invalid. |
| Use `role="alert"` for errors | Announces the error immediately when it appears. |

**Constraints and Limitations:**

- Visual proximity is not enough — labels must be programmatically associated.
- Color alone cannot indicate errors — add text or icons.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Email Field

```html
<label for="email">Email address</label>
<input
  id="email"
  type="email"
  name="email"
  required
  aria-describedby="email-hint"
  aria-invalid="false"
/>
<p id="email-hint">Format: name@example.com</p>
```

**Expected Output:** Screen readers announce "Email address, edit text, required, Format: name@example.com." If `aria-invalid` becomes `"true"` and the error text appears in a `role="alert"` element, the error is announced immediately.

### Real-World Cases

- **Registration forms:** Labels, password requirements, and validation errors.
- **Checkout forms:** Shipping address, payment fields, and error summaries.
- **Search forms:** Labeled search inputs with keyboard-accessible submit buttons.

### References

- W3C — Technique H91: Using HTML form controls and links — https://www.w3.org/WAI/WCAG21/Techniques/html/H91
- Rampstack — Accessible Form Input — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md

---

## 2. Accessible Buttons

### Definitions

**Core Definition:** An accessible button is a native `<button>` element (or a fully ARIA-equivalent custom widget) that is focusable, activatable with Enter and Space, and has a clear accessible name.

**Technical Definition:** The `<button>` element provides the `button` role, keyboard activation (Enter and Space), and focusability automatically. ARIA is only needed for toggle buttons (`aria-pressed`) and disclosure buttons (`aria-expanded`). Anti-patterns include `<div onClick>`, `<a href="#">` used as a button, and missing `type="button"` inside forms (which causes accidental submits) .

**Beginner-Friendly Explanation:** Use a real `<button>`. Do not build a button out of a `<div>` — the `<div>` will not respond to the keyboard or be announced as a button by screen readers.

### Purposes

- To provide a control that is focusable and activatable by keyboard by default.
- To expose the `button` role and accessible name to assistive technology.
- To communicate toggle state (`aria-pressed`) and disclosure state (`aria-expanded`).

### Syntax Rules and Structure

```html
<!-- Basic button -->
<button type="button" onclick="submit()">Submit</button>

<!-- Toggle button -->
<button type="button" aria-pressed="false">Bookmark</button>

<!-- Disclosure button -->
<button type="button" aria-expanded="false" aria-controls="menu-1">
  Menu
</button>
<ul id="menu-1" hidden>...</ul>
```

**Constraints and Limitations:**

- Always specify `type="button"` unless the button is a form submit button — otherwise it defaults to `type="submit"` and may submit a form unexpectedly .
- A `<div role="button">` requires manual `tabindex="0"` and `keydown` handlers for Enter and Space — use `<button>` instead.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Toggle Button

```html
<button type="button" aria-pressed="false" id="bookmark-btn">
  Bookmark
</button>
<script>
  const btn = document.getElementById('bookmark-btn');
  btn.addEventListener('click', () => {
    const pressed = btn.getAttribute('aria-pressed') === 'true';
    btn.setAttribute('aria-pressed', String(!pressed));
  });
</script>
```

**Expected Output:** The button is announced as "Bookmark, toggle button, not pressed." Clicking toggles the state to "pressed."

### Real-World Cases

- **Form submission:** Submit and reset buttons.
- **Toolbars:** Bold, italic, and other formatting toggles.
- **Menu triggers:** Disclosure buttons that open navigation menus.

### References

- W3C — Button Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/button/
- Rampstack — Accessible Button — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md

---

## 3. Accessible Links

### Definitions

**Core Definition:** An accessible link is an `<a>` element with a valid `href` and descriptive link text that clearly indicates its destination or purpose.

**Technical Definition:** Links (`<a href="...">`) navigate to a resource and expose the `link` role. They must have an `href` to be keyboard-focusable and announced as links. Buttons that perform actions must not be built as links with `href="#"`. Link text must be meaningful — "click here" and "read more" are anti-patterns because screen reader users often navigate by a list of links and need to know where each one goes .

**Beginner-Friendly Explanation:** A link takes you somewhere — it needs a real `href` and text that tells you where it goes. Do not use a link to do a button's job.

### Purposes

- To navigate to another resource or page.
- To provide descriptive, context-independent link text.
- To support keyboard navigation and screen reader link lists.

### Syntax Rules and Structure

```html
<a href="/products">View all products</a>
```

**Constraints and Limitations:**

- `<a>` without `href` is not focusable and not announced as a link — use `<button>` for actions.
- `<a href="#">` pollutes the URL and behaves unpredictably — use `<button>`.
- Link text must make sense out of context — never "click here" or "read more" .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Descriptive Link

```html
<a href="/articles/accessibility-guide">
  Read the accessibility guide
</a>
```

**Expected Output:** Screen readers announce "Read the accessibility guide, link." The link text is meaningful even when read in a list of all links on the page.

### Real-World Cases

- **Navigation menus:** Links to major site sections.
- **In-content links:** Links to related articles or resources.
- **Footer links:** Privacy policy, terms of service, and contact pages.

### References

- W3C — Link Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/link/
- Rampstack — Accessible Link — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md

---

## 4. Accessible Modals (Dialogs)

### Definitions

**Core Definition:** An accessible modal is a dialog that moves focus into itself when opened, holds focus inside while open, closes on Escape, and returns focus to the trigger when closed.

**Technical Definition:** The native `<dialog>` element, opened with `showModal()`, provides focus containment, Escape-to-close, an inert background, initial focus, and focus restoration **automatically**. If building a custom modal, it must have `role="dialog"`, `aria-modal="true"`, an accessible name via `aria-labelledby`, and manual focus trapping and restoration . The background content must be made inert so it cannot be tabbed into .

**Beginner-Friendly Explanation:** A modal must trap focus inside it, close with Escape, and put focus back where it was when it closes. The easiest way to get all of this is to use the native `<dialog>` element.

### Purposes

- To demand the user's attention for a critical task or message.
- To prevent interaction with obscured background content.
- To manage focus predictably across open, interaction, and close.

### Syntax Rules and Structure

**Native dialog (recommended):**

```html
<dialog id="my-dialog" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm action</h2>
  <button type="button" data-close>Cancel</button>
  <button type="button" data-confirm>Confirm</button>
</dialog>
<script>
  document.getElementById('my-dialog').showModal();
</script>
```

**Custom modal requirements:**

| Requirement | Implementation |
|-------------|----------------|
| Role | `role="dialog"` |
| Modal semantics | `aria-modal="true"` |
| Accessible name | `aria-labelledby="dialog-title"` |
| Focus trap | JavaScript or library (`focus-trap`) |
| Escape handler | Close on `keydown` Escape |
| Focus restoration | Return focus to trigger on close |
| Background inert | `inert` attribute or `aria-hidden` |

**Constraints and Limitations:**

- If the trigger element is removed from the DOM when the modal closes, focus restoration must target a logical alternative .
- Do not put `tabindex` on the dialog element itself — it is not an interactive control .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Native Dialog Modal

```html
<dialog id="confirm-dialog" aria-labelledby="dlg-title">
  <h2 id="dlg-title">Delete item?</h2>
  <p>This action cannot be undone.</p>
  <button type="button" data-close>Cancel</button>
  <button type="button" data-confirm>Delete</button>
</dialog>
<script>
  const dialog = document.getElementById('confirm-dialog');
  document.getElementById('delete-btn').addEventListener('click', () => {
    dialog.showModal();
  });
  dialog.querySelector('[data-close]').addEventListener('click', () => {
    dialog.close();
  });
</script>
```

**Expected Output:** Focus moves into the dialog, Tab cycles only within it, Escape closes it, and focus returns to the delete button.

### Real-World Cases

- **Confirmation dialogs:** Delete, discard changes, and irreversible actions.
- **Login modals:** Authentication without leaving the page.
- **Alert dialogs:** Critical errors that require acknowledgment.

### References

- WCAGrules — How to Build a Modal That Doesn't Strand People — https://wcagrules.com/guides/accessible-modals-and-dialogs
- W3C — Dialog (Modal) Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- Rampstack — Accessible Modal — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md

---

## 5. Accessible Dropdowns (Combobox / Menu)

### Definitions

**Core Definition:** An accessible dropdown is either a **combobox** (an input with an associated popup list) or a **menu** (a list of commands triggered by a button), each with full keyboard support and ARIA state.

**Technical Definition:** The combobox pattern uses `role="combobox"` on an input, `aria-expanded`, `aria-controls` pointing to the popup, and `aria-activedescendant` to identify the highlighted option. Arrow keys navigate options, Enter selects, and Escape closes. The menu pattern uses `aria-haspopup="menu"` and `aria-expanded` on a trigger button, with `role="menu"` and `role="menuitem"` on the popup. A menu is for **commands**, not navigation — use `<nav>` with links for navigation .

**Beginner-Friendly Explanation:** A dropdown can be a text input with suggestions (combobox) or a button that opens a list of actions (menu). Both need arrow-key navigation, Enter to select, and Escape to close.

### Purposes

- To allow users to select from a list of options or commands.
- To provide filtered suggestions as the user types (combobox).
- To expose the expanded/collapsed state and active option to assistive technology.

### Syntax Rules and Structure

**Combobox:**

```html
<label for="city">City</label>
<input
  id="city"
  type="text"
  role="combobox"
  aria-autocomplete="list"
  aria-controls="city-listbox"
  aria-expanded="false"
  autocomplete="off"
/>
<ul id="city-listbox" role="listbox" hidden>
  <li role="option" id="city-1" aria-selected="false">London</li>
  <li role="option" id="city-2" aria-selected="false">Lagos</li>
</ul>
```

**Menu:**

```html
<button type="button" aria-haspopup="menu" aria-expanded="false" aria-controls="user-menu">
  Account
</button>
<ul role="menu" id="user-menu" hidden>
  <li role="menuitem" tabindex="-1"><a href="/profile">Profile</a></li>
  <li role="menuitem" tabindex="-1"><a href="/settings">Settings</a></li>
</ul>
```

**Keyboard requirements:**

| Key | Action |
|-----|--------|
| `Down Arrow` | Open popup / next option |
| `Up Arrow` | Previous option |
| `Enter` | Select highlighted option |
| `Escape` | Close popup |
| `Home` / `End` | First / last option |

**Constraints and Limitations:**

- The combobox pattern is one of the most complex ARIA patterns — test thoroughly with screen readers .
- `role="menu"` is for application-style command menus, not site navigation .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Select-Only Combobox

```html
<label for="state">State</label>
<input
  id="state"
  type="text"
  role="combobox"
  aria-expanded="false"
  aria-controls="state-listbox"
  aria-activedescendant=""
/>
<ul id="state-listbox" role="listbox" hidden>
  <li role="option" id="state-ca">California</li>
  <li role="option" id="state-ny">New York</li>
</ul>
```

**Expected Output:** Down Arrow opens the listbox, Arrow keys move the active option, Enter selects it, and Escape closes the listbox.

### Real-World Cases

- **Search fields:** Comboboxes with suggested results.
- **Location pickers:** Country, state, and city selection.
- **Account menus:** Profile, settings, and logout actions.

### References

- W3C — Combobox Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/combobox/
- Rampstack — Accessible Combobox — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md

---

## 6. Accessible Tabs

### Definitions

**Core Definition:** An accessible tab interface is a set of tab buttons (in a `tablist`) that switch between associated tab panels, with arrow-key navigation between tabs and only one tab in the page Tab order.

**Technical Definition:** The container has `role="tablist"` and an accessible label. Each tab has `role="tab"`, `aria-selected` (`true` for the active tab), and `aria-controls` pointing to its panel. Panels have `role="tabpanel"` and `aria-labelledby` pointing back to their tab. **Roving tabindex** is used: the active tab has `tabindex="0"`, and inactive tabs have `tabindex="-1"`. Arrow keys move between tabs, Home/End jump to the first/last tab .

**Beginner-Friendly Explanation:** Tabs work like browser tabs — you use the arrow keys to move between them, and only the active tab is reachable with Tab. The panel below shows the content of the selected tab.

### Purposes

- To organize content into switchable panels without leaving the page.
- To provide efficient keyboard navigation with arrow keys.
- To expose the selected state of each tab to assistive technology.

### Syntax Rules and Structure

```html
<div role="tablist" aria-label="Settings sections">
  <button role="tab" aria-selected="true" aria-controls="panel-1" id="tab-1" tabindex="0">
    Profile
  </button>
  <button role="tab" aria-selected="false" aria-controls="panel-2" id="tab-2" tabindex="-1">
    Notifications
  </button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
  Profile content...
</div>
<div role="tabpanel" id="panel-2" aria-labelledby="tab-2" hidden>
  Notifications content...
</div>
```

**Behavior requirements:**

- Arrow keys navigate between tabs.
- Active tab has `aria-selected="true"` and `tabindex="0"`.
- Inactive tabs have `aria-selected="false"` and `tabindex="-1"`.
- Home/End jump to first/last tab.

**Constraints and Limitations:**

- ARIA does not provide keyboard behavior for free — you must implement arrow-key navigation in JavaScript .
- Only one tab should be in the Tab order at a time (roving tabindex).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Roving Tabindex Implementation

```javascript
const tabs = document.querySelectorAll('[role="tab"]');

tabs.forEach((tab, index) => {
  tab.addEventListener('keydown', (e) => {
    let newIndex;
    if (e.key === 'ArrowRight') newIndex = (index + 1) % tabs.length;
    if (e.key === 'ArrowLeft') newIndex = (index - 1 + tabs.length) % tabs.length;
    if (e.key === 'Home') newIndex = 0;
    if (e.key === 'End') newIndex = tabs.length - 1;
    if (newIndex !== undefined) {
      e.preventDefault();
      tabs.forEach((t, i) => {
        t.setAttribute('aria-selected', String(i === newIndex));
        t.setAttribute('tabindex', i === newIndex ? '0' : '-1');
      });
      tabs[newIndex].focus();
    }
  });
});
```

**Expected Output:** Arrow keys move focus between tabs, Home/End jump to the first/last tab, and the selected panel becomes visible.

### Real-World Cases

- **Settings pages:** Profile, security, and notification tabs.
- **Product pages:** Description, specifications, and reviews tabs.
- **Dashboards:** Overview, analytics, and reports tabs.

### References

- W3C — Tabs Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/tabs/
- Rampstack — Accessible Tabs — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md

---

## 7. Accessible Accordions

### Definitions

**Core Definition:** An accessible accordion is a vertically stacked set of headings, each containing a button that expands or collapses an associated content panel.

**Technical Definition:** Each accordion header is a heading element (e.g., `<h3>`) containing a `<button>` with `aria-expanded` (`true` when the panel is open) and `aria-controls` pointing to the panel. The panel has `role="region"` and `aria-labelledby` pointing back to the header button, giving the region an accessible name. Enter or Space expands a collapsed panel. Tab moves through all focusable elements in the accordion . The native `<details>`/`<summary>` elements provide this behavior with no JavaScript .

**Beginner-Friendly Explanation:** An accordion is a list of sections that expand and collapse. Each header is a real button inside a heading, and the panel it controls is a named region. Use `<details>` and `<summary>` if you can — they work out of the box.

### Purposes

- To condense large amounts of content into expandable sections.
- To provide keyboard-operable show/hide functionality.
- To expose the expanded state and associated panel to assistive technology.

### Syntax Rules and Structure

```html
<div class="accordion">
  <h3>
    <button type="button" aria-expanded="true" aria-controls="sect1" id="accordion1id">
      Personal Information
    </button>
  </h3>
  <div id="sect1" role="region" aria-labelledby="accordion1id" class="accordion-panel">
    <p>Content...</p>
  </div>
</div>
```

**Keyboard support:**

| Key | Action |
|-----|--------|
| `Enter` / `Space` | Expand or collapse the focused header's panel |
| `Tab` / `Shift+Tab` | Move focus to the next/previous focusable element |

**Constraints and Limitations:**

- All focusable elements in the accordion (including expanded panel content) are in the page Tab sequence — there is no focus trapping .
- Arrow-key navigation between headers is optional but recommended .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Native `<details>` Accordion

```html
<details>
  <summary>Personal Information</summary>
  <p>Content...</p>
</details>
<details>
  <summary>Billing Address</summary>
  <p>Content...</p>
</details>
```

**Expected Output:** Clicking the summary toggles the panel; Enter or Space on the focused summary also toggles it. Screen readers announce the summary as a button with expanded/collapsed state.

### Real-World Cases

- **FAQ pages:** Questions as headers and answers as panels.
- **Form sections:** Grouping related inputs into collapsible sections.
- **Product details:** Description, specifications, and shipping info.

### References

- W3C — Accordion Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/accordion/
- W3C — Accordion Example — https://www.w3.org/WAI/ARIA/apg/patterns/accordion/examples/accordion/

---

## 8. Accessible Carousels

### Definitions

**Core Definition:** An accessible carousel is a slide-show component that allows users to pause automatic rotation, navigate slides with buttons or keyboard controls, and understand slide position through live region announcements.

**Technical Definition:** The carousel container uses `role="region"` with `aria-roledescription="carousel"` and an accessible label. Each slide has `role="group"`, `aria-roledescription="slide"`, and an `aria-label` that includes its position (e.g., "2 of 4"). Navigation controls must be native `<button>` elements with clear accessible names. **Auto-rotation must be pausable**: hovering pauses, keyboard focus pauses, and a visible pause/play button is provided. If the user prefers reduced motion, auto-rotation should start paused .

**Beginner-Friendly Explanation:** A carousel must let users stop the automatic sliding, navigate with buttons, and hear which slide they are on. Never let it rotate uncontrollably — it disorients screen reader users and people with cognitive disabilities.

### Purposes

- To present multiple pieces of content in a limited space.
- To give users full control over slide rotation and navigation.
- To announce slide changes and position to assistive technology.

### Syntax Rules and Structure

```html
<section role="region" aria-roledescription="carousel" aria-label="Featured products">
  <button type="button" aria-label="Pause carousel">Pause</button>
  <div role="group" aria-roledescription="slide" aria-label="1 of 3">
    <!-- Slide 1 content -->
  </div>
  <button type="button" aria-label="Previous slide">Previous</button>
  <button type="button" aria-label="Next slide">Next</button>
</section>
```

**Accessibility requirements:**

| Feature | Requirement |
|---------|-------------|
| Auto-rotation | Must be pausable (hover, focus, visible button) |
| Reduced motion | Start paused if `prefers-reduced-motion` is set |
| Slide labels | Include position ("2 of 4") |
| Controls | Native `<button>` with accessible names |
| Live announcements | Use `aria-live="polite"` for slide changes |

**Constraints and Limitations:**

- Carousels are not in the APG as a single pattern — the closest is the Alert pattern, but carousels are more complex .
- Only the current slide should be in the Tab order; hidden slides must not be focusable .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Pausable Carousel

```javascript
const carousel = document.querySelector('[aria-roledescription="carousel"]');
const pauseBtn = carousel.querySelector('[aria-label="Pause carousel"]');
let rotationTimer;

function startRotation() {
  rotationTimer = setInterval(() => nextSlide(), 5000);
}

pauseBtn.addEventListener('click', () => {
  clearInterval(rotationTimer);
  pauseBtn.setAttribute('aria-label', 'Play carousel');
});

carousel.addEventListener('mouseenter', () => clearInterval(rotationTimer));
carousel.addEventListener('focusin', () => clearInterval(rotationTimer));
```

**Expected Output:** The carousel pauses on hover, on keyboard focus, and when the pause button is clicked.

### Real-World Cases

- **Homepage hero sliders:** Featured content or promotions.
- **Product image galleries:** Multiple product photos.
- **Testimonial rotators:** Customer quotes.

### References

- W3C — Auto-Rotating Image Carousel Example — https://www.w3.org/WAI/ARIA/apg/patterns/carousel/examples/carousel-1-prev-next/
- TestParty — Making Carousels and Sliders Accessible — https://testparty.ai/blog/accessible-carousels

---

## 9. Accessible Tables

### Definitions

**Core Definition:** An accessible data table is a `<table>` used for tabular data with a `<caption>`, `<th>` header cells, and `scope` attributes that programmatically associate headers with data cells.

**Technical Definition:** Data tables must use `<th>` for header cells with `scope="col"` or `scope="row"` so that screen readers can announce the correct header when navigating to a data cell. A `<caption>` provides the table's title and purpose. Tables must **never** be used for layout — layout tables must not contain `<th>`, `<caption>`, `summary`, `scope`, or `headers` attributes . Complex tables may need `headers` and `id` attributes for explicit associations.

**Beginner-Friendly Explanation:** A data table needs a caption, header cells, and scope attributes so screen readers can tell you which column and row a cell belongs to. Never use a table just to lay out a page.

### Purposes

- To present tabular data with clear row and column relationships.
- To allow screen readers to announce header context when navigating cells.
- To distinguish data tables from layout tables.

### Syntax Rules and Structure

```html
<table>
  <caption>Monthly Sales</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>$10,000</td>
    </tr>
    <tr>
      <th scope="row">February</th>
      <td>$12,000</td>
    </tr>
  </tbody>
</table>
```

**Key requirements:**

| Element | Requirement |
|---------|-------------|
| `<caption>` | Names the table |
| `<th scope="col">` | Column headers |
| `<th scope="row">` | Row headers |
| `<thead>`, `<tbody>`, `<tfoot>` | Group header, body, and footer sections |

**Constraints and Limitations:**

- Layout tables must not use `<th>`, `<caption>`, or `scope` — these mislead screen readers into treating layout as data .
- Complex tables may need `headers` and `id` for explicit cell-to-header associations.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Data Table

```html
<table>
  <caption>Q1 2026 Budget</caption>
  <thead>
    <tr>
      <th scope="col">Department</th>
      <th scope="col">Budget</th>
      <th scope="col">Spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Engineering</th>
      <td>$500,000</td>
      <td>$420,000</td>
    </tr>
  </tbody>
</table>
```

**Expected Output:** When navigating to the "$420,000" cell, screen readers announce the column header "Spent" and the row header "Engineering."

### Real-World Cases

- **Financial reports:** Budgets, revenue, and expenses.
- **Comparison tables:** Product features or pricing plans.
- **Schedules:** Timetables and calendars.

### References

- WCAGrules — How to Build Accessible Data Tables — https://wcagrules.com/guides/accessible-data-tables
- W3C — Technique F46: Failure of SC 1.3.1 due to layout tables — https://www.w3.org/WAI/WCAG21/Techniques/failures/F46

---

## 10. Accessible Images

### Definitions

**Core Definition:** An accessible image is one that has a text alternative (`alt` attribute) that serves the same purpose as the image — or, if decorative, is hidden from assistive technology with `alt=""`.

**Technical Definition:** Every `<img>` must have an `alt` attribute. Informative images get descriptive alt text that conveys the image's purpose in context. Decorative images get `alt=""` so screen readers skip them. Complex images (charts, diagrams) get a short alt text plus a **long description** in the surrounding text or via a link to a detailed description . SVG icons should use `aria-hidden="true"` if decorative, or `role="img"` with an accessible name if meaningful .

**Beginner-Friendly Explanation:** Alt text tells screen reader users what an image is for. If the image is just decoration, use empty alt so screen readers skip it. If the image is complex, provide a longer description nearby.

### Purposes

- To convey the purpose of an image to users who cannot see it.
- To avoid noise from decorative images.
- To provide detailed descriptions for complex visual information.

### Syntax Rules and Structure

```html
<!-- Informative image -->
<img src="chart.png" alt="Sales increased 25% in Q2">

<!-- Decorative image -->
<img src="divider.png" alt="">

<!-- Complex image -->
<img src="diagram.png" alt="System architecture overview">
<p>Long description: The system consists of a frontend, API gateway, and three microservices...</p>
```

**Alt text rules:**

| Image type | Alt text |
|------------|----------|
| Informative | Describe the information the image conveys. |
| Decorative | `alt=""` — screen readers skip it. |
| Functional (e.g., icon button) | Describe the function, not the image. |
| Complex | Short alt + long description in nearby text. |

**Constraints and Limitations:**

- Alt text should be concise — typically no more than 120–150 characters .
- Do not start with "Image of" — screen readers already announce it as an image.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Complex Image with Long Description

```html
<figure>
  <img src="org-chart.png" alt="Organizational chart showing the engineering department">
  <figcaption>
    The engineering department is led by the VP of Engineering and includes
    frontend, backend, and infrastructure teams.
  </figcaption>
</figure>
```

**Expected Output:** Screen readers announce "Organizational chart showing the engineering department, image" and then read the figcaption as the long description.

### Real-World Cases

- **Product images:** Alt text describing the product.
- **Charts and infographics:** Short alt plus a detailed data table or text description.
- **Decorative backgrounds:** Empty alt so screen readers skip them.

### References

- W3C — Complex Images — https://www.w3.org/WAI/tutorials/images/complex/
- Accessibile ANU — Alt Text Best Practices — https://learningandteaching.anu.edu.au/accessible-images/

---

## 11. Accessible Icons

### Definitions

**Core Definition:** An accessible icon is either hidden from assistive technology (if decorative) or given an accessible name (if it conveys meaning), depending on its purpose.

**Technical Definition:** Decorative SVG icons must have `aria-hidden="true"` and `focusable="false"` so they are removed from the accessibility tree and cannot receive focus. Icons that convey meaning must have `role="img"` and an accessible name via `aria-label` or a `<title>` element. **Icon-only buttons** must put the accessible name on the `<button>` via `aria-label`, while the icon itself is hidden with `aria-hidden="true"` . Never use `role="presentation"` on an SVG for hiding — it does not reliably remove the icon from assistive technology .

**Beginner-Friendly Explanation:** If an icon is just decoration, hide it from screen readers. If the icon is the only thing in a button, give the button a label and hide the icon.

### Purposes

- To prevent decorative icons from cluttering screen reader output.
- To provide accessible names for icon-only controls.
- To expose meaningful icons to assistive technology.

### Syntax Rules and Structure

```html
<!-- Decorative icon (hidden) -->
<svg aria-hidden="true" focusable="false">...</svg>

<!-- Meaningful icon (named) -->
<svg role="img" aria-label="Warning">...</svg>

<!-- Icon-only button -->
<button type="button" aria-label="Search">
  <svg aria-hidden="true" focusable="false">...</svg>
</button>
```

**Icon rules:**

| Icon purpose | Implementation |
|--------------|----------------|
| Decorative | `aria-hidden="true"` + `focusable="false"` |
| Meaningful standalone | `role="img"` + `aria-label` or `<title>` |
| Inside icon-only button | Hide icon; `aria-label` on the `<button>` |
| Inside button with visible text | `aria-hidden="true"` on the icon |

**Constraints and Limitations:**

- Do not add `aria-label` to an icon when visible text already provides the name — this causes redundant announcements .
- `role="presentation"` on SVG does not reliably hide the icon .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Icon-Only Button

```html
<button type="button" aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false">
    <path d="M6 6L18 18M18 6L6 18" stroke="currentColor"/>
  </svg>
</button>
```

**Expected Output:** Screen readers announce "Close dialog, button." The SVG is skipped entirely.

### Real-World Cases

- **Close buttons:** Icon-only buttons with `aria-label="Close"`.
- **Search buttons:** Magnifying glass icon with `aria-label="Search"`.
- **Social media icons:** Linked icons with accessible names on the `<a>`.

### References

- React Icons — Accessibility — https://mintlify.wiki/react-icons/accessibility
- IconVectors — Accessible SVG Icons — https://iconvectors.io/blog/accessible-svg-icons

---

## 12. Accessible Notifications

### Definitions

**Core Definition:** An accessible notification is a status message or alert that is announced by screen readers via a live region, without requiring focus to move to the message.

**Technical Definition:** WCAG 4.1.3 (Status Messages) requires that status messages — success confirmations, error counts, loading states — be announced by assistive technology without moving focus. Use `role="status"` (implicit `aria-live="polite"`) for non-urgent updates and `role="alert"` (implicit `aria-live="assertive"`) for urgent errors. The live region **must exist in the DOM before the message is inserted** — if the container and its content are inserted simultaneously, screen readers often announce nothing. Do **not** combine `role="alert"` and `aria-live` on the same element — this causes double announcements in VoiceOver .

**Beginner-Friendly Explanation:** A notification that appears and disappears must be announced by screen readers. Use `role="status"` for polite announcements and `role="alert"` for urgent errors. Make sure the live region exists on the page before you put the message in it.

### Purposes

- To inform screen reader users of dynamic changes without moving focus.
- To announce form submission results, error counts, and loading states.
- To satisfy WCAG 4.1.3 (Status Messages).

### Syntax Rules and Structure

```html
<!-- Non-urgent status message -->
<div role="status" aria-live="polite">
  Your changes have been saved.
</div>

<!-- Urgent alert -->
<div role="alert">
  Your session has expired. Please log in again.
</div>

<!-- Loading status -->
<div role="status" aria-live="polite">
  Loading results...
</div>
```

**Live region rules:**

| Type | Role | Priority |
|------|------|----------|
| Success confirmation | `status` | Polite |
| Error message | `alert` | Assertive |
| Loading indicator | `status` | Polite |
| Error summary | `alert` | Assertive |

**Constraints and Limitations:**

- The live region must exist on the page before content is injected .
- Do not combine `role="alert"` with `aria-live` — pick one .
- Avoid `aria-live="assertive"` for non-critical updates — it interrupts the user .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Status Message After Save

```html
<div id="save-status" role="status" aria-live="polite"></div>
<button type="button" onclick="save()">Save</button>
<script>
  function save() {
    // Perform save...
    document.getElementById('save-status').textContent = 'Saved successfully.';
  }
</script>
```

**Expected Output:** When the save completes, screen readers announce "Saved successfully" without moving focus.

### Real-World Cases

- **Form submissions:** "Registration successful" or "3 errors found."
- **Auto-save:** "Draft saved" confirmations.
- **Live search:** "5 results found" announcements.

### References

- WCAGrules — Notifications That Reach the People Who Cannot See Them — https://wcagrules.com/guides/accessible-notifications-and-toasts
- W3C — Alert Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/alert/

---

## 13. Accessibility Best Practices (Patterns)

### Definitions

**Core Definition:** Accessibility pattern best practices are the overarching principles that guide correct implementation of all UI patterns — semantic HTML first, ARIA only when necessary, keyboard completeness, and real-device testing.

**Technical Definition:** Best practices are codified in the **first rule of ARIA** (use native HTML when possible), the **APG keyboard interaction models**, and WCAG 2.2 conformance requirements. They include: never faking elements that exist (`<div role="button">` is worse than `<button>`), using `type="button"` on all non-submit buttons, testing with real screen readers and keyboards, and ensuring that every interactive component has an accessible name, a correct role, and accurate state .

**Beginner-Friendly Explanation:** Use real HTML elements whenever you can. Add ARIA only when you must. Test with a keyboard and a screen reader. Do not fake a button with a div.

### Purposes

- To prevent the most common accessibility failures across all patterns.
- To ensure consistency between visual and non-visual experiences.
- To comply with WCAG 2.2 and legal accessibility requirements.
- To make components that work for everyone, not just mouse users.

### Best Practices Checklist

| Practice | Description |
|----------|-------------|
| **Native-first** | Use `<button>`, `<a href>`, `<input>`, `<select>`, `<textarea>`, `<details>` before ARIA. |
| **No fake elements** | A `<div role="button">` is worse than a `<button>` — do not reinvent what exists. |
| **`type="button"`** | Always set it on non-submit buttons to prevent accidental form submits. |
| **Accessible names** | Every interactive element must have one. |
| **Keyboard-complete** | Every pattern must work with Tab, Enter, Space, Arrow keys, and Escape. |
| **Focus management** | Move focus into modals, trap it, and return it on close. |
| **Live regions** | Announce dynamic status messages; exist before content is injected. |
| **Less ARIA is more** | Most components need little or no ARIA — semantic HTML does the work. |
| **Test with real AT** | NVDA or VoiceOver; unplug the mouse. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Semantic HTML vs. ARIA Reconstruction

```html
<!-- ✅ Native: full keyboard and screen reader support -->
<button type="button" onclick="submit()">Submit</button>

<!-- ❌ ARIA reconstruction: requires manual keyboard handlers, focus styles, and state management -->
<div role="button" tabindex="0" onclick="submit()" onkeydown="handleKey(event)">
  Submit
</div>
```

**Expected Output:** The native `<button>` is focusable, activates with Enter/Space, and is announced correctly. The ARIA version requires three additional JavaScript behaviors that are easy to get wrong.

### Real-World Cases

- **Design systems:** Every component follows these best practices before release.
- **Code reviews:** Reviewers check for native elements, accessible names, and keyboard support.
- **Accessibility audits:** These are the first checks auditors perform.

### References

- W3C — Read Me First: No ARIA is better than Bad ARIA — https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/
- Rampstack — Accessibility Patterns — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md
- Salesforce — Web Component Accessibility — https://github.com/forcedotcom/sf-skills/blob/0c7cb9304a071266264628a84bc498b23cde9c13/skills/experience-accessibility-validate/SKILL.md

---

## References

- W3C — WAI-ARIA Authoring Practices Guide: Patterns — https://www.w3.org/WAI/ARIA/apg/patterns/
- W3C — Read Me First: No ARIA is better than Bad ARIA — https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/
- W3C — Technique H91: Using HTML form controls and links — https://www.w3.org/WAI/WCAG21/Techniques/html/H91
- W3C — Technique F46: Failure of SC 1.3.1 due to layout tables — https://www.w3.org/WAI/WCAG21/Techniques/failures/F46
- W3C — Complex Images — https://www.w3.org/WAI/tutorials/images/complex/
- W3C — Dialog (Modal) Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- W3C — Combobox Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/combobox/
- W3C — Accordion Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/accordion/
- W3C — Accordion Example — https://www.w3.org/WAI/ARIA/apg/patterns/accordion/examples/accordion/
- W3C — Auto-Rotating Image Carousel Example — https://www.w3.org/WAI/ARIA/apg/patterns/carousel/examples/carousel-1-prev-next/
- W3C — Button Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/button/
- W3C — Link Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/link/
- W3C — Tabs Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/tabs/
- W3C — Alert Pattern — https://www.w3.org/WAI/ARIA/apg/patterns/alert/
- WCAGrules — How to Build a Modal That Doesn't Strand People — https://wcagrules.com/guides/accessible-modals-and-dialogs
- WCAGrules — Notifications That Reach the People Who Cannot See Them — https://wcagrules.com/guides/accessible-notifications-and-toasts
- WCAGrules — How to Build Accessible Data Tables — https://wcagrules.com/guides/accessible-data-tables
- Rampstack — Accessibility Patterns — https://github.com/rampstackco/claude-skills/blob/059051082ab0c1dc02d271506d7318eca6b55f51/skills/frontend-component-build/references/accessibility-patterns.md
- React Icons — Accessibility — https://mintlify.wiki/react-icons/accessibility
- IconVectors — Accessible SVG Icons — https://iconvectors.io/blog/accessible-svg-icons
- TestParty — Making Carousels and Sliders Accessible — https://testparty.ai/blog/accessible-carousels
- Salesforce — Web Component Accessibility — https://github.com/forcedotcom/sf-skills/blob/0c7cb9304a071266264628a84bc498b23cde9c13/skills/experience-accessibility-validate/SKILL.md
- Accessibile ANU — Alt Text Best Practices — https://learningandteaching.anu.edu.au/accessible-images/