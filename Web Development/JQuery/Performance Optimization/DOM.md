# jQuery DOM Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery DOM performance is the study and practice of minimising the computational cost of reading from, writing to, traversing, inserting into, removing from, and cloning the Document Object Model when using jQuery, with the goal of reducing reflows, repaints, and memory consumption while maintaining responsive user interfaces.

**Technical Definition**
DOM performance in jQuery is governed by the browser's rendering pipeline. Every DOM mutation — insertion, removal, attribute change, style update — can trigger **reflow** (layout recalculation) and **repaint** (pixel redrawing). jQuery provides abstractions over native DOM APIs, but these abstractions do not eliminate the underlying cost; in some cases (`.remove()` vs. `removeChild()`, `.clone()` vs. `cloneNode()`) jQuery adds measurable overhead for cleanup and normalisation. Performance is therefore achieved through **batching** (DocumentFragment, HTML string construction), **detaching** (`.detach()` to work off‑document), **caching** (storing jQuery objects in variables), **traversal minimisation** (avoiding repeated ancestor walks), and **lifecycle management** (using jQuery removal methods that call `cleanData` to prevent memory leaks).

**Beginner‑Friendly Explanation**
The DOM is the browser's live representation of your page. Every time you change it — add an element, remove one, move something around — the browser has to recalculate the layout and redraw the screen. Doing this hundreds of times in a loop is slow. jQuery DOM performance is about doing the same work in fewer, larger steps: building everything in memory first, detaching a section before rearranging it, searching for elements once and remembering the result, and cleaning up properly when elements are removed. These habits keep pages fast and smooth, especially on mobile devices.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Reflow and repaint are the primary costs** | Every DOM mutation can trigger layout recalculation and pixel redrawing; batching and detaching minimise these. |
| **jQuery adds convenience, not zero‑cost abstraction** | `.remove()` is slower than `removeChild()` because it cleans data and events; `.clone()` is slower than `cloneNode()` for similar reasons. |
| **DocumentFragment is the batch‑insertion tool** | jQuery uses fragments internally for `.append()`; developers can also build fragments or HTML strings explicitly. |
| **Detaching preserves data and events** | `.detach()` removes an element while keeping its jQuery data and handlers intact, enabling off‑document manipulation. |
| **Caching eliminates redundant traversal** | Repeated selections and repeated ancestor walks are avoidable costs. |
| **Cleanup prevents memory leaks** | jQuery removal methods call `cleanData`; native removal bypasses this, leaving orphaned handlers and data. |

---

### Prerequisites

- Working knowledge of jQuery selection, traversal, and manipulation methods.
- Understanding of the browser rendering pipeline (DOM, CSSOM, reflow, repaint).
- Familiarity with native DOM APIs (`appendChild`, `removeChild`, `cloneNode`, `DocumentFragment`).
- Awareness of jQuery's data and event storage model (`jQuery.cache`, `cleanData`).

---

### Related Programming Areas

- **Browser rendering** — reflow, repaint, compositing, and the critical rendering path.
- **Event handling** — delegation and handler lifecycle interact with DOM insertion and removal.
- **Memory management** — orphaned handlers and detached DOM subtrees cause leaks.
- **Animation** — DOM changes during animation can cause jank.
- **Selector performance** — the entry point for all DOM manipulation.

---

### Core Concepts / Features (Index)

1. DOM Manipulation
2. DOM Traversal
3. DOM Insertion
4. DOM Removal
5. DOM Cloning
6. Document Fragments
7. Detached DOM
8. DOM Caching
9. DOM Best Practices


## Core Concept 1: DOM Manipulation

### Definitions

**Core Definition**
DOM manipulation is the act of changing the structure, content, attributes, or styles of DOM elements — adding, removing, moving, or altering nodes — and is the most expensive category of front‑end operation because each change can force the browser to recalculate layout and repaint the screen.

**Technical Definition**
Every DOM manipulation triggers, at minimum, a **DOM mutation**. Depending on the change, the browser may need to perform **style recalculation**, **layout (reflow)** , and **paint**. Reading layout properties (e.g., `offsetHeight`, `scrollTop`, `getComputedStyle`) after a mutation forces a **synchronous reflow**, which is particularly costly. jQuery's manipulation methods (`.append()`, `.html()`, `.css()`, `.attr()`) ultimately call native DOM APIs, but they add convenience and cross‑browser normalisation. Performance is achieved by reducing the number of mutations, batching writes, separating reads from writes, and performing heavy work off‑document.

**Beginner‑Friendly Explanation**
DOM manipulation is everything you do to change the page: adding a new row to a table, removing a message, changing a button's color. Each change is like telling the browser “stop, recalculate everything, and redraw.” Doing this once is fine; doing it in a loop hundreds of times is slow. The solution is to make all your changes in one batch — build the new content in memory, then insert it once.

