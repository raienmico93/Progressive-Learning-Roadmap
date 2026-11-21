# jQuery in Vue — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Integrating jQuery into a Vue application is the practice of allowing legacy jQuery code — DOM manipulation, plugins, event handlers, and AJAX calls — to coexist with Vue’s reactive, component‑based rendering model, typically during an incremental migration from a jQuery‑based codebase to Vue or when a specific third‑party plugin has no Vue equivalent.

**Technical Definition**
Vue and jQuery have fundamentally incompatible reactivity and DOM ownership models. Vue maintains a virtual DOM and a reactive dependency‑tracking system; it patches the real DOM to match the virtual representation when reactive state changes. jQuery operates directly on the real DOM, bypassing Vue’s reactivity system entirely. When jQuery modifies a DOM node that Vue also manages, Vue’s virtual DOM becomes inconsistent with the actual DOM, leading to lost updates, stale data, or thrown exceptions during patch operations. The official Vue guidance and the GitLab frontend guide both state that mixing Vue and jQuery is **not recommended**; when a specific jQuery plugin must be used, the recommended approach is to **wrap it in a Vue component** and confine jQuery to a `ref`‑attached container that Vue does not update. The integration therefore requires explicit boundaries: Vue owns the component tree and reactive state, jQuery owns isolated DOM islands created via template refs, and communication occurs through props, callbacks, and lifecycle‑managed initialisation and cleanup.

**Beginner‑Friendly Explanation**
Vue and jQuery both want to control the page, but they do it in completely different ways. jQuery says: “I will find the element and change it directly.” Vue says: “I will describe what the page should look like based on my data, and I will handle all the changes myself.” If you let both change the same part of the page, they fight, and things break — Vue may overwrite jQuery’s changes, or jQuery may make Vue’s data out of sync with what the user sees. The solution is to give each one its own territory. Vue owns the overall page and the components. Inside a component, you create a small empty container and tell jQuery: “This box is yours — do whatever you want inside it.” Vue promises not to touch that box. That way, jQuery plugins and legacy code can keep working while the rest of the application moves to Vue.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Conflicting DOM ownership** | Vue patches the DOM through its virtual DOM; jQuery mutates the real DOM directly. Concurrent control of the same nodes causes corruption or stale data. |
| **Reactivity is bypassed** | jQuery plugins maintain their own internal state and do not automatically respond to Vue’s reactive data changes; Vue does not track jQuery‑initiated DOM mutations. |
| **Isolation is mandatory** | jQuery must be confined to a template‑ref container that Vue does not update; the container should be empty from Vue’s perspective. |
| **Lifecycle‑managed initialisation** | jQuery plugins must be initialised after the DOM is committed (`mounted` / `onMounted`) and destroyed before unmount (`beforeDestroy` / `onBeforeUnmount`). |
| **Separate event systems** | Vue uses its own event system; jQuery events do not automatically trigger Vue handlers, and vice versa. Bridging must be explicit. |
| **Migration tool, not a destination** | jQuery in Vue is appropriate for wrapping legacy plugins and incremental migration, not for new development. |

---

### Prerequisites

- Working knowledge of Vue components, template refs (`ref`), and lifecycle hooks (`mounted`, `beforeUnmount`).
- Working knowledge of jQuery selection, DOM manipulation, events, and plugins.
- Understanding of the virtual DOM and Vue’s reactivity system.
- Awareness that Vue’s Composition API (`setup`) has no `this`, which affects how jQuery callbacks are written.

---

### Related Programming Areas

- **DOM performance** — Vue and jQuery both manipulate the DOM; batching and isolation affect performance.
- **Event performance** — Vue’s event system vs. jQuery’s event system.
- **Memory performance** — plugins must be destroyed to prevent leaks in SPA contexts.
- **Security** — jQuery sinks (`.html()`) inside Vue components reintroduce XSS risk.
- **Dependency security** — legacy jQuery versions carry known CVEs.

---

### Core Concepts / Features (Index)

1. jQuery in Vue (General)
2. Vue Refs (`$refs` / `ref()`)
3. Vue Lifecycle Hooks (`mounted`, `beforeUnmount`, `nextTick`)
4. DOM Manipulation Boundaries
5. jQuery Plugins in Vue
6. jQuery Events in Vue
7. jQuery AJAX in Vue
8. Integration Challenges
9. Integration Best Practices
10. When to Avoid jQuery in Vue


## Core Concept 1: jQuery in Vue (General)

### Definitions

**Core Definition**
jQuery in Vue is the coexistence of the jQuery library with Vue components, where jQuery is used for specific isolated tasks — typically legacy plugins or DOM operations that Vue does not cover — while Vue remains responsible for the component tree and reactive state management.

**Technical Definition**
Vue’s reactivity system tracks dependencies and schedules DOM updates asynchronously. jQuery’s operations happen outside this system: when jQuery modifies the DOM, Vue does not know about it, and when Vue re‑renders, it may overwrite jQuery’s changes. The GitLab frontend guide states clearly: “Mixing Vue and jQuery is not recommended. To use a specific jQuery plugin in Vue, create a wrapper around it. It is acceptable for Vue to listen to existing jQuery events using jQuery event listeners. It is not recommended to add new jQuery events for Vue to interact with jQuery”. The wrapper pattern renders an empty container, obtains the real DOM node via a template ref, initialises the plugin in `mounted` (or `onMounted`), and destroys it in `beforeDestroy`/`beforeUnmount`.

**Beginner‑Friendly Explanation**
jQuery in Vue is a compromise. You are saying: “Most of this page is Vue, but this one part is still jQuery.” That is fine as long as the two parts stay in their own lanes. The moment jQuery reaches into a part of the page that Vue is also changing, you get bugs that are hard to diagnose — Vue may wipe out jQuery’s changes, or jQuery may make Vue’s data out of sync with what the user sees. The general rule is: use jQuery only inside a dedicated container that Vue does not touch, and only for things that Vue cannot easily do — usually wrapping an existing jQuery plugin.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them from scratch.
- **To migrate incrementally** from a jQuery codebase to Vue, one component at a time.
- **To isolate legacy DOM manipulation** inside a boundary that Vue does not manage.
- **To bridge event and state changes** between Vue props and jQuery plugin APIs.
- **To avoid the cost of a full rewrite** while still moving toward Vue.

