# DOM Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Document Object Model (DOM) is a programming interface for web documents. It represents the page so that programs can change the document structure, style, and content.

**Technical Definition:** The DOM is the data representation of the objects that comprise the structure and content of a document on the web. It is a programming interface for HTML and XML documents that represents the document as a tree of nodes, where each node is an object representing a part of the document (such as an element, text, or comment). The DOM connects web pages to scripts or programming languages by representing the structure of a document in memory. The DOM is not a programming language and is not part of JavaScript itself; it is a Web API that can be implemented in any language.

**Beginner-Friendly Explanation:** The DOM is how JavaScript sees and interacts with a web page. When a browser loads an HTML page, it creates a "map" of the page in memory — a tree structure where every HTML tag, every piece of text, and every comment becomes a node. JavaScript can read this map, change it, add to it, or remove parts of it. This is what allows web pages to be interactive: when you click a button and something changes on the page, JavaScript is manipulating the DOM.

### Key Characteristics

- **Tree structure:** The DOM represents a document as a hierarchical tree of nodes.
- **Language-independent:** The DOM is designed to be independent of any programming language.
- **Live representation:** The DOM is a live, in-memory representation of the document; changes to the DOM are reflected immediately in the browser.
- **Node-based:** Everything in the DOM is a node — elements, text, comments, and the document itself.
- **Interface hierarchy:** DOM nodes are based on the abstract `Node` interface, with specialised subclasses like `Document`, `Element`, `Text`, and `Comment`.
- **Standardised:** The DOM is standardised by WHATWG (the DOM Living Standard) and W3C.

### Prerequisites

- Basic understanding of HTML structure (tags, attributes, nesting).
- Familiarity with JavaScript variables, functions, and objects.
- Awareness of how a browser loads and renders a web page.

### Related Programming Areas

- **Web development:** The DOM is the foundation of all client-side web interactivity.
- **JavaScript frameworks:** React, Vue, and Angular all interact with the DOM (often through a virtual DOM abstraction).
- **Event handling:** DOM events (clicks, keypresses) are dispatched through the DOM tree.
- **CSSOM:** The CSS Object Model is a companion to the DOM for styling.
- **Shadow DOM:** A technique for encapsulating DOM subtrees in Web Components.

### Core Concepts / Features

---

## DOM

### Definitions

**Core Definition:** The DOM is a programming interface that represents an HTML or XML document as a tree of objects, allowing programs to read and modify the document's structure, style, and content.

**Technical Definition:** The Document Object Model (DOM) is a programming interface for web documents. It represents the page so that programs can change the document structure, style, and content. The DOM represents the document as nodes and objects; that way, programming languages can interact with the page. It is built using multiple APIs that work together: the core DOM defines the entities describing any document and the objects within it, while other APIs (like the HTML DOM API) extend it as needed.

**Beginner-Friendly Explanation:** The DOM is the bridge between your JavaScript code and the HTML on the page. Without the DOM, JavaScript would have no way to know what's on the page or to change it. The DOM turns your HTML into a structure that JavaScript can navigate and manipulate.

### Purposes

- To provide a programming interface for manipulating web documents.
- To represent the document as a tree of nodes that can be traversed and modified.
- To enable dynamic, interactive web pages by allowing scripts to change structure, style, and content.
- To provide a consistent API across different programming languages and platforms.
- To connect web pages to scripts and programming languages.

### Syntax Rules and Structure

The DOM is accessed through the `document` object, which is a property of the `window` object.

```js
// Accessing the DOM:
const paragraphs = document.querySelectorAll("p");
// paragraphs[0] is the first <p> element
// paragraphs[1] is the second <p> element, etc.
alert(paragraphs[0].nodeName);
```

**Component Breakdown:**
- `document`: The entry point to the DOM, representing the entire document.
- `querySelectorAll("p")`: A method that returns all `<p>` elements in the document.
- `paragraphs[0]`: The first element in the returned NodeList.

**Syntax Rules:**
- The `document` object is always available as a global variable in browser environments.
- The DOM can be accessed and manipulated using JavaScript or any other language with a DOM implementation.
- All DOM properties and methods are organised into objects (e.g., `document`, `HTMLTableElement`, etc.).

**Constraints and Limitations:**
- The DOM is not part of the JavaScript language; it is a separate Web API.
- In Node.js, the DOM is not available (Node.js has a different set of APIs).
- Different browsers may have slightly different DOM implementations, though modern browsers are largely standards-compliant.

### Annotated Code Examples

#### Example 1: Accessing and Modifying the DOM

```js
// Step 1: Select an element.
const heading = document.querySelector("h1");
console.log(heading.textContent); // "Welcome"

// Step 2: Modify its content.
heading.textContent = "Hello, DOM!";
console.log(heading.textContent); // "Hello, DOM!"

// Step 3: Change its style.
heading.style.color = "blue";
```

**Expected Output:**
```
Welcome
Hello, DOM!
```

**Why This Output:** `querySelector("h1")` finds the first `<h1>` element. The `textContent` property is read, then changed. The visual change (blue colour) is visible on the page.

#### Example 2: Creating and Appending Elements

```js
// Step 1: Create a new element.
const newParagraph = document.createElement("p");
newParagraph.textContent = "This is a new paragraph.";

// Step 2: Append it to the body.
document.body.appendChild(newParagraph);
```

**Expected Output:** A new paragraph with the text "This is a new paragraph." appears at the end of the page.

**Why This Output:** `createElement("p")` creates a new `<p>` element. Setting `textContent` gives it text. `appendChild` adds it to the document body.

### Real-World Cases

- **Interactive forms:** Validating input and displaying error messages in real time.
- **Single-page applications:** Dynamically updating content without page reloads.
- **Animations:** Modifying element positions and styles over time.
- **Content management systems:** Loading and rendering content from APIs.

---

## DOM Tree

### Definitions

**Core Definition:** The DOM tree is the hierarchical tree structure that represents a document, with the `Document` node at the root and all other nodes (elements, text, comments) as its descendants.

**Technical Definition:** The DOM uses a logical tree to represent a document. A node tree is constrained by the types of nodes that can be children of other nodes. For example, a `Document` node can contain zero or more `ProcessingInstruction` or `Comment` nodes, optionally one `DocumentType` node, and optionally one `Element` node. An `Element` node can contain zero or more `Element`, `ProcessingInstruction`, `Comment`, or `Text` nodes. The document root is a `Document` node, while `Element` nodes form the backbone of this tree.

