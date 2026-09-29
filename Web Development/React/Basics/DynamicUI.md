# React Dynamic UI Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React dynamic UI patterns are the recurring architectural solutions for building interactive, stateful interface components—tabs, accordions, modals, dropdowns, pagination, wizards, expand/collapse regions, and dynamic navigation—that respond to user input while maintaining accessibility and predictable state.

**Technical Definition:** React dynamic UI patterns are the component-level implementations of established WAI-ARIA Authoring Practices, combining React state management (controlled or uncontrolled), composition patterns (compound components, slots, render props), and focus management (focus traps, roving tabindex, focus restoration) to produce interactive widgets that are operable by mouse, keyboard, and assistive technology. Each pattern has a canonical ARIA role structure (e.g., `tablist`/`tab`/`tabpanel` for tabs, `dialog` for modals, `menu`/`menuitem` for dropdowns), a keyboard interaction model (Arrow keys, Enter, Space, Escape, Home, End), and a state model (open/closed, active/inactive, selected/unselected). Libraries such as Radix UI, React Aria, and Headless UI encapsulate these patterns so developers can focus on presentation.

**Beginner-Friendly Explanation:** When users interact with your app, they expect familiar behaviours: tabs that switch content, accordions that expand and collapse, modals that trap focus, dropdowns that open on click, and navigation that adapts to screen size. These patterns are everywhere—but implementing them correctly (especially for keyboard and screen-reader users) is surprisingly hard. React dynamic UI patterns are the proven recipes for building each one. This cheat sheet shows you how each works, when to use it, and how to build it accessibly.

### Key Characteristics

- **ARIA Role Structure:** Each pattern maps to specific ARIA roles and attributes (`role="tablist"`, `aria-expanded`, `aria-modal`, `aria-haspopup`) that define its semantics for assistive technology.
- **Keyboard Interaction Model:** Every pattern specifies how keyboard users navigate and operate it (Arrow keys for tabs, Escape for modals, Enter/Space for buttons).
- **Focus Management:** Focus must be trapped (modals, dropdowns), restored (after close), or moved (tabs, wizards) predictably.
- **State Ownership:** State may be local (uncontrolled) or lifted/controlled by the parent, depending on whether other components need to coordinate.
- **Composition Over Configuration:** Patterns are best implemented as compound components (`Tabs.List`, `Tabs.Tab`, `Tabs.Panel`) rather than monolithic components with dozens of props.
- **Progressive Enhancement:** Native HTML elements (`<details>`, `<dialog>`) provide baseline behaviour; React enhances them.
- **Responsive Behaviour:** Navigation and layouts adapt to viewport or container size using media queries or container queries.

### Prerequisites

- Solid understanding of React function components, JSX, and the `useState`/`useRef` Hooks.
- Working knowledge of props, `children`, and composition patterns.
- Familiarity with HTML semantics and basic ARIA attributes.
- Basic understanding of the DOM, events, and focus.
- Awareness of WAI-ARIA Authoring Practices and WCAG 2.2.

### Related Programming Areas

- **Accessibility (a11y):** ARIA roles, focus management, keyboard navigation.
- **Component Composition:** Compound components, slots, and controlled/uncontrolled patterns.
- **State Management:** Local vs. lifted vs. controlled state.
- **Routing:** Dynamic navigation, active states, and link semantics.
- **Animation:** Expand/collapse transitions, modal enter/exit, and reduced-motion preferences.

### Core Concepts / Features

1. Tabs
2. Accordions
3. Modals
4. Dropdowns
5. Pagination
6. Wizards
7. Expand/Collapse Interfaces
8. Dynamic Navigation

---

## Core Concept 1: Tabs

### Definitions

**Core Definition:** Tabs are a navigation pattern that organises content into multiple panels, where only one panel is visible at a time and each panel is associated with a selectable tab.

**Technical Definition:** The WAI-ARIA Tabs pattern uses `role="tablist"` on the container, `role="tab"` on each tab (with `aria-selected`, `aria-controls`, and `id`), and `role="tabpanel"` on each panel (with `aria-labelledby` and `tabindex="0"`). Keyboard interaction uses a **roving tabindex**: only the active tab is in the tab order (`tabindex="0"`), while inactive tabs have `tabindex="-1"`. Arrow keys move focus between tabs (Left/Right in horizontal tabs, Up/Down in vertical tabs), Home/End jump to the first/last tab, and Tab moves focus from the tablist into the active panel. Activation can be **automatic** (focus follows selection) or **manual** (Enter/Space activates the focused tab).

**Beginner-Friendly Explanation:** Tabs are like file folders in a filing cabinet. You see one folder's contents at a time, and you click a tab to switch folders. But for keyboard users, tabs work differently from buttons: you use the arrow keys to move between tabs, not the Tab key. The Tab key moves you into the content of the selected tab. This distinction is crucial for accessibility and is defined by the WAI-ARIA Tabs pattern.

### Purposes

- To organise related content into discrete, switchable views without navigating away from the page.
- To reduce cognitive load by showing only one section at a time.
- To provide keyboard users with arrow-key navigation between tabs.
- To announce the active tab and its associated panel to screen readers.
- To support both horizontal and vertical orientations.

### Syntax Rules and Structure

**General Syntax (Compound Component):**
```jsx
<Tabs defaultValue="tab1">
  <Tabs.List aria-label="Account settings">
    <Tabs.Tab value="tab1">Profile</Tabs.Tab>
    <Tabs.Tab value="tab2">Notifications</Tabs.Tab>
    <Tabs.Tab value="tab3">Security</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="tab1">Profile content</Tabs.Panel>
  <Tabs.Panel value="tab2">Notifications content</Tabs.Panel>
  <Tabs.Panel value="tab3">Security content</Tabs.Panel>
</Tabs>
```

**Component Breakdown:**
- `<Tabs>`: Owns the active tab state; provides context to children.
- `<Tabs.List>`: The `role="tablist"` container; `aria-label` describes its purpose.
- `<Tabs.Tab>`: A `role="tab"` button with `aria-selected` and `aria-controls`.
- `<Tabs.Panel>`: A `role="tabpanel"` region with `aria-labelledby` and `tabindex="0"`.

**Simplified Implementation:**
```jsx
import { createContext, useContext, useId, useState, useRef } from 'react';

const TabsContext = createContext(null);

function Tabs({ defaultValue, children }) {
  const [activeTab, setActiveTab] = useState(defaultValue);
  const baseId = useId();
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab, baseId }}>
      <div>{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children, 'aria-label': ariaLabel }) {
  const ref = useRef(null);

  function handleKeyDown(e) {
    const tabs = Array.from(ref.current.querySelectorAll('[role="tab"]:not([disabled])'));
    const currentIndex = tabs.indexOf(document.activeElement);
    if (currentIndex === -1) return;

    let nextIndex = currentIndex;
    switch (e.key) {
      case 'ArrowRight': nextIndex = (currentIndex + 1) % tabs.length; break;
      case 'ArrowLeft': nextIndex = (currentIndex - 1 + tabs.length) % tabs.length; break;
      case 'Home': nextIndex = 0; break;
      case 'End': nextIndex = tabs.length - 1; break;
      default: return;
    }
    e.preventDefault();
    tabs[nextIndex].focus();
    tabs[nextIndex].click(); // Automatic activation
  }

  return (
    <div ref={ref} role="tablist" aria-label={ariaLabel} onKeyDown={handleKeyDown}>
      {children}
    </div>
  );
}

function Tab({ value, children }) {
  const { activeTab, setActiveTab, baseId } = useContext(TabsContext);
  const selected = activeTab === value;
  return (
    <button
      role="tab"
      id={`${baseId}-tab-${value}`}
      aria-selected={selected}
      aria-controls={`${baseId}-panel-${value}`}
      tabIndex={selected ? 0 : -1}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function Panel({ value, children }) {
  const { activeTab, baseId } = useContext(TabsContext);
  if (activeTab !== value) return null;
  return (
    <div
      role="tabpanel"
      id={`${baseId}-panel-${value}`}
      aria-labelledby={`${baseId}-tab-${value}`}
      tabIndex={0}
    >
      {children}
    </div>
  );
}

Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = Panel;
```

**Syntax Rules:**
- Use `role="tablist"` on the container; `aria-label` or `aria-labelledby` describes its purpose.
- Use `role="tab"` on each tab with `aria-selected`, `aria-controls`, and unique `id`.
- Use `role="tabpanel"` on each panel with `aria-labelledby` pointing to its tab and `tabindex="0"`.
- Only the active tab has `tabindex="0"`; others have `tabindex="-1"` (roving tabindex).
- Arrow Left/Right (horizontal) or Up/Down (vertical) move focus; Home/End jump to first/last.
- Tab key moves focus from the tablist into the active panel.
- Use `aria-orientation="vertical"` on the tablist for vertical tabs.

**Constraints and Limitations:**
- Automatic activation (focus follows selection) is faster but can be disorienting for screen-reader users; manual activation is safer when panels are expensive to render.
- Tabs should not be used for navigation between routes; use links for that.
- Deeply nested tabs are confusing; limit to one level.
- Always render at least one tab; an empty tablist is invalid.

### Annotated Code Example: Full Accessible Tabs

