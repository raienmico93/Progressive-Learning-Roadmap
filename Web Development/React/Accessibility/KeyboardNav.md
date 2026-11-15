# Keyboard Navigation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Keyboard navigation is the ability to access and operate every interactive element and function of a web application using only a keyboard, without a mouse or other pointing device.

**Technical Definition:** Keyboard navigation is the implementation of the **keyboard interface** — the set of standard keys and interaction patterns (Tab, Shift+Tab, Enter, Space, Arrow keys, Escape) that allow users to move **focus** between elements and activate them. It is governed by WCAG Success Criterion 2.1.1 (Keyboard, Level A), which requires that all functionality be operable through a keyboard interface, and is reinforced by SC 2.1.2 (No Keyboard Trap), SC 2.4.3 (Focus Order), and SC 2.4.7 (Focus Visible). Native HTML interactive elements (`<button>`, `<a href>`, `<input>`, `<select>`, `<textarea>`) provide keyboard navigation automatically; custom widgets built from `<div>`s and `<span>`s require manual implementation using `tabindex`, keyboard event handlers, and focus management.

**Beginner-Friendly Explanation:** Keyboard navigation means you can use the entire website or app with just a keyboard — Tab to move between buttons, Enter or Space to click them, and Escape to close popups. It is essential for people who cannot use a mouse, and it benefits power users who prefer the keyboard.

### Key Characteristics

- **WCAG Level A requirement:** Keyboard operability is the minimum accessibility baseline — without it, users who rely on keyboards cannot use the product at all.
- **Native-first:** Most HTML interactive elements are keyboard-accessible by default; custom widgets must replicate that behavior manually.
- **Focus-centric:** Keyboard navigation is fundamentally about **focus** — which element is currently active and how focus moves between elements.
- **Predictable and logical:** Focus order must preserve meaning and operability (WCAG 2.4.3), and users must never get trapped (WCAG 2.1.2).
- **Visible at all times:** The focused element must always have a clearly visible indicator (WCAG 2.4.7).

### Prerequisites

- Solid understanding of semantic HTML and the accessibility tree.
- Familiarity with `tabindex`, focusable elements, and browser default focus behavior.
- A keyboard and a screen reader (NVDA, JAWS, VoiceOver) for manual testing.
- Knowledge of JavaScript keyboard events for custom widgets.

### Related Programming Areas

- **Focus management** — controlling where focus goes when UI changes.
- **Semantic HTML** — the foundation of keyboard operability.
- **ARIA** — declaring roles and states for custom widgets that implement keyboard behavior.
- **Accessibility testing** — manual keyboard testing and automated audits.
- **Single-page applications (SPAs)** — focus must be managed across route changes.

### Core Concepts / Features

---

## 1. Keyboard Navigation

### Definitions

**Core Definition:** Keyboard navigation is the user's ability to move focus and operate controls using only a keyboard.

**Technical Definition:** Keyboard navigation is achieved through the browser's **sequential focus navigation** (Tab and Shift+Tab) for moving between interactive elements, and through **component-specific key bindings** (Arrow keys, Enter, Space, Escape, Home, End) for interacting within composite widgets. WCAG SC 2.1.1 requires that **all functionality** be operable through a keyboard interface, including content made available via hover, right-click, or other pointer interactions. SC 2.1.2 requires that if focus can move **to** a component, it must also be able to move **away** using only the keyboard.

**Beginner-Friendly Explanation:** Keyboard navigation is how you get around a website without a mouse. Tab moves you forward, Shift+Tab moves you back, Enter or Space activates things, and arrow keys let you move inside menus or lists.

### Purposes

- To enable use by people with motor disabilities who cannot operate a mouse.
- To support blind and low-vision users who rely on keyboards alongside screen readers.
- To satisfy WCAG SC 2.1.1 (Keyboard) and SC 2.1.2 (No Keyboard Trap) — both Level A requirements.
- To provide efficient interaction for power users and in environments without a mouse.

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
| `Home` / `End` | Move to the first or last item in a composite widget. |

**Syntax rules:**

1. **Native elements first:** `<button>`, `<a href>`, `<input>`, `<select>`, `<textarea>` are keyboard-accessible by default — use them.
2. **Custom widgets must replicate keyboard behavior:** If you build a button from a `<div>`, you must add `tabindex="0"`, `role="button"`, and `keydown` handlers for Enter and Space.
3. **Focus must never be trapped:** Users must always be able to move focus away from any component using standard keys (Tab, Shift+Tab, or documented alternatives).
4. **Focus must be visible:** Never remove the focus indicator without providing an equally visible alternative.

**Constraints and Limitations:**

- Positive `tabindex` values disrupt natural focus order and should be avoided entirely.
- Custom arrow-key navigation must not override native scrolling in document flow — only use arrow keys within composite widgets.
- Keyboard shortcuts using only character keys must be remappable or disableable (WCAG 2.1.4).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Native vs. Custom Button