**Beginner-Friendly Explanation:** The DOM tree is like a family tree for your HTML. The `document` is the ancestor (root), the `<html>` element is its child, `<head>` and `<body>` are children of `<html>`, and so on. Text and comments are also part of the tree, as children of the elements that contain them.

### Purposes

- To provide a hierarchical representation of the document's structure.
- To enable navigation from any node to its parent, children, and siblings.
- To allow systematic traversal and manipulation of all parts of a document.
- To serve as the in-memory model that browsers use for rendering.

### Syntax Rules and Structure

The DOM tree is not created by syntax; it is constructed by the browser when it parses HTML.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <p>Welcome to my page.</p>
  </body>
</html>
```

The corresponding DOM tree:
```
Document
  └── html
       ├── head
       │    └── title
       │         └── "My Page" (Text)
       └── body
            ├── h1
            │    └── "Hello" (Text)
            └── p
                 └── "Welcome to my page." (Text)
```

**Component Breakdown:**
- **Document:** The root node.
- **Element nodes:** `<html>`, `<head>`, `<title>`, `<body>`, `<h1>`, `<p>`.
- **Text nodes:** `"My Page"`, `"Hello"`, `"Welcome to my page."`.

**Syntax Rules:**
- Every node except the root has exactly one parent.
- A node can have zero or more children.
- Nodes with the same parent are siblings.
- Tree order (document order) is pre-order depth-first traversal.

**Constraints and Limitations:**
- The DOM tree includes whitespace text nodes between elements, which can affect traversal.
- The tree can be large and complex; efficient traversal requires understanding of the node relationships.

### Annotated Code Examples

#### Example 1: Visualising the Tree

```js
// Step 1: Access the document root.
console.log(document.nodeType); // 9 (DOCUMENT_NODE)
console.log(document.documentElement.tagName); // "HTML"

// Step 2: Navigate to the body.
const body = document.body;
console.log(body.tagName); // "BODY"

// Step 3: Access children.
console.log(body.children.length); // Number of element children
console.log(body.childNodes.length); // Number of all child nodes
```

**Expected Output:**
```
9
HTML
BODY
2
5
```

**Why This Output:** `document.documentElement` gives the `<html>` element. `document.body` gives the `<body>`. `children` counts only element nodes; `childNodes` counts all nodes including text and comments.

### Real-World Cases

- **Rendering:** Browsers use the DOM tree to compute layout and paint pixels.
- **Accessibility:** Screen readers navigate the DOM tree to present content to users.
- **SEO:** Search engines parse the DOM to understand page content.

---

## Nodes

### Definitions

**Core Definition:** A node is a single point in the DOM tree. Everything in the DOM is a node — elements, text, comments, and the document itself.

**Technical Definition:** The `Node` interface is an abstract base class upon which many other DOM API objects are based. As an abstract class, there is no such thing as a plain `Node` object; all objects that implement node functionality are based on one of its subclasses. The most notable subclasses are `Document`, `Element`, and `DocumentFragment`. Other node types include `Attr`, `CharacterData` (which `Text`, `Comment`, `CDATASection`, and `ProcessingInstruction` are based on), and `DocumentType`.

**Beginner-Friendly Explanation:** A node is any "thing" in the DOM tree. An HTML tag is a node, the text inside it is a node, a comment is a node, and even the whole document is a node. Nodes are the building blocks of the DOM.

### Purposes

- To provide a common interface for all parts of the DOM tree.
- To enable uniform traversal and manipulation of different parts of the document.
- To define shared properties and methods for all node types.
- To serve as the base for specialised node subclasses.

### Node Types

| Node Type | `nodeType` Value | Description |
|-----------|-----------------|-------------|
| `ELEMENT_NODE` | 1 | An element node, such as `<p>` or `<div>` |
| `ATTRIBUTE_NODE` | 2 | An attribute of an element (deprecated) |
| `TEXT_NODE` | 3 | The actual text inside an element or attribute |
| `CDATA_SECTION_NODE` | 4 | A CDATA section (XML) |
| `PROCESSING_INSTRUCTION_NODE` | 7 | A processing instruction (XML) |
| `COMMENT_NODE` | 8 | A comment node, such as `<!-- comment -->` |
| `DOCUMENT_NODE` | 9 | The document itself (the root of the DOM tree) |
| `DOCUMENT_TYPE_NODE` | 10 | The DOCTYPE declaration |
| `DOCUMENT_FRAGMENT_NODE` | 11 | A lightweight document fragment |

### Annotated Code Examples

#### Example 1: Checking Node Types

```js
// Step 1: Get the body element.
const body = document.body;

// Step 2: Check its node type.
console.log(body.nodeType); // 1 (ELEMENT_NODE)
console.log(body.nodeType === Node.ELEMENT_NODE); // true

// Step 3: Get a text node.
const textNode = body.firstChild;
console.log(textNode.nodeType); // 3 (TEXT_NODE) or 1 (ELEMENT_NODE)

// Step 4: Get a comment node.
// Assuming: <body><!-- comment --><p>Text</p></body>
const comment = body.firstChild;
console.log(comment.nodeType); // 8 (COMMENT_NODE)
```

**Expected Output:**
```
1
true
3
8
```

**Why This Output:** `nodeType` returns a number identifying the node type. The `Node` interface provides constants like `Node.ELEMENT_NODE` (1) and `Node.TEXT_NODE` (3) for readability.

#### Example 2: Using `nodeName` and `nodeValue`

```js
const element = document.querySelector("p");
console.log(element.nodeName);  // "P"
console.log(element.nodeValue); // null (elements have no value)

const text = element.firstChild;
console.log(text.nodeName);  // "#text"
console.log(text.nodeValue); // "Hello, world!"
```

**Expected Output:**
```
P
null
#text
Hello, world!
```

**Why This Output:** `nodeName` for an element is its tag name in uppercase. For a text node, it is `"#text"`. `nodeValue` is `null` for elements and the text content for text nodes.

### Real-World Cases

- **Content extraction:** Identifying text nodes to extract or modify content.
- **Comment processing:** Reading or removing comments in the DOM.
- **Element manipulation:** Selecting and modifying element nodes.

---

## Elements

### Definitions

**Core Definition:** An element is a node that represents an HTML or XML tag, such as `<div>`, `<p>`, or `<span>`. Elements are the primary building blocks of a web page's structure.

**Technical Definition:** The `Element` interface is the most general base class from which all element objects in a `Document` inherit. It only has methods and properties common to all kinds of elements. More specific classes inherit from `Element`. For example, the `HTMLElement` interface is the base interface for HTML elements, while the `SVGElement` interface is the basis for all SVG elements. Elements are node type 1 (`ELEMENT_NODE`).

**Beginner-Friendly Explanation:** An element is what you probably think of as a "tag" in HTML. `<div>`, `<p>`, `<img>` — these are all elements. Elements can contain other elements, text, and comments.

### Purposes

- To represent HTML or XML tags in the DOM tree.
- To provide the primary interface for selecting and manipulating page content.
- To serve as the container for text, attributes, and other elements.
- To enable styling through CSS and interactivity through JavaScript.

### Syntax Rules and Structure

Elements are created by HTML tags and represented as `Element` objects in the DOM.

```html
<div id="container" class="main">
  <p>Hello</p>
