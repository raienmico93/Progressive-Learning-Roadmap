# jQuery Migration Strategies — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery migration strategies are the structured approaches, patterns, and decision frameworks used to move an application away from jQuery toward modern JavaScript solutions — vanilla JavaScript, component frameworks (React, Vue, Angular, Svelte), lightweight declarative libraries (Alpine.js), or hypermedia-driven tools (htmx) — while maintaining business continuity, preserving tested behaviour, and minimising risk.

**Technical Definition**
jQuery migration is a legacy modernisation problem. The legacy system is characterised by **imperative DOM manipulation**, **global state scattered across the DOM**, and **jQuery plugin dependencies** that assume full-page reloads and direct DOM ownership. The target system may be **native browser APIs** (vanilla JS), a **declarative component framework** (React/Vue/Angular/Svelte), a **lightweight reactive layer** (Alpine), or a **hypermedia-driven architecture** (htmx). The core technical challenge is **DOM ownership conflict**: jQuery assumes it can read and mutate the DOM at any time, while modern frameworks assume exclusive ownership of their subtrees and reconcile against a virtual or compiled representation. Migration strategies therefore centre on **coexistence**, **boundary management**, and **incremental replacement** — typically following the **Strangler Fig pattern**, where the new system grows around the old one and gradually replaces it.

**Beginner‑Friendly Explanation**
jQuery migration is the process of moving a website from jQuery to something more modern. It is like renovating a house while people are still living in it: you cannot knock everything down and start over, because the residents need a place to sleep. So you renovate one room at a time — replace the kitchen, then the bathroom, then the living room — while the rest of the house remains functional. The **Strangler Fig pattern** is the name for this approach: the new code grows around the old code, and piece by piece, the old code is removed. The alternative — a full rewrite — is tempting but risky: it freezes feature development, discards years of bug fixes and business logic, and concentrates all the risk into a single high-stakes release. Most teams should migrate incrementally, using seams such as routes, page boundaries, or module boundaries to separate old and new code.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Strangler Fig is the default** | Incremental, seam-based replacement distributes risk and preserves tested behaviour; full rewrites are the exception for small, isolated surfaces with no seams. |
| **Coexistence is mandatory** | Legacy and modern code must run side by side during the transition; boundaries and adapters prevent DOM ownership conflicts. |
| **Business logic is embedded in the view** | jQuery event handlers and plugins often encode untested business rules; migration must preserve this behaviour, not just the visual output. |
| **Test coverage is the safety net** | Characterisation tests must be added before refactoring begins; migrating without tests risks silent behaviour regressions. |
| **Different targets, different patterns** | Vanilla JS uses direct API replacement; frameworks use wrapper components/directives; Alpine uses declarative attributes; htmx replaces AJAX with HTML attributes. |
| **Dual-stack overhead is real** | Strangler migrations require two build pipelines, two deploy paths, and two sets of dependencies to patch for the duration of the migration. |
| **Seams determine feasibility** | Route boundaries, iframe boundaries, distinct page loads, and module boundaries are the natural seams for coexistence. No seams at all is the strongest signal toward a bounded rewrite of that specific surface. |

---

### Prerequisites

- A working knowledge of the existing jQuery codebase, including plugin dependencies and event-handler logic.
- Familiarity with modern JavaScript (ES2018+), module systems, and at least one target framework or library.
- Access to a test suite (or the ability to create characterisation tests) for the code being migrated.
- Understanding of the browser's DOM APIs and the target framework's rendering model.

---

### Related Programming Areas

- **Legacy modernisation** — the broader discipline of updating old systems without disrupting operations.
- **Front-end architecture** — choosing the right target (vanilla, framework, Alpine, htmx) based on application needs.
- **Software testing** — characterisation tests, regression suites, and visual regression testing.
- **Build tooling** — Vite, Webpack, module federation, and code splitting for gradual adoption.
- **Security** — jQuery sinks (`.html()`) and outdated jQuery versions carry XSS and prototype pollution risks that migration can eliminate.

---

### Core Concepts / Features (Index)

1. jQuery to Vanilla JavaScript
2. jQuery to React
3. jQuery to Vue
4. jQuery to Angular
5. jQuery to Svelte
6. jQuery to Alpine.js
7. jQuery to htmx
8. Incremental Migration
9. The Strangler Pattern
10. Migration Best Practices


## Core Concept 1: jQuery to Vanilla JavaScript

### Definitions

**Core Definition**
Migrating from jQuery to vanilla JavaScript means replacing jQuery's selectors, DOM manipulation, event handling, AJAX, and utility functions with native browser APIs — `querySelector`, `classList`, `addEventListener`, `fetch`, and standard array methods — thereby removing the jQuery dependency entirely.

**Technical Definition**
The migration maps each jQuery method to its native equivalent: `$('#id')` → `document.getElementById('id')`; `$('.class')` → `document.querySelectorAll('.class')`; `.addClass()` → `classList.add()`; `.on()` → `addEventListener()`; `$.ajax()` → `fetch()`; `$.each()` → `Array.prototype.forEach()`; `$(document).ready()` → `DOMContentLoaded` or the `domReady().then(fn)` utility. jQuery plugins that have no native equivalent must either be replaced by vanilla alternatives or retained temporarily as isolated dependencies. The migration is typically performed **module by module**, with a utility layer (`src/utils/dom.js`, `src/utils/events.js`, `src/utils/http.js`) providing consistent wrappers until all call sites are converted.

**Beginner‑Friendly Explanation**
Migrating to vanilla JavaScript is like replacing a universal remote control with the individual remotes that came with your devices. jQuery is convenient because it gives you one way to do everything: select elements, change classes, handle events, make requests. Native JavaScript does the same things, but with slightly different syntax for each task. The migration is about learning those native equivalents and replacing jQuery calls one at a time. The advantage is that you no longer need to load the jQuery library at all, which makes your page smaller and faster.