```html
<!-- ✅ NATIVE: keyboard accessible by default -->
<button type="button" onclick="submit()">Submit</button>

<!-- ❌ CUSTOM: requires manual keyboard implementation -->
<div role="button" tabindex="0" onclick="submit()">Submit</div>
```

**Expected Output:** The `<button>` is focusable with Tab, activates with Enter and Space, and is announced as "Submit, button." The `<div>` with `role="button"` is focusable (via `tabindex="0"`) and announced as a button, but **Enter and Space do nothing** unless you add `keydown` handlers:

```javascript
const customButton = document.querySelector('[role="button"]');
customButton.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();
    customButton.click();
  }
});
```

**Why this matters:** Fixing the custom version requires `tabindex`, `role`, and two keyboard event handlers — three extra steps that native `<button>` provides for free. This is why the first rule of ARIA is to use native HTML.

### Real-World Cases

- **Forms:** Tab through fields, Enter to submit, Space to toggle checkboxes.
- **Navigation menus:** Tab to links, arrow keys within dropdown menus.
- **Modal dialogs:** Tab trapped inside, Escape to close, focus returned on close.

### References

- W3C — Understanding SC 2.1.1 Keyboard — https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html
- W3C — Understanding SC 2.1.2 No Keyboard Trap — https://w3c.github.io/wcag/understanding/no-keyboard-trap
- MDN — Keyboard-navigable JavaScript widgets — https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets

---

## 2. Focus Management

### Definitions

**Core Definition:** Focus management is the programmatic control of which element has keyboard focus at any given moment, ensuring that focus moves logically, remains visible, and never gets lost.

**Technical Definition:** Focus management encompasses: setting initial focus when a page or dialog loads, moving focus in response to user actions (opening a modal, deleting a list item), trapping focus inside modal dialogs, returning focus to the triggering element when a dialog closes, and ensuring that focus is never lost when the focused element is removed from the DOM. It is the practical implementation of WCAG SC 2.4.3 (Focus Order) and SC 2.4.7 (Focus Visible).

**Beginner-Friendly Explanation:** Focus management is controlling where the keyboard "cursor" is. When you open a modal, focus should move into it; when you close it, focus should go back to the button that opened it. Users should never lose track of where they are.

### Purposes

- To ensure keyboard users can always see and track their current position.
- To prevent focus from being lost when DOM elements are removed or hidden.
- To trap focus inside modal dialogs so users cannot accidentally interact with obscured background content.
- To return focus to a logical, predictable location after a dialog, menu, or task completes.

### Syntax Rules and Structure

**Focus management patterns:**

| Scenario | Pattern |
|----------|---------|
| Modal opens | Save the trigger element; move focus to the first focusable element inside the modal (or the modal itself). |
| Modal closes | Return focus to the saved trigger element. |
| List item deleted | Move focus to the next item (or previous if the last item was deleted). |
| Page/route change | Move focus to the main heading or a skip-link target. |
| Element disabled or hidden | Move focus to the nearest logical alternative — never leave focus on a removed element. |

**Syntax rules:**

1. **Always save the previously focused element** before moving focus — you will need it to restore focus later.
2. **Never leave focus on `document.body`** after closing a dialog — this forces users to Tab from the beginning of the page.
3. **Use `element.focus()`** to programmatically move focus; ensure the element is focusable (has `tabindex` if necessary).
4. **Visible focus is mandatory** — never use `outline: none` without a replacement indicator.

**Constraints and Limitations:**

- Focus changes should not happen unexpectedly during normal navigation (WCAG 3.2.1 On Focus).
- Focus must not move to elements that are `aria-hidden="true"` or visually hidden.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Focus Management in a Modal

```javascript
let previouslyFocused = null;

function openModal(modal) {
  // Save the trigger element
  previouslyFocused = document.activeElement;

  // Show the modal
  modal.hidden = false;
  modal.setAttribute('aria-hidden', 'false');

  // Move focus into the modal
  const firstFocusable = modal.querySelector(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  if (firstFocusable) {
    firstFocusable.focus();
  } else {
    modal.focus(); // Modal must have tabindex="-1"
  }
}

function closeModal(modal) {
  modal.hidden = true;
  modal.setAttribute('aria-hidden', 'true');

  // Return focus to the trigger
  if (previouslyFocused) {
    previouslyFocused.focus();
    previouslyFocused = null;
  }
}
```

**Expected Output:** Opening the modal moves focus to its first focusable element; closing it returns focus to the button that opened it. Keyboard users continue exactly where they left off.

### Real-World Cases