</div>
```

```js
// Accessing elements:
const container = document.getElementById("container");
console.log(container.tagName); // "DIV"
console.log(container.id); // "container"
console.log(container.className); // "main"
```

**Component Breakdown:**
- `tagName`: The uppercase name of the element.
- `id`: The element's unique identifier.
- `className`: The element's class attribute.
- `children`: A live collection of child elements.

**Syntax Rules:**
- Elements are node type 1 (`Node.ELEMENT_NODE`).
- Elements can have attributes (e.g., `id`, `class`, `href`).
- Elements can contain other elements, text, and comments.
- `Element` is an abstract base class; actual elements are instances of subclasses like `HTMLDivElement`.

**Constraints and Limitations:**
- Not all `Node` properties apply to `Element` (e.g., `nodeValue` is `null` for elements).
- Elements cannot be added as children of node types that don't allow element children (e.g., `Text` nodes).

### Annotated Code Examples

#### Example 1: Element Properties

```js
// Step 1: Select an element.
const link = document.querySelector("a");

// Step 2: Read element properties.
console.log(link.tagName);      // "A"
console.log(link.href);         // Full URL
console.log(link.textContent);  // Text inside the link
console.log(link.children.length); // Number of child elements
```

**Expected Output:**
```
A
https://example.com
Click me
0
```

**Why This Output:** `tagName` is the element's name. `href` is the resolved URL. `textContent` is the text inside. `children.length` is 0 because the link contains only a text node, not child elements.

#### Example 2: Creating Elements

```js
// Step 1: Create a new element.
const newDiv = document.createElement("div");
newDiv.id = "myDiv";
newDiv.className = "box";
newDiv.textContent = "Hello!";

// Step 2: Add it to the DOM.
document.body.appendChild(newDiv);

// Step 3: Verify.
console.log(document.getElementById("myDiv").textContent); // "Hello!"
```

**Expected Output:**
```
Hello!
```

**Why This Output:** `createElement` creates a new element not yet attached to the DOM. Setting properties configures it. `appendChild` attaches it to the document body.

### Real-World Cases

- **Dynamic content:** Creating and inserting elements in response to user actions.
- **Form manipulation:** Reading and setting form field values.
- **Styling:** Adding, removing, or toggling CSS classes.
- **Event handling:** Attaching event listeners to elements.

---

## Text Nodes

### Definitions

**Core Definition:** A text node is a node that contains the actual text content inside an element. It is always a child of an element node (or an attribute node).

**Technical Definition:** The `Text` interface represents a text node in the DOM tree. It inherits from `CharacterData`, which provides methods for manipulating the text content. Text nodes are node type 3 (`Node.TEXT_NODE`). A `Text` node's `nodeValue` property contains the text string.

**Beginner-Friendly Explanation:** A text node is just the words inside an HTML tag. In `<p>Hello</p>`, the text node is `"Hello"`. Text nodes are separate from elements — the `<p>` is an element, and `"Hello"` is a text node child of that element.

### Purposes

- To store the textual content of a document.
- To allow programs to read and modify text independently of the elements that contain it.
- To enable fine-grained text manipulation within elements.

### Syntax Rules and Structure

Text nodes are created automatically by the browser when parsing HTML, or programmatically using `document.createTextNode()`.

```js
// Creating a text node:
const textNode = document.createTextNode("Hello, world!");

// Appending to an element:
document.body.appendChild(textNode);

// Reading text content:
console.log(textNode.nodeValue); // "Hello, world!"
console.log(textNode.nodeName);  // "#text"
console.log(textNode.nodeType);  // 3
```

**Component Breakdown:**
- `nodeValue`: The text content of the node.
- `nodeName`: Always `"#text"`.
- `nodeType`: Always 3.

**Syntax Rules:**
- Text nodes cannot have children.
- Text nodes can be siblings of element nodes and other text nodes.
- Whitespace between elements creates text nodes.

**Constraints and Limitations:**
- Text nodes cannot contain HTML elements.
- Modifying a text node's `nodeValue` changes the displayed text immediately.
- Text nodes are often overlooked in traversal; use `children` (elements only) vs `childNodes` (all nodes) carefully.

### Annotated Code Examples

#### Example 1: Accessing Text Nodes

```js
// HTML: <p>Hello, world!</p>
const p = document.querySelector("p");

// Step 1: Get the text node.
const textNode = p.firstChild;

// Step 2: Check its properties.
console.log(textNode.nodeType);  // 3
console.log(textNode.nodeName);  // "#text"
console.log(textNode.nodeValue); // "Hello, world!"

// Step 3: Modify the text.
textNode.nodeValue = "Goodbye!";
console.log(p.textContent); // "Goodbye!"
```

**Expected Output:**
```
3
#text
Hello, world!
Goodbye!
```

**Why This Output:** `firstChild` of the `<p>` is the text node. `nodeValue` contains the text. Changing it updates the display.

#### Example 2: Creating and Inserting Text

```js
// Step 1: Create a text node.
const text = document.createTextNode("New text");

// Step 2: Create an element and append the text.
const p = document.createElement("p");
p.appendChild(text);

