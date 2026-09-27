# HTML DOM Fundamentals: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

The Document Object Model (DOM) is a programming interface for web documents that represents the HTML document as a tree of objects, allowing programs to read, manipulate, and update the document's structure, style, and content.

**Technical Definition**

The Document Object Model is defined by the WHATWG DOM Living Standard as a platform-neutral interface that treats an HTML or XML document as a tree structure wherein each node is an object representing a part of the document. The DOM represents a document with a logical tree, where each branch of the tree ends in a node, and each node contains objects. The `Document` interface serves as the entry point to the DOM tree, providing methods like `getElementById()`, `querySelector()`, `createElement()`, and `addEventListener()`. The DOM is language-agnostic — it can be manipulated by JavaScript, Python, Java, and other languages — but JavaScript is the most common interface in web browsers. The DOM is not part of JavaScript; it is a separate Web API that browsers implement.

**Beginner-Friendly Explanation**

When a browser loads your HTML, it doesn't just display it — it builds a family tree of all the elements. At the top is the `document`. Inside it are `<html>`, `<head>`, and `<body>`. Inside those are headings, paragraphs, divs, and so on. This tree is called the DOM. JavaScript can walk this tree, find any element, change its text, add styles, attach click handlers, or even create new elements from scratch. The DOM is the bridge between your static HTML and dynamic, interactive JavaScript.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Tree structure** | The DOM is a hierarchical tree of nodes representing the document |
| **Live representation** | Changes to the DOM are immediately reflected in the rendered page |
| **Language-agnostic** | The DOM is a Web API, not a JavaScript-specific feature |
| **Node-based** | Everything in the DOM is a node (element, text, comment, document) |
| **Event-driven** | User interactions trigger events that scripts can listen for |
| **Traversable** | Nodes have parent, child, and sibling relationships |
| **Manipulable** | Elements can be created, modified, moved, and deleted |
| **Attribute-driven** | Elements expose attributes as properties and via methods |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Basic knowledge of JavaScript syntax (variables, functions, objects)
- Awareness of how browsers parse HTML into a DOM tree
- Basic understanding of CSS selectors (for `querySelector`)

---

### Related Programming Areas

- **JavaScript** – The primary language for DOM manipulation in browsers
- **CSS** – The DOM's `style` property and `classList` API interact with CSS
- **Web Accessibility (A11y)** – DOM manipulation must preserve accessibility semantics
- **Web Performance** – Excessive DOM manipulation causes reflows and repaints
- **Component Frameworks** – React, Vue, and Angular abstract the DOM with virtual representations
- **Web APIs** – The DOM is one of many browser APIs (alongside Fetch, Storage, Canvas, etc.)

---

## Core Concepts / Features

---

### 1. The Document Object Model (DOM)

#### Definitions

**Core Definition**

The Document Object Model is a tree-structured, programmatic representation of an HTML (or XML) document, constructed by the browser during parsing, that exposes the document's content and structure to scripts.

**Technical Definition**

The WHATWG DOM Living Standard defines the DOM as a platform-neutral interface. When an HTML document is parsed, the browser constructs a DOM tree where each element, text fragment, comment, and attribute is represented as a node. The root of the tree is the `Document` node, accessible via the global `document` object. The `Document` interface inherits from `Node`, which provides properties like `parentNode`, `childNodes`, `firstChild`, and `lastChild`, and methods like `appendChild()`, `removeChild()`, and `cloneNode()`. The `Element` interface extends `Node` with element-specific properties and methods. Nodes have a `nodeType` property (1 = Element, 3 = Text, 8 = Comment, 9 = Document).

**Beginner-Friendly Explanation**

Imagine your HTML as a family tree. The `<html>` element is the grandparent. `<head>` and `<body>` are its children. The elements inside `<body>` are grandchildren, and so on. Each element, each piece of text, and each comment is a "node" in this tree. The DOM gives you tools to walk this tree: find a node, change its text, add a new child, remove a child, or move a node somewhere else. When you change the DOM, the browser updates what you see on screen.

#### Purposes

- To provide a programmatic interface to the document's structure and content
- To enable dynamic updates to the page without reloading
- To allow scripts to respond to user interactions
- To serve as the bridge between HTML, CSS, and JavaScript
- To enable traversal and manipulation of the document tree

#### Syntax Rules and Structure

**The DOM Tree**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>DOM Example</title>
</head>
<body>
    <h1 id="title">Hello</h1>
    <p class="intro">Welcome to the DOM.</p>
    <ul id="list">
        <li>Item 1</li>
        <li>Item 2</li>
    </ul>
