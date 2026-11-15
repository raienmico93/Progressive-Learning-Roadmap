# Accessibility Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Accessibility (often abbreviated as **a11y**) is the practice of designing and building digital products so that they can be used by everyone, including people with visual, auditory, motor, cognitive, and neurological disabilities.

**Technical Definition:** Web accessibility is the engineering discipline of ensuring that web content and applications are **perceivable, operable, understandable, and robust** (POUR) for all users, including those who rely on assistive technologies (AT) such as screen readers, switch devices, screen magnifiers, and voice control. It is governed by standards such as the **Web Content Accessibility Guidelines (WCAG)** and implemented through **semantic HTML**, **WAI-ARIA** attributes, **keyboard interface design**, and **focus management**.

**Beginner-Friendly Explanation:** Accessibility means building websites and apps that everyone can use — including people who cannot see, cannot hear, cannot use a mouse, or have cognitive differences. It is about removing barriers so that all users can perceive, understand, navigate, and interact with your product.

### Key Characteristics

- **Standards-driven:** WCAG 2.2 is the current W3C Recommendation and ISO/IEC 40500:2025 standard.
- **POUR principles:** Content must be Perceivable, Operable, Understandable, and Robust.
- **Semantic-first:** Native HTML elements provide built-in accessibility semantics and keyboard behavior; ARIA is a supplement, not a replacement.
- **Legal mandate:** The European Accessibility Act (EAA) makes accessibility compliance mandatory for many digital products in the EU as of June 28, 2025.
- **User-centric testing:** Automated tools catch only 30–50% of issues; manual keyboard and screen reader testing is essential.

### Prerequisites

- A web application (React, Vue, Angular, or plain HTML).
- A modern browser and a screen reader (NVDA, JAWS, VoiceOver, or Narrator) for manual testing.
- Browser DevTools with the Accessibility pane.
- Automated linting tools such as `eslint-plugin-jsx-a11y` and automated audit tools like `axe-core` or Lighthouse.

### Related Programming Areas

- **Semantic HTML** — the foundation of accessible markup.
- **WAI-ARIA** — attributes for custom widgets where native semantics are insufficient.
- **Keyboard navigation and focus management** — ensuring every function is operable without a mouse.
- **Color contrast and visual design** — ensuring content is perceivable for low-vision and color-blind users.
- **Screen reader compatibility** — ensuring content is announced correctly by assistive technology.
- **React accessibility** — applying all of the above within React components.

### Core Concepts / Features

---

## 1. Accessibility (a11y)

### Definitions

**Core Definition:** Accessibility is the design and creation of websites and applications that can be used by everyone, regardless of disability.

**Technical Definition:** Accessibility is the degree to which a product, device, service, or environment can be used by the widest possible range of users, including those with disabilities. On the web, it is achieved by conforming to WCAG success criteria, using semantic HTML, providing text alternatives, ensuring keyboard operability, and supporting assistive technologies.

**Beginner-Friendly Explanation:** Accessibility means building things that work for everyone. If a person who is blind, deaf, or cannot use a mouse can still use your app successfully, your app is accessible.

### Purposes

- To ensure equal access to information and functionality for all users, including those with disabilities.
- To comply with legal requirements such as the European Accessibility Act (EAA) and Section 508.
- To improve overall user experience — accessible design benefits all users, including those on mobile devices or in noisy environments.
- To improve SEO and code quality through semantic, well-structured markup.

### Syntax Rules and Structure

Accessibility is not a single syntax but a set of practices applied across HTML, CSS, JavaScript, and ARIA:

| Layer | Practice |
|-------|----------|
| HTML | Use semantic elements (`<button>`, `<nav>`, `<main>`, `<h1>`–`<h6>`). |
| ARIA | Add `aria-label`, `aria-labelledby`, `role`, and state attributes only when native semantics are insufficient. |
| CSS | Ensure sufficient color contrast (4.5:1 for normal text, 3:1 for large text). |
| JavaScript | Manage focus, announce dynamic changes, and support keyboard interaction. |

**Constraints and Limitations:**

- Automated tools catch only a fraction of accessibility issues — manual testing with a keyboard and screen reader is required.
- ARIA misuse is worse than no ARIA — native HTML is almost always preferable.

### Real-World Cases

- **E-commerce:** Accessible checkout forms, product images with alt text, and keyboard-operable cart controls.
- **Government services:** Public sector websites are legally required to meet WCAG 2.1 AA in many jurisdictions.
- **Banking:** Accessible transaction tables, form validation errors announced to screen readers, and secure focus management in modals.

### References

- W3C — Web Content Accessibility Guidelines (WCAG) 2.2 — https://www.w3.org/TR/WCAG22/
- W3C — WAI-ARIA Authoring Practices Guide — https://www.w3.org/WAI/ARIA/apg/
- rtCamp — React Accessibility Best Practices — https://rtcamp.com/handbook/react-best-practices/accessibility/

---

## 2. A11y

### Definitions

