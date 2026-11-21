# jQuery in React — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Integrating jQuery into a React application is the practice of allowing legacy jQuery code — selectors, DOM manipulation, plugins, event handlers, and AJAX calls — to coexist with React’s declarative, component‑based rendering model, typically during an incremental migration from a jQuery‑based codebase to React.

**Technical Definition**
React and jQuery have fundamentally incompatible DOM ownership models. jQuery assumes it can freely read and mutate the DOM at any time. React assumes it owns a subtree of the DOM exclusively and reconciles that subtree against a virtual representation on every render. When both assumptions hold in different regions of the same document, the application works. When they collide — typically because jQuery reaches into a DOM node that React also manages — the result ranges from a silently dropped event handler to a thrown exception during reconciliation. The integration therefore requires explicit **boundaries**: React owns the component tree, jQuery owns isolated DOM islands created via `ref`, and communication occurs through props, callbacks, and lifecycle‑managed initialisation/cleanup. React’s official guidance states: “React is unaware of changes made to the DOM outside of React. It determines updates based on its own internal representation, and if the same DOM nodes are manipulated by another library, React gets confused and has no way to recover”. The recommended pattern is to prevent React from updating the subtree that jQuery manages — typically by rendering an empty `<div />` and attaching a `ref` to it.

**Beginner‑Friendly Explanation**
React and jQuery both want to control the page, but they do it in completely different ways. jQuery says: “I will find the element and change it directly.” React says: “I will describe what the page should look like, and I will handle all the changes myself.” If you let both change the same part of the page, they fight, and things break. The solution is to give each one its own territory. React owns the overall page and the components. Inside a component, you create a small empty container and tell jQuery: “This box is yours — do whatever you want inside it.” React promises not to touch that box. That way, jQuery plugins and legacy code can keep working while the rest of the application moves to React.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Conflicting DOM ownership** | jQuery mutates the DOM directly; React reconciles a virtual DOM. Concurrent control of the same nodes causes corruption or crashes. |
| **Isolation is mandatory** | jQuery must be confined to a `ref`‑attached container that React does not update. |
| **Lifecycle‑managed initialisation** | jQuery plugins must be initialised after the DOM is committed (`useEffect` / `componentDidMount`) and destroyed on cleanup. |
| **Event systems are separate** | React uses synthetic events delegated at the root; jQuery uses its own event system. Events triggered by jQuery do not automatically trigger React handlers. |
| **AJAX is framework‑agnostic** | jQuery AJAX can be used in React, but the response must be stored in React state, and cleanup must cancel pending requests. |
| **Migration tool, not a destination** | jQuery in React is appropriate for incremental migration and legacy plugin wrappers, not for new development. |

---

### Prerequisites

- Working knowledge of React function components, hooks (`useRef`, `useEffect`), and the component lifecycle.
- Working knowledge of jQuery selection, DOM manipulation, events, and plugins.
- Understanding of the virtual DOM and React’s reconciliation process.
- Awareness that React’s synthetic event system differs from native DOM events.

---

### Related Programming Areas

- **DOM performance** — jQuery and React both manipulate the DOM; batching and isolation affect performance.
- **Event performance** — React’s synthetic events vs. jQuery’s event system.
- **Memory performance** — plugins must be destroyed to prevent leaks.
- **Security** — jQuery sinks (`.html()`) inside React components reintroduce XSS risk.
- **Dependency security** — legacy jQuery versions carry known CVEs.

---

### Core Concepts / Features (Index)

1. jQuery in React (General)
2. React Refs (`useRef`)
3. `useEffect` for jQuery Lifecycle
4. DOM Manipulation Boundaries
5. jQuery Plugins in React
6. jQuery Events in React
7. jQuery AJAX in React
8. Integration Challenges
9. Integration Best Practices
10. When to Avoid jQuery in React


## Core Concept 1: jQuery in React (General)

### Definitions

**Core Definition**
jQuery in React is the coexistence of the jQuery library with React components, where jQuery is used for specific isolated tasks — typically legacy plugins or DOM operations that React does not cover — while React remains responsible for the component tree and state management.

**Technical Definition**
jQuery operates on the real DOM through its own selector engine, event system, and plugin architecture. React maintains a virtual DOM and reconciles the real DOM to match it on every render. When jQuery modifies a node that React also manages, React’s internal representation becomes inconsistent with the actual DOM, leading to lost updates, duplicated nodes, or exceptions. The official React integration guide describes the solution: render an empty `<div />` with a `ref`, pass that ref’s DOM node to jQuery, and prevent React from updating the subtree. The same guide notes: “Just because it’s possible, doesn’t mean that it’s the best approach for React apps. We encourage you to use React components when you can”.

**Beginner‑Friendly Explanation**
jQuery in React is a compromise. You are saying: “Most of this page is React, but this one part is still jQuery.” That is fine as long as the two parts stay in their own lanes. The moment jQuery reaches into a part of the page that React is also changing, you get bugs that are hard to diagnose. The general rule is: use jQuery only inside a dedicated container that React does not touch, and only for things that React cannot easily do — usually wrapping an existing jQuery plugin.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them from scratch.
- **To migrate incrementally** from a jQuery codebase to React, one component at a time.
- **To isolate legacy DOM manipulation** inside a boundary that React does not manage.
- **To bridge event and state changes** between React props and jQuery plugin APIs.
- **To avoid the cost of a full rewrite** while still moving toward React.

---

### Syntax Rules and Structure

#### General Isolation Pattern (Function Component)

