# jQuery UI Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery UI is a curated library of user-interface interactions, effects, widgets, and themes built on top of the jQuery JavaScript Library, designed to help developers build highly interactive web applications with consistent, accessible, and themeable components.

**Technical Definition**
jQuery UI extends jQuery core with four categories of functionality: **Interactions** (Draggable, Droppable, Resizable, Selectable, Sortable) that add mouse-based behaviour to elements; **Widgets** (Accordion, Autocomplete, Button, Datepicker, Dialog, Menu, Progressbar, Slider, Spinner, Tabs, Tooltip) that provide ready-made UI controls; **Effects** (Blind, Bounce, Clip, Drop, Explode, Fade, Fold, Highlight, Puff, Pulsate, Scale, Shake, Size, Slide, Transfer) that extend `.show()`, `.hide()`, `.toggle()`, and `.animate()` with predefined animations; and **Utilities** (Widget Factory and Position) that support stateful plugin creation and element positioning. All widgets follow a consistent API pattern: options are set during initialisation or via `"option"`, methods are called via `$(selector).widgetName("methodName", parameters)`, and events are handled through option callbacks (e.g., `onSelect`) or event binding (e.g., `"datepickerselect"`).

**Beginner-Friendly Explanation**
jQuery UI is a toolbox that sits on top of jQuery. jQuery gives you the ability to find and change elements on a page; jQuery UI gives you ready-made pieces — date pickers, dialogs, tabs, drag-and-drop, fancy show/hide animations — that you can drop into your page with a single function call. It also comes with a theme system so all those pieces can look consistent with your site. You include jQuery, jQuery UI's CSS, and jQuery UI's JavaScript, and then you can turn a plain `<input>` into a calendar or a plain `<div>` into a draggable box.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Built on jQuery** | jQuery UI requires jQuery core and extends its API; it is not a standalone framework. |
| **Four pillars** | Interactions, Widgets, Effects, and Utilities cover behaviour, controls, animation, and infrastructure respectively. |
| **Consistent widget API** | Every widget supports options, methods, and events through the same `$(selector).widgetName(...)` pattern. |
| **Themeable via ThemeRoller** | A web app generates custom CSS themes and a full CSS framework for building ThemeRoller-ready components. |
| **Progressive enhancement** | Widgets enhance existing HTML rather than requiring a completely different markup model. |
| **Legacy relevance** | jQuery UI remains useful for quick enhancements and legacy projects, though modern frameworks offer alternatives for new development. |

---

### Prerequisites

- Working knowledge of jQuery selection, DOM manipulation, events, and AJAX.
- Basic familiarity with HTML forms, CSS, and the browser console.
- Inclusion of jQuery core before jQuery UI.
- Awareness that jQuery UI is a legacy library and may not be the best choice for greenfield projects.

---

### Related Programming Areas

- **jQuery core** — selection, DOM manipulation, events, and AJAX are prerequisites.
- **CSS and theming** — ThemeRoller generates CSS themes that control every widget's appearance.
- **Accessibility** — jQuery UI widgets include ARIA attributes and keyboard support.
- **Modern frameworks** — React, Vue, Angular, and Svelte offer alternatives; jQuery UI is often used in legacy contexts.
- **Performance optimisation** — selector caching, component modularity, and debouncing apply to jQuery UI usage.

---

### Core Concepts / Features (Index)

1. jQuery UI Overview and Installation
2. jQuery UI Themes
3. jQuery UI Widgets
4. jQuery UI Interactions
5. jQuery UI Effects
6. jQuery UI Utilities
7. jQuery UI Best Practices


## Core Concept 1: jQuery UI Overview and Installation

### Definitions

**Core Definition**
jQuery UI is a library that extends jQuery with interactions, widgets, effects, and utilities, and it can be installed via CDN, npm, Bower, or a custom Download Builder build that includes only the components an application needs.