---

### Purposes

- **To reduce bundle size and improve performance** by eliminating the jQuery library.
- **To align with modern web standards** and native browser capabilities.
- **To remove a dependency** that carries historical security vulnerabilities.
- **To prepare the codebase for framework adoption** by first understanding its DOM interactions.
- **To simplify the technology stack** for teams that no longer need jQuery's cross-browser abstractions.

---

### Syntax Rules and Structure

| jQuery | Vanilla JavaScript |
|---|---|
| `$('#id')` | `document.getElementById('id')` |
| `$('.class')` | `document.querySelectorAll('.class')` |
| `$(el).addClass('a')` | `el.classList.add('a')` |
| `$(el).on('click', fn)` | `el.addEventListener('click', fn)` |
| `$(el).text('x')` | `el.textContent = 'x'` |
| `$(el).attr('x', 'y')` | `el.setAttribute('x', 'y')` |
| `$.ajax()` / `$.get()` | `fetch()` with `async/await` |
| `$.each(arr, fn)` | `arr.forEach(fn)` |
| `$(document).ready(fn)` | `document.addEventListener('DOMContentLoaded', fn)` |

---

### Constraints and Limitations

- **No native replacement for jQuery plugins**; plugins must be replaced with vanilla alternatives or retained temporarily.
- **Event delegation syntax is more verbose**; `closest()` and `matches()` replace jQuery's delegated `.on()`.
- **AJAX error handling differs**; `fetch()` does not reject on HTTP error statuses and requires explicit status checks.
- **Browser compatibility polyfills** may be needed for older browsers, though modern browsers have closed most gaps.

---

### Annotated Code Example

```javascript
// ❌ jQuery
$('#increment').on('click', function() {
  let count = parseInt($('#count').text(), 10);
  count++;
  $('#count').text(count);
});

// ✅ Vanilla JavaScript
document.getElementById('increment').addEventListener('click', () => {
  const countEl = document.getElementById('count');
  let count = parseInt(countEl.textContent, 10);
  count++;
  countEl.textContent = count;
});
```

**Expected Output**
Clicking the increment button increases the displayed count in both versions.

**Why this output occurs**: The vanilla version uses `getElementById` to select the elements, `addEventListener` to bind the click handler, and `textContent` to read and write the count. The behaviour is identical to the jQuery version, without the library dependency.

---

### Real‑World Cases

- **TYPO3 CMS**: Removed jQuery from its core extensions and replaced it with vanilla JavaScript.
- **Wagtail CMS**: Replaced jQuery in `privacy-switch.js` with vanilla JavaScript and browser APIs.
- **XenForo**: Moved jQuery to vanilla JavaScript in version 2.3 as part of a broader modernisation effort.

---

## Core Concept 2: jQuery to React

### Definitions

**Core Definition**
Migrating from jQuery to React means replacing imperative DOM manipulation with React's declarative component model, where UI is a function of state and React reconciles the DOM against a virtual representation.

**Technical Definition**
React assumes exclusive ownership of the DOM subtree it renders. jQuery assumes it can freely read and mutate the DOM. When both operate on the same nodes, React's virtual DOM becomes inconsistent with the real DOM, leading to lost updates, duplicated nodes, or exceptions during reconciliation. The migration therefore follows the **Strangler Fig pattern**: React components are introduced at route or page boundaries, jQuery continues to own the remaining DOM, and the two coexist through explicit interoperability strategies. The academic literature identifies seven coexistence patterns, including **mounting React inside jQuery-managed containers**, **wrapping jQuery plugins as React components**, **a custom event bridge**, **a shared state layer**, **route-based strangling**, **islands architecture**, and **build tooling considerations** such as module federation.

**Beginner‑Friendly Explanation**
React wants to be the only chef in the kitchen. It decides what goes on the plate and when. jQuery wants to reach in and rearrange the food whenever it likes. If both do that, the plate gets messy. The solution is to give React its own kitchen and let jQuery handle a separate counter. In practice, you start by replacing one page or one section at a time with React, and you keep jQuery running for everything else until the migration is complete. You can wrap individual jQuery plugins in React components so they live inside React's world without fighting it.

---

### Purposes

- **To adopt a component-based architecture** with better testability and a larger hiring pool.
- **To move to a declarative rendering model** where UI is a function of state.
- **To enable incremental migration** without freezing feature development or discarding business logic.
- **To isolate legacy jQuery code** inside wrapper components until it can be replaced.
- **To improve maintainability** by removing the implicit state scattered across the DOM.

---

### Syntax Rules and Structure

#### Wrapping a jQuery Plugin as a React Component

```javascript
import { useRef, useEffect } from 'react';
import $ from 'jquery';

function JQueryPluginWrapper({ options }) {
  const containerRef = useRef(null);

  useEffect(() => {
    const $el = $(containerRef.current);
    $el.somePlugin(options);

    return () => {
      $el.somePlugin('destroy');
    };
  }, [options]);

  return <div ref={containerRef} />;
}
```

#### Mounting React Inside a jQuery-Managed Container

```javascript
// jQuery page
$('#react-root').each(function() {
  const root = ReactDOM.createRoot(this);
  root.render(<MyReactApp />);
});
```

---

### Constraints and Limitations

- **React must not update the jQuery-managed subtree**; use a ref-attached empty container.
- **jQuery plugins must be destroyed** in the `useEffect` cleanup to prevent leaks.
- **Event systems are separate**; jQuery events do not trigger React handlers, and vice versa.
- **Shared auth/session models** must be solved before coexistence begins.

---

### Annotated Code Example