```jsx
import { useId, useState, useRef } from 'react';

export function SettingsTabs() {
  const [activeTab, setActiveTab] = useState('profile');
  const baseId = useId();
  const tabListRef = useRef(null);

  const tabs = [
    { value: 'profile', label: 'Profile', content: 'Manage your profile.' },
    { value: 'notifications', label: 'Notifications', content: 'Choose what to be notified about.' },
    { value: 'security', label: 'Security', content: 'Update your password.' },
  ];

  function handleKeyDown(e) {
    const buttons = Array.from(
      tabListRef.current.querySelectorAll('[role="tab"]')
    );
    const currentIndex = buttons.indexOf(document.activeElement);
    if (currentIndex === -1) return;

    let nextIndex = currentIndex;
    if (e.key === 'ArrowRight') nextIndex = (currentIndex + 1) % buttons.length;
    else if (e.key === 'ArrowLeft') nextIndex = (currentIndex - 1 + buttons.length) % buttons.length;
    else if (e.key === 'Home') nextIndex = 0;
    else if (e.key === 'End') nextIndex = buttons.length - 1;
    else return;

    e.preventDefault();
    buttons[nextIndex].focus();
    setActiveTab(tabs[nextIndex].value);
  }

  return (
    <div>
      <div
        ref={tabListRef}
        role="tablist"
        aria-label="Settings sections"
        onKeyDown={handleKeyDown}
        style={{ display: 'flex', gap: 8, borderBottom: '1px solid #ccc' }}
      >
        {tabs.map((tab) => (
          <button
            key={tab.value}
            role="tab"
            id={`${baseId}-tab-${tab.value}`}
            aria-selected={activeTab === tab.value}
            aria-controls={`${baseId}-panel-${tab.value}`}
            tabIndex={activeTab === tab.value ? 0 : -1}
            onClick={() => setActiveTab(tab.value)}
            style={{
              padding: '8px 16px',
              borderBottom: activeTab === tab.value ? '2px solid #007bff' : '2px solid transparent',
              background: 'none',
              border: 'none',
              cursor: 'pointer',
            }}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {tabs.map((tab) => (
        <div
          key={tab.value}
          role="tabpanel"
          id={`${baseId}-panel-${tab.value}`}
          aria-labelledby={`${baseId}-tab-${tab.value}`}
          tabIndex={0}
          hidden={activeTab !== tab.value}
          style={{ padding: 16 }}
        >
          <h2>{tab.label}</h2>
          <p>{tab.content}</p>
        </div>
      ))}
    </div>
  );
}
```

**Expected Output:** A tab bar with three tabs ("Profile", "Notifications", "Security"). Clicking a tab shows its panel and hides the others. Arrow keys move focus and selection between tabs. Tab moves focus into the active panel. Screen readers announce the selected tab and its panel.

**Why This Output Occurs:** The `role="tablist"` container and `role="tab"` buttons establish the ARIA structure. `aria-selected` marks the active tab; `aria-controls` and `aria-labelledby` link tabs to panels. The `tabIndex` roving pattern ensures only the active tab is in the tab order. The `handleKeyDown` function implements Arrow/Home/End navigation. The `hidden` attribute removes inactive panels from the accessibility tree.

### Real-World Cases

- **Settings pages:** Profile, Notifications, Security tabs.
- **Dashboard views:** Overview, Analytics, Reports tabs.
- **Product pages:** Description, Specifications, Reviews tabs.
- **Admin panels:** Users, Roles, Permissions tabs.

### References

- W3C WAI-ARIA Authoring Practices – Tabs Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/tabs/
- MDN Web Docs – ARIA: tab role: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/tab_role
- React Aria – Tabs: https://react-spectrum.adobe.com/react-aria/Tabs.html
- Radix UI – Tabs: https://www.radix-ui.com/primitives/docs/components/tabs
- Smashing Magazine – Building an Accessible Tab Component: https://www.smashingmagazine.com/2020/06/accessible-tab-component/

---

## Core Concept 2: Accordions

### Definitions

**Core Definition:** An accordion is a vertically stacked set of interactive headings, each of which reveals or hides an associated section of content when activated.

**Technical Definition:** The WAI-ARIA Accordion pattern uses native heading elements (`<h2>`–`<h6>`) containing a `<button>` with `aria-expanded` and `aria-controls`. The panel is a region with `role="region"` and `aria-labelledby` pointing to the button (optional but recommended for landmark navigation). Keyboard interaction uses Enter/Space to toggle, Tab to move between headers, and optional Arrow Up/Down to move between headers. Accordions can allow one panel open at a time (single-expand) or multiple panels (multi-expand). The `<details>`/`<summary>` HTML elements provide a native accordion with built-in accessibility, though styling is limited.

**Beginner-Friendly Explanation:** An accordion is like a stack of folded papers. Each paper has a title, and you can unfold it to read the content. Only one paper (or multiple, depending on the design) can be unfolded at a time. The key accessibility rule: use a real `<button>` inside a heading, not a `<div>` with a click handler. The button tells screen readers "this expands/collapses content," and the heading tells them "this is a section title."

### Purposes

- To present a large amount of content in a compact space by hiding non-essential sections.
- To let users selectively reveal the information they need.
- To provide keyboard users with Enter/Space toggling and Tab navigation.
- To announce the expanded/collapsed state to screen readers via `aria-expanded`.
- To support single-expand (only one open) or multi-expand (many open) behaviour.

### Syntax Rules and Structure

**General Syntax (Compound Component):**
```jsx
<Accordion type="single" defaultValue="item1">
  <Accordion.Item value="item1">
    <Accordion.Header>
      <Accordion.Trigger>Section 1</Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Content>Content 1</Accordion.Content>
  </Accordion.Item>
  <Accordion.Item value="item2">
    <Accordion.Header>
      <Accordion.Trigger>Section 2</Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Content>Content 2</Accordion.Content>
  </Accordion.Item>
</Accordion>
```

**Simplified Implementation:**
```jsx
import { createContext, useContext, useId, useState } from 'react';

const AccordionContext = createContext(null);

function Accordion({ type = 'single', defaultValue, children }) {
  const [openItems, setOpenItems] = useState(() => {
    if (type === 'multiple') return Array.isArray(defaultValue) ? defaultValue : [];
    return defaultValue ? [defaultValue] : [];
  });

  function toggle(value) {
    setOpenItems((prev) => {
      if (type === 'multiple') {
        return prev.includes(value) ? prev.filter((v) => v !== value) : [...prev, value];
      }
      return prev.includes(value) ? [] : [value];
    });
  }

  return (
    <AccordionContext.Provider value={{ openItems, toggle }}>
      <div>{children}</div>
    </AccordionContext.Provider>
  );
}

function Item({ value, children }) {
  const baseId = useId();
  return <div data-value={value} data-base-id={baseId}>{children}</div>;
}

function Header({ children }) {
  return <h3 style={{ margin: 0 }}>{children}</h3>;
}

function Trigger({ children }) {
  const { openItems, toggle } = useContext(AccordionContext);
  const item = useContext(ItemContext); // In practice, pass value via context
  // ... implementation with aria-expanded and aria-controls
}
```

**Full Working Implementation:**
```jsx
import { useId, useState } from 'react';

export function Accordion({ items }) {
  const [openItems, setOpenItems] = useState([]);
  const baseId = useId();

  function toggle(value) {
    setOpenItems((prev) =>
      prev.includes(value) ? prev.filter((v) => v !== value) : [...prev, value]
    );
  }

  return (
    <div className="accordion">
      {items.map((item) => {
        const isOpen = openItems.includes(item.value);
        const buttonId = `${baseId}-trigger-${item.value}`;
        const panelId = `${baseId}-panel-${item.value}`;

        return (
          <div key={item.value} className="accordion-item">
            <h3 className="accordion-header">
              <button
                id={buttonId}
                className="accordion-trigger"
                aria-expanded={isOpen}
                aria-controls={panelId}
                onClick={() => toggle(item.value)}
              >
                {item.title}
                <span aria-hidden="true">{isOpen ? '−' : '+'}</span>
              </button>
            </h3>
            <div
              id={panelId}
              role="region"
              aria-labelledby={buttonId}
              hidden={!isOpen}
              className="accordion-panel"
            >
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}

// Usage
<Accordion
  items={[
    { value: 'shipping', title: 'Shipping Information', content: 'We ship within 3–5 business days.' },
    { value: 'returns', title: 'Return Policy', content: 'Returns accepted within 30 days.' },
    { value: 'warranty', title: 'Warranty', content: 'One-year limited warranty.' },
  ]}
/>
```

**Syntax Rules:**
- Wrap each trigger in a heading element (`<h3>`, `<h4>`) to provide structure for screen readers.
- Use a real `<button>` with `aria-expanded` and `aria-controls`.
- The panel should have `role="region"` and `aria-labelledby` (optional but recommended when there are few panels).
- Use `hidden` or conditional rendering to hide collapsed panels.
- Enter/Space toggles the focused trigger; Tab moves between triggers.
- For single-expand, keep only one value in the open items array; for multi-expand, keep many.
- Use `<details>`/`<summary>` for a native, zero-JavaScript accordion when styling flexibility is not critical.

**Constraints and Limitations:**
- `role="region"` on every panel can create too many landmarks; use it only when there are few panels (3–5) or omit it and rely on the button's `aria-expanded`.
- Animating height requires measuring content (`scrollHeight`) or using CSS grid tricks (`grid-template-rows: 0fr` → `1fr`).
- Deeply nested accordions are confusing; avoid.
- Auto-collapsing other panels (single-expand) can be disorienting for screen-reader users who expect their previous panel to remain open.

