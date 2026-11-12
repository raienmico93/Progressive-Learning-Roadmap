# Custom Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Custom elements are a Web Components technology that allows developers to define their own fully-featured HTML elements, extending the set of elements available in the browser. They enable the creation of reusable, encapsulated components using standards-based HTML, CSS, and JavaScript.

**Technical Definition**
In the WHATWG HTML Living Standard, custom elements are defined as a way for authors to build their own fully-featured DOM elements. A custom element is implemented as a JavaScript class that extends `HTMLElement` (for autonomous custom elements) or a specific built-in element interface such as `HTMLImageElement` (for customized built-in elements). The class is registered against a tag name using `customElements.define()`. The browser then constructs instances of the class when it encounters the corresponding tag, and invokes lifecycle callbacks at defined moments. There are two types: **autonomous custom elements**, which are entirely new elements with names containing a hyphen, and **customized built-in elements**, which extend existing HTML elements and use the `is` attribute.

**Beginner-Friendly Explanation**
Custom elements let you invent your own HTML tags. Instead of being limited to tags like `<div>` or `<p>`, you can create `<my-card>`, `<user-profile>`, or `<flag-icon>` and define exactly how they behave using JavaScript. The browser treats them like real HTML elements, and you can use them anywhere in your markup.

---

### Key Characteristics

- **Developer-defined behaviour**: The element's functionality is written in JavaScript and attached to a class.
- **Hyphenated naming requirement**: Every custom element name must contain a hyphen (e.g. `my-element`), which reserves the namespace for future native HTML elements.
- **Two element types**: Autonomous elements (extend `HTMLElement`) and customized built-in elements (extend a specific built-in interface).
- **Lifecycle callbacks**: The browser invokes specific methods at key moments — construction, connection to the document, disconnection, attribute changes, and adoption into a new document.
- **Upgrade capability**: Elements can be defined after they appear in the DOM; the browser "upgrades" existing elements when the definition is registered.
- **Integration with Shadow DOM and templates**: Custom elements commonly use Shadow DOM for encapsulation and `<template>` for defining markup.

---

### Prerequisites

1. **HTML fundamentals**: familiarity with elements, attributes, and the DOM.
2. **JavaScript classes and inheritance**: custom elements are implemented as ES6 classes extending `HTMLElement`.
3. **The DOM API**: `document.createElement`, `appendChild`, `querySelector`, and event handling.
4. **Basic understanding of Web Components**: awareness of Shadow DOM and `<template>`.
5. **A modern browser**: Custom Elements v1 is supported in all current browsers (Chrome 54+, Firefox 63+, Safari 10.1+, Edge 79+).

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Web Components** | Custom elements are one of the three core Web Components specifications, alongside Shadow DOM and HTML Templates. |
| **JavaScript frameworks** | Lit, Stencil, FAST, and other libraries build on custom elements. |
| **Progressive enhancement** | The upgrade mechanism allows custom elements to enhance existing HTML content after the definition loads. |
| **Accessibility** | `ElementInternals` allows custom elements to expose default ARIA roles, states, and properties to assistive technologies. |
| **Form participation** | Form-associated custom elements can participate in form submission and validation using `ElementInternals` and `static formAssociated = true`. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Custom elements** — the general concept
2. **`<custom-element>`** — the tag name syntax and usage
3. **Web Components** — the broader technology suite
4. **Custom element naming** — the rules for valid tag names
5. **Custom element lifecycle** — the four lifecycle callbacks
6. **Custom element best practices** — quality and accessibility guidelines

---

## 1. Custom Elements — The General Concept

### Definitions

**Core Definition**
A custom element is a developer-defined HTML element whose behaviour is implemented in JavaScript and registered under a unique, hyphenated tag name.

**Technical Definition**
A custom element is a class that extends `HTMLElement` (autonomous) or a built-in element interface (customized built-in). It is registered via `customElements.define(name, constructor, options)`. The `CustomElementRegistry` interface provides `define()`, `get()`, `whenDefined()`, and `upgrade()`. The registry is accessed through `window.customElements`. Each custom element definition consists of a name, a constructor, a prototype, a list of observed attributes, and a map of lifecycle callbacks.

**Beginner-Friendly Explanation**
A custom element is a new HTML tag that you create yourself. You write a JavaScript class that tells the browser how the tag should work, and then you register it with a name you choose. After that, the browser treats it like any other HTML element.

---