---

### Purposes

- **To update page content dynamically** in response to user actions or data changes.
- **To minimise reflow and repaint** by batching mutations and separating reads from writes.
- **To avoid layout thrashing** caused by alternating reads and writes in loops.
- **To perform heavy manipulation off‑document** using `.detach()` before re‑insertion.
- **To keep the main thread responsive** by reducing the number of expensive DOM operations.

---

### Syntax Rules and Structure

#### Common Manipulation Methods

| Method | Purpose |
|---|---|
| `.append()` / `.prepend()` | Insert content inside an element. |
| `.after()` / `.before()` | Insert content adjacent to an element. |
| `.html()` / `.text()` | Replace inner content. |
| `.css()` | Change style properties. |
| `.attr()` / `.prop()` | Change attributes or properties. |
| `.remove()` / `.detach()` / `.empty()` | Remove elements or content. |

#### Batch Manipulation Pattern

```
var html = "";
for ( … ) { html += "<li>…</li>"; }
$container.html(html);       // one mutation
```

---

### Syntax Rules

1. **Batch DOM writes**: Build HTML strings or DocumentFragments before inserting.
2. **Separate reads from writes**: Do not read layout properties inside a loop that also writes.
3. **Detach before heavy work**: Use `.detach()` when many changes are needed inside a container.
4. **Cache selections**: Store the jQuery object in a variable instead of re‑selecting.
5. **Use jQuery removal methods** that clean data and events.

---

### Constraints and Limitations

- **DOM manipulation is inherently slow**; no abstraction can make it free.
- **jQuery's convenience methods add overhead** compared with native APIs in some cases (e.g., `.remove()` vs. `removeChild()`).
- **Layout thrashing** occurs when reads and writes are interleaved; even batching writes will not help if reads force synchronous reflows.

---

### Annotated Code Examples

#### Example 1: Anti‑Pattern vs. Batched Manipulation

```javascript
// ❌ ANTI-PATTERN: per-item DOM writes in a loop
for (var i = 0; i < 500; i++) {
  $("#list").append("<li>Item " + i + "</li>"); // 500 reflows potentially
}

// ✅ BATCHED: build HTML string, insert once
var html = "";
for (var i = 0; i < 500; i++) {
  html += "<li>Item " + i + "</li>";
}
$("#list").html(html); // 1 DOM mutation
```

**Expected Output**
Both versions render 500 list items, but the batched version performs one DOM insertion instead of 500, dramatically reducing reflow and repaint work.

**Why this output occurs**: Building the HTML string defers all DOM work to a single `.html()` call, consistent with the “Append Outside of Loops” guidance.

---

#### Example 2: Separating Reads and Writes

```javascript
// ❌ LAYOUT THRASHING: read after every write
var total = 0;
$(".item").each(function () {
  $(this).css("width", "100px");       // write
  total += this.offsetHeight;          // read — forces reflow
});

// ✅ SEPARATED: batch writes, then read once
$(".item").css("width", "100px");       // all writes
var total = 0;
$(".item").each(function () {
  total += this.offsetHeight;          // reads after writes
});
```

**Expected Output**
Both calculate the total height after setting widths, but the second version performs one style flush instead of one per item.

**Why this output occurs**: Alternating reads and writes forces the browser to recalculate layout repeatedly. Separating them allows the browser to batch style and layout work.

---

### Real‑World Cases

- **Data tables**: Building rows as an HTML string and inserting once.
- **Chat applications**: Appending a batch of new messages in a single operation.
- **Form initialisation**: Detaching a form section, populating fields, and re‑attaching.

---

## Core Concept 2: DOM Traversal

### Definitions

**Core Definition**
DOM traversal is the act of navigating the DOM tree from a selected element to its ancestors, descendants, or siblings using jQuery traversal methods (`.find()`, `.children()`, `.parent()`, `.siblings()`, etc.), and is a performance concern because each traversal step requires the browser or jQuery to inspect additional nodes.

**Technical Definition**
jQuery traversal methods return new jQuery objects, each of which represents a set of nodes reached by walking the tree. Traversal cost depends on the **depth** of the walk, the **breadth** of the set being traversed, and whether the method can use native DOM properties (`firstChild`, `nextSibling`, `children`) or must filter a larger candidate set. `.find()` delegates to the selector engine and can leverage `querySelectorAll` for CSS‑compatible selectors; `.children()` historically performed slower than `.find()` in some jQuery versions due to implementation differences, though this has been optimised. Caching traversal results and avoiding repeated ancestor walks are the primary performance techniques.