```javascript
import { useRef, useEffect } from 'react';
import $ from 'jquery';

function LegacyTooltip({ text }) {
  const containerRef = useRef(null);

  useEffect(() => {
    // Step 1: Initialise the jQuery plugin on the real DOM node
    const $el = $(containerRef.current);
    $el.tooltip({ title: text });

    // Step 2: Destroy the plugin on unmount
    return () => {
      $el.tooltip('destroy');
    };
  }, [text]);

  // Step 3: React renders an empty container
  return <div ref={containerRef} />;
}
```

**Expected Output**
The tooltip is initialised on the container. When the component unmounts, the tooltip is destroyed and its listeners are removed.

**Why this output occurs**: `useEffect` runs after React commits the empty `<div>`. jQuery initialises the plugin on that node. The cleanup function runs on unmount, calling `destroy` to release resources.

---

### Real‑World Cases

- **Legacy banking platforms**: Strangler Fig migration from jQuery monoliths to React component architecture with zero downtime.
- **Internal tools applications**: Five-phase migration using React islands and event bridges.
- **E-commerce platforms**: Route-based strangling with micro-frontends, replacing jQuery interfaces incrementally.

---

## Core Concept 3: jQuery to Vue

### Definitions

**Core Definition**
Migrating from jQuery to Vue means replacing imperative DOM manipulation with Vue's reactive, component-based model, where data drives the view through template bindings and reactive state.

**Technical Definition**
Vue's reactivity system tracks dependencies and schedules DOM updates asynchronously. jQuery operates outside this system. The migration follows a **progressive coexistence** strategy: Vue 3 is introduced alongside jQuery, mounting into named container elements within existing pages. jQuery plugins are wrapped in Vue directives or components that manage their lifecycle. The recommended steps are: **(1)** reserve mount points in jQuery pages; **(2)** replace feature islands (filters, lists, modals) with Vue components; **(3)** wrap retained jQuery plugins in Vue directives that initialise in `mounted` and destroy in `beforeUnmount`; **(4)** abstract AJAX into services; **(5)** introduce a state management layer (Pinia); and **(6)** migrate routing to `vue-router` incrementally.

**Beginner‑Friendly Explanation**
Vue is like a smart assistant that watches your data and updates the page automatically. jQuery is like a manual worker who changes the page by hand. The migration is about teaching the manual worker to hand over tasks to the smart assistant, one task at a time. You start by putting a Vue component in one corner of the page — maybe a filter widget — and letting it manage itself. The rest of the page stays on jQuery. Over time, more and more of the page becomes Vue, until jQuery is no longer needed.

---

### Purposes

- **To adopt reactive state management** and eliminate DOM-based state.
- **To improve maintainability** through componentisation.
- **To enable better tooling** (Vite, TypeScript, ESLint, Vitest).
- **To migrate incrementally** without disrupting live operations.
- **To wrap legacy jQuery plugins** in a managed lifecycle until they can be replaced.

---

### Syntax Rules and Structure

#### jQuery Plugin as a Vue Directive

```javascript
import $ from 'jquery';

export const jqSelect2 = {
  mounted(el, binding) {
    $(el).select2(binding.value || {});
  },
  beforeUnmount(el) {
    $(el).select2('destroy');
  }
};
```

#### Usage in a Vue Component

```vue
<template>
  <select v-jq-select2="opts">
    <option value="1">Option A</option>
  </select>
</template>
```

---

### Constraints and Limitations

- **Vue must not re-render the jQuery-managed subtree**; use directives or `v-once` boundaries.
- **jQuery plugins must be destroyed** in `beforeUnmount` to prevent leaks.
- **Reactive properties should not hold plugin instances**; store them in plain variables.
- **`$nextTick` is required** when interacting with the DOM after reactive updates.

---

### Annotated Code Example

```javascript
<script setup>
import { onMounted, onBeforeUnmount } from 'vue';
import $ from 'jquery';

let $select;

onMounted(() => {
  $select = $('#mySelect').select2();
  $select.on('change', (e) => {
    // bridge to Vue state
  });
});

onBeforeUnmount(() => {
  $select?.select2('destroy');
});
</script>

<template>
  <select id="mySelect">
    <option value="a">Option A</option>
  </select>
</template>
```

**Expected Output**
Select2 is initialised on mount and destroyed on unmount. Selecting a value updates Vue state through the bridged event handler.

**Why this output occurs**: `onMounted` runs after Vue commits the DOM. The plugin is initialised on the element. `onBeforeUnmount` destroys the plugin before the component is removed, preventing leaks.

---

### Real‑World Cases

- **Logistics SaaS dashboard**: A 120,000-line jQuery dashboard migrated to Vue 3 in phases, starting with the shipment table and state management.
- **Vue 3 progressive migration**: Mounting Vue components into jQuery pages via container elements, then gradually expanding.

---

## Core Concept 4: jQuery to Angular

### Definitions

**Core Definition**
Migrating from jQuery to Angular means abandoning direct DOM manipulation in favour of Angular's model-driven approach, where components, templates, and data binding manage the view, and jQuery is removed from the rendering path.

**Technical Definition**
Angular's most common migration pitfall is that developers continue to think in jQuery terms — "update the page" — rather than "update the model, and let the page reflect it". In Angular, you do not interact with the page directly; you update the component's model, and Angular's change detection updates the view. The migration therefore involves **replacing jQuery widgets with Angular components**, **refactoring initialisation, templates, events, and data binding into Angular patterns**, and **eliminating jQuery DataSource and plugin dependencies** in favour of Angular services and Observables. For incremental migration, the recommended approach is to identify a small, independent set of functionalities (a module) and move it to Angular first, rather than attempting a full rewrite.

**Beginner‑Friendly Explanation**
Angular is like a strict manager who insists on doing things a certain way: you tell the manager what the data should be, and the manager handles the page. jQuery is like a worker who ignores the manager and changes the page directly. The migration is about retraining the worker to follow the manager's rules. You start by identifying one small feature — a widget or a form — and rebuilding it the Angular way. Once that works, you move on to the next feature. The goal is to stop changing the page directly and start changing the model instead.