### Annotated Code Example: Animated Accordion with CSS Grid

```jsx
import { useId, useState } from 'react';
import './Accordion.css';

export function Accordion({ items, allowMultiple = false }) {
  const [openItems, setOpenItems] = useState([]);
  const baseId = useId();

  function toggle(value) {
    setOpenItems((prev) => {
      if (prev.includes(value)) return prev.filter((v) => v !== value);
      return allowMultiple ? [...prev, value] : [value];
    });
  }

  return (
    <div className="accordion">
      {items.map((item) => {
        const isOpen = openItems.includes(item.value);
        const buttonId = `${baseId}-trigger-${item.value}`;
        const panelId = `${baseId}-panel-${item.value}`;

        return (
          <div key={item.value} className="accordion-item">
            <h3 className="accordion-header">
              <button
                id={buttonId}
                className="accordion-trigger"
                aria-expanded={isOpen}
                aria-controls={panelId}
                onClick={() => toggle(item.value)}
              >
                {item.title}
                <svg
                  className={`accordion-icon ${isOpen ? 'is-open' : ''}`}
                  aria-hidden="true"
                  viewBox="0 0 16 16"
                  width="16"
                  height="16"
                >
                  <path d="M4 6l4 4 4-4" stroke="currentColor" fill="none" strokeWidth="2" />
                </svg>
              </button>
            </h3>
            <div
              id={panelId}
              role="region"
              aria-labelledby={buttonId}
              className={`accordion-panel ${isOpen ? 'is-open' : ''}`}
            >
              <div className="accordion-content">{item.content}</div>
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

```css
/* Accordion.css */
.accordion-item {
  border-bottom: 1px solid #e5e7eb;
}

.accordion-trigger {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 16px 0;
  background: none;
  border: none;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  text-align: left;
}

.accordion-icon {
  transition: transform 0.2s ease;
}

.accordion-icon.is-open {
  transform: rotate(180deg);
}

/* CSS Grid trick for smooth height animation */
.accordion-panel {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s ease;
}

.accordion-panel.is-open {
  grid-template-rows: 1fr;
}

.accordion-content {
  overflow: hidden;
}
```

**Expected Output:** An accordion with three sections. Clicking a header expands its panel with a smooth height transition and rotates the chevron icon. In single-expand mode, opening one section closes the others. Keyboard users can Tab to each header and press Enter/Space to toggle.

**Why This Output Occurs:** The `aria-expanded` attribute is toggled between `true` and `false` on the button, and the panel's `grid-template-rows` transitions between `0fr` and `1fr`, creating a smooth height animation without JavaScript measurement. The `role="region"` and `aria-labelledby` link the panel to its trigger. The `hidden` attribute is not used because it would break the transition; instead, the grid row collapses to `0fr`.

### Real-World Cases

- **FAQ pages:** Frequently asked questions with expandable answers.
- **Product details:** Shipping, returns, and warranty information.
- **Settings panels:** Advanced options hidden by default.
- **Documentation:** Collapsible sections for long pages.

### References

- W3C WAI-ARIA Authoring Practices – Accordion Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/accordion/
- MDN Web Docs – `<details>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details
- Radix UI – Accordion: https://www.radix-ui.com/primitives/docs/components/accordion
- React Aria – Disclosure: https://react-spectrum.adobe.com/react-aria/Disclosure.html
- CSS-Tricks – Animating the Accordion with CSS Grid: https://css-tricks.com/animating-the-accordion-with-css-grid/

---

## Core Concept 3: Modals

### Definitions

**Core Definition:** A modal (dialog) is an overlay window that appears on top of the main content, requires the user's attention, and typically blocks interaction with the rest of the page until it is dismissed.

**Technical Definition:** The WAI-ARIA Dialog (Modal) pattern uses `role="dialog"` with `aria-modal="true"` and `aria-labelledby` pointing to the dialog's title. A focus trap keeps keyboard focus within the dialog while it is open. Escape closes the dialog. Focus is moved to the dialog (or its first focusable element) on open and restored to the trigger on close. Background content should be inert (`inert` attribute or `aria-hidden="true"`). The native `<dialog>` element with `showModal()` provides built-in focus trapping, Escape handling, and a `::backdrop` pseudo-element. React portals render the dialog outside the parent DOM hierarchy for correct stacking.

**Beginner-Friendly Explanation:** A modal is like a pop-up window that demands your attention. While it is open, you cannot click anything behind it. For keyboard users, the Tab key cycles only within the modal—this is called a "focus trap." When you close the modal, focus returns to the button that opened it, so you do not lose your place. The native `<dialog>` element does most of this for you; React portals let you render the modal outside the normal DOM tree.

### Purposes

- To demand the user's attention for a critical decision (confirm delete, accept terms).
- To collect input without navigating away from the current page.
- To display additional information (image lightbox, detail view).
- To trap focus so keyboard users do not accidentally interact with background content.
- To restore focus to the trigger on close, preserving the user's context.

### Syntax Rules and Structure

**Native `<dialog>` Implementation:**
```jsx
import { useEffect, useRef } from 'react';

export function NativeModal({ isOpen, onClose, title, children }) {
  const dialogRef = useRef(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;

    if (isOpen) {
      dialog.showModal(); // Traps focus, handles Escape
    } else {
      dialog.close();
    }
  }, [isOpen]);

  // Handle the native close event (Escape, backdrop click via form method="dialog")
  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;
    const handleClose = () => onClose();
    dialog.addEventListener('close', handleClose);
    return () => dialog.removeEventListener('close', handleClose);
  }, [onClose]);

  return (
    <dialog ref={dialogRef} aria-labelledby="modal-title" className="modal">
      <h2 id="modal-title">{title}</h2>
      {children}
      <form method="dialog">
        <button>Close</button>
      </form>
    </dialog>
  );
}
```

**Component Breakdown:**
- `dialog.showModal()`: Opens the dialog as a modal, trapping focus and blocking background interaction.
- `dialog.close()`: Closes the dialog.
- `close` event: Fires when the dialog closes (Escape, close button, form submission).
- `method="dialog"`: Closes the dialog when the form is submitted.
- `::backdrop`: The native backdrop pseudo-element for styling.

**Custom Modal with Portal and Focus Trap:**
```jsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';

const FOCUSABLE = [
  'a[href]', 'button:not([disabled])', 'textarea:not([disabled])',
  'input:not([disabled])', 'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(',');

export function Modal({ isOpen, onClose, title, description, children }) {
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
        aria-describedby={description ? 'modal-desc' : undefined}
        tabIndex={-1}
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        {description && <p id="modal-desc">{description}</p>}
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

**Syntax Rules:**
- Prefer the native `<dialog>` element with `showModal()` for built-in focus trapping and Escape handling.
- For custom modals, use `role="dialog"`, `aria-modal="true"`, and `aria-labelledby`.
- Trap focus: Tab/Shift+Tab cycle within the dialog.
- Escape closes the dialog.
- Move focus to the dialog (or its first focusable element) on open.
- Restore focus to the trigger on close.
- Use `createPortal` to render the modal at the document body level.
- Prevent clicks inside the dialog from closing it (`stopPropagation`).
- Lock body scroll while the modal is open (`overflow: hidden` on `<body>`).
- Use the `inert` attribute on background content for browsers that support it.

**Constraints and Limitations:**
- `aria-modal="true"` is not supported by all screen readers; use `inert` on background siblings as a fallback.
- Manual focus trapping is error-prone; use a library (focus-trap-react, Radix Dialog) or the native `<dialog>`.
- Portals complicate event bubbling and context propagation.
- Nested modals require careful z-index and focus management.
- Body scroll lock can cause layout shift if scrollbar width is not compensated.

### Annotated Code Example: Confirm Dialog with Native `<dialog>`

```jsx
import { useEffect, useRef, useState } from 'react';

export function ConfirmDialog() {
  const dialogRef = useRef(null);
  const [result, setResult] = useState(null);

  function open() {
    dialogRef.current?.showModal();
  }

  function handleClose(confirmed) {
    setResult(confirmed ? 'Confirmed' : 'Cancelled');
    dialogRef.current?.close();
  }

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;

    function handleCancel(e) {
      // Fired on Escape; prevent default if you want custom handling
      e.preventDefault();
      handleClose(false);
    }

    dialog.addEventListener('cancel', handleCancel);
    return () => dialog.removeEventListener('cancel', handleCancel);
  }, []);

  return (
    <div>
      <button onClick={open}>Delete Item</button>
      {result && <p role="status">Result: {result}</p>}

      <dialog ref={dialogRef} aria-labelledby="confirm-title" className="confirm-dialog">
        <h2 id="confirm-title">Confirm Deletion</h2>
        <p>This action cannot be undone. Are you sure?</p>
        <div className="dialog-actions">
          <button onClick={() => handleClose(false)}>Cancel</button>
          <button onClick={() => handleClose(true)} className="danger">Delete</button>
        </div>
      </dialog>
    </div>
  );
}
```

```css
.confirm-dialog {
  border: none;
  border-radius: 12px;
  padding: 24px;
  max-width: 400px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.confirm-dialog::backdrop {
  background: rgba(0, 0, 0, 0.5);
}

.dialog-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
  margin-top: 16px;
}