**Core Definition:** A11y is a numeronym for "accessibility," where "11" stands for the eleven letters between "a" and "y" in the word.

**Technical Definition:** A11y is the standard abbreviation used in the accessibility community, in tooling (e.g., `eslint-plugin-jsx-a11y`, `axe-core`), and in documentation to refer concisely to digital accessibility. It encompasses the same scope as "accessibility" — WCAG conformance, semantic HTML, ARIA, keyboard support, and assistive technology compatibility.

**Beginner-Friendly Explanation:** A11y is just shorthand for "accessibility." When you see "a11y" in a tool name or a code review comment, it means accessibility.

### Purposes

- To provide a concise, widely recognized term for digital accessibility.
- To identify accessibility-focused tooling (e.g., `eslint-plugin-jsx-a11y`).
- To signal accessibility requirements in team workflows and documentation.

### Syntax Rules and Structure

```
a11y
 │
 ├── a (first letter)
 ├── 11 (eleven letters: c-c-e-s-s-i-b-i-l-i-t)
 └── y (last letter)
```

**Constraints and Limitations:**

- The term is informal — formal documents use "accessibility."
- The spelling must be exact (`a11y`, not `A11Y` or `ally`).

### Real-World Cases

- **Linting:** `eslint-plugin-jsx-a11y` enforces accessibility rules in React JSX.
- **Auditing:** `axe-core` is widely used for automated a11y audits.
- **Code review:** Reviewers flag "a11y issues" as blocking concerns.

### References

- rtCamp — React Accessibility (A11y) — https://rtcamp.com/handbook/react-best-practices/accessibility/

---

## 3. WCAG

### Definitions

**Core Definition:** The Web Content Accessibility Guidelines (WCAG) are the internationally recognized technical standards for making web content accessible, published by the W3C.

**Technical Definition:** WCAG 2.2 is the current W3C Recommendation (published 2023, also ISO/IEC 40500:2025) defining testable success criteria organized under four principles: **Perceivable, Operable, Understandable, and Robust (POUR)**. It defines three conformance levels — **A (minimum), AA (industry standard), and AAA (enhanced)** — and 13 guidelines with specific success criteria.

**Beginner-Friendly Explanation:** WCAG is the rulebook for web accessibility. It tells you exactly what you need to do — like "provide alt text for images" or "make sure text has enough contrast" — and gives you three levels of strictness: A (basic), AA (standard), and AAA (advanced).

### Purposes

- To provide a shared, testable standard for web accessibility.
- To define conformance levels (A, AA, AAA) so organizations can set targets.
- To serve as the legal benchmark for accessibility regulations worldwide.
- To guide developers and designers toward inclusive practices.

### Syntax Rules and Structure

**POUR principles and example success criteria:**

| Principle | Meaning | Example Success Criterion |
|-----------|---------|---------------------------|
| **Perceivable** | Content must be presentable in ways users can perceive. | 1.1.1 Non-text Content (Level A). |
| **Operable** | Users must be able to operate the interface. | 2.1.1 Keyboard (Level A); 2.4.3 Focus Order (Level A). |
| **Understandable** | Content and UI must be understandable. | 3.2.1 On Focus (Level A). |
| **Robust** | Content must work with current and future assistive technologies. | 4.1.2 Name, Role, Value (Level A). |

**Conformance levels:**

| Level | Meaning | Target |
|-------|---------|--------|
| A | Minimum requirements | Baseline accessibility. |
| AA | Mid-level, industry standard | Most organizations target AA. |
| AAA | Highest level | Specialized needs and government sites. |

**Constraints and Limitations:**

- WCAG 2.2 extends 2.1 without deprecating it; content conforming to 2.2 also conforms to 2.1 and 2.0.
- Automated tools can verify some criteria, but many require human judgment.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Meeting 1.4.3 Contrast (Minimum) — Level AA

```css
/* ❌ FAIL: contrast ratio ~2.5:1 */
.low-contrast {
  color: #999;
  background: #fff;
}

/* ✅ PASS: contrast ratio ≥ 4.5:1 */
.sufficient-contrast {
  color: #595959; /* darker gray */
  background: #fff;
}
```

**Expected Output:** The second style meets the 4.5:1 minimum contrast ratio for normal text required by WCAG 1.4.3 Level AA.

### Real-World Cases

- **Government websites:** Required by law to meet WCAG 2.1 AA in the EU, US, and many other jurisdictions.
- **Enterprise procurement:** Vendors are often required to provide a VPAT documenting WCAG conformance.
- **Design systems:** WCAG AA is the baseline target for component libraries.

### References

- W3C — WCAG 2.2 Recommendation — https://www.w3.org/TR/WCAG22/
- W3C — WCAG 2.2 Quick Reference — https://www.w3.org/WAI/WCAG22/quickref/
- rtCamp — WCAG Principles and Levels — https://rtcamp.com/handbook/react-best-practices/accessibility/

---

## 4. ARIA (WAI-ARIA)

### Definitions

