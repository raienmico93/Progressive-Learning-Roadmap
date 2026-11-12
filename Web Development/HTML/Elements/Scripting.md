# Scripting Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Scripting elements are the HTML elements that allow authors to embed or reference executable code, provide fallback content for environments without script support, define inert template markup for later instantiation, expose content insertion points within Web Components, and create scriptable bitmap rendering surfaces.

**Technical Definition**
In the WHATWG HTML Living Standard, “Scripting” is a dedicated chapter (4.12) covering `<script>`, `<noscript>`, `<template>`, and `<slot>`. The `<script>` element is a **script-supporting element** that allows authors to include dynamic script, instructions to the user agent, and data blocks; it belongs to the **metadata content**, **flow content**, **phrasing content**, and **script-supporting element** categories. `<noscript>` defines a section of HTML to be inserted when a script type is unsupported or scripting is disabled. `<template>` holds inert client-side content that is parsed but not rendered until instantiated by script, exposing a read-only `content` property containing a `DocumentFragment`. `<slot>` is a placeholder inside a Web Component’s shadow tree that can be filled with consumer-provided markup. `<canvas>` provides a scriptable bitmap surface whose drawing context is obtained via `getContext()`, typically `"2d"` for 2D rendering or `"webgl"` / `"webgl2"` for 3D rendering.

**Beginner-Friendly Explanation**
Scripting elements are the tools that make a web page come alive. `<script>` is how you add JavaScript. `<noscript>` is the “plan B” shown to users who have JavaScript turned off. `<template>` is a reusable blueprint for content that stays hidden until you need it. `<slot>` is a hole in a custom component that lets you plug in your own content. `<canvas>` is a blank drawing board that JavaScript can paint on.

---

### Key Characteristics

- **Script-supporting role**: `<script>` and `<template>` are script-supporting elements that do not render content for the user by default.
- **Execution control**: `<script>` supports `async`, `defer`, `type="module"`, and `nomodule` to control when and how code is fetched and executed.
- **Graceful degradation**: `<noscript>` provides fallback HTML when scripting is unavailable.
- **Inert content**: `<template>` content is parsed but inert — scripts do not run, images do not load, and nothing renders until the template is cloned into the document.
- **Web Components integration**: `<slot>` is the composition mechanism for Shadow DOM, enabling content projection.
- **Canvas rendering contexts**: `<canvas>` exposes multiple context types (`"2d"`, `"webgl"`, `"webgl2"`, `"webgpu"`, `"bitmaprenderer"`) for different rendering needs.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<html>`, `<head>`, `<body>`, and content categories.
2. **JavaScript fundamentals**: variables, functions, DOM manipulation, and event handling.
3. **An understanding of the DOM**: nodes, elements, and the document tree.
4. **Browser developer tools**: Console, Elements panel, and Network panel.
5. **For Web Components**: basic knowledge of custom elements and the Shadow DOM.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **JavaScript execution** | `<script>` is the primary mechanism for loading and running JavaScript. |
| **Progressive enhancement** | `<noscript>` and graceful degradation ensure functionality without scripts. |
| **Web Components** | `<template>` and `<slot>` are foundational to Shadow DOM composition. |
| **Graphics and visualisation** | `<canvas>` enables 2D drawing, 3D WebGL, and data visualisation. |
| **Performance** | `async`, `defer`, `type="module"`, and lazy template instantiation directly affect loading and rendering performance. |
| **Security** | `integrity`, `crossorigin`, and `nonce` control script trust and execution. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. `<script>` — embedded or referenced executable code
2. `<noscript>` — script-disabled fallback content
3. `<template>` — inert client-side template markup
4. `<slot>` — Web Component content insertion point
5. `<canvas>` — scriptable bitmap surface
6. **Scripting element best practices**

---

## 1. `<script>` — The Script Element

### Definitions

**Core Definition**
The `<script>` element is used to embed executable code or data, typically JavaScript, or to reference an external script file.

**Technical Definition**
`<script>` belongs to the **metadata content**, **flow content**, **phrasing content**, and **script-supporting element** categories. Its content model depends on the presence of `src`: without `src`, its content must match script content restrictions; with `src`, the element must be empty or contain only script documentation. Core attributes include `type`, `src`, `async`, `defer`, `nomodule`, `crossorigin`, `integrity`, `referrerpolicy`, `fetchpriority`, and `blocking`. The `type` attribute determines whether the script is a classic script (default), a module script (`type="module"`), or a data block.

**Beginner-Friendly Explanation**
The `<script>` element is how you put JavaScript on a web page. You can either write the code directly inside the tags (inline) or point to a `.js` file with `src`. Attributes like `defer` and `async` control when the code runs.

---

### Purposes

- **To embed inline JavaScript** directly in the document.
- **To reference external script files** via the `src` attribute.
- **To control execution timing** with `async` and `defer`.
- **To load ES modules** with `type="module"`.
- **To prevent execution in module-supporting browsers** with `nomodule`.
- **To verify resource integrity** with `integrity` (Subresource Integrity).
- **To embed data blocks** (e.g. JSON) that are not executed as scripts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- External classic script with defer -->
<script src="app.js" defer></script>

<!-- External module script -->
<script type="module" src="app.mjs"></script>

<!-- Inline classic script -->
<script>
  console.log("Hello from inline script");
</script>

<!-- Data block -->
<script type="application/json" id="data">
  { "key": "value" }
</script>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the external script. |
| `type` | Script type: omitted/JS MIME for classic, `module` for ES modules, or a data block MIME type. |
| `async` | Fetches in parallel and executes as soon as available; does not guarantee order. |
| `defer` | Fetches in parallel and executes after parsing, in document order. |
| `nomodule` | Prevents execution in browsers that support module scripts. |
| `crossorigin` | CORS settings for the fetch. |
| `integrity` | SRI hash for verifying the resource. |
| `referrerpolicy` | Referrer policy for the fetch. |
| `blocking` | Whether the element is potentially render-blocking. |

#### Syntax Rules

1. **Classic scripts block HTML parsing** unless `async` or `defer` is present.
2. **`defer` has no effect on inline scripts** (scripts without `src`) and is ignored.
3. **Module scripts defer by default**; `defer` is ignored on them.
4. **`async` on module scripts** fetches dependencies in parallel and evaluates as soon as possible.
5. **`nomodule` is a boolean attribute** that prevents the script from executing in module-supporting browsers.
6. **If both `async` and `defer` are present**, the element behaves as if only `async` were specified.

#### Constraints and Limitations

- Inline scripts must not contain the literal string `</script>` without escaping.
- `async` scripts execute out of order; do not use `async` for dependent scripts.
- `integrity` must not be specified without `src`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `defer` vs `async` Execution Order

**Setup guide**: Save as `script-defer-async.html` and observe the console.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Defer vs Async</title>

  <!-- Deferred scripts execute in order after parsing -->
  <script src="defer-1.js" defer></script>
  <script src="defer-2.js" defer></script>

  <!-- Async scripts execute as soon as available; order not guaranteed -->
  <script src="async-1.js" async></script>
  <script src="async-2.js" async></script>
</head>
<body>
  <p>Open the console to see execution order.</p>
</body>
</html>
```