**Beginner‑Friendly Explanation**
DOM traversal is like walking through a family tree: moving from a parent to its children, from a child to its siblings, or from a node up to its ancestors. Each step takes time, especially if the tree is large or you repeat the walk many times. If you need the same group of relatives more than once, it is faster to write down who they are (cache the result) than to walk the tree again.

---

### Purposes

- **To navigate from a known element to related elements** without re‑selecting from the document root.
- **To minimise traversal depth** by choosing the closest appropriate starting point.
- **To cache traversal results** when the same relative set is needed repeatedly.
- **To avoid unnecessary ancestor walks** by using ID anchors or context.
- **To choose traversal methods that align with native DOM capabilities**.

---

### Syntax Rules and Structure

| Method | Direction | Performance Consideration |
|---|---|---|
| `.find(selector)` | Descendants | Uses selector engine; can leverage QSA. |
| `.children([selector])` | Direct children | Historically slower than `.find()` in some versions. |
| `.parent()` / `.parents()` | Ancestors | `.parents()` walks the full ancestor chain; limit with a selector. |
| `.siblings()` | Siblings | May inspect many nodes. |
| `.next()` / `.prev()` | Adjacent siblings | Fast (native `nextSibling`/`previousSibling`). |
| `.closest(selector)` | Nearest ancestor match | Stops at first match; efficient. |

---

### Constraints and Limitations

- **`.parents()` without a selector** walks the entire ancestor chain to the document root, which is expensive.
- **`.children()` has historically been slower than `.find()`** in some jQuery versions, though modern implementations have improved.
- **Repeated traversal without caching** multiplies cost.

---

### Annotated Code Example

```javascript
// ❌ REPEATED TRAVERSAL: ancestor walk on every iteration
$(".item").each(function () {
  var $parent = $(this).parent(); // walk up every time
  $parent.addClass("active");
});

// ✅ CACHED TRAVERSAL: walk once, reuse
var $list = $("#list");             // known parent
$list.children(".item").each(function () {
  // no repeated ancestor walk
});
$list.addClass("active");          // operate on the cached parent
```

**Expected Output**
Both versions add the `active` class to the parent, but the second avoids a parent lookup per item.

**Why this output occurs**: The second version starts from a cached parent and uses `.children()` to reach the items, eliminating repeated `.parent()` walks.

---

### Real‑World Cases

- **Tree menus**: Using `.children()` and `.find()` within a cached root instead of global selections.
- **Card components**: Using `.closest(".card")` to find the containing card from a clicked button.
- **List operations**: Caching the list root and using `.children()` for item iteration.

---

## Core Concept 3: DOM Insertion

### Definitions

**Core Definition**
DOM insertion is the act of adding new nodes to the document, either by appending, prepending, or inserting relative to existing elements. It is the most reflow‑sensitive manipulation category because each insertion can change layout geometry.

**Technical Definition**
jQuery provides `.append()`, `.prepend()`, `.after()`, `.before()`, `.appendTo()`, `.prependTo()`, `.insertAfter()`, and `.insertBefore()`. Internally, jQuery's `.append()` and related methods use `document.createDocumentFragment` when inserting multiple nodes from an HTML string, and may cache fragments for strings of 512 bytes or less. Despite this internal optimisation, inserting elements one at a time in a loop still causes repeated DOM touches. The recommended patterns are: build an **HTML string** and set it via `.html()`, build a **DocumentFragment** and append it once, or **detach** the target, insert all content off‑document, and re‑attach.

**Beginner‑Friendly Explanation**
DOM insertion is putting new things on the page. Doing it one item at a time in a loop is like carrying groceries into the house one bag at a time — it works, but it is slow. It is faster to load everything into a cart (a DocumentFragment or HTML string) and bring it in once. jQuery's `.append()` does some of this internally, but the biggest gains come from batching your own insertions.

---

### Purposes

- **To add new content to the page** in response to user actions or data loading.
- **To batch insertions** and reduce the number of reflows.
- **To use DocumentFragment or HTML strings** for bulk insertion.
- **To detach and re‑attach containers** when inserting large amounts of content.
- **To avoid repeated `.append()` calls inside loops**.

---

### Syntax Rules and Structure

| Pattern | Example | Performance |
|---|---|---|
| Per‑item append | `$list.append("<li>…</li>")` in loop | ❌ Many DOM touches |
| HTML string | `$list.html(htmlString)` | ✅ One DOM touch |
| DocumentFragment | `frag.appendChild(node); $list.append(frag)` | ✅ One DOM touch |
| Detach + insert + re‑attach | `$list.detach(); …; $parent.append($list)` | ✅ Off‑document work |

---

### Constraints and Limitations

