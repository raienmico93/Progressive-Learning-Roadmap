# Dynamic UI Construction with jQuery — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Dynamic UI Construction with jQuery is the practice of building interactive, stateful user interface components — navigation menus, tabs, accordions, modals, dropdowns, tooltips, and carousels — using jQuery's DOM manipulation, event handling, and animation capabilities. Each component manages its own state, responds to user interaction, and updates the DOM to reflect visual changes.

**Technical Definition:** Dynamic UI Construction encompasses the architectural patterns and implementation techniques for creating reusable, accessible, and stateful UI widgets with jQuery. These patterns include: event delegation and namespacing for efficient listener management; class-based state toggling for visual updates; ARIA attribute synchronization for accessibility; focus management for keyboard navigation; timer-based auto-play for carousels; and position recalculation for viewport-aware tooltips. The jQuery UI Widget Factory provides a standardized framework for stateful widgets, while custom implementations can follow the same conventions manually.

**Beginner-Friendly Explanation:** Dynamic UI construction is about making web pages interactive. Instead of static HTML that just sits there, you build components that respond to clicks, hovers, and keyboard input — menus that open and close, tabs that switch content, modals that pop up, and carousels that slide through images. jQuery gives you the tools to build these components without writing hundreds of lines of vanilla JavaScript.

### Key Characteristics

- **Stateful behavior:** Each component maintains internal state (open/closed, active/inactive, current slide) that determines its visual appearance and behavior.
- **Event-driven architecture:** Components respond to user events (click, hover, keyboard, touch) and update the DOM accordingly.
- **Accessibility integration:** Components update ARIA attributes (`aria-expanded`, `aria-hidden`, `aria-selected`) to communicate state to assistive technologies.
- **Focus management:** Keyboard users can navigate components via Tab, arrow keys, Escape, and Enter, with focus moved and restored appropriately.
- **Progressive enhancement:** Components enhance semantic HTML rather than replacing it, ensuring basic functionality without JavaScript.

### Prerequisites

- Proficiency in jQuery fundamentals: selectors, events, `.each()`, `.data()`, and animation methods.
- Understanding of the CSS box model, positioning, and `overflow` properties.
- Familiarity with HTML semantics and WAI-ARIA authoring practices.
- Knowledge of the jQuery UI Widget Factory (for standardized widget development).

### Related Programming Areas

- **Accessible Rich Internet Applications (ARIA):** Communicating component state to screen readers.
- **Event-Driven Programming:** Handling user interactions through event binding and delegation.
- **CSS Transitions and Animations:** Providing visual feedback for state changes.
- **Touch and Gesture Programming:** Mapping swipe events for mobile carousels.
- **Timer-Based Programming:** Managing auto-play intervals and cleanup.

### Core Concepts / Features

This cheat sheet covers seven UI components plus two enhanced topics: ARIA attribute updates and focus management.

---

## Core Concept 1: Navigation Menus

### Definitions

**Core Definition:** A navigation menu is a UI component that presents a hierarchical list of links, supporting multi-level submenus that open on hover or click, with keyboard navigation and focus management for accessibility.

**Technical Definition:** A navigation menu is a tree-structured widget typically implemented as nested `<ul>` elements with `role="menubar"`, `role="menu"`, and `role="menuitem"` ARIA roles. Multi-level menus open submenus on hover (desktop) or click/tap (mobile and keyboard). Keyboard interaction follows the WAI-ARIA Menu or Menubar design pattern: arrow keys move focus between items, Enter or Space opens a submenu, Escape closes the submenu, and Tab moves focus out of the menu. A common accessibility challenge is the "keyboard trap," where focus becomes stuck inside the menu because the menu does not properly release focus when the user tabs away.

**Beginner-Friendly Explanation:** A navigation menu is like a table of contents for a website. Some menus have submenus — click or hover over an item and a list of related links appears. For keyboard users, you need to make sure they can navigate the menu with arrow keys and Tab, and that they can escape the menu without getting stuck.

### Purposes

- To provide a structured, hierarchical way for users to navigate a website.
- To support multi-level submenus that reveal additional navigation options.
- To ensure keyboard users can navigate the menu using standard keys (arrows, Enter, Escape, Tab).
- To communicate menu state (expanded/collapsed) to assistive technologies via ARIA attributes.
- To prevent keyboard traps that prevent users from leaving the menu.

### Syntax Rules and Structure

**Complete General Syntax (HTML Structure):**
```html
<nav role="navigation">
    <ul role="menubar">
        <li role="none">
            <a role="menuitem" href="#" aria-haspopup="true" aria-expanded="false">Products</a>
            <ul role="menu" aria-hidden="true">
                <li role="none"><a role="menuitem" href="#">Software</a></li>
                <li role="none"><a role="menuitem" href="#">Hardware</a></li>
            </ul>
        </li>
    </ul>
</nav>
```

**Complete General Syntax (jQuery Initialization):**
```javascript
$("[role=menubar] > li").hover(
    function() { $(this).find("ul").attr("aria-hidden", "false").slideDown(); },
    function() { $(this).find("ul").attr("aria-hidden", "true").slideUp(); }
);
```

| Component | Description |
|-----------|-------------|
| `role="menubar"` | The container for the menu items. |
| `role="menuitem"` | Each clickable menu item. |
| `aria-haspopup="true"` | Indicates the item has a submenu. |
| `aria-expanded` | Toggles between `"true"` and `"false"` as the submenu opens/closes. |
| `aria-hidden` | Toggles on the submenu `<ul>` to hide it from screen readers when collapsed. |

**Syntax Rules:**

- Use semantic `<ul>`/`<li>` structure with ARIA roles to define the menu hierarchy.
- Toggle `aria-expanded` on the trigger and `aria-hidden` on the submenu container during open/close transitions.
- Use `.hover()` for desktop pointer interaction and `.on("click")` for touch and keyboard interaction.
- Bind a `keydown` handler to support Escape (close submenu) and arrow keys (navigate items).
- To prevent keyboard traps, ensure that pressing Tab moves focus out of the menu naturally.

**Constraints and Limitations:**

- Hover-only menus are inaccessible to keyboard and touch users; always provide a click/keyboard alternative.
- Complex multi-level menus can trap focus if not carefully managed; test with a keyboard regularly.
- `aria-haspopup="true"` should only be set on items that actually have submenus.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Multi-Level Menu with Keyboard Support**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Navigation Menu Demo</title>
  <style>
    nav ul { list-style: none; margin: 0; padding: 0; }
    nav > ul > li { display: inline-block; position: relative; }
    nav a { display: block; padding: 10px 20px; text-decoration: none; color: #333; }
    nav ul ul { display: none; position: absolute; top: 100%; left: 0; background: #fff; border: 1px solid #ccc; min-width: 150px; }
    nav ul ul li { display: block; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <nav role="navigation">
    <ul role="menubar">
      <li role="none">
        <a role="menuitem" href="#" aria-haspopup="true" aria-expanded="false">Products</a>
        <ul role="menu" aria-hidden="true">
          <li role="none"><a role="menuitem" href="#">Software</a></li>
          <li role="none"><a role="menuitem" href="#">Hardware</a></li>
        </ul>
      </li>
      <li role="none"><a role="menuitem" href="#">About</a></li>
    </ul>
  </nav>

  <script>
    // Step 1: Handle hover for pointer devices
    $("[role=menubar] > li").hover(
      function() {
        $(this).find("> ul")
          .attr("aria-hidden", "false")
          .stop(true, true).slideDown(200);
        $(this).find("> a").attr("aria-expanded", "true");
      },
      function() {
        $(this).find("> ul")
          .attr("aria-hidden", "true")
          .stop(true, true).slideUp(200);
        $(this).find("> a").attr("aria-expanded", "false");
      }
    );

    // Step 2: Handle click/tap for touch devices
    $("[role=menubar] > li > a[aria-haspopup=true]").on("click", function(e) {
      e.preventDefault();
      var $submenu = $(this).next("ul");
      var isOpen = $submenu.attr("aria-hidden") === "false";
      $submenu.attr("aria-hidden", isOpen ? "true" : "false")
              .stop(true, true).slideToggle(200);
      $(this).attr("aria-expanded", isOpen ? "false" : "true");
    });

    // Step 3: Handle Escape key to close submenu and return focus
    $("[role=menubar] > li").on("keydown", function(e) {
      if (e.key === "Escape") {
        var $submenu = $(this).find("> ul");
        if ($submenu.attr("aria-hidden") === "false") {
          $submenu.attr("aria-hidden", "true").slideUp(200);
          $(this).find("> a").attr("aria-expanded", "false").focus();
        }
      }
    });
  </script>
</body>
</html>
```

**Expected Output:** Hovering over "Products" reveals the submenu with "Software" and "Hardware." Clicking "Products" on a touch device toggles the submenu. Pressing Escape while a submenu is open closes it and returns focus to the trigger link.

**Why this output:** The hover handler manages pointer interaction, the click handler manages touch interaction, and the keydown handler provides Escape-key dismissal. All three paths update `aria-expanded` and `aria-hidden` to keep assistive technology informed.

### Real-World Cases

- **E-commerce sites:** Multi-level menus for product categories (Electronics > Laptops > Gaming Laptops).
- **Enterprise portals:** Hierarchical navigation for departments, resources, and tools.
- **Government websites:** Accessible navigation menus that comply with Section 508 and WCAG 2.2.

---

## Core Concept 2: Tabs

### Definitions

**Core Definition:** A tabs component is a UI pattern that presents multiple content panels in a single space, with tab triggers that switch the visible panel when clicked or activated via keyboard.

**Technical Definition:** A tabs widget follows the WAI-ARIA Tabs design pattern. The tab list has `role="tablist"`, each tab trigger has `role="tab"` with `aria-selected`, `aria-controls`, and `tabindex` attributes, and each content panel has `role="tabpanel"` with `aria-labelledby` and `aria-hidden`. Only one tab is selected at a time; activating a tab updates `aria-selected` on the trigger and `aria-hidden` on the panels. Keyboard interaction uses arrow keys to move between tabs and Enter/Space to activate. jQuery UI's Tabs widget provides this pattern out of the box, with options for remote content loading and event callbacks.

**Beginner-Friendly Explanation:** Tabs are like a filing cabinet with labeled dividers. Clicking a tab brings that section of content to the front while hiding the others. Only one panel is visible at a time. For keyboard users, arrow keys move between tabs and Enter activates the selected tab.

### Purposes

- To organize content into discrete, mutually exclusive panels that share the same screen space.
- To provide a compact navigation mechanism for switching between related content sections.
- To synchronize tab triggers with their corresponding hidden panels using `aria-controls` and `aria-labelledby`.
- To ensure keyboard users can navigate and activate tabs using arrow keys and Enter.
- To support remote content loading via AJAX for panels that are not initially in the DOM.

### Syntax Rules and Structure

**Complete General Syntax (HTML Structure):**
```html
<div id="tabs">
    <ul role="tablist">
        <li role="tab" aria-selected="true" aria-controls="panel1" tabindex="0">Tab 1</li>
        <li role="tab" aria-selected="false" aria-controls="panel2" tabindex="-1">Tab 2</li>
    </ul>
    <div id="panel1" role="tabpanel" aria-hidden="false">Content 1</div>
    <div id="panel2" role="tabpanel" aria-hidden="true">Content 2</div>
</div>
```

**Complete General Syntax (jQuery UI Tabs):**
```javascript
$("#tabs").tabs({
    active: 0,
    collapsible: false,
    event: "click",
    heightStyle: "content"
});
```

| Component | Description |
|-----------|-------------|
| `role="tablist"` | Container for the tab triggers. |
| `role="tab"` | Each clickable tab trigger. |
| `aria-selected` | `"true"` on the active tab, `"false"` on others. |
| `aria-controls` | References the ID of the associated panel. |
| `role="tabpanel"` | Each content panel. |
| `aria-labelledby` | References the ID of the associated tab. |

**Syntax Rules:**

- Only one tab can have `aria-selected="true"` at a time.
- The active tab should have `tabindex="0"`; inactive tabs should have `tabindex="-1"`.
- When a tab is activated, update `aria-selected` on all tabs and `aria-hidden` on all panels.
- Use `aria-controls` on tabs and `aria-labelledby` on panels to associate them.
- jQuery UI Tabs handles this automatically when initialized on a properly structured DOM.

**Constraints and Limitations:**

- The tabs widget requires a specific DOM structure (tab list followed by panels) to function correctly.
- Remote tab loading introduces asynchronous complexity; error handling must be implemented.
- `collapsible: true` allows all tabs to be deselected, but this is not always desirable for accessibility.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: jQuery UI Tabs with ARIA Synchronization**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Tabs Demo</title>
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.13.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
</head>
<body>
  <div id="tabs">
    <ul>
      <li><a href="#tab1">Overview</a></li>
      <li><a href="#tab2">Details</a></li>
      <li><a href="#tab3">Reviews</a></li>
    </ul>
    <div id="tab1"><p>Overview content.</p></div>
    <div id="tab2"><p>Details content.</p></div>
    <div id="tab3"><p>Reviews content.</p></div>
  </div>

  <script>
    // Step 1: Initialize jQuery UI Tabs
    $("#tabs").tabs({
      activate: function(event, ui) {
        // Step 2: jQuery UI automatically updates ARIA attributes
        // Verify by logging the new panel's aria-hidden state
        var $newPanel = ui.newPanel;
        console.log("New panel aria-hidden: " + $newPanel.attr("aria-hidden"));
      }
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking the "Details" tab hides the "Overview" panel and shows the "Details" panel. The console logs the new panel's `aria-hidden` value as `"false"`.

**Why this output:** jQuery UI Tabs automatically manages `aria-selected` on tabs and `aria-hidden` on panels. The `activate` callback provides access to the new and old panels, allowing additional synchronization if needed.

### Real-World Cases

- **Product pages:** Tabs for Description, Specifications, and Reviews.
- **User dashboards:** Tabs for Profile, Settings, and Notifications.
- **Documentation sites:** Tabs for different programming language examples.

---

## Core Concept 3: Accordions

### Definitions

**Core Definition:** An accordion is a UI component that stacks multiple collapsible panels vertically, where activating one panel's header reveals its content and optionally collapses other panels (mutually exclusive behavior).

**Technical Definition:** An accordion consists of a series of header–content pairs. In the mutually exclusive (standard) configuration, only one panel is open at a time; opening a new panel closes the previously open one. In the independent toggle configuration, multiple panels can be open simultaneously. ARIA attributes include `aria-expanded` on the header button (indicating whether its panel is open), `aria-hidden` on the content panel, and `aria-controls` associating the header with its panel. jQuery UI's Accordion widget enforces mutually exclusive behavior by default, with the `collapsible` option allowing all panels to be closed.

**Beginner-Friendly Explanation:** An accordion is like a stack of cards where only the top card is fully visible. Clicking a card's header flips it open to reveal its content, while the previously open card flips closed. Some accordions let you have multiple cards open at once; others only allow one.

### Purposes

- To present large amounts of content in a compact vertical space.
- To enforce mutually exclusive panel states so users focus on one section at a time.
- To allow independent toggling when users need to compare content across panels.
- To communicate the expanded/collapsed state of each panel to assistive technologies.
- To provide a familiar, space-efficient pattern for FAQs, settings panels, and navigation.

### Syntax Rules and Structure

**Complete General Syntax (HTML Structure):**
```html
<div id="accordion">
    <h3><a href="#" aria-expanded="true" aria-controls="panel1">Section 1</a></h3>
    <div id="panel1" aria-hidden="false">Content 1</div>
    <h3><a href="#" aria-expanded="false" aria-controls="panel2">Section 2</a></h3>
    <div id="panel2" aria-hidden="true">Content 2</div>
</div>
```

**Complete General Syntax (jQuery UI Accordion):**
```javascript
$("#accordion").accordion({
    collapsible: true,
    active: 0,
    heightStyle: "content",
    animate: 200
});
```

| Component | Description |
|-----------|-------------|
| `aria-expanded` | `"true"` on open panel headers, `"false"` on closed. |
| `aria-hidden` | `"false"` on open panels, `"true"` on closed. |
| `aria-controls` | References the panel ID from the header. |
| `collapsible` | Allows all panels to be closed (jQuery UI option). |
| `active` | Index of the initially open panel. |

**Syntax Rules:**

- Headers should be semantic heading elements (`<h3>`) containing a link or button.
- `aria-expanded` must be updated on the header when the panel opens or closes.
- `aria-hidden` must be updated on the panel content.
- For mutually exclusive behavior, close all other panels when one opens.
- Use `collapsible: true` to allow all panels to be closed.

**Constraints and Limitations:**

- jQuery UI Accordion does not support multiple open panels; use a custom implementation or the `tabs` widget with `collapsible` for that behavior .
- Accordions should not be used if users need to see content from multiple panels simultaneously.
- Animation timing should be kept short (200–400ms) to avoid disorienting users.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Mutually Exclusive Accordion with ARIA Updates**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Accordion Demo</title>
  <style>
    #accordion { width: 400px; }
    #accordion h3 { margin: 0; padding: 10px; background: #eee; border-bottom: 1px solid #ccc; cursor: pointer; }
    #accordion div { padding: 10px; display: none; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="accordion">
    <h3><a href="#" aria-expanded="true" aria-controls="panel1">Section 1</a></h3>
    <div id="panel1" aria-hidden="false">Content for Section 1.</div>
    <h3><a href="#" aria-expanded="false" aria-controls="panel2">Section 2</a></h3>
    <div id="panel2" aria-hidden="true">Content for Section 2.</div>
    <h3><a href="#" aria-expanded="false" aria-controls="panel3">Section 3</a></h3>
    <div id="panel3" aria-hidden="true">Content for Section 3.</div>
  </div>

  <script>
    // Step 1: Show the first panel initially
    $("#accordion div:first").show();

    // Step 2: Handle header clicks
    $("#accordion h3 a").click(function(e) {
      e.preventDefault();
      var $this = $(this);
      var $target = $($this.attr("aria-controls"));

      // Step 3: If the target is already open, close it (collapsible)
      if ($this.attr("aria-expanded") === "true") {
        $this.attr("aria-expanded", "false");
        $target.attr("aria-hidden", "true").slideUp(200);
        return;
      }

      // Step 4: Close all other panels (mutually exclusive)
      $("#accordion h3 a").attr("aria-expanded", "false");
      $("#accordion h3 a").closest("h3").next("div")
        .attr("aria-hidden", "true").slideUp(200);

      // Step 5: Open the target panel
      $this.attr("aria-expanded", "true");
      $target.attr("aria-hidden", "false").slideDown(200);
    });
  </script>
</body>
</html>
```

**Expected Output:** Only one panel is open at a time. Clicking "Section 2" closes "Section 1" and opens "Section 2." Clicking "Section 2" again closes it (collapsible behavior). ARIA attributes update accordingly.

**Why this output:** The click handler first checks if the clicked panel is already open (for collapsible behavior). If not, it closes all other panels by setting `aria-expanded="false"` and `aria-hidden="true"` and sliding them up. Then it opens the target panel, updating ARIA attributes and sliding it down.

### Real-World Cases

- **FAQ pages:** Each question–answer pair is an accordion panel, keeping the page compact.
- **Settings panels:** Grouped settings categories that expand to reveal individual options.
- **Mobile navigation:** Accordion-style submenus that expand within a mobile menu.

---

## Core Concept 4: Modals

### Definitions

**Core Definition:** A modal is a UI component that displays content in an overlay on top of the main page, blocking interaction with the underlying content until the modal is dismissed. It includes focus trapping, dynamic overlay injection, and Escape-key dismissal.

**Technical Definition:** A modal dialog follows the WAI-ARIA Dialog design pattern. The modal container has `role="dialog"` and `aria-modal="true"`, with `aria-labelledby` referencing its title. An overlay (backdrop) is injected into the DOM to visually dim and block the underlying page. Focus is trapped inside the modal: when the user presses Tab, focus cycles through the modal's focusable elements and does not escape to the page behind it. When the modal opens, focus moves to the first focusable element (or the modal itself); when it closes, focus returns to the element that triggered it. The Escape key closes the modal.

**Beginner-Friendly Explanation:** A modal is a pop-up window that appears on top of the page and demands your attention. While it is open, you cannot click anything behind it. For keyboard users, pressing Tab keeps focus inside the modal, and pressing Escape closes it. When you close the modal, the focus goes back to the button you clicked to open it.

### Purposes

- To draw the user's attention to a critical piece of content (alert, confirmation, form).
- To block interaction with the underlying page until the modal is dismissed.
- To trap keyboard focus inside the modal so users cannot tab to the page behind it.
- To provide Escape-key dismissal for keyboard users.
- To return focus to the trigger element when the modal closes, preserving the user's place.

### Syntax Rules and Structure

**Complete General Syntax (HTML Structure):**
```html
<button class="modal-trigger" data-modal-target="myModal">Open Modal</button>

<div class="modal-overlay" id="modalOverlay" aria-hidden="true"></div>
<div data-modal-target="myModal" class="modal" role="dialog" aria-labelledby="modal-title" aria-modal="true">
    <button class="modal-close" aria-label="Close">×</button>
    <h2 id="modal-title">Modal Title</h2>
    <p>Modal content.</p>
</div>
```

**Complete General Syntax (Focus Trapping Logic):**
```javascript
function trapFocus(element) {
    var focusableElements = element.find("a[href], button, textarea, input, select, [tabindex]:not([tabindex='-1'])")
        .filter(":visible");
    var firstElement = focusableElements.first();
    var lastElement = focusableElements.last();

    element.on("keydown", function(e) {
        if (e.key === "Tab") {
            if (e.shiftKey) {
                if ($(document.activeElement).is(firstElement)) {
                    e.preventDefault();
                    lastElement.focus();
                }
            } else {
                if ($(document.activeElement).is(lastElement)) {
                    e.preventDefault();
                    firstElement.focus();
                }
            }
        }
    });
}
```

| Component | Description |
|-----------|-------------|
| `role="dialog"` | Identifies the modal as a dialog. |
| `aria-modal="true"` | Indicates the dialog is modal (blocks interaction). |
| `aria-labelledby` | References the modal's title. |
| `tabindex="-1"` | Allows the modal itself to receive focus. |
| Focus trapping | Cycles Tab focus between first and last focusable elements. |
| `aria-hidden="true"` on overlay | Hides the overlay from screen readers when closed. |

**Syntax Rules:**

- The modal should have `role="dialog"` and `aria-modal="true"`.
- Focus must move into the modal when it opens and return to the trigger when it closes.
- Tab and Shift+Tab must cycle through the modal's focusable elements.
- Escape must close the modal.
- The overlay should be injected dynamically or toggled via `aria-hidden`.

**Constraints and Limitations:**

- Focus trapping requires careful handling of dynamic content (elements added/removed from the modal).
- If the modal contains an iframe, focus trapping may not work correctly.
- The `inert` attribute (or `aria-hidden` on the page content) is recommended to hide the rest of the page from assistive technology .

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Complete Modal with Focus Trap and Escape**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Modal Demo</title>
  <style>
    .modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 1000; }
    .modal { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #fff; padding: 20px; z-index: 1001; max-width: 500px; width: 90%; }
    .modal-close { position: absolute; top: 10px; right: 10px; cursor: pointer; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button class="modal-trigger" data-modal-target="myModal" id="openBtn">Open Modal</button>

  <div class="modal-overlay" id="modalOverlay" aria-hidden="true"></div>
  <div data-modal-target="myModal" class="modal" role="dialog" aria-labelledby="modal-title" aria-modal="true" id="myModal">
    <button class="modal-close" aria-label="Close">×</button>
    <h2 id="modal-title">Modal Title</h2>
    <p>This is modal content.</p>
    <button>Action</button>
  </div>

  <script>
    $(function() {
      var $lastFocused;

      // Step 1: Open modal
      $(".modal-trigger").click(function() {
        $lastFocused = $(document.activeElement);
        var target = $(this).data("modal-target");
        $("#" + target).show();
        $("#modalOverlay").show().attr("aria-hidden", "false");
        $("#" + target).attr("tabindex", "-1").focus();
        trapFocus($("#" + target));
      });

      // Step 2: Close modal
      function closeModal($modal) {
        $modal.hide();
        $("#modalOverlay").hide().attr("aria-hidden", "true");
        $modal.off("keydown");
        $lastFocused.focus();
      }

      $(".modal-close, #modalOverlay").click(function() {
        closeModal($(".modal:visible"));
      });

      // Step 3: Escape key closes modal
      $(document).on("keydown", function(e) {
        if (e.key === "Escape" && $(".modal:visible").length) {
          closeModal($(".modal:visible"));
        }
      });

      // Step 4: Focus trapping function
      function trapFocus($modal) {
        var focusable = $modal.find("a[href], button, textarea, input, select, [tabindex]:not([tabindex='-1'])")
          .filter(":visible");
        var first = focusable.first();
        var last = focusable.last();

        $modal.on("keydown", function(e) {
          if (e.key === "Tab") {
            if (e.shiftKey) {
              if ($(document.activeElement).is(first)) {
                e.preventDefault();
                last.focus();
              }
            } else {
              if ($(document.activeElement).is(last)) {
                e.preventDefault();
                first.focus();
              }
            }
          }
        });
      }
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Open Modal" displays the modal with the overlay. Focus moves to the modal (or its first focusable element). Pressing Tab cycles focus within the modal. Pressing Escape closes the modal and returns focus to the "Open Modal" button. Clicking the close button or overlay also closes the modal.

**Why this output:** The open handler stores the last focused element, shows the modal and overlay, and moves focus into the modal. The `trapFocus` function intercepts Tab keypresses and cycles focus between the first and last focusable elements. The close handler hides the modal, unbinds the focus trap, and returns focus to the trigger.

### Real-World Cases

- **Login/signup forms:** Modal dialogs for authentication without leaving the page.
- **Confirmation prompts:** "Are you sure you want to delete this?" modals.
- **Image lightboxes:** Clicking a thumbnail opens the full-size image in a modal.
- **Cookie consent banners:** Modal-style consent dialogs.

---

## Core Concept 5: Dropdowns

### Definitions

**Core Definition:** A dropdown is a UI component that reveals a list of options or menu items when a trigger element is clicked or focused. It dismisses when the user clicks outside or presses Escape.

**Technical Definition:** A dropdown widget combines a trigger (button or link) with a hidden panel containing links or options. When opened, the panel is revealed and `aria-expanded` is set to `"true"`. Outside-click dismissal is implemented by binding a click handler to the document that closes the dropdown if the click target is not within the dropdown. Focus management ensures that clicking the trigger focuses the dropdown, and closing it returns focus to the trigger. The WAI-ARIA Menu Button design pattern provides the keyboard interaction model: Enter/Space opens, arrow keys navigate items, Escape closes.

**Beginner-Friendly Explanation:** A dropdown is like a drawer that slides out when you click a button. It shows a list of options. If you click anywhere else on the page, the drawer closes. For keyboard users, pressing Escape closes it and returns focus to the button.

### Purposes

- To present a list of options without cluttering the page.
- To dismiss the dropdown when the user clicks outside, providing intuitive behavior.
- To manage focus so keyboard users can open, navigate, and close the dropdown.
- To communicate the open/closed state to assistive technologies via `aria-expanded`.
- To support single-select and multi-select dropdown patterns.

### Syntax Rules and Structure

**Complete General Syntax (HTML Structure):**
```html
<div class="dropdown">
    <button class="dropdown-trigger" aria-haspopup="true" aria-expanded="false">Options</button>
    <ul class="dropdown-menu" role="menu" aria-hidden="true">
        <li role="menuitem"><a href="#">Option 1</a></li>
        <li role="menuitem"><a href="#">Option 2</a></li>
    </ul>
</div>
```

**Complete General Syntax (Outside-Click Dismissal):**
```javascript
$(document).on("click", function(e) {
    if (!$(e.target).closest(".dropdown").length) {
        $(".dropdown-menu").attr("aria-hidden", "true").hide();
        $(".dropdown-trigger").attr("aria-expanded", "false");
    }
});
```

| Component | Description |
|-----------|-------------|
| `aria-haspopup="true"` | Indicates the trigger opens a menu. |
| `aria-expanded` | `"true"` when open, `"false"` when closed. |
| `aria-hidden` | Hides the menu from screen readers when closed. |
| `.closest(".dropdown")` | Checks if the click was inside the dropdown. |
| `role="menu"` / `role="menuitem"` | ARIA roles for the menu structure. |

**Syntax Rules:**

- Use `e.stopPropagation()` on the trigger click to prevent the document click handler from immediately closing the dropdown.
- Check `$(e.target).closest(".dropdown").length` to determine if the click was inside the dropdown.
- Return focus to the trigger when the dropdown closes.
- Use `keydown` to handle Escape and arrow key navigation.

**Constraints and Limitations:**

- Binding a click handler to `document` can conflict with other click handlers; use namespaced events (`.dropdown`) and unbind on destroy.
- Native `<select>` elements have built-in dropdown behavior; custom dropdowns must reimplement all keyboard and accessibility features.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Dropdown with Outside-Click Dismissal and Focus Management**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Dropdown Demo</title>
  <style>
    .dropdown { position: relative; display: inline-block; }
    .dropdown-menu { display: none; position: absolute; top: 100%; left: 0; background: #fff; border: 1px solid #ccc; list-style: none; padding: 0; margin: 0; min-width: 150px; }
    .dropdown-menu li a { display: block; padding: 8px 15px; text-decoration: none; color: #333; }
    .dropdown-menu li a:hover { background: #f0f0f0; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="dropdown">
    <button class="dropdown-trigger" aria-haspopup="true" aria-expanded="false" id="ddTrigger">Options</button>
    <ul class="dropdown-menu" role="menu" aria-hidden="true">
      <li role="menuitem"><a href="#">Profile</a></li>
      <li role="menuitem"><a href="#">Settings</a></li>
      <li role="menuitem"><a href="#">Logout</a></li>
    </ul>
  </div>

  <script>
    $(function() {
      var $trigger = $(".dropdown-trigger");
      var $menu = $(".dropdown-menu");

      // Step 1: Toggle dropdown on trigger click
      $trigger.on("click", function(e) {
        e.stopPropagation();
        var isOpen = $menu.attr("aria-hidden") === "false";
        $menu.attr("aria-hidden", isOpen ? "true" : "false")
             .slideToggle(150);
        $trigger.attr("aria-expanded", isOpen ? "false" : "true");
      });

      // Step 2: Close on outside click
      $(document).on("click.dropdown", function(e) {
        if (!$(e.target).closest(".dropdown").length) {
          if ($menu.attr("aria-hidden") === "false") {
            $menu.attr("aria-hidden", "true").slideUp(150);
            $trigger.attr("aria-expanded", "false");
          }
        }
      });

      // Step 3: Escape key closes and returns focus
      $(document).on("keydown.dropdown", function(e) {
        if (e.key === "Escape" && $menu.attr("aria-hidden") === "false") {
          $menu.attr("aria-hidden", "true").slideUp(150);
          $trigger.attr("aria-expanded", "false").focus();
        }
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Options" opens the dropdown menu. Clicking anywhere outside the dropdown closes it. Pressing Escape closes it and returns focus to the trigger button.

**Why this output:** The trigger click handler toggles the menu and stops propagation to prevent the document click handler from immediately closing it. The document click handler checks if the click was inside the dropdown; if not, it closes the menu. The keydown handler closes the menu on Escape and returns focus to the trigger.

### Real-World Cases

- **User account menus:** Profile, Settings, Logout.
- **Filter dropdowns:** Selecting categories or sort order.
- **Form select replacements:** Custom-styled dropdowns for better visual control.

---

## Core Concept 6: Tooltips

### Definitions

**Core Definition:** A tooltip is a small pop-up element that displays additional information when the user hovers over or focuses on a trigger element. Dynamic position recalculation ensures the tooltip remains within the viewport even near edges.

**Technical Definition:** A tooltip widget attaches to a trigger element and displays a positioned overlay containing supplementary text or content. The tooltip's position is calculated relative to the trigger, but must be adjusted if it would overflow the viewport edges. The calculation involves: (1) determining the trigger's offset and dimensions; (2) determining the tooltip's dimensions; (3) comparing the proposed position against the viewport boundaries (`$(window).width()`, `$(window).height()`, `$(window).scrollTop()`, `$(window).scrollLeft()`); and (4) adjusting the position (e.g., flipping from top to bottom, or shifting left) to keep the tooltip visible. The WAI-ARIA Tooltip pattern uses `role="tooltip"` and `aria-describedby` to associate the tooltip with its trigger.

**Beginner-Friendly Explanation:** A tooltip is like a sticky note that appears when you hover over something. If the sticky note would go off the edge of the screen, it automatically moves to the other side so you can still read it. This is especially important for elements near the right or bottom edges of the browser window.

### Purposes

- To provide supplementary information (descriptions, hints, labels) without cluttering the UI.
- To keep the tooltip visible within the viewport by recalculating its position near edges.
- To communicate the tooltip's content to assistive technologies via `role="tooltip"` and `aria-describedby`.
- To support both hover and keyboard focus triggers for accessibility.
- To avoid the horizontal scrollbar bug caused by tooltips extending beyond the viewport width .

### Syntax Rules and Structure

**Complete General Syntax (Position Calculation):**
```javascript
function positionTooltip($tooltip, $trigger) {
    var triggerOffset = $trigger.offset();
    var triggerWidth = $trigger.outerWidth();
    var triggerHeight = $trigger.outerHeight();
    var tooltipWidth = $tooltip.outerWidth();
    var tooltipHeight = $tooltip.outerHeight();
    var winWidth = $(window).width();
    var winHeight = $(window).height();
    var scrollTop = $(window).scrollTop();
    var scrollLeft = $(window).scrollLeft();

    // Proposed position (above the trigger, centered)
    var top = triggerOffset.top - tooltipHeight - 10;
    var left = triggerOffset.left + (triggerWidth / 2) - (tooltipWidth / 2);

    // Adjust for viewport edges
    if (top < scrollTop) {
        top = triggerOffset.top + triggerHeight + 10; // Flip below
    }
    if (left < scrollLeft) {
        left = scrollLeft + 5; // Shift right
    }
    if (left + tooltipWidth > scrollLeft + winWidth) {
        left = scrollLeft + winWidth - tooltipWidth - 5; // Shift left
    }

    $tooltip.css({ top: top, left: left });
}
```

| Component | Description |
|-----------|-------------|
| `$trigger.offset()` | Document-relative position of the trigger. |
| `$(window).width()` | Viewport width. |
| `$(window).scrollTop()` | Current vertical scroll offset. |
| Edge checks | Compare proposed position against viewport boundaries. |
| Flip logic | Move tooltip below trigger if it would overflow the top. |
| Shift logic | Move tooltip left/right if it would overflow horizontally. |

**Syntax Rules:**

- Calculate the tooltip's dimensions **after** it is inserted into the DOM (or after making it visible with `visibility: hidden` and `position: absolute`).
- Use `$(window).scrollTop()` and `$(window).scrollLeft()` to account for the current scroll position.
- Apply a safety buffer (e.g., 5–20px) to prevent the tooltip from touching the viewport edge .
- Recalculate position on window resize and scroll if the tooltip remains open.

**Constraints and Limitations:**

- Recalculating position on every `mousemove` event can cause performance issues; throttle or debounce the calculation .
- Tooltips that follow the mouse cursor are more prone to edge overflow than static, trigger-anchored tooltips.
- The tooltip must be positioned absolutely (or fixed) to allow precise coordinate control.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Viewport-Aware Tooltip**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Tooltip Demo</title>
  <style>
    .tooltip { position: absolute; background: #333; color: #fff; padding: 5px 10px; border-radius: 4px; font-size: 14px; display: none; z-index: 1000; white-space: nowrap; }
    .trigger { display: inline-block; margin: 100px; padding: 10px; background: #eee; border: 1px solid #ccc; }
    #edgeTrigger { margin-left: 90%; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <span class="trigger" data-tooltip="This is a tooltip">Hover me</span>
  <span class="trigger" id="edgeTrigger" data-tooltip="This tooltip would overflow">Edge case</span>

  <script>
    $(function() {
      // Step 1: Create the tooltip element
      var $tooltip = $("<div class='tooltip' role='tooltip'></div>").appendTo("body");

      // Step 2: Show tooltip on hover
      $(".trigger").hover(
        function() {
          var $trigger = $(this);
          $tooltip.text($trigger.data("tooltip")).show();
          positionTooltip($tooltip, $trigger);
        },
        function() {
          $tooltip.hide();
        }
      );

      // Step 3: Position tooltip with viewport edge detection
      function positionTooltip($tip, $trigger) {
        var triggerOffset = $trigger.offset();
        var triggerWidth = $trigger.outerWidth();
        var triggerHeight = $trigger.outerHeight();
        var tipWidth = $tip.outerWidth();
        var tipHeight = $tip.outerHeight();
        var winWidth = $(window).width();
        var scrollTop = $(window).scrollTop();
        var scrollLeft = $(window).scrollLeft();

        // Default: above the trigger, centered
        var top = triggerOffset.top - tipHeight - 10;
        var left = triggerOffset.left + (triggerWidth / 2) - (tipWidth / 2);

        // Flip below if overflowing top
        if (top < scrollTop) {
          top = triggerOffset.top + triggerHeight + 10;
        }

        // Shift right if overflowing left
        if (left < scrollLeft) {
          left = scrollLeft + 5;
        }

        // Shift left if overflowing right
        if (left + tipWidth > scrollLeft + winWidth) {
          left = scrollLeft + winWidth - tipWidth - 5;
        }

        $tip.css({ top: top, left: left });
      }
    });
  </script>
</body>
</html>
```

**Expected Output:** Hovering over "Hover me" displays the tooltip above the trigger. Hovering over "Edge case" (near the right edge) displays the tooltip shifted left so it remains fully visible within the viewport.

**Why this output:** The `positionTooltip` function calculates the default position above and centered on the trigger, then checks for overflow against the viewport's top, left, and right edges. If the tooltip would overflow the top, it flips below; if it would overflow left or right, it shifts inward with a 5px safety buffer.

### Real-World Cases

- **Icon buttons:** Tooltips explaining what toolbar icons do.
- **Form fields:** Hints about expected input format.
- **Data tables:** Tooltips showing truncated cell content.
- **Charts:** Tooltips displaying data point values on hover.

---

## Core Concept 7: Carousels

### Definitions

**Core Definition:** A carousel is a UI component that displays a series of slides (images, content panels) in a horizontal or vertical sequence, with navigation controls, optional auto-play, and touch/swipe support for mobile devices.

**Technical Definition:** A carousel widget manages a collection of slide elements, a current slide index, and a timer for auto-play. It provides next/previous navigation controls, pagination indicators, and touch/swipe event handling. Auto-play uses `setInterval` or `setTimeout` to advance slides at a configurable interval; the timer must be cleared on user interaction (hover, touch) and on destroy to prevent memory leaks. Touch/swipe mapping converts touch events (`touchstart`, `touchmove`, `touchend`) into slide transitions based on the swipe distance and velocity. Slide indexing tracks the current position and supports circular (infinite) looping.

**Beginner-Friendly Explanation:** A carousel is like a slide projector that automatically advances through a set of slides. Users can also click arrows or swipe to move between slides. On mobile, swiping left or right changes the slide. The carousel needs to keep track of which slide is currently showing and manage a timer for auto-play.

### Purposes

- To display multiple pieces of content (images, promotions, testimonials) in a compact space.
- To auto-play slides on a timer, automatically advancing without user interaction.
- To support touch and swipe gestures for mobile users.
- To track the current slide index for pagination indicators and navigation controls.
- To pause auto-play on hover or touch interaction, resuming when the user is done.

### Syntax Rules and Structure

**Complete General Syntax (Auto-Play Timer):**
```javascript
var currentIndex = 0;
var slideCount = $(".slide").length;
var autoPlayInterval = 3000;
var timer;

function startAutoPlay() {
    timer = setInterval(function() {
        currentIndex = (currentIndex + 1) % slideCount;
        goToSlide(currentIndex);
    }, autoPlayInterval);
}

function stopAutoPlay() {
    clearInterval(timer);
}
```

**Complete General Syntax (Touch/Swipe Mapping):**
```javascript
var touchStartX = 0;

$(".carousel").on("touchstart", function(e) {
    touchStartX = e.originalEvent.touches[0].clientX;
});

$(".carousel").on("touchend", function(e) {
    var touchEndX = e.originalEvent.changedTouches[0].clientX;
    var diff = touchStartX - touchEndX;
    if (Math.abs(diff) > 50) { // Minimum swipe distance
        if (diff > 0) {
            // Swiped left — next slide
            nextSlide();
        } else {
            // Swiped right — previous slide
            prevSlide();
        }
    }
});
```

| Component | Description |
|-----------|-------------|
| `setInterval` | Timer for auto-play advancement. |
| `currentIndex` | Tracks the active slide. |
| `clearInterval(timer)` | Stops auto-play. |
| `touchstart` / `touchend` | Touch events for swipe detection. |
| Swipe distance threshold | Minimum distance (e.g., 50px) to register a swipe. |

**Syntax Rules:**

- Store the timer reference so it can be cleared on pause or destroy.
- Pause auto-play on `mouseenter`/`touchstart` and resume on `mouseleave`/`touchend` .
- Use `e.originalEvent.touches` to access touch coordinates in jQuery.
- Apply a minimum swipe distance threshold to avoid accidental swipes.
- Update the slide index and translate the slide track accordingly.

**Constraints and Limitations:**

- Auto-play timers must be cleared on destroy to prevent memory leaks.
- Touch events are not available on all devices; provide keyboard and mouse alternatives.
- Circular looping requires careful index management to avoid jumps.
- CSS transitions should be used for smooth slide animations; jQuery's `.animate()` can be used as a fallback.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Carousel with Auto-Play and Touch Swipe**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Carousel Demo</title>
  <style>
    .carousel { width: 400px; overflow: hidden; position: relative; }
    .slides { display: flex; transition: transform 0.3s ease; }
    .slide { min-width: 400px; height: 200px; display: flex; align-items: center; justify-content: center; font-size: 24px; color: #fff; }
    .slide:nth-child(1) { background: #007bff; }
    .slide:nth-child(2) { background: #28a745; }
    .slide:nth-child(3) { background: #dc3545; }
    .controls { text-align: center; margin-top: 10px; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="carousel">
    <div class="slides">
      <div class="slide">Slide 1</div>
      <div class="slide">Slide 2</div>
      <div class="slide">Slide 3</div>
    </div>
  </div>
  <div class="controls">
    <button id="prev">Prev</button>
    <button id="next">Next</button>
  </div>

  <script>
    $(function() {
      var currentIndex = 0;
      var slideCount = $(".slide").length;
      var timer;
      var touchStartX = 0;
      var $slides = $(".slides");

      // Step 1: Go to a specific slide
      function goToSlide(index) {
        if (index < 0) index = slideCount - 1;
        if (index >= slideCount) index = 0;
        currentIndex = index;
        $slides.css("transform", "translateX(-" + (currentIndex * 400) + "px)");
      }

      // Step 2: Auto-play
      function startAutoPlay() {
        timer = setInterval(function() {
          goToSlide(currentIndex + 1);
        }, 3000);
      }

      function stopAutoPlay() {
        clearInterval(timer);
      }

      // Step 3: Navigation controls
      $("#next").click(function() { goToSlide(currentIndex + 1); });
      $("#prev").click(function() { goToSlide(currentIndex - 1); });

      // Step 4: Pause on hover
      $(".carousel").hover(stopAutoPlay, startAutoPlay);

      // Step 5: Touch swipe support
      $(".carousel").on("touchstart", function(e) {
        touchStartX = e.originalEvent.touches[0].clientX;
        stopAutoPlay();
      });

      $(".carousel").on("touchend", function(e) {
        var touchEndX = e.originalEvent.changedTouches[0].clientX;
        var diff = touchStartX - touchEndX;
        if (Math.abs(diff) > 50) {
          if (diff > 0) {
            goToSlide(currentIndex + 1);
          } else {
            goToSlide(currentIndex - 1);
          }
        }
        startAutoPlay();
      });

      // Step 6: Start auto-play
      startAutoPlay();
    });
  </script>
</body>
</html>
```

**Expected Output:** The carousel auto-advances through slides every 3 seconds. Clicking "Next" or "Prev" navigates manually. Hovering over the carousel pauses auto-play. Swiping left or right on a touch device changes the slide.

**Why this output:** The `goToSlide` function updates the `transform: translateX` property to shift the slides. The auto-play timer advances the index every 3 seconds. Hover and touch handlers pause the timer, and the touch handlers detect swipe direction based on the difference between `touchstart` and `touchend` X coordinates.

### Real-World Cases

- **E-commerce homepages:** Auto-playing carousels showcasing promotions and featured products.
- **Testimonial sliders:** Rotating customer reviews.
- **Image galleries:** Swipeable photo galleries on mobile devices.
- **Onboarding tours:** Step-by-step carousels introducing app features.

---

## Enhanced Topic: Programmatic ARIA Attribute Updates During UI Transitions

### Definitions

**Core Definition:** Programmatic ARIA attribute updates are the JavaScript-driven modifications of WAI-ARIA attributes (`aria-expanded`, `aria-hidden`, `aria-selected`) on DOM elements to reflect UI state changes, ensuring assistive technologies remain synchronized with visual transitions.

**Technical Definition:** ARIA attributes communicate the state and relationships of UI components to assistive technologies. During UI transitions (opening a menu, switching a tab, expanding an accordion), the corresponding ARIA attributes must be updated programmatically to reflect the new state. The three core attributes are: `aria-expanded` (indicates whether a collapsible element is expanded or collapsed), `aria-hidden` (indicates whether an element is hidden from assistive technologies), and `aria-selected` (indicates whether a tab or option is currently selected). These attributes must be updated **before or during** the visual transition, not after, to prevent screen readers from announcing stale state information.

**Beginner-Friendly Explanation:** Screen readers rely on ARIA attributes to tell blind users what is happening on the screen. When you open a menu, the screen reader needs to know that the menu is now expanded. If you only update the visual CSS but forget the ARIA attributes, the screen reader tells the user the menu is still closed — which is confusing and inaccessible.

### Purposes

- To keep assistive technologies synchronized with visual UI state changes.
- To communicate expanded/collapsed states (`aria-expanded`) for menus and accordions.
- To communicate visibility states (`aria-hidden`) for panels and overlays.
- To communicate selection states (`aria-selected`) for tabs and listbox options.
- To comply with WCAG 2.2 and Section 508 accessibility requirements.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
// Accordion: toggle aria-expanded and aria-hidden
$header.attr("aria-expanded", "true");
$panel.attr("aria-hidden", "false");

// Tabs: toggle aria-selected and aria-hidden
$tab.attr("aria-selected", "true");
$panel.attr("aria-hidden", "false");

// Dropdown: toggle aria-expanded
$trigger.attr("aria-expanded", "true");
```

| Attribute | Purpose | Values |
|-----------|---------|--------|
| `aria-expanded` | Indicates whether a collapsible element is expanded | `"true"` / `"false"` |
| `aria-hidden` | Indicates whether an element is hidden from assistive tech | `"true"` / `"false"` |
| `aria-selected` | Indicates whether a tab/option is selected | `"true"` / `"false"` |

**Syntax Rules:**

- Update ARIA attributes **before** starting the visual animation or at the same time.
- Use `.attr()` to set ARIA attributes in jQuery.
- Toggle attributes between `"true"` and `"false"` (strings, not booleans).
- Ensure only one tab has `aria-selected="true"` at a time.
- For accordions, update both `aria-expanded` on the header and `aria-hidden` on the panel.

**Constraints and Limitations:**

- `aria-hidden="true"` on an element that contains focusable elements can create accessibility issues; use `inert` or `tabindex="-1"` as well.
- ARIA attributes do not change visual appearance; CSS must still handle the visual state.
- Overusing `aria-hidden` can hide content from screen readers that should be accessible.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: ARIA Updates in an Accordion**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>ARIA Accordion Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="accordion">
    <h3><a href="#" aria-expanded="true" aria-controls="p1">Section 1</a></h3>
    <div id="p1" aria-hidden="false">Content 1</div>
    <h3><a href="#" aria-expanded="false" aria-controls="p2">Section 2</a></h3>
    <div id="p2" aria-hidden="true">Content 2</div>
  </div>

  <script>
    $(function() {
      $("#accordion h3 a").click(function(e) {
        e.preventDefault();
        var $this = $(this);
        var $target = $($this.attr("aria-controls"));

        if ($this.attr("aria-expanded") === "true") {
          // Close the panel
          $this.attr("aria-expanded", "false");
          $target.attr("aria-hidden", "true").slideUp(200);
        } else {
          // Close all other panels
          $("#accordion h3 a").attr("aria-expanded", "false");
          $("#accordion h3 a").closest("h3").next("div")
            .attr("aria-hidden", "true").slideUp(200);

          // Open the target panel
          $this.attr("aria-expanded", "true");
          $target.attr("aria-hidden", "false").slideDown(200);
        }
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Section 2" closes "Section 1" and opens "Section 2." Screen readers announce "Section 2, expanded" and "Section 1, collapsed." The ARIA attributes are updated before the slide animations begin.

**Why this output:** The click handler updates `aria-expanded` and `aria-hidden` **before** calling `.slideUp()` and `.slideDown()`. This ensures that assistive technologies receive the state change immediately, even if the visual animation takes 200ms.

### Real-World Cases

- **Accordion components:** Updating `aria-expanded` and `aria-hidden` on every toggle.
- **Tab widgets:** Updating `aria-selected` on tab triggers and `aria-hidden` on panels.
- **Dropdown menus:** Updating `aria-expanded` on the trigger when the menu opens/closes.
- **Modal dialogs:** Setting `aria-hidden="true"` on the page content behind the modal.

---

## Enhanced Topic: Focus Management — Moving and Restoring Focus Smoothly

### Definitions

**Core Definition:** Focus management is the practice of programmatically controlling which element has keyboard focus during and after UI interactions, ensuring keyboard users can navigate efficiently and are never trapped or disoriented.

**Technical Definition:** Focus management involves three key operations: (1) **Moving focus** to the appropriate element when a UI component opens or activates (e.g., moving focus to the first focusable element in a modal); (2) **Trapping focus** within a component (e.g., keeping Tab focus inside a modal until it closes); and (3) **Restoring focus** to the element that triggered the interaction when the component closes (e.g., returning focus to the "Open Modal" button). These operations are implemented using the native `.focus()` method and the `document.activeElement` property. jQuery's `.focus()` and `.trigger("focus")` methods are used to move focus programmatically.

**Beginner-Friendly Explanation:** When a keyboard user opens a modal, their focus should move into the modal so they can interact with it. While the modal is open, pressing Tab should keep them inside the modal, not let them tab to the page behind it. When they close the modal, focus should return to the button they used to open it, so they do not lose their place on the page.

### Purposes

- To move focus into a component when it opens, so keyboard users can interact with it immediately.
- To trap focus within a component (modal, menu) so users cannot tab to inaccessible content behind it.
- To restore focus to the trigger element when the component closes, preserving the user's navigation context.
- To prevent keyboard traps where focus becomes stuck and users cannot escape.
- To comply with WCAG 2.2 Success Criterion 2.4.3 (Focus Order) and 2.1.2 (No Keyboard Trap).

### Syntax Rules and Structure

**Complete General Syntax (Moving and Restoring Focus):**
```javascript
// Store the trigger element
var $trigger = $(document.activeElement);

// Move focus into the component
$component.find("button, [href], input, select, textarea, [tabindex]:not([tabindex='-1'])")
    .filter(":visible")
    .first()
    .focus();

// On close, restore focus
$trigger.focus();
```

**Complete General Syntax (Focus Trapping):**
```javascript
$component.on("keydown", function(e) {
    if (e.key === "Tab") {
        var focusable = $component.find("button, [href], input, select, textarea, [tabindex]:not([tabindex='-1'])")
            .filter(":visible");
        var first = focusable.first();
        var last = focusable.last();

        if (e.shiftKey) {
            if ($(document.activeElement).is(first)) {
                e.preventDefault();
                last.focus();
            }
        } else {
            if ($(document.activeElement).is(last)) {
                e.preventDefault();
                first.focus();
            }
        }
    }
});
```

| Operation | Method | Purpose |
|-----------|--------|---------|
| Move focus | `.focus()` | Set focus to a specific element. |
| Detect focus | `document.activeElement` | Get the currently focused element. |
| Trap focus | `keydown` + `.preventDefault()` | Cycle Tab focus within a component. |
| Restore focus | `.focus()` on stored trigger | Return focus when the component closes. |

**Syntax Rules:**

- Store the trigger element **before** moving focus away from it: `var $trigger = $(document.activeElement)`.
- Move focus to the first focusable element in the component, or to the component itself if it has `tabindex="-1"`.
- Bind the focus trap `keydown` handler to the component container, not the document.
- Restore focus in the close handler, after the component is hidden.
- Use `.filter(":visible")` to exclude hidden elements from the focusable list.

**Constraints and Limitations:**

- If the trigger element is removed from the DOM while the modal is open, focus restoration will fail; store a reference to the element or its selector.
- Focus trapping requires that the component contains at least one focusable element; if not, focus the component itself with `tabindex="-1"`.
- Browser default focus behavior varies; always test in multiple browsers and screen readers.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Focus Management in a Modal**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Focus Management Demo</title>
  <style>
    .modal { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #fff; padding: 20px; border: 1px solid #ccc; z-index: 1000; }
    .overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 999; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="openBtn">Open Modal</button>
  <div class="overlay" id="overlay"></div>
  <div class="modal" id="modal" role="dialog" aria-modal="true" aria-labelledby="title">
    <h2 id="title">Modal</h2>
    <button id="closeBtn">Close</button>
    <button id="actionBtn">Action</button>
  </div>

  <script>
    $(function() {
      var $lastFocused;

      // Step 1: Open modal and move focus
      $("#openBtn").click(function() {
        $lastFocused = $(document.activeElement);
        $("#overlay, #modal").show();
        $("#modal").attr("tabindex", "-1").focus();
        trapFocus($("#modal"));
      });

      // Step 2: Close modal and restore focus
      $("#closeBtn, #overlay").click(function() {
        $("#overlay, #modal").hide();
        $("#modal").off("keydown");
        $lastFocused.focus();
      });

      // Step 3: Focus trapping
      function trapFocus($modal) {
        var focusable = $modal.find("button, [href], input, select, textarea, [tabindex]:not([tabindex='-1'])")
          .filter(":visible");
        var first = focusable.first();
        var last = focusable.last();

        $modal.on("keydown", function(e) {
          if (e.key === "Tab") {
            if (e.shiftKey) {
              if ($(document.activeElement).is(first)) {
                e.preventDefault();
                last.focus();
              }
            } else {
              if ($(document.activeElement).is(last)) {
                e.preventDefault();
                first.focus();
              }
            }
          }
        });
      }
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Open Modal" displays the modal and moves focus to the modal container (which has `tabindex="-1"`). Pressing Tab cycles focus between the "Close" and "Action" buttons. Pressing Shift+Tab also cycles. Clicking "Close" or the overlay hides the modal and returns focus to the "Open Modal" button.

**Why this output:** The open handler stores the last focused element (the "Open Modal" button), shows the modal, and focuses the modal container. The `trapFocus` function intercepts Tab keypresses and cycles focus between the first and last focusable elements. The close handler hides the modal and restores focus to the stored trigger element.

### Real-World Cases

- **Modal dialogs:** Moving focus into the modal, trapping it, and restoring it on close.
- **Dropdown menus:** Moving focus to the first menu item when the dropdown opens, and returning it to the trigger on close.
- **Navigation menus:** Managing focus as users navigate multi-level menus with arrow keys.
- **Carousels:** Moving focus to the active slide or navigation controls.

---

## References

- jQuery for Designers: Beginners Guide, 2nd Edition — https://dl.acm.org/doi/book/10.5555/2692694
- Adding ARIA Attributes to Accordion — https://stackoverflow.com/questions/44688535/adding-aria-attributes-to-accordion/44688639
- Accessible jQuery Modal Plugin with Focus Trapping and CSS3 Animations — https://www.jqueryscript.net/lightbox/modal-focus-trapping.html
- owl.carousel.js插件 — https://cloud.tencent.com.cn/developer/information/owl.carousel.js插件-video
- [Bug]: dnnHelperTip causes horizontal scrollbar when moving mouse to the right edge — https://github.com/dnnsoftware/Dnn.Platform/issues/6923
- 用 JQuery UI 建立資料驅動的折疊式選單 — https://learn.microsoft.com/zh-tw/previous-versions/msdn10/ff452698(v=msdn.10)
- jQuery UI Tabs Documentation — https://github.itap.purdue.edu/Salesforce-Resource-Center-Orgs/MMTC/blob/NewTriggers/force-app/main/default/staticresources/JQUI1102/development-bundle/docs/tabs.html
- Kendo UI for jQuery Menu Accessibility — https://www.telerik.com/kendo-jquery-ui/documentation/accessibility/menu
- jQuery Menu Accessibility Wai-Aria Support — https://www.telerik.com/kendo-jquery-ui/documentation/accessibility/menu
- Trap Focus Inside the Modal — https://stackoverflow.com/questions/78911655/trap-focus-inside-the-modal
- qTip2 adjustments — https://wordpress.org/support/topic/qtip2-adjustments/
- JQuery keep tooltip in viewport — https://stackoverflow.com/questions/24075503/jquery-keep-tooltip-in-viewport
- flexisel: Responsive carousel jQuery plugin — https://github.com/TYLERSELBY/flexisel
- TabaKordion - jQuery Accessible Tabs, Accordion and Show/Hide — https://cdn.jsdelivr.net/npm/tabakordion@1.0.0/
- jQuery-Accessible-RIA — https://openhub.net/p/jquery-accessible-ria
- WAI-ARIA Authoring Practices — https://www.w3.org/WAI/ARIA/apg/