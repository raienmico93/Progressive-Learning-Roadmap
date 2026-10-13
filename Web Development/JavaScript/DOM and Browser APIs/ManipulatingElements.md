# Manipulating Elements — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** DOM manipulation is the process of using JavaScript to create, modify, move, replace, or delete elements and their attributes, content, and styles within a web document.

**Technical Definition:** The DOM (Document Object Model) provides a set of interfaces and methods for programmatically altering the structure, content, and presentation of an HTML or XML document. Element manipulation encompasses node creation (`createElement`, `createTextNode`), node insertion (`appendChild`, `insertBefore`), node removal (`removeChild`), node replacement (`replaceChild`), content modification (`innerHTML`, `textContent`), attribute management (`setAttribute`, `classList`, `dataset`), style manipulation (`style`, `getComputedStyle`), and dimension measurement (`offsetWidth`, `getBoundingClientRect`). These APIs are defined across the DOM Living Standard and the HTML Living Standard.

**Beginner-Friendly Explanation:** DOM manipulation is how you change a web page after it has loaded. Instead of just displaying static HTML, you can use JavaScript to add new elements, remove old ones, change text, update styles, and much more. It's what makes web pages interactive — when you click a button and a new message appears, that's DOM manipulation in action.

### Key Characteristics

- **Two approaches:** Traditional node methods (`createElement` + `appendChild`) and modern convenience methods (`append`, `prepend`, `before`, `after`).
- **Content properties:** `innerHTML` (HTML parsing, XSS risk), `textContent` (raw text, performant, safe), `innerText` (rendered text, triggers reflow).
- **Attribute vs. property:** Attributes are the HTML-level values; properties are the JavaScript object values.
- **Live vs. static:** Some collections are live (`HTMLCollection`), others are static (`NodeList`).
- **Performance:** Batch DOM updates, cache selections, and minimise reflows.

### Prerequisites

- Understanding of the DOM tree, nodes, and elements.
- Familiarity with selecting elements (`querySelector`, `getElementById`).
- Basic knowledge of HTML attributes and CSS classes.

### Related Programming Areas

- **Event handling:** Manipulating elements in response to user actions.
- **Animation:** Updating styles and positions over time.
- **Form processing:** Reading and setting form values.
- **Single-page applications:** Dynamically rendering and updating UI.

### Core Concepts / Features

---

## Node Creation

### `createElement()`

**Core Definition:** `document.createElement()` creates a new element node with the specified tag name.

**Technical Definition:** The `createElement()` method of the `Document` interface creates the HTML element specified by `tagName`, or an `HTMLUnknownElement` if `tagName` isn't recognised. It returns a new `Element` object that is not yet attached to the document tree.

**Purposes:**
- To create new elements programmatically before inserting them into the DOM.
- To build complex element structures dynamically.
- To clone templates or generate repetitive content.

**Syntax Rules and Structure:**
```js
const element = document.createElement(tagName);
```
- `tagName`: A string specifying the type of element to create (e.g., `"div"`, `"p"`, `"button"`).

**Constraints and Limitations:**
- The element is not attached to the DOM until inserted.
- In HTML documents, `tagName` is lowercased.

**Annotated Code Example:**
```js
// Step 1: Create a new div element.
const newDiv = document.createElement("div");

// Step 2: Configure it.
newDiv.id = "myDiv";
newDiv.textContent = "Hello, world!";

// Step 3: It exists but is not yet in the DOM.
console.log(newDiv.tagName); // "DIV"
console.log(document.body.contains(newDiv)); // false
```
**Expected Output:**
```
DIV
false
```

### `createTextNode()`

**Core Definition:** `document.createTextNode()` creates a new text node with the specified text content.

**Technical Definition:** The `createTextNode()` method creates a new `Text` node. Text nodes are the leaf nodes of the DOM tree that contain the actual text content of elements.

**Purposes:**
- To create text content programmatically.
- To append text to elements without using `innerHTML`.

**Syntax Rules and Structure:**
```js
const textNode = document.createTextNode(data);
```
- `data`: A string containing the text content.

**Annotated Code Example:**
```js
const p = document.createElement("p");
const text = document.createTextNode("This is a text node.");
p.appendChild(text);
document.body.appendChild(p);
```
**Expected Output:** A `<p>` element with the text "This is a text node." is added to the page.

### `cloneNode()`

**Core Definition:** `cloneNode()` creates a duplicate of a node. It takes a boolean parameter indicating whether to perform a deep clone (including descendants) or a shallow clone (excluding descendants).