- **`.html()` replaces all existing content**; use `.append()` with an HTML string if existing content must be preserved.
- **DocumentFragment is a native DOM object**; jQuery can wrap it, but the fragment itself is not a jQuery object.
- **`.append()` with an HTML string still parses the HTML**, which has a cost; for very large content, DocumentFragment may be preferable.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul id="list"></ul>

  <script>
    // ✅ BATCH INSERTION with DocumentFragment
    var $list = $("#list");
    var frag = document.createDocumentFragment();

    for (var i = 1; i <= 500; i++) {
      var li = document.createElement("li");
      li.textContent = "Item " + i;
      frag.appendChild(li);          // off-document
    }

    $list.append(frag);              // one DOM insertion
    console.log("Items:", $list.children().length);
  </script>
</body>
</html>
```

**Expected Output**
```
Items: 500
```

**Why this output occurs**: The DocumentFragment accumulates all 500 `<li>` elements off‑document. Appending the fragment to `#list` performs a single DOM operation, consistent with the “batch operations” guidance.

---

### Real‑World Cases

- **Large lists**: Building a DocumentFragment for search results or feed items.
- **Table population**: Constructing rows in a fragment and appending to `<tbody>`.
- **Dashboard widgets**: Assembling widget HTML in memory and inserting once.

---

## Core Concept 4: DOM Removal

### Definitions

**Core Definition**
DOM removal is the act of deleting nodes from the document, and is a performance and memory concern because jQuery's removal methods perform additional cleanup (`cleanData`) to prevent memory leaks, while native removal does not.

**Technical Definition**
jQuery provides `.remove()`, `.detach()`, and `.empty()`. `.remove()` removes the element and its descendants from the DOM **and** calls `cleanData` to remove event handlers and data, preventing leaks. `.detach()` removes the element but **preserves** data and events, making it suitable for temporary removal and re‑insertion. `.empty()` removes all child nodes and cleans their data. Native `parentNode.removeChild()` bypasses jQuery's cleanup entirely. The cleanup overhead makes `.remove()` measurably slower than `removeChild()` — in one benchmark, removing a 1,500‑row table took ~300 ms with jQuery `.remove()` versus ~30 ms with `removeChild()` — but `.remove()` is the safe choice when handlers and data exist.

**Beginner‑Friendly Explanation**
Removing an element seems simple, but if you attached click handlers to it, those handlers stay in memory unless you clean them up. jQuery's `.remove()` does that cleanup for you, which is why it is slower than the browser's raw removal — but it prevents memory leaks. If you are going to put the element back later, use `.detach()` instead, because it keeps the handlers intact. If you never attached anything and need speed, native `removeChild()` is fastest, but be careful.

---

### Purposes

- **To delete elements safely** without leaking memory.
- **To choose between `.remove()` (cleanup) and `.detach()` (preserve)** based on whether the element will return.
- **To avoid memory leaks** caused by orphaned handlers and data.
- **To use `.empty()`** when only the children need to be removed.
- **To understand the performance trade‑off** between jQuery cleanup and native speed.

---

### Syntax Rules and Structure

| Method | Removes Element? | Cleans Data/Events? | Preserves Data/Events? |
|---|---|---|---|
| `.remove()` | ✅ | ✅ | ❌ |
| `.detach()` | ✅ | ❌ | ✅ |
| `.empty()` | ❌ (children only) | ✅ (children) | ❌ (children) |
| `removeChild()` | ✅ | ❌ | ❌ |

---

### Constraints and Limitations

- **`.remove()` is slower than `removeChild()`** because of cleanup; use it when handlers/data exist.
- **`.detach()` must be followed by re‑insertion**, or it becomes a leak (the detached subtree retains data and handlers).
- **Native removal without cleanup** leaves orphaned entries in jQuery's cache.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="container">
    <button class="action">Click</button>
  </div>

  <script>
    // Step 1: Attach a handler and data
    $(".action").on("click", function () { console.log("clicked"); });
    $(".action").data("info", "important");

    // Step 2: Safe removal with cleanup
    $("#container .action").remove();   // handler and data cleaned up
    console.log("Removed safely");

    // Step 3: Detach (preserve) example
    var $btn = $("<button class='action'>Click</button>")
      .on("click", function () { console.log("clicked again"); });
    $("#container").append($btn);
    $btn.detach();                     // handler preserved
    $("#container").append($btn);      // re-attach — handler still works
  </script>