**Technical Definition**
jQuery UI's architecture is modular: the **Core** file is a prerequisite for widgets and effects; the **Widget Factory** (`jquery.ui.widget.js`) provides the stateful plugin abstraction; **interactions** add mouse behaviour; **widgets** provide UI controls; **effects** extend jQuery's animation methods; and **utilities** such as Position support placement calculations. Installation methods include: **CDN** (simplest, includes jQuery, jQuery UI CSS, and jQuery UI JavaScript), **npm** (`npm install jquery-ui`), **Bower** (`bower install jquery-ui`), and **Download Builder** (custom build with only the components needed). When installed via npm, the package's main entry point is `ui/widget.js` and the jQuery dependency is `>=1.12.0 <5.0.0`.

**Beginner-Friendly Explanation**
To use jQuery UI, you need three things: jQuery, jQuery UI's CSS file (which provides the default theme), and jQuery UI's JavaScript file. The easiest way is to link to them from a CDN. If you are using a build tool, you can install jQuery UI with npm. If you want to keep your page small, you can use the Download Builder to select only the widgets and interactions you actually use, rather than loading the entire library.

---

### Purposes

- **To add ready-made UI components** (date picker, dialog, tabs, slider) without building them from scratch.
- **To add mouse interactions** (drag, drop, resize, sort, select) to existing elements.
- **To add advanced effects** (bounce, explode, fold, puff) to show/hide/toggle operations.
- **To create custom stateful widgets** using the Widget Factory.
- **To position elements precisely** relative to the window, document, or another element.

---

### Syntax Rules and Structure

#### CDN Installation

```html
<!-- jQuery core -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>

<!-- jQuery UI CSS (base theme) -->
<link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">

<!-- jQuery UI JavaScript -->
<script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
```

**Component Breakdown**

| Component | Description |
|---|---|
| jQuery core | Required dependency; must be loaded before jQuery UI. |
| jQuery UI CSS | Provides the base theme styles for all widgets. |
| jQuery UI JavaScript | Provides interactions, widgets, effects, and utilities. |

#### npm Installation

```bash
npm install jquery jquery-ui
```

#### Basic Widget Initialisation

```javascript
$(function() {
  $("#datepicker").datepicker();
});
```

---

### Constraints and Limitations

- **jQuery UI is a legacy library**; modern frameworks may offer better long-term support for new projects.
- **Loading the full library is unnecessary** if only a few components are needed; use Download Builder or modular npm imports.
- **The Widget Factory and Position utility are separate files** in the source; ensure they are included if used directly.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <script>
    $(function() {
      // Step 1: Enhance a text input into a date picker
      $("#datepicker").datepicker({
        dateFormat: "yy-mm-dd",
        showButtonPanel: true
      });
    });
  </script>
</head>
<body>
  <p>Date: <input type="text" id="datepicker"></p>
</body>
</html>
```

**Expected Output**
Clicking the input opens a calendar popup. Selecting a date fills the input with the chosen date in `yyyy-mm-dd` format.

**Why this output occurs**: `datepicker()` initialises the Datepicker widget on the input, replacing it with a calendar-triggering control. The `dateFormat` option controls the output format.

---

### Real-World Cases

- **Legacy admin panels**: Adding a date picker or dialog to an existing jQuery-based admin interface.
- **Prototyping**: Quickly adding drag-and-drop or sortable lists without a full framework.
- **Internal tools**: Using jQuery UI's widgets for simple, consistent UI controls.

---

## Core Concept 2: jQuery UI Themes

### Definitions

**Core Definition**
jQuery UI themes are CSS stylesheets that control the appearance of all widgets and interactions, generated through **ThemeRoller** — a web application that allows designers to configure fonts, colours, corner radii, backgrounds, and icon sprites, and download a complete theme package.

**Technical Definition**
ThemeRoller generates a customised version of the **jQuery UI CSS Framework**, which provides structural classes for common UI design situations, including states (default, hover, active, disabled), icons, and helper classes. The generated theme includes `jquery-ui.theme.css` (the full theme) and an `images` directory containing background images and icon sprites. ThemeRoller's interface is organised into panels for global font and corner radius settings, widget container styles, interaction states, and overlay/shadow styles. The **Theme Gallery** provides pre-designed themes that can be downloaded directly or further tweaked in the **"Roll Your Own"** tab. To install a downloaded theme, copy `jquery-ui.css` and the `images` folder into the project and link to the CSS file.

**Beginner-Friendly Explanation**
A jQuery UI theme is a skin. All jQuery UI widgets share a common set of CSS classes, so a single theme can make the date picker, dialog, tabs, and buttons all look consistent. ThemeRoller is a website where you can design your own skin by changing colours, fonts, and corner radii, and then download it. You can also pick a pre-made theme from the gallery. Once you have the theme, you just include its CSS file instead of the default one, and all your widgets adopt the new look.

---

### Purposes

- **To make all jQuery UI widgets visually consistent** with the application's design.
- **To generate custom themes** without writing CSS from scratch.
- **To use pre-designed themes** from the Theme Gallery for rapid prototyping.
- **To build ThemeRoller-ready custom components** using the jQuery UI CSS Framework classes.
- **To control states** (hover, active, disabled) across all widgets through a single theme.

---

### Syntax Rules and Structure

#### Including a ThemeRoller Theme

```html
<!-- Replace the default base theme with your downloaded theme -->
<link rel="stylesheet" href="css/jquery-ui.css">
```

#### ThemeRoller Workflow

1. Open `http://jqueryui.com/themeroller/`.
2. Use the **"Roll Your Own"** tab to adjust settings, or the **Theme Gallery** tab to pick a pre-made theme.
3. Click **"Download theme"** to go to the Download Builder.
4. Select the components to include and download the zip.
5. Copy `jquery-ui.css` and the `images` folder into your project.