**Technical Definition:** The `cloneNode()` method returns a copy of the node on which it was called. A deep clone (`true`) copies the node and all its descendants; a shallow clone (`false`) copies only the node itself.

**Purposes:**
- To duplicate elements for repeated content.
- To create templates that can be modified before insertion.

**Syntax Rules and Structure:**
```js
const clone = originalNode.cloneNode(deep);
```
- `deep`: `true` for a deep clone, `false` for shallow.

**Annotated Code Example:**
```js
const original = document.getElementById("template");
const deepClone = original.cloneNode(true);
const shallowClone = original.cloneNode(false);

console.log(deepClone.children.length);   // > 0
console.log(shallowClone.children.length); // 0
```
**Expected Output:**
```
2
0
```

---

## Node Insertion

### `appendChild()`

**Core Definition:** `appendChild()` adds a node as the last child of a specified parent node.

**Technical Definition:** The `appendChild()` method adds a node to the end of the list of children of a specified parent node. If the node already exists in the document, it is moved from its current position to the new position.

**Purposes:**
- To add new elements to the DOM.
- To move existing elements to a new parent.

**Syntax Rules and Structure:**
```js
parentNode.appendChild(childNode);
```

**Annotated Code Example:**
```js
const ul = document.getElementById("myList");
const li = document.createElement("li");
li.textContent = "New item";
ul.appendChild(li);
```
**Expected Output:** A new `<li>` with "New item" is appended to the `<ul>`.

### `insertBefore()`

**Core Definition:** `insertBefore()` inserts a node before a reference node as a child of a specified parent.

**Technical Definition:** The `insertBefore()` method inserts a node before a reference node. If the reference node is `null`, the new node is inserted at the end (equivalent to `appendChild`).

**Purposes:**
- To insert elements at a specific position within a parent.
- To maintain order when adding items.

**Syntax Rules and Structure:**
```js
parentNode.insertBefore(newNode, referenceNode);
```

**Annotated Code Example:**
```js
const list = document.getElementById("myList");
const newItem = document.createElement("li");
newItem.textContent = "First item";
const firstItem = list.firstChild;
list.insertBefore(newItem, firstItem);
```
**Expected Output:** The new `<li>` is inserted before the first existing child.

---

## Modern Insertion Methods

### `append()`

**Core Definition:** `append()` inserts one or more nodes or strings at the end of a parent node's children.

**Technical Definition:** The `append()` method of the `ParentNode` interface inserts a set of `Node` or `DOMString` objects after the last child of the `ParentNode`. String objects are inserted as equivalent `Text` nodes.

**Purposes:**
- To append multiple nodes and strings in one call.
- To simplify code that would otherwise require multiple `appendChild` calls.

**Syntax Rules and Structure:**
```js
parentNode.append(...nodesOrStrings);
```

**Annotated Code Example:**
```js
const div = document.querySelector("div");
div.append("Text", document.createElement("span"), "More text");
// Adds: text node "Text", a <span>, and text node "More text"
```

### `prepend()`

**Core Definition:** `prepend()` inserts one or more nodes or strings at the beginning of a parent node's children.

**Syntax Rules and Structure:**
```js
parentNode.prepend(...nodesOrStrings);
```

**Annotated Code Example:**
```js
const list = document.querySelector("ul");
list.prepend("First");
// Adds a text node "First" before the first child
```

### `before()`

**Core Definition:** `before()` inserts nodes or strings immediately before the element itself (as a sibling).

**Technical Definition:** The `before()` method of the `ChildNode` interface inserts a set of `Node` or `DOMString` objects in the children list of this `ChildNode`'s parent, just before this `ChildNode`.

**Syntax Rules and Structure:**
```js
element.before(...nodesOrStrings);
```

**Annotated Code Example:**
```js
const p = document.querySelector("p");
p.before("Text before paragraph");
// Inserts a text node before the <p>
```

### `after()`

**Core Definition:** `after()` inserts nodes or strings immediately after the element itself (as a sibling).

**Syntax Rules and Structure:**
```js
element.after(...nodesOrStrings);
```

**Annotated Code Example:**
```js
const p = document.querySelector("p");
p.after("Text after paragraph");
// Inserts a text node after the <p>
```

### `replaceWith()`

**Core Definition:** `replaceWith()` replaces the element with a set of nodes or strings.

**Technical Definition:** The `replaceWith()` method of the `ChildNode` interface replaces this `ChildNode` in the children list of its parent with a set of `Node` or `DOMString` objects.

