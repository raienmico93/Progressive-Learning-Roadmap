# Accessible React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Accessible React is the discipline of building React applications that are perceivable, operable, understandable, and robust for all users—including those using screen readers, keyboards, voice control, magnification, and other assistive technologies—by combining semantic HTML, WAI-ARIA patterns, focus management, keyboard interaction, and dynamic announcements.

**Technical Definition:** Accessible React applies the Web Content Accessibility Guidelines (WCAG) 2.2 and the WAI-ARIA Authoring Practices Guide (APG) to React's component-based, client-side rendering architecture. It addresses the accessibility gaps introduced by JavaScript-driven rendering—missing route-change announcements, focus loss after component unmounting, inaccessible custom widgets, and dynamic content that screen readers do not detect—by using semantic HTML landmarks, the accessible name computation algorithm (`aria-label`, `aria-labelledby`, `aria-describedby`), programmatic focus control (`ref.current.focus()`), focus traps, roving tabindex, `aria-live` regions, and WAI-ARIA roles for compound components. It also covers React-specific concerns such as `useId` for stable IDs, refs for focus management, and route-transition announcements in client-side routers.

**Beginner-Friendly Explanation:** When you build a React app, most users see a smooth experience—content updates, modals open, forms validate. But users who cannot see the screen or cannot use a mouse experience something very different. Screen readers might not know a page changed. Keyboard users might get stuck behind a modal. Form errors might appear silently. Accessible React is about making sure your app works for everyone, not just the majority. It is not a separate layer you add at the end—it is woven into how you build components.

### Key Characteristics

- **Semantic HTML First:** Native elements (`<button>`, `<nav>`, `<main>`, `<dialog>`) provide accessibility for free; ARIA is a fallback when no native element exists.
- **Accessible Name Computation:** Every interactive element must have a programmatic name derived from visible text, `aria-label`, `aria-labelledby`, or associated labels.
- **Focus is Sacred:** Focus must be visible, predictable, and managed programmatically on mount, unmount, and route changes.
- **Keyboard Parity:** Every action achievable with a mouse must be achievable with a keyboard.
- **Dynamic Content Needs Announcement:** Screen readers do not automatically detect DOM changes; `aria-live` regions announce them.
- **Compound Components Follow APG:** Tabs, accordions, menus, comboboxes, and dialogs follow established WAI-ARIA patterns.
- **Route Changes Need Announcement:** Client-side navigation does not trigger a page load; the app must announce the change.
- **Automated Tools Catch 30–50%:** Manual keyboard and screen-reader testing is required.

### Prerequisites

- Solid understanding of React function components, JSX, and Hooks.
- Working knowledge of HTML semantics and the DOM.
- Familiarity with `useRef`, `useId`, and event handling.
- Basic understanding of WAI-ARIA roles, states, and properties.
- Awareness of WCAG 2.2 success criteria.

### Related Programming Areas

- **Web Accessibility (a11y):** WCAG, ARIA, and assistive technology.
- **Component Architecture:** Compound components, roving tabindex, and focus management.
- **Routing:** Client-side navigation and page-change announcements.
- **Form Design:** Labels, fieldsets, error messaging, and multi-step flows.
- **Testing:** Keyboard testing, screen-reader testing, and automated accessibility tools.

### Core Concepts / Features

1. Semantic Structure (Landmarks and Heading Hierarchies)
2. Accessible Labels & Names
3. Advanced Focus Management
4. Keyboard Navigation Patterns
5. Dynamic Content Announcements
6. Form & Interaction Patterns
7. Complex ARIA Components (APG Patterns)
8. Accessible Routing

---

## Core Concept 1: Semantic Structure — Landmarks and Heading Hierarchies

### Definitions

**Core Definition:** Semantic structure is the use of HTML landmark elements (`<main>`, `<nav>`, `<header>`, `<footer>`, `<aside>`, `<section>`) and hierarchical headings (`<h1>`–`<h6>`) to provide a navigable outline of the page for screen-reader users.

**Technical Definition:** Landmark elements map to ARIA landmark roles (`<main>` → `role="main"`, `<nav>` → `role="navigation"`) and appear in screen readers' landmark navigation menus, allowing users to jump directly to regions. Heading elements (`<h1>`–`<h6>`) create a document outline; screen readers provide heading navigation and announce the heading level. In a component-based SPA, landmarks must be rendered by layout components (not hardcoded in a single root), and headings must follow a logical hierarchy even when components are composed dynamically. WCAG 2.2 SC 1.3.1 (Info and Relationships) and SC 2.4.1 (Bypass Blocks) require proper structure and a mechanism to skip repeated content.

**Beginner-Friendly Explanation:** Imagine reading a book with no chapters, no headings, and no page numbers. It would be impossible to find anything. Screen readers navigate web pages the same way—landmarks are like chapters, and headings are like section titles. A well-structured page lets a screen-reader user jump to the main content, the navigation, or a specific section. In React, you need to make sure these landmarks are rendered by your layout components, not just one top-level `<div>`.

### Purposes

- To provide navigable landmarks for screen-reader users.
- To establish a logical heading hierarchy for content structure.
- To satisfy WCAG 2.2 SC 1.3.1 and SC 2.4.1.
- To allow users to skip repeated navigation and reach the main content.
- To make the document outline meaningful even in a component-based architecture.

### Syntax Rules and Structure

**Landmark Structure:**
```jsx
function AppLayout({ children }) {
  return (
    <>
      <header>
        <h1>My Application</h1>
        <nav aria-label="Main navigation">
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/products">Products</a></li>
          </ul>
        </nav>
      </header>

      <main id="main-content" tabIndex={-1}>
        {children}
      </main>

      <aside aria-label="Related links">
        <h2>Related</h2>
        <ul>...</ul>
      </aside>

      <footer>
        <nav aria-label="Footer navigation">
          <ul>...</ul>
        </nav>
      </footer>
    </>
  );
}
```

**Component Breakdown:**
- `<header>`: Top-level banner landmark.
- `<nav aria-label="Main navigation">`: Navigation landmark; `aria-label` distinguishes multiple navs.
- `<main id="main-content" tabIndex={-1}>`: Main content landmark; `tabIndex={-1}` makes it programmatically focusable.
- `<aside aria-label="Related links">`: Complementary landmark.
- `<footer>`: Contentinfo landmark.

**Heading Hierarchy:**
```jsx
function Page() {
  return (
    <main>
      <h1>Dashboard</h1>          {/* Page title */}
      <section>
        <h2>Revenue</h2>          {/* Section */}
        <h3>Monthly</h3>          {/* Subsection */}
        <h3>Quarterly</h3>
      </section>
      <section>
        <h2>Users</h2>
        <h3>Active</h3>
        <h3>New</h3>
      </section>
    </main>
  );
}
```

**Component Breakdown:**
- One `<h1>` per page (the page title).
- `<h2>` for major sections; `<h3>` for subsections.
- Do not skip levels (e.g., `<h1>` to `<h3>`).
- Use `<section>` with a heading for thematic grouping.

**Syntax Rules:**
- Render landmarks in layout components, not in a single root `<div>`.
- Use one `<main>` per page; multiple `<main>` elements are invalid.
- Label multiple `<nav>` elements with `aria-label` or `aria-labelledby`.
- Use `<h1>`–`<h6>` for content structure, not for styling; use CSS classes for visual size.
- Do not skip heading levels; `<h2>` should follow `<h1>`, not `<h1>` → `<h3>`.
- Provide a skip link to `#main-content` for keyboard users.

**Constraints and Limitations:**
- `<section>` without an accessible name is not exposed as a landmark; use `aria-labelledby` or `aria-label` if needed.
- `<main>` must be unique; do not render it twice.
- Heading levels are hierarchical, not visual; CSS controls size.
- Some screen readers do not announce landmarks if there are too many; keep them meaningful.

### Annotated Code Example: Full Page Structure

```jsx
function App() {
  return (
    <>
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>

      <header>
        <h1>Bookstore</h1>
        <nav aria-label="Main">
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/books">Books</a></li>
            <li><a href="/authors">Authors</a></li>
          </ul>
        </nav>
      </header>

      <main id="main-content" tabIndex={-1}>
        <h2>Featured Books</h2>
        <section aria-labelledby="fiction-heading">
          <h3 id="fiction-heading">Fiction</h3>
          <ul>...</ul>
        </section>
        <section aria-labelledby="nonfiction-heading">
          <h3 id="nonfiction-heading">Non-Fiction</h3>
          <ul>...</ul>
        </section>
      </main>

      <footer>
        <nav aria-label="Footer">
          <ul>
            <li><a href="/privacy">Privacy</a></li>
            <li><a href="/terms">Terms</a></li>
          </ul>
        </nav>
      </footer>
    </>
  );
}
```