---

### Constraints and Limitations

- **ThemeRoller themes are generated for a specific jQuery UI version**; verify compatibility when upgrading jQuery UI.
- **The generated CSS includes all widget classes**, even if you only use a few; the Download Builder can exclude unused components.
- **Custom components must follow the jQuery UI CSS Framework conventions** to inherit theme styles correctly.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <!-- Step 1: Link the downloaded ThemeRoller theme -->
  <link rel="stylesheet" href="css/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <script>
    $(function() {
      // Step 2: Initialise a widget; it automatically uses the theme
      $("#dialog").dialog();
      $("#tabs").tabs();
    });
  </script>
</head>
<body>
  <div id="dialog" title="Themed Dialog">Hello</div>
  <div id="tabs">
    <ul><li><a href="#tab1">Tab 1</a></li></ul>
    <div id="tab1">Content</div>
  </div>
</body>
</html>
```

**Expected Output**
The dialog and tabs render with the downloaded theme's colours, fonts, and corner radii.

**Why this output occurs**: jQuery UI widgets apply the CSS Framework classes, which are styled by the ThemeRoller-generated CSS.

---

### Real-World Cases

- **Corporate applications**: A theme matching the company brand is applied to all jQuery UI widgets.
- **Multi-tenant dashboards**: Different themes are served to different tenants via separate CSS files.
- **Rapid prototyping**: The Theme Gallery provides a polished look without design effort.

---

## Core Concept 3: jQuery UI Widgets

### Definitions

**Core Definition**
jQuery UI widgets are ready-made UI controls — Accordion, Autocomplete, Button, Datepicker, Dialog, Menu, Progressbar, Slider, Spinner, Tabs, and Tooltip — that enhance existing HTML elements with rich behaviour, accessibility, and themeable styling.

**Technical Definition**
All widgets follow a **consistent API pattern**: options are set during initialisation (`$("#datepicker").datepicker({ dateFormat: "yy-mm-dd" })`) or modified later (`$("#datepicker").datepicker("option", "dateFormat", "yy-mm-dd")`); methods are called via `$(selector).widgetName("methodName", parameters)`; and events are handled through option callbacks (`onSelect`) or event binding (`"datepickerselect"`). Widgets are built on the **Widget Factory**, which provides a stateful plugin abstraction with lifecycle hooks (`_create`, `_init`, `_destroy`), option management, and event triggering. Common widgets include Accordion (collapsible panels), Autocomplete (type-ahead suggestions), Datepicker (popup or inline calendar), Dialog (modal overlay), Tabs (multi-panel content area), and Tooltip (customisable hover hints).

**Beginner-Friendly Explanation**
A widget is a ready-made control that you attach to an existing HTML element. For example, you take a plain `<input>` and call `.datepicker()` on it, and it becomes a calendar. You take a `<div>` and call `.dialog()` on it, and it becomes a popup window. Every widget works the same way: you pass options when you create it, you can change options later, you call methods to make it do things, and you listen for events to know when the user interacts with it.

---

### Purposes

- **To add complex UI controls quickly** without building them from scratch.
- **To enhance existing HTML progressively** — the markup remains meaningful without JavaScript.
- **To provide accessible, keyboard-navigable controls** with ARIA attributes.
- **To maintain a consistent look and feel** across all controls through theming.
- **To reduce development time** for common patterns such as date selection, dialogs, and tabs.

---

### Syntax Rules and Structure

#### Widget Initialisation with Options

```javascript
$("#datepicker").datepicker({
  dateFormat: "yy-mm-dd",
  showButtonPanel: true
});
```

#### Changing Options After Initialisation

```javascript
$("#datepicker").datepicker("option", "dateFormat", "dd/mm/yy");
```

#### Calling Widget Methods

```javascript
$("#datepicker").datepicker("setDate", "2026-11-16");
$("#datepicker").datepicker("disable");
```

#### Handling Widget Events

```javascript
// Via option callback
$("#datepicker").datepicker({
  onSelect: function(dateText) {
    console.log("Selected:", dateText);
  }
});