```javascript
import React, { useRef, useEffect } from "react";
import $ from "jquery";

function JQueryContainer(props) {
  const containerRef = useRef(null);

  useEffect(() => {
    const $el = $(containerRef.current);
    $el.somePlugin();                     // initialise

    return () => {
      $el.somePlugin("destroy");          // cleanup
    };
  }, []);                                 // run once on mount

  return <div ref={containerRef} />;      // React renders an empty box
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `useRef(null)` | Creates a stable reference to the DOM node. |
| `useEffect(…, [])` | Runs after the DOM is committed; empty deps = mount/unmount only. |
| `$(containerRef.current)` | Wraps the real DOM node in jQuery. |
| `return <div ref={containerRef} />` | React renders an empty container that it will not update. |

---

### Constraints and Limitations

- **React must not update the jQuery‑managed subtree**; render an empty container.
- **jQuery plugins must be destroyed** to prevent memory leaks.
- **jQuery and React event systems are separate**; events do not automatically propagate between them.
- **jQuery version must be compatible**; legacy jQuery may carry known CVEs.

---

### Annotated Code Example

```javascript
import React, { useRef, useEffect } from "react";
import $ from "jquery";

function LegacyTooltip({ text }) {
  const containerRef = useRef(null);

  useEffect(() => {
    // Step 1: Initialise the jQuery plugin on the real DOM node
    const $el = $(containerRef.current);
    $el.tooltip({ title: text });

    // Step 2: Cleanup when the component unmounts
    return () => {
      $el.tooltip("destroy");
    };
  }, []); // mount once

  // Step 3: React renders only an empty container
  return <div ref={containerRef} />;
}

// Usage
// <LegacyTooltip text="Hello from jQuery" />
```

**Expected Output**
A tooltip is initialised on the container. When the component unmounts, the tooltip is destroyed and its event listeners are removed.

**Why this output occurs**: `useEffect` runs after React commits the empty `<div>` to the DOM. jQuery initialises the plugin on that node. The cleanup function runs on unmount, calling the plugin’s `destroy` method to release resources.

---

### Real‑World Cases

- **Legacy admin panels**: Wrapping a jQuery data table or date picker inside a React route during incremental migration.
- **Third‑party widgets**: Embedding a jQuery‑based chart or map library that has no React equivalent.
- **Gradual migration**: Moving one page at a time from a jQuery‑rendered application to React.

---

## Core Concept 2: React Refs (`useRef`)

### Definitions

**Core Definition**
A React ref (`useRef`) is a mutable object whose `.current` property holds a reference to a DOM node (or any value) that persists across renders without causing re‑renders when changed. It is the bridge between React’s component tree and jQuery’s direct DOM access.

**Technical Definition**
`useRef(initialValue)` returns a ref object with a `.current` property. When attached to a DOM element via the `ref` attribute, React sets `.current` to the DOM node after the commit phase. The ref persists for the lifetime of the component and does not trigger re‑renders when mutated. For jQuery integration, the ref provides the **real DOM node** that jQuery needs, while React retains ownership of the component and its rendering logic. The official React integration guide uses a ref to obtain the underlying DOM element and pass it to the jQuery plugin.

**Beginner‑Friendly Explanation**
A ref is a way to get your hands on the actual HTML element that React created. Normally, React hides the real DOM from you and asks you to work through state and props. But jQuery needs the real element — it cannot work with React’s virtual description. A ref gives you that real element. You attach the ref to a `<div>`, and after React renders it, `ref.current` points to the actual `<div>` in the page. You can then give that element to jQuery. The ref itself does not cause re‑renders when it changes, which is exactly what you want for a stable connection to the DOM.

---

### Purposes

- **To obtain the real DOM node** that jQuery needs for selection and manipulation.
- **To maintain a stable reference** to the container across re‑renders.
- **To avoid querying the DOM** with global selectors inside React components.
- **To pass the DOM node to a jQuery plugin** during initialisation.
- **To store jQuery objects or plugin instances** without triggering re‑renders.

---

### Syntax Rules and Structure

```javascript
const containerRef = useRef(null);
// …
return <div ref={containerRef} />;
```

**Component Breakdown**

| Component | Description |
|---|---|
| `useRef(null)` | Creates a ref with an initial `current` value of `null`. |
| `ref={containerRef}` | Attaches the ref to the `<div>`; React sets `.current` to the DOM node. |
| `containerRef.current` | The real DOM element, available after mount. |

---

### Constraints and Limitations

- **`ref.current` is `null` before mount**; access it only inside `useEffect` or event handlers.
- **Mutating `ref.current` does not trigger re‑renders**; use state for values that affect rendering.
- **Refs should not be overused**; prefer state and props for data flow, and refs only for DOM access.

---

### Annotated Code Example

```javascript
import React, { useRef, useEffect } from "react";
import $ from "jquery";