**Expected Output:** A bookstore page with a skip link, header with navigation, main content with two sections (Fiction and Non-Fiction), and a footer with navigation. Screen readers can navigate by landmarks (header, main, footer, two navs) and by headings (h1, h2, h3, h3).

**Why This Output Occurs:** The `<header>`, `<nav>`, `<main>`, `<section>`, and `<footer>` elements create landmarks. The `<h1>`, `<h2>`, and `<h3>` elements create a heading outline. The skip link lets keyboard users jump to `#main-content`, bypassing the header navigation.

### Real-World Cases

- **SaaS dashboards:** Header with nav, main with widgets, aside with filters.
- **E-commerce:** Header with cart, main with product grid, footer with links.
- **Documentation sites:** Header with search, aside with sidebar nav, main with article.
- **Admin panels:** Header with user menu, nav with sections, main with data tables.

### References

- W3C WAI-ARIA Authoring Practices – Landmark Regions: https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/
- MDN Web Docs – ARIA Landmarks: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/landmark_role
- WCAG 2.2 – Info and Relationships (1.3.1): https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html
- WCAG 2.2 – Bypass Blocks (2.4.1): https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html
- HTML Living Standard – Sections: https://html.spec.whatwg.org/multipage/sections.html

---

## Core Concept 2: Accessible Labels & Names

### Definitions

**Core Definition:** Accessible names are the programmatic labels that assistive technology uses to identify interactive elements; they are computed from visible text, `aria-label`, `aria-labelledby`, or associated `<label>` elements.

**Technical Definition:** The accessible name computation is a W3C algorithm that determines the name of an element from multiple sources in a defined priority order: (1) `aria-labelledby` (referencing other elements), (2) `aria-label` (a string), (3) native label (e.g., `<label for>`, `<caption>`), (4) content (visible text), and (5) `title` (fallback). The accessible description is computed similarly from `aria-describedby` or `title`. Using the wrong technique (e.g., `aria-label` on a `<div>` that should be a `<button>`) results in an element that is announced but not operable. For compound elements like fieldsets, `<legend>` provides the group name.

**Beginner-Friendly Explanation:** Every button, link, and input needs a name that screen readers can announce. If you have a button with an icon and no text, a screen reader says "button" with no context—useless. `aria-label="Close"` fixes that. If the label is already visible on the page (like a form label), you can use `aria-labelledby` to point to it. `aria-describedby` is for additional information (like help text or error messages). The rule is simple: if it is interactive, it must have a name.

### Purposes

- To give every interactive element a programmatic name that screen readers announce.
- To associate form inputs with their visible labels.
- To provide additional descriptions (help text, error messages) via `aria-describedby`.
- To label groups of related controls with `<fieldset>` and `<legend>`.
- To provide context for icon-only buttons.

### Syntax Rules and Structure

**`aria-label` (Direct String):**
```jsx
<button aria-label="Close dialog">
  <svg aria-hidden="true">...</svg>
</button>
```

**`aria-labelledby` (Reference to Visible Text):**
```jsx
<h2 id="dialog-title">Confirm Deletion</h2>
<div role="dialog" aria-labelledby="dialog-title">
  ...
</div>
```

**`aria-describedby` (Additional Description):**
```jsx
<label htmlFor="password">Password</label>
<input id="password" type="password" aria-describedby="password-hint" />
<p id="password-hint">At least 8 characters, one uppercase, one number.</p>
```

**`<fieldset>` and `<legend>` (Grouping):**
```jsx
<fieldset>
  <legend>Shipping Address</legend>
  <label>Street <input type="text" /></label>
  <label>City <input type="text" /></label>
</fieldset>
```

**Accessible Name Priority:**
```
1. aria-labelledby (highest priority)
2. aria-label
3. Native label (<label for>, <caption>, <legend>)
4. Visible content (text inside the element)
5. title attribute (fallback, lowest priority)
```

**Syntax Rules:**
- Use `aria-label` when there is no visible text (icon-only buttons, search inputs).
- Use `aria-labelledby` when there is visible text elsewhere on the page that should be the label.
- Use `aria-describedby` for additional information (hints, errors, formatting rules).
- Use `<label htmlFor>` for form inputs whenever possible.
- Use `<fieldset>` and `<legend>` to group related inputs (radio groups, address fields).
- Mark decorative icons with `aria-hidden="true"` and `focusable="false"`.
- Never use `aria-label` on a `<div>` or `<span>` that is not interactive.
- Never put visible text in `aria-label` that contradicts the visible label (WCAG 2.5.3 Label in Name).

**Constraints and Limitations:**
- `aria-labelledby` can reference multiple IDs (space-separated), concatenating them.
- `aria-label` overrides visible text; if the visible text differs, voice-control users cannot activate the element by speaking the visible text (WCAG 2.5.3).
- `aria-describedby` is announced after a pause; do not put critical information only in the description.
- `title` is not reliably announced by screen readers; do not rely on it.
- Empty or whitespace-only `aria-label` disables the name computation.

### Annotated Code Example: Accessible Search Form

```jsx
function SearchForm() {
  return (
    <form role="search" aria-label="Site search">
      <label htmlFor="search-input">Search books</label>
      <input
        id="search-input"
        type="search"
        aria-describedby="search-hint"
        placeholder="Title, author, or ISBN"
      />
      <p id="search-hint">Use quotes for exact matches.</p>
      <button type="submit" aria-label="Submit search">
        <svg aria-hidden="true" focusable="false" width="16" height="16">
          <path d="..." />
        </svg>
      </button>
    </form>
  );
}
```

**Expected Output:** A search form with a visible label "Search books," a search input with a hint, and a submit button with a magnifying-glass icon. Screen readers announce "Search books, edit text, Use quotes for exact matches" when the input is focused, and "Submit search, button" when the button is focused.

**Why This Output Occurs:** The `<label htmlFor="search-input">` provides the input's name. `aria-describedby="search-hint"` links the input to the hint paragraph, which is announced after the name. The button has no visible text, so `aria-label="Submit search"` provides its name. The SVG is hidden with `aria-hidden="true"`.

### Real-World Cases

- **Icon buttons:** Close, menu, search, favourite, share.
- **Form inputs:** Email, password, address, phone.
- **Dialog titles:** `aria-labelledby` pointing to the dialog's `<h2>`.
- **Radio groups:** `<fieldset>` + `<legend>` for the group name.
- **Data tables:** `<caption>` for the table's accessible name.
- **Navigation:** `aria-label` to distinguish "Main" from "Footer" navigation.

### References

- W3C – Accessible Name and Description Computation 1.2: https://www.w3.org/TR/accname-1.2/
- MDN Web Docs – `aria-label`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label
- MDN Web Docs – `aria-labelledby`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-labelledby
- MDN Web Docs – `aria-describedby`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby
- WCAG 2.2 – Label in Name (2.5.3): https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html
- W3C WAI – Labeling Controls: https://www.w3.org/WAI/tutorials/forms/labels/

---

## Core Concept 3: Advanced Focus Management

### Definitions

**Core Definition:** Advanced focus management is the programmatic control of keyboard focus—moving it to specific elements on mount, trapping it within modal overlays, and providing skip links—so that keyboard and screen-reader users can navigate the application predictably.

**Technical Definition:** Focus management involves using refs (`useRef`) and DOM APIs (`element.focus()`, `element.blur()`) to move focus in response to component lifecycle events. Key patterns include: **focus on mount** (move focus to a dialog or error summary when it opens), **focus restoration** (return focus to the trigger when a modal closes), **focus trapping** (cycle Tab and Shift+Tab within a modal), **skip links** (a visually-hidden link that becomes visible on focus and jumps to the main content), and **roving tabindex** (a single tab stop for a group of related controls, navigated with arrow keys). Focus must always be visible (WCAG 2.4.7), and `:focus-visible` should be used to show focus rings only for keyboard users.