---

### Purposes

- **To adopt a model-driven architecture** with clear separation of concerns.
- **To eliminate jQuery's DOM ownership conflicts** with Angular's change detection.
- **To migrate incrementally** by moving independent modules to Angular one at a time.
- **To replace jQuery widgets with native Angular components** (e.g., Kendo UI for Angular).
- **To improve testability** through Angular's dependency injection and component architecture.

---

### Syntax Rules and Structure

| jQuery Pattern | Angular Pattern |
|---|---|
| `$('#el').text('x')` | `{{ model }}` in template |
| `$('#el').on('click', fn)` | `(click)="fn()"` in template |
| `$.ajax()` | `HttpClient` with Observables |
| jQuery plugin widget | Angular component with `@Input()`/`@Output()` |
| jQuery DataSource | Angular service with Observables |

---

### Constraints and Limitations

- **Do not manipulate the DOM directly** in Angular; update the model instead.
- **jQuery plugins that assume full-page reloads** may be fundamentally incompatible with Angular's SPA lifecycle.
- **Server-side changes** may be needed to expose APIs for the Angular frontend.

---

### Annotated Code Example

```typescript
// ✅ Angular component replaces jQuery DOM manipulation
@Component({
  selector: 'app-counter',
  template: `
    <button (click)="increment()">Increment</button>
    <span>{{ count }}</span>
  `
})
export class CounterComponent {
  count = 0;

  increment() {
    this.count++; // model update, Angular updates the view
  }
}
```

**Expected Output**
Clicking the button increments the displayed count. Angular updates the view automatically when `count` changes.

**Why this output occurs**: The component updates the `count` property, and Angular's change detection reflects the new value in the template. No direct DOM manipulation is needed.

---

### Real‑World Cases

- **Kendo UI migration**: Replacing Kendo UI for jQuery widgets with Kendo UI for Angular components, refactoring initialisation, templates, events, and data binding.
- **ASP.NET Zero**: Merging an MVC/jQuery application into an Angular solution by upgrading to the latest version first.

---

## Core Concept 5: jQuery to Svelte

### Definitions

**Core Definition**
Migrating from jQuery to Svelte means replacing imperative DOM manipulation with Svelte's compiled, reactive component model, where the compiler generates efficient DOM updates from declarative component definitions.

**Technical Definition**
Svelte compiles components into imperative JavaScript that updates the DOM efficiently, without a virtual DOM. jQuery operates directly on the real DOM, bypassing Svelte's reactivity. The migration follows an **incremental module-by-module approach**: start by migrating JavaScript into a module-styled fashion so that Svelte components can be imported into the old layout. Svelte components can be used inside jQuery-based pages via tools like `svelte-injector`, which allows Svelte components to be mounted in any frontend framework, including jQuery, enabling gradual migration while keeping older code running. jQuery plugins can be used in Svelte by importing jQuery directly (`import jQuery from 'jquery'`) and initialising the plugin in `onMount`, since `$` is a reserved prefix in Svelte.

**Beginner‑Friendly Explanation**
Svelte is like a compiler that turns your component code into highly optimised JavaScript. It does not need a virtual DOM; it updates the page directly but intelligently. jQuery, by contrast, updates the page manually. The migration is about replacing manual updates with Svelte's compiled updates, one component at a time. You can start by putting a Svelte component inside a jQuery page — a small widget or a form — and letting Svelte manage that piece. The rest of the page stays on jQuery until you are ready to migrate more.

---

### Purposes

- **To adopt a compiled, reactive framework** with minimal runtime overhead.
- **To migrate incrementally** by embedding Svelte components in existing pages.
- **To replace jQuery plugins with Svelte-native alternatives** (e.g., DataTables → SvGrid).
- **To improve performance** through Svelte's efficient DOM updates.
- **To simplify state management** with Svelte's built-in reactivity.

---

### Syntax Rules and Structure

```svelte
<script>
  import { onMount } from 'svelte';
  import jQuery from 'jquery'; // ✅ not `$`

  let container;

  onMount(() => {
    const $el = jQuery(container);
    $el.somePlugin();

    return () => {
      $el.somePlugin('destroy');
    };
  });
</script>

<div bind:this={container}></div>
```

---

### Constraints and Limitations

- **`$` is reserved in Svelte**; jQuery must be imported as `jQuery` or an alias.
- **Svelte must not re-render the jQuery-managed subtree**; use `bind:this` boundaries.
- **SvelteKit SSR has no DOM**; jQuery calls must be guarded or placed inside `onMount`.

---

### Annotated Code Example

```svelte
<script>
  import { onMount } from 'svelte';
  import jQuery from 'jquery';

  let tooltipContainer;

  onMount(() => {
    const $el = jQuery(tooltipContainer);
    $el.tooltip({ title: 'Hello from jQuery' });

    return () => {
      $el.tooltip('destroy');
    };
  });
</script>

<div bind:this={tooltipContainer}></div>
```

**Expected Output**
The tooltip is initialised on the container. When the component unmounts, the tooltip is destroyed.

**Why this output occurs**: `onMount` runs after Svelte mounts the empty `<div>`. jQuery initialises the plugin on that node. The cleanup function returned from `onMount` runs on unmount, calling `destroy`.

---

### Real‑World Cases

- **Ashley Home Furniture**: Migrated one of the largest e-commerce websites from jQuery to Svelte and SvelteKit.
- **DataTables replacement**: Migrating from jQuery DataTables to SvGrid, a Svelte-native data grid.
- **Kendo UI with Svelte**: Using Kendo UI for jQuery inside Svelte components, with `import * as $jq from 'jquery'`.

---

## Core Concept 6: jQuery to Alpine.js

### Definitions