- **Modals and dialogs:** The most common focus management scenario.
- **Single-page applications:** Route changes must move focus to the new page's heading.
- **Dynamic lists:** After deleting an item, focus must move to the next logical item.
- **Menus and popovers:** Opening a menu moves focus into it; closing returns focus to the trigger.

### References

- W3C — Understanding SC 2.4.3 Focus Order — https://w3c.github.io/wcag/understanding/focus-order
- W3C — Developing a Keyboard Interface: Focus Management — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- TestParty — Accessible Modal Dialogs: Focus Management — https://testparty.ai/blog/modal-dialog-accessibility

---

## 3. Focus Trap

### Definitions

**Core Definition:** A focus trap is a technique that confines keyboard focus within a specific container — typically a modal dialog or popover — preventing users from tabbing into obscured background content.

**Technical Definition:** A focus trap listens for Tab and Shift+Tab events within a container, intercepts them, and manually cycles focus between the first and last focusable elements inside the container. When the user Tabs from the last element, focus wraps to the first; when they Shift+Tab from the first, focus wraps to the last. Focus trapping is appropriate **only** for modal dialogs and similar overlays that demand the user's full attention; WCAG SC 2.1.2 (No Keyboard Trap) requires that users can always exit the trap, typically via Escape or a close button.

**Beginner-Friendly Explanation:** A focus trap keeps the keyboard focus inside a modal. If you open a dialog, Tab should cycle through the dialog's buttons and fields — not jump back to the page behind it. But users must always be able to close the dialog and escape the trap.

### Purposes

- To prevent keyboard users from interacting with background content that is visually obscured by a modal.
- To ensure screen reader users stay within the dialog context.
- To satisfy WCAG SC 2.4.3 (Focus Order) by maintaining a logical focus scope.
- To create a predictable, contained interaction experience for modal workflows.

### Syntax Rules and Structure

```javascript
function trapFocus(container) {
  const focusable = container.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusable[0];
  const last = focusable[focusable.length - 1];

  container.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;

    if (e.shiftKey) {
      // Shift+Tab from first element → wrap to last
      if (document.activeElement === first) {
        e.preventDefault();
        last.focus();
      }
    } else {
      // Tab from last element → wrap to first
      if (document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
  });
}
```

**Syntax rules:**

1. **Identify all focusable elements** inside the container using a selector that includes `button`, `[href]`, `input`, `select`, `textarea`, and `[tabindex]:not([tabindex="-1"])`.
2. **Intercept Tab and Shift+Tab** at the container level via `keydown`.
3. **Wrap focus** from last to first (Tab) and first to last (Shift+Tab).
4. **Always provide an exit:** Escape key must close the dialog, or a visible close button must be reachable via the trap.
5. **Do not trap focus in non-modal components** — dropdowns, tooltips, and non-modal panels should not trap focus.

**Constraints and Limitations:**

- The trap must not prevent the user from closing the dialog — WCAG 2.1.2 requires a standard exit method.
- Dynamically rendered focusable elements require the focusable list to be recalculated on each Tab event (or use a MutationObserver).
- Native `<dialog>` with `showModal()` provides built-in focus trapping — prefer it over manual implementation when possible.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Manual Focus Trap for a Modal

```javascript
const modal = document.getElementById('modal');
const closeBtn = modal.querySelector('.close-btn');

function openModal() {
  modal.hidden = false;
  modal.setAttribute('aria-hidden', 'false');
  trapFocus(modal);
  closeBtn.focus();
}

modal.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    closeModal(modal);
  }
});
```

**Expected Output:** While the modal is open, Tab and Shift+Tab cycle only through the modal's focusable elements. Escape closes the modal and releases the trap.

### Real-World Cases

- **Modal dialogs:** Login, confirmation, and alert dialogs.
- **Full-screen overlays:** Image lightboxes and video players.
- **Drawers and side panels:** Off-canvas navigation that demands focus.

### References

- W3C — Understanding SC 2.1.2 No Keyboard Trap — https://w3c.github.io/wcag/understanding/no-keyboard-trap
- TestParty — Accessible Modal Dialogs: Focus Trapping — https://testparty.ai/blog/modal-dialog-accessibility
- CoreUI — React Focus Trap Component — https://coreui.io/

---

## 4. Focus Restoration

### Definitions

**Core Definition:** Focus restoration is the practice of returning keyboard focus to the element that triggered a UI change (such as opening a dialog) when that change is undone (such as closing the dialog).

**Technical Definition:** Focus restoration requires saving a reference to `document.activeElement` before moving focus into a new context (modal, menu, route), and calling `focus()` on that saved element when the context is closed or dismissed. Without restoration, focus typically falls back to `document.body`, forcing keyboard users to re-navigate from the top of the page. WCAG SC 2.4.3 (Focus Order) and SC 2.1.2 (No Keyboard Trap) together imply that focus should return to a logical, predictable location after a contained interaction ends.