### Purposes

- **To encapsulate reusable functionality** in a standards-based HTML element.
- **To extend the browser's element set** with application-specific components.
- **To provide a clean separation** between markup and behaviour.
- **To enable progressive enhancement** through the upgrade mechanism.
- **To integrate with forms and accessibility APIs** via `ElementInternals`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
// Define the class
class MyElement extends HTMLElement {
  constructor() {
    super();
    // Initialisation (no attributes or children here)
  }
}

// Register the element
customElements.define('my-element', MyElement);
```

**Component breakdown**

| Component | Description |
|---|---|
| `class MyElement extends HTMLElement` | The custom element class. For customized built-ins, extend the specific interface (e.g. `HTMLButtonElement`). |
| `constructor()` | Initialisation; must call `super()` first. Must not inspect attributes or children. |
| `customElements.define(name, constructor)` | Registers the element globally. |
| `name` | Must be a valid custom element name (lowercase, contains a hyphen). |

#### Syntax Rules

1. **The class must extend `HTMLElement`** (autonomous) or a built-in interface (customized built-in).
2. **The constructor must call `super()` first** and must not inspect attributes or children.
3. **Registration must use `customElements.define()`**; the name must be a valid custom element name.
4. **Customized built-in elements** use the `is` attribute in markup: `<button is="my-button">` and register with `{ extends: 'button' }`.

#### Constraints and Limitations

- The same name cannot be registered twice; a `NotSupportedError` is thrown.
- The same constructor cannot be registered under two different names.
- Custom elements cannot extend another custom element (yet); `extends` must refer to a built-in element.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Autonomous Custom Element

**Setup Guide**: Save as `minimal-element.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Minimal Custom Element</title>
</head>
<body>
  <!-- Use the custom element -->
  <my-greeting></my-greeting>

  <script>
    // Define the class
    class MyGreeting extends HTMLElement {
      constructor() {
        super(); // must call super() first
        // Create a shadow root and set content
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = '<p>Hello from a custom element!</p>';
      }
    }

    // Register the element
    customElements.define('my-greeting', MyGreeting);
  </script>
</body>
</html>
```

**Expected output**: The text “Hello from a custom element!” is rendered inside a `<my-greeting>` element.

**Why this output occurs**: The browser constructs an instance of `MyGreeting` when it encounters `<my-greeting>`. The constructor calls `super()`, attaches a shadow root, and sets its content. The element is then rendered in the document.

---

#### Example 2: A Form-Associated Custom Element

**Setup Guide**: Save as `form-element.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Form-Associated Custom Element</title>
</head>
<body>
  <form action="/submit" method="post">
    <label>
      <my-checkbox name="agreed"></my-checkbox>
      I agree to the terms.
    </label>
    <button type="submit">Submit</button>
  </form>

  <script>
    class MyCheckbox extends HTMLElement {
      static formAssociated = true; // makes it form-associated
      static observedAttributes = ['checked'];

      constructor() {
        super();
        this._internals = this.attachInternals(); // ElementInternals
        this.addEventListener('click', () => {
          this.checked = !this.checked;
        });
      }

      get checked() {
        return this.hasAttribute('checked');
      }

      set checked(flag) {
        this.toggleAttribute('checked', Boolean(flag));
      }

      attributeChangedCallback(name, oldValue, newValue) {
        if (name === 'checked') {
          this._internals.setFormValue(this.checked ? 'on' : null);
          this._internals.ariaChecked = this.checked;
        }
      }
    }

    customElements.define('my-checkbox', MyCheckbox);
  </script>