**Core Definition**
Migrating from jQuery to Alpine.js means replacing imperative DOM manipulation with Alpine's declarative, HTML-attribute-based reactivity, where behaviour is declared directly in markup and state is managed reactively.

**Technical Definition**
Alpine.js is often called "the new jQuery" because it fills a similar niche: adding interactivity to HTML without a heavy build system. However, Alpine is **declarative** and **reactive**, whereas jQuery is imperative. In jQuery, you write `$('#button').on('click', fn)` and manually update the DOM. In Alpine, you write `<button @click="clicked = true">` and `<p x-show="clicked">`, and Alpine handles the DOM updates automatically. The migration is **incremental**: keep jQuery for the first pass, add Alpine via CDN to test functionality, then replace jQuery behaviours one by one. Most jQuery dependencies can be removed and replaced with native JavaScript, with Alpine handling the reactive UI logic. Magento's Hyvä theme is the canonical large-scale example: it replaces the entire Luma jQuery/Knockout/RequireJS stack with Alpine.js and Tailwind CSS, reducing the JavaScript payload by roughly 90%.

**Beginner‑Friendly Explanation**
Alpine is like jQuery's smarter cousin. It does the same kind of job — adding interactivity to HTML — but it does it by declaring behaviour in the HTML itself. Instead of writing JavaScript to find a button and attach a click handler, you write `@click` on the button. Instead of manually updating text, you bind it with `x-text`. The migration is about moving your logic from imperative JavaScript into declarative HTML attributes. You can start with one component — a dropdown, a toggle, a form — and let Alpine manage it. jQuery stays for everything else until you are ready to replace more.

---

### Purposes

