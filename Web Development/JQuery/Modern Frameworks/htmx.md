# jQuery in htmx — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Integrating jQuery into an htmx application is the practice of allowing jQuery's event system, DOM helpers, and plugin ecosystem to coexist with htmx's HTML-attribute-driven AJAX model, where htmx handles server communication and DOM swapping declaratively, and jQuery is used for isolated tasks that htmx does not cover — typically legacy plugins, event bridging, or fine-grained DOM manipulation.

**Technical Definition**
htmx is an HTML-attribute library for AJAX, WebSockets, SSE, and dynamic DOM updates without writing JavaScript. It parses `hx-*` attributes on initial page load via `htmx.bootstrap()` and binds event listeners; dynamically inserted nodes containing `hx-*` attributes are **not** automatically recognised and must be initialised with `htmx.process()`. jQuery operates through its own event system and DOM manipulation APIs. The two libraries integrate through three mechanisms: **(1) htmx's event system** — htmx dispatches events such as `htmx:afterRequest` and `htmx:afterSwap` that jQuery can listen to; **(2) `htmx.onLoad`** — the recommended hook for initialising jQuery plugins on content that htmx has swapped into the DOM; and **(3) `htmx.process()`** — the mandatory call to activate `hx-*` attributes on nodes that jQuery dynamically inserts. The integration requires explicit boundaries: htmx owns the declarative request lifecycle, jQuery owns isolated plugin islands and event bridges, and jQuery's AJAX (`$.ajax`) is generally redundant alongside htmx.

**Beginner‑Friendly Explanation**
htmx lets you build interactive pages by writing special attributes like `hx-get` and `hx-target` directly in your HTML — no JavaScript required. jQuery lets you find elements and change them with JavaScript. They can work together as long as each stays in its lane. htmx handles the server requests and page updates. jQuery handles the old plugins you still need and the small bits of JavaScript logic htmx does not cover. The main rule is: when jQuery inserts new HTML that contains htmx attributes, you must call `htmx.process()` so htmx notices them. And when htmx replaces a chunk of the page, you must use `htmx.onLoad` to re-initialise your jQuery plugins on the new content.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Complementary, not competing** | htmx handles server communication and DOM swaps; jQuery handles plugins, legacy code, and event bridging. |
| **`htmx.process()` is mandatory for jQuery-inserted htmx attributes** | Dynamically inserted `hx-*` attributes are inert until explicitly processed. |
| **`htmx.onLoad` is the plugin initialisation hook** | It fires on initial load and after every htmx swap, scoped to the newly loaded content. |
| **htmx events bridge to jQuery** | htmx dispatches events in both camelCase and kebab-case, which jQuery can listen to via `$(document).on('htmx:...')`. |
| **jQuery AJAX is redundant** | htmx replaces the need for `$.ajax`; using both increases complexity without benefit. |
| **jQuery plugins survive swaps only with re-initialisation** | Plugins such as date pickers, tooltips, and carousels lose their behaviour when htmx replaces the DOM unless re-initialised in `htmx.onLoad`. |
| **Load order matters** | jQuery and its plugins must be loaded before htmx processes the DOM if they are referenced in `htmx.onLoad` callbacks. |

---

### Prerequisites

- Working knowledge of htmx attributes (`hx-get`, `hx-post`, `hx-trigger`, `hx-target`, `hx-swap`).
- Working knowledge of jQuery selection, events, and plugins.
- Understanding of the difference between initial page load and dynamically inserted content.
- Awareness that htmx's request lifecycle is declarative and should not be overridden with jQuery AJAX.

---

### Related Programming Areas

- **AJAX performance** — htmx replaces jQuery AJAX; avoid duplicating request logic.
- **DOM performance** — htmx swaps and jQuery mutations interact; re-initialisation is required.
- **Event performance** — htmx's event system and jQuery's event system are separate; bridging must be explicit.
- **Memory performance** — plugins initialised via `htmx.onLoad` must be cleaned up to avoid leaks.
- **Security** — jQuery sinks (`.html()`) inside htmx-swapped content reintroduce XSS risk.