// Step 3: Add to the document.
document.body.appendChild(p);
```

**Expected Output:** A new paragraph with "New text" appears on the page.

**Why This Output:** `createTextNode` creates a standalone text node. Appending it to a `<p>` element makes it the element's child.

### Real-World Cases

- **Dynamic text updates:** Updating the text content of a message or label.
- **Localisation:** Replacing text nodes with translated content.
- **User-generated content:** Inserting user-supplied text into the DOM.

---

## Comment Nodes

### Definitions

**Core Definition:** A comment node is a node that represents an HTML or XML comment, such as `<!-- this is a comment -->`.

**Technical Definition:** The `Comment` interface represents a comment node. It inherits from `CharacterData`, which provides methods for manipulating the comment's text. Comment nodes are node type 8 (`Node.COMMENT_NODE`). A comment node's `nodeValue` contains the comment text (without the `<!--` and `-->` delimiters).

**Beginner-Friendly Explanation:** A comment node is just a comment in the HTML. It's part of the DOM tree, but it's not displayed on the page. Comments can be read and modified by JavaScript.

### Purposes

- To represent comments in the DOM tree.
- To allow programs to read or remove comments.
- To support conditional comments (in older IE) and other comment-based techniques.

### Syntax Rules and Structure

Comment nodes are created by the browser when parsing HTML comments, or programmatically using `document.createComment()`.

```js
// Creating a comment node:
const comment = document.createComment("This is a comment");

// Appending to an element:
document.body.appendChild(comment);

// Reading comment properties:
console.log(comment.nodeValue); // "This is a comment"
console.log(comment.nodeName);  // "#comment"
console.log(comment.nodeType);  // 8
```

**Component Breakdown:**
- `nodeValue`: The comment text (without delimiters).
- `nodeName`: Always `"#comment"`.
- `nodeType`: Always 8.

**Syntax Rules:**
- Comment nodes cannot have children.
- Comments can appear anywhere in the DOM tree.
- Comments are part of `childNodes` but not `children`.

**Constraints and Limitations:**
- Comments are rarely manipulated in modern web development.
- Some frameworks and minifiers remove comments for performance.

### Annotated Code Examples

#### Example 1: Reading and Modifying Comments

```js
// HTML: <body><!-- This is a comment --><p>Text</p></body>
const body = document.body;

// Step 1: Get the comment node.
const comment = body.firstChild;

// Step 2: Check properties.
console.log(comment.nodeType);  // 8
console.log(comment.nodeName);  // "#comment"
console.log(comment.nodeValue); // " This is a comment "

// Step 3: Modify the comment.
comment.nodeValue = " Updated comment ";
console.log(comment.nodeValue); // " Updated comment "
```

**Expected Output:**
```
8
#comment
 This is a comment 
 Updated comment 
```

**Why This Output:** The comment node's `nodeValue` contains the text between `<!--` and `-->`. Modifying it changes the comment in the DOM (though not visibly on the page).

### Real-World Cases

- **Conditional comments:** Historically used for IE-specific content (now obsolete).
- **Debugging:** Adding temporary comments to mark sections of the DOM.
- **Server-side rendering:** Comments used as markers for hydration in frameworks like React.

---

## Document

### Definitions

**Core Definition:** The `Document` interface represents the entire web page loaded in the browser and serves as the entry point to the page's content (the DOM tree).

**Technical Definition:** The `Document` interface represents any web page loaded in the browser and serves as an entry point into the web page's content, which is the DOM tree. It describes the common properties and methods for any kind of document. Depending on the document type (HTML, XML, SVG), a larger API may be available: HTML documents served with `"text/html"` also implement the `HTMLDocument` interface, while XML and SVG documents implement the `XMLDocument` interface.

**Beginner-Friendly Explanation:** The `document` object is your main entry point to the page. It represents the entire HTML document. You use it to find elements, create new ones, and access page-level information like the title and URL.

### Purposes

- To serve as the root of the DOM tree.
- To provide methods for selecting elements (`querySelector`, `getElementById`).
- To provide methods for creating elements and other nodes.
- To expose document-level properties (title, URL, cookies, etc.).
- To serve as the event target for document-level events (e.g., `DOMContentLoaded`).

### Syntax Rules and Structure

The `document` object is a global variable in browser environments.

```js
// Accessing document properties:
console.log(document.title);     // Page title
console.log(document.URL);       // Page URL
console.log(document.documentElement); // The <html> element
console.log(document.body);      // The <body> element

// Selecting elements:
const el = document.getElementById("myId");
const els = document.querySelectorAll(".myClass");

// Creating elements:
const div = document.createElement("div");
const text = document.createTextNode("Hello");
```

**Component Breakdown:**
- `document.title`: Gets or sets the document title.
- `document.URL`: The URL of the document.
- `document.documentElement`: The root `<html>` element.
- `document.body`: The `<body>` element.
- `document.getElementById()`: Selects an element by ID.
- `document.querySelectorAll()`: Selects elements by CSS selector.

**Syntax Rules:**
- `document` is an instance of `Document` (and `HTMLDocument` for HTML pages).
- `document` is a property of the `window` object (`window.document`).
- `document` is a node (node type 9) and has `childNodes` and `children`.

**Constraints and Limitations:**
- The `document` object is only available in browser environments (not in Node.js).
- Some properties and methods are specific to HTML documents.

### Annotated Code Examples

#### Example 1: Document-Level Access

```js
// Step 1: Get document information.
console.log(document.title);   // "My Page"
console.log(document.URL);     // "https://example.com/page"

// Step 2: Access the root element.
const html = document.documentElement;
console.log(html.tagName); // "HTML"

// Step 3: Access the body.
const body = document.body;
console.log(body.tagName); // "BODY"
```

**Expected Output:**
```
My Page
https://example.com/page
HTML
BODY
```

**Why This Output:** `document.title` returns the page title. `document.URL` returns the full URL. `documentElement` is the `<html>` element. `body` is the `<body>` element.

#### Example 2: Creating Elements with Document

```js
// Step 1: Create elements using the document.
const heading = document.createElement("h1");
heading.textContent = "Welcome";

const paragraph = document.createElement("p");
paragraph.textContent = "This is a paragraph.";