// Via event binding
$("#datepicker").on("datepickerselect", function(event, ui) {
  console.log("Selected:", ui.date);
});
```

---

### Constraints and Limitations

- **Widgets may conflict with modern frameworks** that also manage the DOM; use wrapper components or directives.
- **Widgets rely on jQuery UI's CSS**; without the theme, they render unstyled.
- **Some widgets (e.g., Datepicker) have many options**; consult the API documentation for the full list.

---

### Annotated Code Example (Tabs Widget)

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <script>
    $(function() {
      // Step 1: Initialise the Tabs widget on the container
      $("#tabs").tabs({
        // Step 2: Set the initially active tab (zero-based index)
        active: 1
      });
    });
  </script>
</head>
<body>
  <div id="tabs">
    <ul>
      <li><a href="#tab-1">Tab One</a></li>
      <li><a href="#tab-2">Tab Two</a></li>
      <li><a href="#tab-3">Tab Three</a></li>
    </ul>
    <div id="tab-1"><p>Content for Tab One</p></div>
    <div id="tab-2"><p>Content for Tab Two</p></div>
    <div id="tab-3"><p>Content for Tab Three</p></div>
  </div>
</body>
</html>
```

**Expected Output**
The container renders as a tabbed interface with three tabs. "Tab Two" is active on load, and clicking each tab shows its corresponding content panel.

**Why this output occurs**: The `tabs()` widget reads the `<ul>` for tab labels and the sibling `<div>` elements for content, hides all but the active panel, and adds click handlers and ARIA attributes.

---

### Real-World Cases

- **Forms**: Datepicker for date fields, Autocomplete for search fields, Spinner for numeric inputs.
- **Admin interfaces**: Dialog for confirmation prompts, Tabs for organising settings, Accordion for collapsible sections.
- **Data entry**: Progressbar for uploads, Slider for range selection.

---

## Core Concept 4: jQuery UI Interactions

### Definitions

**Core Definition**
jQuery UI interactions are five mouse-driven behaviours — **Draggable**, **Droppable**, **Resizable**, **Selectable**, and **Sortable** — that add direct manipulation to any element, allowing users to move, resize, select, and reorder items with the mouse.

**Technical Definition**
Each interaction is a stateful plugin built on the Widget Factory. **Draggable** allows an element to be moved with the mouse; **Droppable** creates targets that respond when a draggable element is dropped on them; **Resizable** adds resize handles to an element; **Selectable** allows elements to be selected individually or in a group (like file icons); and **Sortable** reorders elements in a list or grid by dragging. Interactions expose options (e.g., `containment`, `connectWith`, `axis`), methods (e.g., `disable`, `enable`), and events (e.g., `drag`, `drop`, `resize`, `stop`). They are commonly combined — for example, Draggable and Droppable together implement drag-and-drop interfaces, and Sortable with `connectWith` implements moving items between lists.

**Beginner-Friendly Explanation**
Interactions are what make a page feel like a desktop app. You can pick up a box and move it (Draggable), drop it onto a target (Droppable), drag the corner to make it bigger (Resizable), click and drag to select multiple items (Selectable), or drag items to reorder them (Sortable). These are applied to existing HTML elements with a single function call, and you can configure them with options and listen for events to react to what the user does.