---

### Core Concepts / Features (Index)

1. jQuery in htmx (General)
2. htmx Attributes and `htmx.process()`
3. DOM Manipulation and `htmx.onLoad`
4. jQuery Plugins in htmx
5. Integration Challenges
6. Integration Best Practices


## Core Concept 1: jQuery in htmx (General)

### Definitions

**Core Definition**
jQuery in htmx is the coexistence of the jQuery library with an htmx-driven application, where htmx manages AJAX requests and DOM swaps through HTML attributes, and jQuery is used for event listening, plugin initialisation, and DOM operations that fall outside htmx's declarative model.

**Technical Definition**
htmx's core design is "hypertext first" — it automatically binds event listeners and manages AJAX behaviour by parsing `hx-*` attributes in HTML, but this mechanism only runs automatically once at initial page load. jQuery operates through its own event system and DOM manipulation APIs, which are invisible to htmx unless explicitly bridged. The two libraries integrate through htmx's **event system** (htmx dispatches events that jQuery can listen to), **`htmx.onLoad`** (the hook for initialising content after swaps), and **`htmx.process()`** (the API for activating htmx attributes on jQuery-inserted nodes). htmx originated as intercooler.js, a library built around jQuery that added behaviour based on HTML attributes, so the two ecosystems have a shared lineage even though they now occupy different niches.

**Beginner‑Friendly Explanation**
htmx and jQuery are like two tools that do different jobs. htmx is the delivery service: you write `hx-get` and it fetches content from the server and puts it on the page. jQuery is the handyman: it finds elements, attaches plugins, and handles events. They can work together as long as you tell each one when the other has done something. When htmx swaps in new content, you tell jQuery "the page changed, please re-initialise your plugins here." When jQuery inserts new HTML that contains htmx attributes, you tell htmx "please notice these new attributes and start managing them."

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them in htmx.
- **To migrate incrementally** from a jQuery codebase to htmx, one feature at a time.
- **To bridge htmx events to jQuery logic** (e.g., showing notifications after a request completes).
- **To initialise third-party UI libraries** (date pickers, tooltips, carousels) on htmx-swapped content.
- **To avoid the cost of a full rewrite** while still moving toward htmx's declarative model.

---

### Syntax Rules and Structure

#### General Integration Pattern

```html
<!-- jQuery and plugins loaded before htmx -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/htmx.org@2.0.4/dist/htmx.min.js"></script>

<div id="content">
  <!-- htmx attributes on initial content -->
  <button hx-get="/api/data" hx-target="#result">Load</button>
  <div id="result"></div>
</div>

<script>
  // Listen to htmx events with jQuery
  $(document).on('htmx:afterRequest', function(evt) {
    console.log('Request completed:', evt.originalEvent.detail.path);
  });

  // Initialise jQuery plugins on htmx-swapped content
  htmx.onLoad(function(content) {
    $(content).find('.datepicker').datepicker();
  });
</script>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `hx-get="/api/data"` | htmx attribute that triggers an AJAX GET request. |
| `hx-target="#result"` | Specifies where the response is inserted. |
| `$(document).on('htmx:afterRequest', …)` | jQuery listens to htmx events via the document. |
| `htmx.onLoad(function(content) { … })` | Runs on initial load and after every swap; scoped to new content. |

---

### Constraints and Limitations

- **jQuery-inserted htmx attributes are inert** until `htmx.process()` is called.
- **htmx swaps remove event listeners**; plugins must be re-initialised in `htmx.onLoad`.
- **jQuery AJAX bypasses htmx's lifecycle** (indicators, error handling, swap behaviour) and should be avoided.
- **htmx events are case-sensitive in kebab-case**; jQuery listeners should use the kebab-case form (`htmx:after-request`) if camelCase is unreliable.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/htmx.org@2.0.4/dist/htmx.min.js"></script>
</head>
<body>
  <div id="app">
    <!-- Step 1: htmx attribute on initial content -->
    <button hx-get="/api/greeting" hx-target="#greeting">Load Greeting</button>
    <div id="greeting"></div>
  </div>

  <script>
    // Step 2: jQuery listens to htmx events
    $(document).on('htmx:afterRequest', function(evt) {
      console.log('Request completed for:', evt.originalEvent.detail.path);
    });

    // Step 3: Re-initialise plugins on swapped content
    htmx.onLoad(function(content) {
      // Example: initialise a tooltip plugin on new content
      $(content).find('[data-tooltip]').tooltip();
    });
  </script>
</body>
</html>
```