**Beginner-Friendly Explanation:** Focus restoration means putting the keyboard cursor back where it was before a dialog opened. If you clicked "Delete" on a row and a confirmation dialog appeared, closing that dialog should put focus back on the "Delete" button — not at the top of the page.

### Purposes

- To prevent keyboard users from losing their place after closing a dialog or menu.
- To reduce the number of keystrokes required to continue a task.
- To satisfy the predictability requirements of WCAG SC 3.2.1 and SC 2.4.3.
- To create a seamless, non-disorienting user experience.

### Syntax Rules and Structure

```javascript
let triggerElement = null;

function openDialog() {
  triggerElement = document.activeElement; // Save the trigger
  dialog.showModal();
}

function closeDialog() {
  dialog.close();
  if (triggerElement) {
    triggerElement.focus(); // Restore focus
  }
}
```

**Syntax rules:**

1. **Save `document.activeElement` before opening** the dialog, menu, or overlay.
2. **Restore focus after closing** by calling `.focus()` on the saved element.
3. **Restore focus only if the trigger still exists** in the DOM — if the trigger was removed, move focus to a logical alternative.
4. **Do not restore focus to `document.body`** unless no other logical target exists.

**Constraints and Limitations:**

- If the trigger element is disabled or removed when the dialog closes, focus restoration must target the nearest logical alternative.
- Focus restoration should happen **after** the dialog is fully closed and hidden from assistive technology.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Focus Restoration with a Delete Confirmation

```javascript
function confirmDelete(deleteButton) {
  const previouslyFocused = document.activeElement;
  const confirmed = window.confirm('Delete this item?');

  if (confirmed) {
    deleteButton.closest('li').remove();
    // Move focus to the next item or a logical fallback
    const nextItem = document.querySelector('li button');
    if (nextItem) nextItem.focus();
  } else {
    // Restore focus to the delete button
    previouslyFocused.focus();
  }
}
```

**Expected Output:** If the user cancels, focus returns to the "Delete" button. If they confirm and the item is removed, focus moves to the next item — never lost.

### Real-World Cases

- **Confirmation dialogs:** After "Cancel" or "OK," focus returns to the triggering control.
- **Menus and dropdowns:** After selecting an item, focus returns to the menu trigger.
- **Route changes:** After navigating back, focus may return to the link that initiated the navigation.

### References

- W3C — Understanding SC 2.4.3 Focus Order — https://w3c.github.io/wcag/understanding/focus-order
- Mantine — use-focus-return — https://v7.mantine.dev/use-focus-return
- SvelteUI — use-focus-return — https://github.com/svelteuidev/svelteui

---

## 5. Tab Order

### Definitions

**Core Definition:** Tab order is the sequence in which focusable elements receive focus when the user presses the Tab key (and Shift+Tab for reverse order).

**Technical Definition:** The default tab order in HTML is the **source order** — the order in which elements appear in the DOM. Positive `tabindex` values can override this, placing elements before the natural order, but this is strongly discouraged because it requires maintaining `tabindex` values for every focusable element and often creates a focus order that contradicts the visual layout. WCAG SC 2.4.3 (Focus Order) requires that the tab order preserve meaning and operability — it must be logical and intuitive for keyboard users.

**Beginner-Friendly Explanation:** Tab order is the path your keyboard takes when you press Tab. By default, it follows the order of the HTML — the first element in the code gets focus first. You should never mess with this order unless absolutely necessary.

### Purposes

- To ensure keyboard users encounter content in a logical, meaningful sequence.
- To maintain consistency between the visual layout and the focus order.
- To satisfy WCAG SC 2.4.3 (Focus Order).
- To avoid confusion and disorientation caused by unpredictable focus jumps.

### Syntax Rules and Structure

**Tab order rules:**

| Scenario | Correct behavior |
|----------|------------------|
| Default | Follow DOM source order — do not override with positive `tabindex`. |
| Reverse | Shift+Tab moves backward through the same order. |
| Composite widgets | Only **one** element in the widget is in the Tab order; Arrow keys move within the widget (roving tabindex). |
| Custom components | Use `tabindex="0"` to add an element to the natural order; `tabindex="-1"` to make it programmatically focusable but not Tab-reachable. |

**Constraints and Limitations:**

- **Never use positive `tabindex` values** (`1`, `2`, `3`…) — they override the natural order and create maintenance nightmares.
- Tab order does not have to match the visual layout exactly, but it must be logical and preserve meaning.
- Hidden or disabled elements are removed from the tab order automatically.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Correct Use of `tabindex` for a Custom Widget

```html
<!-- ✅ CORRECT: tabindex="0" adds the element to the natural tab order -->
<div role="button" tabindex="0" aria-pressed="false">
  Toggle
</div>

<!-- ✅ CORRECT: tabindex="-1" makes an element programmatically focusable
     but NOT reachable via Tab (used for modals, headings, error summaries) -->
<h2 id="dialog-title" tabindex="-1">Dialog Title</h2>
```