// Step 2: Append to the body.
document.body.appendChild(heading);
document.body.appendChild(paragraph);
```

**Expected Output:** An `<h1>` and a `<p>` are added to the page.

**Why This Output:** `document.createElement` creates new elements. `appendChild` attaches them to the body.

### Real-World Cases

- **Element selection:** Finding and manipulating elements on the page.
- **Dynamic content:** Creating and inserting new content.
- **Page metadata:** Reading and setting the page title.
- **Event handling:** Listening for document-level events like `DOMContentLoaded`.

---

## `window`

### Definitions

**Core Definition:** The `window` object represents the browser window (or tab) that contains the DOM document. It is the global object in browser JavaScript.

**Technical Definition:** The `Window` interface represents a window containing a DOM document; the `document` property points to the DOM document loaded in that window. A global variable `window`, representing the window in which the script is running, is exposed to JavaScript code. In tabbed browsers, each tab is represented by its own `Window` object. The `Window` interface contains various functions, namespaces, objects, and constructors that are not necessarily directly related to the concept of a user interface window, but are appropriate places for globally available items.

**Beginner-Friendly Explanation:** The `window` object is the "global" object in browser JavaScript. Everything global — variables, functions, and the `document` — lives inside it. It also represents the browser window itself, so it has properties like `innerWidth` and methods like `alert()`.

### Purposes

- To serve as the global object in browser JavaScript.
- To represent the browser window or tab.
- To provide access to the `document` object.
- To expose browser-level APIs (e.g., `alert`, `setTimeout`, `localStorage`).
- To provide window dimensions and scrolling methods.

### Syntax Rules and Structure

The `window` object is the global object; its properties and methods can be accessed directly.

```js
// Accessing window properties:
console.log(window.innerWidth);  // Window width
console.log(window.innerHeight); // Window height
console.log(window.document);    // The document object
console.log(window.location);    // The URL

// Calling window methods:
window.alert("Hello!");
window.setTimeout(() => console.log("Done"), 1000);

// Global variables are properties of window:
var myVar = 42;
console.log(window.myVar); // 42
```

**Component Breakdown:**
- `window.document`: The DOM document.
- `window.location`: The current URL and navigation methods.
- `window.innerWidth` / `innerHeight`: The viewport dimensions.
- `window.alert()`: Displays an alert dialog.
- `window.setTimeout()`: Schedules a function.

**Syntax Rules:**
- `window` is the global object; `this` at the top level of a script refers to `window`.
- Global variables declared with `var` become properties of `window`.
- `window` is an instance of the `Window` interface.

**Constraints and Limitations:**
- The `window` object is only available in browser environments.
- Some `Window` properties and methods are not directly related to the visible window (e.g., `caches`, `crypto`).

### Annotated Code Examples

#### Example 1: Window Dimensions and Document

```js
// Step 1: Get window dimensions.
console.log(window.innerWidth);  // e.g., 1024
console.log(window.innerHeight); // e.g., 768

// Step 2: Access the document.
const doc = window.document;
console.log(doc === document); // true

// Step 3: Navigate.
console.log(window.location.href); // "https://example.com"
```

**Expected Output:**
```
1024
768
true
https://example.com
```

**Why This Output:** `innerWidth` and `innerHeight` are the viewport dimensions. `window.document` is the same as the global `document`. `location.href` is the current URL.

#### Example 2: Global Variables and Window

```js
// Step 1: Declare a global variable with var.
var globalVar = "Hello";

// Step 2: Access it via window.
console.log(window.globalVar); // "Hello"

// Step 3: Declare with let (does not become a window property).
let blockVar = "World";
console.log(window.blockVar); // undefined
```

**Expected Output:**
```
Hello
undefined
```

**Why This Output:** `var` declarations at the top level become properties of `window`. `let` and `const` declarations do not.

### Real-World Cases

- **Browser APIs:** Accessing `localStorage`, `sessionStorage`, `fetch`, etc.
- **Window manipulation:** Resizing, scrolling, and opening new windows.
- **Global configuration:** Setting global variables for application configuration.

---

## `document`

### Definitions

**Core Definition:** The `document` object is the entry point to the DOM. It represents the HTML or XML document loaded in the browser and provides methods for selecting, creating, and manipulating elements.

**Technical Definition:** The `Document` interface represents any web page loaded in the browser and serves as an entry point into the web page's content, which is the DOM tree. The `document` property of the `window` object points to the DOM document loaded in that window. The `document` object is a node (node type 9, `DOCUMENT_NODE`) and is the root of the DOM tree.

**Beginner-Friendly Explanation:** The `document` object is your main tool for working with the page. It's how you find elements, create new ones, and get information about the page. You'll use `document` constantly in DOM manipulation.

### Purposes

- To provide the primary API for DOM manipulation.
- To serve as the root of the DOM tree.
- To expose document-level information (title, URL, cookies).
- To provide element selection and creation methods.
- To serve as an event target for document-level events.

### Syntax Rules and Structure

The `document` object is globally available.

```js
// Selection methods:
document.getElementById("id");
document.getElementsByClassName("class");
document.getElementsByTagName("tag");
document.querySelector("selector");
document.querySelectorAll("selector");

// Creation methods:
document.createElement("tag");
document.createTextNode("text");
document.createComment("comment");

// Document properties:
document.title;
document.URL;
document.documentElement;
document.body;
document.head;
```

**Component Breakdown:**
- `getElementById`: Returns a single element by ID.
- `querySelector`: Returns the first element matching a CSS selector.
- `querySelectorAll`: Returns a NodeList of all matching elements.
- `createElement`: Creates a new element node.

**Syntax Rules:**
- `document` is available as a global variable in browsers.
- `document` is an instance of `Document` (and `HTMLDocument` for HTML pages).
- All DOM manipulation begins with the `document` object.

**Constraints and Limitations:**
- `document` is not available in Node.js.
- Some methods return live collections (`getElementsByClassName`) while others return static collections (`querySelectorAll`).

### Annotated Code Examples

#### Example 1: Selecting Elements

```js
// Step 1: Select by ID.
const header = document.getElementById("header");

// Step 2: Select by class.
const items = document.getElementsByClassName("item");

// Step 3: Select by CSS selector.
const firstItem = document.querySelector(".item");
const allItems = document.querySelectorAll(".item");

console.log(header.tagName);     // "HEADER" or "DIV", etc.
console.log(items.length);       // Number of .item elements
console.log(firstItem.textContent); // Text of first .item
console.log(allItems.length);    // Number of .item elements
```

**Expected Output:**
```
HEADER
3
Item 1
3
```

**Why This Output:** Each selection method returns elements based on the specified criteria.

#### Example 2: Creating and Appending Elements

```js
// Step 1: Create a list.
const ul = document.createElement("ul");

// Step 2: Create list items.
for (let i = 1; i <= 3; i++) {
  const li = document.createElement("li");
  li.textContent = `Item ${i}`;
  ul.appendChild(li);
}