**Core Definition:** WAI-ARIA (Accessible Rich Internet Applications) is a W3C specification that defines attributes to make dynamic web content and custom UI widgets accessible to assistive technologies.

**Technical Definition:** ARIA provides **roles** (what an element is), **states** (current condition, e.g., `aria-expanded`), and **properties** (relationships and characteristics, e.g., `aria-labelledby`). It is used when native HTML semantics are insufficient — for custom widgets like tabs, menus, and dialogs. The first rule of ARIA is: **do not use ARIA if a native HTML element already provides the required semantics and behavior**.

**Beginner-Friendly Explanation:** ARIA is a set of special HTML attributes that tell screen readers what custom widgets are and what state they are in. If you build a custom dropdown, ARIA tells the screen reader "this is a menu, it is currently closed." But if you can use a native `<button>`, you do not need ARIA at all.

### Purposes

- To expose the role, state, and properties of custom widgets to assistive technologies.
- To provide accessible names and descriptions for elements that lack visible text.
- To manage dynamic content announcements (live regions).
- To fill semantic gaps when native HTML cannot express the widget's behavior.

### Syntax Rules and Structure

**Core ARIA attribute categories:**

| Category | Examples | Purpose |
|----------|----------|---------|
| **Roles** | `role="dialog"`, `role="tablist"`, `role="alert"` | Define what the element is. |
| **States** | `aria-expanded`, `aria-checked`, `aria-selected` | Current condition — must be kept in sync with React state. |
| **Properties** | `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-live` | Relationships and labels. |

**Syntax rules:**

1. **No ARIA is better than bad ARIA** — use native HTML first.
2. All interactive elements must have an **accessible name** — via visible text, `aria-label`, or `aria-labelledby`.
3. ARIA states must be **kept in sync with the UI state** — if a dropdown is open, `aria-expanded` must be `true`.
4. Use **live regions** (`aria-live="polite"` or `"assertive"`) to announce dynamic changes.

**Constraints and Limitations:**

- ARIA does not add keyboard behavior — you must implement keyboard interaction yourself.
- Incorrect ARIA can make the experience worse for screen reader users.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Custom Toggle Button (React)

```jsx
function ToggleButton({ isPressed, onToggle, children }) {
  return (
    <button
      type="button"
      aria-pressed={isPressed}   // ARIA state
      onClick={onToggle}
    >
      {children}
    </button>
  );
}
```

**Expected Output:** A screen reader announces the button as "toggle button, pressed" or "not pressed," matching the visual state.

**Why this works:** `aria-pressed` communicates the toggle state. The native `<button>` provides keyboard operability (Enter/Space) automatically.

### Real-World Cases

- **Modal dialogs:** `role="dialog"`, `aria-modal="true"`, `aria-labelledby` pointing to the title.
- **Tabs:** `role="tablist"`, `role="tab"`, `aria-selected`, and `role="tabpanel"`.
- **Form errors:** `aria-invalid="true"` and `aria-describedby` pointing to the error message.

### References

- W3C — WAI-ARIA Authoring Practices Guide — https://www.w3.org/WAI/ARIA/apg/
- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/
- rtCamp — ARIA Best Practices — https://rtcamp.com/handbook/react-best-practices/accessibility/

---

## 5. Semantic HTML

### Definitions

**Core Definition:** Semantic HTML is the use of HTML elements according to their meaning and purpose, rather than for visual styling, so that browsers and assistive technologies can understand the content's structure and relationships.

**Technical Definition:** Semantic HTML leverages native elements that carry built-in **roles**, **states**, and **keyboard behavior** — `<button>` is focusable and activatable with Enter/Space, `<nav>` is a landmark, `<h1>`–`<h6>` define heading structure, and `<label>` associates with form controls. When semantic HTML is used, assistive technologies can programmatically determine the structure and purpose of content without additional ARIA.

**Beginner-Friendly Explanation:** Semantic HTML means using the right tag for the right job. A button should be a `<button>`, not a `<div>` with a click handler. A navigation menu should be a `<nav>`, not just a bunch of links in a `<div>`. This gives screen readers and keyboards the information they need automatically.

### Purposes

- To provide built-in accessibility semantics (roles, states, keyboard behavior) without additional ARIA.
- To give screen readers a clear, navigable document structure (headings, landmarks, lists, tables).
- To ensure keyboard operability for interactive elements by default.
- To improve code readability, maintainability, and SEO.

### Syntax Rules and Structure

**Semantic element mapping:**

| Purpose | Correct Element | Incorrect (Non-Semantic) |
|---------|----------------|--------------------------|
| Clickable action | `<button>` | `<div onClick={...}>` |
| Navigation | `<nav>` | `<div className="nav">` |
| Main content | `<main>` | `<div id="main">` |
| Heading | `<h1>`–`<h6>` | `<p style="font-size: 24px">` |
| List | `<ul>` / `<ol>` / `<li>` | `<div>` with bullets |
| Form label | `<label htmlFor="...">` | `<span>` next to input |