</body>
</html>
```

**Expected Output**
```
Removed safely
```
(The detached button, when re‑attached, still responds to clicks.)

**Why this output occurs**: `.remove()` calls `cleanData` internally, removing the handler and data. `.detach()` removes the element from the DOM but keeps its jQuery data and events intact, allowing re‑insertion without re‑binding.

---

### Real‑World Cases

- **SPA view teardown**: Calling `.remove()` on a view root to clean up its handlers.
- **Temporary removal**: Using `.detach()` when reordering or reparenting elements.
- **Dynamic tables**: Clearing `<tbody>` with `.empty()` before repopulating.

---

## Core Concept 5: DOM Cloning

### Definitions

**Core Definition**
DOM cloning is the act of creating a copy of a DOM element (and optionally its descendants and associated data/events) using jQuery's `.clone()` method, and is a performance concern because cloning is expensive — especially deep cloning with data and events — and can lead to duplicated IDs and handler confusion.

**Technical Definition**
`.clone([withDataAndEvents][, deepWithDataAndEvents])` creates a deep copy of the matched elements by default. With `true`, it copies data and events; with `true, true`, it also copies data and events of descendants. jQuery's clone uses native `cloneNode()` internally, but on older IE it must work around inappropriate event‑listener copying. Deep cloning is inherently expensive: cloning 1,000 list items is slower than detaching the original list and modifying it in place. The jQuery documentation notes that `.clone()` can modify cloned elements before insertion, which is more efficient than inserting and then modifying.

**Beginner‑Friendly Explanation**
Cloning means making a copy of an element. Copying something big — like a list with a thousand items — takes time. And if you copy event handlers and data along with the structure, it takes even longer. Cloning is useful for templates and simple duplication, but for large structures it is usually faster to detach the original, modify it, and put it back than to clone it. Also, cloned elements keep the same IDs, which can cause problems in the page.

---

### Purposes

- **To duplicate a template element** for repeated use.
- **To modify a copy without affecting the original**.
- **To create new elements from existing markup** (though HTML string construction is often faster).
- **To understand the performance cost** of deep cloning versus detaching and modifying in place.
- **To avoid duplicating data and events unintentionally** by choosing the appropriate arguments.

---

### Syntax Rules and Structure

```
$( selector ).clone()                   // copy structure only
$( selector ).clone(true)              // copy structure + data/events
$( selector ).clone(true, true)        // copy structure + data/events + descendant data/events
```

---

### Constraints and Limitations

- **Deep cloning is expensive**; cloning large structures is slower than detaching and modifying.
- **`.clone(true)` duplicates IDs**, which can break `getElementById` and cause unexpected behaviour.
- **Cloning with data and events can duplicate handler logic**, leading to double‑firing or memory bloat.
- **Native `cloneNode()` is faster** than jQuery `.clone()` for structure‑only cloning.

---

### Annotated Code Example

```javascript
// ❌ EXPENSIVE: cloning a large list to modify it
var $clone = $("#largeList").clone(true); // deep clone with data/events
$clone.find("li").addClass("updated");
$("#largeList").replaceWith($clone);

// ✅ MORE EFFICIENT: detach, modify, re-attach
var $list = $("#largeList");
var $parent = $list.parent();
$list.detach();
$list.find("li").addClass("updated");
$parent.append($list);
```

**Expected Output**
Both versions produce a list with the `updated` class on every item, but the detach‑modify‑reattach pattern avoids the cost of cloning the entire structure.

**Why this output occurs**: Detaching the existing list preserves its data and events without copying, while modifications happen off‑document. Cloning creates a full copy, which is more expensive for large structures.

---

### Real‑World Cases

- **Template rendering**: Cloning a `<template>` or hidden element for each new item.
- **Form duplication**: Adding repeated field groups by cloning a prototype row.
- **Drag‑and‑drop copies**: Cloning an item when the user holds a modifier key.

---

## Core Concept 6: Document Fragments

### Definitions

**Core Definition**
A `DocumentFragment` is a lightweight, in‑memory container for DOM nodes that is not part of the live document. Nodes can be appended to a fragment without triggering reflow, and the entire fragment can then be inserted into the document in a single operation.

**Technical Definition**
`document.createDocumentFragment()` returns an empty `DocumentFragment`. Appending nodes to it does not affect the live DOM or trigger layout. When the fragment is inserted (e.g., via `appendChild()` or jQuery's `.append()`), its children are moved into the target in one DOM operation. jQuery uses DocumentFragment internally for `.append()` and related methods when inserting multiple nodes from an HTML string, and caches fragments for strings of 512 bytes or less. For larger or more complex insertions, developers can build a fragment explicitly for maximum control.

**Beginner‑Friendly Explanation**
A DocumentFragment is like a shopping cart. You can put as many items in it as you want while you are still in the store (off‑document), and then bring the whole cart to the checkout once (insert into the DOM). Putting items in the cart does not affect the store's layout; only the final checkout does. This is much faster than carrying each item to the checkout separately.

---

### Purposes

- **To batch insertions** without triggering reflow for each node.
- **To build complex structures off‑document** before a single insertion.
- **To avoid the parsing overhead** of HTML strings for dynamically created nodes.
- **To improve insertion performance** for large collections of elements.
- **To supplement jQuery's internal fragment usage** when finer control is needed.

---

### Syntax Rules and Structure

```javascript
var frag = document.createDocumentFragment();