</body>
</html>
```

**Resulting DOM Tree (Simplified)**

```
document
└── html
    ├── head
    │   ├── meta
    │   └── title
    │       └── "DOM Example" (text node)
    └── body
        ├── h1#title
        │   └── "Hello" (text node)
        ├── p.intro
        │   └── "Welcome to the DOM." (text node)
        └── ul#list
            ├── li
            │   └── "Item 1" (text node)
            └── li
                └── "Item 2" (text node)
```

**Node Types**

| nodeType | Constant | Description |
|---|---|---|
| 1 | `Node.ELEMENT_NODE` | An element (e.g., `<div>`, `<p>`) |
| 3 | `Node.TEXT_NODE` | Text content |
| 8 | `Node.COMMENT_NODE` | A comment (`<!-- -->`) |
| 9 | `Node.DOCUMENT_NODE` | The document itself |
| 10 | `Node.DOCUMENT_TYPE_NODE` | The DOCTYPE |

**Key Node Properties**

| Property | Description |
|---|---|
| `nodeType` | Numeric type of the node |
| `nodeName` | Name (e.g., `DIV`, `#text`) |
| `parentNode` | Parent node |
| `childNodes` | Live `NodeList` of children |
| `firstChild` / `lastChild` | First/last child node |
| `nextSibling` / `previousSibling` | Adjacent sibling nodes |
| `textContent` | Text content of the node and descendants |

**Syntax Rules**

- The `document` object is the entry point to the DOM
- The DOM is live: changes are immediately reflected
- `childNodes` returns all node types; `children` returns only elements
- Node relationships are bidirectional (parent knows children, children know parent)
- Nodes can be moved (appending an existing node moves it)

**Constraints and Limitations**

- Excessive DOM manipulation causes performance issues (reflows/repaints)
- `childNodes` includes text nodes (whitespace), which can surprise developers
- The DOM is not the same as the HTML source (it is the parsed, corrected representation)
- Direct DOM manipulation can conflict with frameworks (React, Vue)

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Traversing the DOM Tree**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>DOM Traversal</title>
</head>
<body>
    <div id="container">
        <h1>Title</h1>
        <p>First paragraph.</p>
        <p>Second paragraph.</p>
    </div>

    <script>
        // Get the container element
        const container = document.getElementById('container');

        // Access children (elements only)
        console.log('Children:', container.children.length); // 3

        // Access all child nodes (including text nodes)
        console.log('Child nodes:', container.childNodes.length); // 7 (includes whitespace)

        // Access first and last child elements
        console.log('First child:', container.firstElementChild.tagName); // H1
        console.log('Last child:', container.lastElementChild.tagName);   // P

        // Access parent
        console.log('Parent:', container.parentNode.tagName); // BODY

        // Access siblings
        const h1 = container.firstElementChild;
        console.log('Next sibling:', h1.nextElementSibling.tagName); // P
        console.log('Previous sibling:', h1.previousElementSibling); // null
    </script>
</body>
</html>
```

**Expected Output (Console)**

```
Children: 3
Child nodes: 7
First child: H1
Last child: P
Parent: BODY
Next sibling: P
Previous sibling: null
```

**Why This Output Occurs**

`children` returns only element children (3). `childNodes` returns all nodes including text nodes (whitespace between elements), which is why it returns 7. The `nextElementSibling` skips text nodes, returning the next element.

---

**Example 2: Creating and Appending Nodes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Creating Nodes</title>
</head>
<body>
    <ul id="list">
        <li>Existing item</li>
    </ul>

    <script>
        const list = document.getElementById('list');

        // Create a new list item
        const newItem = document.createElement('li');
        newItem.textContent = 'New item';

        // Append it to the list
        list.appendChild(newItem);

        // Create and insert before an existing child
        const firstItem = document.createElement('li');
        firstItem.textContent = 'First item';
        list.insertBefore(firstItem, list.firstElementChild);

        // Clone a node
        const clone = newItem.cloneNode(true);
        clone.textContent = 'Cloned item';
        list.appendChild(clone);
    </script>
</body>
</html>
```

**Expected Output**

The list displays:
- First item
- Existing item
- New item
- Cloned item

**Why This Output Occurs**

`createElement()` creates a new element node. `appendChild()` adds it as the last child. `insertBefore()` inserts it before a reference node. `cloneNode(true)` creates a deep copy.

#### Real-World Cases

**Case 1: Single-Page Applications**

SPAs use the DOM to render views, update content, and respond to user interactions without page reloads.

**Case 2: Form Validation**

Forms use DOM manipulation to add error messages, highlight invalid fields, and disable submit buttons.