</body>
</html>
```

**Expected output**: A checkbox-like custom element that toggles when clicked. When the form is submitted, the value “on” is sent if checked, and the element participates in form submission.

**Why this output occurs**: `static formAssociated = true` makes the element form-associated. `attachInternals()` provides `ElementInternals`, which allows the element to set form values and ARIA states. The `attributeChangedCallback` syncs the `checked` attribute with the form value and `aria-checked`.

---

### Real-World Cases

1. **Design system components**: Buttons, cards, dialogs, and form controls are built as custom elements for reuse across applications.
2. **Third-party widgets**: Embedded widgets (e.g. chat widgets, analytics dashboards) use custom elements to avoid CSS and JavaScript conflicts.
3. **Progressive enhancement**: Content is authored in plain HTML and enhanced by custom elements when their definitions load asynchronously.

---

### References

- MDN Web Docs — Using custom elements — https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements
- WHATWG — Custom elements — https://html.spec.whatwg.org/multipage/custom-elements.html
- W3C — Custom Elements specification — https://www.w3.org/TR/custom-elements/

---

## 2. `<custom-element>` — The Tag Name Syntax and Usage

### Definitions

**Core Definition**
`<custom-element>` is the placeholder syntax for any valid custom element tag name. The actual name is chosen by the developer and must contain a hyphen.

**Technical Definition**
A custom element tag is an HTML element whose local name matches the `PotentialCustomElementName` production and has been registered in the `CustomElementRegistry`. In markup, it is written like any other element: `<my-element>`. For customized built-in elements, the built-in tag is used with an `is` attribute: `<button is="my-button">`

**Beginner-Friendly Explanation**
`<custom-element>` is just a stand-in for whatever name you choose — as long as it has a hyphen. It looks and behaves like a normal HTML tag once you register it.

---

### Purposes

- **To provide a declarative syntax** for using custom components in HTML.
- **To allow the browser's parser** to recognise and construct custom elements.
- **To support both autonomous and customized built-in usage**.

---

### Syntax Rules and Structure

#### Complete General Syntax (Autonomous)

```html
<my-element attribute="value">Content</my-element>
```

#### Complete General Syntax (Customized Built-In)

```html
<button is="my-button">Click me</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<my-element>` | The custom tag; name must contain a hyphen. |
| `is` attribute | Used for customized built-in elements; value is the custom name. |
| Attributes | Any attributes defined by the element; observed attributes trigger lifecycle callbacks. |

#### Syntax Rules

1. **Autonomous custom elements are written with their custom tag name**.
2. **Customized built-in elements use the built-in tag with `is="custom-name"`**.
3. **The `is` attribute must not be used on autonomous custom elements**; it has no effect.
4. **Custom element names are case-insensitive** but must be lowercase in markup.

#### Constraints and Limitations

- Unregistered custom element tags are treated as `HTMLElement` (or `HTMLUnknownElement`) and may not behave as expected.
- The `is` attribute is not supported in all browsers for all built-in elements.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Autonomous Tag Usage with Attributes

**Setup Guide**: Save as `autonomous-usage.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Autonomous Usage</title>
</head>
<body>
  <flag-icon country="nl"></flag-icon>

  <script>
    class FlagIcon extends HTMLElement {
      static observedAttributes = ['country'];

      attributeChangedCallback(name, oldValue, newValue) {
        this.textContent = `Flag: ${newValue.toUpperCase()}`;
      }
    }

    customElements.define('flag-icon', FlagIcon);
  </script>
</body>
</html>
```

**Expected output**: The text “Flag: NL” appears inside the `<flag-icon>` element.

**Why this output occurs**: The `country` attribute is observed via `static observedAttributes`. When the attribute is set (or changes), `attributeChangedCallback` fires and updates the text content.

---

#### Example 2: Customized Built-In Usage

**Setup Guide**: Save as `customized-builtin.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Customized Built-In</title>
</head>
<body>
  <button is="my-button">Click me</button>

  <script>
    class MyButton extends HTMLButtonElement {
      constructor() {
        super();
        this.addEventListener('click', () => {
          alert('Custom button clicked!');
        });
      }
    }

    customElements.define('my-button', MyButton, { extends: 'button' });
  </script>