for ( … ) {
  var node = document.createElement("li");
  node.textContent = "…";
  frag.appendChild(node);           // no reflow
}

$( target ).append( frag );         // one DOM operation
```

---

### Constraints and Limitations

- **DocumentFragment is a native DOM object**, not a jQuery object; jQuery can wrap it, but the fragment itself is not chainable.
- **Fragments are consumed on insertion**; after appending, the fragment is empty.
- **jQuery's internal fragment cache** applies only to HTML strings of 512 bytes or less; larger strings may not benefit.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="grid"></div>

  <script>
    var $grid = $("#grid");
    var frag = document.createDocumentFragment();

    for (var i = 1; i <= 1000; i++) {
      var card = document.createElement("div");
      card.className = "card";
      card.textContent = "Card " + i;
      frag.appendChild(card);          // off-document
    }

    $grid.append(frag);                // one insertion
    console.log("Cards:", $grid.children().length);
  </script>
</body>
</html>
```

**Expected Output**
```
Cards: 1000
```

**Why this output occurs**: All 1,000 cards are created and appended to the fragment off‑document. The single `.append(frag)` moves them into `#grid` in one operation, reducing reflow to a single event.

---

### Real‑World Cases

- **Card grids**: Building hundreds of cards in a fragment before insertion.
- **Table bodies**: Constructing all rows in a fragment and appending to `<tbody>`.
- **List virtualisation**: Creating only visible items in a fragment for efficient rendering.

---

## Core Concept 7: Detached DOM

### Definitions

**Core Definition**
A detached DOM element is a node that has been removed from the live document tree but still exists in memory, along with its data and event handlers. jQuery's `.detach()` method creates detached elements while preserving their data and events, enabling off‑document manipulation before re‑insertion.

**Technical Definition**
When `.detach()` is called, jQuery removes the element from its parent using `removeChild` but **does not** call `cleanData`, so the element's jQuery data and event handlers remain intact. The detached subtree is not part of the document, so modifications to it do not trigger reflow or repaint. When the element is re‑inserted, a single reflow occurs. Detached DOM is a performance tool **only if the element is re‑inserted**; failing to re‑insert creates a memory leak because the detached subtree retains data and handlers indefinitely.

**Beginner‑Friendly Explanation**
Detaching an element is like taking a piece of furniture out of a room to paint it. While it is outside, you can work on it without disturbing the room's layout. When you are done, you bring it back in once. If you take it out and never bring it back, it just sits outside taking up space — that is a memory leak. jQuery's `.detach()` is designed for the “take it out and bring it back” pattern.

---

### Purposes

- **To manipulate elements off‑document** without triggering repeated reflows.
- **To preserve data and events** during temporary removal.
- **To perform heavy structural changes** (adding rows, reordering children) before re‑insertion.
- **To reduce reflow to at most two operations**: one on detach, one on re‑attach.
- **To avoid the cost of cloning** when the same elements can be modified in place.

---

### Syntax Rules and Structure

```javascript
var $el = $( selector );
var $parent = $el.parent();   // remember the parent
$el.detach();                  // remove from DOM, keep data/events
// … heavy manipulation …
$parent.append( $el );         // re-attach (single reflow)
```

---

### Constraints and Limitations

- **Must re‑insert the detached element**, or it becomes a memory leak.
- **Detaching is not beneficial for simple operations** (e.g., `.addClass()`); the detach/re‑attach overhead may outweigh the benefit.
- **Layout‑dependent measurements** (e.g., `offsetHeight`) cannot be taken while detached.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="wrapper">
    <ul id="list">
      <li>Existing 1</li>
      <li>Existing 2</li>
    </ul>
  </div>

  <script>
    var $list = $("#list");
    var $parent = $list.parent();

    // Step 1: Detach the list
    $list.detach();

    // Step 2: Heavy manipulation off-document
    for (var i = 1; i <= 500; i++) {
      $list.append("<li>New " + i + "</li>");
    }

    // Step 3: Re-attach once
    $parent.append($list);
    console.log("Total items:", $list.children().length);
  </script>