**Expected Output:** The custom button is reachable via Tab in its natural DOM position. The dialog title is focusable only via script (`element.focus()`), not via Tab.

#### Example 2: Roving Tabindex for a Composite Widget

```javascript
// Roving tabindex: only one item in the tab order at a time
function updateRovingTabindex(items, activeIndex) {
  items.forEach((item, i) => {
    item.tabIndex = i === activeIndex ? 0 : -1;
  });
  items[activeIndex].focus();
}
```

**Expected Output:** Pressing Tab moves focus into the widget once; Arrow keys move between items, updating which item has `tabindex="0"`.

### Real-World Cases

- **Menus and toolbars:** Roving tabindex keeps the entire menu as one Tab stop.
- **Tabs:** Only the selected tab is in the Tab order; arrow keys navigate between tabs.
- **Grids and trees:** One cell or node is Tab-reachable; arrow keys navigate within.

### References

- W3C — Understanding SC 2.4.3 Focus Order — https://w3c.github.io/wcag/understanding/focus-order
- W3C — WAI-ARIA Authoring Practices: Roving tabindex — https://www.w3.org/TR/2019/WD-wai-aria-practices-1.2-20191218/#kbd_roving_tabindex
- MDN — Keyboard-navigable JavaScript widgets — https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets

---

## 6. Tab Index

### Definitions

**Core Definition:** `tabindex` is a global HTML attribute that controls whether an element is focusable and, if so, where it appears in the tab order.

**Technical Definition:** The `tabindex` attribute accepts an integer value. **Absent:** The element follows the platform convention (interactive elements are focusable; non-interactive elements are not). **`tabindex="0"`:** The element becomes focusable via keyboard and script, and is placed in the tab order according to its DOM position. **`tabindex="-1"`:** The element becomes focusable via script (`element.focus()`) but is **not** included in the tab order. **`tabindex="1"` (positive):** The element is placed **before** all elements with `tabindex="0"` — this is strongly discouraged because it disrupts natural focus order and requires maintaining values for all focusable elements.

**Beginner-Friendly Explanation:** `tabindex` tells the browser whether an element can be focused with the keyboard. `tabindex="0"` means "include this in the normal Tab order." `tabindex="-1"` means "focusable by JavaScript, but not by Tab." Positive values (1, 2, 3…) are almost always a bad idea.

### Purposes

- To make non-interactive elements focusable when building custom widgets.
- To remove an element from the tab order while keeping it programmatically focusable (`tabindex="-1"`).
- To implement roving tabindex in composite widgets.
- To provide a focus target for dialogs, headings, or error summaries.

### Syntax Rules and Structure

| `tabindex` value | Focusable by mouse/script? | In Tab order? | When to use |
|------------------|---------------------------|---------------|-------------|
| **Absent** | Platform-dependent | Platform-dependent | Native interactive elements (default). |
| **`0`** | Yes | Yes (DOM position) | Custom interactive widgets. |
| **`-1`** | Yes | No | Modals, headings, error summaries, roving tabindex items. |
| **Positive (`1+`)** | Yes | Yes, before `0` | **Avoid** — breaks natural order. |

**Syntax rules:**

1. **Use `tabindex="0"`** to add a custom widget to the natural tab order.
2. **Use `tabindex="-1"`** for elements that should receive focus programmatically but not via Tab (e.g., a dialog container or a heading that receives focus on route change).
3. **Never use positive `tabindex` values** — they override source order and create fragile, hard-to-maintain focus sequences.
4. **Native interactive elements do not need `tabindex`** — `<button>`, `<a href>`, `<input>`, `<select>`, `<textarea>` are already focusable.

**Constraints and Limitations:**

- `tabindex="0"` on a non-interactive element (e.g., a `<div>`) does **not** make it operable with Enter/Space — you must add keyboard handlers.
- `tabindex="-1"` elements are still focusable by script — do not use it to "hide" elements from focus entirely.
- Positive `tabindex` values are a common accessibility anti-pattern — WCAG 2.4.3 is often violated when they are used carelessly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: `tabindex="-1"` for Dialog Focus

```html
<div role="dialog" aria-modal="true" tabindex="-1" id="dialog">
  <h2>Confirm deletion</h2>
  <button>Cancel</button>
  <button>Delete</button>
</div>
```

```javascript
document.getElementById('dialog').focus(); // Focus the dialog container
```

**Expected Output:** The dialog container receives focus, and screen readers announce the dialog role. The `tabindex="-1"` ensures the dialog is not Tab-reachable from the page — focus is moved into it programmatically.

### Real-World Cases