function JQueryButton({ label, onClick }) {
  const buttonRef = useRef(null);

  useEffect(() => {
    // Step 1: Wrap the real DOM node
    const $btn = $(buttonRef.current);

    // Step 2: Bind a jQuery event handler
    $btn.on("click.jqueryWrapper", function () {
      onClick();                         // call React callback
    });

    // Step 3: Cleanup
    return () => {
      $btn.off(".jqueryWrapper");
    };
  }, [onClick]);

  return <button ref={buttonRef}>{label}</button>;
}
```

**Expected Output**
Clicking the button invokes the React `onClick` callback. The jQuery handler is removed on unmount or when `onClick` changes.

**Why this output occurs**: `buttonRef.current` provides the real `<button>` DOM node. jQuery binds a namespaced click handler that calls the React callback. The cleanup removes only the namespaced handler.

---

### Real‑World Cases

- **Wrapping a jQuery plugin** that requires a DOM element to initialise.
- **Reading layout measurements** (e.g., `offsetHeight`) from a React‑rendered element.
- **Integrating with a non‑React library** that needs a DOM node.

---

## Core Concept 3: `useEffect` for jQuery Lifecycle

### Definitions

**Core Definition**
`useEffect` is a React hook that runs side effects after the component has been rendered and committed to the DOM. It is the correct place to initialise jQuery plugins, bind jQuery events, and perform cleanup when the component unmounts or dependencies change.

**Technical Definition**
`useEffect(callback, dependencies)` schedules the callback to run after the browser has painted the committed DOM. An **empty dependency array** (`[]`) runs the effect once after mount and runs its cleanup once before unmount, analogous to `componentDidMount` and `componentWillUnmount`. A **dependency array with values** re‑runs the effect when those values change, with cleanup running before each re‑run. For jQuery integration, the effect should: (1) obtain the DOM node from the ref, (2) initialise the plugin, (3) return a cleanup function that destroys the plugin and removes handlers. If the plugin does not provide a cleanup method, the developer must remove event listeners manually to prevent memory leaks.

**Beginner‑Friendly Explanation**
`useEffect` is React’s way of saying: “After you have put this component on the page, run this code.” That is exactly when jQuery needs to run — it cannot initialise a plugin on an element that does not exist yet. The empty dependency array (`[]`) means “run this once when the component appears, and run the cleanup when it disappears.” The cleanup function is where you tell jQuery to clean up after itself: destroy the plugin, remove event handlers, and cancel timers. If you forget the cleanup, the plugin keeps running in the background even after the component is gone, which wastes memory and can cause bugs.

---

### Purposes

- **To initialise jQuery plugins after the DOM is committed**.
- **To synchronise plugin state with React props** when dependencies change.
- **To clean up plugins and event handlers** on unmount to prevent leaks.
- **To avoid accessing `ref.current` before it is available**.
- **To manage the lifecycle of jQuery objects** inside a React component.

---

### Syntax Rules and Structure

#### Mount‑Only Pattern (Empty Dependencies)

```javascript
useEffect(() => {
  const $el = $(ref.current);
  $el.somePlugin();

  return () => {
    $el.somePlugin("destroy");
  };
}, []);                                 // mount once, cleanup on unmount
```

#### Dependency‑Driven Pattern

```javascript
useEffect(() => {
  const $el = $(ref.current);
  $el.somePlugin({ option: propValue });

  return () => {
    $el.somePlugin("destroy");
  };
}, [propValue]);                         // re-initialise when propValue changes
```

---

### Constraints and Limitations

- **`useEffect` runs after paint**; layout measurements may cause a flash if the plugin changes the DOM.
- **Missing cleanup causes leaks**; every `useEffect` that initialises a plugin should return a cleanup function.
- **Dependency arrays must be complete**; omitting dependencies can cause stale closures or missed updates.

---

### Annotated Code Example

```javascript
import React, { useRef, useEffect } from "react";
import $ from "jquery";

function ChosenWrapper({ options, value, onChange }) {
  const selectRef = useRef(null);

  useEffect(() => {
    // Step 1: Get the real <select> node
    const $select = $(selectRef.current);

    // Step 2: Initialise the jQuery Chosen plugin
    $select.chosen();

    // Step 3: Bridge Chosen's change event to React
    $select.on("change.chosenWrapper", function () {
      onChange(this.value);
    });

    // Step 4: Cleanup
    return () => {
      $select.off(".chosenWrapper");
      $select.chosen("destroy");
    };
  }, [onChange]); // re-run if onChange changes

  return (
    <select ref={selectRef} defaultValue={value}>
      {options.map(opt => (
        <option key={opt} value={opt}>{opt}</option>
      ))}
    </select>
  );
}
```

**Expected Output**
The `<select>` is enhanced by Chosen. Selecting an option triggers the React `onChange` callback. When the component unmounts, Chosen is destroyed and the handler is removed.

**Why this output occurs**: `useEffect` runs after React renders the `<select>`. jQuery initialises Chosen and binds a namespaced change handler. The cleanup removes the handler and destroys the plugin.

---

### Real‑World Cases

- **Date pickers**: Initialising a jQuery date picker on a React‑rendered input.
- **Rich text editors**: Wrapping a legacy jQuery‑based WYSIWYG editor.
- **Data tables**: Initialising a jQuery DataTable on a React‑rendered table.

---

## Core Concept 4: DOM Manipulation Boundaries

### Definitions

**Core Definition**
DOM manipulation boundaries are explicit regions of the page where either React or jQuery has exclusive control, preventing the two libraries from mutating the same nodes and corrupting each other’s internal state.

**Technical Definition**
React assumes exclusive ownership of the DOM subtree it renders and reconciles that subtree against its virtual representation. jQuery assumes it can read and mutate any node at any time. When both operate on the same node, React’s virtual DOM becomes inconsistent with the real DOM, and React cannot recover. The boundary pattern renders an **empty container** (`<div ref={ref} />`) that React has no reason to update, and jQuery manages everything inside that container. The boundary must be respected in both directions: React must not render children into the jQuery container, and jQuery must not reach outside it.

**Beginner‑Friendly Explanation**
Imagine two chefs in one kitchen. One chef (React) insists on controlling every pot and pan. The other chef (jQuery) just grabs whatever is nearby and starts cooking. If they both grab the same pan, disaster. The solution is to give each chef their own counter. React gets the main kitchen; jQuery gets a small separate counter (the empty container). React promises never to touch jQuery’s counter, and jQuery promises to stay on its own counter. That is a DOM manipulation boundary.

---

### Purposes

- **To prevent React and jQuery from mutating the same nodes**.
- **To ensure React’s virtual DOM remains consistent** with the real DOM.
- **To isolate jQuery plugins** inside a container that React does not update.
- **To allow legacy code to run safely** alongside React components.
- **To make the integration predictable** and easier to debug.

---

### Syntax Rules and Structure

```javascript
// ✅ React renders an empty container; jQuery owns its contents
return <div ref={containerRef} />;