</body>
</html>
```

**Expected Output**
```
Total items: 502
```

**Why this output occurs**: The list is detached before the 500 appends, so those operations do not trigger reflow. Re‑attaching causes one reflow. jQuery's documentation explicitly recommends this pattern: “The DOM is slow; you want to avoid manipulating it as much as possible”.

---

### Real-World Cases

- **Large table updates**: Detach the table, add/modify rows, re‑attach.
- **Sortable lists**: Detach a list while reordering its children.
- **Widget re‑initialisation**: Detach a widget, rebuild its internals, re‑attach.

---

## Core Concept 8: DOM Caching

### Definitions

**Core Definition**
DOM caching is the practice of storing a jQuery object (or a raw DOM reference) in a variable so that the same DOM query or traversal is not repeated, reducing the number of expensive searches through the document.

**Technical Definition**
Every call to `$(selector)` performs a DOM query and creates a new jQuery object. Caching assigns the result to a variable (`var $el = $("#id");`) and reuses that reference. The benefit is greatest for complex selectors and for selections used inside loops or event handlers. However, cached objects become **stale** if the DOM changes and the selected elements are removed or replaced. The recommended scope for caching is **local** (within the function or component that uses the selection) rather than global, to avoid stale references and unnecessary memory retention.

**Beginner‑Friendly Explanation**
Caching is like writing down a phone number instead of looking it up every time you need to call. The first lookup takes time; the rest are instant. In jQuery, the first `$(".item")` searches the page; if you save the result in a variable, later uses do not search again. The only caveat is that if the page changes and those items are replaced, your saved reference is out of date.

---

### Purposes

- **To eliminate redundant DOM queries** when the same selection is used multiple times.
- **To reduce jQuery object creation overhead** in loops and handlers.
- **To improve performance in repetitive code paths**.
- **To make code more readable** by naming selections.
- **To cache at the appropriate scope** (local, not global) to avoid stale data.

---

### Syntax Rules and Structure

```javascript
// ❌ REPEATED QUERY
$(".item").addClass("a");
$(".item").removeClass("b");
$(".item").css("color", "red");

// ✅ CACHED
var $items = $(".item");
$items.addClass("a");
$items.removeClass("b");
$items.css("color", "red");
```

---

### Constraints and Limitations

- **Cached objects become stale** if the selected elements are removed or replaced; re‑query when the DOM changes.
- **Global caching** can retain references longer than needed and cause memory issues.
- **Caching a selection used only once** adds a variable without benefit.

---

### Annotated Code Example

```javascript
// ✅ LOCAL CACHING inside a function
function highlightItems() {
  var $items = $(".item");          // query once per call
  $items.addClass("highlight");
  $items.attr("data-highlighted", "true");
  return $items.length;
}

// ✅ CACHING a stable parent for repeated scoped queries
var $list = $("#list");
function addItem(text) {
  $list.append("<li>" + text + "</li>"); // reuse cached parent
}
```

**Expected Output**
`highlightItems()` returns the number of highlighted items, and `addItem()` appends to the cached list without re‑querying `#list`.

**Why this output occurs**: The cached `$items` and `$list` variables avoid repeated DOM searches. Local caching prevents stale global references.

---

### Real-World Cases

- **Event handlers**: Caching `var $panel = $("#panel")` outside a click handler.
- **Loops**: Caching the target container before iterating over data.
- **Component methods**: Caching the component root (`var $el = this.$el`) for reuse.

---

## Core Concept 9: DOM Best Practices

### Definitions

**Core Definition**
jQuery DOM best practices are the consolidated conventions — batching, detaching, caching, separating reads and writes, cleaning up, and choosing appropriate insertion/removal/cloning strategies — that produce fast, memory‑safe, and maintainable DOM manipulation code.

**Technical Definition**
Best practices synthesise the individual techniques: (1) **batch DOM writes** using HTML strings or DocumentFragments; (2) **detach before heavy manipulation** and re‑attach once; (3) **cache selections and traversals** locally; (4) **separate layout reads from DOM writes** to avoid thrashing; (5) **use jQuery removal methods** (`.remove()`, `.empty()`) that clean data and events; (6) **avoid deep cloning** for large structures — detach and modify instead; (7) **clean up handlers and data** before native removal if native methods are required; and (8) **measure** with browser DevTools and high‑resolution timers to verify improvements.

**Beginner‑Friendly Explanation**
Best practices are the habits that keep DOM work fast. Make changes in batches instead of one at a time. Take elements out of the page before rearranging them, then put them back once. Remember the results of searches instead of repeating them. Do not mix reading layout information with changing the page in a loop. Remove elements with jQuery so handlers are cleaned up. Avoid copying large structures when you can just modify the original. And always measure before and after to see what actually helped.

---

### Purposes