---

### Syntax Rules and Structure

#### General Isolation Pattern (Options API)

```javascript
import $ from "jquery";

export default {
  mounted() {
    const $el = $(this.$refs.container);
    $el.somePlugin();                     // initialise

    // cleanup
    this.$once("hook:beforeDestroy", () => {
      $el.somePlugin("destroy");
    });
  },
  template: `<div ref="container"></div>`
};
```

#### General Isolation Pattern (Composition API, Vue 3)

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  setup() {
    const containerRef = ref(null);

    onMounted(() => {
      const $el = $(containerRef.value);
      $el.somePlugin();
    });

    onBeforeUnmount(() => {
      const $el = $(containerRef.value);
      $el.somePlugin("destroy");
    });

    return { containerRef };
  },
  template: `<div ref="containerRef"></div>`
};
```

**Component Breakdown**

| Component | Description |
|---|---|
| `ref="container"` / `ref(null)` | Creates a reference to the DOM node. |
| `mounted` / `onMounted` | Runs after Vue commits the DOM; jQuery initialises here. |
| `beforeDestroy` / `onBeforeUnmount` | Runs before Vue removes the component; jQuery cleans up here. |
| `<div ref="…"></div>` | Vue renders an empty container that it will not update. |

---

### Constraints and Limitations

- **Vue must not update the jQuery‑managed subtree**; render an empty container.
- **jQuery plugins must be destroyed** to prevent memory leaks in SPAs.
- **Vue and jQuery event systems are separate**; events do not automatically propagate between them.
- **jQuery version must be compatible**; legacy jQuery may carry known CVEs.

---

### Annotated Code Example

```javascript
import $ from "jquery";

export default {
  data() {
    return { message: "Hello from Vue" };
  },
  mounted() {
    // Step 1: Initialise the jQuery plugin on the real DOM node
    $(this.$refs.tooltip).tooltip({ title: this.message });
  },
  beforeDestroy() {
    // Step 2: Destroy the plugin before the component is removed
    $(this.$refs.tooltip).tooltip("destroy");
  },
  template: `<div ref="tooltip"></div>`
};
```

**Expected Output**
A tooltip is initialised on the container. When the component is destroyed, the tooltip is destroyed and its event listeners are removed.

**Why this output occurs**: `mounted` runs after Vue commits the empty `<div>` to the DOM. jQuery initialises the plugin on that node. `beforeDestroy` runs before Vue removes the component, allowing the plugin to clean up.

---

### Real‑World Cases

- **Legacy admin panels**: Wrapping a jQuery data table or date picker inside a Vue route during incremental migration.
- **Third‑party widgets**: Embedding a jQuery‑based chart or map library that has no Vue equivalent.
- **Gradual migration**: Moving one page at a time from a jQuery‑rendered application to Vue.

---

## Core Concept 2: Vue Refs (`$refs` / `ref()`)

### Definitions

**Core Definition**
A Vue template ref is a mechanism for obtaining a direct reference to a DOM element or child component after it has been rendered. In the Options API, refs are accessed via `this.$refs`; in the Composition API, a `ref()` is declared and attached to the template, and its `.value` property holds the DOM node after mount.

**Technical Definition**
Template refs are Vue’s supported escape hatch for direct DOM access. When the `ref` attribute is placed on an element, Vue registers the element in the component instance’s `$refs` object (Options API) or assigns it to the corresponding `ref` variable (Composition API). The ref is populated only after the component has been mounted; it is `null` before mount and is cleared after unmount. For jQuery integration, the ref provides the real DOM node that jQuery needs, while Vue retains ownership of the component and its rendering logic. The ref itself does not cause re‑renders when changed, which is desirable for a stable connection to the DOM.

**Beginner‑Friendly Explanation**
A Vue ref is a way to get your hands on the actual HTML element that Vue created. Normally, Vue hides the real DOM from you and asks you to work through data and templates. But jQuery needs the real element — it cannot work with Vue’s virtual description. A ref gives you that real element. You attach the ref to a `<div>`, and after Vue renders it, the ref points to the actual `<div>` in the page. You can then give that element to jQuery. The ref itself does not cause re‑renders when it changes, which is exactly what you want for a stable connection to the DOM.

---

### Purposes

- **To obtain the real DOM node** that jQuery needs for selection and manipulation.
- **To maintain a stable reference** to the container across re‑renders.
- **To avoid querying the DOM** with global selectors inside Vue components.
- **To pass the DOM node to a jQuery plugin** during initialisation.
- **To store jQuery objects or plugin instances** without triggering re‑renders.

---

### Syntax Rules and Structure

#### Options API

```javascript
export default {
  mounted() {
    const $el = $(this.$refs.myElement);
  },
  template: `<div ref="myElement"></div>`
};
```

#### Composition API (Vue 3)

```javascript
import { ref, onMounted } from "vue";

export default {
  setup() {
    const myElement = ref(null);

    onMounted(() => {
      const $el = $(myElement.value);
    });

    return { myElement };
  },
  template: `<div ref="myElement"></div>`
};
```

**Component Breakdown**

| Component | Description |
|---|---|
| `ref="myElement"` (template) | Registers the element with Vue. |
| `this.$refs.myElement` (Options API) | The real DOM element after mount. |
| `const myElement = ref(null)` (Composition API) | A reactive ref whose `.value` holds the DOM node. |
| `myElement.value` | The real DOM element after mount. |

---

### Constraints and Limitations

- **Refs are `null` before mount**; access them only inside `mounted`/`onMounted` or later.
- **Refs are not reactive in the jQuery sense**; mutating the DOM node does not trigger Vue re‑renders.
- **Refs should not be overused**; prefer Vue’s declarative data flow for state, and use refs only for DOM access.

---

### Annotated Code Example

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  setup() {
    const buttonRef = ref(null);

    onMounted(() => {
      // Step 1: Wrap the real DOM node
      const $btn = $(buttonRef.value);

      // Step 2: Bind a jQuery event handler
      $btn.on("click.jqueryWrapper", function () {
        alert("Button clicked via jQuery");
      });
    });

    onBeforeUnmount(() => {
      // Step 3: Cleanup
      $(buttonRef.value).off(".jqueryWrapper");
    });

    return { buttonRef };
  },
  template: `<button ref="buttonRef">Click me</button>`
};
```