// ❌ React renders children into the jQuery container
return <div ref={containerRef}>{reactChildren}</div>;
```

**Rule**: The container that jQuery manages must be **empty from React’s perspective**. React must not render children into it, and must not update its attributes or content.

---

### Constraints and Limitations

- **The boundary must be absolute**; even a single React update to the jQuery container breaks the guarantee.
- **Communication must go through props and callbacks**, not through shared DOM nodes.
- **The boundary adds a wrapper element** to the DOM, which may affect layout or styling.

---

### Annotated Code Example

```javascript
import React, { useRef, useEffect } from "react";
import $ from "jquery";

function JQueryChart({ data }) {
  const chartRef = useRef(null);

  useEffect(() => {
    const $el = $(chartRef.current);
    $el.someChart({ data });

    return () => {
      $el.someChart("destroy");
    };
  }, [data]);

  // ✅ React renders an empty container — jQuery owns the inside
  return <div ref={chartRef} className="chart-boundary" />;
}
```

**Expected Output**
React renders an empty `<div>`. jQuery draws the chart inside it. When `data` changes, React re‑runs the effect, jQuery destroys and recreates the chart. React never touches the chart’s internal DOM.

**Why this output occurs**: The container is empty from React’s perspective, so React has no reason to reconcile its contents. jQuery has exclusive control over the inside.

---

### Real‑World Cases

- **Dashboard widgets**: A React dashboard embedding a legacy jQuery chart in an isolated container.
- **Map integrations**: A jQuery‑based map plugin rendered inside a React component.
- **Rich media**: A jQuery slideshow or video player wrapped in a boundary.

---

## Core Concept 5: jQuery Plugins in React

### Definitions

**Core Definition**
Integrating a jQuery plugin into React means creating a React wrapper component that initialises the plugin on a `ref`‑attached DOM node, synchronises React props with the plugin’s API, and destroys the plugin on unmount.

**Technical Definition**
jQuery plugins attach to DOM nodes and often add their own event listeners, timers, and DOM elements. React’s official integration guide describes the wrapper pattern: attach a `ref` to the root DOM element, initialise the plugin inside `componentDidMount` (or `useEffect`), and destroy it inside `componentWillUnmount` (or the `useEffect` cleanup). The wrapper should render an **empty element** so React does not attempt to update the plugin‑managed DOM. For plugins that expose a `destroy` or `remove` method, that method must be called in cleanup; if the plugin does not provide one, the wrapper must manually remove event listeners to prevent memory leaks. The **Chosen** plugin is the canonical example: it hides the original `<select>`, creates its own visual control, and communicates changes via jQuery events.

**Beginner‑Friendly Explanation**
A jQuery plugin is like a piece of furniture that assembles itself when you plug it in. It needs a real spot in the room (a DOM element) to stand on. React’s job is to build the room and the components. To bring the plugin in, you create a small empty space (a `<div>` with a ref), plug the plugin into that space, and let it do its thing. When the component is removed, you must tell the plugin to pack up and leave (call its `destroy` method). If you do not, the plugin stays in memory, still listening to events, and you get leaks.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them in React.
- **To synchronise plugin state with React props** so the plugin reflects the component’s data.
- **To bridge plugin events to React callbacks** so the component can respond to user interaction.
- **To clean up plugin resources** on unmount to prevent leaks.
- **To isolate plugin DOM changes** inside a boundary that React does not touch.

---

### Syntax Rules and Structure

#### General Wrapper Pattern

```javascript
function PluginWrapper(props) {
  const containerRef = useRef(null);

  useEffect(() => {
    const $el = $(containerRef.current);
    $el.pluginName(/* options from props */);

    return () => {
      $el.pluginName("destroy");
    };
  }, [/* dependencies from props */]);

  return <div ref={containerRef} />;
}
```

#### Synchronising Props to Plugin API

```javascript
useEffect(() => {
  const $el = $(containerRef.current);
  $el.pluginName("update", { option: props.value });
}, [props.value]);
```

---

### Constraints and Limitations

- **Plugins must provide a cleanup method**; otherwise, manual listener removal is required.
- **Plugin options must be mapped carefully** from React props.
- **Plugin events must be bridged** to React callbacks; they do not propagate automatically.
- **React 19 and Strict Mode** may double‑invoke effects in development; ensure the plugin can handle re‑initialisation or guard with a ref.

---

### Annotated Code Example (Chosen Plugin Wrapper)

```javascript
import React, { useRef, useEffect } from "react";
import $ from "jquery";
import "chosen-js/chosen.css";
import "chosen-js/chosen.jquery.js";