**Expected output**: The `defer` scripts execute after HTML parsing, in the order `defer-1.js` then `defer-2.js`. The `async` scripts execute as soon as each finishes downloading, which may be in any order depending on network timing.

**Why this output occurs**: `defer` queues scripts for execution after parsing while preserving document order. `async` executes immediately upon download completion, without waiting for parsing or other scripts. This makes `defer` suitable for dependent scripts and `async` suitable for independent analytics or ads.

---

#### Example 2: Module Script with `nomodule` Fallback

**Setup guide**: Save as `script-module.html`. Modern browsers run the module; older browsers run the fallback.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Module and Nomodule</title>

  <!-- Modern browsers: load the module -->
  <script type="module" src="modern-app.mjs"></script>

  <!-- Older browsers that do not support modules: load the classic fallback -->
  <script nomodule src="legacy-app.js"></script>
</head>
<body>
  <p>Module-capable browsers run modern-app.mjs; older browsers run legacy-app.js.</p>
</body>
</html>
```

**Expected output**: In a modern browser, `modern-app.mjs` executes and `legacy-app.js` is ignored because `nomodule` prevents its execution. In an older browser, the module is ignored and the classic script runs.

**Why this output occurs**: `type="module"` marks the script as an ES module. `nomodule` is a boolean attribute that suppresses execution in browsers that support modules, providing a clean legacy fallback.

---

### Real-World Cases

1. **Analytics and tracking**: Loaded with `async` so they do not block page rendering.
2. **Application bundles**: Loaded with `defer` so they execute after the DOM is parsed.
3. **Modern JavaScript applications**: Use `type="module"` for native ES modules, with `nomodule` fallbacks for legacy browsers.
4. **Data embedding**: `<script type="application/json">` blocks are used to embed structured data for scripts to consume.

---

### References

- MDN Web Docs — `<script>`: The Script element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script
- WHATWG — The `script` element — https://html.spec.whatwg.org/dev/scripting.html#the-script-element
- W3C — HTML 5.2: The `script` element — https://www.w3.org/TR/html52/scripting-1.html#the-script-element

---

## 2. `<noscript>` — The Noscript Element

### Definitions

**Core Definition**
The `<noscript>` element defines a section of HTML to be inserted if a script type on the page is unsupported or if scripting is currently turned off in the browser.

**Technical Definition**
`<noscript>` belongs to the **metadata content**, **flow content**, and **phrasing content** categories. Its permitted content depends on context. When scripting is disabled and the element is a descendant of `<head>`, it may contain zero or more `<link>`, `<style>`, and `<meta>` elements, in any order. When scripting is disabled and the element is not a descendant of `<head>`, it may contain transparent content, but no descendant `<noscript>` elements. When scripting is enabled, its children are treated as text and are not rendered as markup.

**Beginner-Friendly Explanation**
The `<noscript>` element is a fallback for people who have JavaScript turned off. You put alternative content or alternative resources inside it, and the browser shows or applies them only when scripting is unavailable.

---

### Purposes

- **To provide alternative content when scripting is disabled**.
- **To load fallback styles or metadata in `<head>`**.
- **To support progressive enhancement** and graceful degradation.
- **To warn users about missing functionality**.

---

### Syntax Rules and Structure

#### Complete General Syntax (in `<head>`)

```html
<noscript>
  <link rel="stylesheet" href="no-script.css">
  <style>
    /* fallback styles */
  </style>
  <meta http-equiv="refresh" content="0; url=/no-script-fallback">