**Expected Output**
Clicking the button triggers the jQuery‑bound alert. The handler is removed on unmount.

**Why this output occurs**: `buttonRef.value` provides the real `<button>` DOM node. jQuery binds a namespaced click handler. The cleanup removes only the namespaced handler.

---

### Real‑World Cases

- **Wrapping a jQuery plugin** that requires a DOM element to initialise.
- **Reading layout measurements** (e.g., `offsetHeight`) from a Vue‑rendered element.
- **Integrating with a non‑Vue library** that needs a DOM node.

---

## Core Concept 3: Vue Lifecycle Hooks (`mounted`, `beforeUnmount`, `nextTick`)

### Definitions

**Core Definition**
Vue lifecycle hooks are functions that run at specific stages of a component’s existence. For jQuery integration, the relevant hooks are `mounted` (or `onMounted`), which runs after the component’s DOM has been inserted, `beforeDestroy`/`beforeUnmount` (or `onBeforeUnmount`), which runs before the component is removed, and `nextTick`, which defers execution until after the next DOM update cycle.

**Technical Definition**
`mounted` is the point at which the component’s DOM is available; it is the Vue equivalent of jQuery’s `$(document).ready()` for a component’s own subtree. jQuery plugins should be initialised in `mounted` because the DOM node does not exist before that point. `beforeDestroy` (Vue 2) / `beforeUnmount` (Vue 3) is where plugins should be destroyed and event handlers removed to prevent memory leaks. `nextTick` defers a callback until after Vue has flushed pending DOM updates; it is essential when a jQuery plugin must be re‑initialised or updated after reactive data changes, because Vue’s DOM updates are asynchronous.

**Beginner‑Friendly Explanation**
Vue lifecycle hooks are the “before” and “after” moments of a component’s life. `mounted` means “the component is now on the page” — that is when jQuery can safely find and enhance its elements. `beforeUnmount` means “the component is about to be removed” — that is when jQuery must clean up. `nextTick` means “wait until Vue has finished updating the page” — that is when you must re‑initialise a jQuery plugin after Vue has changed the DOM. If you try to use jQuery before `mounted`, the elements do not exist yet. If you forget to clean up in `beforeUnmount`, the plugin stays in memory after the component is gone.

---

### Purposes

- **To initialise jQuery plugins after the DOM is committed** (`mounted` / `onMounted`).
- **To destroy plugins and remove handlers before unmount** (`beforeDestroy` / `onBeforeUnmount`).
- **To re‑initialise or update plugins after Vue’s DOM updates** (`nextTick`).
- **To avoid accessing `$refs` before they are available**.
- **To prevent memory leaks** in single‑page applications.

---

### Syntax Rules and Structure

#### Options API

```javascript
export default {
  mounted() {
    $(this.$refs.el).somePlugin();
  },
  beforeDestroy() {
    $(this.$refs.el).somePlugin("destroy");
  }
};
```

#### Composition API

```javascript
import { onMounted, onBeforeUnmount } from "vue";

export default {
  setup() {
    const el = ref(null);

    onMounted(() => {
      $(el.value).somePlugin();
    });

    onBeforeUnmount(() => {
      $(el.value).somePlugin("destroy");
    });

    return { el };
  }
};
```

#### Using `nextTick`

```javascript
import { nextTick } from "vue";

watch(someReactiveValue, async () => {
  await nextTick();                     // wait for Vue’s DOM update
  $(el.value).somePlugin("update");
});
```

---

### Constraints and Limitations

- **`mounted` does not guarantee that child components’ asynchronous data has loaded**; use `nextTick` or watchers for dynamic content.
- **Vue 3 renamed `beforeDestroy` to `beforeUnmount`**; use the correct hook for your Vue version.
- **`nextTick` is asynchronous**; use `await` or a callback.

---

### Annotated Code Example (Chosen Plugin Wrapper)

```javascript
import { ref, onMounted, onBeforeUnmount, watch, nextTick } from "vue";
import $ from "jquery";
import "chosen-js/chosen.jquery.js";

export default {
  props: { modelValue: String, options: Array },
  emits: ["update:modelValue"],
  setup(props, { emit }) {
    const selectRef = ref(null);

    onMounted(() => {
      const $select = $(selectRef.value);
      $select.chosen();

      $select.on("change.chosenWrapper", function () {
        emit("update:modelValue", this.value);
      });
    });

    onBeforeUnmount(() => {
      const $select = $(selectRef.value);
      $select.off(".chosenWrapper");
      $select.chosen("destroy");
    });

    // Sync prop changes to the plugin
    watch(() => props.modelValue, async (newVal) => {
      await nextTick();
      $(selectRef.value).val(newVal).trigger("chosen:updated");
    });

    return { selectRef };
  },
  template: `
    <select ref="selectRef" :value="modelValue">
      <option v-for="opt in options" :key="opt" :value="opt">{{ opt }}</option>
    </select>
  `
};
```

**Expected Output**
The `<select>` is enhanced by Chosen. Selecting an option emits `update:modelValue`. When the prop changes externally, the plugin is updated after Vue’s DOM update.

**Why this output occurs**: `onMounted` initialises Chosen and binds a namespaced change handler. `onBeforeUnmount` cleans up. The watcher uses `nextTick` to wait for Vue’s DOM update before syncing the value to the plugin.

---

### Real‑World Cases