**Expected Output**
Clicking the button triggers an htmx request, the response is inserted into `#greeting`, the jQuery `htmx:afterRequest` listener logs the request path, and any `[data-tooltip]` elements in the swapped content are enhanced with the tooltip plugin.

**Why this output occurs**: htmx processes the `hx-get` attribute on the button at initial load. The jQuery listener on `document` catches the `htmx:afterRequest` event. `htmx.onLoad` fires after the swap, giving jQuery a chance to initialise plugins on the newly inserted content.

---

### Real‑World Cases

- **Legacy admin panels**: A page migrating to htmx that still needs a jQuery date picker and tooltip library.
- **Django formsets**: Dynamically inserted forms containing `hx-*` attributes that must be activated with `htmx.process()`.
- **Bootstrap components**: Dropdowns and tooltips that lose their behaviour after htmx swaps and must be re-initialised in `htmx.onLoad`.

---

## Core Concept 2: htmx Attributes and `htmx.process()`

### Definitions

**Core Definition**
htmx attributes (`hx-get`, `hx-post`, `hx-trigger`, `hx-target`, `hx-swap`) are HTML attributes that declare AJAX behaviour declaratively. `htmx.process(element)` is the API that scans a DOM element (and its subtree) for `hx-*` attributes and binds the corresponding event listeners.

**Technical Definition**
htmx's bootstrap phase parses all `hx-*` attributes on initial page load. When new DOM nodes are inserted via JavaScript (including jQuery's `.append()`, `.html()`, or native `innerHTML`), those nodes are **not** automatically processed, and their `hx-*` attributes remain inert. `htmx.process(element)` manually triggers attribute scanning and event binding on the specified element or subtree. It accepts native DOM elements or NodeLists, **not** jQuery objects; jQuery objects must be unwrapped with `[0]` or accessed via `.each()`'s `this`. Calling `htmx.process(document.body)` globally should be avoided because it may cause duplicate bindings and memory leaks.

**Beginner‑Friendly Explanation**
htmx attributes are like instructions written on a parcel: "deliver this to that address." When the page first loads, htmx reads all the instructions. But if you use jQuery to add a new parcel (a new HTML element with `hx-get` on it), htmx does not automatically read the instructions on the new parcel. You have to hand it to htmx and say "please read this one too." That hand-off is `htmx.process()`. Without it, the new element looks like it has htmx attributes, but clicking it does nothing.

---

### Purposes

- **To activate htmx behaviour on dynamically inserted content**.
- **To integrate jQuery-generated HTML** (AJAX responses, templates, formset additions) with htmx.
- **To avoid the "attributes present but non-functional" problem** that occurs when jQuery bypasses htmx's bootstrap.
- **To scope processing to new content**, avoiding global re-initialisation and duplicate bindings.

---

### Syntax Rules and Structure

#### Correct Usage

```javascript
// ✅ Process a jQuery object's underlying DOM node
var $newContent = $(responseHtml);
$('#container').append($newContent);
htmx.process($newContent[0]);          // native DOM element

// ✅ Process each newly inserted element
$newContent.find('[hx-get],[hx-post]').each(function() {
  htmx.process(this);                   // `this` is the native element
});
```