function Chosen({ options, value, onChange }) {
  const selectRef = useRef(null);

  // Step 1: Initialise the plugin on mount
  useEffect(() => {
    const $select = $(selectRef.current);
    $select.chosen();

    $select.on("change.chosenWrapper", function () {
      onChange(this.value);
    });

    // Step 2: Destroy on unmount
    return () => {
      $select.off(".chosenWrapper");
      $select.chosen("destroy");
    };
  }, [onChange]);

  // Step 3: Update the plugin when the value changes
  useEffect(() => {
    const $select = $(selectRef.current);
    $select.val(value).trigger("chosen:updated");
  }, [value]);

  return (
    <select ref={selectRef} defaultValue={value}>
      {options.map(opt => (
        <option key={opt} value={opt}>{opt}</option>
      ))}
    </select>
  );
}
```

**Expected Output**
The `<select>` is enhanced by Chosen. Changing the value via React props updates the Chosen control. Selecting an option in Chosen triggers the React `onChange` callback. On unmount, Chosen is destroyed and the handler is removed.

**Why this output occurs**: The first effect initialises Chosen and binds a namespaced change handler. The second effect synchronises the `value` prop with the plugin via `trigger("chosen:updated")`. The cleanup removes the handler and calls `destroy`.

---

### Real‑World Cases

- **Admin dashboards**: Wrapping a jQuery DataTable, date picker, or autocomplete widget in a React component.
- **Legacy editors**: Embedding a jQuery‑based WYSIWYG editor in a React form.
- **Third‑party widgets**: Integrating a jQuery carousel, modal, or chart library that has no React counterpart.

---

## Core Concept 6: jQuery Events in React

### Definitions

**Core Definition**
jQuery events in React refers to the challenge of handling events triggered by jQuery code within React’s synthetic event system, and vice versa — the two systems are independent and do not automatically propagate to each other.

**Technical Definition**
React implements a **synthetic event system** that delegates event handling to the document root and normalises behaviour across browsers. jQuery uses its own event system, which attaches handlers directly to elements and manages its own propagation. When jQuery triggers an event with `.trigger()`, it does not fire React’s synthetic `onChange` or `onClick` handlers, because React’s delegation listens for native events at the root and jQuery’s synthetic trigger does not always produce a native event. Similarly, React’s synthetic events do not automatically invoke jQuery handlers bound to the same element. The bridge pattern is to use jQuery’s `.trigger()` to fire a native event that React can intercept, or to call React callbacks directly from jQuery handlers. React’s `onChange` is particularly problematic: jQuery’s `.trigger("change")` does not update React’s controlled input value tracker, so React may ignore the event unless the native value setter is invoked.

**Beginner‑Friendly Explanation**
React and jQuery each have their own event system, and they do not talk to each other. If jQuery fires a “change” event, React may not notice. If React handles a click, jQuery may not know. This is like two people in the same room using different languages — they can both hear sounds, but they do not understand each other. The bridge is to translate: when jQuery needs React to know something, call the React callback directly. When React needs jQuery to know something, trigger the jQuery event with `.trigger()`.

---

### Purposes

- **To bridge plugin events to React callbacks** so the component can respond to user interaction.
- **To trigger jQuery plugin updates** from React state changes.
- **To avoid relying on automatic event propagation** between the two systems.
- **To handle controlled inputs** that are manipulated by jQuery plugins.
- **To maintain a single source of truth** for state, even when jQuery is involved.

---

### Syntax Rules and Structure

#### jQuery → React (Direct Callback)

```javascript
$el.on("change.jqueryWrapper", function () {
  onChange(this.value);                  // call React callback directly
});
```

#### React → jQuery (Trigger Plugin Update)

```javascript
$el.val(newValue).trigger("chosen:updated");
```

#### Bridging Native Events to React

```javascript
// Use native event dispatch if React's synthetic system must see it
const nativeInputValueSetter = Object.getOwnPropertyDescriptor(
  window.HTMLInputElement.prototype, "value"
).set;
nativeInputValueSetter.call(input, newValue);
input.dispatchEvent(new Event("input", { bubbles: true }));
```

---

### Constraints and Limitations

- **jQuery `.trigger()` does not always produce a native event** that React’s delegation can see.
- **React’s `onChange` requires the native value setter** for programmatic updates to be recognised.
- **`stopPropagation` in one system does not stop the other** unless the event is genuinely native and propagation is controlled at the native level.

---

### Annotated Code Example

```javascript
import React, { useRef, useEffect, useState } from "react";
import $ from "jquery";

function JQuerySlider({ value, onChange }) {
  const sliderRef = useRef(null);
  const [internalValue, setInternalValue] = useState(value);

  useEffect(() => {
    const $slider = $(sliderRef.current);
    $slider.slider({
      value: internalValue,
      slide: function (event, ui) {
        setInternalValue(ui.value);     // update React state
        onChange(ui.value);              // call React callback
      }
    });

    return () => {
      $slider.slider("destroy");
    };
  }, [onChange]);

  // Sync React state to jQuery widget
  useEffect(() => {
    const $slider = $(sliderRef.current);
    if ($slider.slider("value") !== internalValue) {
      $slider.slider("value", internalValue);
    }
  }, [internalValue]);

  return <div ref={sliderRef} />;
}
```

**Expected Output**
Dragging the slider updates React state and calls the `onChange` callback. When the `value` prop changes externally, the slider’s position updates.

**Why this output occurs**: The first effect initialises the jQuery UI slider and binds its `slide` event to React state and the callback. The second effect synchronises React state back to the slider when it changes externally.

---

### Real‑World Cases

- **Form validation**: A jQuery validation plugin firing events that must update React form state.
- **Custom controls**: A jQuery slider or colour picker whose value must be reflected in React.
- **Legacy widgets**: A jQuery autocomplete that must notify React when a selection is made.

---

## Core Concept 7: jQuery AJAX in React

### Definitions

**Core Definition**
jQuery AJAX in React is the use of `$.ajax()`, `$.get()`, or `$.post()` inside a React component to fetch or send data, with the response stored in React state and the request lifecycle managed through `useEffect` to prevent memory leaks and race conditions.

**Technical Definition**
React does not include built‑in AJAX capabilities and allows any HTTP library, including jQuery AJAX, Axios, or the native `fetch` API. In class components, AJAX calls are made in `componentDidMount` so that the component is mounted and state updates trigger a re‑render. In function components, the equivalent is `useEffect` with an empty dependency array. The critical practices are: (1) **do not make AJAX calls during render**, (2) **cancel pending requests on unmount** to avoid setting state on an unmounted component, and (3) **handle loading and error states**. React’s official guidance states: “You can use any AJAX library you like with React. Some popular ones are Axios, jQuery AJAX, and the browser built‑in `window.fetch`”.

**Beginner‑Friendly Explanation**
AJAX is how your React component talks to the server. jQuery’s `$.ajax()` works fine inside React, but you have to follow two rules. First, do not send the request while React is still rendering the component — wait until the component is on the page (`useEffect`). Second, if the component is removed before the request finishes, cancel the request so it does not try to update a component that no longer exists. React does not care which library you use — jQuery AJAX, Axios, or `fetch` are all fine. The important thing is to put the response into React state so the component re‑renders with the new data.

---

### Purposes

- **To reuse existing jQuery AJAX code** during migration.
- **To fetch data after the component mounts** and store it in state.
- **To cancel pending requests** on unmount to prevent state updates on unmounted components.
- **To handle loading and error states** in a React‑friendly way.
- **To bridge jQuery AJAX responses** into React’s state‑driven rendering.

---

### Syntax Rules and Structure

```javascript
useEffect(() => {
  let cancelled = false;

  $.ajax({
    url: "/api/data",
    type: "GET",
    dataType: "json",
    success: function (data) {
      if (!cancelled) setData(data);
    },
    error: function (xhr) {
      if (!cancelled) setError(xhr.status);
    }
  });

  return () => {
    cancelled = true;                    // prevent state updates after unmount
  };
}, []);
```

---

### Constraints and Limitations

- **Do not call AJAX during render**; it causes repeated requests and unexpected behaviour.
- **Cancel pending requests** on unmount to avoid memory leaks.
- **jQuery AJAX returns a jqXHR object** that can be aborted with `.abort()`.
- **Race conditions** are possible if multiple requests are in flight; use a cancellation flag or abort the previous request.

---

### Annotated Code Example

```javascript
import React, { useEffect, useState } from "react";
import $ from "jquery";