</body>
</html>
```

**Expected output**: The button looks and behaves like a normal `<button>`, but clicking it shows an alert.

**Why this output occurs**: The class extends `HTMLButtonElement`, inheriting all button behaviour. The `{ extends: 'button' }` option registers it as a customized built-in, and the `is` attribute in markup instructs the browser to use the custom class for that button.

---

### Real-World Cases

1. **Component libraries**: Components are used as `<my-button>`, `<my-card>`, `<my-dialog>` in application templates.
2. **Third-party embeds**: Widgets are shipped as custom elements that consumers add with a single tag.
3. **Progressive enhancement**: Markup is authored with custom tags that later become interactive when the definition loads.

---

### References

- MDN Web Docs — Using custom elements — https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements
- WHATWG — Custom elements — https://html.spec.whatwg.org/multipage/custom-elements.html

---

## 3. Web Components — The Broader Technology Suite

### Definitions

**Core Definition**
Web Components is a suite of web platform APIs that allows developers to create reusable, encapsulated custom HTML elements. It consists of three main technologies: Custom Elements, Shadow DOM, and HTML Templates.

**Technical Definition**
Web Components is not a single specification but a collection of standards: the Custom Elements specification (defining and registering custom elements), the Shadow DOM specification (encapsulating DOM and styles), and the HTML Templates specification (`<template>` and `<slot>`). Together, these enable the creation of components that are self-contained, interoperable, and framework-agnostic.

**Beginner-Friendly Explanation**
Web Components are a set of browser features that let you build your own HTML tags with encapsulated styles and behaviour. Custom elements are the part that defines the tag and its behaviour; Shadow DOM hides the internals; and templates provide reusable markup.

---

### Purposes

- **To create reusable, encapsulated UI components** using web standards.
- **To achieve style and DOM encapsulation** with Shadow DOM.
- **To provide framework-agnostic components** that work in any HTML environment.
- **To enable interoperability** between different libraries and frameworks.

---

### Syntax Rules and Structure

A typical Web Component combines all three technologies:

```javascript
class MyCard extends HTMLElement {
  constructor() {
    super();
    // Shadow DOM for encapsulation
    this.attachShadow({ mode: 'open' });
    // Template for reusable markup
    const template = document.getElementById('my-card-template');
    this.shadowRoot.appendChild(template.content.cloneNode(true));
  }
}

