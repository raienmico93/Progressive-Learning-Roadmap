# Selecting Elements — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Selecting elements is the process of retrieving references to specific DOM nodes from the document tree using JavaScript methods. It is the foundational step for any DOM manipulation, event handling, or content update.

**Technical Definition:** The DOM provides multiple methods for element selection, each with different performance characteristics, return types, and matching capabilities. The primary methods are: `getElementById()` (returns a single `Element` by unique ID), `getElementsByClassName()` and `getElementsByTagName()` (return live `HTMLCollection`s), `getElementsByName()` (returns a live `NodeList` by `name` attribute), and the Selectors API methods `querySelector()` and `querySelectorAll()` (return static `NodeList`s based on CSS selectors). Additionally, `closest()` and `matches()` provide relational selection capabilities. Understanding the differences between these methods — particularly live vs static collections and performance implications — is essential for writing efficient DOM manipulation code.

**Beginner-Friendly Explanation:** Selecting elements means finding the HTML elements you want to work with. Think of it like searching a house for specific items: you can find something by its unique address (`getElementById`), by a shared label (`getElementsByClassName`), by its type (`getElementsByTagName`), or by a complex description (`querySelector`). Once you've found the element, you can change its text, style, or behaviour.

### Key Characteristics

- **Multiple selection strategies:** ID-based, class-based, tag-based, name-based, and CSS selector-based.
- **Live vs static collections:** `getElementsBy*` methods return live collections that update automatically; `querySelectorAll` returns a static `NodeList`.
- **Performance hierarchy:** ID selection is fastest, followed by class and tag selection; complex CSS selectors are slower.
- **CSS selector power:** `querySelector` and `querySelectorAll` accept any valid CSS selector, including combinators and pseudo-classes.
- **Relational selection:** `closest()` traverses ancestors; `matches()` tests a single element.
- **Caching importance:** Repeatedly querying the DOM for the same elements is inefficient; caching references is a best practice.

### Prerequisites

- Basic understanding of HTML structure and CSS selectors.
- Familiarity with the DOM tree and node types.
- Knowledge of JavaScript variables and functions.

### Related Programming Areas

- **Event handling:** Selecting elements to attach event listeners.
- **Dynamic content:** Creating, updating, and removing elements.
- **Form manipulation:** Reading and setting form field values.
- **CSS-in-JS:** Programmatically applying styles to selected elements.
- **Performance optimisation:** Efficient selection and caching strategies.

### Core Concepts / Features

---

## `getElementById()`

### Definitions

**Core Definition:** `getElementById()` is a method of the `Document` interface that returns the element whose `id` attribute matches the specified string.

**Technical Definition:** Per MDN, "`getElementById()` 方法是 `Document` 接口的方法，它返回一个 `Element` 对象，表示与指定字符串匹配的 `id` 属性的元素". Since element IDs must be unique within a document, this method returns the first matching element if duplicates exist. Unlike `querySelector()`, `getElementById()` can only be called on the global `document` object, not on individual elements.

**Beginner-Friendly Explanation:** `getElementById()` finds an element by its unique ID. If you have `<div id="header">`, you use `document.getElementById("header")` to get it. It's the fastest way to find an element because IDs are unique.

### Purposes

- To retrieve a single element by its unique identifier.
- To provide the fastest possible element selection method.
- To access elements without needing to specify their type or class.

### Syntax Rules and Structure

```js
const element = document.getElementById(id);
```

**Component Breakdown:**
- `id`: A case-sensitive string matching the element's `id` attribute.
- Returns: An `Element` object, or `null` if no match is found.

**Syntax Rules:**
- The method name is case-sensitive: `getElementById` (not `getElementByID` or `getelementbyid`).
- Only available on the `document` object, not on elements.
- ID must be unique within the document for predictable behaviour.

**Constraints and Limitations:**
- Cannot be called on an element to search its subtree.
- Does not search elements not connected to the document.
- Returns `null` if no matching element is found.

### Annotated Code Examples

#### Example 1: Basic Usage