- **Custom dropdowns:** The trigger has `tabindex="0"` (or is a native button); menu items use roving tabindex.
- **Modals:** The dialog container has `tabindex="-1"` to receive initial focus.
- **Skip links:** The main content target may have `tabindex="-1"` to receive focus when the skip link is activated.

### References

- MDN — tabindex — https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex
- MDN — Keyboard-navigable JavaScript widgets: Using tabindex — https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets
- W3C — WAI-ARIA Authoring Practices: Roving tabindex — https://www.w3.org/TR/2019/WD-wai-aria-practices-1.2-20191218/#kbd_roving_tabindex

---

## 7. Keyboard Shortcuts

### Definitions

**Core Definition:** Keyboard shortcuts are key combinations (or single keys) that trigger specific application functions without navigating the UI.

**Technical Definition:** Keyboard shortcuts are single-key or multi-key combinations (e.g., `Ctrl+S`, `/`, `g` then `i`) that activate functionality. WCAG SC 2.1.4 (Character Key Shortcuts, Level A) requires that any shortcut implemented using **only character keys** (letters, numbers, punctuation, symbols) must be **turnable off**, **remappable** to include a modifier key (Ctrl, Alt), or **active only when the relevant component has focus**. This prevents accidental activation by speech input users (whose dictation is interpreted as character sequences) and by keyboard users prone to mistyping.

**Beginner-Friendly Explanation:** Keyboard shortcuts are things like Ctrl+S to save or "/" to search. If you use a shortcut that is just a single letter like "d" for delete, you must let users turn it off or change it — otherwise, someone using voice dictation might accidentally trigger it by saying the letter.

### Purposes

- To provide efficient interaction for power users.
- To enable access to functions that might otherwise require many navigation steps.
- To support users who cannot use a mouse for fine-grained interaction.
- To satisfy WCAG SC 2.1.4 (Character Key Shortcuts) when single-character shortcuts are used.

### Syntax Rules and Structure

**WCAG SC 2.1.4 compliance options:**

| Option | Implementation |
|--------|----------------|
| **Turn off** | Provide a setting to disable all single-character shortcuts. |
| **Remap** | Allow users to add a modifier key (Ctrl, Alt) to the shortcut. |
| **Active only on focus** | The shortcut only works when the relevant component has focus. |

**Syntax rules:**

1. **Prefer modifier-based shortcuts** (`Ctrl+S`, `Alt+N`) over single-character shortcuts — they are less prone to accidental activation.
2. **Document all shortcuts** — WCAG 3.0 requires custom keyboard commands to be documented and programmatically available.
3. **Do not override browser or assistive technology shortcuts** — avoid conflicts with `Ctrl+` combinations that browsers and screen readers use.
4. **Provide a help dialog or documentation** listing available shortcuts.

**Constraints and Limitations:**

- Single-character shortcuts are the most problematic — they conflict with speech input and rapid typing.
- Shortcuts must not interfere with standard keyboard navigation (Tab, Arrow keys, Enter, Space, Escape).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Compliant Character Key Shortcut

```javascript
// Single-key shortcut "d" for delete, but only active when the list has focus
const list = document.getElementById('item-list');

list.addEventListener('keydown', (e) => {
  if (e.key === 'd' && !e.ctrlKey && !e.metaKey && !e.altKey) {
    if (document.activeElement.closest('#item-list')) {
      deleteSelectedItem();
    }
  }
});
```

**Expected Output:** Pressing "d" only triggers deletion when focus is inside the list. If the user is typing in a search box elsewhere on the page, "d" types the letter normally. This satisfies the "active only on focus" option of SC 2.1.4.

### Real-World Cases

- **Webmail:** `c` to compose, `r` to reply — must be disableable or remappable.
- **Code editors:** `Ctrl+S` to save, `Ctrl+F` to find — modifier-based, low risk.
- **Project management tools:** `g` then `p` to go to projects — sequence shortcuts that require a dedicated mode or documentation.

### References

- W3C — Understanding SC 2.1.4 Character Key Shortcuts — https://www.w3.org/WAI/WCAG21/Understanding/character-key-shortcuts.html
- W3C — Technique G217: Providing a mechanism to remap or turn off character key shortcuts — https://www.w3.org/WAI/WCAG21/Techniques/general/G217
- W3C — WCAG 3.0: Custom keys documented — https://www.w3.org/TR/2026/WD-wcag-3.0-20260303/

---

## 8. Keyboard Events

### Definitions

**Core Definition:** Keyboard events are JavaScript events fired by the browser when the user interacts with the keyboard — `keydown`, `keypress`, and `keyup`.

**Technical Definition:** The browser fires three primary keyboard events: **`keydown`** when a key is pressed down (fires repeatedly if held), **`keypress`** when a key that produces a character is pressed (deprecated in modern browsers), and **`keyup`** when the key is released. The `event.key` property provides the **semantic key value** (e.g., `"Enter"`, `"Escape"`, `"a"`, `"ArrowLeft"`), while `event.code` provides the **physical key location** (e.g., `"KeyA"`, `"Enter"`). For accessibility, `keydown` is generally preferred for handling navigation and activation because it fires before the default action can be prevented.