---

### Purposes

- **To add direct manipulation** (drag, drop, resize, sort, select) without custom mouse-event code.
- **To build drag-and-drop interfaces** by combining Draggable and Droppable.
- **To implement sortable lists and grids** for reordering content.
- **To allow users to resize panels or windows** with resize handles.
- **To enable multi-select behaviour** for groups of elements.

---

### Syntax Rules and Structure

#### Draggable

```javascript
$("#draggable").draggable({
  containment: "parent",
  axis: "x"
});
```

#### Droppable

```javascript
$("#droppable").droppable({
  accept: ".draggable",
  drop: function(event, ui) {
    console.log("Dropped");
  }
});
```

#### Resizable

```javascript
$("#resizable").resizable({
  handles: "se",
  minWidth: 100,
  minHeight: 100
});
```

#### Selectable

```javascript
$("#selectable").selectable();
```

#### Sortable

```javascript
$("#sortable").sortable({
  placeholder: "ui-state-highlight",
  connectWith: "#other-list"
});
```

---

### Constraints and Limitations

- **Interactions are mouse-oriented**; touch support requires additional handling or modern alternatives.
- **Performance can degrade with many draggable/sortable elements**; limit the number of interactive elements.
- **`connectWith` performance can be improved** by setting it after initialisation in some jQuery UI versions.

---

### Annotated Code Example (Drag and Drop)

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <style>
    #draggable { width: 100px; height: 100px; background: #4a90d9; color: white; padding: 10px; }
    #droppable { width: 200px; height: 200px; background: #eee; border: 2px dashed #999; }
  </style>
  <script>
    $(function() {
      // Step 1: Make the blue box draggable
      $("#draggable").draggable();

      // Step 2: Make the grey box a drop target
      $("#droppable").droppable({
        drop: function(event, ui) {
          $(this).addClass("ui-state-highlight");
          console.log("Dropped on target");
        }
      });
    });
  </script>
</head>
<body>
  <div id="draggable">Drag me</div>
  <div id="droppable">Drop here</div>
</body>
</html>
```

**Expected Output**
The blue box can be dragged with the mouse. When it is released over the grey box, the grey box highlights and the console logs “Dropped on target”.

**Why this output occurs**: `draggable()` enables mouse-based movement. `droppable()` defines a target that accepts draggable elements and fires the `drop` event when one is released over it.

---

### Real-World Cases

- **Kanban boards**: Draggable cards and droppable columns.
- **File managers**: Selectable file icons and sortable file lists.
- **Dashboard widgets**: Draggable and resizable panels.
- **Form builders**: Sortable list of form fields.

---

## Core Concept 5: jQuery UI Effects

### Definitions

**Core Definition**
jQuery UI effects are predefined animations — Blind, Bounce, Clip, Drop, Explode, Fade, Fold, Highlight, Puff, Pulsate, Scale, Shake, Size, Slide, and Transfer — that extend jQuery's `.show()`, `.hide()`, `.toggle()`, and `.effect()` methods with richer visual transitions.

**Technical Definition**
jQuery UI extends the core jQuery animation engine by adding an `effect` parameter to `.show()`, `.hide()`, and `.toggle()`. For example, `.show("bounce", { times: 3 }, 300)` bounces an element as it appears. The `.effect()` method applies an effect without hiding or showing the element. jQuery UI also extends `.addClass()`, `.removeClass()`, `.toggleClass()`, and `.switchClass()` to animate style changes. The `$.effects.define()` utility allows developers to define custom effects that work with `.effect()`, `.show()`, `.hide()`, and `.toggle()`. Effects are part of the jQuery UI Effects module, which is separate from the core and must be included in custom builds.

**Beginner-Friendly Explanation**
jQuery already has `.fadeIn()` and `.slideUp()`. jQuery UI adds more dramatic effects: you can make an element bounce into view, explode into pieces, fold away, puff out, or shake to get attention. You use them by passing the effect name to `.show()`, `.hide()`, or `.toggle()`, or by calling `.effect()` directly. For example, instead of `.show()`, you write `.show("bounce")` and the element bounces as it appears.

---

### Purposes

- **To add visual interest** to show/hide/toggle transitions.
- **To draw attention** with effects like `highlight`, `shake`, or `pulsate`.
- **To create dramatic entrances and exits** with `explode`, `fold`, `puff`, or `scale`.
- **To animate class changes** with `addClass`, `removeClass`, `toggleClass`, and `switchClass`.
- **To define custom effects** using `$.effects.define()`.

---

### Syntax Rules and Structure

#### Using an Effect with `.show()` / `.hide()` / `.toggle()`

```javascript
$("#element").show("bounce", { times: 3 }, 300);
$("#element").hide("explode", 500);
$("#element").toggle("fold", 400);
```

#### Using `.effect()` Directly

```javascript
$("#element").effect("highlight", { color: "#ff0" }, 1000);
```

#### Defining a Custom Effect

```javascript
$.effects.define("myEffect", "toggle", function(options, done) {
  var show = options.mode === "show";
  $(this).animate({ opacity: show ? 1 : 0 }, 300, done);
});
```

---

### Constraints and Limitations

- **Effects are CPU-intensive** compared to CSS transitions; avoid using many simultaneous effects.
- **Effects require the jQuery UI Effects module**; ensure it is included in custom builds.
- **Custom effects must call the `done` callback** to signal completion.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <script>
    $(function() {
      // Step 1: Toggle the box with a bounce effect
      $("#toggleBtn").on("click", function() {
        $("#box").toggle("bounce", { times: 2 }, 400);
      });

      // Step 2: Highlight the box on hover
      $("#box").on("mouseenter", function() {
        $(this).effect("highlight", {}, 800);
      });
    });
  </script>
</head>
<body>
  <button id="toggleBtn">Toggle Box</button>
  <div id="box" style="width:100px;height:100px;background:#4a90d9;"></div>
</body>
</html>
```

