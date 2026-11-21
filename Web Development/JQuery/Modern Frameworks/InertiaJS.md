# jQuery in Inertia.js — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Integrating jQuery into an Inertia.js application is the practice of allowing legacy jQuery code — selectors, DOM manipulation, plugins, and event handlers — to coexist with Inertia's SPA-style navigation and page-component rendering model, typically during an incremental migration from a jQuery-based Laravel application or when a specific third-party plugin has no native Vue/React/Svelte equivalent.

**Technical Definition**
Inertia.js is a routing and state-sharing library that builds single-page applications using classic server-side routing and controllers, without requiring an API. Inertia intercepts link clicks and form submissions, sends XHR requests, and swaps **page components** without a full page reload. This means that `$(document).ready()` — the traditional jQuery initialisation hook — fires only once on the initial page load and never again during Inertia navigation. jQuery plugins that were initialised on the first page are destroyed when Inertia unmounts the page component, and new page components are rendered without the plugin enhancements. The integration therefore requires explicit **lifecycle management**: jQuery plugins must be initialised inside the frontend framework's `mounted` hook (or equivalent), re-initialised after Inertia navigation, and protected from the frontend framework's virtual DOM reconciliation. Inertia provides a **lifecycle event system** (`router.on('navigate', …)`, `router.on('finish', …)`, and native `inertia:*` events) that serves as the bridge between Inertia's navigation lifecycle and jQuery's initialisation logic.