**Beginner-Friendly Explanation:** Keyboard events let your JavaScript know when a key is pressed or released. `keydown` fires when you press down, `keyup` fires when you let go. Use `event.key` to know which key was pressed — like `"Enter"` or `"Escape"`.

### Purposes

- To implement custom keyboard behavior for widgets (arrow-key navigation, Enter/Space activation).
- To handle keyboard shortcuts.
- To detect Escape key presses for closing dialogs.
- To manage focus movement within composite widgets.

### Syntax Rules and Structure

```javascript
element.addEventListener('keydown', (event) => {
  if (event.key === 'Escape') {
    closeDialog();
  }
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault();
    activateItem();
  }
});
```

**Key properties:**

| Property | Description | Example values |
|----------|-------------|----------------|
| `event.key` | Semantic key value | `"Enter"`, `"Escape"`, `"a"`, `"ArrowDown"` |
| `event.code` | Physical key location | `"KeyA"`, `"Enter"`, `"ArrowUp"` |
| `event.repeat` | Whether the key is held down | `true` / `false` |

**Syntax rules:**

1. **Prefer `event.key`** for semantic behavior — `"Enter"` and `"Escape"` are more meaningful than key codes.
2. **Use `keydown` for activation** (Enter, Space) and **`keyup` for release-based interactions**.
3. **Call `event.preventDefault()`** when you handle a key to prevent the browser's default action (e.g., Space scrolling the page when activating a custom button).
4. **Check modifier keys** (`event.ctrlKey`, `event.metaKey`, `event.altKey`, `event.shiftKey`) for shortcuts.

**Constraints and Limitations:**

- `keypress` is deprecated and should not be used in new code.
- `event.key` values for punctuation and symbols vary by keyboard layout — use `event.code` for physical key detection.
- Holding a key fires `keydown` repeatedly — use `event.repeat` to ignore repeats if necessary.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Custom Button with Keyboard Events

```javascript
const customButton = document.querySelector('[role="button"]');

customButton.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault(); // Prevent Space from scrolling
    customButton.click();
  }
});
```

**Expected Output:** Pressing Enter or Space on the custom button triggers its click handler, matching native `<button>` behavior.

### Real-World Cases

- **Modal dialogs:** `keydown` for Escape to close.
- **Composite widgets:** `keydown` for Arrow keys, Home, End to navigate items.
- **Form controls:** `keydown` for Enter to submit and Escape to cancel.

### References

- W3C — Technique SCR20: Using both keyboard and other device-specific functions — https://www.w3.org/WAI/WCAG21/Techniques/client-side-script/SCR20
- MDN — KeyboardEvent — https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent

---

## 9. Keyboard Navigation Best Practices

### Definitions

**Core Definition:** Keyboard navigation best practices are the guidelines that ensure all users can operate the interface efficiently, predictably, and without frustration using only a keyboard.

**Technical Definition:** Best practices encompass: **native-first development** (use `<button>`, `<a>`, `<input>` before custom ARIA widgets), **logical focus order** (never override with positive `tabindex`), **visible focus indicators** (never remove without a replacement), **no keyboard traps** (except intentional modal traps with a clear exit), **efficient navigation** (skip links, landmarks, roving tabindex for composite widgets), **standard key conventions** (Tab, Shift+Tab, Enter, Space, Escape, Arrow keys), and **character key shortcut compliance** (turn off/remap, or focus-scoped).

**Beginner-Friendly Explanation:** Best practices are the habits that make keyboard navigation work well — use real buttons, keep focus order natural, always show focus, never trap users without an exit, and test with just a keyboard.

### Purposes

- To ensure WCAG 2.1.1 (Keyboard) and 2.1.2 (No Keyboard Trap) conformance.
- To provide efficient, predictable navigation for keyboard-only users.
- To prevent the most common keyboard accessibility failures.
- To create an interface that works equally well for keyboard and mouse users.

### Syntax Rules and Structure

**Best practices checklist:**

| Practice | Description |
|----------|-------------|
| **Native-first** | Use `<button>`, `<a href>`, `<input>`, `<select>`, `<textarea>` — they are keyboard-accessible by default. |
| **No positive `tabindex`** | Use `tabindex="0"` and `tabindex="-1"` only; never `1`, `2`, `3`… |
| **Visible focus** | Never remove `:focus` styles; ensure the indicator has sufficient contrast. |
| **No keyboard traps** | Users must always be able to Tab away from any component (or Escape from a modal). |
| **Logical order** | Tab order must preserve meaning and operability (WCAG 2.4.3). |
| **Roving tabindex** | Composite widgets (menus, tabs, grids) use one Tab stop and Arrow keys within. |
| **Standard keys** | Tab, Shift+Tab, Enter, Space, Escape, Arrow keys — do not redefine. |
| **Character key shortcuts** | Turn off, remap, or scope to focus (WCAG 2.1.4). |
| **Test with keyboard** | Unplug the mouse and navigate the entire app. |