- **Date pickers**: Initialising a jQuery date picker on a Vue‑rendered input.
- **Rich text editors**: Wrapping a legacy jQuery‑based WYSIWYG editor.
- **Data tables**: Initialising a jQuery DataTable on a Vue‑rendered table.

---

## Core Concept 4: DOM Manipulation Boundaries

### Definitions

**Core Definition**
DOM manipulation boundaries are explicit regions of the page where either Vue or jQuery has exclusive control, preventing the two libraries from mutating the same nodes and corrupting each other’s internal state.

**Technical Definition**
Vue’s reactivity system requires a clear **DOM ownership boundary**; third‑party libraries that directly modify the DOM must be confined to regions that Vue does not manage. The boundary pattern renders an **empty container** (`<div ref="…"></div>`) that Vue has no reason to update, and jQuery manages everything inside that container. The boundary must be respected in both directions: Vue must not render children into the jQuery container, and jQuery must not reach outside it. When a third‑party library modifies DOM that Vue also controls, Vue’s patch process may fail or produce inconsistent results; the boundary prevents this by giving jQuery a territory that Vue promises not to touch.

**Beginner‑Friendly Explanation**
Imagine two chefs in one kitchen. One chef (Vue) insists on controlling every pot and pan. The other chef (jQuery) just grabs whatever is nearby and starts cooking. If they both grab the same pan, disaster. The solution is to give each chef their own counter. Vue gets the main kitchen; jQuery gets a small separate counter (the empty container). Vue promises never to touch jQuery’s counter, and jQuery promises to stay on its own counter. That is a DOM manipulation boundary.

---

### Purposes

- **To prevent Vue and jQuery from mutating the same nodes**.
- **To ensure Vue’s virtual DOM remains consistent** with the real DOM.
- **To isolate jQuery plugins** inside a container that Vue does not update.
- **To allow legacy code to run safely** alongside Vue components.
- **To make the integration predictable** and easier to debug.

---

### Syntax Rules and Structure

```javascript
// ✅ Vue renders an empty container; jQuery owns its contents
template: `<div ref="container"></div>`

// ❌ Vue renders children into the jQuery container
template: `<div ref="container">{{ vueContent }}</div>`
```

**Rule**: The container that jQuery manages must be **empty from Vue’s perspective**. Vue must not render children into it, and must not update its attributes or content.

---

### Constraints and Limitations

- **The boundary must be absolute**; even a single Vue update to the jQuery container breaks the guarantee.
- **Communication must go through props and events**, not through shared DOM nodes.
- **The boundary adds a wrapper element** to the DOM, which may affect layout or styling.

---

### Annotated Code Example

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  setup() {
    const chartRef = ref(null);

    onMounted(() => {
      $(chartRef.value).someChart({ data: [1, 2, 3] });
    });

    onBeforeUnmount(() => {
      $(chartRef.value).someChart("destroy");
    });

    return { chartRef };
  },
  // ✅ Vue renders an empty container — jQuery owns the inside
  template: `<div ref="chartRef" class="chart-boundary"></div>`
};
```

**Expected Output**
Vue renders an empty `<div>`. jQuery draws the chart inside it. Vue never touches the chart’s internal DOM.

**Why this output occurs**: The container is empty from Vue’s perspective, so Vue has no reason to reconcile its contents. jQuery has exclusive control over the inside.

---

### Real‑World Cases

- **Dashboard widgets**: A Vue dashboard embedding a legacy jQuery chart in an isolated container.
- **Map integrations**: A jQuery‑based map plugin rendered inside a Vue component.
- **Rich media**: A jQuery slideshow or video player wrapped in a boundary.

---

## Core Concept 5: jQuery Plugins in Vue

### Definitions

**Core Definition**
Integrating a jQuery plugin into Vue means creating a Vue wrapper component that initialises the plugin on a template‑ref DOM node, synchronises Vue props with the plugin’s API, bridges plugin events to Vue events, and destroys the plugin before unmount.

**Technical Definition**
jQuery plugins attach to DOM nodes and often add their own event listeners, timers, and DOM elements. The Vue wrapper pattern is: render an **empty container** with a ref, initialise the plugin in `mounted`/`onMounted`, destroy it in `beforeDestroy`/`onBeforeUnmount`, and use `watch` + `nextTick` to synchronise prop changes to the plugin. The wrapper should render an empty element so Vue does not attempt to update the plugin‑managed DOM. For plugins that expose a `destroy` method, that method must be called in cleanup; if the plugin does not provide one, the wrapper must manually remove event listeners to prevent memory leaks.

**Beginner‑Friendly Explanation**
A jQuery plugin is like a piece of furniture that assembles itself when you plug it in. It needs a real spot in the room (a DOM element) to stand on. Vue’s job is to build the room and the components. To bring the plugin in, you create a small empty space (a `<div>` with a ref), plug the plugin into that space, and let it do its thing. When the component is removed, you must tell the plugin to pack up and leave (call its `destroy` method). If you do not, the plugin stays in memory, still listening to events, and you get leaks.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them in Vue.
- **To synchronise plugin state with Vue props** so the plugin reflects the component’s data.
- **To bridge plugin events to Vue events** so the component can respond to user interaction.
- **To clean up plugin resources** on unmount to prevent leaks.
- **To isolate plugin DOM changes** inside a boundary that Vue does not touch.

---

### Syntax Rules and Structure

#### General Wrapper Pattern

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  props: { options: Object },
  setup(props) {
    const containerRef = ref(null);

    onMounted(() => {
      $(containerRef.value).pluginName(props.options);
    });

    onBeforeUnmount(() => {
      $(containerRef.value).pluginName("destroy");
    });

    return { containerRef };
  },
  template: `<div ref="containerRef"></div>`
};
```

#### Synchronising Props to the Plugin

```javascript
watch(() => props.options, async (newOpts) => {
  await nextTick();
  $(containerRef.value).pluginName("update", newOpts);
}, { deep: true });
```

---

### Constraints and Limitations