```html
<!-- HTML: <p id="para">Some text here</p> -->
```

```js
// Step 1: Select the element by ID.
const elem = document.getElementById("para");

// Step 2: Modify its style.
elem.style.color = "blue";

// Step 3: Verify the selection.
console.log(elem.tagName);      // "P"
console.log(elem.textContent);  // "Some text here"
```

**Expected Output:**
```
P
Some text here
```

**Why This Output:** `getElementById("para")` finds the `<p>` element with `id="para"`. The element's `tagName` is `"P"` and its `textContent` is `"Some text here"`.

#### Example 2: Handling Null Returns

```js
// Step 1: Try to select a non-existent element.
const missing = document.getElementById("does-not-exist");
console.log(missing); // null

// Step 2: Guard against null.
if (missing) {
  missing.textContent = "Found!";
} else {
  console.log("Element not found");
}
```

**Expected Output:**
```
null
Element not found
```

**Why This Output:** `getElementById` returns `null` when no matching element exists. Always check for `null` before using the result.

### Real-World Cases

- **Form submission:** Getting a form by ID to read its values.
- **Modal dialogs:** Selecting and showing/hiding a modal by ID.
- **Single-page applications:** Accessing the root container by ID.

---

## `getElementsByClassName()`

### Definitions

**Core Definition:** `getElementsByClassName()` returns a live `HTMLCollection` of all elements that have all of the specified class names.

**Technical Definition:** Per MDN, "`getElementsByClassName()` 返回一个实时 `HTMLCollection`，其中包含每个具有指定类名或类名的后代元素". When called on the `Document`, it searches the entire document; when called on an `Element`, it searches only the descendants of that element. The returned collection is live: it updates automatically as elements are added or removed.

**Beginner-Friendly Explanation:** `getElementsByClassName()` finds all elements that have a certain class. If you have several elements with `class="item"`, it returns a collection of all of them. It's like asking "show me everything with this label."

### Purposes

- To select multiple elements that share one or more class names.
- To get a live collection that reflects DOM changes automatically.
- To search within a specific parent element's subtree.

### Syntax Rules and Structure

```js
const collection = document.getElementsByClassName(names);
const collection = element.getElementsByClassName(names);
```

**Component Breakdown:**
- `names`: A string containing one or more class names separated by spaces.
- Returns: A live `HTMLCollection` of matching elements.

**Syntax Rules:**
- Multiple class names are separated by spaces: `"red test"` matches elements with both `red` and `test` classes.
- The returned collection is live and updates automatically.
- In quirks mode, class names are compared case-insensitively; otherwise, case-sensitively.

**Constraints and Limitations:**
- The collection is live, so iterating while modifying classes can cause unexpected behaviour.
- Not a true array; use `Array.from()` or spread syntax to convert.
- Does not support CSS selectors or combinators.

### Annotated Code Examples

#### Example 1: Selecting by Class

```html
<!-- HTML: <div class="item">A</div><div class="item">B</div><div class="item">C</div> -->
```

```js
// Step 1: Select all elements with class "item".
const items = document.getElementsByClassName("item");

// Step 2: Check the collection.
console.log(items.length); // 3

// Step 3: Access individual elements.
console.log(items[0].textContent); // "A"
console.log(items[1].textContent); // "B"
```

**Expected Output:**
```
3
A
B
```

**Why This Output:** `getElementsByClassName("item")` returns a live collection of all three `<div>` elements with class `"item"`.

#### Example 2: Live Collection Behaviour

```js
// Step 1: Get the live collection.
const items = document.getElementsByClassName("item");
console.log(items.length); // 3

// Step 2: Add a new element with the class.
const newItem = document.createElement("div");
newItem.className = "item";
newItem.textContent = "D";
document.body.appendChild(newItem);

// Step 3: The collection updates automatically.
console.log(items.length); // 4 (live!)
```

**Expected Output:**
```
3
4
```

**Why This Output:** The `HTMLCollection` is live. Adding a new element with class `"item"` automatically updates the collection's length.

### Real-World Cases