**Case 3: Interactive Widgets**

Tabs, accordions, modals, and carousels rely on DOM manipulation to show/hide content and update state.

---

### 2. Element Selection

#### Definitions

**Core Definition**

Element selection is the process of retrieving one or more DOM nodes from the document using methods like `querySelector()`, `querySelectorAll()`, and `getElementById()`.

**Technical Definition**

The DOM provides multiple methods for selecting elements. Modern methods use CSS selector syntax: `document.querySelector(selector)` returns the first matching element or `null`; `document.querySelectorAll(selector)` returns a static `NodeList` of all matching elements. Legacy methods include `document.getElementById(id)` (returns a single element by ID), `document.getElementsByClassName(className)` (returns a live `HTMLCollection`), `document.getElementsByTagName(tagName)` (returns a live `HTMLCollection`), and `document.getElementsByName(name)`. Modern methods (`querySelector`/`querySelectorAll`) are preferred because they use the full power of CSS selectors. The `NodeList` returned by `querySelectorAll` is static (does not update when the DOM changes), while `HTMLCollection` is live.

**Beginner-Friendly Explanation**

Element selection is how you find the elements you want to work with. If you want to change the text of a heading, you first need to find that heading in the DOM. You can search by ID (`getElementById`), by class (`getElementsByClassName`), by tag name (`getElementsByTagName`), or by any CSS selector (`querySelector` and `querySelectorAll`). Modern code prefers `querySelector` and `querySelectorAll` because they use the same selector syntax you already know from CSS.

#### Purposes

- To locate specific elements for manipulation
- To select groups of elements for batch operations
- To traverse the DOM using CSS selector syntax
- To enable event delegation on parent elements
- To query elements dynamically as the DOM changes

#### Syntax Rules and Structure

**Modern Methods**

```javascript
// Returns the first matching element (or null)
const firstParagraph = document.querySelector('p');
const byClass = document.querySelector('.intro');
const byId = document.querySelector('#title');
const complex = document.querySelector('ul > li:first-child');

// Returns a static NodeList of all matching elements
const allParagraphs = document.querySelectorAll('p');
const allButtons = document.querySelectorAll('.btn');
```

**Legacy Methods**

```javascript
// Returns a single element by ID
const title = document.getElementById('title');

// Returns a live HTMLCollection
const paragraphs = document.getElementsByTagName('p');
const items = document.getElementsByClassName('item');
```

**Component Breakdown**

| Method | Returns | Live? |
|---|---|---|
| `querySelector(sel)` | First matching `Element` or `null` | — |
| `querySelectorAll(sel)` | Static `NodeList` | No |
| `getElementById(id)` | `Element` or `null` | — |
| `getElementsByClassName(cls)` | `HTMLCollection` | Yes |
| `getElementsByTagName(tag)` | `HTMLCollection` | Yes |
| `getElementsByName(name)` | `NodeList` | Yes |

**Syntax Rules**

- `querySelector` and `querySelectorAll` accept any valid CSS selector
- `querySelectorAll` returns a `NodeList` (use `forEach()` or convert to array)
- `getElementsBy*` methods return live `HTMLCollection` (updates automatically)
- `getElementById` is the fastest method for selecting by ID
- All selection methods are available on `document` and on any `Element`

**Constraints and Limitations**

- `NodeList` from `querySelectorAll` is static; it does not update when the DOM changes
- `HTMLCollection` is live; it updates automatically but can cause performance issues
- `querySelectorAll` can throw `SyntaxError` for invalid selectors
- Selecting elements before the DOM is ready returns `null` or empty collections

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Modern Selection Methods**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Element Selection</title>
</head>
<body>
    <h1 id="title">Page Title</h1>
    <p class="intro">First paragraph.</p>
    <p class="intro">Second paragraph.</p>
    <ul class="nav">
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
    </ul>

    <script>
        // By ID
        const title = document.getElementById('title');
        console.log(title.textContent); // "Page Title"

        // By class (first match)
        const firstIntro = document.querySelector('.intro');
        console.log(firstIntro.textContent); // "First paragraph."

        // All matches (static NodeList)
        const allIntros = document.querySelectorAll('.intro');
        console.log(allIntros.length); // 2
        allIntros.forEach(p => console.log(p.textContent));

        // Complex CSS selector
        const firstNavLink = document.querySelector('.nav li:first-child a');
        console.log(firstNavLink.textContent); // "Home"

        // All links inside nav
        const navLinks = document.querySelectorAll('.nav a');
        console.log(navLinks.length); // 2
    </script>