// Step 3: Append to the document.
document.body.appendChild(ul);
```

**Expected Output:** An unordered list with three items is added to the page.

**Why This Output:** `createElement` creates new elements. `appendChild` builds the tree. The final `appendChild` attaches the complete list to the body.

### Real-World Cases

- **Form handling:** Reading and validating form input values.
- **Dynamic lists:** Building lists from data arrays.
- **Content updates:** Replacing or updating text and HTML content.
- **Event delegation:** Attaching a single event listener to the document for multiple elements.

---

## DOM Traversal

### Definitions

**Core Definition:** DOM traversal is the process of navigating through the DOM tree by moving from one node to its parent, children, or siblings.

**Technical Definition:** The DOM provides properties for navigating between parents, children, and siblings. The DOM only exposes one traversal order: pre-order depth-first search (DFS), which is called document order or tree order. In pre-order, an element precedes its descendants, and earlier siblings and their descendants precede later siblings.

**Beginner-Friendly Explanation:** DOM traversal is how you move around the DOM tree. From any node, you can go up to its parent, down to its children, or sideways to its siblings. This is how you find related elements when you have a reference to one.

### Purposes

- To navigate from a known node to related nodes.
- To walk the entire DOM tree systematically.
- To find elements based on their position in the tree.
- To implement custom selection and filtering logic.

### Node Relationship Properties

| Property | Description |
|----------|-------------|
| `parentNode` | The parent of the node |
| `childNodes` | A live NodeList of all child nodes (elements, text, comments) |
| `firstChild` | The first child node |
| `lastChild` | The last child node |
| `nextSibling` | The next sibling node |
| `previousSibling` | The previous sibling node |
| `children` | A live HTMLCollection of child elements only |
| `firstElementChild` | The first child element |
| `lastElementChild` | The last child element |
| `nextElementSibling` | The next sibling element |
| `previousElementSibling` | The previous sibling element |

### Annotated Code Examples

#### Example 1: Navigating the Tree

```js
// HTML: <div id="parent"><p>First</p><p>Second</p></div>
const parent = document.getElementById("parent");

// Step 1: Get children.
const children = parent.children;
console.log(children.length); // 2

// Step 2: Get first and last element children.
console.log(parent.firstElementChild.textContent); // "First"
console.log(parent.lastElementChild.textContent);  // "Second"

// Step 3: Navigate siblings.
const firstP = parent.firstElementChild;
console.log(firstP.nextElementSibling.textContent); // "Second"
console.log(firstP.nextElementSibling.previousElementSibling.textContent); // "First"

// Step 4: Navigate to parent.
console.log(firstP.parentNode.id); // "parent"
```

**Expected Output:**
```
2
First
Second
Second
First
parent
```

**Why This Output:** `children` gives element children. `firstElementChild` and `lastElementChild` give the first and last element children. Sibling navigation moves sideways. `parentNode` moves up.

#### Example 2: Traversing All Nodes

```js
// Step 1: Recursive traversal function.
function traverse(node, depth = 0) {
  console.log(" ".repeat(depth) + node.nodeName);
  for (const child of node.childNodes) {
    traverse(child, depth + 1);
  }
}

// Step 2: Traverse the body.
traverse(document.body);
```

**Expected Output:** A tree-like indented listing of all nodes in the body.

**Why This Output:** The function visits each node, prints its name, and recursively visits all children. `childNodes` includes text and comment nodes, so they appear in the output.

### Real-World Cases

- **Finding related elements:** Getting the parent of a clicked button.
- **Tree walking:** Implementing custom algorithms that process every node.
- **Form navigation:** Moving between form fields using next/previous siblings.

---

## Parent Nodes

### Definitions

**Core Definition:** A parent node is a node that contains other nodes (its children) in the DOM tree.

**Technical Definition:** The `parentNode` property returns the parent of the specified node in the DOM tree. A node's parent is the node that contains it. The document root (`Document` node) has no parent (`parentNode` is `null`).

**Beginner-Friendly Explanation:** A parent node is the "container" of another node. In `<div><p>Hello</p></div>`, the `<div>` is the parent of the `<p>`, and the `<p>` is the parent of the text node `"Hello"`.

### Purposes

- To navigate upward in the DOM tree.
- To find the containing element of a given node.
- To access ancestor elements for styling or event delegation.

### Syntax Rules and Structure

```js
const child = document.querySelector("p");
const parent = child.parentNode;
console.log(parent.tagName); // The parent element's tag name
```

**Component Breakdown:**
- `parentNode`: Returns the parent `Node` or `null` if there is no parent.

**Syntax Rules:**
- Every node except the `Document` root has a parent.
- `parentNode` is read-only.
- The parent of an element is typically another element, but can be a `Document` or `DocumentFragment`.

### Annotated Code Examples

#### Example 1: Finding a Parent

```js
// HTML: <div id="container"><p id="text">Hello</p></div>
const p = document.getElementById("text");
const parent = p.parentNode;

console.log(parent.id);       // "container"
console.log(parent.tagName);  // "DIV"
```

**Expected Output:**
```
container
DIV
```

**Why This Output:** The `<p>`'s parent is the `<div>` with id `"container"`.

### Real-World Cases

- **Event delegation:** Finding the parent element of a clicked target.
- **Traversing up:** Moving from a text node to its containing element.
- **Styling ancestors:** Adding a class to a parent based on a child's state.

---

## Child Nodes

### Definitions

**Core Definition:** Child nodes are the nodes directly contained within a parent node in the DOM tree.

**Technical Definition:** The `childNodes` property returns a live `NodeList` containing all child nodes of the specified node, including element nodes, text nodes, and comment nodes. The `children` property returns a live `HTMLCollection` containing only the element children.

**Beginner-Friendly Explanation:** Child nodes are the "contents" of a node. An element's child nodes are everything directly inside it — other elements, text, and comments.

### Purposes

- To access and iterate over the contents of an element.
- To count the number of children.
- To manipulate child nodes (add, remove, replace).

### Syntax Rules and Structure

```js
// All child nodes (elements, text, comments):
const allChildren = parent.childNodes;

// Element children only:
const elementChildren = parent.children;
```

**Component Breakdown:**
- `childNodes`: A live NodeList of all child nodes.
- `children`: A live HTMLCollection of element children only.

**Syntax Rules:**
- `childNodes` includes text and comment nodes; `children` includes only elements.
- Both are live collections: they update automatically when the DOM changes.
- `childNodes.length` gives the total count; `children.length` gives the element count.

### Annotated Code Examples

#### Example 1: Comparing `childNodes` and `children`

```js
// HTML: <div id="parent">Text <p>Paragraph</p><!-- comment --></div>
const parent = document.getElementById("parent");