function UserProfile({ userId }) {
  const [profile, setProfile] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    setError(null);

    // Step 1: Fire the AJAX request after mount
    const jqXHR = $.ajax({
      url: `/api/users/${userId}`,
      type: "GET",
      dataType: "json",
      success: function (data) {
        if (!cancelled) {
          setProfile(data);
          setLoading(false);
        }
      },
      error: function (xhr) {
        if (!cancelled) {
          setError("Failed to load profile");
          setLoading(false);
        }
      }
    });

    // Step 2: Cancel on unmount or userId change
    return () => {
      cancelled = true;
      jqXHR.abort();
    };
  }, [userId]);

  if (loading) return <p>Loading…</p>;
  if (error) return <p>{error}</p>;
  return <div>{profile.name}</div>;
}
```

**Expected Output**
The component displays “Loading…” while the request is in flight. On success, it renders the user’s name. On failure, it displays an error message. If the component unmounts or `userId` changes, the request is aborted and state is not updated.

**Why this output occurs**: `useEffect` runs after mount and fires the AJAX request. The `cancelled` flag prevents state updates after unmount. The cleanup aborts the jqXHR, cancelling the request.

---

### Real‑World Cases

- **Data‑driven pages**: Fetching user profiles, product lists, or dashboards.
- **Form submission**: Sending form data via `$.post()` and updating state on success.
- **Polling**: Periodically fetching updates with jQuery AJAX and updating state.

---

## Core Concept 8: Integration Challenges

### Definitions

**Core Definition**
Integration challenges are the specific technical and architectural problems that arise when jQuery and React share the same runtime — including conflicting DOM ownership, separate event systems, state synchronisation difficulties, and lifecycle mismatches.

**Technical Definition**
The core challenge is **DOM ownership conflict**: React assumes exclusive control of its subtree and reconciles it against a virtual representation, while jQuery mutates the DOM directly. Secondary challenges include: **event system separation** (jQuery events do not trigger React synthetic events, and vice versa); **state synchronisation** (jQuery plugins often maintain their own internal state, which must be reconciled with React state); **lifecycle mismatches** (jQuery plugins are initialised after mount and must be destroyed on unmount, but React’s Strict Mode may double‑invoke effects in development); and **dependency management** (loading jQuery globally increases bundle size and may introduce version conflicts). The academic literature describes seven coexistence strategies, including mounting React inside jQuery‑managed containers, wrapping jQuery plugins as React components, custom event bridges, shared state layers, route‑based strangler‑fig migration, islands architecture, and build tooling considerations.

**Beginner‑Friendly Explanation**
The biggest challenge is that both libraries want to be the boss of the page. React says “I control this tree”; jQuery says “I can change anything I want.” When they disagree, bugs happen. There are smaller challenges too: React and jQuery have separate event systems, so an event in one does not automatically reach the other. jQuery plugins often have their own internal state, which can drift out of sync with React state. And React’s Strict Mode can initialise a plugin twice in development, causing duplicate widgets. Knowing these challenges is the first step to avoiding them — which is why the wrapper and boundary patterns exist.

---

### Purposes

- **To understand why jQuery and React conflict** at a technical level.
- **To anticipate the failure modes** of mixing the two libraries.
- **To design integration patterns** that avoid the conflicts.
- **To debug issues** that arise from DOM ownership, event separation, or state drift.
- **To make informed decisions** about migration strategy.

---

### Syntax Rules and Structure

| Challenge | Manifestation | Mitigation |
|---|---|---|
| **DOM ownership** | React drops updates, duplicates nodes, or throws during reconciliation | Empty container boundary; React does not update jQuery subtree |
| **Event separation** | jQuery `.trigger()` does not fire React `onChange` | Call React callbacks directly from jQuery handlers |
| **State synchronisation** | Plugin state and React state drift apart | Sync props to plugin via `useEffect`; bridge plugin events to callbacks |
| **Lifecycle mismatch** | Plugin initialised twice in Strict Mode; leaks on unmount | Guard with ref; always destroy in cleanup |
| **Global dependency** | jQuery loaded globally increases bundle size | Load jQuery only in components that need it |

---

### Constraints and Limitations

- **The boundary pattern is not free**; it adds wrapper elements and requires explicit communication.
- **Event bridging adds boilerplate**; every plugin event must be wired to a React callback.
- **Strict Mode double‑invocation** requires idempotent initialisation or guards.

---

### Annotated Code Example (Strict Mode Guard)

```javascript
function PluginWrapper() {
  const containerRef = useRef(null);
  const pluginInstanceRef = useRef(null);

  useEffect(() => {
    const $el = $(containerRef.current);

    // Guard against double initialisation in Strict Mode
    if (!pluginInstanceRef.current) {
      pluginInstanceRef.current = $el.somePlugin();
    }

    return () => {
      if (pluginInstanceRef.current) {
        pluginInstanceRef.current.destroy();
        pluginInstanceRef.current = null;
      }
    };
  }, []);

  return <div ref={containerRef} />;
}
```

**Expected Output**
The plugin is initialised once, even if the effect runs twice in development. The cleanup destroys it and clears the instance ref.

**Why this output occurs**: The `pluginInstanceRef` holds the plugin instance and prevents re‑initialisation. The cleanup resets it, allowing correct behaviour on subsequent mounts.

---

### Real‑World Cases

- **Incremental migration**: A legacy jQuery page gaining a React component without breaking the jQuery widgets.
- **Third‑party plugin integration**: A jQuery plugin that maintains its own state while the rest of the form is React‑controlled.
- **Strict Mode debugging**: Duplicate plugin instances in development caused by double‑invoked effects.

---

## Core Concept 9: Integration Best Practices

### Definitions

**Core Definition**
jQuery‑in‑React best practices are the consolidated conventions — isolate jQuery in ref‑attached empty containers, initialise in `useEffect`, destroy in cleanup, bridge events explicitly, synchronise state deliberately, and prefer React alternatives — that make the integration safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual patterns: (1) **render an empty container** with a `ref` and give jQuery exclusive control of its contents; (2) **initialise jQuery plugins in `useEffect`** after the DOM is committed; (3) **destroy plugins in the cleanup function** to remove listeners and timers; (4) **bridge jQuery events to React callbacks** explicitly, rather than relying on propagation; (5) **synchronise React props to the plugin** via a dedicated effect; (6) **guard against Strict Mode double‑invocation** with an instance ref; (7) **use jQuery AJAX only when necessary**, and always store the response in React state with cancellation on unmount; and (8) **prefer React components or modern libraries** when no legacy plugin requires jQuery.

**Beginner‑Friendly Explanation**
Best practices are the rules that keep the integration from exploding. Give jQuery its own box. Start it after React has built the page. Clean it up when the component is removed. Wire its events to React deliberately. Keep React state and plugin state in sync on purpose, not by accident. Handle Strict Mode. And always ask: do I actually need jQuery here, or can React do this? If React can do it, use React.

---

### Purposes

- **To provide a single actionable checklist** for jQuery‑in‑React integration.
- **To prevent the most common integration failures** (DOM conflicts, leaks, event mismatches).
- **To balance legacy compatibility with React’s declarative model**.
- **To establish a baseline for code reviews and migration planning**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Boundary** | Empty `<div ref={ref} />` | React children inside the jQuery container |
| **Initialisation** | `useEffect(…, [])` | jQuery during render or before mount |
| **Cleanup** | Destroy plugin, remove handlers | Leaving listeners and timers |
| **Events** | Explicit callbacks and `.trigger()` | Relying on automatic propagation |
| **State sync** | Dedicated `useEffect` for props → plugin | Letting state drift |
| **Strict Mode** | Instance ref guard | Double initialisation |
| **AJAX** | `useEffect` + cancellation flag | AJAX during render |
| **Alternatives** | React components when possible | jQuery for new UI logic |

---

### Constraints and Limitations

- **Best practices are migration tools**, not a permanent architecture for new development.
- **Strict Mode guards add complexity**; they are necessary because React deliberately double‑invokes effects in development.
- **Event bridging requires discipline**; every plugin event must be accounted for.

---

### Annotated Code Example (Consolidated)

```javascript
import React, { useRef, useEffect, useState } from "react";
import $ from "jquery";