**Beginner-Friendly Explanation:** Focus is like a cursor for keyboard users. When a modal opens, focus should move into it—otherwise, the user has to Tab through the entire page behind the modal. When the modal closes, focus should return to the button that opened it, so the user does not lose their place. A skip link lets keyboard users bypass the navigation. These patterns are not optional—they are the difference between a usable app and a frustrating one for keyboard users.

### Purposes

- To move focus into modals, dialogs, and error summaries when they open.
- To restore focus to the trigger when an overlay closes.
- To trap focus within modals so users do not tab behind them.
- To provide a skip link that jumps to the main content.
- To implement roving tabindex for tabs, toolbars, and menus.
- To ensure focus is always visible and never lost.

### Syntax Rules and Structure

**Focus on Mount:**
```jsx
function Dialog({ isOpen, onClose }) {
  const dialogRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      dialogRef.current?.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div ref={dialogRef} role="dialog" aria-modal="true" tabIndex={-1}>
      ...
    </div>
  );
}
```

**Focus Restoration:**
```jsx
function Modal({ isOpen, onClose, children }) {
  const triggerRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement;
    } else {
      previousFocusRef.current?.focus();
    }
  }, [isOpen]);

  return (
    <>
      <button ref={triggerRef} onClick={() => onClose()}>Open</button>
      {isOpen && <div role="dialog">...</div>}
    </>
  );
}
```

**Focus Trap (Tab Cycling):**
```jsx
const FOCUSABLE = [
  'a[href]',
  'button:not([disabled])',
  'textarea:not([disabled])',
  'input:not([disabled])',
  'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(',');

function useFocusTrap(ref, isActive) {
  useEffect(() => {
    if (!isActive) return;

    function handleKeyDown(e) {
      if (e.key !== 'Tab') return;

      const focusable = ref.current?.querySelectorAll(FOCUSABLE);
      if (!focusable?.length) return;

      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [ref, isActive]);
}
```

**Skip Link:**
```jsx
<a href="#main-content" className="skip-link">
  Skip to main content
</a>
<main id="main-content" tabIndex={-1}>
  ...
</main>
```

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px 16px;
  z-index: 1000;
}

.skip-link:focus {
  top: 0;
}
```

**Roving Tabindex:**
```jsx
function TabList({ tabs, activeIndex, onSelect }) {
  const buttonRefs = useRef([]);

  function handleKeyDown(e) {
    let nextIndex = activeIndex;
    if (e.key === 'ArrowRight') nextIndex = (activeIndex + 1) % tabs.length;
    else if (e.key === 'ArrowLeft') nextIndex = (activeIndex - 1 + tabs.length) % tabs.length;
    else if (e.key === 'Home') nextIndex = 0;
    else if (e.key === 'End') nextIndex = tabs.length - 1;
    else return;

    e.preventDefault();
    onSelect(nextIndex);
    buttonRefs.current[nextIndex]?.focus();
  }

  return (
    <div role="tablist" onKeyDown={handleKeyDown}>
      {tabs.map((tab, i) => (
        <button
          key={tab.id}
          ref={(el) => (buttonRefs.current[i] = el)}
          role="tab"
          tabIndex={i === activeIndex ? 0 : -1}
          aria-selected={i === activeIndex}
          onClick={() => onSelect(i)}
        >
          {tab.label}
        </button>
      ))}
    </div>
  );
}
```

**Syntax Rules:**
- Use `tabIndex={-1}` to make an element programmatically focusable without adding it to the tab order.
- Always restore focus to the trigger when a modal closes.
- Trap focus within modals; do not allow Tab to escape to the background.
- Use `:focus-visible` for focus rings so they appear only for keyboard users.
- Never remove the focus outline without providing a visible replacement (WCAG 2.4.7).
- Skip links must be the first focusable element on the page.
- Use roving tabindex for groups of related controls (tabs, toolbars, radio groups).

**Constraints and Limitations:**
- Manual focus trapping is error-prone; use a library (focus-trap-react, Radix Dialog) or the native `<dialog>`.
- `document.activeElement` may be `body` if nothing is focused.
- Focus management does not work on elements with `display: none`; ensure the target is visible first.
- Focus restoration can fail if the trigger element has been removed from the DOM.
- Skip links require CSS to hide and reveal on focus; `display: none` prevents focus.

### Annotated Code Example: Accessible Modal with Focus Trap

```jsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';