**Syntax Rules and Structure:**
```js
element.replaceWith(...nodesOrStrings);
```

**Annotated Code Example:**
```js
const old = document.getElementById("old");
const newEl = document.createElement("div");
newEl.textContent = "New content";
old.replaceWith(newEl);
```
**Expected Output:** The old element is replaced by the new `<div>`.

### `remove()`

**Core Definition:** `remove()` removes the element from its parent in the DOM tree.

**Technical Definition:** The `remove()` method of the `ChildNode` interface removes this `ChildNode` from the children list of its parent.

**Syntax Rules and Structure:**
```js
element.remove();
```

**Annotated Code Example:**
```js
const element = document.getElementById("toRemove");
element.remove();
```
**Expected Output:** The element is removed from the document.

### `insertAdjacentElement()`, `insertAdjacentHTML()`, `insertAdjacentText()`

**Core Definition:** These methods insert an element, HTML string, or text string at a specified position relative to an element.

**Technical Definition:** The `insertAdjacentElement()` method inserts a given element node at a given position relative to the element it is invoked upon. The `insertAdjacentHTML()` parses the specified text as HTML or XML and inserts the resulting nodes into the DOM tree at a specified position. The `insertAdjacentText()` inserts a given text node at a given position.

**Positions:**
- `'beforebegin'`: Before the element itself.
- `'afterbegin'`: Just inside the element, before its first child.
- `'beforeend'`: Just inside the element, after its last child.
- `'afterend'`: After the element itself.

**Syntax Rules and Structure:**
```js
element.insertAdjacentElement(position, element);
element.insertAdjacentHTML(position, text);
element.insertAdjacentText(position, text);
```

**Annotated Code Example:**
```js
const target = document.querySelector(".target");
target.insertAdjacentHTML("beforeend", "<p>New paragraph</p>");
target.insertAdjacentText("afterbegin", "Text at start");
```
**Expected Output:** A `<p>` is added after the last child, and text is added before the first child.

---

## Content Manipulation

### `innerHTML`

**Core Definition:** `innerHTML` gets or sets the HTML markup contained within an element.

**Technical Definition:** The `innerHTML` property of the `Element` interface gets or sets the HTML or XML markup contained within the element. Setting `innerHTML` parses the string as HTML and replaces the element's children with the resulting nodes.

**Purposes:**
- To quickly set or retrieve HTML content.
- To replace entire subtrees of HTML.

**Constraints and Limitations:**
- **XSS risk:** Never use `innerHTML` with untrusted input. It can execute malicious scripts.
- **Performance:** Parsing HTML is slower than setting text.
- **Existing content:** Setting `innerHTML` destroys existing children and their event listeners.

**Annotated Code Example:**
```js
const div = document.getElementById("content");
div.innerHTML = "<p>New <strong>HTML</strong> content</p>";
console.log(div.innerHTML); // "<p>New <strong>HTML</strong> content</p>"
```
**Expected Output:**
```
<p>New <strong>HTML</strong> content</p>
```

### `outerHTML`

**Core Definition:** `outerHTML` gets or sets the HTML markup of the element itself, including its start and end tags.

**Technical Definition:** The `outerHTML` property gets the serialised HTML fragment describing the element including its descendants. Setting `outerHTML` replaces the element with the parsed nodes from the given string.

**Annotated Code Example:**
```js
const p = document.querySelector("p");
console.log(p.outerHTML); // "<p>Hello</p>"
p.outerHTML = "<div>Replaced</div>";
// The <p> is replaced by a <div>
```

### `textContent`

**Core Definition:** `textContent` gets or sets the text content of an element, including all descendants.

**Technical Definition:** The `textContent` property of the `Node` interface represents the text content of the node and its descendants. Setting `textContent` replaces all children with a single text node containing the given string.

**Purposes:**
- To read or write text without parsing HTML.
- To safely set text content (no XSS risk).
- To efficiently update text without triggering reflow.

**Constraints and Limitations:**
- `textContent` includes the text of `<script>` and `<style>` elements.
- It does not trigger reflow (unlike `innerText`).

**Annotated Code Example:**
```js
const p = document.querySelector("p");
console.log(p.textContent); // "Hello, world!"
p.textContent = "Updated text";
console.log(p.textContent); // "Updated text"
```

### `innerText`

**Core Definition:** `innerText` gets or sets the rendered text content of an element, respecting CSS styling.

**Technical Definition:** The `innerText` property is aware of CSS styling and will not return the text of hidden elements. It triggers a reflow to compute the rendered text.