</body>
</html>
```

**Expected Output (Console)**

```
Page Title
First paragraph.
2
First paragraph.
Second paragraph.
Home
2
```

**Why This Output Occurs**

`getElementById` returns the element with the matching ID. `querySelector` returns the first match. `querySelectorAll` returns all matches as a static `NodeList`, which supports `forEach`.

---

**Example 2: Live vs. Static Collections**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Live vs Static</title>
</head>
<body>
    <ul id="list">
        <li>Item 1</li>
        <li>Item 2</li>
    </ul>

    <script>
        const list = document.getElementById('list');

        // Static NodeList (querySelectorAll)
        const staticItems = document.querySelectorAll('#list li');
        console.log('Static before:', staticItems.length); // 2

        // Live HTMLCollection (getElementsByTagName)
        const liveItems = list.getElementsByTagName('li');
        console.log('Live before:', liveItems.length); // 2

        // Add a new item
        const newItem = document.createElement('li');
        newItem.textContent = 'Item 3';
        list.appendChild(newItem);

        // Static is unchanged; live is updated
        console.log('Static after:', staticItems.length); // 2
        console.log('Live after:', liveItems.length); // 3
    </script>
</body>
</html>
```

**Expected Output (Console)**

```
Static before: 2
Live before: 2
Static after: 2
Live after: 3
```

**Why This Output Occurs**

`querySelectorAll` returns a static `NodeList` — it does not update when the DOM changes. `getElementsByTagName` returns a live `HTMLCollection` — it reflects the current DOM state.

#### Real-World Cases

**Case 1: Form Validation**

`document.querySelectorAll('input[required]')` selects all required inputs for validation.

**Case 2: Event Delegation**

Attaching a listener to a parent and using `event.target.closest()` to find the clicked element.

**Case 3: Dynamic Content**

Selecting newly added elements after an AJAX request using `querySelector`.

---

### 3. Event Handling

#### Definitions

**Core Definition**

Event handling is the process of attaching listeners to DOM elements that respond to user interactions (clicks, keypresses, mouse movements) or browser events (load, resize).

**Technical Definition**