- **List manipulation:** Selecting all list items to apply styles or event handlers.
- **Form validation:** Getting all inputs with an error class.
- **Bulk updates:** Applying changes to all elements with a specific class.

---

## `getElementsByTagName()`

### Definitions

**Core Definition:** `getElementsByTagName()` returns a live `HTMLCollection` of elements with the specified tag name.

**Technical Definition:** Per MDN, "`getElementsByTagName()` 返回提供されたタグ名を持つ要素の有効な `HTMLCollection` を返します". When called on a specific element, it searches only the descendants of that element, excluding the element itself. When called on an HTML document, the tag name argument is converted to lowercase before matching. The special string `"*"` represents all elements.

**Beginner-Friendly Explanation:** `getElementsByTagName()` finds all elements of a certain type, like all `<p>` tags or all `<div>` tags. It's like asking "show me everything that is a paragraph."

### Purposes

- To select all elements of a specific tag type.
- To get a live collection of elements by tag name.
- To search within a specific element's subtree by tag name.

### Syntax Rules and Structure

```js
const collection = document.getElementsByTagName(tagName);
const collection = element.getElementsByTagName(tagName);
```

**Component Breakdown:**
- `tagName`: The tag name to search for. `"*"` matches all elements.
- Returns: A live `HTMLCollection` of matching elements in document order.

**Syntax Rules:**
- In HTML documents, the tag name is lowercased before matching.
- For SVG elements in HTML subtrees, use `getElementsByTagNameNS()` instead.
- The returned collection is live.

**Constraints and Limitations:**
- Does not support CSS selectors.
- The collection is live, with the same iteration caveats as `getElementsByClassName`.

### Annotated Code Examples

#### Example 1: Selecting by Tag Name

```html
<!-- HTML: <p>One</p><p>Two</p><div>Three</div> -->
```

```js
// Step 1: Select all paragraphs.
const paragraphs = document.getElementsByTagName("p");
console.log(paragraphs.length); // 2

// Step 2: Select all elements.
const allElements = document.getElementsByTagName("*");
console.log(allElements.length); // 3 (plus html, head, body, etc.)
```

**Expected Output:**
```
2
3
```

**Why This Output:** `getElementsByTagName("p")` returns the two `<p>` elements. `getElementsByTagName("*")` returns all elements in the document.

#### Example 2: Scoped Search

```js
// HTML: <div id="container"><p>A</p><p>B</p></div><p>C</p>
const container = document.getElementById("container");
const paragraphs = container.getElementsByTagName("p");

console.log(paragraphs.length); // 2 (only A and B, not C)
```

**Expected Output:**
```
2
```

**Why This Output:** When called on an element, `getElementsByTagName` searches only that element's descendants, not the entire document.

### Real-World Cases

- **Table processing:** Getting all `<td>` cells in a table.
- **Link handling:** Selecting all `<a>` elements to attach click handlers.
- **Image optimisation:** Selecting all `<img>` elements for lazy loading.

---

## `getElementsByName()`

### Definitions

**Core Definition:** `getElementsByName()` returns a live `NodeList` of elements that have a `name` attribute matching the specified string.

**Technical Definition:** Per MDN, "`getElementsByName()` 方法是 `Document` 对象的一个方法，它返回一个 `NodeList` 集合，包含文档中具有给定 `name` 属性的元素". The returned collection is a live `NodeList` that updates automatically when elements with the same name are added or removed. This method is only available on the `Document` object, not on individual elements.

**Beginner-Friendly Explanation:** `getElementsByName()` finds elements by their `name` attribute, which is commonly used for form elements like radio buttons and checkboxes. It's like asking "show me everything with this name tag."

### Purposes

- To select form elements that share a `name` attribute (e.g., radio button groups).
- To get a live collection of elements by name.
- To access elements that may not have a unique ID.

### Syntax Rules and Structure

```js
const collection = document.getElementsByName(name);
```

**Component Breakdown:**
- `name`: The value of the `name` attribute to search for.
- Returns: A live `NodeList` of matching elements.