customElements.define('my-card', MyCard);
```

**Component breakdown**

| Technology | Role |
|---|---|
| **Custom Elements** | Defines the tag and its lifecycle. |
| **Shadow DOM** | Encapsulates styles and markup inside the element. |
| **HTML Templates** | Provides inert, reusable markup for the component. |

#### Syntax Rules

1. **Shadow DOM** is attached via `this.attachShadow({ mode: 'open' })`.
2. **Templates** are defined with `<template>` and cloned with `content.cloneNode(true)`.
3. **Slots** (`<slot>`) project light-DOM content into the shadow tree.

#### Constraints and Limitations

- Shadow DOM styles are encapsulated by default; global styles do not penetrate.
- Custom elements are not a replacement for all components; simple elements may not need Shadow DOM or templates.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Complete Web Component with Shadow DOM and Template

**Setup Guide**: Save as `web-component.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Web Component</title>
</head>
<body>
  <my-card>
    <h2 slot="title">Card Title</h2>
    <p>Card body content.</p>
  </my-card>

  <template id="card-template">
    <style>
      .card { border: 1px solid #ccc; padding: 16px; margin: 8px; }
    </style>
    <div class="card">
      <slot name="title">Default Title</slot>
      <slot>Default body</slot>
    </div>
  </template>

  <script>
    class MyCard extends HTMLElement {
      constructor() {
        super();
        const template = document.getElementById('card-template');
        this.attachShadow({ mode: 'open' })
            .appendChild(template.content.cloneNode(true));
      }
    }
    customElements.define('my-card', MyCard);
  </script>
</body>
</html>
```

**Expected output**: A bordered card displaying “Card Title” and “Card body content.”, with styles encapsulated in the shadow root.

**Why this output occurs**: The custom element class attaches a shadow root and clones the template content into it. The `<slot>` elements project the light-DOM content into the shadow tree, and the `<style>` applies only within the shadow root.

---

### Real-World Cases

1. **Design systems**: Complete component libraries built with Web Components are portable across frameworks.
2. **Micro-frontends**: Independently deployed frontend modules use Web Components to avoid style conflicts.
3. **Embeddable widgets**: Third-party widgets use Web Components for isolation and ease of integration.

---

### References

- MDN Web Docs — Web Components — https://developer.mozilla.org/en-US/docs/Web/API/Web_components
- web.dev — Web Components — https://web.dev/learn/web-components

---

## 4. Custom Element Naming

### Definitions

**Core Definition**
Custom element naming refers to the rules that a tag name must follow to be a valid custom element name. The most distinctive rule is that the name must contain a hyphen.

**Technical Definition**
A valid custom element name must match the `PotentialCustomElementName` production: it must start with a lowercase ASCII letter, contain at least one hyphen (`-`), and not contain any uppercase ASCII letters. It must also not be one of the reserved names such as `annotation-xml`, `color-profile`, `font-face`, and others inherited from SVG and MathML. The hyphen serves as a namespace to prevent collisions with current and future native HTML elements.

**Beginner-Friendly Explanation**
Your custom element name must be all lowercase, start with a letter, and contain at least one hyphen. For example, `my-element` is valid, but `myelement` and `MyElement` are not.

---

### Purposes

- **To prevent naming collisions** with current and future native HTML elements.
- **To ensure the HTML parser treats the name as a tag** rather than text.
- **To support case-insensitive matching** in the HTML parser.
- **To provide a namespace mechanism** for third-party components.

---

### Syntax Rules and Structure

#### Complete General Syntax (Pattern)

```
[a-z] (PCENChar)* '-' (PCENChar)*
```

**Component breakdown**

| Component | Rule |
|---|---|
| First character | Must be a lowercase ASCII letter (`a`–`z`). |
| Hyphen | Must contain at least one `-`. |
| Subsequent characters | May include lowercase letters, digits, `-`, `.`, `_`, and certain Unicode characters. |
| Uppercase | Must not contain uppercase ASCII letters. |
| Reserved names | Must not be `annotation-xml`, `color-profile`, `font-face`, `font-face-src`, `font-face-uri`, `font-face-format`, `font-face-name`, `missing-glyph`. |

#### Syntax Rules

1. **Start with a lowercase letter**: `my-element` is valid; `1-element` is not.
2. **Contain a hyphen**: `my-element` is valid; `myelement` is not.
3. **All lowercase**: `my-element` is valid; `My-Element` is not.
4. **Do not use reserved names**: these are hyphenated names from SVG and MathML.

#### Constraints and Limitations

- The hyphen rule is mandatory; there are no exceptions for autonomous custom elements.
- Namespaces via prefixes (e.g. `myapp-card`) are encouraged to avoid collisions between libraries.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Valid vs Invalid Names

**Setup Guide**: Save as `naming-example.html`. The valid element works; the invalid ones do not.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Naming Rules</title>
</head>
<body>
  <!-- Valid: lowercase, contains a hyphen -->
  <my-valid-element></my-valid-element>

  <!-- Invalid: no hyphen -->
  <myvalidelement></myvalidelement>

  <!-- Invalid: starts with uppercase -->
  <My-Invalid-Element></My-Invalid-Element>

  <script>
    class MyValidElement extends HTMLElement {
      constructor() {
        super();
        this.textContent = 'Valid!';
      }
    }
    customElements.define('my-valid-element', MyValidElement);

    // The following would throw a SyntaxError:
    // customElements.define('myvalidelement', class extends HTMLElement {});
    // customElements.define('My-Invalid-Element', class extends HTMLElement {});
  </script>
</body>
</html>
```

**Expected output**: Only `<my-valid-element>` renders “Valid!”. The other tags are treated as unknown elements and do not have custom behaviour.

**Why this output occurs**: The name `my-valid-element` satisfies all naming rules. The others violate the hyphen or lowercase requirements, and `customElements.define()` would throw a `SyntaxError` for them.

---

### Real-World Cases

1. **Namespaced components**: Libraries use prefixes like `acme-button`, `acme-card` to avoid conflicts.
2. **Standardised prefixes**: Some organisations mandate a project-specific prefix for all custom elements.
3. **Reserved names**: Developers must check that their chosen name is not in the reserved list.

---

### References

- W3C — Custom Elements: Valid custom element names — https://www.w3.org/TR/custom-elements/#valid-custom-element-name
- WHATWG — Custom elements — https://html.spec.whatwg.org/multipage/custom-elements.html

---

## 5. Custom Element Lifecycle

### Definitions

**Core Definition**
The custom element lifecycle is the set of callbacks that the browser invokes at specific moments in a custom element's existence — when it is constructed, connected to the document, disconnected, adopted into a new document, and when its observed attributes change.

**Technical Definition**
The lifecycle callbacks are: `constructor()` (called when the element is created), `connectedCallback()` (called each time the element is connected to the document), `disconnectedCallback()` (called each time the element is disconnected), `adoptedCallback()` (called when the element is moved to a new document), and `attributeChangedCallback()` (called when an observed attribute is changed, appended, removed, or replaced). The `observedAttributes` static property lists the attributes that trigger `attributeChangedCallback`

**Beginner-Friendly Explanation**
Lifecycle callbacks are methods you can add to your custom element class that the browser calls automatically. For example, `connectedCallback` runs when the element is added to the page, and `disconnectedCallback` runs when it is removed. `attributeChangedCallback` runs when one of the attributes you're watching changes.

---

### Purposes

- **To run setup code when the element is connected** (`connectedCallback`).
- **To clean up resources when the element is removed** (`disconnectedCallback`).
- **To react to attribute changes** (`attributeChangedCallback`).
- **To handle document adoption** (`adoptedCallback`).
- **To initialise state** in the constructor without accessing attributes or children.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
class MyElement extends HTMLElement {
  static observedAttributes = ['attr1', 'attr2'];

  constructor() {
    super();
    // Initialisation: do NOT inspect attributes or children
  }

  connectedCallback() {
    // Element added to the document
  }

  disconnectedCallback() {
    // Element removed from the document
  }

  adoptedCallback(oldDocument, newDocument) {
    // Element moved to a new document
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // An observed attribute changed
  }
}
```

**Component breakdown**

| Callback | When it runs | Key constraint |
|---|---|---|
| `constructor()` | When the element is created. | Must call `super()` first; must not inspect attributes or children. |
| `connectedCallback()` | Each time the element is connected to the document. | Can run multiple times; do setup here rather than in the constructor. |
| `disconnectedCallback()` | Each time the element is disconnected. | Use for cleanup (remove listeners, abort fetches). |
| `adoptedCallback(oldDoc, newDoc)` | When the element is adopted into a new document. | Rarely used; useful for reacting to document changes. |
| `attributeChangedCallback(name, old, new)` | When an observed attribute changes. | Only fires for attributes listed in `static observedAttributes`. |

#### Syntax Rules

1. **Call `super()` first** in the constructor.
2. **Do not inspect attributes or children in the constructor**; wait for `connectedCallback` or `attributeChangedCallback`.
3. **`connectedCallback` may run multiple times** if the element is moved or re-inserted.
4. **Use `disconnectedCallback` for cleanup** to prevent memory leaks.
5. **`attributeChangedCallback` only fires for attributes in `observedAttributes`**.

#### Constraints and Limitations

- The constructor must not add children or set attributes; this is required by the specification.
- `connectedCallback` is not called for elements created with `document.createElement` until they are inserted into the document.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Lifecycle Callbacks in Action

**Setup Guide**: Save as `lifecycle-example.html`. Open the console and observe the log messages.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Lifecycle Example</title>
</head>
<body>
  <lifecycle-demo name="Alice"></lifecycle-demo>

  <script>
    class LifecycleDemo extends HTMLElement {
      static observedAttributes = ['name'];

      constructor() {
        super();
        console.log('constructor: element created');
      }

      connectedCallback() {
        console.log('connectedCallback: element added to page');
      }

      disconnectedCallback() {
        console.log('disconnectedCallback: element removed from page');
      }

      adoptedCallback(oldDoc, newDoc) {
        console.log('adoptedCallback: element moved to new document');
      }

      attributeChangedCallback(name, oldValue, newValue) {
        console.log(`attributeChangedCallback: ${name} changed from ${oldValue} to ${newValue}`);
      }
    }

    customElements.define('lifecycle-demo', LifecycleDemo);

    // Trigger an attribute change
    setTimeout(() => {
      document.querySelector('lifecycle-demo').setAttribute('name', 'Bob');
    }, 1000);
  </script>
</body>
</html>
```

**Expected output**: The console logs:
1. `constructor: element created`
2. `connectedCallback: element added to page`
3. `attributeChangedCallback: name changed from Alice to Bob`

**Why this output occurs**: The element is constructed, connected to the document, and then its `name` attribute changes after a delay. Each event invokes the corresponding callback.

---

#### Example 2: Cleanup in `disconnectedCallback`

**Setup Guide**: Save as `cleanup-example.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Cleanup Example</title>
</head>
<body>
  <cleanup-demo></cleanup-demo>
  <button onclick="removeElement()">Remove Element</button>

  <script>
    class CleanupDemo extends HTMLElement {
      connectedCallback() {
        this._interval = setInterval(() => {
          console.log('tick');
        }, 1000);
      }

      disconnectedCallback() {
        clearInterval(this._interval);
        console.log('Interval cleared');
      }
    }

    customElements.define('cleanup-demo', CleanupDemo);

    function removeElement() {
      const el = document.querySelector('cleanup-demo');
      el.remove();
    }
  </script>
</body>
</html>
```

**Expected output**: A “tick” message every second while the element is present. When “Remove Element” is clicked, “Interval cleared” is logged and the ticks stop.

**Why this output occurs**: `connectedCallback` starts an interval. `disconnectedCallback` clears it when the element is removed, preventing a memory leak.

---

### Real-World Cases

1. **Event listener management**: Add listeners in `connectedCallback` and remove them in `disconnectedCallback`.
2. **Data fetching**: Fetch data when the element connects and abort the request when it disconnects.
3. **Attribute-driven rendering**: Update the element's view in `attributeChangedCallback` whenever observed attributes change.

---

### References

- MDN Web Docs — Using custom elements: Lifecycle callbacks — https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements#lifecycle_callbacks
- WHATWG — Custom element reactions — https://html.spec.whatwg.org/multipage/custom-elements.html#custom-element-reactions
- Chrome for Developers — Custom elements — https://developer.chrome.com/docs/web-platform/custom-elements

---

## 6. Custom Element Best Practices

### Definitions

**Core Definition**
Custom element best practices are a set of guidelines for building custom elements that are robust, accessible, well-integrated with HTML and CSS, and easy to consume.

**Technical Definition**
The web.dev Custom Element Best Practices checklist identifies key areas: Shadow DOM encapsulation, constructor discipline, attribute and property reflection, event dispatch, and accessibility. These practices ensure that custom elements behave like native elements, interoperate cleanly with the platform, and avoid common pitfalls such as flashing unstyled content, breaking attribute/property sync, or leaking memory.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for building custom elements. They help your components work well with the rest of the page, behave predictably, and be accessible to everyone.

---

### Purposes

- **To ensure encapsulation** of styles and markup via Shadow DOM.
- **To maintain attribute/property synchronization** so the element works from both HTML and JavaScript.
- **To avoid flashing unstyled content** when the element is not yet defined.
- **To respect global attributes** such as `hidden`, `class`, and `id`.
- **To properly dispatch events** and handle cleanup.
- **To make custom elements accessible** with appropriate roles and states.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Create a shadow root in the constructor** | Encapsulate styles and DOM; use `this.attachShadow({ mode: 'open' })`. |
| **Set a `:host` display style** | Unless the default of `inline` is desired; respect the `hidden` attribute. |
| **Do not inspect attributes or children in the constructor** | Wait for `connectedCallback`. |
| **Accept primitive data as attributes or properties** | Keep attributes and properties in sync. |
| **Reflect properties to attributes** | So the element works from HTML markup as well as JavaScript. |
| **Do not reflect rich data (objects, arrays) to attributes** | Attributes are strings; use properties for rich data. |
| **Hide undefined elements to avoid FOUC** | Use `:not(:defined) { visibility: hidden; }` or a similar technique. |
| **Do not override global author-set attributes** | Respect `class`, `id`, `style`, `hidden`, and ARIA attributes. |
| **Dispatch events for internal activity** | Use `CustomEvent` with `bubbles: true` and `composed: true` where appropriate. |
| **Clean up in `disconnectedCallback`** | Remove listeners, clear intervals, abort fetches. |
| **Use `ElementInternals` for accessibility** | Set default roles and states for form-associated and other custom elements. |

#### Syntax Rules

1. **Constructor discipline**: Call `super()` first; do not inspect attributes or children.
2. **Property/attribute sync**: Implement getters and setters that reflect to attributes.
3. **Shadow DOM encapsulation**: Use Shadow DOM unless there is a reason not to.
4. **Progressive enhancement**: Ensure the element has meaningful fallback content or is hidden until defined.

#### Constraints and Limitations

- Reflecting rich data to attributes is not recommended; attributes are string-only.
- Overriding global attributes can break consumer expectations and accessibility.
- Shadow DOM encapsulation prevents global styles from penetrating; this is intentional but must be managed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Attribute/Property Reflection

**Setup Guide**: Save as `reflection-example.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Reflection Example</title>
  <style>
    :not(:defined) { visibility: hidden; }
  </style>
</head>
<body>
  <my-counter count="5"></my-counter>

  <script>
    class MyCounter extends HTMLElement {
      static observedAttributes = ['count'];

      constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = '<p>Count: <span id="value"></span></p>';
      }

      get count() {
        return Number(this.getAttribute('count')) || 0;
      }

      set count(value) {
        this.setAttribute('count', String(value));
      }

      attributeChangedCallback(name, oldValue, newValue) {
        if (name === 'count') {
          this.shadowRoot.getElementById('value').textContent = newValue;
        }
      }

      connectedCallback() {
        this.shadowRoot.getElementById('value').textContent = this.count;
      }
    }

    customElements.define('my-counter', MyCounter);
  </script>
</body>
</html>
```

**Expected output**: The counter displays “Count: 5”. Setting `element.count = 10` in the console updates both the attribute and the displayed value.

**Why this output occurs**: The getter and setter synchronise the `count` property with the `count` attribute. `attributeChangedCallback` updates the rendering whenever the attribute changes. The `:not(:defined)` rule hides the element until its definition loads, preventing a flash of unstyled content.

---

#### Example 2: Accessible Form-Associated Element

**Setup Guide**: Save as `accessible-checkbox.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Checkbox</title>
</head>
<body>
  <form>
    <label>
      <my-checkbox name="agree"></my-checkbox>
      I agree
    </label>
    <button type="submit">Submit</button>
  </form>

  <script>
    class MyCheckbox extends HTMLElement {
      static formAssociated = true;
      static observedAttributes = ['checked'];

      constructor() {
        super();
        this._internals = this.attachInternals();
        this._internals.role = 'checkbox';
        this._internals.ariaChecked = 'false';
        this.addEventListener('click', () => {
          this.checked = !this.checked;
        });
      }

      get checked() {
        return this.hasAttribute('checked');
      }

      set checked(flag) {
        this.toggleAttribute('checked', Boolean(flag));
      }

      attributeChangedCallback(name, oldValue, newValue) {
        if (name === 'checked') {
          this._internals.setFormValue(this.checked ? 'on' : null);
          this._internals.ariaChecked = this.checked;
        }
      }
    }

    customElements.define('my-checkbox', MyCheckbox);
  </script>
</body>
</html>
```

**Expected output**: A checkbox-like element that is announced as a checkbox by screen readers, toggles when clicked, and participates in form submission.

**Why this output occurs**: `ElementInternals` sets the default ARIA role to `checkbox` and the `aria-checked` state. `setFormValue` ensures the element's value is included in form submission. This follows the accessibility best practice for custom elements.

---

### Real-World Cases

1. **Design system components**: Following best practices ensures components work consistently across teams and applications.
2. **Third-party component libraries**: Libraries like Lit and FAST codify these practices into their base classes.
3. **Accessibility-critical applications**: `ElementInternals` and proper ARIA defaults are essential for screen-reader users.

---

### References

- web.dev — Custom Element Best Practices — https://web.dev/articles/custom-elements-best-practices
- WHATWG — Custom elements: Requirements for custom element constructors and reactions — https://html.spec.whatwg.org/multipage/custom-elements.html#custom-element-conformance
- Chrome for Developers — Custom elements — https://developer.chrome.com/docs/web-platform/custom-elements
- MDN Web Docs — Using custom elements — https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements

---

## Summary Table of Key Aspects

| Aspect | Key Takeaway |
|---|---|
| **Definition** | Developer-defined HTML elements registered via `customElements.define()`. |
| **Tag syntax** | `<my-element>` (autonomous) or `<button is="my-button">` (customized built-in). |
| **Naming** | Must be lowercase, start with a letter, and contain a hyphen. |
| **Lifecycle callbacks** | `constructor`, `connectedCallback`, `disconnectedCallback`, `adoptedCallback`, `attributeChangedCallback`. |
| **Upgrade** | Elements defined after parsing are upgraded when the definition loads. |
| **Best practices** | Shadow DOM encapsulation, attribute/property sync, cleanup in `disconnectedCallback`, `ElementInternals` for accessibility. |

---

## Final Notes

- **Browser support**: Custom Elements v1 is supported in all modern browsers (Chrome 54+, Firefox 63+, Safari 10.1+, Edge 79+).
- **Web Components ecosystem**: Custom elements are the foundation of the Web Components suite, working alongside Shadow DOM and HTML Templates.
- **Framework interoperability**: Custom elements are framework-agnostic and can be used with React, Vue, Angular, Svelte, and vanilla JavaScript.
- **Progressive enhancement**: The upgrade mechanism allows custom elements to enhance existing markup without requiring a full page rewrite.

---

This cheat sheet has covered custom elements, the `<custom-element>` syntax, Web Components, naming rules, lifecycle callbacks, and best practices with annotated examples and authoritative references. Use it as a reference and a teaching aid, and validate custom element implementations against the WHATWG HTML Living Standard and the web.dev best practices checklist.