**Syntax rules:**

1. Use headings in logical order (`<h1>` → `<h2>` → `<h3>`), never skip levels.
2. Use real lists for lists, real tables for tabular data, and real `<label>` elements for form fields.
3. Do not use `<div>` or `<span>` when a semantic element exists.
4. Use `<button>` for actions and `<a>` for navigation — they have different keyboard and screen reader semantics.

**Constraints and Limitations:**

- Styling a `<div>` to look like a button does not make it a button — you must add `role`, `tabIndex`, and keyboard handlers manually, which is error-prone.
- Native elements carry browser-default behavior that may need to be overridden with CSS, but their semantics should be preserved.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Semantic Form vs. Non-Semantic Form

```jsx
// ❌ Non-semantic: div with click handler, no keyboard support
<div onClick={handleSubmit} className="button-style">
  Submit
</div>

// ✅ Semantic: native button, keyboard accessible
<button type="submit" onClick={handleSubmit}>
  Submit
</button>
```

**Expected Output:** The `<button>` is focusable with Tab, activates with Enter/Space, and is announced as a "button" by screen readers. The `<div>` is none of these things without extensive manual work.

#### Example 2: Semantic Page Structure

```jsx
<header>
  <nav aria-label="Main">...</nav>
</header>
<main>
  <h1>Page Title</h1>
  <section aria-labelledby="section-heading">
    <h2 id="section-heading">Section</h2>
    <p>Content</p>
  </section>
</main>
<footer>...</footer>
```

**Expected Output:** Screen readers can navigate directly to the navigation, main content, and footer via landmark shortcuts, and headings provide a table of contents for the page.

### Real-World Cases

- **Documentation sites:** Semantic headings and landmarks enable quick navigation for screen reader users.
- **E-commerce:** Semantic product listings, forms, and checkout flows.
- **Design systems:** Semantic HTML is the foundation of accessible component APIs.

### References

- MDN — HTML: A Good Basis for Accessibility — https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML
- W3C — ARIA Landmarks — https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/
- a11y.tech.gov.sg — Presentation of Info and Relationships — https://a11y.tech.gov.sg/checklist/wp6/

---

## 6. Screen Readers

### Definitions

**Core Definition:** A screen reader is assistive technology software that converts digital text and UI elements into synthesized speech or braille output for users who are blind or have low vision.

**Technical Definition:** Screen readers interpret the **accessibility tree** — a browser-generated representation of the DOM that exposes roles, names, states, and properties — and announce it to the user. They rely on semantic HTML and ARIA to understand what each element is and how it behaves. For example, a `<button>` is announced as a "button," a heading as a "heading level 2," and a form input is announced with its associated label.

**Beginner-Friendly Explanation:** A screen reader is software that reads the screen aloud for people who cannot see it. It uses the HTML and ARIA you write to know what to say — so if you write a `<button>`, it says "button"; if you write a `<div>` with a click handler, it says nothing useful.

### Purposes

- To provide non-visual access to web content for blind and low-vision users.
- To convey document structure (headings, lists, landmarks) through navigation commands.
- To announce dynamic changes (live regions) and form states (errors, required fields).
- To enable efficient navigation via landmarks, headings, links, and form controls.

### Syntax Rules and Structure

**What screen readers announce:**

| Element | Announced as |
|---------|-------------|
| `<button>` | "Button, [name]" |
| `<a href="...">` | "Link, [text]" |
| `<h2>` | "Heading level 2, [text]" |
| `<ul>` | "List, [n] items" |
| `<input>` with `<label>` | "[label], edit text" |
| `<img alt="...">` | "[alt text], image" |

**Syntax rules:**

1. Use semantic HTML first — screen readers rely on native roles.
2. Every interactive element must have an accessible name.
3. Use `aria-live` regions for dynamic content that should be announced.
4. Test with at least one screen reader (NVDA + Firefox on Windows, VoiceOver + Safari on macOS/iOS).

**Constraints and Limitations:**

- Screen readers behave differently across browsers and operating systems — test combinations.
- Visually hidden text (`sr-only` class) is still read by screen readers and is useful for additional context.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Icon Button

```jsx
// ❌ Screen reader announces "button" with no name
<button><svg>...</svg></button>

// ✅ Screen reader announces "Search, button"
<button aria-label="Search">
  <svg aria-hidden="true">...</svg>
</button>
```

**Expected Output:** The second button is announced as "Search, button," while the first is announced only as "button" — the user does not know what it does.

### Real-World Cases

- **News sites:** Screen reader users navigate by headings and landmarks to find articles.
- **Banking apps:** Form labels and error announcements are critical for completing transactions.
- **Documentation:** Semantic structure enables efficient navigation of long technical pages.

### References

- MDN — Screen Readers — https://developer.mozilla.org/en-US/docs/Glossary/Screen_reader
- WebAIM — Screen Reader User Survey — https://webaim.org/projects/screenreadersurvey/

---