**Syntax Rules:**
- Only available on the `document` object.
- Returns a `NodeList` (not an `HTMLCollection`).
- The collection is live.

**Constraints and Limitations:**
- `name` attributes are not required to be unique, so multiple elements may match.
- Does not support CSS selectors.
- Less commonly used in modern development (except for form handling).

### Annotated Code Examples

#### Example 1: Selecting Form Elements by Name

```html
<!-- HTML:
<form>
  <input type="radio" name="gender" value="male"> Male
  <input type="radio" name="gender" value="female"> Female
</form>
-->
```

```js
// Step 1: Select all elements with name="gender".
const genders = document.getElementsByName("gender");
console.log(genders.length); // 2

// Step 2: Iterate over the collection.
genders.forEach((input) => {
  console.log(input.value);
});
```

**Expected Output:**
```
2
male
female
```

**Why This Output:** `getElementsByName("gender")` returns a live `NodeList` of both radio inputs with `name="gender"`.

### Real-World Cases

- **Form handling:** Getting all radio buttons in a group to check which is selected.
- **Checkbox groups:** Selecting multiple checkboxes with the same name.
- **Legacy form processing:** Reading form data by element name.

---

## `querySelector()`

### Definitions

**Core Definition:** `querySelector()` returns the first element within the document or an element's subtree that matches the specified CSS selector.

**Technical Definition:** Per MDN, "`querySelector()` 返回节点子树中第一个匹配的 `Element` 节点。如果没有找到匹配的节点，则返回 `null`". The method accepts any valid CSS selector, including combinators, attribute selectors, and pseudo-classes. It is available on `Document`, `DocumentFragment`, and `Element` interfaces.

**Beginner-Friendly Explanation:** `querySelector()` finds the first element that matches a CSS selector. It's like using a search query: "find the first button with class 'submit' inside the header."

### Purposes

- To select a single element using any CSS selector.
- To search for elements by complex criteria (attributes, combinators, pseudo-classes).
- To scope searches to a specific parent element's subtree.

### Syntax Rules and Structure

```js
const element = document.querySelector(selectors);
const element = element.querySelector(selectors);
```

**Component Breakdown:**
- `selectors`: A string containing one or more CSS selectors.
- Returns: The first matching `Element`, or `null` if no match is found.

**Syntax Rules:**
- Accepts any valid CSS selector, including selector lists (comma-separated).
- Only elements are returned; pseudo-elements are not supported.
- Some pseudo-classes (like `:visited`) return no matches for privacy reasons.

**Constraints and Limitations:**
- Returns only the first match; use `querySelectorAll` for all matches.
- Less performant than `getElementById` for simple ID lookups.
- Does not return text or comment nodes.

### Annotated Code Examples

#### Example 1: Basic Selector Queries

```js
// Step 1: Select by ID.
const header = document.querySelector("#header");

// Step 2: Select by class.
const firstItem = document.querySelector(".item");

// Step 3: Select by attribute.
const emailInput = document.querySelector("input[type='email']");

// Step 4: Select by complex selector.
const firstLink = document.querySelector("nav ul li:first-child a");
```

**Expected Output:** Each query returns the first matching element, or `null` if not found.

**Why This Output:** `querySelector` accepts any CSS selector, providing maximum flexibility for element selection.

#### Example 2: Scoped Selection

```js
// HTML: <div id="main"><button class="btn">Click</button></div>
const main = document.getElementById("main");
const button = main.querySelector(".btn");

console.log(button.textContent); // "Click"
```

**Expected Output:**
```
Click
```

**Why This Output:** When called on an element, `querySelector` searches only that element's descendants.

### Real-World Cases

- **Form validation:** Selecting inputs by type or name.
- **Navigation:** Finding the active link in a menu.
- **Component scoping:** Selecting elements within a specific component's root.

---

## `querySelectorAll()`

### Definitions

**Core Definition:** `querySelectorAll()` returns a static `NodeList` of all elements that match the specified CSS selector.