**Differences from `textContent`:**
- `innerText` is aware of style and will not return hidden text.
- `innerText` triggers a reflow; `textContent` does not.
- `innerText` does not include text from `<script>` and `<style>` elements.

**Annotated Code Example:**
```js
// HTML: <p>Visible <span style="display:none;">Hidden</span> Text</p>
const p = document.querySelector("p");
console.log(p.innerText);    // "Visible  Text"
console.log(p.textContent);  // "Visible Hidden Text"
```
**Expected Output:**
```
Visible  Text
Visible Hidden Text
```

### `value`

**Core Definition:** `value` gets or sets the value of form elements such as `<input>`, `<textarea>`, and `<select>`.

**Technical Definition:** The `value` property of form elements represents the current value of the control. For `<input>` and `<textarea>`, it is the text entered by the user. For `<select>`, it is the value of the selected option.

**Annotated Code Example:**
```js
const input = document.querySelector("input");
console.log(input.value); // Current value
input.value = "New value"; // Sets the value
```

---

## Element Attributes

### `getAttribute()`

**Core Definition:** `getAttribute()` returns the value of a specified attribute on the element.

**Syntax Rules and Structure:**
```js
const value = element.getAttribute(attributeName);
```
- Returns `null` if the attribute does not exist.

**Annotated Code Example:**
```js
const link = document.querySelector("a");
console.log(link.getAttribute("href")); // "https://example.com"
```

### `setAttribute()`

**Core Definition:** `setAttribute()` sets the value of an attribute on the element. If the attribute already exists, its value is updated.

**Technical Definition:** The `setAttribute()` method sets the value of an attribute on the specified element. If the attribute already exists, the value is updated; otherwise, a new attribute is added with the specified name and value.

**Syntax Rules and Structure:**
```js
element.setAttribute(name, value);
```
- The value is always converted to a string.

**Annotated Code Example:**
```js
const button = document.querySelector("button");
button.setAttribute("disabled", "true");
console.log(button.getAttribute("disabled")); // "true"
```

### `removeAttribute()`

**Core Definition:** `removeAttribute()` removes an attribute from the element.

**Syntax Rules and Structure:**
```js
element.removeAttribute(attributeName);
```

**Annotated Code Example:**
```js
const div = document.getElementById("myDiv");
div.removeAttribute("align");
```

### `hasAttribute()`

**Core Definition:** `hasAttribute()` returns a boolean indicating whether the element has the specified attribute.

**Syntax Rules and Structure:**
```js
const has = element.hasAttribute(attributeName);
```

**Annotated Code Example:**
```js
const button = document.querySelector("button");
console.log(button.hasAttribute("disabled")); // true or false
```

### `dataset`

**Core Definition:** `dataset` provides access to `data-*` custom attributes on an element, converting kebab-case attribute names to camelCase property names.

**Technical Definition:** The `dataset` property of the `HTMLElement` interface provides a read/write interface to custom data attributes (`data-*`). The naming convention converts between kebab-case in HTML and camelCase in JavaScript: `data-order-status` becomes `dataset.orderStatus`.

**Syntax Rules and Structure:**
```js
// Reading:
const value = element.dataset.keyName;

// Writing:
element.dataset.keyName = "value";

// Deleting:
delete element.dataset.keyName;
```

**Annotated Code Example:**
```js
// HTML: <div id="user" data-user-id="123" data-role="admin"></div>
const user = document.getElementById("user");
console.log(user.dataset.userId); // "123"
console.log(user.dataset.role);   // "admin"

user.dataset.status = "active";
console.log(user.getAttribute("data-status")); // "active"
```
**Expected Output:**
```
123
admin
active
```

---

## Element Classes

### `classList`

**Core Definition:** `classList` returns a live `DOMTokenList` collection representing the element's class attribute, providing methods for adding, removing, toggling, and checking classes.

**Technical Definition:** The `classList` property of the `Element` interface returns a live `DOMTokenList` collection of the class attributes of the element. This can then be used to manipulate the class list. Using `classList` is a convenient alternative to accessing an element's list of classes as a space-delimited string via `element.className`.

**Methods:**

| Method | Description |
|--------|-------------|
| `add(class1, class2, ...)` | Adds one or more classes |
| `remove(class1, class2, ...)` | Removes one or more classes |
| `toggle(class, force)` | Toggles a class; if `force` is `true`, adds; if `false`, removes |
| `contains(class)` | Returns `true` if the class exists |
| `replace(oldClass, newClass)` | Replaces one class with another |