**Beginner-Friendly Explanation**
Inertia makes your Laravel application feel like a single-page app: clicking a link does not reload the whole page; instead, Inertia swaps in a new page component. jQuery plugins, however, are attached to the old page's DOM, and when Inertia swaps the page, those plugins disappear. The classic `$(document).ready()` runs only once, so the new page's elements never get enhanced. The solution is to tell jQuery to re-initialise its plugins every time Inertia finishes navigating to a new page. Inertia provides events such as `navigate` and `finish` that fire at exactly those moments, so you can hook your jQuery initialisation logic into Inertia's lifecycle.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`$(document).ready()` fires only once** | Inertia's SPA navigation does not trigger document-ready again; plugins must be re-initialised on navigation. |
| **Inertia swaps page components** | The frontend framework (Vue/React/Svelte) unmounts the old component and mounts the new one, destroying jQuery plugin state. |
| **Inertia events are the lifecycle bridge** | `router.on('navigate')`, `router.on('finish')`, and native `inertia:*` events provide the correct hooks for re-initialisation. |
| **Frontend framework's `mounted` hook is the local initialisation point** | Vue's `onMounted` / React's `useEffect` / Svelte's `onMount` is where jQuery plugins should be initialised within a page component. |
| **Persistent layouts preserve plugin state across pages** | Layouts that persist across navigation are not re-mounted, so plugins inside them are not destroyed — but this requires deliberate design. |
| **jQuery and Vue/React/Svelte fight over the DOM** | jQuery's direct DOM mutations are overwritten by the frontend framework's re-renders, causing plugins to break or behave unpredictably. |
| **Alpine.js is the preferred alternative** | Inertia ships with Alpine.js (in Laravel's default stack), which is designed for the same declarative UI niche and integrates natively with Inertia. |

---

### Prerequisites

- Working knowledge of Inertia.js page components, `router` events, and persistent layouts.
- Working knowledge of jQuery selection, DOM manipulation, events, and plugins.
- Understanding of the frontend framework's lifecycle (`mounted` / `onMounted` / `useEffect`).
- Awareness of the difference between full page loads and Inertia's SPA navigation.

---

### Related Programming Areas

- **DOM performance** — Inertia's page swaps vs. jQuery's direct DOM access.
- **Event performance** — Inertia's lifecycle events vs. jQuery's event system.
- **Memory performance** — plugins must be destroyed to prevent leaks across navigation.
- **Security** — jQuery sinks (`.html()`) inside page components reintroduce XSS risk.
- **Dependency security** — legacy jQuery versions carry known CVEs.

---

### Core Concepts / Features (Index)

1. jQuery in Inertia (General)
2. Inertia Lifecycle Events
3. DOM Manipulation and Frontend Framework Boundaries
4. jQuery Plugins in Inertia
5. Integration Challenges
6. Integration Best Practices


## Core Concept 1: jQuery in Inertia (General)

### Definitions

**Core Definition**
jQuery in Inertia is the coexistence of the jQuery library with Inertia-driven page components, where jQuery is used for specific isolated tasks — typically legacy plugins or DOM operations that the frontend framework does not cover — while Inertia remains responsible for routing, page-component swapping, and server-driven state management.

**Technical Definition**
Inertia's architecture separates the **server-side routing** (Laravel controllers returning Inertia responses) from the **client-side page components** (Vue, React, or Svelte components). When a visit occurs, Inertia sends an XHR request, receives a new page object, and swaps the current page component for the new one. The frontend framework unmounts the old component (running cleanup hooks) and mounts the new component (running `mounted`/`onMounted`/`useEffect`). jQuery plugins that were initialised on the old component are destroyed during unmount, and the new component's `mounted` hook is the correct place to initialise jQuery on the new DOM. The CodeHunger article documents the canonical failure: Select2 was initialised in `$(document).ready()`, but because Inertia replaces page content without full reloads, `document.ready` never re-fired, leaving new dropdowns uninitialised. The fix was to initialise Select2 inside Inertia's mounted lifecycle hook and listen to `inertia:finish`.

**Beginner-Friendly Explanation**
jQuery in Inertia is a compromise. You are saying: "Most of this page is Inertia with Vue/React/Svelte, but this one part is still jQuery." That is fine as long as the two parts stay in their own lanes. Inertia owns routing and page swaps. The frontend framework owns the component's DOM and reactivity. jQuery owns the isolated plugin that is initialised inside the component's `mounted` hook and cleaned up when the component unmounts. The moment jQuery reaches into a part of the page that the frontend framework is also re-rendering, you get bugs that are hard to diagnose — the framework may wipe out jQuery's changes, or jQuery may make the component's data out of sync with what the user sees.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them in Vue/React/Svelte.
- **To migrate incrementally** from a jQuery-based Laravel codebase to Inertia, one page at a time.
- **To isolate legacy DOM manipulation** inside a page component's mounted lifecycle.
- **To bridge plugin events to Inertia navigation** (e.g., re-initialising on `navigate` or `finish`).
- **To avoid the cost of a full rewrite** while still moving toward Inertia's SPA model.

---

### Syntax Rules and Structure

#### General Isolation Pattern (Vue 3 + Inertia)

```javascript
<script setup>
import { onMounted, onUnmounted } from 'vue'
import $ from 'jquery'
import 'select2'

let $select

onMounted(() => {
  $select = $('#mySelect').select2()
  $select.on('change', (e) => {
    // bridge to Vue state
  })
})

onUnmounted(() => {
  $select?.select2('destroy')
})
</script>

<template>
  <select id="mySelect">
    <option value="a">Option A</option>
  </select>
</template>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `onMounted` | Runs after the component is mounted; initialise jQuery plugins here. |
| `onUnmounted` | Runs before the component is destroyed; clean up plugins here. |
| `$('#mySelect').select2()` | Initialises the plugin on the component's DOM. |
| `select2('destroy')` | Destroys the plugin to prevent leaks. |

---

### Constraints and Limitations

- **The frontend framework must not re-render the jQuery-managed subtree**; use `v-once` (Vue) or a stable DOM container.
- **jQuery plugins must be re-initialised on every Inertia navigation**; the component's `mounted` hook handles this if the component is swapped.
- **jQuery events are not native DOM events**; the framework's event bindings will not receive plugin updates without explicit bridging.
- **jQuery version must be compatible**; legacy jQuery may carry known CVEs.

---

### Annotated Code Example

```javascript
<script setup>
import { onMounted } from 'vue'
import $ from 'jquery'

onMounted(() => {
  // Step 1: Initialise the plugin on the component's DOM
  $('#tooltip').tooltip({ title: 'Hello from jQuery' })

  // Step 2: Cleanup is handled by the component's unmount
})
</script>

<template>
  <div id="tooltip">Hover me</div>
</template>
```

**Expected Output**
The tooltip is initialised on the component's `<div>`. When Inertia navigates away, the component unmounts and the tooltip is cleaned up by the framework's unmount process (or by an explicit cleanup hook).

**Why this output occurs**: `onMounted` runs after the component is inserted into the DOM. jQuery initialises the plugin on the component's element. On navigation, Inertia unmounts the old page component, destroying its DOM and the plugin's state.

---

### Real-World Cases

- **Legacy admin panels**: Wrapping a jQuery DataTable or date picker inside an Inertia page component during incremental migration.
- **Third-party widgets**: Embedding a jQuery-based chart or map library that has no Vue/React/Svelte equivalent.
- **Gradual migration**: Moving one page at a time from a jQuery-rendered Laravel application to Inertia.

---

## Core Concept 2: Inertia Lifecycle Events

### Definitions

**Core Definition**
Inertia's lifecycle events are a set of global events that fire at specific points during an Inertia visit — before the request is sent, when it starts, when it succeeds or fails, when it finishes, and when navigation completes — allowing external JavaScript, including jQuery, to hook into Inertia's navigation pipeline.

**Technical Definition**
Inertia provides an event system accessible via `router.on(eventName, callback)` or native browser events prefixed with `inertia:` (e.g., `document.addEventListener('inertia:start', …)`). The documented events include: **`before`** (before a request is made; cancelable), **`start`** (when a request starts), **`progress`** (during file uploads), **`success`** (on successful page visits), **`error`** (when validation errors are present), **`invalid`** (when a non-Inertia response is received), **`exception`** (on unexpected XHR errors), **`finish`** (after the XHR request completes, for both successful and unsuccessful responses), **`navigate`** (on successful page visits and history navigation), **`prefetching`**, and **`prefetched`**. For jQuery integration, `navigate` and `finish` are the most relevant: `navigate` fires when a new page component has been mounted, making it the correct hook for re-initialising plugins on the new page; `finish` fires after the XHR request completes and is useful for hiding loading indicators and triggering re-initialisation.

**Beginner-Friendly Explanation**
Inertia events are like announcements that Inertia makes at each stage of navigation: "I'm about to start a request", "I'm starting", "I succeeded", "I finished", "I navigated". You can listen for these announcements and run your jQuery code at the right moment. The most important for jQuery is `navigate`, which fires after Inertia has swapped in the new page component. That is when you re-initialise your plugins on the new page. If you try to initialise before the new page is mounted, the elements do not exist yet. If you use `$(document).ready()`, it only fires once and never again.

---

### Purposes

- **To re-initialise jQuery plugins after Inertia navigates to a new page** (`navigate`).
- **To trigger plugin updates after an Inertia request completes** (`finish`).
- **To hide loading indicators and clean up transient UI** (`finish`).
- **To intercept navigation and prompt the user before leaving** (`before`).
- **To handle validation errors and display them via jQuery plugins** (`error`).

---

### Syntax Rules and Structure

#### Registering Listeners with `router.on()`

```javascript
import { router } from '@inertiajs/vue3'

router.on('navigate', (event) => {
  // Re-initialise jQuery plugins after navigation
  $('.select2').select2()
})
```

#### Registering Listeners with Native Events

```javascript
document.addEventListener('inertia:navigate', (event) => {
  $('.select2').select2()
})
```

#### Combined with Component Unmount Cleanup

```javascript
import { onUnmounted } from 'vue'

onUnmounted(
  router.on('navigate', (event) => {
    // listener is automatically removed when the component unmounts
  })
)
```

**Component Breakdown**

| Event | When It Fires | Typical jQuery Use |
|---|---|---|
| `before` | Before the request is made | Confirm navigation, cancel if needed |
| `start` | When the request starts | Show loading indicator |
| `finish` | After the request completes (success or failure) | Hide loading indicator, re-initialise |
| `navigate` | After successful navigation / history | Re-initialise plugins on the new page |
| `error` | When validation errors are present | Display errors via jQuery plugins |

---

### Constraints and Limitations

- **`router.on()` returns a callback** that removes the listener; use it in `onUnmounted` to prevent leaks.
- **`navigate` fires after the page component is mounted**, so the DOM is available.
- **Events are not cancelable** except `before`, `invalid`, and `exception`.
- **Native `inertia:*` events** require the `inertia:` prefix.

---

### Annotated Code Example (Re-initialisation on Navigate)

```javascript
import { router } from '@inertiajs/vue3'
import $ from 'jquery'

// ✅ Re-initialise jQuery plugins after every Inertia navigation
router.on('navigate', () => {
  // Use a short timeout to ensure the DOM is fully patched
  setTimeout(() => {
    $('.select2').each(function () {
      if (!$(this).hasClass('select2-hidden-accessible')) {
        $(this).select2()
      }
    })
  }, 0)
})
```

**Expected Output**
After every Inertia navigation, the `navigate` event fires and the callback re-initialises Select2 on any new `.select2` elements that are not already enhanced.

**Why this output occurs**: `navigate` fires after the new page component is mounted. The `setTimeout(0)` defers execution until the microtask queue is flushed, ensuring the DOM is fully settled. The guard class prevents double-initialisation on elements that were already enhanced.

---

### Real-World Cases

- **Select2 after Inertia navigation**: Re-initialising Select2 on every page that uses it.
- **Loading indicators**: Using `start` and `finish` to show and hide a spinner via jQuery.
- **Unsaved changes warning**: Using `before` to confirm navigation away from a dirty form.

---

## Core Concept 3: DOM Manipulation and Frontend Framework Boundaries

### Definitions

**Core Definition**
A DOM manipulation boundary in Inertia is an explicit region of a page component where either the frontend framework (Vue/React/Svelte) or jQuery has exclusive control, preventing the framework's virtual DOM reconciliation from overwriting jQuery's changes and vice versa.

**Technical Definition**
Inertia page components are rendered by a frontend framework with a virtual DOM (Vue, React) or compiled reactivity (Svelte). When the framework re-renders a component — due to a reactive state change or an Inertia page swap — it may overwrite DOM modifications made by jQuery. The Laracasts discussion states: "If your jQuery/vanilla JS code modifies DOM elements directly, that bypasses Vue logic, and the changes will be overwritten whenever Vue re-renders that section of the page". The boundary pattern isolates jQuery inside a container that the framework does not re-render, such as a `v-once` block in Vue or a DOM node outside the reactive template. The jQuery plugin manages everything inside that container, and the framework manages everything outside it.

**Beginner-Friendly Explanation**
Imagine a page component as a document that the framework constantly edits. If jQuery writes something on the document, the framework may later erase it when it re-renders. The boundary is like giving jQuery its own separate notebook: the framework promises not to touch jQuery's notebook, and jQuery promises to stay in it. In Vue, this is often done with `v-once` on a container, which tells Vue to render it once and never update it. In React, a `ref`‑attached container that React does not re-render serves the same purpose.

---

### Purposes

- **To prevent the frontend framework from overwriting jQuery's DOM changes**.
- **To ensure the framework's virtual DOM remains consistent** with the real DOM.
- **To isolate jQuery plugins** inside a container the framework does not re-render.
- **To allow legacy code to run safely** alongside Inertia page components.
- **To make the integration predictable** and easier to debug.

---

### Syntax Rules and Structure

#### Vue: `v-once` Boundary

```vue
<template>
  <!-- ✅ Framework renders this once and never updates it -->
  <div v-once ref="pluginContainer"></div>
</template>
```

#### React: Stable Ref Container

```jsx
// ✅ React renders an empty div; jQuery manages the inside
return <div ref={containerRef} />
```

#### Svelte: `bind:this` Container

```svelte
<!-- ✅ Svelte renders an empty container -->
<div bind:this={container}></div>
```

---

### Constraints and Limitations

- **The boundary must be absolute**; even a single framework update to the jQuery container breaks the guarantee.
- **Communication must go through props and callbacks**, not through shared DOM nodes.
- **The boundary adds a wrapper element** to the DOM, which may affect layout or styling.

---

### Annotated Code Example (Vue `v-once` Boundary)

```vue
<script setup>
import { onMounted } from 'vue'
import $ from 'jquery'

onMounted(() => {
  $('#chart').someChart({ data: [1, 2, 3] })
})
</script>

<template>
  <!-- ✅ v-once prevents Vue from re-rendering the chart container -->
  <div v-once id="chart"></div>
</template>
```

**Expected Output**
Vue renders the empty `<div>` once. jQuery draws the chart inside it. Vue never re-renders the chart's internal DOM, so the chart survives reactive updates elsewhere in the component.

**Why this output occurs**: `v-once` tells Vue to treat the element as static. The chart plugin has exclusive control over the inside, and Vue does not attempt to reconcile it.

---

### Real-World Cases

- **Charts and graphs**: A Vue dashboard embedding a legacy jQuery chart in a `v-once` container.
- **Map integrations**: A jQuery-based map plugin rendered inside a React ref container.
- **Rich text editors**: A jQuery WYSIWYG editor wrapped in a Svelte `bind:this` container.

---

## Core Concept 4: jQuery Plugins in Inertia

### Definitions

**Core Definition**
Integrating a jQuery plugin into Inertia means initialising the plugin inside the page component's mounted lifecycle, re-initialising it after Inertia navigation if necessary, and cleaning it up when the component unmounts — while protecting the plugin's DOM from the frontend framework's re-renders.

**Technical Definition**
jQuery plugins attach to DOM nodes and often add their own event listeners, timers, and DOM elements. Inertia's page swap unmounts the old component (destroying plugin state) and mounts the new component. The correct integration pattern is: **(1)** initialise the plugin in the component's `mounted` hook (`onMounted` in Vue, `useEffect` in React, `onMount` in Svelte); **(2)** clean up in the component's unmount hook; **(3)** use a boundary (`v-once`, stable ref) to prevent the framework from overwriting plugin DOM; **(4)** if the plugin is used in multiple pages and the component is not re-mounted (persistent layout), listen to `inertia:navigate` to re-initialise; and **(5)** bridge plugin events to the component's state or Inertia's state.

**Beginner-Friendly Explanation**
A jQuery plugin is like a piece of furniture that assembles itself when you plug it in. In Inertia, each page component is a room that gets torn down and rebuilt when you navigate. You plug the plugin in when the room is built (`mounted`), and you pack it up when the room is destroyed (`unmounted`). If the plugin needs to persist across pages (e.g., in a persistent layout), you tell it to rebuild itself every time Inertia finishes navigating, using the `navigate` or `finish` event.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them in the frontend framework.
- **To synchronise plugin state with Inertia page state** so the component reflects user interaction.
- **To protect plugin DOM from the framework's re-renders**.
- **To re-initialise plugins across Inertia navigation**.
- **To clean up plugin resources** on unmount to prevent leaks.

---

### Syntax Rules and Structure

#### Complete Plugin Integration Pattern (Vue 3)

```vue
<script setup>
import { onMounted, onUnmounted } from 'vue'
import $ from 'jquery'

let $select

onMounted(() => {
  // Initialise plugin on component mount
  $select = $('#mySelect').select2()

  // Bridge plugin event to Vue state
  $select.on('change', (e) => {
    // update state
  })
})

onUnmounted(() => {
  // Destroy plugin on component unmount
  $select?.select2('destroy')
})
</script>

<template>
  <select id="mySelect">
    <option value="a">Option A</option>
  </select>
</template>
```

#### Persistent Layout Re-initialisation

```javascript
import { router } from '@inertiajs/vue3'

router.on('navigate', () => {
  // Re-initialise plugins in persistent layouts
  $('.persistent-select2').select2()
})
```

---

### Constraints and Limitations

- **Plugins must provide a cleanup method** (`destroy`, `remove`, etc.); otherwise, manual listener removal is required.
- **Plugin options must be mapped carefully** from component props.
- **Plugin events must be bridged** to the framework's state; they do not propagate automatically.
- **Duplicate initialisation** can occur if both the `mounted` hook and the `navigate` event attempt to initialise the same plugin; use a guard class.

---

### Annotated Code Example (Select2 with Navigation Re-initialisation)

```javascript
<script setup>
import { onMounted, onUnmounted } from 'vue'
import { router } from '@inertiajs/vue3'
import $ from 'jquery'

let removeNavigateListener

function initSelect2() {
  $('.select2').each(function () {
    if (!$(this).hasClass('select2-hidden-accessible')) {
      $(this).select2()
    }
  })
}

onMounted(() => {
  initSelect2() // initial mount
  removeNavigateListener = router.on('navigate', initSelect2) // after navigation
})

onUnmounted(() => {
  removeNavigateListener?.() // clean up listener
  $('.select2').select2('destroy')
})
</script>
```

**Expected Output**
Select2 is initialised when the component mounts and re-initialised after every Inertia navigation. The listener is removed when the component unmounts, and the plugin is destroyed.

**Why this output occurs**: `onMounted` handles the initial mount. `router.on('navigate', initSelect2)` handles subsequent navigations. The guard class prevents double-initialisation. `onUnmounted` removes the listener and destroys the plugin.

---

### Real-World Cases

- **Select2 across pages**: Re-initialising Select2 on every Inertia navigation.
- **Date pickers in persistent layouts**: Re-initialising the date picker after navigation.
- **Charts in dashboards**: Re-initialising chart plugins when Inertia swaps the dashboard page.

---

## Core Concept 5: Integration Challenges

### Definitions

**Core Definition**
Integration challenges are the specific technical problems that arise when jQuery and Inertia share the same runtime — including `document.ready` firing only once, plugins being destroyed by page swaps, virtual DOM overwriting jQuery changes, duplicate initialisation, and multiple jQuery versions.

**Technical Definition**
The core challenges are: **(1) `$(document).ready()` does not re-fire on Inertia navigation** — the classic jQuery initialisation hook runs only on the initial full page load, leaving new page components unenhanced. **(2) Page swaps destroy plugin state** — Inertia unmounts the old component, removing all event listeners and DOM modifications; plugins must be re-initialised on the new component. **(3) Virtual DOM reconciliation overwrites jQuery changes** — the frontend framework re-renders and may erase jQuery's DOM modifications. **(4) Duplicate initialisation** — re-initialising a plugin on an already-enhanced element causes double widgets or errors; a guard is required. **(5) Multiple jQuery versions** — including jQuery twice (e.g., via CDN and bundle) causes plugins to attach to one instance while the code uses another, producing "is not a function" errors. **(6) Script load order** — jQuery must be loaded before plugins, and both must be loaded before the page component's `mounted` hook runs.

**Beginner-Friendly Explanation**
The main problem is that Inertia does not reload the page, but jQuery expects a page reload to trigger its initialisation. When Inertia swaps in a new page, the old page's plugins are gone, and the new page never gets initialised because `$(document).ready()` does not fire again. There are smaller problems too: the frontend framework may erase jQuery's changes, re-initialising a plugin twice creates duplicate widgets, and loading jQuery twice breaks everything. Knowing these challenges is the first step to avoiding them.

---

### Purposes

- **To understand why jQuery plugins stop working after Inertia navigation**.
- **To anticipate the failure modes** of mixing the two libraries.
- **To design integration patterns** that avoid the conflicts.
- **To debug issues** that arise from page swaps, virtual DOM overwrites, and duplicate initialisation.
- **To make informed decisions** about migration strategy.

---

### Syntax Rules and Structure

| Challenge | Manifestation | Mitigation |
|---|---|---|
| **`document.ready` only once** | Plugins stop working after navigation | Initialise in `mounted` and `navigate` events |
| **Page swap destroys plugins** | Plugin state lost | Re-initialise in `mounted` / `navigate` |
| **Virtual DOM overwrites** | jQuery changes disappear | `v-once` / stable ref boundary |
| **Duplicate initialisation** | Double widgets or errors | Guard class (`plugin-initialized`) |
| **Multiple jQuery versions** | "is not a function" errors | Ensure only one jQuery instance |
| **Script load order** | Plugin functions undefined | Load jQuery, then plugins, then app code |

---

### Constraints and Limitations

- **`router.on('navigate')` fires for every navigation**; scope re-initialisation to the relevant elements.
- **`setTimeout(0)` may be needed** to ensure the DOM is fully settled after navigation.
- **Persistent layouts do not re-mount**; plugins inside them are not re-initialised by the component's `mounted` hook and require the `navigate` event.

---

### Annotated Code Example (Duplicate Initialisation Guard)

```javascript
function initSelect2(el) {
  var $el = $(el)
  if ($el.hasClass('select2-hidden-accessible')) return // ✅ guard
  $el.select2()
}
```

**Expected Output**
The plugin is initialised exactly once per element, even if `initSelect2` is called multiple times (e.g., from both `mounted` and `navigate`).

**Why this output occurs**: Select2 adds the `select2-hidden-accessible` class to the original `<select>` when it initialises. The guard checks for this class and returns early, preventing duplicate widgets.

---

### Real-World Cases

- **Select2 "is not a function"**: Caused by missing or multiple jQuery versions.
- **Plugins disappearing after navigation**: Caused by `document.ready` firing only once.
- **jQuery changes overwritten by Vue**: Caused by direct DOM manipulation without a boundary.

---

## Core Concept 6: Integration Best Practices

### Definitions

**Core Definition**
jQuery-in-Inertia best practices are the consolidated conventions — initialise in the component's mounted hook, re-initialise on `navigate`, guard against duplicates, use a boundary to protect plugin DOM, bridge events explicitly, and prefer framework-native alternatives — that make the integration safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual patterns: **(1) initialise jQuery plugins in the page component's `mounted` hook** (`onMounted`, `useEffect`, `onMount`), not in `$(document).ready()`; **(2) re-initialise on `inertia:navigate`** for plugins used across pages or in persistent layouts; **(3) guard against duplicate initialisation** with a class or attribute; **(4) use a DOM boundary** (`v-once`, stable ref) to prevent the framework from overwriting jQuery's DOM; **(5) bridge plugin events to the component's state** via callbacks, not `wire:model` or `v-model`; **(6) ensure only one jQuery version is loaded** and that it is loaded before plugins; and **(7) prefer Vue/React/Svelte-native alternatives** or Alpine.js for new UI logic.

**Beginner-Friendly Explanation**
Best practices are the rules that keep the integration from exploding. Initialise your plugins when the component mounts, not when the document is ready. Re-initialise when Inertia navigates to a new page. Prevent double-initialisation. Protect your plugin's DOM from the framework. Connect plugin events to the framework's data. Use only one jQuery. And ask yourself: do I actually need jQuery here, or can Vue/React/Svelte/Alpine do this? If the framework can do it, use the framework.

---

### Purposes

- **To provide a single actionable checklist** for jQuery-in-Inertia integration.
- **To prevent the most common integration failures** (plugins not re-initialising, virtual DOM overwrites, duplicate widgets).
- **To balance legacy compatibility with Inertia's SPA model**.
- **To establish a baseline for code reviews and migration planning**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Initialisation** | Component's `mounted` hook | `$(document).ready()` |
| **Re-initialisation** | `router.on('navigate', …)` | Assuming `mounted` fires on every navigation |
| **Duplicate guard** | Class/attribute check | Re-initialising without a guard |
| **DOM boundary** | `v-once` / stable ref | Direct jQuery on framework-managed DOM |
| **Event bridge** | Callbacks / `emit` | Relying on framework bindings |
| **jQuery version** | Single instance, loaded first | Multiple versions or wrong order |
| **Alternatives** | Vue/React/Svelte/Alpine-native | jQuery for new UI logic |

---

### Constraints and Limitations

- **Best practices are migration tools**, not a permanent architecture for new development.
- **`navigate` fires for every navigation**; scope re-initialisation carefully.
- **Boundaries add wrapper elements** and require explicit communication.

---

### Annotated Code Example (Consolidated)

```javascript
<script setup>
import { onMounted, onUnmounted } from 'vue'
import { router } from '@inertiajs/vue3'
import $ from 'jquery'

let removeNavigate

function initPlugins() {
  $('.select2').each(function () {
    if (!$(this).hasClass('select2-hidden-accessible')) {
      $(this).select2()
    }
  })
}

onMounted(() => {
  initPlugins() // initial mount
  removeNavigate = router.on('navigate', initPlugins) // after navigation
})

onUnmounted(() => {
  removeNavigate?.()
  $('.select2').select2('destroy')
})
</script>
```

**Expected Output**
Select2 is initialised on mount and re-initialised after every Inertia navigation. Duplicate initialisation is prevented by the guard class. The listener and plugin are cleaned up on unmount.

**Why this output occurs**: `onMounted` handles the first page. `router.on('navigate', initPlugins)` handles subsequent pages. The guard prevents double widgets. `onUnmounted` cleans up the listener and destroys the plugin.

---

### Real-World Cases

- **Code reviews**: A checklist ensures that new jQuery integrations follow safe patterns.
- **Migration planning**: Each wrapped plugin is a candidate for eventual replacement with a framework-native component or Alpine.
- **Legacy systems**: The pattern allows incremental migration without breaking existing functionality.

---

## References

- Inertia.js Events | Inertia.js Documentation — https://v2.inertiajs.com/events
- Events | Inertia.js v3 Documentation — https://inertiajs.mintlify.dev/docs/v3/advanced/events
- Fix "select2 is not a function" or "mask is not a function" Errors – Causes and Solutions | CodeHunger — https://www.codehunger.in/blog/fixing-select2-is-not-a-function-or-mask-is-not-a-function-errors-when-navigating-why-it-happens-and-how-to-solve-it
- Breaking the js in vue with inertia and laravel | Laracasts — https://laracasts.com/discuss/channels/laravel/breaking-the-js-in-vue-with-inertia-and-laravel
- Using jQuery-based libraries with Inertia app | Laracasts — https://laracasts.com/discuss/channels/inertia/using-jquery-based-libraries-with-inertia-app
- 〖Laravel + Inertia + Vue〗ライフサイクルごとに発火するイベントを全て紐解く！ | Qiita — https://qiita.com/sigeta/items/58734b7296742a2ff262
- Inertia.js Persistent Layouts | Inertia.js Documentation — https://inertiajs.com/pages#persistent-layouts
- jQuery in Vue 3 with Inertia | Laracasts — https://laracasts.com/discuss/channels/inertia
- Inertia.js - Events (v2) | Inertia.js Documentation — https://v2.inertiajs.com/events
- Inertia.js - Events (GitHub Docs) | GitHub — https://github.com/inertiajs/docs/blob/master/docs/advanced/events.mdx