**Technical Definition:** Per MDN, "`querySelectorAll()` 返回一个包含节点子树中所有匹配的 `Element` 节点的 `NodeList`，如果没有找到匹配项，则返回一个空的 `NodeList`". The returned `NodeList` is **not live**: DOM changes do not affect it after creation. It accepts any valid CSS selector, including selector lists.

**Beginner-Friendly Explanation:** `querySelectorAll()` finds all elements that match a CSS selector. Unlike `getElementsByClassName`, the returned list is a snapshot — it won't change if the DOM changes later.

### Purposes

- To select all elements matching a CSS selector.
- To get a stable, non-live collection for iteration.
- To support complex selection criteria (combinators, attributes, pseudo-classes).

### Syntax Rules and Structure

```js
const nodeList = document.querySelectorAll(selectors);
const nodeList = element.querySelectorAll(selectors);
```

**Component Breakdown:**
- `selectors`: A string containing one or more CSS selectors.
- Returns: A static `NodeList` of all matching elements.

**Syntax Rules:**
- Accepts multiple selectors: `"p.warning, p.note"`.
- Returns a static `NodeList` (not live).
- Can be iterated with `forEach`, `for...of`, or converted to an array.

**Constraints and Limitations:**
- The `NodeList` is static; DOM changes after the call are not reflected.
- Does not return `null`; returns an empty `NodeList` if no matches.
- Less performant than `getElementsByClassName` for simple class queries.

### Annotated Code Examples

#### Example 1: Selecting Multiple Elements

```js
// Step 1: Select all paragraphs with class "warning" or "note".
const special = document.querySelectorAll("p.warning, p.note");
console.log(special.length); // Number of matching elements

// Step 2: Iterate over the NodeList.
special.forEach((el) => {
  console.log(el.textContent);
});
```

**Expected Output:** The text content of each matching paragraph is logged.

**Why This Output:** `querySelectorAll` accepts selector lists, allowing multiple selectors in one query.

#### Example 2: Static vs Live Collection

```js
// Step 1: Get a static NodeList.
const staticList = document.querySelectorAll(".item");
console.log(staticList.length); // 3

// Step 2: Add a new element with the class.
const newItem = document.createElement("div");
newItem.className = "item";
document.body.appendChild(newItem);

// Step 3: The static list does NOT update.
console.log(staticList.length); // 3 (still!)
```

**Expected Output:**
```
3
3
```

**Why This Output:** `querySelectorAll` returns a static `NodeList`. Adding a new element does not change the existing list.

### Real-World Cases

- **Bulk event binding:** Attaching listeners to all buttons in a container.
- **Styling:** Applying a class to all elements matching a selector.
- **Data extraction:** Collecting text from all matching elements.

---

## `closest()`

### Definitions

**Core Definition:** `closest()` traverses the element and its ancestors (heading toward the document root) until it finds a node that matches the specified CSS selector.

**Technical Definition:** Per MDN, "`closest()` 方法属于 `Element` 接口，它会遍历当前元素及其父元素（朝向文档根方向），直到找到一个匹配指定 CSS 选择器的节点". It returns the closest matching ancestor (which may be the element itself) or `null` if no match is found.

**Beginner-Friendly Explanation:** `closest()` finds the nearest ancestor element that matches a selector. It starts from the current element and walks up the tree until it finds a match. It's like asking "which of my parents or grandparents matches this description?"

### Purposes

- To find the nearest ancestor element matching a selector.
- To implement event delegation by finding the responsible parent.
- To navigate upward from a deeply nested element to a known container.

### Syntax Rules and Structure

```js
const element = element.closest(selectors);
```

**Component Breakdown:**
- `selectors`: A string containing one or more CSS selectors.
- Returns: The closest matching `Element` (including the element itself), or `null`.

**Syntax Rules:**
- The element itself is included in the search.
- Traverses only ancestors (parent, grandparent, etc.).
- Returns `null` if no matching ancestor is found.

**Constraints and Limitations:**
- Does not traverse siblings or descendants.
- In Edge 15–18, if the element is not connected to the context object, it returns `null`.

### Annotated Code Examples