**Constraints and Limitations:**

- Accessibility is not fully testable by automation — manual keyboard testing is essential.
- Arrow-key overrides must not break native scrolling in document flow (WCAG 1.4.10).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Best-Practice-Compliant Keyboard Interface

```html
<!-- Skip link for efficient navigation -->
<a href="#main-content" class="skip-link">Skip to main content</a>

<!-- Native button — fully keyboard accessible -->
<button type="button" onclick="submitForm()">Submit</button>

<!-- Modal with focus trap and restoration -->
<dialog id="my-dialog">
  <h2>Confirm</h2>
  <button onclick="closeDialog()">Cancel</button>
  <button onclick="confirm()">Confirm</button>
</dialog>

<!-- Composite widget with roving tabindex -->
<div role="tablist" aria-label="Settings">
  <button role="tab" tabindex="0" aria-selected="true">Profile</button>
  <button role="tab" tabindex="-1" aria-selected="false">Security</button>
</div>
```

**Expected Output:** Keyboard users can skip to main content, Tab to the submit button and activate it, open the modal (focus is trapped and restored on close), and use Arrow keys to navigate tabs — all without a mouse.

### Real-World Cases

- **Design systems:** Every component is tested with keyboard navigation before release.
- **Government services:** Keyboard accessibility is legally required and audited.
- **Enterprise applications:** Power users rely on keyboard efficiency for data entry and navigation.

### References

- Microsoft Learn — Test keyboard accessibility — https://learn.microsoft.com/en-us/training/modules/develop-keyboard-accessible-products/7-test-keyboard-accessibility
- W3C — Developing a Keyboard Interface — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- W3C — Keyboard Compatibility — https://www.w3.org/WAI/perspective-videos/keyboard/
- Chrome for Developers — Spatial Navigation (Directional Keypad Focus) — https://developer.chrome.com/docs/modern-web-guidance/guides/ui-behaviors/spatial-navigation

---

## References

- W3C — Understanding SC 2.1.1 Keyboard — https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html
- W3C — Understanding SC 2.1.2 No Keyboard Trap — https://w3c.github.io/wcag/understanding/no-keyboard-trap
- W3C — Understanding SC 2.4.3 Focus Order — https://w3c.github.io/wcag/understanding/focus-order
- W3C — Understanding SC 2.4.7 Focus Visible — https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html
- W3C — Understanding SC 2.1.4 Character Key Shortcuts — https://www.w3.org/WAI/WCAG21/Understanding/character-key-shortcuts.html
- W3C — Technique G217: Providing a mechanism to remap or turn off character key shortcuts — https://www.w3.org/WAI/WCAG21/Techniques/general/G217
- W3C — Technique SCR20: Using both keyboard and other device-specific functions — https://www.w3.org/WAI/WCAG21/Techniques/client-side-script/SCR20
- W3C — WAI-ARIA Authoring Practices: Roving tabindex — https://www.w3.org/TR/2019/WD-wai-aria-practices-1.2-20191218/#kbd_roving_tabindex
- W3C — Developing a Keyboard Interface — https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- W3C — Keyboard Compatibility — https://www.w3.org/WAI/perspective-videos/keyboard/
- W3C — WCAG 3.0 Working Draft — https://www.w3.org/TR/2026/WD-wcag-3.0-20260303/
- MDN — Keyboard-navigable JavaScript widgets — https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets
- MDN — tabindex — https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex
- MDN — KeyboardEvent — https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent
- TestParty — Accessible Modal Dialogs: Focus Trapping and Screen Reader Support — https://testparty.ai/blog/modal-dialog-accessibility
- Microsoft Learn — Test keyboard accessibility — https://learn.microsoft.com/en-us/training/modules/develop-keyboard-accessible-products/7-test-keyboard-accessibility
- Microsoft Learn — Keyboard navigation fundamentals — https://learn.microsoft.com/en-us/training/modules/develop-keyboard-accessible-products/2-keyboard-navigation-fundamentals
- Chrome for Developers — Spatial Navigation (Directional Keypad Focus) — https://developer.chrome.com/docs/modern-web-guidance/guides/ui-behaviors/spatial-navigation
- Mantine — use-focus-return — https://v7.mantine.dev/use-focus-return
- SvelteUI — use-focus-return — https://github.com/svelteuidev/svelteui
- CoreUI — React Focus Trap Component — https://coreui.io/