</noscript>
```

#### Complete General Syntax (in `<body>`)

```html
<noscript>
  <p>This feature requires JavaScript. Please enable JavaScript or use the <a href="/alternative">alternative version</a>.</p>
</noscript>
```

**Component breakdown**

| Context | Permitted content |
|---|---|
| In `<head>`, scripting disabled | `<link>`, `<style>`, `<meta>` only |
| Outside `<head>`, scripting disabled | Transparent content; no nested `<noscript>` |
| Scripting enabled | Children treated as text; markup not parsed or rendered |

#### Syntax Rules

1. **In `<head>`, only `<link>`, `<style>`, and `<meta>` are allowed** when scripting is disabled.
2. **Outside `<head>`, the content model is transparent**, matching the parent’s content model.
3. **Both start and end tags are mandatory**; there is no tag omission.
4. **When scripting is enabled, the content is inert text** and is not rendered as HTML.

#### Constraints and Limitations

- Nested `<noscript>` elements are invalid.
- `<noscript>` in `<head>` cannot contain arbitrary flow content.
- Critical content should not be placed exclusively inside `<noscript>`, because it is invisible to users with JavaScript enabled.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Body Fallback Message

**Setup guide**: Save as `noscript-body.html`. Disable JavaScript in the browser and reload to see the fallback.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Noscript Body Fallback</title>
</head>
<body>
  <div id="app">
    <p>Loading interactive content…</p>
  </div>

  <noscript>
    <p><strong>JavaScript is disabled.</strong> The interactive features of this page require JavaScript. Please enable it or use the <a href="/basic">basic version</a>.</p>
  </noscript>

  <script>
    document.getElementById('app').innerHTML = '<p>Interactive content loaded.</p>';
  </script>
</body>
</html>
```

**Expected output**: With JavaScript enabled, the page shows “Interactive content loaded.” and the `<noscript>` message is not displayed. With JavaScript disabled, the page shows “Loading interactive content…” followed by the fallback warning and link.

**Why this output occurs**: When scripting is enabled, the `<noscript>` children are treated as text and are not rendered as HTML. When scripting is disabled, the `<noscript>` content is parsed as HTML and displayed.

---

#### Example 2: Head Fallback Stylesheet