function LegacyWidget({ value, onChange }) {
  const containerRef = useRef(null);
  const instanceRef = useRef(null);
  const [pluginState, setPluginState] = useState(null);

  // ✅ Initialise once, guard Strict Mode, destroy on cleanup
  useEffect(() => {
    const $el = $(containerRef.current);

    if (!instanceRef.current) {
      instanceRef.current = $el.legacyWidget({
        value,
        onUpdate: function (newValue) {
          setPluginState(newValue);       // bridge plugin event to React
          onChange(newValue);              // call React callback
        }
      });
    }

    return () => {
      if (instanceRef.current) {
        instanceRef.current.destroy();
        instanceRef.current = null;
      }
    };
  }, []); // mount/unmount only

  // ✅ Sync React prop to plugin
  useEffect(() => {
    if (instanceRef.current) {
      instanceRef.current.setValue(value);
    }
  }, [value]);

  // ✅ React renders an empty boundary
  return <div ref={containerRef} />;
}
```

**Expected Output**
The widget is initialised once. Plugin events update React state and call the callback. Prop changes are synchronised to the plugin. On unmount, the plugin is destroyed.

**Why this output occurs**: Each best practice is applied: the boundary is empty, initialisation is in `useEffect`, Strict Mode is guarded, cleanup destroys the plugin, events are bridged to React, and state is synchronised.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new jQuery integrations follow safe patterns.
- **Migration planning**: Each wrapped plugin is a candidate for eventual replacement with a React component.
- **Strict Mode debugging**: Guard patterns prevent duplicate initialisation in development.

---

## Core Concept 10: When to Avoid jQuery in React

### Definitions

**Core Definition**
Avoiding jQuery in React means recognising the situations in which using jQuery is unnecessary, counterproductive, or actively harmful, and choosing React’s native APIs or modern alternatives instead.

**Technical Definition**
The React community consensus is that jQuery should not be used in React applications unless absolutely necessary. The primary reason is the **virtual DOM conflict**: jQuery manipulates the real DOM directly, while React manages the DOM through its virtual representation; mixing the two can break React’s reconciliation. Specific cases to avoid include: **AJAX** — use `fetch` or Axios instead of `$.ajax()`; **DOM manipulation** — use refs and React state instead of `$(".selector").hide()`; **event handling** — use React’s synthetic events instead of `$(element).on()`; **animation** — use CSS transitions or React animation libraries instead of `$.animate()`; and **utility functions** — use native `Array.prototype` methods instead of `$.each()` or `$.map()`. jQuery should be reserved for **legacy plugins that have no React equivalent** and **incremental migration scenarios** where rewriting is not feasible.

**Beginner‑Friendly Explanation**
Most of what people use jQuery for in 2026 is already built into React or the browser. React handles DOM updates, events, and state. `fetch` handles AJAX. CSS handles animations. Native JavaScript handles array iteration. jQuery adds a second, conflicting system for things React already does well. The only good reason to use jQuery in a React app is when you have a legacy plugin that you cannot easily replace. For everything else, avoid it. As one GitHub project puts it: “Be a smart developer and avoid using this hook. It’s not a good idea to combine jQuery with React. Since you’re already using React, choose one bad idea — you don’t need two”.

---

### Purposes

- **To prevent unnecessary virtual DOM conflicts**.
- **To keep the application maintainable** by using one rendering model.
- **To reduce bundle size** by avoiding jQuery when it is not needed.
- **To use modern, purpose‑built APIs** (fetch, CSS transitions, React state).
- **To reserve jQuery for genuine legacy compatibility** rather than habit.

---

### Syntax Rules and Structure

| jQuery Use Case | React Alternative |
|---|---|
| `$.ajax()`, `$.get()` | `fetch()` or Axios |
| `$(".el").hide()` | React conditional rendering / state |
| `$(el).on("click", fn)` | React `onClick` prop |
| `$.animate()` | CSS transitions / React animation libraries |
| `$.each()` / `$.map()` | `Array.prototype.forEach` / `.map()` |
| `$.extend()` | Spread operator / `Object.assign()` |

---

### Constraints and Limitations

- **Legacy plugins may have no React equivalent**; in that case, wrapping is justified.
- **Incremental migration may require temporary coexistence**; avoid jQuery for new code within the migration.
- **Some jQuery utilities have no direct native equivalent**; evaluate case by case.

---

### Annotated Code Example

```javascript
// ❌ AVOID: jQuery for AJAX in React
$.ajax({
  url: "/api/data",
  success: function (data) { setData(data); }
});