console.log(parent.childNodes.length); // 3 (text, p, comment)
console.log(parent.children.length);   // 1 (only p)

// Iterating:
for (const node of parent.childNodes) {
  console.log(node.nodeType, node.nodeName);
}
```

**Expected Output:**
```
3
1
3 #text
1 P
8 #comment
```

**Why This Output:** `childNodes` includes the text node, the `<p>` element, and the comment. `children` includes only the `<p>`.

### Real-World Cases

- **Iterating over content:** Processing all child nodes of a container.
- **Clearing an element:** Removing all children by iterating `childNodes`.
- **Counting elements:** Using `children.length` to count element children.

---

## Sibling Nodes

### Definitions

**Core Definition:** Sibling nodes are nodes that share the same parent in the DOM tree.

**Technical Definition:** The `nextSibling` and `previousSibling` properties return the next and previous sibling nodes (including text and comment nodes). The `nextElementSibling` and `previousElementSibling` properties return the next and previous sibling elements only.

**Beginner-Friendly Explanation:** Siblings are nodes that are "next to each other" under the same parent. In `<div><p>A</p><p>B</p></div>`, the two `<p>` elements are siblings.

### Purposes

- To navigate horizontally between nodes with the same parent.
- To find adjacent elements.
- To implement loops that iterate over siblings.

### Syntax Rules and Structure

```js
const node = document.querySelector("p");

// All siblings (including text and comments):
const next = node.nextSibling;
const prev = node.previousSibling;

// Element siblings only:
const nextEl = node.nextElementSibling;
const prevEl = node.previousElementSibling;
```

**Component Breakdown:**
- `nextSibling` / `previousSibling`: The next/previous node (any type).
- `nextElementSibling` / `previousElementSibling`: The next/previous element node.

**Syntax Rules:**
- The first child has no `previousSibling` (returns `null`).
- The last child has no `nextSibling` (returns `null`).
- Element sibling properties skip text and comment nodes.

### Annotated Code Examples

#### Example 1: Navigating Siblings

```js
// HTML: <ul><li>One</li><li>Two</li><li>Three</li></ul>
const firstLi = document.querySelector("li");

console.log(firstLi.nextElementSibling.textContent); // "Two"
console.log(firstLi.nextElementSibling.nextElementSibling.textContent); // "Three"
console.log(firstLi.nextElementSibling.previousElementSibling.textContent); // "One"
```

**Expected Output:**
```
Two
Three
One
```

**Why This Output:** `nextElementSibling` moves to the next `<li>`. Chaining allows navigation through all siblings. `previousElementSibling` moves back.

### Real-World Cases

- **Tab navigation:** Moving focus between adjacent form fields.
- **List manipulation:** Reordering list items by moving siblings.
- **Styling:** Applying styles to adjacent elements (e.g., `li + li`).

---

## Node Relationships

### Definitions

**Core Definition:** Node relationships describe how nodes are connected in the DOM tree: parent-child, sibling, and ancestor-descendant relationships.

**Technical Definition:** The DOM tree defines relationships between nodes based on their positions. A node's parent is the node that contains it; its children are the nodes it contains; its siblings are nodes with the same parent. Tree order (document order) determines the sequence in which nodes appear in a pre-order depth-first traversal.

**Beginner-Friendly Explanation:** Node relationships are the "family tree" connections in the DOM. Every node has a place: it has a parent (except the root), it may have children, and it may have siblings.

### Purposes

- To understand the structure of the DOM tree.
- To navigate between related nodes efficiently.
- To implement algorithms that depend on tree relationships.

### Annotated Code Examples

#### Example 1: Mapping Relationships

```js
// HTML: <div id="a"><p id="b">Text</p><p id="c">More</p></div>
const b = document.getElementById("b");
const c = document.getElementById("c");
const a = document.getElementById("a");

// Parent-child:
console.log(b.parentNode === a); // true
console.log(a.children[0] === b); // true

// Siblings:
console.log(b.nextElementSibling === c); // true
console.log(c.previousElementSibling === b); // true

// Ancestor-descendant:
console.log(a.contains(b)); // true
console.log(b.contains(a)); // false
```

**Expected Output:**
```
true
true
true
true
true
false
```

**Why This Output:** `b`'s parent is `a`. `a`'s first child is `b`. `b`'s next sibling is `c`. `a` contains `b` as a descendant.

### Real-World Cases

- **Event bubbling:** Events propagate up the parent chain.
- **CSS selectors:** Combinators like `>`, `+`, `~` use node relationships.
- **Tree algorithms:** Recursive processing of nested structures.

---

## Node Types

### Definitions

**Core Definition:** Node types are the different kinds of nodes that can exist in the DOM tree, each with a specific `nodeType` numeric value and corresponding interface.

**Technical Definition:** Every kind of DOM node is represented by an interface based on `Node`. These include `Attr`, `CharacterData` (which `Text`, `Comment`, `CDATASection`, and `ProcessingInstruction` are based on), and `DocumentType`. The `nodeType` property returns an unsigned short integer representing the type.

**Beginner-Friendly Explanation:** Nodes come in different "flavours" — elements, text, comments, and more. The `nodeType` property tells you which flavour a node is.

### Node Type Constants

| Constant | Value | Interface | Description |
|----------|-------|-----------|-------------|
| `Node.ELEMENT_NODE` | 1 | `Element` | An HTML or XML element |
| `Node.ATTRIBUTE_NODE` | 2 | `Attr` | An attribute (deprecated) |
| `Node.TEXT_NODE` | 3 | `Text` | Text content |
| `Node.CDATA_SECTION_NODE` | 4 | `CDATASection` | CDATA section (XML) |
| `Node.PROCESSING_INSTRUCTION_NODE` | 7 | `ProcessingInstruction` | Processing instruction (XML) |
| `Node.COMMENT_NODE` | 8 | `Comment` | An HTML/XML comment |
| `Node.DOCUMENT_NODE` | 9 | `Document` | The document root |
| `Node.DOCUMENT_TYPE_NODE` | 10 | `DocumentType` | The DOCTYPE |
| `Node.DOCUMENT_FRAGMENT_NODE` | 11 | `DocumentFragment` | A lightweight document fragment |

### Annotated Code Examples

#### Example 1: Identifying Node Types

```js
// HTML: <p>Hello<!-- comment --></p>
const p = document.querySelector("p");