- **Plugins must provide a cleanup method**; otherwise, manual listener removal is required.
- **Plugin options must be mapped carefully** from Vue props.
- **Plugin events must be bridged** to Vue’s `emit`; they do not propagate automatically.
- **Vue 3’s `beforeUnmount`** replaces Vue 2’s `beforeDestroy`.

---

### Annotated Code Example (Date Picker Wrapper)

```javascript
import { ref, onMounted, onBeforeUnmount, watch, nextTick } from "vue";
import $ from "jquery";
import "jquery-datepicker";

export default {
  props: { modelValue: String },
  emits: ["update:modelValue"],
  setup(props, { emit }) {
    const dateRef = ref(null);

    onMounted(() => {
      const $input = $(dateRef.value);
      $input.datepicker({
        onSelect: function (date) {
          emit("update:modelValue", date);
        }
      });
    });

    onBeforeUnmount(() => {
      $(dateRef.value).datepicker("destroy");
    });

    watch(() => props.modelValue, async (val) => {
      await nextTick();
      $(dateRef.value).datepicker("setDate", val);
    });

    return { dateRef };
  },
  template: `<input ref="dateRef" type="text" />`
};
```

**Expected Output**
The `<input>` is enhanced by the jQuery date picker. Selecting a date emits `update:modelValue`. When the prop changes externally, the date picker is updated after Vue’s DOM update. On unmount, the date picker is destroyed.

**Why this output occurs**: `onMounted` initialises the plugin and binds its `onSelect` callback to Vue’s `emit`. `onBeforeUnmount` destroys the plugin. The watcher uses `nextTick` to synchronise external prop changes.

---

### Real‑World Cases

- **Admin dashboards**: Wrapping a jQuery DataTable, date picker, or autocomplete widget in a Vue component.
- **Legacy editors**: Embedding a jQuery‑based WYSIWYG editor in a Vue form.
- **Third‑party widgets**: Integrating a jQuery carousel, modal, or chart library that has no Vue counterpart.

---

## Core Concept 6: jQuery Events in Vue

### Definitions

**Core Definition**
jQuery events in Vue refers to the challenge of handling events triggered by jQuery code within Vue’s event system, and vice versa — the two systems are independent and do not automatically propagate to each other.

**Technical Definition**
Vue uses its own event system with `v-on` / `@event` bindings and component `emits`. jQuery uses its own event system with `.on()` and `.trigger()`. When jQuery triggers an event with `.trigger()`, it does not fire Vue’s `@event` handlers, because Vue listens for native events through its own delegation mechanism. Similarly, Vue’s event handlers do not automatically invoke jQuery handlers bound to the same element. The bridge pattern is to **call Vue’s `emit` directly from jQuery handlers**, or to use Vue’s `nextTick` to re‑initialise jQuery plugins after Vue’s DOM updates. The GitLab guide states: “It is acceptable for Vue to listen to existing jQuery events using jQuery event listeners. It is not recommended to add new jQuery events for Vue to interact with jQuery”.

**Beginner‑Friendly Explanation**
Vue and jQuery each have their own event system, and they do not talk to each other. If jQuery fires an event, Vue may not notice. If Vue handles an event, jQuery may not know. This is like two people in the same room using different languages — they can both hear sounds, but they do not understand each other. The bridge is to translate: when jQuery needs Vue to know something, call Vue’s `emit` directly. When Vue needs jQuery to know something, trigger the jQuery event with `.trigger()`.

---

### Purposes

- **To bridge plugin events to Vue events** so the component can respond to user interaction.
- **To trigger jQuery plugin updates** from Vue state changes.
- **To avoid relying on automatic event propagation** between the two systems.
- **To handle controlled inputs** that are manipulated by jQuery plugins.
- **To maintain a single source of truth** for state, even when jQuery is involved.

---

### Syntax Rules and Structure

#### jQuery → Vue (Direct Emit)

```javascript
$el.on("change.jqueryWrapper", (e) => {
  emit("update:modelValue", e.target.value);
});
```

#### Vue → jQuery (Trigger Plugin Update)

```javascript
await nextTick();
$(el.value).val(newValue).trigger("chosen:updated");
```

---

### Constraints and Limitations

- **jQuery `.trigger()` does not always produce a native event** that Vue’s delegation can see.
- **Vue’s `emit` is the recommended bridge** for component events.
- **`stopPropagation` in one system does not stop the other** unless the event is genuinely native and propagation is controlled at the native level.

---