.danger {
  background: #ef4444;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
}
```

**Expected Output:** A "Delete Item" button that opens a modal with a backdrop. The modal traps focus, closes on Escape, and shows a "Confirmed" or "Cancelled" message based on the user's choice.

**Why This Output Occurs:** `showModal()` opens the dialog as a modal, trapping focus and applying the `::backdrop` styles. The `cancel` event fires on Escape; the handler calls `handleClose(false)`. The Cancel and Delete buttons call `handleClose` with different values. The `aria-labelledby` attribute links the dialog to its title.

### Real-World Cases

- **Confirmation dialogs:** Delete, discard, or irreversible actions.
- **Login/signup modals:** Authentication without leaving the page.
- **Image lightboxes:** Full-screen image viewing.
- **Form wizards in modals:** Multi-step processes.
- **Cookie consent banners:** Modal dialogs for legal consent.

### References

- W3C WAI-ARIA Authoring Practices – Dialog (Modal) Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- MDN Web Docs – `<dialog>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog
- MDN Web Docs – `showModal()`: https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal
- Radix UI – Dialog: https://www.radix-ui.com/primitives/docs/components/dialog
- React Aria – Dialog: https://react-spectrum.adobe.com/react-aria/Dialog.html

---

## Core Concept 4: Dropdowns

### Definitions

**Core Definition:** A dropdown is a menu that opens on click or hover, presenting a list of options or actions from which the user can select one.

**Technical Definition:** The WAI-ARIA Menu Button pattern uses a `<button>` with `aria-haspopup="menu"` and `aria-expanded`, and a `role="menu"` container with `role="menuitem"` children. Keyboard interaction: Enter/Space/Down Arrow opens the menu and focuses the first item; Arrow Up/Down moves between items; Home/End jump to first/last; Escape closes and returns focus to the button; Tab closes the menu. Clicking outside closes the menu. Focus is trapped within the menu while open (or focus management is used without a full trap). For simple selection lists, use `<select>` or a listbox pattern instead of a menu.

**Beginner-Friendly Explanation:** A dropdown is like a pull-down menu on a website. You click a button, and a list of options appears. You can click an option or use the arrow keys to navigate. Escape closes the menu. The tricky part is distinguishing between a "menu" (a list of actions, like "Edit / Copy / Delete") and a "listbox" (a list of choices, like a select dropdown). Menus use `role="menu"`; listboxes use `role="listbox"`.

### Purposes

- To present a list of actions without cluttering the interface.
- To let users select from a list of options in a compact space.
- To provide keyboard navigation (Arrow keys, Enter, Escape).
- To close automatically when the user clicks outside or presses Escape.
- To announce the expanded/collapsed state to screen readers.

### Syntax Rules and Structure

**General Syntax (Menu):**
```jsx
<div className="dropdown">
  <button
    aria-haspopup="menu"
    aria-expanded={isOpen}
    aria-controls="dropdown-menu"
    onClick={toggle}
    onKeyDown={handleButtonKeyDown}
  >
    Actions
  </button>
  {isOpen && (
    <ul id="dropdown-menu" role="menu">
      <li role="menuitem" tabIndex={-1}>Edit</li>
      <li role="menuitem" tabIndex={-1}>Duplicate</li>
      <li role="menuitem" tabIndex={-1}>Delete</li>
    </ul>
  )}
</div>
```

**Full Implementation with Click Outside and Keyboard Navigation:**
```jsx
import { useEffect, useRef, useState } from 'react';

export function Dropdown({ label, items, onSelect }) {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const menuRef = useRef(null);
  const buttonRef = useRef(null);

  // Close on outside click
  useEffect(() => {
    if (!isOpen) return;
    function handleClickOutside(e) {
      if (!menuRef.current?.contains(e.target) && !buttonRef.current?.contains(e.target)) {
        setIsOpen(false);
      }
    }
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [isOpen]);

  // Focus the active item
  useEffect(() => {
    if (isOpen && activeIndex >= 0) {
      const items = menuRef.current?.querySelectorAll('[role="menuitem"]');
      items?.[activeIndex]?.focus();
    }
  }, [isOpen, activeIndex]);

  function handleButtonKeyDown(e) {
    if (e.key === 'ArrowDown' || e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      setIsOpen(true);
      setActiveIndex(0);
    }
  }

  function handleMenuKeyDown(e) {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex((i) => (i + 1) % items.length);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex((i) => (i - 1 + items.length) % items.length);
        break;
      case 'Home':
        e.preventDefault();
        setActiveIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setActiveIndex(items.length - 1);
        break;
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        buttonRef.current?.focus();
        break;
      case 'Tab':
        setIsOpen(false);
        break;
    }
  }

  function handleSelect(item) {
    onSelect(item);
    setIsOpen(false);
    buttonRef.current?.focus();
  }

  return (
    <div className="dropdown">
      <button
        ref={buttonRef}
        aria-haspopup="menu"
        aria-expanded={isOpen}
        aria-controls="dropdown-menu"
        onClick={() => setIsOpen((o) => !o)}
        onKeyDown={handleButtonKeyDown}
      >
        {label}
      </button>

      {isOpen && (
        <ul
          ref={menuRef}
          id="dropdown-menu"
          role="menu"
          aria-label={label}
          onKeyDown={handleMenuKeyDown}
          className="dropdown-menu"
        >
          {items.map((item, index) => (
            <li
              key={item.value}
              role="menuitem"
              tabIndex={-1}
              onClick={() => handleSelect(item)}
              onMouseEnter={() => setActiveIndex(index)}
              className={activeIndex === index ? 'is-active' : ''}
            >
              {item.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

// Usage
<Dropdown
  label="Actions"
  items={[
    { value: 'edit', label: 'Edit' },
    { value: 'duplicate', label: 'Duplicate' },
    { value: 'delete', label: 'Delete' },
  ]}
  onSelect={(item) => console.log(item.value)}
/>
```

**Component Breakdown:**
- `aria-haspopup="menu"`: Tells assistive tech that the button opens a menu.
- `aria-expanded`: Indicates whether the menu is open.
- `role="menu"` and `role="menuitem"`: Establish the menu structure.
- `tabIndex={-1}` on items: Focus is managed by the arrow keys, not Tab.
- `handleMenuKeyDown`: Arrow Up/Down, Home/End, Escape, Tab.
- `handleClickOutside`: Closes the menu when clicking outside.

**Syntax Rules:**
- Use `aria-haspopup="menu"` and `aria-expanded` on the trigger button.
- Use `role="menu"` on the container and `role="menuitem"` on each item.
- Use `tabIndex={-1}` on menu items; focus is moved programmatically.
- Arrow Up/Down navigates; Home/End jumps; Escape closes and returns focus to the button; Tab closes the menu.
- Close on outside click.
- For selection lists (not action menus), use `role="listbox"` or a native `<select>`.
- Use `aria-label` or `aria-labelledby` on the menu to describe its purpose.

**Constraints and Limitations:**
- Menus are for *actions*, not navigation; use links for navigation.
- `role="menu"` should not be used for a simple list of links; that is a navigation list.
- Focus trapping in menus is different from modals: Tab closes the menu rather than cycling within it.
- Positioning the menu (floating-ui, Popper) requires additional logic for viewport collision.
- Submenus add significant complexity; use a library (Radix DropdownMenu) for nested menus.

### Annotated Code Example: User Menu with Keyboard Navigation