#### Example 1: Finding the Nearest Container

```html
<!-- HTML: <div class="card"><button class="btn">Click</button></div> -->
```

```js
// Step 1: Select the button.
const button = document.querySelector(".btn");

// Step 2: Find the nearest ancestor with class "card".
const card = button.closest(".card");
console.log(card.className); // "card"

// Step 3: Find the nearest ancestor with class "container" (doesn't exist).
const container = button.closest(".container");
console.log(container); // null
```

**Expected Output:**
```
card
null
```

**Why This Output:** `closest(".card")` traverses from the button up to the `<div class="card">`. `closest(".container")` finds no match and returns `null`.

#### Example 2: Event Delegation with `closest()`

```js
// Step 1: Attach a single click listener to the list.
document.querySelector("ul").addEventListener("click", (event) => {
  // Step 2: Find the nearest <li> ancestor of the clicked element.
  const li = event.target.closest("li");
  if (li) {
    console.log("Clicked item:", li.textContent);
  }
});
```

**Expected Output:** Clicking any `<li>` or its children logs the `<li>`'s text content.

**Why This Output:** `closest("li")` finds the nearest `<li>` ancestor of the clicked element, enabling efficient event delegation.

### Real-World Cases

- **Event delegation:** Finding the list item that was clicked.
- **Form navigation:** Finding the containing form of an input.
- **Component boundaries:** Finding the root element of a component from a child.

---

## `matches()`

### Definitions

**Core Definition:** `matches()` tests whether an element would be selected by a specified CSS selector, returning `true` or `false`.

**Technical Definition:** Per MDN, "`matches()` 返回 `true` 如果 `Element` が `selectors` に一致すれば `true` です。そうでなければ `false` です". It does not traverse the tree; it only tests the element itself against the selector.

**Beginner-Friendly Explanation:** `matches()` checks if an element matches a selector. It's like asking "does this element qualify as a button?" and getting a yes or no answer.

### Purposes

- To test a single element against a CSS selector.
- To filter elements based on their attributes or classes.
- To implement conditional logic based on element state.

### Syntax Rules and Structure

```js
const result = element.matches(selectors);
```

**Component Breakdown:**
- `selectors`: A string containing one or more CSS selectors.
- Returns: `true` if the element matches, `false` otherwise.

**Syntax Rules:**
- The element itself is tested, not its ancestors or descendants.
- Accepts any valid CSS selector.
- Some browsers implement this as `matchesSelector()` with vendor prefixes.

**Constraints and Limitations:**
- Does not traverse the tree; only tests the element itself.
- Returns a boolean, not an element.

### Annotated Code Examples

#### Example 1: Testing Element Matches

```js
// Step 1: Select an element.
const el = document.querySelector(".btn");

// Step 2: Test against selectors.
console.log(el.matches(".btn"));         // true
console.log(el.matches("button"));       // true (if it's a button)
console.log(el.matches(".container"));   // false
console.log(el.matches("div > button")); // true if parent is a div
```

**Expected Output:**
```
true
true
false
true
```

**Why This Output:** `matches()` tests whether the element would be selected by the given CSS selector.

#### Example 2: Filtering Elements

```js
// Step 1: Get all elements.
const allElements = document.querySelectorAll("*");

// Step 2: Filter for elements matching a selector.
const buttons = Array.from(allElements).filter((el) =>
  el.matches("button, [role='button']")
);
console.log(buttons.length); // Number of buttons
```

**Expected Output:** The count of elements that are buttons or have `role="button"`.

**Why This Output:** `matches()` provides a boolean result that can be used with `Array.prototype.filter` to select elements by selector.

### Real-World Cases

- **Conditional styling:** Checking if an element has a specific class before applying changes.
- **Form validation:** Testing if an input matches a required pattern.
- **Event handling:** Checking if the event target matches a selector.

---

## Selection Performance

### Definitions

**Core Definition:** Selection performance refers to the relative speed and efficiency of different DOM element selection methods, which depends on the method used, the complexity of the selector, and the size of the DOM.