## 7. Keyboard Navigation

### Definitions

**Core Definition:** Keyboard navigation is the ability to operate every interactive element and function of a web application using only a keyboard, without a mouse.

**Technical Definition:** Keyboard navigation requires that all interactive elements are **focusable** (reachable via Tab), **operable** (activatable with Enter, Space, Arrow keys, or other standard keys), and that focus moves in a **logical, predictable order** (typically matching the visual reading order). Native HTML elements provide this automatically; custom ARIA widgets require the developer to implement keyboard behavior.

**Beginner-Friendly Explanation:** Keyboard navigation means you can use the entire app with just a keyboard — Tab to move between elements, Enter or Space to activate them, and arrow keys to navigate within menus or lists.

### Purposes

- To enable use by people with motor disabilities who cannot use a mouse.
- To support power users who prefer keyboard shortcuts.
- To satisfy WCAG 2.1.1 (Keyboard) — a Level A requirement.
- To ensure that custom widgets are operable by assistive technology users.

### Syntax Rules and Structure

**Standard keyboard conventions:**

| Key | Expected behavior |
|-----|-------------------|
| `Tab` | Move focus to the next focusable element. |
| `Shift + Tab` | Move focus to the previous focusable element. |
| `Enter` | Activate links and buttons. |
| `Space` | Activate buttons and toggle checkboxes. |
| `Arrow keys` | Navigate within composite widgets (menus, tabs, radio groups). |
| `Escape` | Close dialogs, menus, and popups. |

**Syntax rules:**

1. All functionality must be operable with a keyboard — no mouse-only interactions.
2. Focus order must preserve meaning and operability.
3. Custom widgets must implement expected keyboard patterns (follow the ARIA Authoring Practices).
4. Avoid keyboard traps — users must be able to move focus away from any component.

**Constraints and Limitations:**

- `tabIndex` values greater than 0 disrupt natural focus order and should be avoided.
- Custom widgets require manual keyboard event handling — this is where most accessibility bugs occur.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Keyboard-Accessible Custom Dropdown