**Expected Output**
Clicking the button bounces the box as it hides or shows. Hovering over the box briefly highlights its background colour.

**Why this output occurs**: The `toggle` method with the `"bounce"` effect uses jQuery UI's Bounce effect to animate the element's visibility. The `effect("highlight", …)` call animates the background colour to draw attention.

---

### Real-World Cases

- **Notifications**: `highlight` or `pulsate` to draw attention to a new message.
- **Modal exits**: `explode` or `fold` for dramatic dismissal.
- **Form validation**: `shake` to indicate an invalid field.
- **Onboarding tours**: `bounce` or `puff` to spotlight elements.

---

## Core Concept 6: jQuery UI Utilities

### Definitions

**Core Definition**
jQuery UI utilities are two infrastructure tools — the **Widget Factory** and the **Position** utility — that support the creation of stateful jQuery plugins and the precise placement of elements relative to the window, document, or another element.

**Technical Definition**
The **Widget Factory** (`jquery.ui.widget.js`) provides a stateful plugin abstraction used by all jQuery UI widgets. It handles lifecycle hooks (`_create`, `_init`, `_destroy`), option management, event triggering, and the `classes` option for managing CSS class names. The `jQuery.widget.bridge()` method acts as the middleman between the object created by `$.widget()` and the jQuery API. The **Position** utility (`position()`) positions an element relative to the window, document, a particular element, or the cursor/mouse, with support for collision detection and flipping.

**Beginner-Friendly Explanation**
The Widget Factory is a toolkit for building your own jQuery UI–style plugins. If you want to create a reusable component that has options, methods, and events like the built-in widgets, you use the Widget Factory instead of writing all that plumbing yourself. The Position utility is a helper for placing one element relative to another — for example, showing a tooltip next to a button, or centring a dialog in the window. Both are the “behind the scenes” tools that make the visible widgets possible.

---

### Purposes

- **To create custom stateful widgets** with the same API as jQuery UI widgets.
- **To manage widget options, methods, and events** consistently.
- **To position elements precisely** relative to other elements or the viewport.
- **To handle collision detection** when positioning would push an element off-screen.
- **To extend existing widgets** with additional functionality.

---

### Syntax Rules and Structure

#### Widget Factory