```jsx
import { useEffect, useRef, useState } from 'react';

export function UserMenu({ user }) {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const menuRef = useRef(null);
  const buttonRef = useRef(null);

  const items = [
    { value: 'profile', label: 'Your Profile', href: '/profile' },
    { value: 'settings', label: 'Settings', href: '/settings' },
    { value: 'logout', label: 'Sign Out', action: 'logout' },
  ];

  useEffect(() => {
    if (!isOpen) return;
    function handleClick(e) {
      if (!menuRef.current?.contains(e.target) && !buttonRef.current?.contains(e.target)) {
        setIsOpen(false);
      }
    }
    document.addEventListener('mousedown', handleClick);
    return () => document.removeEventListener('mousedown', handleClick);
  }, [isOpen]);

  useEffect(() => {
    if (isOpen && activeIndex >= 0) {
      menuRef.current?.querySelectorAll('[role="menuitem"]')[activeIndex]?.focus();
    }
  }, [isOpen, activeIndex]);

  function handleMenuKeyDown(e) {
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      setActiveIndex((i) => (i + 1) % items.length);
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      setActiveIndex((i) => (i - 1 + items.length) % items.length);
    } else if (e.key === 'Escape') {
      setIsOpen(false);
      buttonRef.current?.focus();
    } else if (e.key === 'Tab') {
      setIsOpen(false);
    }
  }

  return (
    <div className="user-menu">
      <button
        ref={buttonRef}
        aria-haspopup="menu"
        aria-expanded={isOpen}
        aria-label="User menu"
        onClick={() => { setIsOpen((o) => !o); setActiveIndex(0); }}
        onKeyDown={(e) => {
          if (e.key === 'ArrowDown') { e.preventDefault(); setIsOpen(true); setActiveIndex(0); }
        }}
      >
        <img src={user.avatar} alt="" width={32} height={32} />
      </button>

      {isOpen && (
        <ul
          ref={menuRef}
          role="menu"
          aria-label="User menu"
          onKeyDown={handleMenuKeyDown}
          className="user-menu-list"
        >
          {items.map((item, i) => (
            <li
              key={item.value}
              role="menuitem"
              tabIndex={-1}
              className={activeIndex === i ? 'is-active' : ''}
            >
              {item.href ? (
                <a href={item.href}>{item.label}</a>
              ) : (
                <button onClick={() => console.log(item.action)}>{item.label}</button>
              )}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Expected Output:** A user avatar button that opens a menu with "Your Profile", "Settings", and "Sign Out". Arrow keys navigate the menu; Escape closes it and returns focus to the avatar; clicking outside closes it.

**Why This Output Occurs:** The `aria-haspopup="menu"` and `aria-expanded` attributes define the button's relationship to the menu. The `role="menu"` and `role="menuitem"` establish the menu structure. The `handleMenuKeyDown` function implements Arrow/Escape/Tab navigation. The click-outside handler closes the menu when the user clicks elsewhere.

### Real-World Cases

- **User menus:** Profile, settings, logout.
- **Action menus:** Edit, duplicate, delete on a table row.
- **Filter menus:** Sort by, filter by.
- **Navigation dropdowns:** Product categories, documentation sections.

### References

- W3C WAI-ARIA Authoring Practices – Menu Button Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/
- MDN Web Docs – ARIA: menu role: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/menu_role
- Radix UI – Dropdown Menu: https://www.radix-ui.com/primitives/docs/components/dropdown-menu
- React Aria – Menu: https://react-spectrum.adobe.com/react-aria/Menu.html
- Floating UI – Positioning: https://floating-ui.com/

---

## Core Concept 5: Pagination

### Definitions

**Core Definition:** Pagination is the practice of dividing a large dataset into discrete pages, providing navigation controls to move between them.

**Technical Definition:** Pagination can be **offset/limit** (`?page=2&limit=10`) or **cursor-based** (`?cursor=abc123`). The UI typically includes Previous/Next buttons, page numbers, and sometimes first/last and ellipsis. Accessibility requires `aria-label="Pagination"` on the container, `aria-current="page"` on the active page, and descriptive labels on Previous/Next buttons. Keyboard users navigate with Tab and activate with Enter/Space. React implementations either use URL state (`useSearchParams`) for shareability or local state for transient lists. TanStack Query's `useQuery` with `keepPreviousData` avoids loading flashes during page transitions.

**Beginner-Friendly Explanation:** Pagination is like a book with numbered pages. Instead of showing all 1,000 products at once, you show 10 at a time and provide "Next" and "Previous" buttons plus page numbers. The tricky part is accessibility: screen readers need to know which page is current, and keyboard users need clear focus indicators. For very large datasets, cursor-based pagination is more stable than page numbers.

### Purposes

- To divide large datasets into manageable chunks.
- To reduce initial load time and memory usage.
- To provide shareable, bookmarkable URLs (when using URL state).
- To announce the current page to screen readers via `aria-current="page"`.
- To support keyboard navigation with clear focus indicators.

### Syntax Rules and Structure

**Offset/Limit with URL State:**
```jsx
import { useSearchParams } from 'react-router';

export function Pagination({ totalPages }) {
  const [searchParams, setSearchParams] = useSearchParams();
  const currentPage = Number(searchParams.get('page') ?? '1');

  function goToPage(page) {
    setSearchParams({ page: String(page) });
  }

  const pages = Array.from({ length: totalPages }, (_, i) => i + 1);

  return (
    <nav aria-label="Pagination">
      <ul className="pagination">
        <li>
          <button
            onClick={() => goToPage(currentPage - 1)}
            disabled={currentPage === 1}
            aria-label="Previous page"
          >
            Previous
          </button>
        </li>
        {pages.map((page) => (
          <li key={page}>
            <button
              onClick={() => goToPage(page)}
              aria-current={page === currentPage ? 'page' : undefined}
              aria-label={`Page ${page}`}
            >
              {page}
            </button>
          </li>
        ))}
        <li>
          <button
            onClick={() => goToPage(currentPage + 1)}
            disabled={currentPage === totalPages}
            aria-label="Next page"
          >
            Next
          </button>
        </li>
      </ul>
    </nav>
  );
}
```

**Component Breakdown:**
- `nav aria-label="Pagination"`: Identifies the pagination as a navigation landmark.
- `aria-current="page"`: Marks the active page.
- `aria-label="Previous page"` / `"Next page"`: Descriptive labels for icon buttons.
- `disabled`: Prevents navigation beyond the first/last page.

**Pagination with Ellipsis:**
```jsx
function getPaginationRange(current, total) {
  const delta = 2;
  const range = [];
  const rangeWithDots = [];
  let last;

  for (let i = 1; i <= total; i++) {
    if (i === 1 || i === total || (i >= current - delta && i <= current + delta)) {
      range.push(i);
    }
  }

  for (const i of range) {
    if (last) {
      if (i - last === 2) {
        rangeWithDots.push(last + 1);
      } else if (i - last > 2) {
        rangeWithDots.push('...');
      }
    }
    rangeWithDots.push(i);
    last = i;
  }

  return rangeWithDots;
}
```

**TanStack Query with `keepPreviousData`:**
```jsx
import { useQuery, keepPreviousData } from '@tanstack/react-query';