- **To adopt a lightweight, declarative reactivity model** without a build step.
- **To reduce JavaScript bundle size** (Alpine is ~15kb vs. jQuery's ~30kb).
- **To eliminate DOM-based state** by centralising state in Alpine components.
- **To migrate incrementally** by adding Alpine alongside jQuery and replacing behaviours one at a time.
- **To simplify markup** by expressing behaviour directly in HTML attributes.

---

### Syntax Rules and Structure

| jQuery | Alpine.js |
|---|---|
| `$('#el').text('x')` | `x-text="value"` |
| `$('#el').on('click', fn)` | `@click="fn()"` |
| `$('#el').show()` | `x-show="visible"` |
| `$('#el').addClass('active')` | `:class="{ active: isActive }"` |
| Manual `render()` function | `x-for` template loop |

---

### Constraints and Limitations

- **Alpine is declarative**; complex imperative logic may still require vanilla JavaScript.
- **Alpine does not replace jQuery plugins**; plugins must be wrapped or replaced.
- **Alpine's reactivity is component-scoped**; global state requires `Alpine.store`.
- **Large-scale replacements** (e.g., Magento Hyvä) require rebuilding templates, not just JavaScript.

---

### Annotated Code Example

```html
<!-- ❌ jQuery -->
<script>
  let count = 0;
  $('#increment').on('click', function() {
    count++;
    $('#count').text(count);
  });
</script>

<!-- ✅ Alpine.js -->
<div x-data="{ count: 0 }">
  <button @click="count++" id="increment">Increment</button>
  <span x-text="count"></span>
</div>
```

**Expected Output**
Clicking the button increments the displayed count in both versions.

**Why this output occurs**: Alpine's `x-data` declares the reactive state (`count`). `@click="count++"` binds the click handler. `x-text="count"` binds the display. Alpine updates the DOM automatically when `count` changes.

---

### Real‑World Cases

- **Magento Hyvä theme**: Swaps Luma's RequireJS/Knockout/jQuery stack for Alpine.js and Tailwind CSS, reducing the JS payload by ~90%.
- **Hyvä component migration**: Converting Luma-based JavaScript and templates to Alpine-based Hyvä components, replacing jQuery dependencies with native JavaScript and Alpine.
- **Foundation to Atrium**: jQuery skin behaviour (skin switcher, search-as-you-type, mobile drawer) moved to Alpine, with jQuery surviving only as a shim for plugins.

---

## Core Concept 7: jQuery to htmx

### Definitions

**Core Definition**
Migrating from jQuery to htmx means replacing jQuery AJAX calls and DOM updates with htmx's HTML-attribute-driven hypermedia model, where server responses are HTML fragments that htmx swaps into the page declaratively.

**Technical Definition**
htmx is described by its creator as "the new jQuery" — a low-cost, long-lived tool that does not dominate the project. The migration overlaps with jQuery in three areas: **partial page updates** (`$.ajax` + `.html(response)` → `hx-get` + `hx-target` + `hx-swap`), **event handling** (`$(...).on('click', ...)` → `hx-trigger`), and **CSS transitions** (`$(...).fadeIn()` → CSS transitions). The recommended order is: **(1)** inventory all `$(`, `$.ajax`, `$.get`, and `$.post` usage; **(2)** migrate AJAX calls to htmx attributes first, since this yields the largest benefit; **(3)** migrate the remaining jQuery (selectors, class manipulation) to vanilla JavaScript; and **(4)** finally remove the jQuery load entirely. Crucially, htmx adoption and jQuery removal should **not** happen at the same time; the htmx migration should be completed and stabilised before jQuery is removed, so that any breakage can be attributed to the correct change.

**Beginner‑Friendly Explanation**
htmx is like jQuery's successor for AJAX. Instead of writing JavaScript to fetch data and insert it into the page, you write `hx-get` and `hx-target` directly on the HTML element. The server sends back HTML, and htmx puts it in the right place. The migration is about moving your AJAX logic from JavaScript into HTML attributes. It is often the easiest part of a jQuery migration because it removes code rather than adding it. The rule is: do the htmx migration first, stabilise it, and then remove jQuery — not both at once.

---

### Purposes

- **To replace jQuery AJAX with declarative HTML attributes**.
- **To reduce JavaScript code** by moving request logic to the server and markup.
- **To simplify partial page updates** with `hx-target` and `hx-swap`.
- **To migrate incrementally** without a full framework rewrite.
- **To eliminate jQuery's AJAX and event-handling code** in favour of standard HTML.

---

### Syntax Rules and Structure

| jQuery | htmx |
|---|---|
| `$.get('/posts/', data, fn)` | `hx-get="/posts/"` |
| `$('#list').html(response)` | `hx-target="#list" hx-swap="innerHTML"` |
| `$(...).on('click', ...)` | `hx-trigger="click"` |
| `$(...).fadeIn()` | CSS transitions |
| `$(...).addClass()` | `classList.add` (vanilla JS) |

---

### Constraints and Limitations

- **htmx does not replace jQuery plugins**; plugins must be migrated to vanilla JS or retained temporarily.
- **htmx requires server-rendered HTML fragments**; it is not suitable for JSON APIs without a server-side adapter.
- **Do not migrate htmx and remove jQuery simultaneously**; stabilise htmx first.
- **`$.ajaxSetup` and jQuery Validation** must be handled separately, not removed in the same step as htmx adoption.

---

### Annotated Code Example

```javascript
// ❌ jQuery AJAX
$('#search-btn').on('click', function() {
  $.get('/posts/', $('#search-form').serialize(), function(html) {
    $('#post-list').html(html);
  });
});
```

```html
<!-- ✅ htmx -->
<form id="search-form" hx-get="/posts/" hx-target="#post-list" hx-swap="innerHTML">
  <button type="submit" id="search-btn">Search</button>
</form>
<div id="post-list"></div>
```

**Expected Output**
Submitting the form fetches `/posts/` and inserts the response into `#post-list`. No JavaScript is required.

**Why this output occurs**: htmx processes the `hx-get`, `hx-target`, and `hx-swap` attributes at page load. The form submission is intercepted by htmx, the request is sent, and the response is swapped into the target element.

---

### Real‑World Cases

- **Drupal core**: A proposal to use htmx to power AJAX behaviours, replacing one of the largest consumers of jQuery in core.
- **Forgejo/Gitea**: Removed jQuery AJAX calls from the repo editor and code diff expansion buttons, replacing them with htmx.
- **Django + htmx**: A documented migration path from jQuery to htmx, with a clear order: list all jQuery usage, migrate AJAX first, then standard JavaScript, then remove jQuery.

---

## Core Concept 8: Incremental Migration

### Definitions

**Core Definition**
Incremental migration is the practice of moving an application from jQuery to a modern target in small, independent, verifiable steps — replacing one route, page, module, or component at a time — while keeping the application functional and shippable throughout the transition.

**Technical Definition**
Incremental migration distributes risk across many small releases rather than concentrating it into a single high-stakes cutover. Each increment is scoped by a **seam** — a route boundary, an iframe boundary, a distinct page load, or a module boundary — where legacy and new code can coexist without deep coupling. The migration proceeds **business-risk-first**: low-risk surfaces (internal admin pages, isolated widgets) are migrated before high-risk surfaces (customer checkout flows). Test coverage is established before refactoring begins, and rollback procedures are defined for each phase. The migration is complete when the legacy code paths are deprecated and removed, and the new architecture is documented.

**Beginner‑Friendly Explanation**
Incremental migration is like replacing the parts of a car while it is still driving. You do not take the engine out and hope for the best; you replace one part at a time, test it, and move on to the next part. Each part is a small, contained change that you can verify and, if necessary, roll back. This is safer than a big-bang rewrite, which is like taking the whole car apart and hoping it goes back together correctly. Incremental migration keeps the product shippable and the team sane.

---

### Purposes

- **To distribute risk** across many small, reversible changes.
- **To keep the application shippable** throughout the migration.
- **To preserve tested behaviour** by migrating one slice at a time and diffing behaviour.
- **To maintain feature velocity** by avoiding a feature freeze.
- **To enable rollback** at each phase if something goes wrong.

---

### Syntax Rules and Structure

| Phase | Activity |
|---|---|
| **Assessment** | Inventory jQuery usage, map dependencies, identify high-risk areas, define success metrics. |
| **Safety net** | Add characterisation tests, feature flags, compatibility layers, and rollback procedures. |
| **Incremental execution** | Replace components one at a time using the Strangler Fig pattern; maintain backward compatibility; run old and new paths in parallel. |
| **Stabilisation** | Run regression suites, monitor adoption and error rates, deprecate legacy paths, document the new architecture. |

---

### Constraints and Limitations

- **Dual-stack overhead** is real: two build pipelines, two deploy paths, and two sets of dependencies must be maintained during the migration.
- **Seams may not exist** in tightly coupled codebases; in that case, a bounded rewrite of a small surface may be necessary.
- **Shared auth/session models** must be solved before coexistence begins.

---

### Annotated Code Example

```javascript
// Phase 1: Introduce the new framework alongside jQuery
// Mount a React component into a jQuery-managed page
$('#react-widget').each(function() {
  ReactDOM.createRoot(this).render(<NewWidget />);
});

// Phase 2: Migrate a second widget
$('#react-widget-2').each(function() {
  ReactDOM.createRoot(this).render(<SecondWidget />);
});

// Phase 3: Once all widgets are migrated, remove jQuery
// (jQuery load is removed from the page template)
```

**Expected Output**
Each new React widget is mounted into its container. jQuery continues to manage the rest of the page until all widgets are migrated and jQuery is removed.

**Why this output occurs**: The incremental approach introduces React at the component level, leaving the existing jQuery code untouched. Each migration step is independent and verifiable.

---

### Real‑World Cases

- **Vue 3 logistics dashboard**: Phased migration starting with the shipment table, then state management, then routing, over multiple releases.
- **React banking platform**: Strangler Fig migration with zero downtime, using React component libraries and incremental traffic routing.
- **Magento Hyvä**: Ground-up frontend replacement, but executed as a full theme swap rather than incremental coexistence, because the Luma stack is fundamentally incompatible with Alpine.

---

## Core Concept 9: The Strangler Pattern

### Definitions

**Core Definition**
The Strangler Fig pattern is a migration strategy where the new system is built around the edges of the old system and gradually replaces it — route by route, module by module — until the old system can be removed entirely.

**Technical Definition**
Named after the strangler fig vine that gradually envelops and eventually replaces its host tree, the pattern was described by Martin Fowler as an alternative to big-bang rewrites. The new system is built alongside the old one, and a **routing layer** or **adapter layer** directs traffic to either the legacy or the new implementation depending on the route, feature, or user segment. The old system continues to operate until every surface has been migrated, at which point it is decommissioned. The Strangler Fig pattern is the **default recommendation** for jQuery migrations because it distributes risk, preserves tested behaviour, and avoids feature freezes. A **bounded rewrite** is recommended only when no viable seam exists, the surface is small and isolated, and the team explicitly accepts the discovery risk.

**Beginner‑Friendly Explanation**
The Strangler Fig pattern is like building a new road next to an old road and gradually diverting traffic to the new road, one lane at a time. The old road stays open until every car is using the new road. Then you close the old road. In software, the new system grows around the old one, and a routing layer decides which system handles each request. This is safer than closing the old road and hoping the new road works. The Strangler Fig is the default way to migrate from jQuery because it lets you keep shipping features while you modernise.

---

### Purposes

- **To distribute migration risk** across many small cutovers.
- **To preserve business logic** embedded in the legacy system.
- **To maintain feature velocity** during the migration.
- **To enable rollback** at the route or feature level.
- **To avoid the failure modes of big-bang rewrites** (feature freeze, concentrated risk, knowledge loss).

---

### Syntax Rules and Structure

#### Routing-Level Strangler

```
┌──────────────────┐
│   Routing Layer  │
│  (nginx, CDN,    │
│   framework)     │
└────────┬─────────┘
         │
    ┌────┴─────┐
    │          │
┌───▼────┐  ┌──▼───┐
│ Legacy │  │ New  │
│ jQuery │  │React │
└────────┘  └──────┘
```

**Rule**: New routes go to the modern app; old routes stay in the legacy app. Migrate one route at a time until the legacy app is empty.

#### Module-Level Strangler

```
Legacy jQuery module  →  New module (React/Vue/etc.)
Legacy jQuery module  →  New module
...
Legacy jQuery module  →  Decommissioned
```

---

### Constraints and Limitations

- **No seams at all** is the strongest signal toward a bounded rewrite of that specific surface.
- **Dual-stack operational overhead** must be sustainable for the duration of the migration.
- **Shared auth/session models** must be solved before coexistence begins.

---

### Annotated Code Example (Routing-Level Strangler)

```javascript
// server.js (Express)
const LEGACY_ROUTES = ['/dashboard', '/reports', '/settings'];
const NEW_ROUTES = ['/new-dashboard', '/new-reports'];

app.use((req, res, next) => {
  if (NEW_ROUTES.some(route => req.path.startsWith(route))) {
    // Serve the new React app
    return newAppHandler(req, res, next);
  }
  if (LEGACY_ROUTES.some(route => req.path.startsWith(route))) {
    // Serve the legacy jQuery app
    return legacyAppHandler(req, res, next);
  }
  next();
});
```

**Expected Output**
Requests to `/new-dashboard` are served by the React app. Requests to `/dashboard` are served by the legacy jQuery app. The routing layer directs traffic based on the route.

**Why this output occurs**: The middleware checks the request path against the new and legacy route lists and dispatches to the appropriate handler. As routes are migrated, they are moved from `LEGACY_ROUTES` to `NEW_ROUTES`.

---

### Real‑World Cases

- **Banking platform**: Strangler Fig migration from jQuery monoliths to React, with zero downtime and incremental traffic routing.
- **Enterprise UI modernisation**: Route-level strangler fig with both apps coexisting, migrating one route at a time.
- **Vue 3 logistics dashboard**: Feature-by-feature strangling, starting with low-risk components and expanding.

---

## Core Concept 10: Migration Best Practices

### Definitions

**Core Definition**
jQuery migration best practices are the consolidated conventions — audit before migrating, establish tests first, use the Strangler Fig pattern, migrate in risk order, maintain dual-stack discipline, and document the transition — that make the migration safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual strategies: **(1)** audit current jQuery usage comprehensively, including plugins and global state; **(2)** establish characterisation tests before refactoring; **(3)** default to the Strangler Fig pattern, scoped by seams (routes, modules, iframes); **(4)** sequence migrations business-risk-first (low-risk surfaces first); **(5)** maintain backward compatibility at every step and run old and new paths in parallel; **(6)** define rollback procedures for each phase; **(7)** monitor for regressions continuously and use feature flags for gradual rollout; and **(8)** document the migration decisions and the new architecture.

**Beginner‑Friendly Explanation**
Best practices are the rules that keep a migration from turning into a disaster. Start by understanding what you have — count the jQuery calls, list the plugins, find the global state. Write tests before you change anything, so you know when you break something. Use the Strangler Fig pattern: replace one small piece at a time, test it, and move on. Migrate the easy, low-risk stuff first. Keep the old and new systems running side by side until the new one is proven. Have a rollback plan. Watch for errors. And write down what you did, so the next person can understand it. These habits turn a risky, stressful migration into a controlled, manageable process.

---

### Purposes

- **To provide a single actionable checklist** for jQuery migration.
- **To prevent the most common migration failures** (big-bang rewrites, untested refactoring, feature freezes).
- **To balance migration speed with operational stability**.
- **To establish a baseline for planning and executing the migration**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Audit** | Inventory all jQuery usage, plugins, and global state | Starting without knowing the scope |
| **Tests** | Characterisation tests before refactoring | Refactoring untested code |
| **Pattern** | Strangler Fig, scoped by seams | Big-bang rewrite |
| **Sequencing** | Low-risk surfaces first | Starting with the highest-risk flow |
| **Coexistence** | Old and new paths run in parallel | Cutting over without verification |
| **Rollback** | Defined rollback procedure per phase | No way to undo a failed migration |
| **Monitoring** | Continuous regression monitoring | Assuming it works without checking |
| **Documentation** | Document decisions and new architecture | Leaving the next team guessing |

---

### Constraints and Limitations

- **Best practices must be adapted to context**; a small internal tool has different risk tolerances than a customer-facing checkout.
- **Strangler migrations require sustained dual-stack capacity**; if the team cannot maintain two stacks, the sequencing must change.
- **Some legacy plugins are fundamentally incompatible** with modern frameworks; a bounded rewrite of that specific surface may be the only viable option.

---

### Annotated Code Example (Consolidated Migration Checklist)

```text
✅ 1. Audit: count $(, $.ajax, $.get, $.post; list plugins; find global state
✅ 2. Tests: add characterisation tests for the surfaces to be migrated
✅ 3. Seams: identify route, module, and iframe boundaries
✅ 4. Strangler: set up routing layer; new routes → new app, old routes → legacy
✅ 5. Sequence: migrate low-risk routes/widgets first
✅ 6. Coexist: run old and new paths in parallel; compare behaviour
✅ 7. Rollback: feature flags and rollback procedures for each phase
✅ 8. Monitor: regression suites, error rates, adoption metrics
✅ 9. Decommission: remove legacy code paths; document new architecture
```

**Expected Output**
A structured, risk-managed migration that proceeds in verifiable increments and ends with the legacy system fully decommissioned.

**Why this output occurs**: Each step addresses a specific migration risk: scope uncertainty (audit), behaviour regression (tests), concentrated risk (strangler), operational disruption (coexistence and rollback), and incomplete migration (decommission and documentation).

---

### Real‑World Cases

- **Vue 3 logistics dashboard**: Audited 120,000 lines of jQuery, migrated in phases (table → state → routing), with performance improvements measured 60 days after cutover.
- **React banking platform**: Strangler Fig with zero downtime, using incremental traffic routing and React component libraries.
- **Magento Hyvä**: Full frontend replacement with a documented migration checklist for scoping and acceptance.

---

## References

- Migrate jQuery to Vanilla JavaScript | Aviator Runbooks — https://github.com/aviator-co/runbooks-library/blob/master/templates/migrate_jquery_to_vanilla_javascript.md
- Migrating from jQuery to Vanilla JavaScript | GUVI — https://www.guvi.in/blog/migrating-from-jquery-to-vanilla-javascript/
- Hybrid jQuery React Interoperability Strategies During Incremental Frontend Framework Migration | IJCTECE — https://ijctece.com/index.php/IJCTEC/article/download/716/651/1381
- jQuery to Vue 3 Migration: How We Scaled a Legacy App | Bacancy Technology — https://www.bacancytechnology.com/insights/jquery-to-vue-3-migration
- 重构旧项目：从 jQuery 迁移到 Vue 3 的分步方案与避坑指南 | Tencent Cloud — https://cloud.tencent.com.cn/developer/article/2601178
- Do you understand that with Angular you read and write to the model... never to the page? | SSW — https://www.ssw.com.au/rules/angular-read-and-write-to-the-model
- Angular Migrating from jQuery to Angular | Kendo UI — https://www.telerik.com/kendo-angular-ui/components/migrating-from-jquery
- Importing jQuery in SvelteKit | Stack Overflow — https://stackoverflow.com/questions/71017937/importing-jquery-in-sveltekit
- Is there a way to gradually migrate to Svelte? | Stack Overflow — https://stackoverflow.com/questions/70859769/is-there-a-way-to-gradually-migrate-to-svelte
- Alpine.js vs jQuery | Codeberg — https://codeberg.org/ImL1s/TodoListDemo/src/commit/7661651d231aec92de4d9047658df9492bfaed3d/03-modern-frameworks/17-alpinejs
- Migrating JS & Templates | Hyvä Docs — https://docs.hyva.io/hyva-themes/working-with-javascript/migrating-js-templates.html
- Magento 2 Luma to Hyvä Migration Guide | Rock Technolabs — https://rocktechnolabs.com/magento-2-luma-to-hyva-migration-guide/
- 02. jQuery에서 넘어오기 — 실전 Django와 htmx — https://text.ibetter.kr/django-htmx/part-08-도입-판단과-공존/02장-기존-자산과의-공존/02-jquery에서-넘어오기
- 03. 기존 jQuery 설정과의 관계 — 실전 Django와 htmx — https://text.ibetter.kr/django-htmx/part-08-도입-판단과-공존/03장-기존-자산과의-공존/03-jquery-설정과의-관계
- Strangler Fig & Codemods | Steve Kinney — https://stevekinney.com/courses/enterprise-ui/strangler-fig-and-codemods
- Rewrite vs. Incremental Strangler Decision | Vanguard Frontier Agentic — https://github.com/VincentChuWaiChow/vanguard-frontier-agentic/blob/master/skills/frontend/frontend-migration-modernization-plan/references/rewrite-vs-incremental-decision.md
- Legacy Modernization | Claude Cortex — https://github.com/nickcrew/claude-cortex/blob/main/skills/legacy-modernization/SKILL.md
- Modernizing a Legacy Banking Platform: Strangler Fig Approach | JISEM — https://www.jisem-journal.com
- jQuery to React Migration: Incremental, SEO-Safe | VooStack — https://voostack.com/blog/jquery-to-react-migration