#### Incorrect Usage

```javascript
// ❌ Passes a jQuery object — htmx.process expects a DOM element
htmx.process($newContent);

// ❌ Global re-processing — may cause duplicate bindings and leaks
htmx.process(document.body);
```

---

### Constraints and Limitations

- **`htmx.process()` does not rewrite attributes**; it binds listeners based on existing attributes. Re-setting attributes has no effect.
- **Global processing is discouraged**; it can cause repeated binding and performance degradation.
- **Processing a subtree recursively processes children**; calling it on the root of the new fragment is sufficient.

---

### Annotated Code Example

```html
<button id="addForm">Add Form Row</button>
<div id="formset"></div>

<script>
  $('#addForm').on('click', function() {
    // Step 1: jQuery inserts HTML containing htmx attributes
    var newRow = '<div hx-get="/form/row" hx-trigger="click" hx-target="#detail">Row</div>';
    var $row = $(newRow);
    $('#formset').append($row);

    // Step 2: Activate htmx on the new node
    htmx.process($row[0]);

    // Now clicking the new row triggers the htmx request
  });
</script>
```

**Expected Output**
Clicking "Add Form Row" inserts a new `<div>` with `hx-get` attributes. `htmx.process($row[0])` activates those attributes, so clicking the new row triggers the specified request and targets `#detail`.

**Why this output occurs**: jQuery's `.append()` inserts the node, but htmx does not automatically notice it. `htmx.process()` manually scans the node and binds the `hx-trigger` listener, making the attributes functional.

---

### Real‑World Cases