### Annotated Code Example

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  emits: ["selected"],
  setup(props, { emit }) {
    const listRef = ref(null);

    onMounted(() => {
      const $list = $(listRef.value);
      $list.on("click.item", "li", function () {
        emit("selected", $(this).text());
      });
    });

    onBeforeUnmount(() => {
      $(listRef.value).off(".item");
    });

    return { listRef };
  },
  template: `<ul ref="listRef"><li>Item 1</li><li>Item 2</li></ul>`
};
```

**Expected Output**
Clicking an `<li>` emits the `selected` event with the item’s text. The handler is removed on unmount.

**Why this output occurs**: The jQuery delegated handler on `listRef` captures clicks on `<li>` elements and calls Vue’s `emit` directly, bridging the two event systems.

---

### Real‑World Cases

- **Form validation**: A jQuery validation plugin firing events that must update Vue form state.
- **Custom controls**: A jQuery slider or colour picker whose value must be reflected in Vue.
- **Legacy widgets**: A jQuery autocomplete that must notify Vue when a selection is made.

---

## Core Concept 7: jQuery AJAX in Vue

### Definitions

**Core Definition**
jQuery AJAX in Vue is the use of `$.ajax()`, `$.get()`, or `$.post()` inside a Vue component to fetch or send data, with the response stored in Vue’s reactive state and the request lifecycle managed through lifecycle hooks to prevent memory leaks and race conditions.

**Technical Definition**
Vue does not include built‑in AJAX capabilities and allows any HTTP library. However, jQuery AJAX does not integrate directly with Vue’s reactivity system; the response must be assigned to a `ref` or reactive property to trigger UI updates. In the Composition API, there is no `this` inside `setup`, so jQuery callbacks must reference the `ref` variable directly and assign to its `.value`. The critical practices are: (1) **do not make AJAX calls during setup/render**, (2) **cancel pending requests on unmount** to avoid updating a destroyed component, and (3) **handle loading and error states**. The Vue community consensus is that `fetch` or Axios are preferred over jQuery AJAX for new code.

**Beginner‑Friendly Explanation**
AJAX is how your Vue component talks to the server. jQuery’s `$.ajax()` works inside Vue, but you have to follow two rules. First, do not send the request while Vue is still setting up the component — wait until the component is on the page (`mounted` / `onMounted`). Second, if the component is removed before the request finishes, cancel the request so it does not try to update a component that no longer exists. The response must be stored in Vue’s reactive data so the page updates. Vue does not care which library you use — jQuery AJAX, Axios, or `fetch` are all fine — but `fetch` and Axios are simpler and integrate more naturally with modern Vue code.

---

### Purposes

- **To reuse existing jQuery AJAX code** during migration.
- **To fetch data after the component mounts** and store it in reactive state.
- **To cancel pending requests** on unmount to prevent state updates on destroyed components.
- **To handle loading and error states** in a Vue‑friendly way.
- **To bridge jQuery AJAX responses** into Vue’s reactive rendering.

---

### Syntax Rules and Structure

#### Options API

```javascript
export default {
  data() {
    return { items: [] };
  },
  mounted() {
    $.ajax({
      url: "/api/items",
      success: (data) => { this.items = data; }
    });
  }
};
```

#### Composition API

```javascript
import { ref, onMounted } from "vue";

export default {
  setup() {
    const items = ref([]);

    onMounted(() => {
      $.ajax({
        url: "/api/items",
        success: (data) => { items.value = data; }
      });
    });

    return { items };
  }
};
```

---

### Constraints and Limitations

- **Do not call AJAX during setup/render**; it causes repeated requests and unexpected behaviour.
- **Cancel pending requests** on unmount to avoid memory leaks.
- **jQuery AJAX returns a jqXHR object** that can be aborted with `.abort()`.
- **Race conditions** are possible if multiple requests are in flight; use a cancellation flag or abort the previous request.

---

### Annotated Code Example

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  setup() {
    const profile = ref(null);
    const loading = ref(true);
    const error = ref(null);
    let jqXHR = null;

    onMounted(() => {
      jqXHR = $.ajax({
        url: "/api/profile",
        dataType: "json",
        success: (data) => {
          profile.value = data;
          loading.value = false;
        },
        error: () => {
          error.value = "Failed to load profile";
          loading.value = false;
        }
      });
    });

    onBeforeUnmount(() => {
      if (jqXHR) jqXHR.abort();          // cancel pending request
    });

    return { profile, loading, error };
  }
};
```

**Expected Output**
The component displays “Loading…” while the request is in flight. On success, it renders the profile. On failure, it displays an error message. If the component unmounts before the request completes, the request is aborted.

**Why this output occurs**: `onMounted` fires the AJAX request after the component is on the page. The response is assigned to the `profile` ref, triggering a reactive update. `onBeforeUnmount` aborts the request to prevent state updates on a destroyed component.

---

### Real‑World Cases

- **Data‑driven pages**: Fetching user profiles, product lists, or dashboards.
- **Form submission**: Sending form data via `$.post()` and updating state on success.
- **Polling**: Periodically fetching updates with jQuery AJAX and updating reactive state.

---

## Core Concept 8: Integration Challenges

### Definitions

**Core Definition**
Integration challenges are the specific technical and architectural problems that arise when jQuery and Vue share the same runtime — including conflicting DOM ownership, separate event systems, reactivity bypass, lifecycle mismatches, and AJAX race conditions.

**Technical Definition**
The core challenge is **DOM ownership conflict**: Vue patches the DOM through its virtual DOM and reactivity system, while jQuery mutates the DOM directly. When jQuery modifies a node that Vue also manages, Vue’s virtual DOM becomes inconsistent with the actual DOM, leading to lost updates, stale data, or patch failures. Secondary challenges include: **event system separation** (jQuery events do not trigger Vue handlers, and vice versa); **reactivity bypass** (jQuery plugin state does not automatically synchronise with Vue reactive data; `nextTick` is required to update plugins after Vue’s DOM updates); **lifecycle mismatches** (jQuery plugins initialised in `mounted` must be destroyed in `beforeUnmount` to prevent leaks in SPAs); and **AJAX race conditions** (a pending request may return after the component has been destroyed, attempting to update non‑existent DOM).

**Beginner‑Friendly Explanation**
The biggest challenge is that both libraries want to be the boss of the page. Vue says “I control this tree”; jQuery says “I can change anything I want.” When they disagree, bugs happen. There are smaller challenges too: Vue and jQuery have separate event systems, so an event in one does not automatically reach the other. jQuery plugins often have their own internal state, which can drift out of sync with Vue state. And AJAX requests may return after the component is gone, trying to update something that no longer exists. Knowing these challenges is the first step to avoiding them — which is why the wrapper and boundary patterns exist.

---

### Purposes

- **To understand why jQuery and Vue conflict** at a technical level.
- **To anticipate the failure modes** of mixing the two libraries.
- **To design integration patterns** that avoid the conflicts.
- **To debug issues** that arise from DOM ownership, event separation, or reactivity bypass.
- **To make informed decisions** about migration strategy.

---

### Syntax Rules and Structure

| Challenge | Manifestation | Mitigation |
|---|---|---|
| **DOM ownership** | Vue drops updates, stale data, or patch failures | Empty container boundary; Vue does not update jQuery subtree |
| **Event separation** | jQuery events do not trigger Vue handlers | Call Vue `emit` directly from jQuery handlers |
| **Reactivity bypass** | Plugin state and Vue state drift apart | Sync props to plugin via `watch` + `nextTick` |
| **Lifecycle mismatch** | Plugin leaks on unmount; AJAX updates destroyed component | Destroy in `beforeUnmount`; abort AJAX on unmount |
| **Global dependency** | jQuery loaded globally increases bundle size | Load jQuery only in components that need it |