- **To provide a single actionable checklist** for efficient DOM work.
- **To avoid the most common performance mistakes** (per‑item appends, repeated queries, layout thrashing, memory leaks).
- **To balance performance with readability and maintainability**.
- **To establish a baseline for code reviews and performance audits**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| Batching | Build HTML string or fragment, insert once | `.append()` in a loop |
| Detaching | `.detach()` before heavy work, re‑attach after | Manipulating live DOM repeatedly |
| Caching | `var $el = $("#id");` reuse | Repeated `$("#id")` |
| Reads vs. writes | Batch writes, then reads | Read‑write‑read‑write in a loop |
| Removal | `.remove()` / `.empty()` for cleanup | `removeChild()` without cleanup |
| Cloning | Detach‑modify‑re‑attach for large structures | Deep `.clone(true)` on large trees |
| Cleanup | `.off()` + `.removeData()` before native removal | Native removal with active handlers |
| Measurement | `performance.now()`, DevTools | Guessing without data |

---

### Constraints and Limitations

- **Best practices are context‑dependent**; caching a selection used once adds a variable without benefit.
- **Detaching has overhead**; for simple operations (e.g., a single `.addClass()`), detaching may be slower than manipulating in place.
- **DocumentFragment and HTML strings have different trade‑offs**; fragment avoids parsing but requires DOM API calls, while HTML strings are concise but parsed.

---

### Annotated Code Example (Consolidated)

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="app">
    <ul id="feed"></ul>
  </div>

  <script>
    // ✅ CACHE the stable root
    var $app = $("#app");
    var $feed = $("#feed");

    // ✅ DETACH before heavy insertion
    var $parent = $feed.parent();
    $feed.detach();

    // ✅ BATCH using a DocumentFragment
    var frag = document.createDocumentFragment();
    for (var i = 1; i <= 300; i++) {
      var li = document.createElement("li");
      li.textContent = "Post " + i;
      frag.appendChild(li);
    }

    // ✅ SINGLE insertion
    $feed.append(frag);

    // ✅ RE-ATTACH once
    $parent.append($feed);

    console.log("Posts rendered:", $feed.children().length);
  </script>
</body>
</html>
```

**Expected Output**
```
Posts rendered: 300
```

**Why this output occurs**: The feed is detached, 300 items are built in a fragment, the fragment is inserted in one operation, and the feed is re‑attached once. This combines caching, detaching, batching, and single‑insertion practices, minimising reflow and DOM touches.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new DOM code follows established performance patterns.
- **Plugin development**: Plugins should batch insertions, detach during rebuilds, cache their root, and clean up on destroy.
- **Legacy refactoring**: Identifying per‑item `.append()` calls and repeated selections as candidates for batching and caching.

---

## References

- Detach Elements to Work with Them | Learn jQuery — https://learn.jquery.com/performance/detach-elements-before-work-with-them/
- Append Outside of Loops | Learn jQuery — https://learn.jquery.com/performance/append-outside-loop/
- Cache Length During Loops | Learn jQuery — https://learn.jquery.com/performance/cache-loop-length/
- Don't Act on Absent Elements | Learn jQuery — https://learn.jquery.com/performance/dont-act-on-absent-elements/
- jQuery 高频 DOM 操作与性能优化指南 | H3C — https://kms.h3c.com/questions/dispcont/310443
- jQuery Performance Best Practices for Efficient Development | MoldStud — https://moldstud.com/articles/p-jquery-performance-best-practices-for-efficient-development
- Best Practices for jQuery and Sizzle | LoginRadius — https://www.loginradius.com/blog/engineering/optimize-jquery-sizzle-element-selector
- Cache results from your jQuery selectors | Progress — https://www.progress.com/documentation/sitefinity-cms/10/for-developers-cache-results-from-your-jquery-selectors
- Performance: jQuery remove() vs JavaScript removeChild() | Stack Overflow — https://stackoverflow.com/questions/16058894/performance-jquery-remove-vs-javascript-removechild
- jQuery Clone Performance | Stack Overflow — https://stackoverflow.com/questions/3120927/jquery-clone-performance
- Does manual garbage collection of DOM elements in jQuery improve browser performance at all? | Stack Overflow — https://stackoverflow.com/questions/15748609/does-manual-garbage-collection-of-dom-elements-in-jquery-improve-browser-performa
- `.detach()` | jQuery API Documentation — https://api.jquery.com/detach/
- `.remove()` | jQuery API Documentation — https://api.jquery.com/remove/
- `.empty()` | jQuery API Documentation — https://api.jquery.com/empty/
- `.clone()` | jQuery API Documentation — https://api.jquery.com/clone/
- DocumentFragment | MDN — https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment
- Core: Drop getElementsBy{Tag,Class}Name in favor of querySelectorAll | jQuery GitHub PR #5798 — https://github.com/jquery/jquery/pull/5798
- Performance degradation with delegate events and pseudo-classes | jQuery Bug Tracker — https://bugs.jquery.com/ticket/12203/
- jQuery source viewer (James Padolsey) — http://james.padolsey.com/jquery/