```javascript
$.widget("custom.myWidget", {
  options: {
    color: "blue"
  },

  _create: function() {
    this.element.addClass("my-widget");
  },

  _destroy: function() {
    this.element.removeClass("my-widget");
  },

  setColor: function(color) {
    this.options.color = color;
    this._refresh();
  },

  _refresh: function() {
    this.element.css("color", this.options.color);
  }
});
```

#### Position Utility

```javascript
$("#tooltip").position({
  my: "left top",
  at: "right bottom",
  of: "#button",
  collision: "flip"
});
```

---

### Constraints and Limitations

- **The Widget Factory requires the jQuery UI Core**; it is included in the full jQuery UI build.
- **Position calculations may be affected by scroll position and CSS transforms**; test in context.
- **Custom widgets must follow the naming convention** (`namespace.widgetName`) to avoid collisions.

---

### Annotated Code Example (Custom Widget)

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <script>
    // Step 1: Define a custom widget using the Widget Factory
    $.widget("custom.greeter", {
      options: {
        name: "World"
      },

      _create: function() {
        this._render();
      },

      _render: function() {
        this.element.text("Hello, " + this.options.name + "!");
      },

      name: function(newName) {
        this.options.name = newName;
        this._render();
      }
    });

    $(function() {
      // Step 2: Initialise the custom widget
      $("#greeting").greeter({ name: "jQuery UI" });

      // Step 3: Call a custom method
      $("#greeting").greeter("name", "Developer");
    });
  </script>
</head>
<body>
  <div id="greeting"></div>
</body>
</html>
```

**Expected Output**
The `#greeting` div displays “Hello, Developer!” after the custom `name` method is called.

**Why this output occurs**: The Widget Factory creates a `greeter` plugin with options and methods. The `_create` hook renders the initial text, and the `name` method updates the option and re-renders.

---

### Real-World Cases

- **Plugin development**: Building reusable components that follow jQuery UI conventions.
- **Tooltips and popovers**: Using the Position utility to place elements next to their triggers.
- **Custom form controls**: Creating a stateful widget that wraps a native input with enhanced behaviour.

---

## Core Concept 7: jQuery UI Best Practices

### Definitions

**Core Definition**
jQuery UI best practices are the consolidated conventions — load only the components you need, use the Download Builder or modular imports, cache selections, prefer CSS for styling, namespace widgets, and test accessibility — that keep jQuery UI usage performant, maintainable, and compatible with modern development.

**Technical Definition**
Best practices synthesise the individual techniques: **modular loading** (Download Builder or npm tree-shaking to include only used components), **selector optimisation** (cache selections, avoid jQuery extensions where possible), **CSS over JavaScript for styling** (use CSS selectors for hover/active states instead of JavaScript class manipulation), **performance-aware configuration** (set `connectWith` after Sortable initialisation if it causes lag), **consistent API usage** (options during initialisation, `"option"` for changes, `"method"` for actions), and **accessibility verification** (test keyboard navigation and ARIA attributes). jQuery UI is a legacy library, so best practices also include **evaluating modern alternatives** for new projects and **planning a migration path** for long-term maintainability.

**Beginner-Friendly Explanation**
Best practices for jQuery UI are mostly about keeping your page fast and your code clean. Do not load the entire library if you only need a date picker — use the Download Builder to include only what you use. Do not re-query the same elements repeatedly; save them in variables. Do not use JavaScript to add hover effects that CSS can handle. Test your widgets with the keyboard to make sure they are accessible. And remember that jQuery UI is older technology — if you are starting a new project, consider whether a modern framework would be a better fit.

---

### Purposes

- **To keep page weight low** by loading only the components that are used.
- **To improve runtime performance** by caching selections and avoiding unnecessary JavaScript.
- **To maintain clean, consistent widget usage** through the standard options/methods/events pattern.
- **To ensure accessibility** by testing keyboard navigation and ARIA attributes.
- **To make informed technology choices** by recognising jQuery UI's legacy status and planning accordingly.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Loading** | Download Builder or modular npm imports | Loading the full library for one widget |
| **Selectors** | Cache selections in variables | Re-querying the same elements |
| **Styling** | CSS for hover/active states | JavaScript class manipulation for pure styling |
| **Sortable performance** | Set `connectWith` after initialisation if slow | Setting it during initialisation in large lists |
| **API usage** | Options at init, `"option"` for changes, `"method"` for actions | Mixing direct property access with API calls |
| **Accessibility** | Test keyboard and screen reader behaviour | Assuming widgets are accessible by default |
| **Technology choice** | Evaluate modern alternatives for new projects | Using jQuery UI for greenfield development without consideration |