**Setup guide**: Save as `noscript-head-style.html`. Disable JavaScript and reload.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Noscript Head Styles</title>

  <!-- Normal styles -->
  <style>
    body { background: #ffffff; color: #000000; }
  </style>

  <!-- Fallback styles applied only when scripting is disabled -->
  <noscript>
    <style>
      body { background: #fff3cd; color: #856404; }
    </style>
    <link rel="stylesheet" href="no-script-extra.css">
  </noscript>
</head>
<body>
  <p>If JavaScript is disabled, a warning-coloured theme is applied.</p>
</body>
</html>
```

**Expected output**: With JavaScript enabled, the page has a white background and black text. With JavaScript disabled, the page has a warm yellow background, brown text, and additional styles from `no-script-extra.css` are applied.

**Why this output occurs**: In `<head>`, when scripting is disabled, `<noscript>` may contain `<link>`, `<style>`, and `<meta>` elements, and those elements take effect. When scripting is enabled, the content is treated as text and has no effect.

---

### Real-World Cases

1. **Progressive enhancement**: Warning users that client-side validation will not run when JavaScript is disabled.
2. **Analytics fallback**: A `<noscript>` image pixel can provide basic analytics when JavaScript-based tracking is unavailable.
3. **Critical CSS fallback**: Loading a full stylesheet when JavaScript is disabled while JavaScript-enabled browsers load critical CSS asynchronously.

---

### References

- MDN Web Docs — `<noscript>`: The Noscript element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/noscript
- WHATWG — The `noscript` element — https://html.spec.whatwg.org/dev/scripting.html#the-noscript-element
- W3C — HTML 5.2: The `noscript` element — https://www.w3.org/TR/html52/scripting-1.html#the-noscript-element

---

## 3. `<template>` — The Content Template Element

### Definitions

**Core Definition**
The `<template>` element is a mechanism for holding client-side HTML content that is not rendered when the page loads but can be instantiated later at runtime using JavaScript.

**Technical Definition**
`<template>` belongs to the **metadata content**, **flow content**, **phrasing content**, and **script-supporting element** categories. Its content is parsed as HTML but is inert: scripts do not execute, images do not load, and nothing renders until the template is activated. The `HTMLTemplateElement` interface exposes a read-only `content` property whose value is a `DocumentFragment` containing the template’s DOM subtree. The `content` property is a separate document fragment whose `ownerDocument` is distinct from the document containing the `<template>` itself.

**Beginner-Friendly Explanation**
A `<template>` is a reusable blueprint for HTML. The content inside it is hidden and inactive until you use JavaScript to clone it and insert it into the page.

---

### Purposes

- **To declare reusable HTML fragments** that are parsed but not rendered at load time.
- **To support client-side templating** in a standards-based, DOM-oriented way.
- **To define Shadow DOM markup** for Web Components.
- **To avoid side effects** from inactive content (no script execution, no image loading).
- **To provide a feature-detectable mechanism** via the `content` property.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<template id="my-template">
  <div class="card">
    <h2 class="card-title"></h2>
    <p class="card-body"></p>
  </div>
</template>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<template>` | The template container. |
| `id` | Used to select the template from JavaScript. |
| Content | Any HTML content; parsed but inert until instantiated. |
| `.content` | Read-only `DocumentFragment` containing the template’s DOM subtree. |

#### Syntax Rules

1. **`<template>` has no restrictions on its content**; it can contain any HTML.
2. **Both start and end tags are mandatory**.
3. **The `content` property returns a `DocumentFragment`** that is separate from the main document.
4. **Content is inert until cloned and inserted**; scripts do not run and resources do not load.

#### Constraints and Limitations

- Template content is not rendered and is not part of the main document tree until instantiated.
- The template’s `content` fragment must be cloned (e.g. via `cloneNode(true)`) before insertion; inserting the fragment directly moves its children.
- Browser support is broad, but older environments may require a feature detection check (`'content' in document.createElement('template')`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Instantiating a Template with Data

**Setup guide**: Save as `template-card.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Template Card</title>
</head>
<body>
  <div id="card-container"></div>

  <!-- Inert template -->
  <template id="card-template">
    <div class="card">
      <h2 class="card-title"></h2>
      <p class="card-body"></p>
    </div>
  </template>

  <script>
    const template = document.getElementById('card-template');
    const container = document.getElementById('card-container');

    // Clone the template content
    const clone = template.content.cloneNode(true);

    // Populate the clone
    clone.querySelector('.card-title').textContent = 'Dynamic Card';
    clone.querySelector('.card-body').textContent = 'This card was created from a template.';

    // Insert the populated clone into the document
    container.appendChild(clone);
  </script>
</body>
</html>
```

**Expected output**: A card with the heading “Dynamic Card” and the body text “This card was created from a template.” appears inside the container after the page loads.

**Why this output occurs**: The `<template>` content is parsed but inert. The script clones the `content` fragment, populates it, and appends it to the visible document. The template itself remains hidden.

---

#### Example 2: Template for Shadow DOM in a Web Component

**Setup Guide**: Save as `template-shadow.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Template in Shadow DOM</title>
</head>
<body>
  <my-paragraph></my-paragraph>

  <template id="my-paragraph-template">
    <style>
      p { color: navy; font-weight: bold; }
    </style>
    <p>My default text</p>
  </template>

  <script>
    customElements.define('my-paragraph',
      class extends HTMLElement {
        constructor() {
          super();
          const template = document.getElementById('my-paragraph-template');
          const shadowRoot = this.attachShadow({ mode: 'open' });
          // Append a clone of the template content to the shadow root
          shadowRoot.appendChild(template.content.cloneNode(true));
        }
      }
    );
  </script>
</body>
</html>
```

**Expected output**: The `<my-paragraph>` custom element renders a navy bold paragraph with the text “My default text”, styled by the `<style>` inside the template and encapsulated in the shadow DOM.

**Why this output occurs**: The template provides the shadow DOM markup. The custom element clones the template’s `content` into its shadow root, where the `<style>` applies only within the shadow tree.

---

### Real-World Cases

1. **Web Components**: Defining the shadow DOM markup for custom elements.
2. **Client-side rendering**: Frameworks and vanilla JavaScript use `<template>` to define reusable UI fragments.
3. **Deferred content**: Holding large markup blocks that should not render or load resources until needed.

---

### References

- MDN Web Docs — `<template>`: The Content Template element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template
- WHATWG — The `template` element — https://html.spec.whatwg.org/dev/scripting.html#the-template-element
- MDN Web Docs — `HTMLTemplateElement.content` — https://developer.mozilla.org/en-US/docs/Web/API/HTMLTemplateElement/content

---

## 4. `<slot>` — The Web Component Slot Element

### Definitions

**Core Definition**
The `<slot>` element is a placeholder inside a Web Component that you can fill with your own markup when the component is used.

**Technical Definition**
`<slot>` is part of the Web Components technology suite. It lives inside a shadow tree and provides an insertion point for light DOM content. Slots are identified by their `name` attribute; content from the host element is assigned to slots whose `name` matches the element’s `slot` attribute. If no content is assigned to a slot, its fallback content (any children of the `<slot>` element) is rendered.

**Beginner-Friendly Explanation**
A `<slot>` is a hole in a custom component where you can plug in your own content. When you use the component, you put elements inside it with a `slot` attribute, and those elements appear in the matching slot inside the component.

---

### Purposes

- **To enable content projection** in Web Components.
- **To allow consumers to inject custom markup** into a component’s shadow DOM.
- **To provide fallback content** when no content is assigned to a slot.
- **To support named and default slot assignment** for flexible component composition.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- Inside the shadow DOM template -->
<slot name="title">Default title</slot>
<slot>Default content</slot>
```

```html
<!-- Usage in light DOM -->
<my-component>
  <h2 slot="title">Custom Title</h2>
  <p>Custom body content</p>
</my-component>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<slot>` | The insertion point inside the shadow DOM. |
| `name` | Identifies the slot for named assignment. |
| Fallback content | Children of `<slot>`, rendered when no assigned nodes exist. |
| `slot` attribute | On light-DOM elements, assigns the element to a named slot. |

#### Syntax Rules

1. **Slot names should be unique per shadow root**; if two slots share a name, content is assigned to the first.
2. **The default slot** (unnamed) receives content without a matching `slot` attribute.
3. **Fallback content** is displayed when no nodes are assigned to the slot.

#### Constraints and Limitations

- Slots only function within a shadow tree; they are not general-purpose HTML elements.
- The `slot` attribute on light-DOM elements is a global attribute that assigns the element to a named slot.
- Accessibility behaviour at the shadow DOM boundary can be complex; testing is required.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Named and Default Slots

**Setup Guide**: Save as `slot-example.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Slot Example</title>
</head>
<body>
  <my-card>
    <h2 slot="title">Custom Card Title</h2>
    <p>This is the default slot content.</p>
  </my-card>

  <template id="card-template">
    <style>
      .card { border: 2px solid #333; padding: 16px; margin: 8px; }
    </style>
    <div class="card">
      <slot name="title">Default Title</slot>
      <slot>Default body content</slot>
    </div>
  </template>

  <script>
    customElements.define('my-card',
      class extends HTMLElement {
        constructor() {
          super();
          const template = document.getElementById('card-template');
          this.attachShadow({ mode: 'open' })
              .appendChild(template.content.cloneNode(true));
        }
      }
    );
  </script>
</body>
</html>
```

**Expected output**: A bordered card displaying “Custom Card Title” in the title slot and “This is the default slot content.” in the default slot. If the consumer provided no content, the fallback texts “Default Title” and “Default body content” would appear instead.

**Why this output occurs**: The `<slot name="title">` receives the `<h2 slot="title">` element, and the unnamed `<slot>` receives the `<p>`. The fallback content is overridden because assigned nodes exist.

---

### Real-World Cases

1. **Design system components**: Cards, dialogs, and layout components use slots to accept arbitrary consumer content.
2. **Web Components libraries**: Lit, FAST, and other libraries rely on `<slot>` for composition.
3. **Custom elements**: Any custom element that needs to wrap or arrange consumer markup uses slots.

---

### References

- MDN Web Docs — `<slot>`: The Web Component Slot element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/slot
- MDN Web Docs — Using templates and slots — https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_templates_and_slots
- WHATWG — The `slot` element — https://html.spec.whatwg.org/dev/scripting.html#the-slot-element

---

## 5. `<canvas>` — The Scriptable Bitmap Surface

### Definitions

**Core Definition**
The `<canvas>` element provides a scriptable bitmap surface on which graphics can be drawn using the Canvas API or WebGL API.

**Technical Definition**
`<canvas>` belongs to the **flow content**, **phrasing content**, **embedded content**, and **palpable content** categories. Its content model is **transparent fallback content** displayed when the browser does not support canvas. The element has `width` and `height` attributes (defaulting to 300 × 150 CSS pixels). All drawing is performed via JavaScript using `getContext()`, which returns a rendering context: `"2d"` for `CanvasRenderingContext2D`, `"webgl"` or `"webgl2"` for 3D rendering, `"webgpu"` for WebGPU, or `"bitmaprenderer"` for bitmap rendering. Subsequent calls with the same `contextType` return the same context instance.

**Beginner-Friendly Explanation**
The `<canvas>` element is a blank drawing board. You use JavaScript to draw shapes, text, images, and animations on it. You can draw in 2D or 3D depending on which context you request.

---

### Purposes

- **To draw 2D graphics** with the Canvas 2D API.
- **To render 3D scenes** with WebGL or WebGPU.
- **To create animations and interactive visualisations**.
- **To generate dynamic images** that cannot be expressed as static files.
- **To manipulate pixel data** for image processing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<canvas id="myCanvas" width="400" height="300">
  Fallback content for browsers without canvas support.
</canvas>

<script>
  const canvas = document.getElementById('myCanvas');
  const ctx = canvas.getContext('2d');
  // drawing commands…
</script>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<canvas>` | The drawing surface. |
| `width` / `height` | Dimensions in CSS pixels; default 300 × 150. |
| Fallback content | Displayed if the browser does not support canvas. |
| `getContext(contextType)` | Returns a rendering context; `"2d"` creates a `CanvasRenderingContext2D`. |

#### Syntax Rules

1. **Set `width` and `height` attributes** to define the drawing surface size.
2. **Use `getContext('2d')`** for 2D drawing.
3. **Use `getContext('webgl')` or `getContext('webgl2')`** for 3D rendering.
4. **Fallback content is permitted** and should be meaningful.
5. **The same context type returns the same instance** on repeated calls; you cannot obtain different context objects on the same canvas.

#### Constraints and Limitations

- Canvas content is not accessible to screen readers unless alternative text or ARIA is provided.
- Canvas graphics are raster-based and do not scale as cleanly as SVG.
- `getContext()` returns `null` if the context type is unsupported.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Drawing a Rectangle with the 2D Context

**Setup guide**: Save as `canvas-rect.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Canvas Rectangle</title>
</head>
<body>
  <canvas id="myCanvas" width="300" height="200">
    Your browser does not support the canvas element.
  </canvas>

  <script>
    const canvas = document.getElementById('myCanvas');
    // Obtain the 2D rendering context
    const ctx = canvas.getContext('2d');

    // Set fill colour
    ctx.fillStyle = '#0077cc';
    // Draw a filled rectangle at (50, 50) with width 200 and height 100
    ctx.fillRect(50, 50, 200, 100);
  </script>
</body>
</html>
```

**Expected output**: A blue rectangle drawn on the canvas.

**Why this output occurs**: `getContext('2d')` returns a `CanvasRenderingContext2D` object. The `fillStyle` sets the colour, and `fillRect()` draws a filled rectangle at the specified coordinates.

---

#### Example 2: Drawing Text

**Setup guide**: Save as `canvas-text.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Canvas Text</title>
</head>
<body>
  <canvas id="textCanvas" width="400" height="100">
    Canvas not supported.
  </canvas>

  <script>
    const canvas = document.getElementById('textCanvas');
    const ctx = canvas.getContext('2d');

    ctx.font = '24px sans-serif';
    ctx.fillStyle = '#333';
    ctx.fillText('Hello, Canvas!', 50, 60);
  </script>
</body>
</html>
```

**Expected output**: The text “Hello, Canvas!” rendered on the canvas in 24px sans-serif.

**Why this output occurs**: The context’s `font` and `fillStyle` are set, and `fillText()` draws the text at the specified coordinates.

---

### Real-World Cases

1. **Data visualisation**: Charts and graphs drawn with libraries like Chart.js.
2. **Games**: 2D and 3D browser games using Canvas 2D or WebGL.
3. **Image editing**: Client-side photo filters and cropping tools.
4. **Generative art**: Procedural graphics and animations.

---

### References

- MDN Web Docs — `<canvas>`: The Graphics Canvas element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas
- MDN Web Docs — `HTMLCanvasElement.getContext()` — https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/getContext
- MDN Web Docs — Canvas API — https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API

---

## 6. Scripting Element Best Practices

### Definitions

**Core Definition**
Scripting element best practices are authoring guidelines for loading, executing, and structuring scripts, fallbacks, templates, slots, and canvas content so that pages remain accessible, performant, secure, and maintainable.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG accessibility techniques, and web performance guidance. They encompass progressive enhancement, graceful degradation, execution-timing control, Subresource Integrity, accessibility of dynamic content, and the appropriate use of declarative alternatives.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for scripting elements. They ensure that your JavaScript does not block the page, that users without JavaScript still get a usable experience, and that dynamic content is accessible to everyone.

---

### Purposes

- **To maximise performance** by controlling script fetching and execution.
- **To ensure accessibility** of dynamic and canvas-rendered content.
- **To improve security** with Subresource Integrity and appropriate CORS settings.
- **To support progressive enhancement** so that core content works without scripts.
- **To maintainability** through declarative alternatives where possible.

---

### Syntax Rules and Structure

Best practices are applied across elements. The following table summarises the core rules.

| Practice | Rule |
|---|---|
| **Prefer declarative alternatives** | Use `<details>` for disclosure, native form validation where possible, and CSS for simple interactivity. |
| **Use `defer` for DOM-dependent scripts** | Scripts that query the DOM should be deferred so they run after parsing. |
| **Use `async` only for independent scripts** | Analytics, ads, and other scripts that do not depend on other scripts. |
| **Use `type="module"` with `nomodule` fallback** | Modern JavaScript with a legacy path. |
| **Provide meaningful `<noscript>` fallbacks** | Warn users or load alternative resources; do not rely on it for critical content exclusively. |
| **Clone template content before insertion** | Use `cloneNode(true)` to avoid moving the original fragment. |
| **Use named slots for flexible composition** | Unique slot names per shadow root; provide fallback content. |
| **Provide accessible alternatives for canvas** | Use fallback content, `aria-label`, or a text alternative outside the canvas. |
| **Use Subresource Integrity for third-party scripts** | `integrity` and `crossorigin` verify that fetched scripts have not been tampered with. |
| **Avoid inline event handlers** | Prefer `addEventListener` for maintainability and CSP compatibility. |

#### Syntax Rules

1. **Performance first**: Use `defer` or `async` to avoid parser-blocking; use `type="module"` for modern code splitting.
2. **Accessibility first**: Provide fallbacks, accessible names, and text alternatives for all scripted content.
3. **Security by default**: Use `integrity` for third-party scripts and avoid inline event handlers where CSP is enforced.
4. **Progressive enhancement**: Ensure core content and functionality work without JavaScript where feasible.

#### Constraints and Limitations

- Overusing `preload` or `async` can harm performance if resources are not needed immediately.
- `<noscript>` fallbacks are invisible to users with JavaScript, so critical content must not be exclusive to them.
- Canvas accessibility requires deliberate effort; screen readers cannot interpret bitmap content.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Well-Configured Script Loading Strategy

**Setup Guide**: Save as `best-practice-scripts.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Script Loading Best Practices</title>

  <!-- Critical CSS -->
  <link rel="stylesheet" href="styles.css">

  <!-- Modern module with legacy fallback -->
  <script type="module" src="app.mjs"></script>
  <script nomodule src="legacy-app.js"></script>

  <!-- Independent analytics loaded asynchronously -->
  <script async src="analytics.js"></script>

  <!-- Fallback warning for users without JavaScript -->
  <noscript>
    <style>.js-only { display: none; }</style>
  </noscript>
</head>
<body>
  <main>
    <h1>Script Loading Best Practices</h1>
    <p class="js-only">This paragraph requires JavaScript to be functional.</p>
  </main>
</body>
</html>
```

**Expected output**: The module script loads and executes in modern browsers; the legacy script runs only in older browsers. The analytics script loads asynchronously without blocking. When JavaScript is disabled, the `<noscript>` style hides `.js-only` content.

**Why this output occurs**: `type="module"` and `nomodule` provide a modern/legacy split. `async` makes analytics non-blocking. The `<noscript>` fallback adjusts presentation when scripting is unavailable.

---

#### Example 2: Accessible Canvas with Fallback

**Setup Guide**: Save as `best-practice-canvas.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Canvas</title>
</head>
<body>
  <canvas id="chart" width="400" height="300" role="img" aria-label="Bar chart showing sales increased by 20% this quarter">
    <!-- Fallback content for browsers without canvas and for screen readers -->
    <p>Bar chart: Sales increased by 20% this quarter.</p>
  </canvas>

  <script>
    const canvas = document.getElementById('chart');
    const ctx = canvas.getContext('2d');
    // Drawing commands for the chart…
  </script>
</body>
</html>
```

**Expected output**: The canvas renders the chart. Screen readers announce the `aria-label` describing the chart’s meaning, and browsers without canvas support display the fallback paragraph.

**Why this output occurs**: `role="img"` and `aria-label` give the canvas an accessible name and role. The fallback content provides a text alternative when canvas is unsupported. Together, they ensure the information is available to all users.

---

### Real-World Cases

1. **Performance-critical pages**: `defer` and `async` scripts reduce blocking and improve Core Web Vitals.
2. **Web Components libraries**: Templates and slots enable reusable, composable components.
3. **Data visualisation dashboards**: Canvas with accessible alternatives ensures charts are perceivable by all users.
4. **Security-sensitive applications**: SRI and CSP-compatible scripting practices protect against supply-chain attacks.

---

### References

- WHATWG — HTML Living Standard: Scripting — https://html.spec.whatwg.org/dev/scripting.html
- MDN Web Docs — `<script>`: The Script element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script
- MDN Web Docs — `<noscript>`: The Noscript element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/noscript
- MDN Web Docs — `<template>`: The Content Template element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template
- MDN Web Docs — `<slot>`: The Web Component Slot element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/slot
- MDN Web Docs — `<canvas>`: The Graphics Canvas element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas
- W3C — WCAG 2.2: Name, Role, Value — https://www.w3.org/WAI/WCAG22/Understanding/name-role-value.html

---

## Summary Table of Scripting Elements

| Element | Primary Role | Category | Key Attributes |
|---|---|---|---|
| `<script>` | Embedded or referenced executable code | Metadata, flow, phrasing, script-supporting | `src`, `type`, `async`, `defer`, `nomodule`, `integrity` |
| `<noscript>` | Script-disabled fallback | Metadata, flow, phrasing | global attributes only |
| `<template>` | Inert client-side template | Metadata, flow, phrasing, script-supporting | `id` (global attributes) |
| `<slot>` | Web Component content insertion point | — (shadow DOM) | `name` |
| `<canvas>` | Scriptable bitmap surface | Flow, phrasing, embedded, palpable | `width`, `height` |

---

## Final Notes on Deprecated, Obsolete, and Implementation-Dependent Features

- **`language` attribute on `<script>`**: Obsolete; use `type` instead.
- **`charset` attribute on `<script>`**: Obsolete; encoding is determined by the document.
- **`scoped` attribute on `<style>`**: Removed from the specification; not supported in modern browsers.
- **`<script>` in XHTML**: Requires CDATA sections or escaping of `<` and `&`; in HTML parsing, these are not needed.
- **`async` on inline scripts**: Has no effect; `async` is only meaningful with `src`.
- **`nomodule` on module scripts**: Only affects classic scripts; module scripts ignore it.
- **Canvas context types**: `"webgpu"` and `"bitmaprenderer"` have varying levels of browser support; feature detection is recommended.

---

This cheat sheet has covered the scripting elements — `<script>`, `<noscript>`, `<template>`, `<slot>`, and `<canvas>` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate scripting markup against the WHATWG HTML Living Standard and WCAG accessibility guidelines.