p.childNodes.forEach(node => {
  switch (node.nodeType) {
    case Node.ELEMENT_NODE:
      console.log("Element:", node.tagName);
      break;
    case Node.TEXT_NODE:
      console.log("Text:", node.nodeValue);
      break;
    case Node.COMMENT_NODE:
      console.log("Comment:", node.nodeValue);
      break;
  }
});
```

**Expected Output:**
```
Text: Hello
Comment:  comment 
```

**Why This Output:** The `<p>` has two child nodes: a text node and a comment node. The switch statement identifies each by `nodeType`.

### Real-World Cases

- **Content filtering:** Removing comments or whitespace text nodes.
- **Serialisation:** Converting the DOM to HTML by processing each node type.
- **Tree processing:** Handling different node types appropriately in algorithms.

---

## Node Properties

### Definitions

**Core Definition:** Node properties are the attributes and characteristics exposed by the `Node` interface that describe a node's type, name, value, and relationships.

**Technical Definition:** The `Node` interface provides properties including `nodeName`, `nodeType`, `nodeValue`, `textContent`, `childNodes`, `parentNode`, `firstChild`, `lastChild`, `nextSibling`, `previousSibling`, `ownerDocument`, `baseURI`, and `isConnected`.

**Beginner-Friendly Explanation:** Node properties are the "details" you can read about a node: what kind it is, what it's called, what's inside it, and who its relatives are.

### Key Node Properties

| Property | Description |
|----------|-------------|
| `nodeName` | The name of the node (e.g., `"DIV"`, `"#text"`, `"#comment"`) |
| `nodeType` | A number identifying the node type (1 = element, 3 = text, 8 = comment) |
| `nodeValue` | The value of the node (`null` for elements, text for text/comment nodes) |
| `textContent` | The text content of the node and its descendants |
| `childNodes` | A live NodeList of all child nodes |
| `parentNode` | The parent node |
| `firstChild` | The first child node |
| `lastChild` | The last child node |
| `nextSibling` | The next sibling node |
| `previousSibling` | The previous sibling node |
| `ownerDocument` | The Document object this node belongs to |
| `isConnected` | Whether the node is connected to the document |

### Annotated Code Examples

#### Example 1: Reading Node Properties

```js
const div = document.querySelector("div");

console.log(div.nodeName);        // "DIV"
console.log(div.nodeType);        // 1
console.log(div.nodeValue);       // null
console.log(div.textContent);     // Text content
console.log(div.childNodes.length); // Number of children
console.log(div.parentNode.tagName); // Parent tag name
console.log(div.ownerDocument === document); // true
console.log(div.isConnected);     // true
```

**Expected Output:**
```
DIV
1
null
Hello, world!
3
BODY
true
true
```

**Why This Output:** Each property returns the corresponding information about the `<div>` node.

### Real-World Cases

- **Conditional logic:** Using `nodeType` to handle different node types.
- **Text extraction:** Using `textContent` to get all text from a subtree.
- **DOM validation:** Checking `isConnected` before manipulating a node.

---

## Node Methods

### Definitions

**Core Definition:** Node methods are the functions provided by the `Node` interface for manipulating the DOM tree, such as adding, removing, and replacing nodes.

**Technical Definition:** The `Node` interface provides methods including `appendChild()`, `removeChild()`, `replaceChild()`, `insertBefore()`, `cloneNode()`, `contains()`, `hasChildNodes()`, `isEqualNode()`, and `normalize()`.

**Beginner-Friendly Explanation:** Node methods are the "actions" you can perform on nodes: adding children, removing them, replacing them, and checking relationships.

### Key Node Methods

| Method | Description |
|--------|-------------|
| `appendChild(node)` | Adds a node as the last child |
| `insertBefore(newNode, refNode)` | Inserts a node before a reference node |
| `removeChild(node)` | Removes a child node |
| `replaceChild(newNode, oldNode)` | Replaces a child node |
| `cloneNode(deep)` | Clones a node (deep or shallow) |
| `contains(node)` | Checks if a node is a descendant |
| `hasChildNodes()` | Checks if a node has children |
| `isEqualNode(other)` | Checks if two nodes are equal |
| `normalize()` | Merges adjacent text nodes and removes empty text nodes |

### Annotated Code Examples

#### Example 1: Creating and Inserting Nodes

```js
// Step 1: Create a new element.
const newP = document.createElement("p");
newP.textContent = "New paragraph";

// Step 2: Insert before an existing element.
const existing = document.querySelector("p");
existing.parentNode.insertBefore(newP, existing);

// Step 3: Remove an element.
existing.parentNode.removeChild(existing);
```

**Expected Output:** The new paragraph is inserted before the existing one, then the existing one is removed.

**Why This Output:** `insertBefore` places the new node before the reference node. `removeChild` removes a node from its parent.

#### Example 2: Cloning Nodes

```js
// Step 1: Select an element.
const original = document.querySelector("div");

// Step 2: Clone it (deep clone includes children).
const clone = original.cloneNode(true);

// Step 3: Modify the clone.
clone.id = "clone";
document.body.appendChild(clone);

console.log(original.id); // "original"
console.log(clone.id);    // "clone"
```

**Expected Output:**
```
original
clone
```

**Why This Output:** `cloneNode(true)` creates a deep copy. Modifying the clone does not affect the original.

### Real-World Cases

- **Dynamic lists:** Adding and removing list items.
- **UI components:** Cloning templates for repeated content.
- **DOM cleanup:** Removing elements when no longer needed.

---

## References

- MDN Web Docs – DOM 简介 — https://mdn.org.cn/en-US/docs/Web/API/Document_Object_Model/Introduction
- MDN Web Docs – Introduction to the DOM — https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction
- MDN Web Docs – Node — https://developer.mozilla.org/en-US/docs/Web/API/Node
- MDN Web Docs – Node (中文) — https://developer.mozilla.org/zh-CN/docs/Web/API/Node
- MDN Web Docs – Node: nodeType — https://mdn.org.cn/en-US/docs/Web/API/Node/nodeType
- MDN Web Docs – Selection and traversal on the DOM tree — https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Selection_and_traversal_on_the_DOM_tree
- MDN Web Docs – Window — https://mdn.org.cn/zh-CN/docs/Web/API/Window
- MDN Web Docs – Document — https://mdn.org.cn/en-US/docs/Web/API/Document
- MDN Web Docs – Element — https://developer.mozilla.org/en-US/docs/Web/API/Element
- WHATWG DOM Living Standard — https://dom.spec.whatwg.org/
- W3C DOM4 Specification — https://www.w3.org/TR/dom/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf