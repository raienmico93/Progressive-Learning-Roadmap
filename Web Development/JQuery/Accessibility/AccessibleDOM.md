# Accessible DOM Manipulation with jQuery — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Accessible DOM Manipulation with jQuery is the practice of dynamically creating, modifying, and removing DOM elements while ensuring that the resulting interface remains fully usable by people who rely on assistive technologies such as screen readers, keyboard navigation, and voice control.

**Technical Definition:** Accessible DOM Manipulation encompasses the techniques and conventions for synchronizing visual UI state changes with the WAI-ARIA (Web Accessibility Initiative – Accessible Rich Internet Applications) state and property attributes that assistive technologies depend on. This includes preserving semantic HTML structure, generating appropriate accessible names through `aria-labelledby` and `aria-label`, programmatically toggling ARIA states (`aria-expanded`, `aria-checked`, `aria-selected`) during UI transitions, ensuring native HTML attributes (`disabled`, `required`) are kept in sync with their ARIA equivalents, and differentiating between hiding content from all users (`display: none`) versus hiding it only visually while keeping it available to screen readers (`.sr-only` / `.visually-hidden`).

**Beginner-Friendly Explanation:** When you use jQuery to change what is on the screen — showing a menu, hiding a panel, disabling a button — people who cannot see the screen need to know what changed. Screen readers announce things like “menu expanded” or “button disabled” based on special HTML attributes called ARIA attributes. Accessible DOM manipulation means making sure that whenever you change the visual state with jQuery, you also update the ARIA attributes so screen reader users get the same information. It also means using proper HTML tags instead of generic `<div>` elements, giving elements meaningful labels, and choosing the right hiding technique for the situation.

### Key Characteristics

- **Synchronization requirement:** Visual state changes must be paired with ARIA state updates; they must not drift apart. The W3C recommends binding UI changes directly to WAI-ARIA states and properties, such as through CSS attribute selectors or JavaScript class assignments .
- **Semantic-first approach:** Prefer native HTML elements (`<button>`, `<nav>`, `<dialog>`) over generic `<div>` elements; use `role` attributes only when no native element exists.
- **Unique label association:** Every interactive element must have an accessible name, typically via `aria-labelledby` (referencing a visible label's ID) or `aria-label` (providing a string directly).
- **Native and ARIA property pairing:** For form elements, native attributes (`disabled`, `required`) should be updated using `.prop()`, and any ARIA equivalents (`aria-disabled`, `aria-required`) should be synchronized.
- **Contextual hiding:** `display: none` removes content from both visual and assistive technology access; `.sr-only` / `.visually-hidden` hides content visually while keeping it available to screen readers .

### Prerequisites

- Proficiency in jQuery fundamentals: selectors, `.attr()`, `.prop()`, `.addClass()`, `.removeClass()`, and event handling.
- Understanding of HTML semantics and the WAI-ARIA specification.
- Familiarity with the WAI-ARIA Authoring Practices for common UI patterns (menus, tabs, accordions, dialogs).
- Awareness of screen reader behavior and keyboard navigation requirements.

### Related Programming Areas

- **Web Accessibility (a11y):** The broader discipline of making web content usable by everyone.
- **WAI-ARIA Specification:** The standard that defines ARIA roles, states, and properties.
- **Progressive Enhancement:** Ensuring basic functionality works without JavaScript or CSS.
- **Focus Management:** Moving and trapping keyboard focus during dynamic UI interactions.
- **Form Validation:** Communicating error and success states to assistive technologies.

### Core Concepts / Features

This cheat sheet covers six core concepts: preserving semantic HTML, appropriate labels, ARIA attributes, accessible state changes, native attribute overrides, and contextual hiding.

---

## Core Concept 1: Preserving Semantic HTML — Avoiding "Div-itis" by Dynamically Updating Native Tags or Using Explicit Role Modifications

### Definitions

**Core Definition:** Preserving semantic HTML is the practice of using native HTML elements that convey meaning (e.g., `<button>`, `<nav>`, `<dialog>`, `<ul>`, `<h1>`–`<h6>`) rather than generic `<div>` or `<span>` elements, and when dynamic manipulation requires creating or replacing elements, using the most semantically appropriate tag available.

**Technical Definition:** "Div-itis" is the colloquial term for the overuse of generic `<div>` elements to structure content, resulting in a DOM that lacks semantic meaning and provides no useful information to assistive technologies. When jQuery dynamically creates or replaces DOM elements, the developer should select the most semantically appropriate HTML element for the task: `<button>` for clickable actions, `<a>` for navigation, `<nav>` for navigation regions, `<dialog>` for modal dialogs, `<ul>`/`<li>` for lists, and heading elements for section titles. When no native element exists for the required behavior (e.g., a tablist or a tree view), an explicit `role` attribute (e.g., `role="tablist"`, `role="tree"`) should be added to the nearest appropriate container, and the element should be tested with assistive technologies to ensure the role is correctly communicated.

**Beginner-Friendly Explanation:** Using `<div>` for everything is like writing a book without chapters, headings, or paragraphs — it is just a wall of text. Screen readers cannot tell what a `<div>` is supposed to do. Using `<button>` tells the screen reader "this is a button you can click." Using `<nav>` says "this is a navigation area." When you create elements with jQuery, you should use the most specific HTML tag for the job. If no tag fits, you can add a `role` attribute to tell the screen reader what the element is supposed to be.

### Purposes

- To provide assistive technologies with accurate information about the purpose and behavior of each element.
- To ensure that keyboard navigation and screen reader announcements follow standard expectations.
- To reduce the need for excessive ARIA attributes when native semantics already convey the required meaning.
- To improve the maintainability and readability of dynamically generated DOM structures.
- To comply with WCAG 2.2 Success Criterion 4.1.2 (Name, Role, Value) and 1.3.1 (Info and Relationships).

### Syntax Rules and Structure

**Complete General Syntax (Semantic Element Creation):**
```javascript
// Correct: use a semantic button element
var $button = $("<button>", {
    type: "button",
    text: "Toggle menu",
    "aria-expanded": "false",
    "aria-controls": "menu-panel"
});

// Incorrect: using a div for a button
var $divButton = $("<div>", { text: "Toggle menu" });
```

**Complete General Syntax (Explicit Role for Non-Native Patterns):**
```javascript
// Tablist pattern: no native element exists
var $tablist = $("<div>", {
    role: "tablist",
    "aria-label": "Content sections"
});
```

| Component | Description |
|-----------|-------------|
| `$("<button>", { ... })` | Creates a semantic `<button>` element with attributes. |
| `role="tablist"` | Explicit ARIA role for a pattern with no native equivalent. |
| `type="button"` | Prevents the button from submitting a form. |
| `aria-expanded="false"` | Communicates the toggle state to assistive technologies. |

**Syntax Rules:**

- Prefer native HTML elements over ARIA roles whenever possible; the first rule of ARIA is "Don't use ARIA if you can use native HTML."
- When no native element exists, add the appropriate `role` attribute to the container element.
- Use `$("<tag>", { attributes })` syntax to create elements with attributes in a single step.
- Ensure that the `role` attribute is placed on the element that owns the behavior, not on a child or parent.
- Do not override native semantics with conflicting ARIA roles (e.g., do not add `role="button"` to a `<button>` element).

**Constraints and Limitations:**

- Adding an ARIA `role` to an element does not automatically make it keyboard-focusable or add keyboard behavior; these must be implemented manually.
- Some ARIA roles require specific child roles (e.g., `role="tablist"` should contain `role="tab"` elements).
- Using a `role` that conflicts with the element's native semantics is invalid and may be ignored by assistive technologies.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Creating a Semantic Button vs. a Div**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Semantic HTML Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div id="container"></div>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: Create a semantic button
      var $semanticButton = $("<button>", {
        type: "button",
        text: "Semantic Button",
        "aria-expanded": "false"
      }).appendTo("#container");

      // Step 2: Create a non-semantic div (for comparison)
      var $divButton = $("<div>", {
        text: "Div Button",
        role: "button",
        tabindex: "0"
      }).appendTo("#container");

      // Step 3: Check the tag names
      $("#log").html(
        "First element: " + $semanticButton.prop("tagName") + "<br>" +
        "Second element: " + $divButton.prop("tagName")
      );
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
First element: BUTTON
Second element: DIV
```

**Why this output:** The semantic button uses the `<button>` element, which screen readers automatically recognize as a clickable button. The div button requires `role="button"` and `tabindex="0"` to approximate the native semantics, but it lacks built-in keyboard behavior (Enter/Space activation) that the native button provides.

### Real-World Cases

- **Navigation menus:** Using `<nav>` and `<ul>`/`<li>` instead of nested `<div>` elements.
- **Modal dialogs:** Using `<dialog>` element where supported, or `role="dialog"` on a `<div>` with proper focus management.
- **Tabs:** Using `role="tablist"`, `role="tab"`, and `role="tabpanel"` since no native tabs element exists.
- **Accordions:** Using `<button>` elements inside `<h3>` headings for the accordion headers.

---

## Core Concept 2: Appropriate Labels — Injecting Unique IDs and Mapping Items Dynamically Using `.attr('aria-labelledby', id)`

### Definitions

**Core Definition:** Appropriate labels in accessible DOM manipulation refers to the practice of ensuring that every interactive element and meaningful content region has an accessible name, typically by generating unique IDs for label elements and programmatically associating them with their target elements using `aria-labelledby` or `aria-label`.

**Technical Definition:** An accessible name is the text that assistive technologies use to identify an element. For native form elements, the `<label for="id">` association provides the accessible name. For non-native widgets (e.g., a `<div role="tablist">`), the accessible name must be provided via `aria-labelledby` (which references the ID of a visible labeling element) or `aria-label` (which provides a string directly). When jQuery dynamically generates multiple instances of a widget, each instance must have a **unique** ID for its labeling element, because IDs must be unique within a document. The `.attr("aria-labelledby", id)` method is used to establish the association programmatically.

**Beginner-Friendly Explanation:** Every button, input, and interactive widget needs a name so screen readers can announce what it is. For example, a search input should be announced as “Search input, not just “input.” You can give an element a name by pointing to another element that contains the label text, using `aria-labelledby`. When you create multiple widgets with jQuery, you must make sure each label has a unique ID — like giving each student in a class a unique student number.

### Purposes

- To provide assistive technologies with a human-readable name for every interactive element.
- To associate dynamically generated labels with their target elements without relying on native `<label for="">` (which requires a static ID).
- To ensure that multiple instances of the same widget on a page do not conflict by using duplicate IDs.
- To support complex widgets (tabs, dialogs, trees) where the accessible name is derived from a visible heading or text element.
- To comply with WCAG 2.2 Success Criterion 4.1.2 (Name, Role, Value).

### Syntax Rules and Structure

**Complete General Syntax (Setting `aria-labelledby` with jQuery):**
```javascript
// Single element: set aria-labelledby to the ID of the label
$element.attr("aria-labelledby", "labelId");

// Multiple elements: use a function to compute the ID dynamically
$("figure").attr("aria-labelledby", function() {
    return $(this).find("figcaption").attr("id");
});
```

**Complete General Syntax (Generating Unique IDs):**
```javascript
var uniqueId = "widget-" + (++$.fn.widget.instanceCount);
$label.attr("id", uniqueId + "-label");
$target.attr("aria-labelledby", uniqueId + "-label");
```

| Component | Description |
|-----------|-------------|
| `.attr("aria-labelledby", id)` | Sets the ARIA attribute to reference the labeling element's ID. |
| `function() { return ...; }` | A function that computes the ID for each element in the set. |
| Unique ID generation | Ensures no two labeling elements share the same ID. |

**Syntax Rules:**

- IDs must be unique within the document; use a counter or a unique identifier when generating multiple instances.
- `aria-labelledby` can reference multiple IDs (space-separated) to concatenate label text from multiple elements.
- The referenced element must exist in the DOM and must contain text content.
- For native form elements, prefer `<label for="">` over `aria-labelledby`; use `aria-labelledby` only when native labeling is not possible.

**Constraints and Limitations:**

- `aria-labelledby` does not work with class selectors; only IDs are valid .
- If the referenced element is hidden with `display: none`, its text is still used for the accessible name in most screen readers.
- Dynamically generated IDs must be carefully managed to avoid collisions when multiple widgets are initialized on the same page.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Dynamically Associating a Figure with Its Caption**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>aria-labelledby Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <figure id="figure1">
    <img src="photo.jpg" alt="">
    <figcaption id="caption1">A beautiful sunset</figcaption>
  </figure>
  <figure id="figure2">
    <img src="photo2.jpg" alt="">
    <figcaption id="caption2">A mountain landscape</figcaption>
  </figure>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: For each figure, set aria-labelledby to its caption's ID
      $("figure").attr("aria-labelledby", function() {
        return $(this).find("figcaption").attr("id");
      });

      // Step 2: Verify the associations
      $("figure").each(function() {
        var $fig = $(this);
        $("#log").append(
          $fig.attr("id") + " labelled by: " + $fig.attr("aria-labelledby") + "<br>"
        );
      });
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
figure1 labelled by: caption1
figure2 labelled by: caption2
```

**Why this output:** The `.attr("aria-labelledby", function() { ... })` call iterates over each `<figure>` element, finds its child `<figcaption>`, and uses the caption's ID as the value of `aria-labelledby`. This establishes the association between the figure and its caption, so screen readers announce the figure's content with its caption as the accessible name .

### Real-World Cases

- **Modal dialogs:** Setting `aria-labelledby` to the ID of the dialog's title heading.
- **Form field groups:** Using `aria-labelledby` to associate a group of radio buttons with a fieldset legend when native `<fieldset>`/`<legend>` is not feasible.
- **Data tables:** Associating a table caption with the table using `aria-labelledby` when the caption is rendered outside the table.
- **Tabs:** Associating each `role="tabpanel"` with its corresponding `role="tab"` using `aria-labelledby`.

---

## Core Concept 3: ARIA Attributes — Toggling Element Properties Like `aria-expanded`, `aria-checked`, and `aria-controls` Programmatically

### Definitions

**Core Definition:** ARIA attribute toggling is the practice of programmatically updating WAI-ARIA state and property attributes (`aria-expanded`, `aria-checked`, `aria-selected`, `aria-controls`, `aria-hidden`, etc.) in response to user interactions and UI state changes, ensuring that assistive technologies receive accurate information about the current state of the interface.

**Technical Definition:** ARIA attributes are divided into roles, states, and properties. States are dynamic attributes that change in response to user interaction, including `aria-expanded` (whether a collapsible element is expanded), `aria-checked` (whether a checkbox or switch is checked), `aria-selected` (whether a tab or option is selected), and `aria-hidden` (whether an element is hidden from assistive technologies). Properties are less dynamic and include `aria-controls` (which identifies the element(s) controlled by the current element), `aria-labelledby`, and `aria-describedby`. These attributes are manipulated using jQuery's `.attr()` method, and their values must be synchronized with the visual UI state. The W3C recommends binding UI changes directly to WAI-ARIA states and properties, such as through CSS attribute selectors or JavaScript class assignments .

**Beginner-Friendly Explanation:** ARIA attributes are like the status lights on a machine. When you open a menu, the “expanded” light should turn on. When you check a box, the “checked” light should turn on. These lights tell screen readers what is happening. jQuery is the hand that flips the switches: `.attr("aria-expanded", "true")` turns on the “expanded” light.

### Purposes

- To communicate the current state of interactive widgets to assistive technologies.
- To synchronize visual UI changes (opening a menu, checking a box, selecting a tab) with programmatic state changes.
- To establish relationships between elements (e.g., a button that controls a panel) using `aria-controls`.
- To ensure that screen reader users receive the same feedback as sighted users.
- To comply with WCAG 2.2 Success Criterion 4.1.2 (Name, Role, Value).

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
// Toggle aria-expanded
$trigger.attr("aria-expanded", "true");  // expanded
$trigger.attr("aria-expanded", "false"); // collapsed

// Toggle aria-checked
$checkbox.attr("aria-checked", "true");
$checkbox.attr("aria-checked", "false");

// Set aria-controls (static relationship)
$trigger.attr("aria-controls", "panelId");

// Toggle aria-hidden
$panel.attr("aria-hidden", "true");
$panel.attr("aria-hidden", "false");
```

| Attribute | Purpose | Values |
|-----------|---------|--------|
| `aria-expanded` | Indicates whether a collapsible element is expanded | `"true"` / `"false"` |
| `aria-checked` | Indicates whether a checkbox or switch is checked | `"true"` / `"false"` / `"mixed"` |
| `aria-selected` | Indicates whether a tab or option is selected | `"true"` / `"false"` |
| `aria-controls` | Identifies the element(s) controlled by the current element | Space-separated IDs |
| `aria-hidden` | Indicates whether an element is hidden from assistive tech | `"true"` / `"false"` |

**Syntax Rules:**

- Use `.attr()` to set ARIA attributes; ARIA attributes are treated as attributes, not properties.
- Always use string values (`"true"` / `"false"`), not Booleans.
- Update ARIA attributes **before** or **during** the visual transition, not after.
- Use `aria-controls` on the controlling element (e.g., a button) to reference the ID of the controlled element (e.g., a panel).
- The `aria-hidden` attribute should be synchronized with CSS that hides the element (e.g., `[aria-hidden="true"] { display: none; }`) .

**Constraints and Limitations:**

- ARIA attributes do not change visual appearance; CSS must still handle the visual state.
- Setting `aria-hidden="true"` on an element that contains focusable elements can create accessibility issues; use `inert` or `tabindex="-1"` as well.
- The `aria-controls` attribute is not supported by all screen readers; it is best used as a supplementary relationship.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Toggling `aria-expanded` and `aria-hidden` in an Accordion**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>ARIA Toggle Demo</title>
  <style>
    .panel { display: none; padding: 10px; border: 1px solid #ccc; }
    .panel[aria-hidden="false"] { display: block; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <h3>
    <button type="button" aria-expanded="false" aria-controls="panel1">
      Section 1
    </button>
  </h3>
  <div id="panel1" class="panel" aria-hidden="true">
    Content for Section 1.
  </div>

  <script>
    $(function() {
      $("button[aria-controls]").click(function() {
        var $trigger = $(this);
        var $panel = $("#" + $trigger.attr("aria-controls"));
        var isExpanded = $trigger.attr("aria-expanded") === "true";

        // Step 1: Toggle aria-expanded on the trigger
        $trigger.attr("aria-expanded", isExpanded ? "false" : "true");

        // Step 2: Toggle aria-hidden on the panel
        $panel.attr("aria-hidden", isExpanded ? "true" : "false");
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Section 1" expands the panel (content becomes visible) and the button's `aria-expanded` changes from `"false"` to `"true"`. The panel's `aria-hidden` changes from `"true"` to `"false"`. Clicking again collapses the panel and reverts both attributes.

**Why this output:** The click handler reads the current `aria-expanded` value, toggles it, and then uses the same logic to toggle `aria-hidden` on the panel. The CSS rule `.panel[aria-hidden="false"] { display: block; }` ensures that the visual state follows the ARIA state, so screen readers and sighted users see the same thing.

### Real-World Cases

- **Accordions:** Toggling `aria-expanded` on headers and `aria-hidden` on panels.
- **Dropdown menus:** Toggling `aria-expanded` on triggers and `aria-hidden` on menu lists.
- **Checkboxes and switches:** Toggling `aria-checked` on custom-styled checkbox elements.
- **Tabs:** Toggling `aria-selected` on tabs and `aria-hidden` on tab panels.

---

## Core Concept 4: Accessible State Changes — Syncing Visual Cues Directly with Functional Attributes So Screen Readers Match CSS States

### Definitions

**Core Definition:** Accessible state changes are the practice of ensuring that whenever a visual state change occurs (a menu opens, a tab is selected, a form field becomes invalid), the corresponding functional attribute change (ARIA state, native property) occurs simultaneously, so that assistive technologies receive the same information that sighted users perceive.

**Technical Definition:** The W3C WAI-ARIA Authoring Practices explicitly state that developers "must set any additional WAI-ARIA states and properties on document elements" and "synchronize the visual UI with accessibility states and properties" . One recommended technique is to bind CSS styling directly to ARIA attribute values using CSS attribute selectors (e.g., `[aria-selected="true"] { background-color: #222; }`), so that the visual state is driven by the ARIA state rather than by a separate class . When CSS attribute selectors are not supported in older browsers, the alternative is to assign a class name based on the ARIA attribute value using JavaScript .

**Beginner-Friendly Explanation:** Imagine a traffic light where the red light turns on but the sign still says "Go." That is what happens when the visual state changes but the ARIA state does not. Accessible state changes mean keeping the light and the sign synchronized: when the menu opens visually, the ARIA attribute says "expanded." When the tab is selected visually, the ARIA attribute says "selected."

### Purposes

- To ensure that screen reader users receive the same state information as sighted users.
- To prevent the "silent change" problem where visual updates occur without corresponding accessibility updates.
- To provide a single source of truth for state (the ARIA attribute) that drives both visual styling and assistive technology announcements.
- To reduce code duplication by using CSS attribute selectors to style based on ARIA state.
- To comply with WCAG 2.2 Success Criterion 4.1.2 (Name, Role, Value) and 4.1.3 (Status Messages).

### Syntax Rules and Structure

**Complete General Syntax (CSS Attribute Selector Approach):**
```css
/* Style based on ARIA state */
[aria-selected="true"] {
    background-color: #007bff;
    color: #fff;
}
[aria-selected="false"] {
    background-color: #eee;
    color: #333;
}
```

```javascript
// Update ARIA attribute; CSS automatically reflects the change
$tab.attr("aria-selected", "true");
```

**Complete General Syntax (Class-Based Approach for Older Browsers):**
```javascript
function setSelectedTab($tab) {
    $tab.attr("aria-selected", "true").addClass("is-selected");
    $tab.siblings().attr("aria-selected", "false").removeClass("is-selected");
}
```

| Approach | Method | Browser Support |
|----------|--------|-----------------|
| CSS attribute selector | `[aria-selected="true"] { ... }` | Modern browsers; IE7+ |
| Class-based | `.addClass("is-selected")` | All browsers |

**Syntax Rules:**

- Update the ARIA attribute **and** the visual state in the same handler, ideally with the ARIA attribute driving the visual state via CSS.
- If using CSS attribute selectors, ensure that the ARIA attribute value is always set (e.g., `aria-selected="false"` on unselected tabs).
- If using the class-based approach, keep the class and the ARIA attribute in sync; never update one without the other.
- Test with a screen reader to verify that state changes are announced correctly.

**Constraints and Limitations:**

- CSS attribute selectors are not supported in IE6, though this is rarely a concern in modern development .
- Screen readers may not announce all ARIA state changes automatically; some require the element to be focused or the change to occur in a live region.
- Overusing ARIA state changes can lead to excessive announcements, which can be disruptive.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Syncing Visual and ARIA State in a Tab Widget**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Accessible State Change Demo</title>
  <style>
    .tabs { display: flex; list-style: none; padding: 0; }
    .tab { padding: 10px 20px; cursor: pointer; border: 1px solid #ccc; }
    .tab[aria-selected="true"] { background: #007bff; color: #fff; }
    .panel { display: none; padding: 15px; }
    .panel[aria-hidden="false"] { display: block; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="tabs" role="tablist">
    <div class="tab" role="tab" aria-selected="true" aria-controls="panel1" tabindex="0">Tab 1</div>
    <div class="tab" role="tab" aria-selected="false" aria-controls="panel2" tabindex="-1">Tab 2</div>
  </div>
  <div id="panel1" class="panel" role="tabpanel" aria-hidden="false">Content 1</div>
  <div id="panel2" class="panel" role="tabpanel" aria-hidden="true">Content 2</div>

  <script>
    $(function() {
      $(".tab").click(function() {
        var $this = $(this);
        var targetId = $this.attr("aria-controls");

        // Step 1: Update ARIA state on all tabs
        $(".tab").attr("aria-selected", "false").attr("tabindex", "-1");
        $this.attr("aria-selected", "true").attr("tabindex", "0");

        // Step 2: Update ARIA state on all panels
        $(".panel").attr("aria-hidden", "true");
        $("#" + targetId).attr("aria-hidden", "false");

        // Step 3: CSS automatically reflects the ARIA changes
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Tab 2" sets `aria-selected="false"` on Tab 1 and `aria-selected="true"` on Tab 2. The CSS rule `.tab[aria-selected="true"]` automatically highlights Tab 2. The panel visibility also follows: Panel 1 gets `aria-hidden="true"` and Panel 2 gets `aria-hidden="false"`, and the CSS rule `.panel[aria-hidden="false"]` shows Panel 2.

**Why this output:** The JavaScript only updates the ARIA attributes; the CSS attribute selectors handle the visual presentation. This guarantees that the visual state and the ARIA state are always in sync — they are the same thing.

### Real-World Cases

- **Tab widgets:** `aria-selected` drives both the visual highlight and the screen reader announcement.
- **Accordions:** `aria-expanded` drives the open/close styling and the screen reader announcement.
- **Custom checkboxes:** `aria-checked` drives the checked/unchecked visual and the screen reader announcement.
- **Toggle switches:** `aria-checked` on a `role="switch"` element drives the on/off visual state.

---

## Core Concept 5: Native Attribute Overrides — Ensuring Properties Like `required` and `disabled` Are Updated Alongside Their `aria-*` Equivalents

### Definitions

**Core Definition:** Native attribute overrides is the practice of using jQuery's `.prop()` method to update native DOM properties such as `disabled`, `required`, `checked`, and `selected`, and ensuring that any corresponding ARIA attributes (`aria-disabled`, `aria-required`) are synchronized when the native property alone is insufficient or when the element does not support the native property.

**Technical Definition:** Since jQuery 1.6, the `.attr()` and `.prop()` methods serve different purposes. `.attr()` manipulates HTML attributes (the values written in the markup), while `.prop()` manipulates DOM properties (the current state of the element as tracked by the browser). For boolean properties such as `disabled`, `required`, `checked`, and `selected`, `.prop()` is the correct method: `.prop("disabled", true)` sets the DOM property to `true`, which both prevents interaction and is reflected in the element's visual state . The HTML5 `required` attribute has the same accessibility effect as `aria-required="true"`; when a host language (HTML) declares a native attribute with the same implicit semantic as an ARIA attribute, user agents must use the native attribute and ignore the ARIA attribute . Therefore, for form elements, updating the native property is usually sufficient for accessibility.

**Beginner-Friendly Explanation:** There are two ways to tell a browser that a button is disabled: you can change the HTML attribute (`disabled="disabled"`) or the DOM property (`element.disabled = true`). The DOM property is the “live” state; the attribute is the initial state. jQuery's `.prop()` method changes the live state. For accessibility, changing the DOM property is usually enough because screen readers read the live state. But for custom elements that do not have a native `disabled` property, you need to add `aria-disabled="true"` to tell the screen reader.

### Purposes

- To correctly disable or enable form elements so that they cannot be interacted with and are announced as disabled by screen readers.
- To set or remove the `required` state on form fields so that screen readers announce them as required.
- To synchronize native properties with ARIA attributes when both are used (e.g., `disabled` and `aria-disabled`).
- To avoid the common pitfall of using `.attr()` for boolean properties, which does not always update the live state correctly.
- To ensure that dynamically toggled form states are both functionally and semantically correct.

### Syntax Rules and Structure

**Complete General Syntax (Native Property):**
```javascript
// Disable
$input.prop("disabled", true);

// Enable
$input.prop("disabled", false);

// Required
$input.prop("required", true);

// Checked
$checkbox.prop("checked", true);
```

**Complete General Syntax (ARIA Synchronization for Non-Native Elements):**
```javascript
// For a custom button (div with role="button")
$customButton.attr("aria-disabled", "true");

// For a native input, prop is sufficient
$input.prop("disabled", true);
```

| Native Property | ARIA Equivalent | When to Use Both |
|-----------------|-----------------|-------------------|
| `disabled` | `aria-disabled` | When using a non-form element (e.g., `<div role="button">`) |
| `required` | `aria-required` | When the element does not support the native `required` attribute |
| `checked` | `aria-checked` | When using a custom checkbox (e.g., `<div role="checkbox">`) |

**Syntax Rules:**

- Use `.prop()` for boolean properties (`disabled`, `required`, `checked`, `selected`); use `.attr()` for string attributes.
- For native form elements, `.prop("disabled", true)` is sufficient for accessibility; no additional `aria-disabled` is needed.
- For non-form elements that mimic form behavior, use `aria-disabled="true"` and implement JavaScript to prevent interaction.
- Do not use `.removeProp()` on native properties; use `.prop(property, false)` instead .

**Constraints and Limitations:**

- The `.attr("disabled", "disabled")` method is deprecated for setting the disabled state since jQuery 1.6; use `.prop()` instead.
- Disabled form elements are not submitted with the form and cannot receive focus.
- A disabled element with `aria-disabled="true"` still receives focus and can be activated by keyboard unless JavaScript prevents it.
- Using both `disabled` and `required` on the same element can cause problems because the user cannot input the required value .

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Toggling `disabled` with `.prop()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Native Attribute Override Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <input type="text" id="nameInput" placeholder="Enter your name">
  <button type="button" id="toggleBtn">Toggle Disabled</button>
  <p id="log"></p>

  <script>
    $(function() {
      $("#toggleBtn").click(function() {
        var $input = $("#nameInput");
        var isDisabled = $input.prop("disabled");

        // Step 1: Toggle the disabled property using .prop()
        $input.prop("disabled", !isDisabled);

        // Step 2: Log the current state
        $("#log").text(
          "Input disabled: " + $input.prop("disabled") +
          " | Attribute: " + $input.attr("disabled")
        );
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Initially, the input is enabled. Clicking "Toggle Disabled" disables the input (it becomes greyed out and unfocusable) and logs "Input disabled: true | Attribute: disabled". Clicking again re-enables it and logs "Input disabled: false | Attribute: undefined".

**Why this output:** `.prop("disabled", true)` sets the DOM property, which both prevents interaction and updates the visual state. The `.attr("disabled")` returns `"disabled"` when the property is `true` and `undefined` when it is `false`, demonstrating that the property and attribute are synchronized.

### Real-World Cases

- **Form validation:** Disabling the submit button until all required fields are filled.
- **Conditional fields:** Enabling a text input only when a checkbox is checked.
- **Read-only fields:** Using `prop("readonly", true)` for fields that can be selected but not edited.
- **Custom controls:** Adding `aria-disabled="true"` to non-form elements that should be announced as disabled.

---

## Core Concept 6: Contextual Hiding — Differentiating Between Hiding Elements with `.hide()` / `display: none` Versus Visually Hidden Screen-Reader Utility Classes

### Definitions

**Core Definition:** Contextual hiding is the practice of choosing the appropriate hiding technique based on whether the content should be hidden from all users (including screen reader users) or hidden only from sighted users while remaining available to assistive technologies.

**Technical Definition:** There are two fundamentally different ways to hide content, each with different accessibility implications. `display: none` (and jQuery's `.hide()`, which sets `display: none` inline) removes the element from the accessibility tree entirely — screen readers cannot perceive it, and it is not focusable. The `.sr-only` / `.visually-hidden` utility class (which uses `position: absolute`, `clip`, `width: 1px`, `height: 1px`, etc.) hides the element visually while keeping it in the accessibility tree and focusable. Bootstrap's `.hidden` class applies `display: none` (hiding from everyone), while `.sr-only` hides from visual users but remains available to screen readers . jQuery's `.slideUp()`, `.fadeOut()`, and `.hide()` all use `display: none` when complete, which means screen readers cannot read the hidden content . For content that must remain accessible to screen readers while being visually hidden, the `.sr-only` class or a manual off-screen positioning technique should be used.

**Beginner-Friendly Explanation:** Imagine you have a note that says “Important information.” If you throw it in the trash (`display: none`), nobody can see it — not sighted people, not screen reader users. If you put it in a drawer that only screen readers can open (`.sr-only`), sighted people cannot see it, but screen reader users can still read it. You need to choose the right method depending on whether the content is for everyone or just for screen reader users.

### Purposes

- To hide content from all users when it is not relevant to the current state (e.g., a closed accordion panel).
- To hide content only visually while keeping it available to screen readers (e.g., skip links, additional context for icon buttons).
- To avoid the common mistake of hiding content with `display: none` when it should remain accessible.
- To comply with WCAG 2.2 Success Criterion 1.3.1 (Info and Relationships) and 4.1.2 (Name, Role, Value).
- To ensure that dynamically toggled content (e.g., accordion panels, dropdown menus) is correctly announced or hidden by screen readers.

### Syntax Rules and Structure

**Complete General Syntax (Hide from Everyone):**
```javascript
// Using jQuery .hide() — sets display: none
$element.hide();

// Using .css() — sets display: none
$element.css("display", "none");

// Using a class — CSS: .hidden { display: none !important; }
$element.addClass("hidden");
```

**Complete General Syntax (Hide Visually, Keep for Screen Readers):**
```css
/* .sr-only / .visually-hidden */
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

```javascript
$element.addClass("sr-only");
```

| Technique | Visual Users | Screen Reader Users | Focusable |
|-----------|-------------|---------------------|-----------|
| `display: none` | Hidden | Hidden | No |
| `.sr-only` / `.visually-hidden` | Hidden | Accessible | Yes (unless `tabindex="-1"`) |
| `visibility: hidden` | Hidden | Hidden | No |
| `opacity: 0` | Hidden | Accessible (may be) | Yes |

**Syntax Rules:**

- Use `display: none` (or `.hide()`) when the content should be completely removed from both visual and assistive technology access.
- Use `.sr-only` / `.visually-hidden` when the content is needed by screen reader users but should not be visible on screen (e.g., additional context for icon buttons, skip links).
- When using `.slideUp()` or `.fadeOut()`, be aware that the element ends with `display: none` and becomes inaccessible to screen readers .
- For content that should be hidden visually but revealed on focus (e.g., skip links), combine `.sr-only` with `.sr-only-focusable` .

**Constraints and Limitations:**

- `.sr-only` content is still focusable by default; if it should not be focusable, add `tabindex="-1"`.
- Some screen readers may still announce `.sr-only` content in unexpected ways; test with actual screen readers.
- Using `visibility: hidden` hides from both visual users and screen readers, but unlike `display: none`, the element still occupies layout space.
- jQuery's `.show()` and `.hide()` set inline `display` styles; if a CSS class uses `display: none !important`, jQuery's methods will not override it .

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Choosing the Right Hiding Technique**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Contextual Hiding Demo</title>
  <style>
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
    .icon-btn { font-size: 24px; cursor: pointer; }
  </style>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button class="icon-btn" aria-label="Delete item">
    🗑️
  </button>

  <button class="icon-btn" id="contextBtn">
    ❌
    <span class="sr-only">Close dialog</span>
  </button>

  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: Demonstrate display: none (hidden from everyone)
      var $temp = $("<div>").text("This is hidden from everyone.").appendTo("body");
      $temp.hide(); // display: none

      // Step 2: Demonstrate .sr-only (hidden visually, available to screen readers)
      // The #contextBtn already has an .sr-only span as a child

      // Step 3: Log the accessibility implications
      $("#log").html(
        "Temp div display: " + $temp.css("display") + "<br>" +
        "Close button text (visible): " + $("#contextBtn").text().trim() + "<br>" +
        "Close button accessible name: " + $("#contextBtn").attr("aria-label") || "Derived from .sr-only span"
      );
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
Temp div display: none
Close button text (visible): ❌
Close button accessible name: Derived from .sr-only span
```

**Why this output:** The temporary `<div>` is hidden with `.hide()`, which sets `display: none`, removing it from both visual and screen reader access. The close button's visible text is just the ❌ emoji, but the `.sr-only` span provides the accessible name "Close dialog" for screen reader users.

### Real-World Cases

- **Skip links:** Using `.sr-only` so the “Skip to main content” link is hidden visually but available to screen readers.
- **Icon buttons:** Adding `.sr-only` text inside an icon button to provide an accessible name.
- **Accordion panels:** Using `display: none` (via `.slideUp()`) to hide closed panels from everyone.
- **Form help text:** Using `.sr-only` to provide additional context that is not needed visually but is helpful for screen reader users.
- **Modal dialogs:** Using `display: none` to hide the modal when closed, and `aria-hidden="true"` on the background content.

---

## References

- WAI-ARIA Authoring Practices 1.1 — https://www.w3.org/TR/2015/WD-wai-aria-practices-1.1-20151119/
- WAI-ARIA Specification — https://www.w3.org/TR/wai-aria-1.1/
- jQuery .attr() — https://api.jquery.com/attr/
- jQuery .prop() — https://api.jquery.com/prop/
- jQuery .hide() — https://api.jquery.com/hide/
- CSS-Tricks — “Places it’s tempting to use display: none; but don’t” — https://css-tricks.com/places-its-tempting-to-use-display-none-but-dont/
- Bootstrap Issue #10446 — “Resolve .hide/.hidden/.sr-only redundancy/confusion” — https://github.com/twbs/bootstrap/issues/10446
- Stack Overflow — “Get child ID and add to parent as ARIA attribute” — https://stackoverflow.com/questions/28232491/get-child-id-and-add-to-parent-as-aria-attribute
- Stack Overflow — “Disable/enable an input with jQuery” — https://stackoverflow.com/questions/1414365/disable-enable-an-input-with-jquery
- W3C WAI-ARIA Authoring Practices — https://www.w3.org/WAI/ARIA/apg/
- MDN Web Docs — ARIA — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA
- WebAIM — Introduction to ARIA — https://webaim.org/techniques/aria/
- The A11Y Project — How to Hide Content Responsibly — https://www.a11yproject.com/posts/how-to-hide-content/