---

### Constraints and Limitations

- **The boundary pattern is not free**; it adds wrapper elements and requires explicit communication.
- **Event bridging adds boilerplate**; every plugin event must be wired to Vue’s `emit`.
- **`nextTick` is asynchronous**; plugin updates after Vue’s DOM updates must wait for it.

---

### Annotated Code Example (AJAX Race Condition Guard)

```javascript
import { ref, onMounted, onBeforeUnmount } from "vue";
import $ from "jquery";

export default {
  setup() {
    const data = ref(null);
    let active = true;

    onMounted(() => {
      $.ajax({
        url: "/api/data",
        success: (result) => {
          if (active) data.value = result;  // guard against destroyed component
        }
      });
    });

    onBeforeUnmount(() => {
      active = false;                       // prevent state updates after unmount
    });

    return { data };
  }
};
```

**Expected Output**
The data is stored only if the component is still alive when the request completes.

**Why this output occurs**: The `active` flag is set to `false` on unmount, so the success callback does not attempt to update a destroyed component.

---

### Real‑World Cases

- **Incremental migration**: A legacy jQuery page gaining a Vue component without breaking the jQuery widgets.
- **Third‑party plugin integration**: A jQuery plugin that maintains its own state while the rest of the form is Vue‑controlled.
- **SPA route changes**: jQuery handlers and AJAX requests must be cleaned up when the route changes.

---

## Core Concept 9: Integration Best Practices

### Definitions

**Core Definition**
jQuery‑in‑Vue best practices are the consolidated conventions — isolate jQuery in ref‑attached empty containers, initialise in `mounted`, destroy in `beforeUnmount`, bridge events explicitly, synchronise state with `nextTick`, cancel AJAX on unmount, and prefer Vue alternatives — that make the integration safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual patterns: (1) **render an empty container** with a `ref` and give jQuery exclusive control of its contents; (2) **initialise jQuery plugins in `mounted`/`onMounted`** after the DOM is committed; (3) **destroy plugins in `beforeDestroy`/`onBeforeUnmount`** to remove listeners and timers; (4) **bridge jQuery events to Vue `emit`** explicitly, rather than relying on propagation; (5) **synchronise Vue props to the plugin** via `watch` + `nextTick`; (6) **cancel pending AJAX requests** on unmount to prevent state updates on destroyed components; and (7) **prefer Vue components or modern libraries** when no legacy plugin requires jQuery.

**Beginner‑Friendly Explanation**
Best practices are the rules that keep the integration from exploding. Give jQuery its own box. Start it after Vue has built the page. Clean it up when the component is removed. Wire its events to Vue deliberately. Keep Vue state and plugin state in sync on purpose, not by accident. Cancel AJAX when the component goes away. And always ask: do I actually need jQuery here, or can Vue do this? If Vue can do it, use Vue.

---

### Purposes

- **To provide a single actionable checklist** for jQuery‑in‑Vue integration.
- **To prevent the most common integration failures** (DOM conflicts, leaks, event mismatches).
- **To balance legacy compatibility with Vue’s reactive model**.
- **To establish a baseline for code reviews and migration planning**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Boundary** | Empty `<div ref="…"></div>` | Vue children inside the jQuery container |
| **Initialisation** | `mounted` / `onMounted` | jQuery before mount |
| **Cleanup** | `beforeDestroy` / `onBeforeUnmount` | Leaving listeners and timers |
| **Events** | Vue `emit` + jQuery `.on()` bridge | Relying on automatic propagation |
| **State sync** | `watch` + `nextTick` | Letting state drift |
| **AJAX** | `mounted` + cancellation flag / abort | AJAX during setup/render |
| **Alternatives** | Vue components when possible | jQuery for new UI logic |

---

### Constraints and Limitations

- **Best practices are migration tools**, not a permanent architecture for new development.
- **`nextTick` adds asynchrony**; plugin updates after Vue’s DOM changes must wait for it.
- **Event bridging requires discipline**; every plugin event must be accounted for.

---

### Annotated Code Example (Consolidated)

```javascript
import { ref, onMounted, onBeforeUnmount, watch, nextTick } from "vue";
import $ from "jquery";

export default {
  props: { value: String },
  emits: ["update:value"],
  setup(props, { emit }) {
    const containerRef = ref(null);
    let jqXHR = null;

    // ✅ Initialise once in mounted
    onMounted(() => {
      const $el = $(containerRef.value);
      $el.somePlugin({ value: props.value });
      $el.on("change.plugin", (e) => emit("update:value", e.target.value));
    });

    // ✅ Sync prop changes to plugin
    watch(() => props.value, async (val) => {
      await nextTick();
      $(containerRef.value).somePlugin("setValue", val);
    });

    // ✅ Cleanup on unmount
    onBeforeUnmount(() => {
      if (jqXHR) jqXHR.abort();
      const $el = $(containerRef.value);
      $el.off(".plugin");
      $el.somePlugin("destroy");
    });

    // ✅ Vue renders an empty boundary
    return { containerRef };
  },
  template: `<div ref="containerRef"></div>`
};
```

**Expected Output**
The plugin is initialised once. Plugin events emit Vue events. Prop changes are synchronised to the plugin. On unmount, the plugin is destroyed and AJAX is cancelled.

**Why this output occurs**: Each best practice is applied: the boundary is empty, initialisation is in `onMounted`, cleanup is in `onBeforeUnmount`, events are bridged to `emit`, state is synchronised with `nextTick`, and AJAX is cancellable.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new jQuery integrations follow safe patterns.
- **Migration planning**: Each wrapped plugin is a candidate for eventual replacement with a Vue component.
- **SPA debugging**: `nextTick` and AJAX cancellation prevent common race conditions and leaks.

---

## Core Concept 10: When to Avoid jQuery in Vue

### Definitions