```jsx
function Dropdown({ items }) {
  const [isOpen, setIsOpen] = useState(false);

  function handleKeyDown(e) {
    if (e.key === 'Escape') setIsOpen(false);
    if (e.key === 'ArrowDown' && !isOpen) setIsOpen(true);
  }

  return (
    <div onKeyDown={handleKeyDown}>
      <button
        aria-haspopup="listbox"
        aria-expanded={isOpen}
        onClick={() => setIsOpen(!isOpen)}
      >
        Options
      </button>
      {isOpen && (
        <ul role="listbox">
          {items.map((item) => (
            <li key={item} role="option" tabIndex={0}>
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Expected Output:** The dropdown opens with Enter/Space, closes with Escape, and exposes `aria-expanded` state to screen readers.

### Real-World Cases

- **Modal dialogs:** Focus must move into the dialog on open, be trapped inside, and return to the trigger on close.
- **Menus and tabs:** Arrow-key navigation within the widget, Tab to exit.
- **Forms:** Tab through fields, Enter to submit, Escape to cancel.

### References

- W3C — Developing a Keyboard Interface — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- W3C — Understanding SC 2.1.1 Keyboard — https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html

---

## 8. Focus Management

### Definitions

**Core Definition:** Focus management is the practice of programmatically controlling which element has keyboard focus at any given time, ensuring that focus moves logically and remains visible.

**Technical Definition:** Focus management encompasses: setting initial focus when a page or dialog loads, trapping focus inside modal dialogs, returning focus to the triggering element when a dialog closes, ensuring a **visible focus indicator** is always present, and handling focus when elements are removed from the DOM (e.g., after deleting a list item). WCAG 2.4.3 (Focus Order) and 2.4.7 (Focus Visible) are the governing success criteria.

**Beginner-Friendly Explanation:** Focus management means controlling where the keyboard "cursor" is. When you open a modal, focus should move into it; when you close it, focus should return to the button that opened it. Users should never lose track of where they are on the page.

### Purposes

- To ensure keyboard users can always see and track their current position.
- To prevent focus from being lost when DOM elements are removed or hidden.
- To trap focus inside modal dialogs so users cannot accidentally interact with background content.
- To return focus to the logical place after a dialog or menu closes.

### Syntax Rules and Structure

**Focus management patterns:**

| Scenario | Pattern |
|----------|---------|
| Modal opens | Move focus to the first focusable element (or the dialog itself). |
| Modal closes | Return focus to the element that triggered it. |
| List item deleted | Move focus to the next item (or previous if last). |
| Page navigation | Move focus to the main heading or a skip-link target. |
| Visible focus | Never remove `:focus` styles without providing an accessible alternative. |

**Constraints and Limitations:**

- Do not set `outline: none` without a replacement focus indicator — this fails WCAG 2.4.7.
- Focus must not move unexpectedly (WCAG 3.2.1 On Focus).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Focus Management in a React Modal

```jsx
function Modal({ isOpen, onClose, triggerRef, children }) {
  const modalRef = useRef(null);
  const previousFocus = useRef(null);

  useEffect(() => {
    if (isOpen) {
      previousFocus.current = document.activeElement;
      modalRef.current?.focus();
    } else {
      previousFocus.current?.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
      onKeyDown={(e) => e.key === 'Escape' && onClose()}
    >
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

**Expected Output:** Opening the modal moves focus into it; closing it returns focus to the element that was focused before. Escape closes the modal.

### Real-World Cases

- **Modals and dialogs:** Focus trap, initial focus, and return focus are the three core requirements.
- **Single-page applications:** Route changes should move focus to the new page's heading.
- **Dynamic lists:** After deletion, focus should move to the next logical item.

### References

- W3C — Understanding SC 2.4.3 Focus Order — https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html
- W3C — Developing a Keyboard Interface: Focus Management — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- react-focus-on — Focus trap library — https://www.npmjs.com/package/react-focus-on

---

## 9. Color Contrast

### Definitions

**Core Definition:** Color contrast is the difference in luminance between foreground text (or UI components) and their background, measured as a ratio, ensuring that content is legible for users with low vision or color blindness.

**Technical Definition:** WCAG defines minimum contrast ratios under Success Criterion 1.4.3 (Contrast — Minimum, Level AA): **4.5:1 for normal text** and **3:1 for large text** (≥18pt, or ≥14pt bold). Non-text contrast (UI components, graphical objects) is covered by SC 1.4.11 (Level AA) at **3:1**.

**Beginner-Friendly Explanation:** Color contrast is about making text easy to read against its background. Dark text on a light background has high contrast; light gray text on white has low contrast and is hard for many people to read. WCAG sets minimum ratios to ensure readability.

### Purposes

- To ensure text is readable by people with low vision, color blindness, or in poor lighting conditions.
- To make UI components (buttons, form borders, icons) distinguishable.
- To comply with WCAG 1.4.3 (AA) and 1.4.11 (AA).
- To ensure that color is not the only means of conveying information (WCAG 1.4.1).

### Syntax Rules and Structure

**Contrast requirements:**

| Content | Minimum ratio | WCAG SC |
|---------|--------------|---------|
| Normal text | 4.5:1 | 1.4.3 AA |
| Large text (≥18pt, or ≥14pt bold) | 3:1 | 1.4.3 AA |
| UI components and graphics | 3:1 | 1.4.11 AA |
| Enhanced contrast (AAA) | 7:1 / 4.5:1 | 1.4.6 AAA |

**Syntax rules:**

1. Check contrast ratios with a tool (WebAIM Contrast Checker, axe DevTools, or browser DevTools).
2. Never use color alone to convey information (e.g., red for errors) — add text or icons.
3. Focus indicators must also meet contrast requirements (3:1 against adjacent colors).

**Constraints and Limitations:**

- Contrast ratios are based on relative luminance — tools calculate them precisely.
- Opacity and background images complicate contrast calculations; test the rendered result.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Passing and Failing Contrast

```css
/* ❌ FAIL: light gray on white (~1.5:1) */
.fail {
  color: #cccccc;
  background: #ffffff;
}

/* ✅ PASS AA: dark gray on white (≥ 4.5:1) */
.pass {
  color: #595959;
  background: #ffffff;
}
```

**Expected Output:** The `.pass` style meets the 4.5:1 ratio for normal text; `.fail` would be flagged by automated audit tools.

### Real-World Cases

- **Form error messages:** Red text must have sufficient contrast against the background, and must not rely on color alone.
- **Buttons:** Button text and borders must meet 4.5:1 (text) and 3:1 (border/background).
- **Data tables:** Alternating row colors must still provide sufficient contrast for text.

### References

- W3C — Understanding SC 1.4.3 Contrast (Minimum) — https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- WebAIM — Contrast Checker — https://webaim.org/resources/contrastchecker/

---

## 10. Text Alternatives

### Definitions

**Core Definition:** Text alternatives (alt text) are textual descriptions of non-text content (images, charts, icons, audio) that serve the equivalent purpose for users who cannot perceive the original content.

**Technical Definition:** WCAG Success Criterion 1.1.1 (Non-text Content, Level A) requires that **all non-text content presented to the user has a text alternative that serves the equivalent purpose**. Text alternatives are rendered through any sensory modality (visual, auditory, tactile) and are announced by screen readers. Decorative images should have `alt=""` (empty alt) so they are ignored by assistive technology.

**Beginner-Friendly Explanation:** Alt text describes an image for someone who cannot see it. If an image conveys information, the alt text should say what that information is. If an image is purely decorative, use `alt=""` so screen readers skip it.

### Purposes

- To provide equivalent information for images, charts, and other non-text content.
- To ensure screen reader users can understand visual content.
- To comply with WCAG 1.1.1 (Level A).
- To improve SEO and provide fallback content when images fail to load.

### Syntax Rules and Structure

**Alt text decision tree:**

| Content type | Alt text rule |
|--------------|---------------|
| Informative image | Describe the information it conveys. |
| Decorative image | `alt=""` (empty) — screen readers skip it. |
| Functional image (e.g., icon button) | Describe the function, not the image. |
| Complex image (chart, graph) | Short alt + detailed description elsewhere. |
| Image of text | Use real text instead, or provide the text in alt. |

**Syntax rules:**

1. Alt text should be concise — under ~125 characters when possible.
2. Do not start with "Image of..." — screen readers already announce it as an image.
3. For icon buttons, use `aria-label` on the button rather than alt text on the SVG.
4. Decorative images must have `alt=""` (empty string), not `alt="decorative"` or a missing alt attribute.

**Constraints and Limitations:**

- Alt text must convey the **purpose** of the image in context, not just describe its appearance.
- Complex images may require a longer description via `aria-describedby` or a linked page.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Informative vs. Decorative Images

```jsx
// ❌ Missing alt — screen reader may read the file name
<img src="chart.png" />

// ✅ Informative: conveys the data
<img src="chart.png" alt="Sales increased 25% from Q1 to Q2" />

// ✅ Decorative: ignored by screen readers
<img src="divider.png" alt="" role="presentation" />
```

**Expected Output:** The informative image is announced with its data summary; the decorative image is skipped entirely.

#### Example 2: Icon Button with aria-label

```jsx
// ❌ Screen reader announces "button" with no name
<button><svg>...</svg></button>

// ✅ Screen reader announces "Search, button"
<button aria-label="Search">
  <svg aria-hidden="true">...</svg>
</button>
```

### Real-World Cases

- **E-commerce:** Product images need alt text describing the product; decorative lifestyle images use empty alt.
- **Charts and dashboards:** Data visualizations need short alt text plus a detailed table or description.
- **Icon buttons:** Search, menu, and close buttons need `aria-label` since they have no visible text.

### References

- W3C — Understanding SC 1.1.1 Non-text Content — https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html
- WebAIM — Alternative Text — https://webaim.org/techniques/alttext/

---

## 11. Accessible React

### Definitions

**Core Definition:** Accessible React is the practice of building React applications that conform to accessibility standards by leveraging semantic HTML, ARIA attributes where necessary, proper focus management, and keyboard support.

**Technical Definition:** React supports accessibility through standard HTML techniques and ARIA attributes. Key patterns include: using semantic elements over `<div>` with roles, keeping ARIA states in sync with React state (`aria-expanded={isOpen}`), managing focus with `useRef` and `useEffect`, using `htmlFor` instead of `for`, and leveraging `eslint-plugin-jsx-a11y` to catch violations at lint time. React 19 and modern tooling also support `inert` for focus trapping and `useId` for unique, SSR-safe IDs.

**Beginner-Friendly Explanation:** Accessible React means building React components the right way so everyone can use them. Use real HTML elements instead of divs, add ARIA only when needed, keep ARIA state in sync with your React state, and manage focus when you open dialogs or change pages.

### Purposes

- To ensure React applications are usable by people with disabilities.
- To catch accessibility issues early with linting and automated testing.
- To manage focus and keyboard navigation in single-page applications.
- To keep ARIA state synchronized with React's declarative state.

### Syntax Rules and Structure

**React-specific accessibility rules:**

| Rule | Correct pattern |
|------|----------------|
| Semantic elements first | `<button>` not `<div role="button">`. |
| Sync ARIA state | `aria-expanded={isOpen}` — not hardcoded. |
| Labels | `htmlFor="id"` on `<label>`, `id` on input. |
| Focus management | `useRef` + `useEffect` to move focus into modals. |
| Unique IDs | `useId()` for SSR-safe label associations. |
| Linting | `eslint-plugin-jsx-a11y` catches common violations. |
| Announce dynamic changes | `aria-live` regions for async updates. |

**Constraints and Limitations:**

- React's `onClick` on a `<div>` does not provide keyboard support — use `<button>` or add `role`, `tabIndex`, and `onKeyDown` manually.
- `dangerouslySetInnerHTML` can introduce inaccessible or unsafe content.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible React Form Field

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

### Real-World Cases

- **Design systems:** Every component is built with semantic HTML and ARIA, tested with `jest-axe` and keyboard navigation.
- **Single-page applications:** Route changes move focus to the new page's heading.
- **Modals and dialogs:** Focus trapping with `react-focus-on` or the native `<dialog>` element.

### References

- React — Accessibility — https://react.dev/learn/accessibility
- rtCamp — React Accessibility (A11y) Best Practices — https://rtcamp.com/handbook/react-best-practices/accessibility/
- eslint-plugin-jsx-a11y — https://github.com/jsx-eslint/eslint-plugin-jsx-a11y

---

## 12. Accessibility Best Practices

### Definitions

**Core Definition:** Accessibility best practices are the guidelines and habits that ensure digital products are usable by everyone — from semantic HTML and ARIA discipline to keyboard testing and continuous auditing.

**Technical Definition:** Best practices encompass: **semantic-first development** (native HTML before ARIA), **keyboard-first design** (every function operable without a mouse), **visible focus** (never remove focus indicators without replacement), **contrast compliance** (4.5:1 normal text, 3:1 large text), **text alternatives** for all non-text content, **ARIA discipline** (no ARIA is better than bad ARIA), and **continuous testing** (automated linting + manual screen reader + keyboard testing).

**Beginner-Friendly Explanation:** Best practices are the habits that make accessibility work. Use real HTML elements, make everything keyboard-accessible, keep focus visible, provide alt text, only use ARIA when you must, and test with a screen reader and keyboard regularly.

### Purposes

- To build accessibility into the development process rather than bolting it on afterward.
- To catch issues early through linting, automated testing, and manual checks.
- To ensure compliance with WCAG 2.1 AA and legal requirements.
- To create a consistent, inclusive user experience across the application.

### Syntax Rules and Structure

**Accessibility best practices checklist:**

| Area | Practice |
|------|----------|
| **Semantic HTML** | Use `<button>`, `<nav>`, `<main>`, `<h1>`–`<h6>` — not divs with roles. |
| **ARIA** | Only when native HTML cannot express the semantics; keep states in sync. |
| **Keyboard** | Every function operable with Tab, Enter, Space, Arrow keys, and Escape. |
| **Focus** | Visible focus indicator always; logical focus order; trap focus in modals; return focus on close. |
| **Contrast** | 4.5:1 normal text; 3:1 large text and UI components. |
| **Alt text** | Informative images described; decorative images `alt=""`. |
| **Forms** | Every input has a `<label>`; errors are announced with `role="alert"` and `aria-describedby`. |
| **Testing** | `eslint-plugin-jsx-a11y` + `jest-axe` + manual keyboard + screen reader testing. |

**Constraints and Limitations:**

- Automated tools catch only 30–50% of issues — manual testing is essential.
- Accessibility is not a one-time audit — it requires continuous attention as the codebase evolves.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Modal Dialog

```jsx
function AccessibleModal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);

  useEffect(() => {
    if (isOpen) modalRef.current?.focus();
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      tabIndex={-1}
      onKeyDown={(e) => e.key === 'Escape' && onClose()}
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

**Expected Output:** The modal is announced as a dialog with its title, focus moves into it on open, Escape closes it, and the background is hidden from screen readers via `aria-modal="true"`.

### Real-World Cases

- **Design system audits:** Every component is tested with `jest-axe` and manual keyboard navigation before release.
- **CI pipelines:** `eslint-plugin-jsx-a11y` fails the build on accessibility violations.
- **User testing:** Sessions with screen reader users uncover issues that automated tools miss.

### References

- W3C — WCAG 2.2 Quick Reference — https://www.w3.org/WAI/WCAG22/quickref/
- W3C — ARIA Authoring Practices Guide — https://www.w3.org/WAI/ARIA/apg/
- rtCamp — React Accessibility Best Practices — https://rtcamp.com/handbook/react-best-practices/accessibility/
- a11y.tech.gov.sg — Accessibility Checklists — https://a11y.tech.gov.sg/checklist/

---

## References

- W3C — Web Content Accessibility Guidelines (WCAG) 2.2 — https://www.w3.org/TR/WCAG22/
- W3C — WCAG 2.2 Quick Reference — https://www.w3.org/WAI/WCAG22/quickref/
- W3C — WAI-ARIA Authoring Practices Guide — https://www.w3.org/WAI/ARIA/apg/
- W3C — Providing Accessible Names and Descriptions — https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/
- W3C — Developing a Keyboard Interface — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- W3C — Understanding SC 1.1.1 Non-text Content — https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html
- W3C — Understanding SC 1.4.3 Contrast (Minimum) — https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- W3C — Understanding SC 2.4.3 Focus Order — https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html
- W3C — Understanding SC 2.4.7 Focus Visible — https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html
- W3C — Understanding SC 3.2.1 On Focus — https://www.w3.org/WAI/WCAG22/Understanding/on-focus.html
- MDN — HTML: A Good Basis for Accessibility — https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML
- MDN — Screen Readers — https://developer.mozilla.org/en-US/docs/Glossary/Screen_reader
- React — Accessibility — https://react.dev/learn/accessibility
- rtCamp — React Accessibility (A11y) Best Practices — https://rtcamp.com/handbook/react-best-practices/accessibility/
- a11y.tech.gov.sg — Presentation of Info and Relationships — https://a11y.tech.gov.sg/checklist/wp6/
- WebAIM — Contrast Checker — https://webaim.org/resources/contrastchecker/
- WebAIM — Alternative Text — https://webaim.org/techniques/alttext/
- eslint-plugin-jsx-a11y — https://github.com/jsx-eslint/eslint-plugin-jsx-a11y
- react-focus-on — Focus trap library — https://www.npmjs.com/package/react-focus-on