**Technical Definition:** ID-based selection (`getElementById`) is the fastest because browsers maintain a direct index of IDs. Class and tag selection (`getElementsByClassName`, `getElementsByTagName`) are also optimised but slower than ID lookup. CSS selector-based methods (`querySelector`, `querySelectorAll`) are the most flexible but have parsing and matching overhead, especially for complex selectors. A "flatter" DOM improves selector performance because the selector engine has fewer layers to traverse.

**Beginner-Friendly Explanation:** Not all ways of finding elements are equally fast. Finding by ID is like looking up a word in a dictionary — very fast. Finding by complex CSS selector is like searching the whole dictionary for a phrase — slower.

### Performance Hierarchy

| Method | Speed | Notes |
|--------|-------|-------|
| `getElementById()` | Fastest | Direct index lookup |
| `getElementsByClassName()` | Fast | Optimised native method |
| `getElementsByTagName()` | Fast | Optimised native method |
| `querySelector()` | Moderate | CSS selector parsing overhead |
| `querySelectorAll()` | Moderate | CSS selector parsing + full traversal |

### Purposes

- To choose the most efficient selection method for a given task.
- To understand the performance implications of complex selectors.
- To optimise DOM querying in performance-critical applications.

### Syntax Rules and Structure

Performance considerations are not syntax rules but best practices:

```js
// FAST: ID-based selection.
const header = document.getElementById("header");

// FAST: Class-based selection.
const items = document.getElementsByClassName("item");

// SLOWER: Complex CSS selector.
const firstItem = document.querySelector("nav ul li:first-child a.active");

// SLOWER: Universal selector.
const all = document.querySelectorAll("*");
```

**Component Breakdown:**
- **ID selectors:** Start with an ID for best performance.
- **Specificity on the right:** Be specific on the right-hand side of the selector, less specific on the left.
- **Avoid universal selector:** `*` is the slowest selector.

**Syntax Rules:**
- Prefer ID-based selection when possible.
- Avoid overly complex selectors.
- Use context parameters to narrow the search scope.

**Constraints and Limitations:**
- Performance differences are negligible for small DOMs.
- Modern browsers optimise many selectors, but the hierarchy generally holds.

### Annotated Code Examples

#### Example 1: Selecting Within a Container

```js
// SLOW: Searches the entire document.
const items = document.querySelectorAll(".item");

// FAST: Searches only within the container.
const container = document.getElementById("container");
const containerItems = container.querySelectorAll(".item");
```

**Expected Output:** Both return the same items (if all `.item` elements are inside `#container`), but the scoped version is faster because it searches a smaller subtree.

**Why This Output:** Scoping the search to a parent element reduces the number of nodes the selector engine must traverse.

### Real-World Cases

- **Large lists:** Using `getElementById` to get a container, then `querySelectorAll` within it.
- **Animation loops:** Caching element references instead of re-querying every frame.
- **Form handling:** Using `getElementsByName` for form fields instead of complex selectors.

---

## Caching Selections

### Definitions

**Core Definition:** Caching selections is the practice of storing references to selected DOM elements in variables instead of re-querying the DOM each time the element is needed.

**Technical Definition:** Per the caching best practices, "Cache element references at initialization". Repeated `querySelectorAll()` calls traverse the DOM tree each time, which is inefficient when called frequently or when the elements don't change. A DOM cache object can store frequently accessed elements and provide fallback patterns.

**Beginner-Friendly Explanation:** Instead of searching for the same element over and over, find it once and save it in a variable. It's like writing down a phone number instead of looking it up in the phone book every time you want to call.

### Purposes

- To improve performance by avoiding repeated DOM traversal.
- To reduce code duplication.
- To make code more maintainable by centralising element references.
- To avoid re-querying the DOM on every event handler invocation.

### Syntax Rules and Structure

```js
// WITHOUT caching (inefficient):
function updateUI() {
  document.querySelector(".status").textContent = "Updated";
  document.querySelector(".status").classList.add("active");
}

// WITH caching (efficient):
const elements = {
  status: document.querySelector(".status"),
};

function updateUI() {
  elements.status.textContent = "Updated";
  elements.status.classList.add("active");
}
```