**Core Definition**
Avoiding jQuery in Vue means recognising the situations in which using jQuery is unnecessary, counterproductive, or actively harmful, and choosing Vue’s native APIs or modern alternatives instead.

**Technical Definition**
The Vue and GitLab guidance is explicit: “Mixing Vue and jQuery is not recommended”. The primary reason is the **virtual DOM and reactivity conflict**: jQuery manipulates the real DOM directly, while Vue manages the DOM through its virtual representation and reactive state; mixing the two can break Vue’s reconciliation and produce stale data. Specific cases to avoid include: **AJAX** — use `fetch` or Axios instead of `$.ajax()`; **DOM manipulation** — use Vue directives (`v-if`, `v-show`, `v-for`) and reactive state instead of `$(".selector").hide()`; **event handling** — use Vue’s `@event` bindings instead of `$(element).on()`; **animation** — use CSS transitions or Vue’s `<Transition>` component instead of `$.animate()`; and **utility functions** — use native `Array.prototype` methods instead of `$.each()` or `$.map()`. jQuery should be reserved for **legacy plugins that have no Vue equivalent** and **incremental migration scenarios** where rewriting is not feasible.

**Beginner‑Friendly Explanation**
Most of what people use jQuery for in 2026 is already built into Vue or the browser. Vue handles DOM updates, events, and state. `fetch` handles AJAX. CSS handles animations. Native JavaScript handles array iteration. jQuery adds a second, conflicting system for things Vue already does well. The only good reason to use jQuery in a Vue app is when you have a legacy plugin that you cannot easily replace. For everything else, avoid it. As the GitLab guide puts it: “Mixing Vue and jQuery is not recommended”.

---

### Purposes

- **To prevent unnecessary virtual DOM conflicts**.
- **To keep the application maintainable** by using one reactive model.
- **To reduce bundle size** by avoiding jQuery when it is not needed.
- **To use modern, purpose‑built APIs** (fetch, CSS transitions, Vue directives).
- **To reserve jQuery for genuine legacy compatibility** rather than habit.

---

### Syntax Rules and Structure

| jQuery Use Case | Vue Alternative |
|---|---|
| `$.ajax()`, `$.get()` | `fetch()` or Axios |
| `$(".el").hide()` | `v-if` / `v-show` + reactive state |
| `$(el).on("click", fn)` | `@click="fn"` |
| `$.animate()` | CSS transitions / `<Transition>` |
| `$.each()` / `$.map()` | `Array.prototype.forEach` / `.map()` |
| `$.extend()` | Spread operator / `Object.assign()` |

---

### Constraints and Limitations

- **Legacy plugins may have no Vue equivalent**; in that case, wrapping is justified.
- **Incremental migration may require temporary coexistence**; avoid jQuery for new code within the migration.
- **Some jQuery utilities have no direct native equivalent**; evaluate case by case.

---

### Annotated Code Example

```javascript
// ❌ AVOID: jQuery for AJAX in Vue
$.ajax({ url: "/api/data", success: (data) => { this.items = data; } });

// ✅ USE: fetch in onMounted
import { ref, onMounted } from "vue";
const items = ref([]);
onMounted(async () => {
  const res = await fetch("/api/data");
  items.value = await res.json();
});

// ❌ AVOID: jQuery for DOM manipulation
$("#myDiv").hide();

// ✅ USE: Vue conditional rendering
// <div v-if="isVisible">…</div>

// ❌ AVOID: jQuery for events
$(element).on("click", handler);

// ✅ USE: Vue event binding
// <button @click="handler">Click</button>
```

**Expected Output**
The `fetch` version loads data into Vue’s reactive state without jQuery. Vue directives handle visibility and events.

**Why this output occurs**: Each alternative uses Vue’s native mechanism, avoiding the virtual DOM conflict and keeping the application consistent.

---

### Real‑World Cases

- **New Vue development**: No jQuery at all; use Vue and modern APIs.
- **Migration projects**: New features are built in Vue; jQuery is confined to legacy wrappers.
- **Code reviews**: Flags jQuery usage for AJAX, events, or DOM manipulation as a candidate for replacement.

---

## References

- Mixing Vue and jQuery | GitLab Frontend Guide — https://git.ustc.edu.cn/help/development/fe_guide/vue.md#mixing-vue-and-jquery
- Vue Wrapper Component Example (select2) | Vue.js v2 — https://v2.vuejs.org/v2/examples/select2
- vue 如何结合jq 组件 | Worktile — https://worktile.com/kb/p/3618448
- vue如何封装jquery插件 | Worktile — https://worktile.com/kb/p/3631299
- 如何在Vue组件中使用JQuery？ | Tencent Cloud — https://cloud.tencent.cn/developer/information/如何在Vue组件中使用JQuery？-ask
- Vue3 fetch json with JQuery | Stack Overflow — https://stackoverflow.com/questions/79334646/vue3-fetch-json-with-jquery
- 分享｜单页应用中 jQuery 资源回收清单 | LeetCode — https://leetcode.cn/discuss/post/3838559/dan-ye-ying-yong-zhong-jquery-zi-yuan-hu-z96d/
- 如何解决第三方库（如 jQuery/D3）直接修改 DOM 导致的 Patch 失败？ | PHP.cn — https://www.php.cn
- Integrating third-party libraries with VNodes | Borstch — https://borstch-blog.vercel.app
- Vue.nextTick for jQuery plugins | Stack Overflow — https://stackoverflow.com
- 在vue方法中使用jquery函数 | Tencent Cloud — https://cloud.tencent.cn/developer/information/在vue方法中使用jquery函数
- vue使用jquery的ajax | Tencent Cloud — https://cloud.tencent.cn/developer/information/vue使用jquery的ajax
- Ways of Using Vue | Vue.js Official Documentation — https://vuejs.org/guide/extras/ways-of-using-vue.html
- Vue refs and $refs | Laracasts — https://laracasts.com
- vue 中的 ref 和 $refs | Alibaba Cloud — https://developer.aliyun.com