The DOM Event API is defined by the WHATWG DOM Living Standard and the UI Events specification. The `addEventListener(type, listener, options)` method attaches an event listener to an `EventTarget` (elements, `document`, `window`, etc.). The `type` is the event name (e.g., `click`, `keydown`, `submit`), and the `listener` is a function or object with a `handleEvent()` method. The `options` parameter can specify `capture` (use capture phase), `once` (remove after first invocation), `passive` (hint that the listener won't call `preventDefault()`), and `signal` (an `AbortSignal` to remove the listener). Events propagate through three phases: capture (from `window` to target), target (at the target), and bubble (from target back to `window`). The `event` object provides properties (`target`, `currentTarget`, `type`, `key`, `clientX`) and methods (`preventDefault()`, `stopPropagation()`).

**Beginner-Friendly Explanation**

Event handling is how you make your page respond to the user. When someone clicks a button, you want something to happen. You attach an "event listener" to the button that says "when this is clicked, run this function." The function receives an event object with details about what happened. This is how buttons, forms, keyboard shortcuts, and interactive widgets work.

#### Purposes

- To respond to user interactions (clicks, keys, mouse, touch)
- To handle form submissions and validation
- To react to browser events (load, resize, scroll)
- To implement interactive UI components
- To enable event delegation and dynamic content

#### Syntax Rules and Structure

**General Syntax**

```javascript
element.addEventListener(type, listener, options);
```

**Basic Example**

```javascript
const button = document.querySelector('#btn');

button.addEventListener('click', function(event) {
    console.log('Clicked!', event.target);
});
```

**Options Parameter**

| Option | Description |
|---|---|
| `capture` | If `true`, use capture phase (default: `false`) |
| `once` | Remove listener after first invocation |
| `passive` | Hint that `preventDefault()` won't be called |
| `signal` | `AbortSignal` to remove the listener |

**Common Events**

| Event | Description |
|---|---|
| `click` | Mouse click |
| `dblclick` | Double click |
| `keydown` / `keyup` | Keyboard press/release |
| `input` | Value changes in an input |
| `change` | Value committed (e.g., select, checkbox) |
| `submit` | Form submission |
| `focus` / `blur` | Element gains/loses focus |
| `mouseover` / `mouseout` | Mouse enters/leaves |
| `DOMContentLoaded` | DOM parsed and ready |
| `load` | All resources loaded |

**Syntax Rules**

- `addEventListener` can be called multiple times for the same event (multiple listeners)
- Use `removeEventListener` with the same function reference to remove a listener
- The `event` object is passed automatically to the listener
- `event.target` is the element that triggered the event
- `event.currentTarget` is the element the listener is attached to
- `preventDefault()` cancels the default action
- `stopPropagation()` stops the event from bubbling

**Constraints and Limitations**

- Anonymous functions cannot be removed with `removeEventListener`
- `passive: true` prevents `preventDefault()` (useful for scroll performance)
- Event listeners attached before the element exists will fail
- Memory leaks can occur if listeners are not removed when elements are removed

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Click Handler**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Click Handler</title>
</head>
<body>
    <button id="btn">Click me</button>
    <p id="output">No clicks yet.</p>

    <script>
        const btn = document.getElementById('btn');
        const output = document.getElementById('output');
        let count = 0;

        btn.addEventListener('click', (event) => {
            count++;
            output.textContent = `Clicked ${count} time(s).`;
            console.log('Event target:', event.target.tagName); // BUTTON
        });
    </script>
</body>
</html>
```

**Expected Output**

Each click increments the counter and updates the paragraph text.

**Why This Output Occurs**

The `click` listener runs on every click. The `event` object provides the target (the button). The `count` variable persists because it's in the closure.

---

**Example 2: Event Delegation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Event Delegation</title>
</head>
<body>
    <ul id="list">
        <li data-id="1">Item 1</li>
        <li data-id="2">Item 2</li>
        <li data-id="3">Item 3</li>
    </ul>

    <script>
        const list = document.getElementById('list');

        // Single listener on the parent handles all child clicks
        list.addEventListener('click', (event) => {
            const li = event.target.closest('li');
            if (li) {
                console.log('Clicked item ID:', li.dataset.id);
            }
        });

        // Dynamically added items work without new listeners
        const newItem = document.createElement('li');
        newItem.dataset.id = '4';
        newItem.textContent = 'Item 4';
        list.appendChild(newItem);
    </script>
</body>
</html>
```

**Expected Output**

Clicking any item (including the dynamically added one) logs its ID.

**Why This Output Occurs**

The listener is attached to the parent `<ul>`. When a child is clicked, the event bubbles up to the parent. `event.target.closest('li')` finds the clicked list item. This pattern works for dynamically added elements.

---

**Example 3: Form Submission with preventDefault**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Form Handler</title>
</head>
<body>
    <form id="myForm">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <button type="submit">Submit</button>
    </form>
    <p id="status"></p>

    <script>
        const form = document.getElementById('myForm');
        const status = document.getElementById('status');

        form.addEventListener('submit', (event) => {
            event.preventDefault(); // Prevent page reload
            const email = form.elements.email.value;
            status.textContent = `Submitted: ${email}`;
        });
    </script>
</body>
</html>
```

**Expected Output**

Submitting the form displays the email in the status paragraph without reloading the page.

**Why This Output Occurs**

`preventDefault()` cancels the default form submission. The email value is read from `form.elements.email.value` and displayed.

#### Real-World Cases

**Case 1: Single-Page Applications**

SPAs use event delegation and `preventDefault()` to handle navigation without page reloads.

**Case 2: Interactive Forms**

Forms use `input` and `change` events for real-time validation.

**Case 3: Keyboard Shortcuts**

Applications use `keydown` events to implement keyboard shortcuts.

---

### 4. Attribute Manipulation

#### Definitions

**Core Definition**

Attribute manipulation is the process of reading, modifying, or removing HTML attributes on DOM elements using methods like `getAttribute()`, `setAttribute()`, and the `dataset` property.

**Technical Definition**

The `Element` interface provides methods for attribute manipulation: `getAttribute(name)` returns the value of the specified attribute or `null`; `setAttribute(name, value)` sets or creates the attribute; `removeAttribute(name)` removes the attribute; `hasAttribute(name)` returns a boolean; and `toggleAttribute(name, force)` toggles the attribute. For `data-*` attributes, the `dataset` property provides a convenient `DOMStringMap` interface: `element.dataset.foo` reads/writes `data-foo`. Attribute names are case-insensitive in HTML (they are lowercased). The `attributes` property returns a live `NamedNodeMap` of all attributes. Properties like `element.id`, `element.className`, `element.href`, and `element.value` are reflected properties that map to attributes.

**Beginner-Friendly Explanation**

Elements have attributes — like `id`, `class`, `href`, `src`, `disabled`, and custom `data-*` attributes. You can read them with `getAttribute()`, change them with `setAttribute()`, and remove them with `removeAttribute()`. For custom `data-*` attributes, you can use the shorter `dataset` property. For common attributes like `id` and `className`, you can use properties directly.

#### Purposes

- To read attribute values from elements
- To modify attributes dynamically (e.g., `src`, `href`, `disabled`)
- To store custom data on elements using `data-*` attributes
- To toggle states (e.g., `disabled`, `hidden`, `aria-expanded`)
- To update accessibility attributes dynamically

#### Syntax Rules and Structure

**General Methods**

```javascript
// Get an attribute
const src = img.getAttribute('src');

// Set an attribute
img.setAttribute('alt', 'A description');

// Remove an attribute
img.removeAttribute('title');

// Check for an attribute
if (img.hasAttribute('loading')) { /* ... */ }

// Toggle an attribute
button.toggleAttribute('disabled');
```

**The `dataset` Property**

```html
<div id="user" data-user-id="123" data-role="admin"></div>
```

```javascript
const user = document.getElementById('user');

// Read data attributes
console.log(user.dataset.userId); // "123"
console.log(user.dataset.role);   // "admin"

// Set a data attribute
user.dataset.status = 'active';
// Results in: data-status="active"

// Remove a data attribute
delete user.dataset.role;
```

**Component Breakdown**

| Method/Property | Description |
|---|---|
| `getAttribute(name)` | Returns attribute value or `null` |
| `setAttribute(name, value)` | Sets or creates attribute |
| `removeAttribute(name)` | Removes attribute |
| `hasAttribute(name)` | Returns `true`/`false` |
| `toggleAttribute(name)` | Toggles boolean attribute |
| `dataset` | `DOMStringMap` for `data-*` attributes |
| `attributes` | Live `NamedNodeMap` of all attributes |

**Syntax Rules**

- Attribute names are case-insensitive in HTML
- `data-*` attributes map to `dataset` in camelCase (e.g., `data-user-id` → `dataset.userId`)
- `dataset` values are always strings
- `setAttribute` converts values to strings
- Some attributes have reflected properties (`id`, `className`, `value`, `href`)

**Constraints and Limitations**

- `dataset` does not work for non-`data-*` attributes
- CamelCase conversion in `dataset` can be confusing (e.g., `data-foo-bar` → `dataset.fooBar`)
- Setting `class` via `setAttribute` overwrites all classes; use `classList` instead
- `getAttribute` returns `null` for missing attributes (not `undefined`)

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Reading and Setting Attributes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Attribute Manipulation</title>
</head>
<body>
    <img id="photo" src="placeholder.jpg" alt="Placeholder">

    <script>
        const img = document.getElementById('photo');

        // Read attributes
        console.log(img.getAttribute('src'));  // "placeholder.jpg"
        console.log(img.getAttribute('alt'));  // "Placeholder"
        console.log(img.getAttribute('width')); // null (not set)

        // Set attributes
        img.setAttribute('src', 'real-photo.jpg');
        img.setAttribute('width', '400');
        img.setAttribute('height', '300');

        // Check attribute
        console.log(img.hasAttribute('width')); // true

        // Remove attribute
        img.removeAttribute('alt');
        console.log(img.hasAttribute('alt')); // false
    </script>
</body>
</html>
```

**Expected Output**

The image source changes to `real-photo.jpg` and it is resized to 400×300. The `alt` attribute is removed.

**Why This Output Occurs**

`setAttribute` changes the attribute value, which the browser reflects in the rendered output. `removeAttribute` deletes the attribute.

---

**Example 2: Using `dataset` for Custom Data**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Dataset Demo</title>
</head>
<body>
    <div id="product"
         data-product-id="42"
         data-price="29.99"
         data-in-stock="true">
        Product Card
    </div>

    <script>
        const product = document.getElementById('product');

        // Read data attributes
        console.log(product.dataset.productId); // "42"
        console.log(product.dataset.price);     // "29.99"
        console.log(product.dataset.inStock);   // "true"

        // Modify
        product.dataset.price = '24.99';
        product.dataset.inStock = 'false';

        // Add new
        product.dataset.discount = '10%';

        console.log(product.outerHTML);
        // <div id="product" data-product-id="42" data-price="24.99"
        //      data-in-stock="false" data-discount="10%">Product Card</div>
    </script>
</body>
</html>
```

**Expected Output (Console)**

```
42
29.99
true
<div id="product" data-product-id="42" data-price="24.99" data-in-stock="false" data-discount="10%">Product Card</div>
```

**Why This Output Occurs**

`dataset` provides camelCase access to `data-*` attributes. Setting `dataset.price` updates the `data-price` attribute. Adding `dataset.discount` creates a new `data-discount` attribute.

#### Real-World Cases

**Case 1: Image Galleries**

Galleries store image metadata in `data-*` attributes and update `src` on click.

**Case 2: Accessibility State**

Components use `setAttribute('aria-expanded', 'true')` to communicate state to screen readers.

**Case 3: Form State**

Forms use `setAttribute('disabled', '')` and `removeAttribute('disabled')` to enable/disable fields.

---

### 5. Content Manipulation

#### Definitions

**Core Definition**

Content manipulation is the process of reading or modifying the text and HTML content of DOM elements using properties like `textContent`, `innerText`, and `innerHTML`.

**Technical Definition**

The `Node` interface provides `textContent`, which returns the concatenated text content of the node and all its descendants, excluding comments and processing instructions. The `HTMLElement` interface adds `innerText`, which approximates the rendered text (respecting CSS visibility and layout). The `Element` interface provides `innerHTML`, which returns or sets the HTML markup inside the element; setting `innerHTML` parses the string as HTML and replaces the element's children. The `outerHTML` property includes the element itself. For safe insertion of DOM nodes, use `createElement()`, `append()`, and `insertAdjacentHTML()`. The `insertAdjacentText()` method inserts text without parsing. The `innerHTML` setter is a common XSS vector if used with untrusted input.

**Beginner-Friendly Explanation**

Content manipulation is how you change what's inside an element. Want to change a paragraph's text? Use `textContent` or `innerText`. Want to replace the HTML inside a div? Use `innerHTML`. But be careful — `innerHTML` parses whatever you give it as HTML, so if you insert user input, you could be opening yourself up to XSS attacks. For plain text, always use `textContent`.

#### Purposes

- To read the current text or HTML of an element
- To update text content dynamically
- To insert HTML markup dynamically (with caution)
- To clear an element's content
- To safely handle user-generated content

#### Syntax Rules and Structure

**Text Content**

```javascript
const el = document.getElementById('output');

// Read text
console.log(el.textContent); // All text, including hidden

// Set text (safe — does not parse HTML)
el.textContent = 'New text content';

// innerText (rendered text, respects CSS)
console.log(el.innerText);
```

**HTML Content**

```javascript
// Set HTML (parses the string)
el.innerHTML = '<strong>Bold</strong> text';

// Read HTML
console.log(el.innerHTML);

// Insert HTML at a position
el.insertAdjacentHTML('beforeend', '<p>New paragraph</p>');
```

**Comparison Table**

| Property | Parses HTML? | Respects CSS? | Performance | Safety |
|---|---|---|---|---|
| `textContent` | No | No (includes hidden) | Fast | Safe |
| `innerText` | No | Yes (excludes hidden) | Slower (reflow) | Safe |
| `innerHTML` | Yes | N/A | Fast | XSS risk |
| `outerHTML` | Yes | N/A | Fast | XSS risk |

**insertAdjacentHTML Positions**

| Position | Description |
|---|---|
| `'beforebegin'` | Before the element itself |
| `'afterbegin'` | Inside, before first child |
| `'beforeend'` | Inside, after last child |
| `'afterend'` | After the element itself |

**Syntax Rules**

- `textContent` gets/sets the text of the node and descendants
- `innerText` is aware of CSS (hidden elements are excluded)
- `innerHTML` parses the string as HTML
- Setting `innerHTML` removes existing children and event listeners
- `insertAdjacentHTML` does not re-parse the element itself
- Never insert untrusted input with `innerHTML`

**Constraints and Limitations**

- `innerHTML` is a common XSS vector
- Setting `innerHTML` destroys existing event listeners on children
- `innerText` triggers a reflow (performance cost)
- `textContent` includes hidden text; `innerText` does not
- For user-generated content, use `textContent` or sanitise

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Safe Text Manipulation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Text Content</title>
</head>
<body>
    <div id="output"></div>
    <input id="userInput" type="text" placeholder="Enter text">
    <button id="btn">Update</button>

    <script>
        const output = document.getElementById('output');
        const input = document.getElementById('userInput');
        const btn = document.getElementById('btn');

        btn.addEventListener('click', () => {
            // SAFE: textContent does not parse HTML
            output.textContent = input.value;
        });
    </script>
</body>
</html>
```

**Expected Output**

Entering `<script>alert('XSS')</script>` displays the literal text, not an alert.

**Why This Output Occurs**

`textContent` treats the input as plain text, not HTML. The angle brackets are displayed as text rather than parsed as a script tag.

---

**Example 2: Using `innerHTML` for Markup**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>innerHTML Demo</title>
</head>
<body>
    <div id="container">Original content.</div>

    <script>
        const container = document.getElementById('container');

        // Replace with HTML markup
        container.innerHTML = `
            <h2>New Heading</h2>
            <p>This is <strong>bold</strong> and <em>italic</em>.</p>
        `;

        // Insert more HTML at the end
        container.insertAdjacentHTML('beforeend',
            '<p>Appended paragraph.</p>');
    </script>
</body>
</html>
```

**Expected Output**

The container now contains a heading, a formatted paragraph, and an appended paragraph.

**Why This Output Occurs**

`innerHTML` parses the string as HTML and replaces the element's children. `insertAdjacentHTML` adds HTML without destroying existing content or listeners.

---

**Example 3: Clearing and Replacing Content**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Content Replacement</title>
</head>
<body>
    <ul id="list">
        <li>Item 1</li>
        <li>Item 2</li>
    </ul>
    <button id="clear">Clear</button>
    <button id="replace">Replace</button>

    <script>
        const list = document.getElementById('list');

        document.getElementById('clear').addEventListener('click', () => {
            list.textContent = ''; // Fastest way to clear
        });

        document.getElementById('replace').addEventListener('click', () => {
            list.innerHTML = '<li>Replaced item</li>';
        });
    </script>
</body>
</html>
```

**Expected Output**

Clicking "Clear" empties the list. Clicking "Replace" replaces the list content with a single item.

**Why This Output Occurs**

`textContent = ''` removes all children quickly. `innerHTML` replaces the content by parsing the new HTML.

#### Real-World Cases

**Case 1: Live Search Results**

Search interfaces use `innerHTML` or `textContent` to update results as the user types.

**Case 2: Chat Applications**

Chat apps append new messages using `insertAdjacentHTML` or `createElement`.

**Case 3: Form Validation Messages**

Forms set error messages with `textContent` for safety.

---

### 6. Choosing the Right DOM Approach

#### Definitions

**Core Definition**

Choosing the right DOM approach means selecting the appropriate method, property, or pattern based on the task, the browser support requirements, and the security and performance implications.

**Technical Definition**

The choice depends on the task: element selection (`querySelector`/`querySelectorAll` for CSS selector power; `getElementById` for speed), event handling (`addEventListener` for multiple listeners and options; event delegation for dynamic content), attribute manipulation (`dataset` for `data-*`; `setAttribute` for others), and content manipulation (`textContent` for plain text; `innerHTML` for trusted HTML; `insertAdjacentHTML` for insertion without destroying listeners). Modern code prefers `querySelector`/`querySelectorAll` and `addEventListener` over legacy alternatives.

#### Decision Guide

| Task | Recommended | Avoid |
|---|---|---|
| Select by ID | `getElementById()` | `querySelector('#id')` (slower) |
| Select by CSS selector | `querySelector()` / `querySelectorAll()` | `getElementsByClassName()` (less flexible) |
| Attach event | `addEventListener()` | `onclick` attribute |
| Handle dynamic elements | Event delegation | Individual listeners |
| Read/write `data-*` | `dataset` | `getAttribute('data-...')` |
| Set text | `textContent` | `innerHTML` (XSS risk) |
| Insert HTML | `insertAdjacentHTML()` | `innerHTML +=` (destroys listeners) |
| Clear content | `textContent = ''` | `innerHTML = ''` (slower) |

---

## References

- MDN Web Docs – Document Object Model (DOM) – https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model
- MDN Web Docs – Introduction to the DOM – https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction
- MDN Web Docs – `Document.querySelector()` – https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector
- MDN Web Docs – `Document.querySelectorAll()` – https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll
- MDN Web Docs – `Document.getElementById()` – https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById
- MDN Web Docs – `EventTarget.addEventListener()` – https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
- MDN Web Docs – Event reference – https://developer.mozilla.org/en-US/docs/Web/Events
- MDN Web Docs – `Element.getAttribute()` – https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute
- MDN Web Docs – `Element.setAttribute()` – https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute
- MDN Web Docs – `HTMLElement.dataset` – https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset
- MDN Web Docs – `Node.textContent` – https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent
- MDN Web Docs – `HTMLElement.innerText` – https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/innerText
- MDN Web Docs – `Element.innerHTML` – https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML
- MDN Web Docs – `Element.insertAdjacentHTML()` – https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML
- WHATWG – DOM Living Standard – https://dom.spec.whatwg.org/
- WHATWG – HTML Living Standard – https://html.spec.whatwg.org/multipage/
- W3C – UI Events – https://www.w3.org/TR/uievents/
- web.dev – DOM manipulation – https://web.dev/learn/javascript/dom