const FOCUSABLE = [
  'a[href]', 'button:not([disabled])', 'textarea:not([disabled])',
  'input:not([disabled])', 'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(',');

export function Modal({ isOpen, onClose, title, children }) {
  const dialogRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (!isOpen) return;

    previousFocusRef.current = document.activeElement;
    dialogRef.current?.focus();

    function handleKeyDown(e) {
      if (e.key === 'Escape') { onClose(); return; }
      if (e.key !== 'Tab') return;

      const focusable = dialogRef.current?.querySelectorAll(FOCUSABLE);
      if (!focusable?.length) return;

      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault(); last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault(); first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      previousFocusRef.current?.focus();
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

**Expected Output:** A modal dialog that, when opened, receives focus. Tab cycles within the modal (focus trap). Escape closes the modal. When closed, focus returns to the element that opened it. Screen readers announce the dialog's title.

**Why This Output Occurs:** `previousFocusRef` captures the trigger element. `dialogRef.current.focus()` moves focus into the dialog. The `handleKeyDown` function implements the Tab cycle. `previousFocusRef.current.focus()` on cleanup restores focus. The `role="dialog"`, `aria-modal="true"`, and `aria-labelledby` provide the correct semantics.

### Real-World Cases

- **Modals and dialogs:** Focus trap and restoration.
- **Error summaries:** Move focus to the summary on form submission failure.
- **Skip links:** Bypass navigation for keyboard users.
- **Tabs and toolbars:** Roving tabindex for arrow-key navigation.
- **Route changes:** Move focus to the new page's `<h1>` or main content.

### References

- W3C WAI-ARIA Authoring Practices – Dialog (Modal) Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- W3C WAI – Focus Visible (WCAG 2.4.7): https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html
- MDN Web Docs – `HTMLElement.focus()`: https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus
- React – Manipulating the DOM with Refs: https://react.dev/learn/manipulating-the-dom-with-refs
- focus-trap-react – npm: https://www.npmjs.com/package/focus-trap-react
- Radix UI – Dialog: https://www.radix-ui.com/primitives/docs/components/dialog

---

## Core Concept 4: Keyboard Navigation Patterns

### Definitions

**Core Definition:** Keyboard navigation patterns are the physical interaction models—`tabIndex`, `onKeyDown`, key matching—that make custom React components operable via keyboard.

**Technical Definition:** Native HTML elements (`<button>`, `<a>`, `<input>`) have built-in keyboard behaviour: Tab to focus, Enter/Space to activate. Custom components (divs styled as buttons, custom dropdowns, drag-and-drop interfaces) must replicate this behaviour manually. The `tabIndex` attribute controls tab order: `tabIndex={0}` adds an element to the natural tab order, `tabIndex={-1}` makes it programmatically focusable but removes it from the tab order, and positive values override the natural order (discouraged). The `onKeyDown` handler receives a `KeyboardEvent` with `key` (e.g., `"Enter"`, `"ArrowDown"`, `"Escape"`), `shiftKey`, `ctrlKey`, `altKey`, and `metaKey`. Key matching should use `event.key` (the character or key name) rather than `event.keyCode` (deprecated). WAI-ARIA patterns define the expected keys for each component type.

**Beginner-Friendly Explanation:** Keyboard navigation is how people who cannot use a mouse interact with your app. Native HTML elements handle this for free—a `<button>` responds to Enter and Space, a link responds to Enter. But if you build a custom dropdown with `<div>` and `<span>`, you must add the keyboard handling yourself: arrow keys to navigate, Enter to select, Escape to close. The WAI-ARIA Authoring Practices define exactly which keys each component should respond to—follow them.

### Purposes

- To make custom components operable without a mouse.
- To follow the WAI-ARIA keyboard conventions for each component type.
- To provide a predictable tab order that matches the visual order.
- To support arrow-key navigation within groups of related controls.
- To handle Escape, Enter, Space, Home, and End consistently.

### Syntax Rules and Structure

**`tabIndex` Values:**

| Value | Behaviour | Use When |
|---|---|---|
| `tabIndex={0}` | Focusable via Tab, in natural order | Custom interactive element (should be a `<button>` instead) |
| `tabIndex={-1}` | Focusable programmatically, not via Tab | Modal container, error summary, skip-link target |
| `tabIndex={1+}` | Overrides natural order | Almost never; avoid |
| No `tabIndex` | Native behaviour | `<button>`, `<a>`, `<input>`, etc. |

**Key Matching:**
```jsx
function handleKeyDown(e) {
  switch (e.key) {
    case 'Enter':
    case ' ':
      e.preventDefault();
      activate();
      break;
    case 'Escape':
      close();
      break;
    case 'ArrowDown':
      e.preventDefault();
      focusNext();
      break;
    case 'ArrowUp':
      e.preventDefault();
      focusPrevious();
      break;
    case 'Home':
      e.preventDefault();
      focusFirst();
      break;
    case 'End':
      e.preventDefault();
      focusLast();
      break;
    case 'Tab':
      // Let the browser handle Tab (or implement a focus trap)
      break;
  }
}
```

**Custom Button (Better: Use `<button>`):**
```jsx
// ❌ Div pretending to be a button — keyboard inaccessible by default
<div className="btn" onClick={handleClick}>Click me</div>

// ✅ Use a real button
<button className="btn" onClick={handleClick}>Click me</button>

// ⚠️ If you must use a div, add role, tabIndex, and keyboard handlers
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
>
  Click me
</div>
```

**Arrow-Key Navigation in a Menu:**
```jsx
function Menu({ items, onSelect }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const itemRefs = useRef([]);

  function handleKeyDown(e) {
    let nextIndex = activeIndex;
    if (e.key === 'ArrowDown') nextIndex = (activeIndex + 1) % items.length;
    else if (e.key === 'ArrowUp') nextIndex = (activeIndex - 1 + items.length) % items.length;
    else if (e.key === 'Home') nextIndex = 0;
    else if (e.key === 'End') nextIndex = items.length - 1;
    else if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onSelect(items[activeIndex]);
      return;
    } else return;

    e.preventDefault();
    setActiveIndex(nextIndex);
    itemRefs.current[nextIndex]?.focus();
  }

  return (
    <ul role="menu" onKeyDown={handleKeyDown}>
      {items.map((item, i) => (
        <li
          key={item.id}
          ref={(el) => (itemRefs.current[i] = el)}
          role="menuitem"
          tabIndex={-1}
        >
          {item.label}
        </li>
      ))}
    </ul>
  );
}
```

**Syntax Rules:**
- Use native elements (`<button>`, `<a>`, `<input>`) whenever possible; they handle keyboard events automatically.
- Use `event.key` for key matching, not `event.keyCode` (deprecated).
- Call `e.preventDefault()` only when you handle the key; do not block Tab or browser shortcuts.
- Use `tabIndex={-1}` for elements focused via arrow keys (roving tabindex).
- Implement arrow keys for groups of related controls (menus, tabs, radio groups).
- Implement Home/End for long lists.
- Implement Escape for closing overlays.
- Never remove the focus outline; use `:focus-visible` for custom styles.

**Constraints and Limitations:**
- `event.key` values are locale-dependent for character keys but consistent for named keys (`"Enter"`, `"Escape"`, `"ArrowDown"`).
- Positive `tabIndex` values disrupt the natural order and confuse screen-reader users; avoid.
- Key combinations (Ctrl+S, Cmd+K) must respect browser shortcuts; only override with care.
- Mobile devices with external keyboards follow the same conventions.
- Some keys (e.g., `"Space"`) are represented as `" "` (space character), not `"Space"`.

### Annotated Code Example: Keyboard-Accessible Accordion

```jsx
import { useState, useRef } from 'react';

export function Accordion({ items }) {
  const [openItems, setOpenItems] = useState([]);
  const buttonRefs = useRef([]);

  function toggle(value) {
    setOpenItems((prev) =>
      prev.includes(value) ? prev.filter((v) => v !== value) : [...prev, value]
    );
  }

  function handleKeyDown(e, index) {
    const buttons = buttonRefs.current;
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      buttons[(index + 1) % buttons.length]?.focus();
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      buttons[(index - 1 + buttons.length) % buttons.length]?.focus();
    } else if (e.key === 'Home') {
      e.preventDefault();
      buttons[0]?.focus();
    } else if (e.key === 'End') {
      e.preventDefault();
      buttons[buttons.length - 1]?.focus();
    }
  }

  return (
    <div>
      {items.map((item, i) => {
        const isOpen = openItems.includes(item.value);
        return (
          <div key={item.value}>
            <h3>
              <button
                ref={(el) => (buttonRefs.current[i] = el)}
                aria-expanded={isOpen}
                aria-controls={`panel-${item.value}`}
                onClick={() => toggle(item.value)}
                onKeyDown={(e) => handleKeyDown(e, i)}
              >
                {item.title}
              </button>
            </h3>
            <div id={`panel-${item.value}`} hidden={!isOpen}>
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

**Expected Output:** An accordion where each header is a real `<button>`. Enter/Space toggles the panel. Arrow Up/Down moves focus between headers. Home/End jumps to the first/last header. Screen readers announce the expanded state.

**Why This Output Occurs:** The `<button>` element provides Enter/Space handling automatically. The `onKeyDown` handler adds Arrow/Home/End navigation. `aria-expanded` and `aria-controls` link the button to its panel. Focus moves programmatically via `buttonRefs.current[i].focus()`.

### Real-World Cases

- **Menus and dropdowns:** Arrow keys, Enter, Escape.
- **Tabs:** Arrow keys, Home/End, Tab into panel.
- **Accordions:** Enter/Space to toggle, Arrow Up/Down to navigate.
- **Sliders:** Arrow keys to increment/decrement, Home/End for min/max.
- **Date pickers:** Arrow keys to navigate the calendar grid.
- **Drag-and-drop:** Keyboard alternatives (move up/down/left/right).

### References

- W3C WAI-ARIA Authoring Practices – Keyboard Interface: https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- MDN Web Docs – KeyboardEvent.key: https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key
- MDN Web Docs – tabindex: https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex
- MDN Web Docs – `:focus-visible`: https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible
- WCAG 2.2 – Keyboard (2.1.1): https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html

---

## Core Concept 5: Dynamic Content Announcements

### Definitions

**Core Definition:** Dynamic content announcements are the use of `aria-live` regions to notify screen-reader users when content changes without a page reload—form errors, search results, notifications, and async updates.

**Technical Definition:** The `aria-live` attribute marks a region as a live region, which screen readers monitor for changes and announce automatically. Values are `aria-live="polite"` (announce when the user is idle) and `aria-live="assertive"` (announce immediately, interrupting the user). The `role="alert"` attribute is equivalent to `aria-live="assertive"` + `role="alert"`. The `role="status"` attribute is equivalent to `aria-live="polite"`. The `aria-atomic` attribute controls whether the entire region or only the changed node is announced. Live regions must be present in the DOM *before* the change occurs; dynamically inserting a live region with content may not trigger an announcement. React's rendering model requires care: mount the live region empty, then update its content.

**Beginner-Friendly Explanation:** When you click "Submit" and a form error appears, sighted users see it immediately. Screen-reader users hear nothing—unless you use a live region. `aria-live="polite"` says "announce this when you have a moment"; `aria-live="assertive"` says "announce this immediately." Use polite for search results and status updates; use assertive (or `role="alert"`) for errors that need immediate attention. The tricky part: the live region must exist on the page before the content changes, or the announcement may not happen.

### Purposes

- To announce form validation errors to screen-reader users.
- To announce search results, filter changes, or pagination updates.
- To announce async operation results (save success, upload complete).
- To announce cart updates, notification counts, or status changes.
- To announce loading states and background activity.
- To replace visual-only feedback with programmatic announcements.

### Syntax Rules and Structure

**Polite Live Region (`role="status"`):**
```jsx
function SearchResults({ results, isSearching }) {
  return (
    <div>
      <div role="status" aria-live="polite" aria-atomic="true">
        {isSearching
          ? 'Searching...'
          : `${results.length} results found.`}
      </div>
      <ul>{results.map((r) => <li key={r.id}>{r.name}</li>)}</ul>
    </div>
  );
}
```

**Assertive Live Region (`role="alert"`):**
```jsx
function FormErrors({ errors }) {
  if (!errors.length) return null;
  return (
    <div role="alert" aria-live="assertive" aria-atomic="true">
      {errors.length} error{errors.length > 1 ? 's' : ''}: {errors.join(', ')}
    </div>
  );
}
```

**Field-Level Error with `aria-describedby`:**
```jsx
function EmailField({ error }) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>Email</label>
      <input
        id={id}
        type="email"
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
      />
      {error && (
        <p id={errorId} role="alert" className="error">
          {error}
        </p>
      )}
    </div>
  );
}
```

**Async Save Status:**
```jsx
function SaveButton() {
  const [status, setStatus] = useState('idle');

  async function handleSave() {
    setStatus('saving');
    try {
      await save();
      setStatus('saved');
    } catch {
      setStatus('error');
    }
  }

  return (
    <div>
      <button onClick={handleSave} disabled={status === 'saving'}>
        {status === 'saving' ? 'Saving...' : 'Save'}
      </button>
      <div role="status" aria-live="polite">
        {status === 'saved' && 'Changes saved successfully.'}
        {status === 'error' && 'Failed to save. Please try again.'}
      </div>
    </div>
  );
}
```

**Syntax Rules:**
- Use `aria-live="polite"` (or `role="status"`) for non-critical updates (search results, save status).
- Use `aria-live="assertive"` (or `role="alert"`) for critical updates (errors, alerts).
- The live region must be present in the DOM before the content changes.
- Use `aria-atomic="true"` to announce the entire region, not just the changed text.
- Do not put interactive elements inside a live region; screen readers may not announce them correctly.
- Do not over-announce; frequent updates can be overwhelming.
- For form errors, combine `role="alert"` with `aria-describedby` on the input.

**Constraints and Limitations:**
- Dynamically inserted live regions (with content) may not trigger an announcement; render the region empty first.
- `aria-live="assertive"` interrupts the user; use sparingly.
- Live regions are not announced on page load; only changes after mount.
- Some screen readers have quirks with live regions in React's concurrent rendering; test with real screen readers.
- The `role="alert"` announcement may be delayed or interrupted by other announcements.

### Annotated Code Example: Search with Live Results

```jsx
import { useState, useMemo } from 'react';

export function SearchPage() {
  const [query, setQuery] = useState('');
  const [isSearching, setIsSearching] = useState(false);

  const results = useMemo(() => {
    if (!query) return [];
    setIsSearching(true);
    const filtered = ALL_ITEMS.filter((item) =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
    setIsSearching(false);
    return filtered;
  }, [query]);

  return (
    <div>
      <label htmlFor="search">Search products</label>
      <input
        id="search"
        type="search"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />

      <div role="status" aria-live="polite" aria-atomic="true">
        {query && !isSearching && `${results.length} results for "${query}".`}
      </div>

      <ul>
        {results.map((item) => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

**Expected Output:** As the user types, the status region announces "N results for 'query'." The results list updates visually. Screen-reader users hear the count without losing focus on the search input.

**Why This Output Occurs:** The `role="status"` region is present in the DOM from the start. When the query changes and results are computed, the region's text updates, triggering the announcement. `aria-atomic="true"` ensures the whole message is announced.

### Real-World Cases

- **Form validation:** Announcing error counts and field-specific errors.
- **Search:** Announcing result counts and "no results."
- **Shopping cart:** Announcing item added, quantity updated.
- **Async operations:** Announcing save success, upload progress, connection lost.
- **Chat:** Announcing new messages.
- **Notifications:** Announcing unread counts.

### References

- W3C WAI-ARIA – `aria-live`: https://www.w3.org/TR/wai-aria-1.2/#aria-live
- MDN Web Docs – ARIA Live Regions: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions
- MDN Web Docs – `role="alert"`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/alert_role
- MDN Web Docs – `role="status"`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/status_role
- W3C WAI – Dynamic Content: https://www.w3.org/WAI/tutorials/

---

## Core Concept 6: Form & Interaction Patterns

### Definitions

**Core Definition:** Form and interaction patterns are the accessible structures—labels, fieldsets, error messaging, and multi-step flows—that make React forms usable by keyboard and screen-reader users.

**Technical Definition:** Accessible forms combine semantic HTML (`<form>`, `<label>`, `<input>`, `<fieldset>`, `<legend>`, `<button>`), programmatic relationships (`htmlFor`/`id`, `aria-describedby`, `aria-invalid`), error announcement (`role="alert"`, `aria-live`), and focus management (moving focus to the first invalid field on submission failure). Multi-step forms (wizards) require preserving state across steps, validating only the current step, and announcing step changes. React Hook Form, Formik, and React 19's Actions provide structured APIs; `useId` generates stable IDs. Field grouping with `<fieldset>` and `<legend>` is essential for radio groups and related checkboxes. Form-level error summaries (with links to fields) are recommended for long forms.

**Beginner-Friendly Explanation:** A form is accessible when a screen-reader user can (1) understand what each field is for, (2) know when a field is required, (3) hear error messages when validation fails, (4) navigate between fields with the keyboard, and (5) know when the form has been submitted. This means every input needs a label, error messages need to be announced, focus must move to the first error, and multi-step forms must announce the current step and preserve data.

### Purposes

- To give every form field a programmatic label.
- To group related fields with `<fieldset>` and `<legend>`.
- To announce validation errors to screen readers.
- To move focus to the first error on submission failure.
- To preserve data across multi-step forms.
- To announce step changes in wizards.
- To provide an error summary for long forms.

### Syntax Rules and Structure

**Labeled Input with Error:**
```jsx
function EmailField({ register, error }) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>
        Email <span aria-hidden="true">*</span>
      </label>
      <input
        id={id}
        type="email"
        required
        aria-required="true"
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
        {...register('email')}
      />
      {error && (
        <p id={errorId} role="alert" className="error">
          {error.message}
        </p>
      )}
    </div>
  );
}
```

**Radio Group with Fieldset:**
```jsx
function ShippingMethod({ register }) {
  return (
    <fieldset>
      <legend>Shipping Method</legend>
      <label>
        <input type="radio" value="standard" {...register('shipping')} />
        Standard (5–7 days)
      </label>
      <label>
        <input type="radio" value="express" {...register('shipping')} />
        Express (2–3 days)
      </label>
      <label>
        <input type="radio" value="overnight" {...register('shipping')} />
        Overnight
      </label>
    </fieldset>
  );
}
```

**Form-Level Error Summary:**
```jsx
function ErrorSummary({ errors }) {
  const ref = useRef(null);

  useEffect(() => {
    if (Object.keys(errors).length > 0) {
      ref.current?.focus();
    }
  }, [errors]);

  if (Object.keys(errors).length === 0) return null;

  return (
    <div
      ref={ref}
      role="alert"
      tabIndex={-1}
      aria-labelledby="error-summary-title"
    >
      <h2 id="error-summary-title">
        There {Object.keys(errors).length === 1 ? 'is' : 'are'}{' '}
        {Object.keys(errors).length} error
        {Object.keys(errors).length > 1 ? 's' : ''}
      </h2>
      <ul>
        {Object.entries(errors).map(([field, error]) => (
          <li key={field}>
            <a href={`#${field}`}>{error.message}</a>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Multi-Step Form (Wizard):**
```jsx
function Wizard({ steps }) {
  const [step, setStep] = useState(0);
  const methods = useForm({ mode: 'onBlur' });

  async function next() {
    const valid = await methods.trigger(steps[step].fields);
    if (valid) setStep((s) => s + 1);
  }

  return (
    <FormProvider {...methods}>
      <form>
        <ol aria-label="Progress">
          {steps.map((s, i) => (
            <li key={s.id} aria-current={i === step ? 'step' : undefined}>
              {s.label}
            </li>
          ))}
        </ol>

        <div role="status" aria-live="polite" className="sr-only">
          Step {step + 1} of {steps.length}: {steps[step].label}
        </div>

        {steps.map((s, i) => (
          <div key={s.id} style={{ display: i === step ? 'block' : 'none' }}>
            <s.component />
          </div>
        ))}

        <div>
          {step > 0 && <button type="button" onClick={() => setStep((s) => s - 1)}>Back</button>}
          {step < steps.length - 1 && <button type="button" onClick={next}>Next</button>}
          {step === steps.length - 1 && <button type="submit">Submit</button>}
        </div>
      </form>
    </FormProvider>
  );
}
```

**Syntax Rules:**
- Every input must have a `<label htmlFor>` or `aria-label`.
- Mark required fields with `required` and `aria-required="true"`.
- Mark invalid fields with `aria-invalid="true"` and link errors with `aria-describedby`.
- Use `role="alert"` on error messages so they are announced.
- Group related inputs with `<fieldset>` and `<legend>`.
- Move focus to the first invalid field (or error summary) on submission failure.
- Preserve form state across steps by keeping it in the parent.
- Validate only the current step's fields before advancing.
- Announce step changes with a live region.

**Constraints and Limitations:**
- `aria-required` and `required` should both be set for maximum compatibility.
- Error summaries are essential for long forms; inline errors alone are insufficient.
- Multi-step forms must keep all steps mounted (or lift state) to preserve data.
- Disabled submit buttons are not focusable; use `aria-disabled` and prevent submission instead.
- Native browser validation messages are not fully accessible; use custom validation with ARIA.

### Annotated Code Example: Accessible Registration Form

```jsx
import { useForm } from 'react-hook-form';
import { useId, useRef, useEffect } from 'react';

export function RegistrationForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({ mode: 'onBlur' });
  const summaryRef = useRef(null);
  const errorCount = Object.keys(errors).length;

  useEffect(() => {
    if (errorCount > 0) summaryRef.current?.focus();
  }, [errorCount]);

  const emailId = useId();
  const passwordId = useId();
  const termsId = useId();

  return (
    <form onSubmit={handleSubmit((data) => alert(JSON.stringify(data)))} noValidate>
      {errorCount > 0 && (
        <div ref={summaryRef} role="alert" tabIndex={-1}>
          <h2>{errorCount} error{errorCount > 1 ? 's' : ''} found</h2>
          <ul>
            {Object.entries(errors).map(([field, err]) => (
              <li key={field}><a href={`#${field}`}>{err.message}</a></li>
            ))}
          </ul>
        </div>
      )}

      <div>
        <label htmlFor={emailId}>Email <span aria-hidden="true">*</span></label>
        <input
          id={emailId}
          type="email"
          aria-required="true"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? `${emailId}-error` : undefined}
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /@/, message: 'Invalid email' },
          })}
        />
        {errors.email && (
          <p id={`${emailId}-error`} role="alert">{errors.email.message}</p>
        )}
      </div>

      <div>
        <label htmlFor={passwordId}>Password <span aria-hidden="true">*</span></label>
        <input
          id={passwordId}
          type="password"
          aria-required="true"
          aria-invalid={!!errors.password}
          aria-describedby={`${passwordId}-hint ${errors.password ? `${passwordId}-error` : ''}`.trim() || undefined}
          {...register('password', {
            required: 'Password is required',
            minLength: { value: 8, message: 'At least 8 characters' },
          })}
        />
        <p id={`${passwordId}-hint`}>At least 8 characters.</p>
        {errors.password && (
          <p id={`${passwordId}-error`} role="alert">{errors.password.message}</p>
        )}
      </div>

      <div>
        <label>
          <input type="checkbox" {...register('terms', { required: 'You must accept the terms' })} />
          I accept the terms and conditions
        </label>
        {errors.terms && <p role="alert">{errors.terms.message}</p>}
      </div>

      <button type="submit">Create Account</button>
    </form>
  );
}
```

**Expected Output:** A registration form with email, password, and terms checkbox. Submitting with invalid data shows an error summary (with focus) and inline errors below each field. Screen readers announce the summary and each field's error.

**Why This Output Occurs:** `useId` generates stable IDs for `htmlFor` and `aria-describedby`. `aria-invalid` marks invalid fields. `role="alert"` announces errors. The error summary receives focus via `useEffect`. `aria-required` and `required` mark required fields. The `noValidate` attribute disables native browser validation so custom messages are used.

### Real-World Cases

- **Registration forms:** Email, password, terms.
- **Checkout flows:** Shipping address, payment, review.
- **Profile editing:** Name, bio, avatar.
- **Surveys:** Multiple choice, text, rating scales.
- **Multi-step wizards:** Onboarding, loan applications, insurance quotes.

### References

- W3C WAI – Forms Tutorial: https://www.w3.org/WAI/tutorials/forms/
- W3C WAI-ARIA – `aria-invalid`: https://www.w3.org/TR/wai-aria-1.2/#aria-invalid
- WCAG 2.2 – Error Identification (3.3.1): https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html
- WCAG 2.2 – Labels or Instructions (3.3.2): https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html
- WCAG 2.2 – Error Suggestion (3.3.3): https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html
- React Hook Form – Documentation: https://react-hook-form.com/

---

## Core Concept 7: Complex ARIA Components (APG Patterns)

### Definitions

**Core Definition:** Complex ARIA components are interactive widgets (disclosure, tabs, accordions, comboboxes, menus, dialogs, sliders, trees) that follow the WAI-ARIA Authoring Practices Guide (APG) patterns for roles, states, properties, and keyboard interaction.

**Technical Definition:** The WAI-ARIA Authoring Practices Guide defines canonical patterns for common widgets. Each pattern specifies: (1) the required ARIA roles (`role="tablist"`, `role="dialog"`, `role="combobox"`), (2) the required states and properties (`aria-expanded`, `aria-selected`, `aria-activedescendant`), (3) the keyboard interaction model (arrow keys, Enter, Escape, Home/End), and (4) the focus management strategy (roving tabindex, focus trap, `aria-activedescendant`). In React, these patterns are best implemented as compound components (`Tabs.List`, `Tabs.Tab`, `Tabs.Panel`) with context for shared state. Libraries like Radix UI, React Aria, and Headless UI implement these patterns correctly and are recommended over hand-rolling complex widgets.

**Beginner-Friendly Explanation:** The APG is a recipe book for accessible widgets. It tells you exactly which ARIA attributes to add, which keys to handle, and where focus should go. For example, a tab component needs `role="tablist"`, `role="tab"`, and `role="tabpanel"`; arrow keys move between tabs; Home/End jump to the first/last; and only the active tab is in the tab order. If you follow the recipe exactly, your widget will work with screen readers and keyboards. If you improvise, it probably will not.

### Purposes

- To implement widgets that work correctly with screen readers and keyboards.
- To follow established conventions that users already know.
- To avoid reinventing patterns and introducing bugs.
- To ensure consistency across applications and libraries.
- To pass automated and manual accessibility audits.

### Syntax Rules and Structure

**Disclosure (Show/Hide):**
```jsx
function Disclosure({ title, children }) {
  const [isOpen, setIsOpen] = useState(false);
  const id = useId();

  return (
    <div>
      <button
        aria-expanded={isOpen}
        aria-controls={`${id}-content`}
        onClick={() => setIsOpen((o) => !o)}
      >
        {title}
      </button>
      <div id={`${id}-content`} hidden={!isOpen}>
        {children}
      </div>
    </div>
  );
}
```

**Tabs:**
```jsx
function Tabs({ tabs, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  const baseId = useId();
  const refs = useRef([]);

  function handleKeyDown(e, index) {
    let next = index;
    if (e.key === 'ArrowRight') next = (index + 1) % tabs.length;
    else if (e.key === 'ArrowLeft') next = (index - 1 + tabs.length) % tabs.length;
    else if (e.key === 'Home') next = 0;
    else if (e.key === 'End') next = tabs.length - 1;
    else return;

    e.preventDefault();
    setActiveTab(tabs[next].id);
    refs.current[next]?.focus();
  }

  return (
    <div>
      <div role="tablist" aria-label="Content sections">
        {tabs.map((tab, i) => (
          <button
            key={tab.id}
            ref={(el) => (refs.current[i] = el)}
            role="tab"
            id={`${baseId}-tab-${tab.id}`}
            aria-selected={activeTab === tab.id}
            aria-controls={`${baseId}-panel-${tab.id}`}
            tabIndex={activeTab === tab.id ? 0 : -1}
            onClick={() => setActiveTab(tab.id)}
            onKeyDown={(e) => handleKeyDown(e, i)}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {tabs.map((tab) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`${baseId}-panel-${tab.id}`}
          aria-labelledby={`${baseId}-tab-${tab.id}`}
          tabIndex={0}
          hidden={activeTab !== tab.id}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

**Combobox (Autocomplete):**
```jsx
function Combobox({ options, onChange }) {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const listboxId = useId();

  const filtered = options.filter((o) =>
    o.label.toLowerCase().includes(query.toLowerCase())
  );

  function handleKeyDown(e) {
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      setIsOpen(true);
      setActiveIndex((i) => Math.min(i + 1, filtered.length - 1));
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      setActiveIndex((i) => Math.max(i - 1, 0));
    } else if (e.key === 'Enter') {
      e.preventDefault();
      if (activeIndex >= 0) {
        onChange(filtered[activeIndex]);
        setQuery(filtered[activeIndex].label);
        setIsOpen(false);
      }
    } else if (e.key === 'Escape') {
      setIsOpen(false);
    }
  }

  return (
    <div>
      <input
        role="combobox"
        aria-expanded={isOpen}
        aria-controls={listboxId}
        aria-activedescendant={activeIndex >= 0 ? `${listboxId}-opt-${activeIndex}` : undefined}
        aria-autocomplete="list"
        value={query}
        onChange={(e) => { setQuery(e.target.value); setIsOpen(true); }}
        onKeyDown={handleKeyDown}
      />
      {isOpen && (
        <ul id={listboxId} role="listbox">
          {filtered.map((option, i) => (
            <li
              key={option.id}
              id={`${listboxId}-opt-${i}`}
              role="option"
              aria-selected={i === activeIndex}
              onClick={() => { onChange(option); setQuery(option.label); setIsOpen(false); }}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Syntax Rules:**
- Follow the APG pattern exactly for each widget: roles, states, properties, keyboard, focus.
- Use `aria-activedescendant` for comboboxes and other widgets where focus stays on the input.
- Use roving tabindex for tabs, toolbars, and radio groups.
- Use `aria-expanded` on disclosure triggers.
- Use `aria-selected` on tabs and options.
- Use `aria-controls` to link a control to the element it controls.
- Use `useId` for stable IDs across SSR.
- Use a library (Radix, React Aria, Headless UI) for complex widgets; hand-rolling is error-prone.

**Constraints and Limitations:**
- Complex widgets are hard to implement correctly; libraries are recommended.
- ARIA attributes must be kept in sync with visual state; React's declarative model helps but requires care.
- `aria-activedescendant` requires the referenced element to exist in the DOM.
- `role="combobox"` has evolved; follow the latest APG guidance.
- Testing with real screen readers is essential; automated tools catch only some issues.

### Annotated Code Example: Accordion (APG Pattern)

```jsx
function Accordion({ items, allowMultiple = false }) {
  const [openItems, setOpenItems] = useState([]);
  const baseId = useId();

  function toggle(value) {
    setOpenItems((prev) => {
      if (prev.includes(value)) return prev.filter((v) => v !== value);
      return allowMultiple ? [...prev, value] : [value];
    });
  }

  return (
    <div>
      {items.map((item) => {
        const isOpen = openItems.includes(item.value);
        const buttonId = `${baseId}-trigger-${item.value}`;
        const panelId = `${baseId}-panel-${item.value}`;

        return (
          <div key={item.value}>
            <h3>
              <button
                id={buttonId}
                aria-expanded={isOpen}
                aria-controls={panelId}
                onClick={() => toggle(item.value)}
              >
                {item.title}
              </button>
            </h3>
            <div
              id={panelId}
              role="region"
              aria-labelledby={buttonId}
              hidden={!isOpen}
            >
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

**Expected Output:** An accordion following the APG pattern. Each trigger is a `<button>` inside an `<h3>`. `aria-expanded` and `aria-controls` link the button to its panel. The panel is a `role="region"` with `aria-labelledby`. Enter/Space toggles; Tab moves between triggers.

**Why This Output Occurs:** The APG accordion pattern requires: (1) a heading element wrapping a button, (2) `aria-expanded` on the button, (3) `aria-controls` linking to the panel, (4) the panel as a region with `aria-labelledby`. Each requirement is implemented.

### Real-World Cases

- **Design systems:** Component libraries (Radix, React Aria, Headless UI) implement APG patterns.
- **Dashboards:** Tabs, accordions, menus, and comboboxes.
- **E-commerce:** Autocomplete search, filter dropdowns, quantity steppers.
- **Documentation:** Collapsible navigation, tabbed code examples.
- **Admin panels:** Data tables with sortable headers, pagination, filters.

### References

- W3C WAI-ARIA Authoring Practices Guide – Patterns: https://www.w3.org/WAI/ARIA/apg/patterns/
- W3C WAI-ARIA APG – Disclosure: https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/
- W3C WAI-ARIA APG – Tabs: https://www.w3.org/WAI/ARIA/apg/patterns/tabs/
- W3C WAI-ARIA APG – Accordion: https://www.w3.org/WAI/ARIA/apg/patterns/accordion/
- W3C WAI-ARIA APG – Combobox: https://www.w3.org/WAI/ARIA/apg/patterns/combobox/
- W3C WAI-ARIA APG – Dialog: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- Radix UI – Primitives: https://www.radix-ui.com/primitives
- React Aria – Components: https://react-spectrum.adobe.com/react-aria/
- Headless UI – Components: https://headlessui.com/

---

## Core Concept 8: Accessible Routing

### Definitions

**Core Definition:** Accessible routing is the practice of announcing client-side page changes to screen readers and resetting focus to the new page's content, since SPA navigation does not trigger a browser page load.

**Technical Definition:** In a single-page application, clicking a link does not reload the document; the URL changes and React renders new components, but the browser does not announce the change, and focus remains where it was (often on the clicked link or lost). Accessible routing addresses this by: (1) moving focus to the new page's `<h1>` or `<main>` element, (2) announcing the new page title in a live region, (3) resetting scroll position to the top, and (4) updating the document title. In React Router v7, `<ScrollRestoration />` handles scroll; focus and announcement must be implemented in a layout component using `useLocation` and `useEffect`. Next.js App Router handles some of this automatically; Remix has a `<LiveReload>` equivalent. The pattern must respect `prefers-reduced-motion` for scroll behaviour.

**Beginner-Friendly Explanation:** When you click a link on a traditional website, the browser loads a new page, announces the title, and puts focus at the top. In a React SPA, none of that happens automatically—the screen reader says nothing, focus stays on the link, and the user is left wondering if anything changed. Accessible routing fixes this: move focus to the new page's heading, announce the page title, and reset scroll. It is a small amount of code that makes a huge difference for screen-reader users.

### Purposes

- To announce page changes to screen-reader users.
- To move focus to the new page's main heading or content.
- To reset scroll position to the top on navigation.
- To update the document title for browser tabs and history.
- To avoid disorienting focus jumps or lost focus.
- To comply with WCAG 2.2 SC 2.4.2 (Page Titled) and SC 4.1.3 (Status Messages).

### Syntax Rules and Structure

**Focus and Announcement on Route Change (React Router v7):**
```jsx
import { useEffect, useRef } from 'react';
import { useLocation, Outlet } from 'react-router';

function RouteAnnouncer() {
  const location = useLocation();
  const mainRef = useRef(null);
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }

    // Move focus to the main content
    mainRef.current?.focus();

    // Announce the new page in a live region
    const heading = mainRef.current?.querySelector('h1');
    const title = heading?.textContent || document.title;
    const announcer = document.getElementById('route-announcer');
    if (announcer) {
      announcer.textContent = `Navigated to ${title}`;
    }
  }, [location.pathname]);

  return (
    <>
      <div
        id="route-announcer"
        role="status"
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"
      />
      <main ref={mainRef} id="main-content" tabIndex={-1}>
        <Outlet />
      </main>
    </>
  );
}
```

**Updating Document Title:**
```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    const previous = document.title;
    document.title = title;
    return () => { document.title = previous; };
  }, [title]);
}

function ProductPage({ product }) {
  useDocumentTitle(`${product.name} — My Store`);
  return <h1>{product.name}</h1>;
}
```

**Scroll Restoration (React Router v7):**
```jsx
import { ScrollRestoration } from 'react-router';

function RootLayout() {
  return (
    <>
      <header>...</header>
      <main id="main-content" tabIndex={-1}>
        <Outlet />
      </main>
      <footer>...</footer>
      <ScrollRestoration />
    </>
  );
}
```

**Next.js App Router (Automatic):**
```jsx
// Next.js App Router automatically handles focus, scroll, and title on navigation.
// For custom announcements, use a client component with usePathname:
'use client';
import { usePathname } from 'next/navigation';
import { useEffect, useRef } from 'react';

export function RouteAnnouncer() {
  const pathname = usePathname();
  const ref = useRef(null);

  useEffect(() => {
    ref.current.textContent = `Navigated to ${document.title}`;
  }, [pathname]);

  return <div ref={ref} role="status" aria-live="polite" className="sr-only" />;
}
```

**Syntax Rules:**
- Move focus to the `<main>` or `<h1>` of the new page on route change.
- Use `tabIndex={-1}` on the `<main>` element so it can receive focus.
- Announce the new page title via a `role="status"` or `aria-live="polite"` region.
- Update `document.title` on every route change.
- Reset scroll position to the top on new navigations; restore on back/forward.
- Skip the first render (initial page load) to avoid unnecessary focus jumps.
- Respect `prefers-reduced-motion` when scrolling.
- In React Router v7, use `<ScrollRestoration />` for scroll and a custom hook for focus/announcement.
- In Next.js App Router, focus and scroll are handled automatically; add announcements if needed.

**Constraints and Limitations:**
- Moving focus on every route change can be disorienting if overdone; use a `<main>` with `tabIndex={-1}` and a clear `<h1>`.
- Screen readers announce live regions asynchronously; the message may be delayed.
- Scroll restoration must not fight browser-native back/forward behaviour.
- Focus management may conflict with route-level focus traps (e.g., modals).
- Some routers handle this automatically (Next.js App Router); others require manual implementation (React Router, Wouter).

### Annotated Code Example: Complete Accessible Route Layout

```jsx
import { useEffect, useRef } from 'react';
import { Outlet, ScrollRestoration, useLocation } from 'react-router';

export function RootLayout() {
  const location = useLocation();
  const mainRef = useRef(null);
  const announcerRef = useRef(null);
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }

    // 1. Move focus to main content
    mainRef.current?.focus();

    // 2. Announce the new page
    const heading = mainRef.current?.querySelector('h1');
    const pageTitle = heading?.textContent?.trim() || document.title;
    if (announcerRef.current) {
      announcerRef.current.textContent = `Navigated to ${pageTitle}`;
    }
  }, [location.pathname]);

  return (
    <>
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>

      <header>
        <nav aria-label="Main">
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/products">Products</a></li>
          </ul>
        </nav>
      </header>

      {/* Live region for route announcements */}
      <div
        ref={announcerRef}
        role="status"
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"
      />

      <main ref={mainRef} id="main-content" tabIndex={-1}>
        <Outlet />
      </main>

      <footer>© 2025 My Store</footer>

      <ScrollRestoration />
    </>
  );
}
```

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

**Expected Output:** When the user navigates to a new route, focus moves to the `<main>` element, and a screen reader announces "Navigated to [Page Title]". Scroll position resets to the top on forward navigation and restores on back/forward. The skip link lets keyboard users bypass navigation.

**Why This Output Occurs:** `useLocation` detects route changes. The `useEffect` moves focus to `mainRef` and updates the live region. The `isFirstRender` ref skips the initial load. `<ScrollRestoration />` handles scroll. The `role="status"` region is present in the DOM before the announcement.

### Real-World Cases

- **E-commerce:** Announcing product page changes, resetting scroll after navigation.
- **SaaS dashboards:** Announcing dashboard section changes.
- **Documentation sites:** Announcing article changes.
- **Blogs:** Announcing post navigation.
- **Any SPA:** Focus and announcement on route change.

### References

- React Router – `ScrollRestoration`: https://reactrouter.com/api/components/ScrollRestoration
- React Router – `useLocation`: https://reactrouter.com/api/hooks/useLocation
- Next.js – Accessibility: https://nextjs.org/docs/architecture/accessibility
- WCAG 2.2 – Page Titled (2.4.2): https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html
- WCAG 2.2 – Status Messages (4.1.3): https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html
- Marcy Sutton – Accessibility of Client-Side Routing: https://marcysutton.com/
- Gatsby – Announcing Route Changes: https://www.gatsbyjs.com/blog/2019-07-11-user-testing-accessible-client-routing

---

## Comparison and Decision Guidance

| Concern | Approach | Key Technique | Common Pitfall |
|---|---|---|---|
| **Page structure** | Semantic landmarks + headings | `<main>`, `<nav>`, `<h1>`–`<h6>` | Using `<div>` with `role` unnecessarily |
| **Element names** | Accessible name computation | `aria-label`, `aria-labelledby`, `<label>` | `aria-label` overriding visible text (WCAG 2.5.3) |
| **Focus management** | Programmatic focus | `useRef`, `element.focus()`, focus trap | Removing focus outlines without replacement |
| **Keyboard navigation** | `tabIndex` + `onKeyDown` | Roving tabindex, `event.key` | Using positive `tabIndex`; using `keyCode` |
| **Dynamic content** | `aria-live` regions | `role="status"` (polite), `role="alert"` (assertive) | Inserting live region with content (no announcement) |
| **Forms** | Labels + fieldsets + error summaries | `<label>`, `<fieldset>`, `aria-describedby`, `role="alert"` | Missing labels; no focus on first error |
| **Complex widgets** | APG patterns | Roles, states, keyboard, focus per APG | Hand-rolling without following APG |
| **Routing** | Focus + announcement on route change | `useLocation` + `useEffect` + live region | No announcement; focus lost after navigation |

**Decision Guidance:**
- **Use semantic HTML first**; add ARIA only when no native element exists.
- **Give every interactive element a name** using `<label>`, visible text, `aria-label`, or `aria-labelledby`.
- **Manage focus deliberately:** move into modals, restore to triggers, trap within overlays, provide skip links.
- **Use native keyboard behaviour** when possible; add `onKeyDown` only for custom widgets.
- **Announce dynamic changes** with `aria-live="polite"` for status and `role="alert"` for errors.
- **Group related fields** with `<fieldset>` and `<legend>`; provide error summaries for long forms.
- **Follow the APG** for tabs, accordions, dialogs, comboboxes, and other complex widgets—or use a library.
- **Announce route changes** and move focus to the new page's main content.
- **Test with a keyboard and a screen reader**; automated tools catch only 30–50% of issues.

---

## References

- W3C WAI-ARIA Authoring Practices Guide: https://www.w3.org/WAI/ARIA/apg/
- W3C WAI-ARIA Authoring Practices – Patterns: https://www.w3.org/WAI/ARIA/apg/patterns/
- W3C WAI-ARIA Authoring Practices – Landmark Regions: https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/
- W3C WAI-ARIA Authoring Practices – Keyboard Interface: https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- W3C – Accessible Name and Description Computation 1.2: https://www.w3.org/TR/accname-1.2/
- W3C WAI – Forms Tutorial: https://www.w3.org/WAI/tutorials/forms/
- WCAG 2.2 – Quick Reference: https://www.w3.org/WAI/WCAG22/quickref/
- WCAG 2.2 – Info and Relationships (1.3.1): https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html
- WCAG 2.2 – Bypass Blocks (2.4.1): https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html
- WCAG 2.2 – Focus Visible (2.4.7): https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html
- WCAG 2.2 – Label in Name (2.5.3): https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html
- WCAG 2.2 – Error Identification (3.3.1): https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html
- WCAG 2.2 – Status Messages (4.1.3): https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html
- MDN Web Docs – ARIA: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA
- MDN Web Docs – ARIA Live Regions: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions
- MDN Web Docs – `aria-label`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label
- MDN Web Docs – `aria-labelledby`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-labelledby
- MDN Web Docs – `aria-describedby`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby
- MDN Web Docs – `tabindex`: https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex
- MDN Web Docs – `KeyboardEvent.key`: https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key
- React – Manipulating the DOM with Refs: https://react.dev/learn/manipulating-the-dom-with-refs
- React – `useId`: https://react.dev/reference/react/useId
- React Router – `ScrollRestoration`: https://reactrouter.com/api/components/ScrollRestoration
- React Router – `useLocation`: https://reactrouter.com/api/hooks/useLocation
- Next.js – Accessibility: https://nextjs.org/docs/architecture/accessibility
- Radix UI – Primitives: https://www.radix-ui.com/primitives
- React Aria – Components: https://react-spectrum.adobe.com/react-aria/
- Headless UI – Components: https://headlessui.com/
- focus-trap-react – npm: https://www.npmjs.com/package/focus-trap-react
- Deque – axe-core: https://github.com/dequelabs/axe-core
- Marcy Sutton – Accessibility of Client-Side Routing: https://marcysutton.com/
- Gatsby – Announcing Route Changes: https://www.gatsbyjs.com/blog/2019-07-11-user-testing-accessible-client-routing