---

### Constraints and Limitations

- **jQuery UI's performance is adequate for most use cases**, but it is not as fast as native or modern framework alternatives.
- **The library is in maintenance mode**; new features are unlikely, and long-term support may be limited.
- **Custom builds require careful dependency management**; missing a required module breaks widgets.

---

### Annotated Code Example (Optimised Usage)

```html
<!DOCTYPE html>
<html>
<head>
  <!-- Step 1: Load only the base theme and required modules -->
  <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.2/themes/base/jquery-ui.css">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://code.jquery.com/ui/1.14.2/jquery-ui.min.js"></script>
  <script>
    $(function() {
      // Step 2: Cache the selection
      var $datepicker = $("#datepicker");

      // Step 3: Initialise with options
      $datepicker.datepicker({ dateFormat: "yy-mm-dd" });

      // Step 4: Use CSS for hover styling instead of JavaScript
      // (defined in stylesheet, not in JS)
    });
  </script>
  <style>
    /* CSS handles hover/active states */
    #datepicker:hover { border-color: #4a90d9; }
  </style>
</head>
<body>
  <input type="text" id="datepicker">
</body>
</html>
```

**Expected Output**
The date picker is initialised with the specified format, the selection is cached, and hover styling is handled by CSS rather than JavaScript.

**Why this output occurs**: The example applies three best practices: modular loading, selection caching, and CSS-based styling.

---

### Real-World Cases

- **Legacy application maintenance**: Optimising existing jQuery UI usage without a full rewrite.
- **Prototype development**: Using jQuery UI for rapid prototyping while planning a modern replacement.
- **Internal tooling**: Accepting jQuery UI's trade-offs for speed of development.

---

## References

- jQuery UI: Tutorial for Beginners (updated) | GUVI — https://www.guvi.in/blog/jquery-ui-tutorial-for-beginners/
- JQuery UI In Depth | Microsoft Learn — https://learn.microsoft.com/en-us/previous-versions/dn535765(v=vs.85)
- Getting Started | jquery/jquery-ui | DeepWiki — https://deepwiki.com/jquery/jquery-ui/1.2-getting-started
- Using jQuery UI ThemeRoller | Learn jQuery — https://learn.jquery.com/jquery-ui/themeroller/
- Theming jQuery UI | Learn jQuery — https://learn.jquery.com/jquery-ui/theming/
- ThemeRoller | jQuery UI — https://jqueryui.com/themeroller/
- Widgets | jQuery UI 1.9 Documentation — https://api.jqueryui.com/1.9/category/widgets/
- Introduction to shinyjqui (jQuery UI interactions) | CRAN — https://cran.r-project.org/web/packages/shinyjqui/vignettes/introduction.html
- Effects | jQuery UI 1.9 Documentation — https://api.jqueryui.com/1.9/category/effects/
- Utilities | jQuery UI — https://jqueryui.com/category/utilities/
- Widget Factory | jQuery UI — https://jqueryui.com/widget/
- Position | jQuery UI — https://jqueryui.com/position/
- Optimize Selectors | Learn jQuery — https://learn.jquery.com/performance/optimize-selectors/
- jQuery UI + loads of buttons = slow? | Stack Overflow — https://stackoverflow.com/questions/1721461/jquery-ui-loads-of-buttons-slow
- connectWith performance problem with Sortable | jQuery UI Bug Tracker — https://bugs.jqueryui.com/ticket/8883
- Using jQuery UI with AMD | Learn jQuery — https://learn.jquery.com/jquery-ui/environments/amd/
- Using jQuery UI with Bower | Learn jQuery — https://learn.jquery.com/jquery-ui/environments/bower/
- Using the classes Option | Learn jQuery — https://learn.jquery.com/jquery-ui/widget-factory/classes-option/
- Extending Widgets with the Widget Factory | Learn jQuery — https://learn.jquery.com/jquery-ui/widget-factory/extending-widgets/