- **Django formsets**: Dynamically added form rows with `hx-get` and `hx-trigger` attributes that must be activated with `htmx.process()`.
- **AJAX-loaded content**: HTML fragments fetched via `fetch` or jQuery and inserted into the DOM that contain htmx attributes.
- **Template libraries**: Content generated from `<template>` elements (e.g., Alpine's `x-if`) that includes htmx attributes and requires processing after insertion.

---

## Core Concept 3: DOM Manipulation and `htmx.onLoad`

### Definitions

**Core Definition**
`htmx.onLoad(callback)` is an htmx API that registers a callback to run on the initial page load and after every htmx swap, receiving the newly loaded content as its argument. It is the correct place to initialise jQuery plugins and DOM-dependent behaviour on swapped content.

**Technical Definition**
`htmx.onLoad` fires with the swapped content as its argument, allowing callbacks to scope their work to the newly inserted nodes rather than the entire document. This is critical because htmx replaces DOM nodes during swaps, which destroys any event listeners and plugin state attached to the old nodes. Without re-initialisation in `htmx.onLoad`, jQuery plugins such as date pickers, tooltips, carousels, and dropdowns stop working after the first swap. The callback receives a native DOM element (the swapped content root), which jQuery can wrap or query with `.find()`. `htmx.onLoad` is the htmx-native replacement for jQuery's `$(document).ready()` for dynamically loaded content.

**Beginner‑Friendly Explanation**
Every time htmx replaces a piece of the page, the old piece — along with any jQuery plugins attached to it — is thrown away. The new piece looks the same but has no plugins attached. `htmx.onLoad` is htmx's way of saying: "Here is the new piece; please set up your plugins on it." If you forget this step, your date pickers and tooltips will work on the first load and then stop working after the first htmx request.

---

### Purposes

- **To re-initialise jQuery plugins** after htmx swaps replace the DOM.
- **To scope plugin initialisation to new content**, avoiding duplicate bindings on existing nodes.
- **To ensure that dynamically loaded UI components** (tooltips, date pickers, carousels) remain functional across swaps.
- **To replace jQuery's `$(document).ready()`** for content that htmx loads asynchronously.

---

### Syntax Rules and Structure

```javascript
// ✅ Scope initialisation to the newly loaded content
htmx.onLoad(function(content) {
  $(content).find('.datepicker').datepicker();
  $(content).find('.tooltip').tooltip();
});

// ✅ Use a selector to narrow the scope
htmx.onLoad('.chart-container', function(elt) {
  new Chart(elt.querySelector('canvas'), { /* config */ });
});
```

**Component Breakdown**

| Component | Description |
|---|---|
| `htmx.onLoad(callback)` | Registers a callback for initial load and every swap. |
| `content` | The swapped content root (native DOM element). |
| `$(content).find(...)` | Scopes the jQuery selection to the new content. |
| `htmx.onLoad('.selector', callback)` | Narrows the callback to matching elements within the swapped content. |

---

### Constraints and Limitations

- **`htmx.onLoad` fires for the swapped content**, not the entire document; selections must be scoped with `$(content).find(...)` rather than global `$('.selector')`.
- **Plugins initialised outside `htmx.onLoad`** on swapped content will not survive the next swap.
- **Cleanup of plugins before swap** may be necessary for plugins that hold external resources (timers, WebSocket connections); htmx provides `htmx:beforeCleanupElement` for this purpose.

---

### Annotated Code Example

```html
<!-- Initial content with a datepicker -->
<div id="form-area">
  <input type="text" class="datepicker" />
</div>

<!-- htmx button that swaps the form area -->
<button hx-get="/form/new" hx-target="#form-area">New Form</button>

<script>
  // ✅ Initialise datepicker on initial load AND after every swap
  htmx.onLoad(function(content) {
    $(content).find('.datepicker').datepicker();
  });
</script>
```

**Expected Output**
The datepicker works on the initial input. After clicking "New Form", htmx replaces the content of `#form-area` with new HTML containing a `.datepicker` input, and `htmx.onLoad` re-initialises the datepicker on the new input.

**Why this output occurs**: `htmx.onLoad` runs on initial load and after the swap. The callback receives the swapped content and scopes the jQuery selection to it, initialising the plugin only on the new `.datepicker` element.

---

### Real‑World Cases

- **Bootstrap tooltips and dropdowns**: Re-initialising after htmx swaps so they remain functional.
- **jQuery carousels**: Re-initialising after content updates so the carousel does not lose its behaviour.
- **Charts and graphs**: Initialising chart libraries on swapped containers using `htmx.onLoad('.chart-container', …)`.

---

## Core Concept 4: jQuery Plugins in htmx

### Definitions

**Core Definition**
Integrating jQuery plugins into htmx means initialising them on content that htmx has swapped into the DOM, typically via `htmx.onLoad`, and ensuring they are re-initialised after every swap so they remain functional.

**Technical Definition**
jQuery plugins attach to DOM nodes and add event listeners, timers, and DOM elements. When htmx replaces a DOM subtree, all of that plugin state is destroyed. The plugin must be re-initialised on the new nodes. `htmx.onLoad` is the htmx-native hook for this: it fires after every swap and receives the new content, allowing the plugin to be applied only to the newly inserted elements. Plugins that create external resources (timers, WebSocket connections) should also be cleaned up before the swap using `htmx:beforeCleanupElement` to prevent leaks.

**Beginner‑Friendly Explanation**
A jQuery plugin is like a sticker you put on a webpage element. When htmx replaces that element with a new one, the sticker disappears — the new element does not have it. You have to put the sticker on again. `htmx.onLoad` is the moment right after htmx swaps the content; that is when you re-apply your plugins to the new elements. If you do not, your date picker will work on the first page load and then vanish the first time htmx updates the page.

---

### Purposes

- **To keep jQuery plugins functional across htmx swaps**.
- **To initialise plugins only on new content**, avoiding duplicate bindings on existing nodes.
- **To integrate third-party UI libraries** that have no htmx-native equivalent.
- **To prevent memory leaks** from plugins that hold timers or external resources.

---

### Syntax Rules and Structure

```javascript
// ✅ Re-initialise plugins after every swap
htmx.onLoad(function(content) {
  $(content).find('.datepicker').datepicker();
  $(content).find('.tooltip').tooltip();
  $(content).find('.carousel').carousel();
});

// ✅ Cleanup plugins that hold external resources before swap
document.body.addEventListener('htmx:beforeCleanupElement', function(evt) {
  $(evt.detail.elt).find('.carousel').carousel('dispose');
});
```

---

### Constraints and Limitations

- **Plugins that replace DOM elements** (e.g., plugins that wrap or move nodes) may conflict with htmx's own DOM management.
- **Plugins that rely on unique IDs** may duplicate IDs after repeated swaps, causing `getElementById` conflicts.
- **Plugin cleanup is essential** for plugins that create timers or network connections.

---

### Annotated Code Example

```html
<div id="dashboard">
  <div class="chart" data-values="1,2,3"></div>
</div>

<button hx-get="/dashboard/update" hx-target="#dashboard">Refresh</button>

<script>
  htmx.onLoad(function(content) {
    $(content).find('.chart').each(function() {
      var values = $(this).data('values').split(',');
      $(this).someChart({ data: values });
    });
  });
</script>
```

**Expected Output**
The chart plugin renders the initial data. After clicking "Refresh", htmx swaps `#dashboard`, and `htmx.onLoad` re-initialises the chart on the new `.chart` element with the new data.

**Why this output occurs**: `htmx.onLoad` fires after the swap and scopes the selection to the new content. The chart plugin is applied only to the new `.chart` element, so the chart remains functional after the update.

---

### Real‑World Cases

- **Dashboard widgets**: Charts and graphs that must be re-initialised after htmx refreshes the dashboard.
- **Data tables**: jQuery DataTables or similar plugins that lose their behaviour after a swap.
- **Form enhancements**: Date pickers, select2, and validation plugins that must be re-applied to swapped form fields.

---

## Core Concept 5: Integration Challenges

### Definitions

**Core Definition**
Integration challenges are the specific technical problems that arise when jQuery and htmx share the same runtime — including inert `hx-*` attributes on jQuery-inserted nodes, plugin loss after swaps, event system mismatches, and AJAX duplication.

**Technical Definition**
The core challenges are: **(1) htmx does not process dynamically inserted attributes** — jQuery's `.append()`, `.html()`, or native `innerHTML` insert nodes with `hx-*` attributes that htmx ignores until `htmx.process()` is called. **(2) Plugin state is destroyed by swaps** — htmx replaces DOM nodes, removing all event listeners and plugin state, so plugins must be re-initialised in `htmx.onLoad`. **(3) Event system mismatch** — htmx dispatches custom events that jQuery can listen to, but jQuery's own events do not trigger htmx behaviour, and vice versa. **(4) AJAX duplication** — jQuery's `$.ajax` bypasses htmx's request lifecycle (indicators, error handling, swap behaviour), creating two competing request systems. **(5) `htmx-include` timing** — htmx may trigger a request before jQuery has set input values, requiring a custom event trigger.

**Beginner‑Friendly Explanation**
The main problem is that htmx and jQuery do not automatically notice each other. If you use jQuery to add a new button with `hx-get`, htmx will not see it until you call `htmx.process()`. If htmx replaces a section of the page, your jQuery plugins disappear and you have to re-initialise them with `htmx.onLoad`. If you use jQuery AJAX alongside htmx, you end up with two systems doing the same job and competing for control. These are not bugs — they are the consequence of two libraries with different assumptions sharing the same page.

---

### Purposes

- **To understand why htmx behaviour fails on dynamically inserted content**.
- **To anticipate and prevent plugin loss after swaps**.
- **To avoid duplicating AJAX logic** with jQuery and htmx.
- **To choose the correct bridging mechanism** (`htmx.process`, `htmx.onLoad`, or htmx events) for each problem.
- **To make informed decisions** about migration strategy.

---

### Syntax Rules and Structure

| Challenge | Manifestation | Mitigation |
|---|---|---|
| **Inert `hx-*` attributes** | Clicking new element does nothing | Call `htmx.process(element)` after insertion |
| **Plugin loss after swap** | Date picker/tooltip stops working | Re-initialise in `htmx.onLoad` |
| **Event mismatch** | htmx event not caught by jQuery handler | Listen with `$(document).on('htmx:event', …)` |
| **AJAX duplication** | Two competing request systems | Use htmx for requests; remove `$.ajax` |
| **`hx-include` timing** | htmx request fires before jQuery sets values | Trigger a custom event after setting values |
| **Global re-processing** | Duplicate bindings, leaks | Scope `htmx.process()` to new content only |

---

### Constraints and Limitations

- **`htmx.process()` does not rewrite attributes**; re-setting `hx-*` attributes has no effect.
- **Manual click handlers on htmx elements break the request lifecycle**; do not override htmx's bindings.
- **Global `htmx.process(document.body)` is an anti-pattern**; it causes repeated binding and performance degradation.

---

### Annotated Code Example (Plugin Loss After Swap)

```html
<!-- ❌ Plugin initialised only once — lost after swap -->
<script>
  $(document).ready(function() {
    $('.datepicker').datepicker();
  });
</script>

<!-- ✅ Plugin re-initialised after every swap -->
<script>
  htmx.onLoad(function(content) {
    $(content).find('.datepicker').datepicker();
  });
</script>
```

**Expected Output**
The first version works on initial load but the date picker stops working after htmx swaps the content. The second version re-initialises the date picker on the new content after every swap.

**Why this output occurs**: `$(document).ready()` runs once. htmx swaps replace the original `.datepicker` element, removing the plugin. `htmx.onLoad` runs after every swap and applies the plugin to the new element.

---

### Real‑World Cases

- **Django formsets with jQuery insertion**: New form rows with htmx attributes are inert until `htmx.process()` is called.
- **Bootstrap components after swaps**: Dropdowns and tooltips stop working because Bootstrap's JS is not re-initialised.
- **jQuery AJAX alongside htmx**: Two request systems compete, causing duplicate requests and confusing state.

---

## Core Concept 6: Integration Best Practices

### Definitions

**Core Definition**
jQuery-in-htmx best practices are the consolidated conventions — use `htmx.process()` for jQuery-inserted htmx attributes, use `htmx.onLoad` to re-initialise plugins after swaps, listen to htmx events with jQuery, avoid jQuery AJAX, and keep htmx as the primary request system — that make the integration safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual patterns: **(1) call `htmx.process()` on any DOM node inserted via jQuery that contains `hx-*` attributes**; **(2) use `htmx.onLoad` to re-initialise jQuery plugins after every swap**, scoping the selection to the new content; **(3) listen to htmx events via jQuery's event system** for notifications and side effects; **(4) avoid jQuery AJAX** in favour of htmx attributes for server communication; **(5) scope `htmx.process()` to new content**, never `document.body`; and **(6) clean up plugins that hold external resources** before swaps.

**Beginner‑Friendly Explanation**
Best practices are the rules that keep htmx and jQuery working together. When jQuery adds new HTML with htmx attributes, tell htmx to process it. When htmx swaps content, tell jQuery to re-initialise its plugins. Listen to htmx events with jQuery when you need to react to requests. Do not use jQuery AJAX — let htmx handle the requests. And never process the whole page at once; process only the new part.

---

### Purposes

- **To provide a single actionable checklist** for jQuery-in-htmx integration.
- **To prevent the most common integration failures** (inert attributes, lost plugins, duplicate AJAX).
- **To keep htmx as the primary request system** while still supporting legacy jQuery code.
- **To establish a baseline for code reviews and migration planning**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **jQuery-inserted htmx attributes** | `htmx.process($newContent[0])` | Inserting without processing |
| **Plugin re-initialisation** | `htmx.onLoad(function(content) { … })` | `$(document).ready()` only |
| **Event listening** | `$(document).on('htmx:event', …)` | Relying on jQuery events to trigger htmx |
| **AJAX** | htmx attributes (`hx-get`, `hx-post`) | `$.ajax` alongside htmx |
| **Processing scope** | New content only | `htmx.process(document.body)` |
| **Cleanup** | `htmx:beforeCleanupElement` | Leaving timers and connections |

---

### Constraints and Limitations

- **Best practices are migration tools**, not a permanent architecture for new development.
- **`htmx.process()` requires native DOM elements**, not jQuery objects.
- **Event bridging requires discipline**; htmx events and jQuery events are separate systems.

---

### Annotated Code Example (Consolidated)

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/htmx.org@2.0.4/dist/htmx.min.js"></script>
</head>
<body>
  <div id="container">
    <button hx-get="/api/items" hx-target="#items">Load Items</button>
    <div id="items"></div>
  </div>

  <script>
    // ✅ Re-initialise plugins after every htmx swap
    htmx.onLoad(function(content) {
      $(content).find('.datepicker').datepicker();
      $(content).find('.tooltip').tooltip();
    });

    // ✅ Listen to htmx events with jQuery
    $(document).on('htmx:afterRequest', function(evt) {
      console.log('Request completed:', evt.originalEvent.detail.path);
    });

    // ✅ Process htmx attributes on jQuery-inserted content
    function addDynamicButton(html) {
      var $new = $(html);
      $('#container').append($new);
      htmx.process($new[0]);           // native DOM element
    }
  </script>
</body>
</html>
```

**Expected Output**
Clicking "Load Items" triggers an htmx request, the response is inserted into `#items`, `htmx.onLoad` re-initialises any date pickers and tooltips in the new content, and the jQuery listener logs the request path. Dynamically added buttons with htmx attributes are activated via `htmx.process()`.

**Why this output occurs**: Each best practice is applied: `htmx.onLoad` handles plugin re-initialisation, jQuery listens to htmx events, and `htmx.process()` activates htmx attributes on jQuery-inserted nodes.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new jQuery integrations follow safe patterns.
- **Migration planning**: htmx handles new features; jQuery is confined to legacy plugin wrappers.
- **Django formset workflows**: jQuery inserts new rows; `htmx.process()` activates their htmx attributes.

---

## References

- Third-Party JavaScript — htmx Documentation — https://raw.githubusercontent.com/yawaramin/htmx/4c85a36716d175765428dc81903e050c8a570647/www/content/docs.md
- How to properly enable HTMX functionality for jQuery dynamically inserted HTML elements — https://global.php.cn/faq/1797076278.html
- How can I using jQuery dynamically add HTML elements containing htmx-attributes? — Stack Overflow — https://stackoverflow.com/questions/77753431/how-can-i-using-jquery-dynamically-add-html-elements-containing-htmx-attributes
- htmx-2.0.10 Third-Party Integrations — https://github.com/tangledgroup/tangled-skills/blob/main/misc/htmx-2-0-10/reference/16-third-party-integrations.md
- hx-include cannot get the input value that generate from jquery · Issue #1875 — https://github.com/bigskysoftware/htmx/issues/1875
- htmx-2-0-10 Complete Reference — SkillsMP — https://skillsmp.com
- The Future of htmx: Stability and Compatibility Insights — https://tsecurity.de
- JavaScript not functioning after htmx get/post request swapped — Stack Overflow — https://stackoverflow.com
- htmx_jquery — GitHub — https://github.com/rocco-gossmann/htmx_jquery
- hx-live Programmers Guide — https://four.htmx.org