**Annotated Code Example:**
```js
const div = document.createElement("div");
div.classList = "foo";

// Remove and add:
div.classList.remove("foo");
div.classList.add("another-class");

// Toggle:
div.classList.toggle("visible"); // Adds if absent, removes if present

// Contains:
console.log(div.classList.contains("foo")); // false

// Add multiple:
div.classList.add("foo", "bar", "baz");

// Replace:
div.classList.replace("foo", "bar");
```
**Expected Output:**
```
false
```

---

## Element Styles

### `style`

**Core Definition:** `element.style` returns a `CSSStyleDeclaration` object representing the inline styles of the element.

**Technical Definition:** The `style` property of the `HTMLElement` interface returns a `CSSStyleDeclaration` object that represents the element's `style` attribute. It can be used to read and write inline styles.

**Purposes:**
- To set inline styles directly on an element.
- To read inline styles that were set via JavaScript or the `style` attribute.

**Syntax Rules and Structure:**
```js
// Setting a style:
element.style.property = "value";

// Reading a style:
const value = element.style.property;
```

**Constraints and Limitations:**
- Only inline styles are accessible; styles from `<style>` tags or external stylesheets are not.
- Property names use camelCase (e.g., `backgroundColor`, not `background-color`).

**Annotated Code Example:**
```js
const div = document.querySelector("div");
div.style.color = "blue";
div.style.backgroundColor = "yellow";
div.style.fontSize = "16px";

console.log(div.style.color); // "blue"
```

### `getComputedStyle()`

**Core Definition:** `window.getComputedStyle()` returns a read-only `CSSStyleDeclaration` object representing the final computed styles applied to an element.

**Technical Definition:** The `window.getComputedStyle()` method returns an object containing the values of all CSS properties of an element, after applying active stylesheets and resolving any basic computation those values may contain.

**Purposes:**
- To read the final, rendered styles of an element (including styles from stylesheets).
- To inspect computed values like resolved sizes and colours.

**Syntax Rules and Structure:**
```js
const style = window.getComputedStyle(element);
const value = style.getPropertyValue(propertyName);
```

**Constraints and Limitations:**
- The returned object is read-only.
- Triggers a reflow (performance cost).

**Annotated Code Example:**
```js
const div = document.querySelector("div");
const computed = window.getComputedStyle(div);
console.log(computed.color);           // e.g., "rgb(0, 0, 255)"
console.log(computed.getPropertyValue("background-color")); // e.g., "rgb(255, 255, 0)"
```

### `setProperty()` / `getPropertyValue()`

**Core Definition:** These methods of `CSSStyleDeclaration` set and get CSS property values, including custom properties (CSS variables).

**Syntax Rules and Structure:**
```js
element.style.setProperty(propertyName, value, priority);
const value = element.style.getPropertyValue(propertyName);
```

**Annotated Code Example:**
```js
const div = document.querySelector("div");
div.style.setProperty("--my-var", "10px");
console.log(div.style.getPropertyValue("--my-var")); // "10px"
```

---

## Element Dimensions

### `offsetWidth` / `offsetHeight`

**Core Definition:** `offsetWidth` and `offsetHeight` return the layout width and height of an element, including padding, borders, and scrollbars.

**Technical Definition:** The `offsetWidth` and `offsetHeight` properties return the element's layout width and height, including its border, padding, and vertical scrollbar (if present). Values are rounded to integers.

**When to Use:** When you need to know how much space an element occupies on screen, including borders.

**Annotated Code Example:**
```js
const div = document.querySelector("div");
console.log(div.offsetWidth);  // e.g., 200
console.log(div.offsetHeight); // e.g., 150
```

### `clientWidth` / `clientHeight`

**Core Definition:** `clientWidth` and `clientHeight` return the inner width and height of an element, including padding but excluding borders and scrollbars.

**When to Use:** When you need to know the size of the content area plus padding, without borders.

**Annotated Code Example:**
```js
const div = document.querySelector("div");
console.log(div.clientWidth);  // Content width + padding
console.log(div.clientHeight); // Content height + padding
```

### `scrollWidth` / `scrollHeight`

**Core Definition:** `scrollWidth` and `scrollHeight` return the total width and height of an element's content, including content that is not currently visible due to overflow.

**When to Use:** When you need to know the full content size, regardless of the visible viewport.