function PaginatedList() {
  const [page, setPage] = useState(1);

  const { data, isPlaceholderData } = useQuery({
    queryKey: ['items', page],
    queryFn: () => fetchItems({ page, limit: 10 }),
    placeholderData: keepPreviousData,
  });

  return (
    <div>
      <ul>{data?.items.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
      <div>
        <button onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
          Previous
        </button>
        <span>Page {page}</span>
        <button
          onClick={() => setPage((p) => p + 1)}
          disabled={isPlaceholderData || !data?.hasMore}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

**Syntax Rules:**
- Wrap pagination in `<nav aria-label="Pagination">`.
- Use `aria-current="page"` on the active page button/link.
- Provide `aria-label` on icon-only Previous/Next buttons.
- Disable Previous on the first page and Next on the last page.
- Use `placeholderData: keepPreviousData` to avoid loading flashes.
- For cursor-based pagination, include the cursor in the query key.
- Use `rel="prev"` and `rel="next"` on links for SEO (when using `<a>`).
- Consider `role="list"` on the `<ul>` for screen-reader count announcements.

**Constraints and Limitations:**
- Offset/limit pagination drifts when items are inserted or deleted between requests.
- Cursor-based pagination cannot jump to arbitrary pages.
- Too many page buttons clutter the UI; use ellipsis or a "jump to page" input.
- `aria-current="page"` should be on only one element per page.
- Disabled buttons are not focusable; consider using `aria-disabled` on links instead.

### Annotated Code Example: Accessible Pagination

```jsx
import { useState } from 'react';

export function AccessiblePagination({ totalPages, onPageChange }) {
  const [currentPage, setCurrentPage] = useState(1);

  function goTo(page) {
    setCurrentPage(page);
    onPageChange(page);
  }

  return (
    <nav aria-label="Search results pages">
      <ul className="pagination" role="list">
        <li>
          <button
            onClick={() => goTo(currentPage - 1)}
            disabled={currentPage === 1}
            aria-label="Go to previous page"
          >
            ‹ Previous
          </button>
        </li>

        {Array.from({ length: totalPages }, (_, i) => i + 1).map((page) => (
          <li key={page}>
            <button
              onClick={() => goTo(page)}
              aria-current={page === currentPage ? 'page' : undefined}
              aria-label={`Go to page ${page}`}
            >
              {page}
            </button>
          </li>
        ))}

        <li>
          <button
            onClick={() => goTo(currentPage + 1)}
            disabled={currentPage === totalPages}
            aria-label="Go to next page"
          >
            Next ›
          </button>
        </li>
      </ul>

      <p role="status" aria-live="polite">
        Page {currentPage} of {totalPages}
      </p>
    </nav>
  );
}
```

**Expected Output:** A pagination bar with Previous, numbered pages, and Next buttons. The active page is highlighted and marked with `aria-current="page"`. A live region announces "Page X of Y" when the page changes. Screen readers announce the current page and available navigation.

**Why This Output Occurs:** The `nav aria-label` identifies the pagination landmark. `aria-current="page"` marks the active page. `aria-label` on each button provides context for screen readers. The `role="status"` live region announces page changes politely.

### Real-World Cases

- **Search results:** Product listings, article search, user directories.
- **Admin tables:** Data grids with thousands of rows.
- **Blog archives:** Posts by month, category, or tag.
- **Comment sections:** Paginated comments or reviews.

### References

- W3C WAI-ARIA Authoring Practices – Pagination: https://www.w3.org/WAI/ARIA/apg/patterns/
- MDN Web Docs – `aria-current`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-current
- TanStack Query – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- React Router – useSearchParams: https://reactrouter.com/api/hooks/useSearchParams
- Inclusive Components – Pagination: https://inclusive-components.design/pagination/

---

## Core Concept 6: Wizards

### Definitions

**Core Definition:** A wizard is a multi-step form or process that guides users through a sequence of steps, collecting data or making decisions at each step before proceeding to the next.

**Technical Definition:** A wizard maintains all form state in a persistent parent shell, rendering one step at a time while keeping the others mounted (or preserving their state). Each step has a validation gate: the "Next" button is enabled only when the current step's fields are valid. Navigation includes Back/Next, optional step indicators, and a final Review/Submit step. React Hook Form's `trigger(fieldNames)` validates only the current step's fields; TanStack Query's `useMutation` handles the final submission. State persistence across steps is achieved via `FormProvider` and `useFormContext`. Progress indicators use `role="list"` with `aria-current="step"` on the active step.

**Beginner-Friendly Explanation:** A wizard is like a guided interview. Instead of one long form, you answer a few questions at a time. The interviewer (parent shell) keeps all your previous answers, so you never lose data when going back. You cannot proceed to the next question until you have answered the current one. A progress bar shows how far you have come.

### Purposes

- To break long forms into manageable steps, reducing cognitive load.
- To preserve all data across step transitions (Back/Next).
- To validate only the current step's fields before allowing progression.
- To provide a review step that summarises all prior data.
- To support conditional branching (skip steps based on answers).
- To show progress to the user.

### Syntax Rules and Structure

**General Syntax:**
```jsx
<Wizard>
  <Wizard.Stepper steps={steps} currentStep={currentStep} />
  <Wizard.Step index={0}><PersonalInfo /></Wizard.Step>
  <Wizard.Step index={1}><Address /></Wizard.Step>
  <Wizard.Step index={2}><Review /></Wizard.Step>
  <Wizard.Navigation onBack={back} onNext={next} isFirst={...} isLast={...} />
</Wizard>
```

**Full Implementation with React Hook Form:**
```jsx
import { useState } from 'react';
import { useForm, FormProvider, useFormContext } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const fullSchema = z.object({
  firstName: z.string().min(1, 'First name required'),
  lastName: z.string().min(1, 'Last name required'),
  email: z.string().email('Invalid email'),
  street: z.string().min(1, 'Street required'),
  city: z.string().min(1, 'City required'),
});

const steps = [
  { id: 'personal', label: 'Personal Info', fields: ['firstName', 'lastName', 'email'] },
  { id: 'address', label: 'Address', fields: ['street', 'city'] },
  { id: 'review', label: 'Review', fields: [] },
];

function Stepper({ currentStep }) {
  return (
    <ol className="stepper" aria-label="Progress">
      {steps.map((step, i) => (
        <li
          key={step.id}
          aria-current={i === currentStep ? 'step' : undefined}
          className={i === currentStep ? 'is-active' : i < currentStep ? 'is-complete' : ''}
        >
          <span className="stepper-number">{i + 1}</span>
          <span className="stepper-label">{step.label}</span>
        </li>
      ))}
    </ol>
  );
}

function PersonalStep() {
  const { register, formState: { errors } } = useFormContext();
  return (
    <fieldset>
      <legend>Personal Info</legend>
      <label>First Name <input {...register('firstName')} /></label>
      {errors.firstName && <span role="alert">{errors.firstName.message}</span>}
      <label>Last Name <input {...register('lastName')} /></label>
      {errors.lastName && <span role="alert">{errors.lastName.message}</span>}
      <label>Email <input type="email" {...register('email')} /></label>
      {errors.email && <span role="alert">{errors.email.message}</span>}
    </fieldset>
  );
}

function AddressStep() {
  const { register, formState: { errors } } = useFormContext();
  return (
    <fieldset>
      <legend>Address</legend>
      <label>Street <input {...register('street')} /></label>
      {errors.street && <span role="alert">{errors.street.message}</span>}
      <label>City <input {...register('city')} /></label>
      {errors.city && <span role="alert">{errors.city.message}</span>}
    </fieldset>
  );
}

function ReviewStep() {
  const { getValues } = useFormContext();
  const data = getValues();
  return (
    <fieldset>
      <legend>Review</legend>
      <dl>
        {Object.entries(data).map(([key, value]) => (
          <div key={key}>
            <dt>{key}</dt>
            <dd>{value}</dd>
          </div>
        ))}
      </dl>
    </fieldset>
  );
}

export function Wizard() {
  const [step, setStep] = useState(0);
  const methods = useForm({
    resolver: zodResolver(fullSchema),
    mode: 'onBlur',
    defaultValues: { firstName: '', lastName: '', email: '', street: '', city: '' },
  });

  async function next() {
    const valid = await methods.trigger(steps[step].fields);
    if (valid) setStep((s) => s + 1);
  }

  function back() {
    setStep((s) => Math.max(0, s - 1));
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit((data) => alert(JSON.stringify(data)))}>
        <Stepper currentStep={step} />

        <div style={{ display: step === 0 ? 'block' : 'none' }}><PersonalStep /></div>
        <div style={{ display: step === 1 ? 'block' : 'none' }}><AddressStep /></div>
        <div style={{ display: step === 2 ? 'block' : 'none' }}><ReviewStep /></div>

        <div className="wizard-navigation">
          {step > 0 && <button type="button" onClick={back}>Back</button>}
          {step < steps.length - 1 && <button type="button" onClick={next}>Next</button>}
          {step === steps.length - 1 && <button type="submit">Submit</button>}
        </div>
      </form>
    </FormProvider>
  );
}
```

**Component Breakdown:**
- `Stepper`: Renders the step indicator with `aria-current="step"`.
- `FormProvider`: Provides form methods to all steps.
- `trigger(steps[step].fields)`: Validates only the current step's fields.
- `display: none`: Hides inactive steps without unmounting them, preserving their state.
- `getValues()`: Reads all values for the review step.

**Syntax Rules:**
- Keep all form state in a parent component using `FormProvider`.
- Render all steps but toggle visibility (`display: none`) rather than unmounting them.
- Validate only the current step's fields before advancing (`trigger(fieldNames)`).
- Use `aria-current="step"` on the active step in the stepper.
- Provide Back/Next/Submit buttons with clear labels.
- Persist the draft to `localStorage` or `sessionStorage` for multi-session forms.
- Use `useTransition` for non-blocking step transitions (React 19).

**Constraints and Limitations:**
- Unmounting step components destroys their state; keep them mounted or lift state.
- `trigger` returns a promise; await it before advancing.
- Wizards are not page navigation; treating them as pages causes flicker and data loss.
- Conditional steps require careful state management to skip without losing data.
- Deep wizards (7+ steps) are tiring; consider grouping steps.

### Annotated Code Example: Three-Step Wizard with Validation

```jsx
import { useState } from 'react';
import { useForm, FormProvider, useFormContext } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  firstName: z.string().min(1, 'Required'),
  lastName: z.string().min(1, 'Required'),
  email: z.string().email('Invalid email'),
  street: z.string().min(1, 'Required'),
  city: z.string().min(1, 'Required'),
});

const steps = [
  { fields: ['firstName', 'lastName', 'email'], label: 'Personal' },
  { fields: ['street', 'city'], label: 'Address' },
  { fields: [], label: 'Review' },
];

function StepContent({ index }) {
  const { register, formState: { errors }, getValues } = useFormContext();

  if (index === 0) {
    return (
      <>
        <label>First Name <input {...register('firstName')} /></label>
        {errors.firstName && <span role="alert">{errors.firstName.message}</span>}
        <label>Last Name <input {...register('lastName')} /></label>
        {errors.lastName && <span role="alert">{errors.lastName.message}</span>}
        <label>Email <input {...register('email')} /></label>
        {errors.email && <span role="alert">{errors.email.message}</span>}
      </>
    );
  }

  if (index === 1) {
    return (
      <>
        <label>Street <input {...register('street')} /></label>
        {errors.street && <span role="alert">{errors.street.message}</span>}
        <label>City <input {...register('city')} /></label>
        {errors.city && <span role="alert">{errors.city.message}</span>}
      </>
    );
  }

  return <pre>{JSON.stringify(getValues(), null, 2)}</pre>;
}

export function Wizard() {
  const [step, setStep] = useState(0);
  const methods = useForm({
    resolver: zodResolver(schema),
    mode: 'onBlur',
    defaultValues: { firstName: '', lastName: '', email: '', street: '', city: '' },
  });

  async function next() {
    const valid = await methods.trigger(steps[step].fields);
    if (valid) setStep((s) => s + 1);
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit((d) => alert(JSON.stringify(d)))}>
        <ol aria-label="Progress">
          {steps.map((s, i) => (
            <li key={s.label} aria-current={i === step ? 'step' : undefined}>
              {i + 1}. {s.label}
            </li>
          ))}
        </ol>

        {steps.map((_, i) => (
          <div key={i} style={{ display: i === step ? 'block' : 'none' }}>
            <StepContent index={i} />
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

**Expected Output:** A three-step wizard with a progress indicator. Step 1 collects name and email; Step 2 collects address; Step 3 shows a review of all data. Clicking "Next" validates only the current step. "Back" preserves all data. "Submit" shows an alert with the complete data.

**Why This Output Occurs:** All steps are rendered but hidden with `display: none`, so their registered inputs remain in the form context. `trigger(steps[step].fields)` validates only the current step's fields. The `aria-current="step"` on the active step announces progress to screen readers.

### Real-World Cases

- **Onboarding flows:** Account setup, profile completion, preferences.
- **Checkout processes:** Cart, shipping, payment, review.
- **Loan applications:** Personal, employment, financial, review.
- **Insurance quotes:** Vehicle, driver, coverage, review.
- **Survey tools:** Multi-section questionnaires with branching.

### References

- React Hook Form – `trigger`: https://react-hook-form.com/docs/useform/trigger
- React Hook Form – FormProvider: https://react-hook-form.com/docs/formprovider
- W3C WAI-ARIA Authoring Practices – Steps: https://www.w3.org/WAI/ARIA/apg/patterns/
- Educative – Multi-Step and Wizard Forms: https://www.educative.io/courses/learn-react/lta/multi-step-and-wizard-forms
- AppSignal – Smooth Async Transitions in React 19: https://blog.appsignal.com/2025/08/27/smooth-async-transitions-in-react-19.html

---

## Core Concept 7: Expand/Collapse Interfaces

### Definitions

**Core Definition:** Expand/collapse is a disclosure pattern where a trigger reveals or hides additional content, reducing visual clutter while keeping information accessible on demand.

**Technical Definition:** The WAI-ARIA Disclosure pattern uses a `<button>` with `aria-expanded` and `aria-controls`, and a content region that is shown/hidden. Unlike accordions (which group multiple disclosures under headings), a standalone disclosure is used for individual "show more" links, collapsible sidebars, or expandable table rows. The native `<details>`/`<summary>` element provides this pattern with zero JavaScript. Animated expand/collapse is achieved with CSS grid (`grid-template-rows: 0fr` → `1fr`) or measured height (`scrollHeight`). `prefers-reduced-motion` should be respected to disable animations for users who prefer reduced motion.

**Beginner-Friendly Explanation:** Expand/collapse is like a "Read more" link. You click it, and more text appears. Click again, and it hides. The accessibility rule is simple: use a `<button>` with `aria-expanded` and `aria-controls`. For animation, the CSS grid trick is the modern, clean approach—no JavaScript height measurement needed.

### Purposes

- To hide secondary content until the user requests it.
- To reduce visual clutter on dense pages.
- To provide keyboard users with Enter/Space toggling.
- To announce the expanded/collapsed state via `aria-expanded`.
- To animate the transition smoothly without layout jank.

### Syntax Rules and Structure

**Native `<details>`/`<summary>`:**
```html
<details>
  <summary>Show more details</summary>
  <p>Additional content goes here.</p>
</details>
```

**Component Breakdown:**
- `<details>`: The container; toggles open/closed.
- `<summary>`: The trigger; must be the first child.
- The `open` attribute controls the expanded state.
- Screen readers announce "expanded" or "collapsed" automatically.

**Custom Disclosure with Button:**
```jsx
import { useId, useState } from 'react';

export function Disclosure({ title, children, defaultOpen = false }) {
  const [isOpen, setIsOpen] = useState(defaultOpen);
  const id = useId();

  return (
    <div className="disclosure">
      <button
        aria-expanded={isOpen}
        aria-controls={`${id}-content`}
        onClick={() => setIsOpen((o) => !o)}
        className="disclosure-trigger"
      >
        {title}
        <span aria-hidden="true">{isOpen ? '−' : '+'}</span>
      </button>
      <div
        id={`${id}-content`}
        className={`disclosure-content ${isOpen ? 'is-open' : ''}`}
      >
        <div className="disclosure-inner">{children}</div>
      </div>
    </div>
  );
}
```

```css
.disclosure-trigger {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding: 12px 0;
  background: none;
  border: none;
  font-size: 1rem;
  cursor: pointer;
}

.disclosure-content {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s ease;
}

.disclosure-content.is-open {
  grid-template-rows: 1fr;
}

.disclosure-inner {
  overflow: hidden;
}

@media (prefers-reduced-motion: reduce) {
  .disclosure-content {
    transition: none;
  }
}
```

**Syntax Rules:**
- Use `<details>`/`<summary>` for the simplest, most accessible implementation.
- For custom styling, use a `<button>` with `aria-expanded` and `aria-controls`.
- The content region should have an `id` matching `aria-controls`.
- Use the CSS grid trick (`grid-template-rows: 0fr` → `1fr`) for smooth height animation.
- Respect `prefers-reduced-motion` by disabling transitions.
- Use `overflow: hidden` on the inner wrapper to clip content during collapse.
- Do not use `hidden` if you want to animate; use grid rows instead.

**Constraints and Limitations:**
- The CSS grid trick requires a wrapper element with `overflow: hidden`.
- `<details>`/`<summary>` styling is limited (no smooth animation without JavaScript in some browsers).
- Animating `height: auto` is not possible with CSS alone; the grid trick is the modern solution.
- Nested disclosures are confusing; use accordions instead.
- The `aria-controls` attribute is technically optional but recommended for older screen readers.

### Annotated Code Example: Animated "Show More" Disclosure

```jsx
import { useId, useState } from 'react';
import './Disclosure.css';

export function ShowMore({ preview, full }) {
  const [isOpen, setIsOpen] = useState(false);
  const id = useId();

  return (
    <div>
      <p>{isOpen ? full : preview}</p>
      <button
        aria-expanded={isOpen}
        aria-controls={`${id}-content`}
        onClick={() => setIsOpen((o) => !o)}
      >
        {isOpen ? 'Show less' : 'Show more'}
      </button>
      <div
        id={`${id}-content`}
        className={`disclosure-content ${isOpen ? 'is-open' : ''}`}
      >
        <div className="disclosure-inner">
          <p>{full}</p>
        </div>
      </div>
    </div>
  );
}
```

**Expected Output:** A short preview text with a "Show more" button. Clicking the button expands a hidden section with the full text, animated smoothly. The button text changes to "Show less," and `aria-expanded` toggles between `true` and `false`.

**Why This Output Occurs:** The `aria-expanded` attribute on the button indicates the state. The `disclosure-content` div uses CSS grid to animate between `0fr` and `1fr`. The `aria-controls` links the button to the content region.

### Real-World Cases

- **"Read more" links:** Blog post previews, product descriptions.
- **Collapsible sidebars:** Navigation panels that expand/collapse.
- **Expandable table rows:** Show details of a selected row.
- **FAQ items:** Individual questions that expand.
- **Code blocks:** Collapsible source code snippets.

### References

- W3C WAI-ARIA Authoring Practices – Disclosure Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/
- MDN Web Docs – `<details>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details
- CSS-Tricks – Animating the Accordion with CSS Grid: https://css-tricks.com/animating-the-accordion-with-css-grid/
- Inclusive Components – Collapsible Sections: https://inclusive-components.design/collapsible-sections/
- Radix UI – Collapsible: https://www.radix-ui.com/primitives/docs/components/collapsible

---

## Core Concept 8: Dynamic Navigation

### Definitions

**Core Definition:** Dynamic navigation is a navigation system that adapts its structure, appearance, and behaviour based on application state, user role, viewport size, or current route.

**Technical Definition:** Dynamic navigation encompasses responsive navigation (hamburger menus on mobile, horizontal menus on desktop), role-based navigation (menu items shown/hidden based on permissions), active-state management (`aria-current="page"` on the current route), breadcrumbs, and nested navigation (multi-level menus). It uses `useLocation` (React Router) or `usePathname` (Next.js) to determine the active route, `useMediaQuery` or container queries for responsive behaviour, and `aria-current="page"` for the active link. Mobile navigation typically uses a disclosure or dialog pattern with focus trapping. Breadcrumbs use `nav aria-label="Breadcrumb"` with an ordered list.

**Beginner-Friendly Explanation:** Dynamic navigation is navigation that changes based on context. On a phone, the menu collapses into a hamburger button; on desktop, it is a horizontal bar. If you are an admin, you see admin links; if you are a regular user, you do not. The current page is highlighted with `aria-current="page"`. Breadcrumbs show you where you are in the hierarchy. All of this must work for keyboard and screen-reader users.

### Purposes

- To adapt navigation to different viewport sizes (responsive).
- To show or hide navigation items based on user permissions (role-based).
- To indicate the current page with `aria-current="page"`.
- To provide hierarchical context via breadcrumbs.
- To support multi-level nested navigation.
- To announce navigation changes to screen readers.

### Syntax Rules and Structure

**Responsive Navigation (Mobile Hamburger):**
```jsx
import { useState } from 'react';
import { NavLink, useLocation } from 'react-router';

export function Navigation() {
  const [isOpen, setIsOpen] = useState(false);
  const location = useLocation();

  const links = [
    { to: '/', label: 'Home' },
    { to: '/products', label: 'Products' },
    { to: '/about', label: 'About' },
    { to: '/contact', label: 'Contact' },
  ];

  return (
    <nav aria-label="Main navigation">
      {/* Hamburger button (visible on mobile) */}
      <button
        className="nav-toggle"
        aria-expanded={isOpen}
        aria-controls="nav-menu"
        aria-label={isOpen ? 'Close menu' : 'Open menu'}
        onClick={() => setIsOpen((o) => !o)}
      >
        <span aria-hidden="true">{isOpen ? '✕' : '☰'}</span>
      </button>

      {/* Navigation menu */}
      <ul
        id="nav-menu"
        className={`nav-list ${isOpen ? 'is-open' : ''}`}
      >
        {links.map((link) => (
          <li key={link.to}>
            <NavLink
              to={link.to}
              end={link.to === '/'}
              className={({ isActive }) => (isActive ? 'nav-link is-active' : 'nav-link')}
              aria-current={location.pathname === link.to ? 'page' : undefined}
              onClick={() => setIsOpen(false)}
            >
              {link.label}
            </NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

```css
.nav-toggle {
  display: block;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
}

.nav-list {
  display: none;
  list-style: none;
  padding: 0;
}

.nav-list.is-open {
  display: flex;
  flex-direction: column;
}

@media (min-width: 768px) {
  .nav-toggle {
    display: none;
  }
  .nav-list {
    display: flex;
    flex-direction: row;
    gap: 16px;
  }
}
```

**Breadcrumbs:**
```jsx
import { Link, useLocation } from 'react-router';

export function Breadcrumbs() {
  const location = useLocation();
  const pathnames = location.pathname.split('/').filter(Boolean);

  return (
    <nav aria-label="Breadcrumb">
      <ol className="breadcrumbs">
        <li>
          <Link to="/">Home</Link>
        </li>
        {pathnames.map((name, index) => {
          const to = `/${pathnames.slice(0, index + 1).join('/')}`;
          const isLast = index === pathnames.length - 1;
          return (
            <li key={to}>
              {isLast ? (
                <span aria-current="page">{decodeURIComponent(name)}</span>
              ) : (
                <Link to={to}>{decodeURIComponent(name)}</Link>
              )}
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

**Role-Based Navigation:**
```jsx
function Navigation({ user }) {
  const links = [
    { to: '/', label: 'Home', roles: ['user', 'admin'] },
    { to: '/admin', label: 'Admin', roles: ['admin'] },
    { to: '/settings', label: 'Settings', roles: ['user', 'admin'] },
  ];

  const visibleLinks = links.filter((link) =>
    link.roles.some((role) => user.roles.includes(role))
  );

  return (
    <nav aria-label="Main navigation">
      <ul>
        {visibleLinks.map((link) => (
          <li key={link.to}>
            <NavLink to={link.to}>{link.label}</NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

**Syntax Rules:**
- Use `<nav aria-label="...">` for navigation landmarks; label multiple navs distinctly.
- Use `aria-current="page"` on the active link (or `NavLink` handles it automatically).
- Use `aria-expanded` and `aria-controls` on the hamburger button.
- Close the mobile menu on link click.
- Use `end` on `NavLink` for exact matching (e.g., home link).
- Breadcrumbs use `<nav aria-label="Breadcrumb">` with an `<ol>`.
- Use `aria-current="page"` on the last breadcrumb item (non-link).
- Hide navigation items based on roles by filtering the array, not by CSS `display: none`.

**Constraints and Limitations:**
- Hiding links with CSS `display: none` leaves them in the DOM and accessible to screen readers; filter them from the array instead.
- Mobile menus that cover content should trap focus or use a dialog pattern.
- Nested dropdowns are complex; use a library or keep navigation flat.
- Breadcrumbs require a route hierarchy; they do not work well for flat structures.
- Always provide a skip link to the main content for keyboard users.

### Annotated Code Example: Responsive Navigation with Skip Link

```jsx
import { useState } from 'react';
import { NavLink, useLocation } from 'react-router';

const links = [
  { to: '/', label: 'Home' },
  { to: '/products', label: 'Products' },
  { to: '/pricing', label: 'Pricing' },
  { to: '/about', label: 'About' },
];

export function Navigation() {
  const [isOpen, setIsOpen] = useState(false);
  const location = useLocation();

  return (
    <>
      <a href="#main-content" className="skip-link">Skip to main content</a>

      <nav aria-label="Main navigation" className="nav">
        <button
          className="nav-toggle"
          aria-expanded={isOpen}
          aria-controls="nav-menu"
          aria-label={isOpen ? 'Close navigation menu' : 'Open navigation menu'}
          onClick={() => setIsOpen((o) => !o)}
        >
          <span aria-hidden="true">{isOpen ? '✕' : '☰'}</span>
        </button>

        <ul id="nav-menu" className={`nav-list ${isOpen ? 'is-open' : ''}`}>
          {links.map((link) => (
            <li key={link.to}>
              <NavLink
                to={link.to}
                end={link.to === '/'}
                className={({ isActive }) => `nav-link ${isActive ? 'is-active' : ''}`}
                aria-current={location.pathname === link.to ? 'page' : undefined}
                onClick={() => setIsOpen(false)}
              >
                {link.label}
              </NavLink>
            </li>
          ))}
        </ul>
      </nav>

      <main id="main-content" tabIndex={-1}>
        {/* Page content */}
      </main>
    </>
  );
}
```

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}

.nav-toggle {
  display: block;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 8px;
}

.nav-list {
  display: none;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav-list.is-open {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
}

@media (min-width: 768px) {
  .nav-toggle { display: none; }
  .nav-list {
    display: flex;
    flex-direction: row;
    gap: 24px;
  }
}

.nav-link.is-active {
  font-weight: 700;
  border-bottom: 2px solid currentColor;
}
```

**Expected Output:** On mobile, a hamburger button toggles a vertical menu. On desktop, the menu is horizontal. The active link is bold with a bottom border and `aria-current="page"`. A skip link appears on focus for keyboard users.

**Why This Output Occurs:** The `useState` controls the mobile menu's open state. The `aria-expanded` and `aria-controls` attributes link the button to the menu. `NavLink` automatically applies the active class and `aria-current="page"`. The media query switches the layout from vertical to horizontal at 768px. The skip link allows keyboard users to bypass navigation.

### Real-World Cases

- **E-commerce:** Category navigation that adapts to mobile and desktop.
- **SaaS dashboards:** Role-based navigation (admin vs. user).
- **Documentation sites:** Hierarchical navigation with breadcrumbs.
- **Marketing sites:** Responsive navigation with mobile hamburger menus.
- **Admin panels:** Multi-level navigation with permissions.

### References

- W3C WAI-ARIA Authoring Practices – Disclosure Navigation: https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/
- MDN Web Docs – `aria-current`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-current
- React Router – NavLink: https://reactrouter.com/api/components/NavLink
- Inclusive Components – Navigation: https://inclusive-components.design/
- web.dev – Building a Responsive Navigation: https://web.dev/learn/design/navigation

---

## Comparison and Decision Guidance

| Pattern | ARIA Role | Keyboard Interaction | Focus Management | State |
|---|---|---|---|---|
| **Tabs** | `tablist`/`tab`/`tabpanel` | Arrows, Home/End, Tab into panel | Roving tabindex | Active tab |
| **Accordion** | `button` + `region` | Enter/Space, Tab | Natural tab order | Open items |
| **Modal** | `dialog` + `aria-modal` | Escape, Tab (trapped) | Trap + restore | Open/closed |
| **Dropdown** | `menu`/`menuitem` | Arrows, Escape, Tab | Focus active item | Open/closed |
| **Pagination** | `nav` + `aria-current` | Tab, Enter | Natural tab order | Current page |
| **Wizard** | `list` + `aria-current="step"` | Tab, Enter | Natural tab order | Current step |
| **Expand/Collapse** | `button` + `aria-expanded` | Enter/Space | Natural tab order | Open/closed |
| **Navigation** | `nav` + `aria-current` | Tab, Enter | Skip link | Active route |

**Decision Guidance:**
- **Use the native element when possible:** `<dialog>` for modals, `<details>` for disclosures, `<select>` for simple dropdowns.
- **Use a library for complex patterns:** Radix UI, React Aria, or Headless UI for tabs, modals, dropdowns, and accordions.
- **Always implement keyboard navigation:** Every interactive pattern must be operable without a mouse.
- **Always manage focus:** Trap focus in modals and dropdowns; restore focus to the trigger on close.
- **Always provide accessible names:** `aria-label` or visible text on every interactive element.
- **Always test with a keyboard and a screen reader:** Automated tools catch only 30–50% of issues.
- **Use `aria-current="page"`** on the active navigation link; use `aria-current="step"` on the active wizard step.
- **Respect `prefers-reduced-motion`** for animations.

---

## References

- W3C WAI-ARIA Authoring Practices – Patterns: https://www.w3.org/WAI/ARIA/apg/patterns/
- W3C WAI-ARIA Authoring Practices – Tabs: https://www.w3.org/WAI/ARIA/apg/patterns/tabs/
- W3C WAI-ARIA Authoring Practices – Accordion: https://www.w3.org/WAI/ARIA/apg/patterns/accordion/
- W3C WAI-ARIA Authoring Practices – Dialog (Modal): https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- W3C WAI-ARIA Authoring Practices – Menu Button: https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/
- W3C WAI-ARIA Authoring Practices – Disclosure: https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/
- MDN Web Docs – ARIA Roles: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles
- MDN Web Docs – `<dialog>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog
- MDN Web Docs – `<details>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details
- MDN Web Docs – `aria-current`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-current
- MDN Web Docs – `aria-expanded`: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-expanded
- React Aria – Components: https://react-spectrum.adobe.com/react-aria/
- Radix UI – Primitives: https://www.radix-ui.com/primitives
- Headless UI – Components: https://headlessui.com/
- Inclusive Components – Patterns: https://inclusive-components.design/
- TanStack Query – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- React Hook Form – FormProvider: https://react-hook-form.com/docs/formprovider
- React Router – NavLink: https://reactrouter.com/api/components/NavLink
- CSS-Tricks – Animating the Accordion with CSS Grid: https://css-tricks.com/animating-the-accordion-with-css-grid/
- Smashing Magazine – Building an Accessible Tab Component: https://www.smashingmagazine.com/2020/06/accessible-tab-component/
- web.dev – Building a Responsive Navigation: https://web.dev/learn/design/navigation