// ✅ USE: fetch (or Axios) in useEffect
useEffect(() => {
  let cancelled = false;
  fetch("/api/data")
    .then(res => res.json())
    .then(data => { if (!cancelled) setData(data); });
  return () => { cancelled = true; };
}, []);

// ❌ AVOID: jQuery for DOM manipulation
$("#myDiv").hide();

// ✅ USE: React conditional rendering
{isVisible && <div id="myDiv">…</div>}

// ❌ AVOID: jQuery for events
$(element).on("click", handler);

// ✅ USE: React event props
<button onClick={handler}>Click</button>
```

**Expected Output**
The `fetch` version loads data into React state without jQuery. The React conditional rendering hides the div through state. The `onClick` prop handles the click through React’s synthetic event system.

**Why this output occurs**: Each alternative uses the framework’s native mechanism, avoiding the virtual DOM conflict and keeping the application consistent.

---

### Real‑World Cases

- **New React development**: No jQuery at all; use React and modern APIs.
- **Migration projects**: New features are built in React; jQuery is confined to legacy wrappers.
- **Code reviews**: Flags jQuery usage for AJAX, events, or DOM manipulation as a candidate for replacement.

---

## References

- Integrating with Other Libraries | React (Legacy) — https://ru.react.js.org/docs/integrating-with-other-libraries.html
- Integrating with Other Libraries | React (Raw Docs) — https://raw.githubusercontent.com/wacii/react/d52245f2bd7f15686ccfd849d31f94efe7759c08/docs/docs/integrating-with-other-libraries.md
- Rendering a jQuery component with React: How to write the useEffects? | Stack Overflow — https://stackoverflow.com/feeds/question/78920430
- AJAX and APIs | React (Legacy) — https://vi.legacy.reactjs.org/docs/faq-ajax.html
- AJAX Requests | ruanyf/react-demos | DeepWiki — https://deepwiki.com/ruanyf/react-demos/4.1-ajax-requests
- Is it a bad practice to use jQuery in ReactJS? | Stack Overflow — https://stackoverflow.com/questions/53570925
- use-jquery-react | GitHub (MKAbuMattar) — https://github.com/MKAbuMattar/use-jquery-react
- Hybrid jQuery React Interoperability Strategies During Incremental Frontend Framework Migration | IJCTECE — https://ijctece.com/index.php/IJCTEC/article/download/716/651/1381
- Safely Wrapping Legacy jQuery Plugins in React 19: Managing Refs and Lifecycle Cleanup — https://coldfusion-example.blogspot.com/2026/01/safely-wrapping-legacy-jquery-plugins.html
- jQuery event handling vs React synthetic events | Stack Overflow — https://stackoverflow.com/questions/32497035
- React refs and componentDidMount for jQuery plugins | Stack Overflow — https://stackoverflow.com/questions/30259411