**Annotated Code Example:**
```js
const scrollable = document.querySelector(".scroll-container");
console.log(scrollable.scrollWidth);  // Full content width
console.log(scrollable.scrollHeight); // Full content height
```

### `getBoundingClientRect()`

**Core Definition:** `getBoundingClientRect()` returns a `DOMRect` object containing the size and position of an element relative to the viewport.

**Technical Definition:** The `getBoundingClientRect()` method returns a `DOMRect` object providing information about the size of an element and its position relative to the viewport. It provides more precise (fractional) values than `offsetWidth`/`offsetHeight`.

**Differences from `offsetWidth`/`offsetHeight`:**
- `getBoundingClientRect()` returns rendering dimensions (after CSS transforms); `offsetWidth`/`offsetHeight` return layout dimensions.
- `getBoundingClientRect()` returns fractional values; `offsetWidth`/`offsetHeight` are rounded to integers.

**Annotated Code Example:**
```js
const rect = element.getBoundingClientRect();
console.log(rect.width);    // e.g., 200.5
console.log(rect.height);   // e.g., 150.2
console.log(rect.top);      // Distance from viewport top
console.log(rect.left);     // Distance from viewport left
console.log(rect.bottom);   // Distance from viewport top to bottom edge
console.log(rect.right);    // Distance from viewport left to right edge
```

---

## References

- MDN Web Docs – Document: createElement() — https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement
- MDN Web Docs – Document: createTextNode() — https://developer.mozilla.org/en-US/docs/Web/API/Document/createTextNode
- MDN Web Docs – Node: appendChild() — https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild
- MDN Web Docs – Node: insertBefore() — https://developer.mozilla.org/en-US/docs/Web/API/Node/insertBefore
- MDN Web Docs – Node: removeChild() — https://developer.mozilla.org/en-US/docs/Web/API/Node/removeChild
- MDN Web Docs – Node: replaceChild() — https://developer.mozilla.org/en-US/docs/Web/API/Node/replaceChild
- MDN Web Docs – Node: cloneNode() — https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode
- MDN Web Docs – Element: append() — https://developer.mozilla.org/en-US/docs/Web/API/Element/append
- MDN Web Docs – Element: prepend() — https://developer.mozilla.org/en-US/docs/Web/API/Element/prepend
- MDN Web Docs – Element: before() — https://developer.mozilla.org/en-US/docs/Web/API/Element/before
- MDN Web Docs – Element: after() — https://developer.mozilla.org/en-US/docs/Web/API/Element/after
- MDN Web Docs – Element: replaceWith() — https://developer.mozilla.org/en-US/docs/Web/API/Element/replaceWith
- MDN Web Docs – Element: remove() — https://developer.mozilla.org/en-US/docs/Web/API/Element/remove
- MDN Web Docs – Element: insertAdjacentElement() — https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement
- MDN Web Docs – Element: insertAdjacentHTML() — https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML
- MDN Web Docs – Element: insertAdjacentText() — https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentText
- MDN Web Docs – Element: innerHTML — https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML
- MDN Web Docs – Element: outerHTML — https://developer.mozilla.org/en-US/docs/Web/API/Element/outerHTML
- MDN Web Docs – Node: textContent — https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent
- MDN Web Docs – HTMLElement: innerText — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/innerText
- MDN Web Docs – HTMLInputElement: value — https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/value
- MDN Web Docs – Element: getAttribute() — https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute
- MDN Web Docs – Element: setAttribute() — https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute
- MDN Web Docs – Element: removeAttribute() — https://developer.mozilla.org/en-US/docs/Web/API/Element/removeAttribute
- MDN Web Docs – Element: hasAttribute() — https://developer.mozilla.org/en-US/docs/Web/API/Element/hasAttribute
- MDN Web Docs – HTMLElement: dataset — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset
- MDN Web Docs – Element: classList — https://developer.mozilla.org/en-US/docs/Web/API/Element/classList
- MDN Web Docs – HTMLElement: style — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style
- MDN Web Docs – Window: getComputedStyle() — https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle
- MDN Web Docs – CSSStyleDeclaration: setProperty() — https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty
- MDN Web Docs – CSSStyleDeclaration: getPropertyValue() — https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/getPropertyValue
- MDN Web Docs – Determining the dimensions of elements — https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model/Determining_the_dimensions_of_elements
- MDN Web Docs – Element: getBoundingClientRect() — https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect
- WHATWG DOM Living Standard — https://dom.spec.whatwg.org/
- WHATWG HTML Living Standard — https://html.spec.whatwg.org/