**Component Breakdown:**
- **Cache object:** An object that stores element references by name.
- **Initialisation:** Populate the cache once, typically on page load or component mount.
- **Usage:** Access cached elements via the cache object.

**Syntax Rules:**
- Cache elements that are queried more than once.
- Re-cache if the DOM is dynamically replaced.
- Use a fallback pattern for elements that may not exist at initialisation.

**Constraints and Limitations:**
- Cached references become stale if the element is removed from the DOM.
- Dynamic content requires re-caching.
- Over-caching can increase memory usage.

### Annotated Code Examples

#### Example 1: Basic Caching Pattern

```js
// Step 1: Cache elements at initialisation.
const DOMCache = {
  header: document.getElementById("header"),
  navItems: document.querySelectorAll(".nav-item"),
  statusMessage: document.querySelector(".status-message"),
};

// Step 2: Use cached references.
function updateStatus(message) {
  DOMCache.statusMessage.textContent = message;
}

function highlightNav() {
  DOMCache.navItems.forEach((item) => item.classList.add("active"));
}
```

**Expected Output:** The functions access elements from the cache, avoiding repeated DOM queries.

**Why This Output:** The cache object stores element references, so each function accesses the stored reference instead of re-querying the DOM.

#### Example 2: Cache with Fallback Pattern

```js
// Step 1: Define the cache with null initial values.
const DOMCache = {
  authModal: null,
  prayerCards: null,
};

// Step 2: Initialise the cache.
function initDOMCache() {
  DOMCache.authModal = document.getElementById("authModal");
  DOMCache.prayerCards = document.querySelectorAll(".prayer-card");
}

// Step 3: Use with fallback.
function getModal() {
  return DOMCache.authModal || document.getElementById("authModal");
}
```

**Expected Output:** The cache provides fast access when initialised, with a fallback query when needed.

**Why This Output:** The fallback pattern ensures the element is found even if the cache wasn't initialised or the element was dynamically added.

### Real-World Cases

- **Single-page applications:** Caching the root container and frequently used UI elements.
- **Game development:** Caching canvas and control elements for fast access during game loops.
- **Data dashboards:** Caching chart containers and data displays for efficient updates.

---

## References

- MDN Web Docs – Document: getElementById() — https://mdn.org.cn/en-US/docs/Web/API/Document/getElementById
- MDN Web Docs – Element: getElementsByClassName() — https://mdn.org.cn/en-US/docs/Web/API/Element/getElementsByClassName
- MDN Web Docs – Element.getElementsByTagName() — https://developer.mozilla.org/ko/docs/Web/API/Element/getElementsByTagName
- MDN Web Docs – Document: getElementsByName() — https://mdn.org.cn/en-US/docs/Web/API/Document/getElementsByName
- MDN Web Docs – Selection and traversal on the DOM tree — https://mdn.org.cn/en-US/docs/Web/API/Document_Object_Model/Selection_and_traversal_on_the_DOM_tree
- MDN Web Docs – Element: closest() — https://mdn.org.cn/en-US/docs/Web/API/Element/closest
- MDN Web Docs – Element: matches() — https://developer.mozilla.org/fr/docs/Web/API/Element/matches
- MDN Web Docs – Locating DOM elements using selectors — https://mdn.org.cn/en-US/docs/Web/API/Document_Object_Model/Locating_DOM_elements_using_selectors
- Chrome for Developers – CSS Selector Costs — https://chrome-dot-devsite-v2-prod-3p.appspot.com
- freeCodeCamp – JS DOM Manipulation Best Practices — https://www.freecodecamp.org/news/javascript-dom-manipulation-best-practices
- GitHub – Cache DOM element references instead of repeated queries — https://github.com/ammar412/salah-buddy/issues/24
- LoginRadius – Best Practices for jQuery and Sizzle — https://www.loginradius.com
- Tencent Cloud – How to optimize the performance of jQuery code — https://www.tencentcloud.com
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf