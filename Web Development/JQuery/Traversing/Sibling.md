# jQuery Sibling Traversal: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery sibling traversal methods are a set of jQuery functions that navigate **sideways** through the DOM tree, allowing developers to select and manipulate elements that share the same parent as a starting element.

**Technical Definition**
Sibling traversal methods operate on a jQuery object and construct a new jQuery object containing sibling elements of the original set. Sibling elements are elements that share the same parent. The methods differ in direction and scope: `.siblings()` returns all siblings (both before and after); `.next()`, `.nextAll()`, and `.nextUntil()` traverse forward through following siblings; and `.prev()`, `.prevAll()`, and `.prevUntil()` traverse backward through preceding siblings. All methods optionally accept a selector expression to filter the results. These methods are part of jQuery's **Traversing** API and are essential for navigating horizontally through the DOM.

**Beginner-Friendly Explanation**
Think of the DOM as a family tree. When you select an element — say, a `<li>` in a list — sibling traversal methods let you move sideways to its brothers and sisters (other `<li>` elements under the same parent). Some methods give you the very next sibling (`.next()`), some give you all following siblings (`.nextAll()`), some give you all preceding siblings (`.prevAll()`), and some let you specify a stopping point (`.nextUntil()`, `.prevUntil()`). The `.siblings()` method gives you everything — both before and after.

### Key Characteristics

- **Sideways Direction**: All sibling traversal methods move **horizontally** through the DOM tree, not up (ancestors) or down (descendants).
- **Shared Parent**: All selected siblings must share the same parent element.
- **New jQuery Objects**: Each method returns a new jQuery object; the original selection is not modified.
- **Optional Selector Filtering**: Most methods accept an optional selector to narrow results.
- **Order of Results**: `.nextAll()` returns elements in document order (closest first); `.prevAll()` returns elements in reverse document order (closest first).
- **Performance-Varied**: `.next()` and `.prev()` stop at one element and are generally faster; `.siblings()` and `.nextAll()`/`.prevAll()` may collect many elements.

### Prerequisites

- Basic understanding of HTML structure and the DOM tree (parents, children, siblings).
- Familiarity with jQuery basic selectors and the `$()` function.
- jQuery library included in the page via a `<script>` tag or CDN.
- A browser with developer tools for testing and inspection.

### Related Programming Areas

- **DOM Traversal**: Sibling traversal is one direction of the broader jQuery traversing API.
- **Event Delegation**: Sibling methods are used to navigate between related elements in event handlers.
- **List and Menu Manipulation**: Sibling traversal is essential for working with lists, navigation menus, and tab interfaces.
- **Form Layout**: Sibling methods help navigate between labels, inputs, and error messages in forms.

### Core Concepts / Features

1. `.siblings()` — All Siblings
2. `.next()` — Immediately Following Sibling
3. `.nextAll()` — All Following Siblings
4. `.nextUntil()` — Following Siblings Up to a Boundary
5. `.prev()` — Immediately Preceding Sibling
6. `.prevAll()` — All Preceding Siblings
7. `.prevUntil()` — Preceding Siblings Up to a Boundary

---

## Core Concept 1: `.siblings()` — All Siblings

### Definitions

**Core Definition**
`.siblings()` retrieves all siblings of each element in the current set, optionally filtered by a selector.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.siblings()` method allows us to search through the siblings of these elements in the DOM tree and construct a new jQuery object from the matching elements. Sibling elements are elements that share the same parent. The method traverses **both forward and backward** along siblings of DOM elements. The original element is **not** included in the result. The method optionally accepts a selector expression of the same type that we can pass to the `$()` function; if supplied, the siblings are filtered by testing whether they match it.

**Beginner-Friendly Explanation**
`.siblings()` gives you all the brothers and sisters of an element — every other element that shares the same parent. It looks both before and after the element in the DOM. The element itself is not included.

### Purposes

- To select all sibling elements of a set of elements for batch operations.
- To apply styles or behaviours to all siblings of an element.
- To navigate horizontally through the DOM without knowing the exact position of siblings.
- To filter siblings by a selector to target specific types of siblings.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).siblings()
$(selector).siblings(filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements whose siblings you want to find.
- `.siblings()` : Traverses sideways to all siblings.
- `filterSelector` (Optional) : A string containing a selector expression to match sibling elements against.
- Returns: A jQuery object containing the matched sibling elements.

**Syntax Rules**

1. Returns all siblings of each element in the original set, excluding the element itself.
2. The optional selector filters the siblings; only siblings matching the selector are included.
3. If multiple elements share the same siblings, duplicates are removed.
4. If an element has no siblings, the result is an empty jQuery object.

**Constraints and Limitations**

- Does not include the original element itself.
- If the original set contains elements with different parents, the results are the union of all their siblings.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.siblings()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.siblings() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    ul { border: 2px solid gray; padding: 10px; margin: 10px; }
    li { margin: 5px; }
  </style>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li class="target">Item 2 (target)</li>
    <li>Item 3</li>
    <li>Item 4</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select the target li and get all its siblings
      var $siblings = $("li.target").siblings();
      console.log("Siblings found:", $siblings.length); // 3

      // Step 2: Apply a red background to all siblings
      $siblings.css("background-color", "red");

      // Step 3: Log each sibling's text
      $siblings.each(function () {
        console.log("Sibling:", $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 1, 3, and 4 receive a red background.
- Item 2 (the target) remains unchanged.
- Console output:
```
Siblings found: 3
Sibling: Item 1
Sibling: Item 3
Sibling: Item 4
```

**Why This Output Occurs**
`$("li.target").siblings()` selects all siblings of the target list item. The target has three siblings: Items 1, 3, and 4. The target itself is not included. The red background is applied to all three siblings.

---

**Example 2: `.siblings()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.siblings() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="container">
    <p>Paragraph</p>
    <span>Span</span>
    <p class="special">Special paragraph</p>
    <div>Div</div>
  </div>

  <script>
    $(function () {
      // Step 1: Get only the paragraph siblings of .special
      var $paragraphSiblings = $(".special").siblings("p");
      console.log("Paragraph siblings:", $paragraphSiblings.length); // 1

      // Step 2: Get only div siblings
      var $divSiblings = $(".special").siblings("div");
      console.log("Div siblings:", $divSiblings.length); // 1

      // Step 3: Style them
      $paragraphSiblings.css("color", "blue");
      $divSiblings.css("border", "2px solid green");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first paragraph turns blue.
- The div receives a green border.
- Console output:
```
Paragraph siblings: 1
Div siblings: 1
```

**Why This Output Occurs**
`$(".special").siblings("p")` filters the siblings to include only `<p>` elements. The special paragraph has one paragraph sibling (the first `<p>`) and one div sibling. The span sibling is excluded because it does not match either filter.

### Real-World Cases

- **Tab Interfaces**: `$(".tab.active").siblings().removeClass("active")` removes the active class from all sibling tabs.
- **Navigation Menus**: `$("li.current").siblings().addClass("inactive")` marks all other menu items as inactive.
- **Form Layouts**: `$("input.error").siblings("label").addClass("error-label")` marks labels of error fields.
- **Card Layouts**: `$(".card.selected").siblings().css("opacity", 0.5)` dims all other cards.

### References

- .siblings() – https://api.jquery.com/siblings/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery siblings() Method – https://www.w3schools.com/jquery/traversing_siblings.asp

---

## Core Concept 2: `.next()` — Immediately Following Sibling

### Definitions

**Core Definition**
`.next()` retrieves the immediately following sibling of each element in the current set, optionally filtered by a selector.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.next()` method allows us to search through the successors of these elements in the DOM tree and construct a new jQuery object from the matching elements. The method retrieves the immediately following sibling of each element in the set of matched elements. If a selector is provided, it retrieves the next sibling only if it matches that selector. It traverses forward along the next sibling of DOM elements.

**Beginner-Friendly Explanation**
`.next()` gives you the very next sibling of an element — the one immediately after it in the DOM. If you have a list, `.next()` on one item gives you the item right below it.

### Purposes

- To select the immediately following sibling of an element for targeted manipulation.
- To navigate from one element to the next in a sequence.
- To apply styles or behaviours to the element that follows a specific element.
- To serve as a precise, single-step traversal method.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).next()
$(selector).next(filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements whose next sibling you want to find.
- `.next()` : Traverses forward one sibling.
- `filterSelector` (Optional) : A string containing a selector expression to match the next sibling against.
- Returns: A jQuery object containing the matched next sibling elements.

**Syntax Rules**

1. Returns the immediately following sibling of each element in the original set.
2. If the next sibling does not match the filter selector, the result for that element is empty.
3. Only travels **one step forward**; it does not traverse further siblings.
4. If an element has no next sibling (e.g., it is the last child), the result is empty.

**Constraints and Limitations**

- Only selects the immediately following sibling; use `.nextAll()` for all following siblings.
- If the next sibling does not match the filter, the result is empty — it does not continue searching.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.next()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.next() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li class="start">Item 2 (start)</li>
    <li>Item 3</li>
    <li>Item 4</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select the start li and find its next sibling
      var $next = $("li.start").next();
      console.log("Next sibling:", $next.text()); // "Item 3"

      // Step 2: Apply a red background to the next sibling
      $next.css("background-color", "red");

      // Step 3: Log the length
      console.log("Next count:", $next.length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- Item 3 receives a red background.
- Console output:
```
Next sibling: Item 3
Next count: 1
```

**Why This Output Occurs**
`$("li.start").next()` selects the immediately following sibling of the start item, which is Item 3. Item 3 receives the red background.

---

**Example 2: `.next()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.next() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <p>Paragraph 1</p>
    <span>Span</span>
    <p>Paragraph 2</p>
  </div>

  <script>
    $(function () {
      // Step 1: Get the next sibling if it is a paragraph
      var $nextP = $("p:first").next("p");
      console.log("Next p sibling:", $nextP.length); // 0

      // Step 2: Get the next sibling regardless of type
      var $nextAny = $("p:first").next();
      console.log("Next any sibling:", $nextAny.get(0).tagName); // "SPAN"

      // Step 3: The next sibling of the span is a paragraph
      var $spanNext = $("span").next("p");
      console.log("Span's next p:", $spanNext.text()); // "Paragraph 2"
    });
  </script>
</body>
</html>
```

**Expected Output**
- Console output:
```
Next p sibling: 0
Next any sibling: SPAN
Span's next p: Paragraph 2
```

**Why This Output Occurs**
`$("p:first").next("p")` returns 0 because the next sibling of the first paragraph is a `<span>`, not a `<p>`. `.next()` does not skip non-matching elements; it only checks the immediate next sibling. The span's next sibling is a paragraph, which matches.

### Real-World Cases

- **Form Validation**: `$("input").next(".error-message").show()` shows error messages next to inputs.
- **List Navigation**: `$("li.active").next().addClass("next-item")` highlights the item after the active one.
- **Tab Panels**: `$(".tab.active").next(".panel").show()` shows the panel immediately after the active tab.
- **Accordion Menus**: `$(".accordion-header").next(".accordion-content").slideToggle()`.

### References

- .next() – https://api.jquery.com/next/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery next() Method – https://www.w3schools.com/jquery/traversing_next.asp

---

## Core Concept 3: `.nextAll()` — All Following Siblings

### Definitions

**Core Definition**
`.nextAll()` retrieves all following siblings of each element in the current set, optionally filtered by a selector.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.nextAll()` method allows us to search through the successors of these elements in the DOM tree and construct a new jQuery object from the matching elements. The method returns all next sibling elements of the selected element. Sibling elements are elements that share the same parent. This method traverses forward along siblings of DOM elements. The method optionally accepts a selector expression of the same type that we can pass to the `$()` function.

**Beginner-Friendly Explanation**
`.nextAll()` gives you **all** the siblings that come after an element — not just the immediate next one, but every sibling that follows.

### Purposes

- To select all following siblings of an element for batch operations.
- To apply styles or behaviours to all elements that come after a specific element.
- To navigate forward through the DOM from a starting point.
- To filter following siblings by a selector to target specific types.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).nextAll()
$(selector).nextAll(filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements whose following siblings you want to find.
- `.nextAll()` : Traverses forward through all following siblings.
- `filterSelector` (Optional) : A string containing a selector expression to match sibling elements against.
- Returns: A jQuery object containing all matching following sibling elements.

**Syntax Rules**

1. Returns **all** following siblings, not just the immediate one.
2. The optional selector filters the siblings; only siblings matching the selector are included.
3. Results are returned in document order (closest sibling first).
4. If no following siblings exist, the result is empty.

**Constraints and Limitations**

- May return a large number of elements for elements near the start of a long list.
- Does not include the original element itself.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.nextAll()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.nextAll() — All Following Siblings</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li class="start">Item 2 (start)</li>
    <li>Item 3</li>
    <li>Item 4</li>
    <li>Item 5</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Get all following siblings of the start item
      var $following = $("li.start").nextAll();
      console.log("Following siblings:", $following.length); // 3

      // Step 2: Apply a red background to all of them
      $following.css("background-color", "red");

      // Step 3: Log each sibling's text
      $following.each(function () {
        console.log("Sibling:", $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 3, 4, and 5 receive a red background.
- Console output:
```
Following siblings: 3
Sibling: Item 3
Sibling: Item 4
Sibling: Item 5
```

**Why This Output Occurs**
`$("li.start").nextAll()` selects all siblings that follow the start item: Items 3, 4, and 5. The red background is applied to all three.

---

**Example 2: `.nextAll()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.nextAll() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <h2>Heading</h2>
    <p class="first">First paragraph</p>
    <span>Span</span>
    <p class="second">Second paragraph</p>
    <p class="third">Third paragraph</p>
  </div>

  <script>
    $(function () {
      // Step 1: Get all following paragraph siblings
      var $followingP = $("h2").nextAll("p");
      console.log("Following paragraphs:", $followingP.length); // 3

      // Step 2: Get only paragraphs with class "first" or "second"
      var $specific = $("h2").nextAll("p.first, p.second");
      console.log("Specific paragraphs:", $specific.length); // 2

      // Step 3: Style them
      $followingP.css("color", "darkgreen");
      $specific.css("font-weight", "bold");
    });
  </script>
</body>
</html>
```

**Expected Output**
- All three following paragraphs turn dark green.
- The first and second paragraphs also become bold.
- Console output:
```
Following paragraphs: 3
Specific paragraphs: 2
```

**Why This Output Occurs**
`$("h2").nextAll("p")` selects all following siblings that are `<p>` elements, skipping the `<span>`. The second call filters further to only paragraphs with classes `first` or `second`.

### Real-World Cases

- **List Styling**: `$("li.active").nextAll().addClass("after-active")` marks all items after the active one.
- **Table Row Highlighting**: `$("tr.selected").nextAll("tr").css("background", "#f0f0f0")` highlights all rows after the selected one.
- **Accordion Sections**: `$(".header.open").nextAll(".content").slideUp()` closes all content sections after an open header.
- **Breadcrumb Navigation**: `$(".breadcrumb li.current").nextAll().remove()` removes all breadcrumb items after the current one.

### References

- .nextAll() – https://api.jquery.com/nextAll/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery nextAll() Method – https://www.w3schools.com/jquery/traversing_nextall.asp

---

## Core Concept 4: `.nextUntil()` — Following Siblings Up to a Boundary

### Definitions

**Core Definition**
`.nextUntil()` retrieves all following siblings of each element up to **but not including** the element matched by a selector, DOM node, or jQuery object.

**Technical Definition**
Given a selector expression that represents a set of DOM elements, the `.nextUntil()` method searches through the successors of these elements in the DOM tree, stopping when it reaches an element matched by the method's argument. The new jQuery object that is returned contains all following siblings up to but not including the one matched by the `.nextUntil()` selector. If the selector is not matched or is not supplied, all following siblings will be selected; in these cases it selects the same elements as the `.nextAll()` method does when no filter selector is provided. As of jQuery 1.6, a DOM node or jQuery object, instead of a selector, may be used for the first `.nextUntil()` argument.

**Beginner-Friendly Explanation**
`.nextUntil()` is like `.nextAll()` but with a stopping point. It collects all following siblings until it reaches a specific element, and it does **not** include that stopping element in the results.

### Purposes

- To select a range of following siblings between a starting element and a boundary element.
- To apply styles or behaviours to a specific group of siblings without affecting the boundary element.
- To avoid traversing beyond a known element that defines the end of a section.
- To narrow forward traversal to a specific scope defined by an end element.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).nextUntil(stopSelector)
$(selector).nextUntil(stopSelector, filterSelector)
$(selector).nextUntil(element)
$(selector).nextUntil(element, filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements from which to start the traversal.
- `.nextUntil(stopSelector)` : Traverses forward until reaching an element matching `stopSelector`, excluding it from the results.
- `filterSelector` (Optional) : A selector to filter the returned siblings.
- `element` (Optional) : A DOM node or jQuery object to use as the stop element (added in jQuery 1.6).
- Returns: A jQuery object containing siblings between the starting element and the stop element.

**Syntax Rules**

1. The **stop element is not included** in the returned set.
2. If the stop selector does not match any sibling, the method behaves like `.nextAll()` and returns all following siblings.
3. Added in jQuery 1.4; extended to accept DOM nodes and jQuery objects in 1.6.
4. The optional filter selector is applied **after** the stop boundary is determined.
5. If both parameters are empty, the method returns all next sibling elements (same as `.nextAll()`).

**Constraints and Limitations**

- If the stop element is not found, the method returns all following siblings.
- Performance depends on the number of siblings traversed.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.nextUntil()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.nextUntil() — Stop at a Specific Element</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li class="start">Item 2 (start)</li>
    <li>Item 3</li>
    <li>Item 4</li>
    <li class="stop">Item 5 (stop)</li>
    <li>Item 6</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Get siblings between start and stop
      var $between = $("li.start").nextUntil("li.stop");
      console.log("Items between:", $between.length); // 2

      // Step 2: Apply a red background
      $between.css("background-color", "red");

      // Step 3: Log each item
      $between.each(function () {
        console.log("Between:", $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 3 and 4 receive a red background.
- Item 5 (the stop element) is not included.
- Console output:
```
Items between: 2
Between: Item 3
Between: Item 4
```

**Why This Output Occurs**
`$("li.start").nextUntil("li.stop")` traverses forward from Item 2, collecting Item 3 and Item 4, and stops when it reaches Item 5 (the stop element). Item 5 is not included in the results.

---

**Example 2: `.nextUntil()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.nextUntil() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <h2>Heading</h2>
    <p>Paragraph 1</p>
    <span>Span</span>
    <p>Paragraph 2</p>
    <p>Paragraph 3</p>
    <h3>Subheading</h3>
    <p>After subheading</p>
  </div>

  <script>
    $(function () {
      // Step 1: Get all siblings between h2 and h3
      var $between = $("h2").nextUntil("h3");
      console.log("All between:", $between.length); // 4

      // Step 2: Filter to only paragraphs
      var $betweenP = $("h2").nextUntil("h3", "p");
      console.log("Paragraphs between:", $betweenP.length); // 3

      // Step 3: Style them
      $betweenP.css("color", "blue");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The three paragraphs between the `<h2>` and `<h3>` turn blue.
- The span is not styled.
- Console output:
```
All between: 4
Paragraphs between: 3
```

**Why This Output Occurs**
`$("h2").nextUntil("h3")` collects all siblings between the `<h2>` and `<h3>`: Paragraph 1, Span, Paragraph 2, and Paragraph 3 (4 elements). The filter selector `"p"` narrows the result to only the three paragraphs.

### Real-World Cases

- **Form Sections**: `$("fieldset").nextUntil("fieldset").addClass("section-content")` styles content between fieldsets.
- **Table Sections**: `$("tr.header").nextUntil("tr.header")` selects all rows between two header rows.
- **Accordion Content**: `$(".accordion-header").nextUntil(".accordion-header")` selects the content between headers.
- **Navigation Groups**: `$("li.group-start").nextUntil("li.group-start")` selects items within a group.

### References

- .nextUntil() – https://api.jquery.com/nextUntil/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery nextUntil() Method – https://www.w3schools.com/jquery/traversing_nextuntil.asp

---

## Core Concept 5: `.prev()` — Immediately Preceding Sibling

### Definitions

**Core Definition**
`.prev()` retrieves the immediately preceding sibling of each element in the current set, optionally filtered by a selector.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.prev()` method allows us to search through the predecessors of these elements in the DOM tree and construct a new jQuery object from the matching elements. The method retrieves the immediately preceding sibling of each element in the set of matched elements. If a selector is provided, it retrieves the previous sibling only if it matches that selector. To select all preceding sibling elements, rather than just the preceding adjacent sibling, use the `.prevAll()` method.

**Beginner-Friendly Explanation**
`.prev()` gives you the very previous sibling of an element — the one immediately before it in the DOM.

### Purposes

- To select the immediately preceding sibling of an element for targeted manipulation.
- To navigate backward from one element to the previous one in a sequence.
- To apply styles or behaviours to the element that comes before a specific element.
- To serve as a precise, single-step backward traversal method.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).prev()
$(selector).prev(filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements whose previous sibling you want to find.
- `.prev()` : Traverses backward one sibling.
- `filterSelector` (Optional) : A string containing a selector expression to match the previous sibling against.
- Returns: A jQuery object containing the matched previous sibling elements.

**Syntax Rules**

1. Returns the immediately preceding sibling of each element in the original set.
2. If the previous sibling does not match the filter selector, the result for that element is empty.
3. Only travels **one step backward**; it does not traverse further siblings.
4. If an element has no previous sibling (e.g., it is the first child), the result is empty.

**Constraints and Limitations**

- Only selects the immediately preceding sibling; use `.prevAll()` for all preceding siblings.
- If the previous sibling does not match the filter, the result is empty.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.prev()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prev() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li class="target">Item 3 (target)</li>
    <li>Item 4</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select the target li and find its previous sibling
      var $prev = $("li.target").prev();
      console.log("Previous sibling:", $prev.text()); // "Item 2"

      // Step 2: Apply a red background to the previous sibling
      $prev.css("background-color", "red");

      // Step 3: Log the length
      console.log("Prev count:", $prev.length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- Item 2 receives a red background.
- Console output:
```
Previous sibling: Item 2
Prev count: 1
```

**Why This Output Occurs**
`$("li.target").prev()` selects the immediately preceding sibling of the target item, which is Item 2.

---

**Example 2: `.prev()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prev() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <p>Paragraph 1</p>
    <span>Span</span>
    <p class="target">Paragraph 2 (target)</p>
  </div>

  <script>
    $(function () {
      // Step 1: Get the previous sibling if it is a paragraph
      var $prevP = $(".target").prev("p");
      console.log("Previous p sibling:", $prevP.length); // 0

      // Step 2: Get the previous sibling regardless of type
      var $prevAny = $(".target").prev();
      console.log("Previous any sibling:", $prevAny.get(0).tagName); // "SPAN"

      // Step 3: The previous sibling of the span is a paragraph
      var $spanPrev = $("span").prev("p");
      console.log("Span's previous p:", $spanPrev.text()); // "Paragraph 1"
    });
  </script>
</body>
</html>
```

**Expected Output**
- Console output:
```
Previous p sibling: 0
Previous any sibling: SPAN
Span's previous p: Paragraph 1
```

**Why This Output Occurs**
`$(".target").prev("p")` returns 0 because the previous sibling of the target paragraph is a `<span>`, not a `<p>`. The span's previous sibling is a paragraph, which matches.

### Real-World Cases

- **Form Validation**: `$("input").prev(".label").addClass("error-label")` marks labels before error fields.
- **List Navigation**: `$("li.active").prev().addClass("prev-item")` highlights the item before the active one.
- **Tab Panels**: `$(".tab.active").prev(".panel").show()` shows the panel immediately before the active tab.
- **Accordion Menus**: `$(".accordion-content").prev(".accordion-header")` selects the header before content.

### References

- .prev() – https://api.jquery.com/prev/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery prev() Method – https://www.w3schools.com/jquery/traversing_prev.asp

---

## Core Concept 6: `.prevAll()` — All Preceding Siblings

### Definitions

**Core Definition**
`.prevAll()` retrieves all preceding siblings of each element in the current set, optionally filtered by a selector, in reverse document order.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.prevAll()` method searches through the predecessors of these elements in the DOM tree and constructs a new jQuery object from the matching elements. The method returns all preceding siblings of each element in the set of matched elements, optionally filtered by a selector, in the reverse document order. This means the closest sibling is returned first, followed by more distant siblings.

**Beginner-Friendly Explanation**
`.prevAll()` gives you **all** the siblings that come before an element — every sibling that precedes it. The closest sibling is listed first.

### Purposes

- To select all preceding siblings of an element for batch operations.
- To apply styles or behaviours to all elements that come before a specific element.
- To navigate backward through the DOM from a starting point.
- To filter preceding siblings by a selector to target specific types.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).prevAll()
$(selector).prevAll(filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements whose preceding siblings you want to find.
- `.prevAll()` : Traverses backward through all preceding siblings.
- `filterSelector` (Optional) : A string containing a selector expression to match sibling elements against.
- Returns: A jQuery object containing all matching preceding sibling elements in reverse document order.

**Syntax Rules**

1. Returns **all** preceding siblings, not just the immediate one.
2. Results are returned in **reverse document order** (closest sibling first).
3. The optional selector filters the siblings; only siblings matching the selector are included.
4. If no preceding siblings exist, the result is empty.

**Constraints and Limitations**

- May return a large number of elements for elements near the end of a long list.
- Does not include the original element itself.
- The reverse order of results may be counter-intuitive; use `.get().reverse()` if document order is needed.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.prevAll()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prevAll() — All Preceding Siblings</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li class="target">Item 4 (target)</li>
    <li>Item 5</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Get all preceding siblings of the target
      var $preceding = $("li.target").prevAll();
      console.log("Preceding siblings:", $preceding.length); // 3

      // Step 2: Apply a red background to all of them
      $preceding.css("background-color", "red");

      // Step 3: Log each sibling's text (in reverse order)
      $preceding.each(function () {
        console.log("Preceding:", $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 1, 2, and 3 receive a red background.
- Console output (in reverse order):
```
Preceding siblings: 3
Preceding: Item 3
Preceding: Item 2
Preceding: Item 1
```

**Why This Output Occurs**
`$("li.target").prevAll()` selects all siblings before Item 4: Items 1, 2, and 3. The results are returned in reverse document order, so Item 3 (closest) comes first.

---

**Example 2: `.prevAll()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prevAll() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <p class="first">First paragraph</p>
    <span>Span</span>
    <p class="second">Second paragraph</p>
    <p class="target">Target paragraph</p>
  </div>

  <script>
    $(function () {
      // Step 1: Get all preceding paragraph siblings
      var $precedingP = $(".target").prevAll("p");
      console.log("Preceding paragraphs:", $precedingP.length); // 2

      // Step 2: Get only paragraphs with class "first"
      var $first = $(".target").prevAll("p.first");
      console.log("First paragraph:", $first.length); // 1

      // Step 3: Style them
      $precedingP.css("color", "darkgreen");
      $first.css("font-weight", "bold");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two preceding paragraphs turn dark green.
- The first paragraph also becomes bold.
- Console output:
```
Preceding paragraphs: 2
First paragraph: 1
```

**Why This Output Occurs**
`$(".target").prevAll("p")` selects all preceding siblings that are `<p>` elements: the first and second paragraphs. The span is excluded. The second call filters further to only the paragraph with class `first`.

### Real-World Cases

- **List Styling**: `$("li.active").prevAll().addClass("before-active")` marks all items before the active one.
- **Table Row Highlighting**: `$("tr.selected").prevAll("tr").css("background", "#f0f0f0")` highlights all rows before the selected one.
- **Accordion Sections**: `$(".header.open").prevAll(".content").slideUp()` closes all content sections before an open header.
- **Breadcrumb Navigation**: `$(".breadcrumb li.current").prevAll().remove()` removes all breadcrumb items before the current one.

### References

- .prevAll() – https://api.jquery.com/prevAll/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery prevAll() Method – https://www.w3schools.com/jquery/traversing_prevall.asp

---

## Core Concept 7: `.prevUntil()` — Preceding Siblings Up to a Boundary

### Definitions

**Core Definition**
`.prevUntil()` retrieves all preceding siblings of each element up to **but not including** the element matched by a selector, DOM node, or jQuery object.

**Technical Definition**
Given a selector expression that represents a set of DOM elements, the `.prevUntil()` method searches through the predecessors of these elements in the DOM tree, stopping when it reaches an element matched by the method's argument. The new jQuery object that is returned contains all previous siblings up to but not including the one matched by the `.prevUntil()` selector; the elements are returned in order from the closest sibling to the farthest. If the selector is not matched or is not supplied, all previous siblings will be selected; in these cases it selects the same elements as the `.prevAll()` method does when no filter selector is provided. As of jQuery 1.6, a DOM node or jQuery object, instead of a selector, may be used for the first `.prevUntil()` argument.

**Beginner-Friendly Explanation**
`.prevUntil()` is like `.prevAll()` but with a stopping point. It collects all preceding siblings until it reaches a specific element, and it does **not** include that stopping element in the results.

### Purposes

- To select a range of preceding siblings between a starting element and a boundary element.
- To apply styles or behaviours to a specific group of siblings without affecting the boundary element.
- To avoid traversing beyond a known element that defines the start of a section.
- To narrow backward traversal to a specific scope defined by a start element.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).prevUntil(stopSelector)
$(selector).prevUntil(stopSelector, filterSelector)
$(selector).prevUntil(element)
$(selector).prevUntil(element, filterSelector)
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object containing elements from which to start the traversal.
- `.prevUntil(stopSelector)` : Traverses backward until reaching an element matching `stopSelector`, excluding it from the results.
- `filterSelector` (Optional) : A selector to filter the returned siblings.
- `element` (Optional) : A DOM node or jQuery object to use as the stop element (added in jQuery 1.6).
- Returns: A jQuery object containing siblings between the starting element and the stop element, in reverse document order.

**Syntax Rules**

1. The **stop element is not included** in the returned set.
2. If the stop selector does not match any sibling, the method behaves like `.prevAll()` and returns all preceding siblings.
3. Added in jQuery 1.4; extended to accept DOM nodes and jQuery objects in 1.6.
4. The optional filter selector is applied **after** the stop boundary is determined.
5. Elements are returned in order from the closest sibling to the farthest.

**Constraints and Limitations**

- If the stop element is not found, the method returns all preceding siblings.
- Performance depends on the number of siblings traversed.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `.prevUntil()` Usage**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prevUntil() — Stop at a Specific Element</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li class="stop">Item 2 (stop)</li>
    <li>Item 3</li>
    <li>Item 4</li>
    <li class="start">Item 5 (start)</li>
    <li>Item 6</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Get siblings between start and stop (backward)
      var $between = $("li.start").prevUntil("li.stop");
      console.log("Items between:", $between.length); // 2

      // Step 2: Apply a red background
      $between.css("background-color", "red");

      // Step 3: Log each item (in reverse order)
      $between.each(function () {
        console.log("Between:", $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 4 and 3 receive a red background (in that order).
- Item 2 (the stop element) is not included.
- Console output:
```
Items between: 2
Between: Item 4
Between: Item 3
```

**Why This Output Occurs**
`$("li.start").prevUntil("li.stop")` traverses backward from Item 5, collecting Item 4 and Item 3, and stops when it reaches Item 2 (the stop element). Item 2 is not included. The results are returned in reverse document order (closest first).

---

**Example 2: `.prevUntil()` with a Filter Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prevUntil() — With Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <h3>Subheading</h3>
    <p>Paragraph 1</p>
    <span>Span</span>
    <p>Paragraph 2</p>
    <p>Paragraph 3</p>
    <h2>Heading</h2>
  </div>

  <script>
    $(function () {
      // Step 1: Get all siblings between h2 and h3 (backward)
      var $between = $("h2").prevUntil("h3");
      console.log("All between:", $between.length); // 4

      // Step 2: Filter to only paragraphs
      var $betweenP = $("h2").prevUntil("h3", "p");
      console.log("Paragraphs between:", $betweenP.length); // 3

      // Step 3: Style them
      $betweenP.css("color", "blue");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The three paragraphs between the `<h2>` and `<h3>` turn blue.
- The span is not styled.
- Console output:
```
All between: 4
Paragraphs between: 3
```

**Why This Output Occurs**
`$("h2").prevUntil("h3")` collects all siblings between the `<h2>` and `<h3>`: Paragraph 3, Paragraph 2, Span, and Paragraph 1 (4 elements, in reverse order). The filter selector `"p"` narrows the result to only the three paragraphs.

### Real-World Cases

- **Form Sections**: `$("fieldset").prevUntil("fieldset").addClass("section-content")` styles content between fieldsets.
- **Table Sections**: `$("tr.header").prevUntil("tr.header")` selects all rows between two header rows.
- **Accordion Content**: `$(".accordion-header").prevUntil(".accordion-header")` selects the content between headers.
- **Navigation Groups**: `$("li.group-end").prevUntil("li.group-start")` selects items within a group.

### References

- .prevUntil() – https://api.jquery.com/prevUntil/
- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- jQuery prevUntil() Method – https://www.w3schools.com/jquery/traversing_prevuntil.asp

---

## Summary Table: jQuery Sibling Traversal Methods at a Glance

| Method | Direction | Scope | Includes Self? | Returns | Optional Filter? | Added |
|---|---|---|---|---|---|---|
| `.siblings()` | Both directions | All siblings | No | All siblings | Yes | 1.0 |
| `.next()` | Forward | One sibling | No | Immediately following sibling | Yes | 1.0 |
| `.nextAll()` | Forward | All following | No | All following siblings | Yes | 1.2 |
| `.nextUntil()` | Forward | Until boundary | No | Following siblings up to boundary | Yes | 1.4 |
| `.prev()` | Backward | One sibling | No | Immediately preceding sibling | Yes | 1.0 |
| `.prevAll()` | Backward | All preceding | No | All preceding siblings (reverse order) | Yes | 1.2 |
| `.prevUntil()` | Backward | Until boundary | No | Preceding siblings up to boundary (reverse order) | Yes | 1.4 |

### Key Differences

| Aspect | `.siblings()` | `.next()` | `.nextAll()` | `.nextUntil()` | `.prev()` | `.prevAll()` | `.prevUntil()` |
|---|---|---|---|---|---|---|---|
| Direction | Both | Forward | Forward | Forward | Backward | Backward | Backward |
| Scope | All | One | All | Until stop | One | All | Until stop |
| Order | Document | N/A | Document | Document | N/A | Reverse | Reverse |
| Stop element included | N/A | N/A | N/A | No | N/A | N/A | No |

---

## General References

- Traversing | jQuery API Documentation – https://api.jquery.com/category/traversing/
- Tree Traversal | jQuery API Documentation – https://api.jquery.com/category/traversing/tree-traversal/
- .siblings() – https://api.jquery.com/siblings/
- .next() – https://api.jquery.com/next/
- .nextAll() – https://api.jquery.com/nextAll/
- .nextUntil() – https://api.jquery.com/nextUntil/
- .prev() – https://api.jquery.com/prev/
- .prevAll() – https://api.jquery.com/prevAll/
- .prevUntil() – https://api.jquery.com/prevUntil/
- jQuery Learning Center — Traversing – https://learn.jquery.com/using-jquery-core/traversing/
- W3Schools — jQuery Traversing Siblings – https://www.w3schools.com/jquery/jquery_traversing_siblings.asp
- W3Schools — jQuery siblings() Method – https://www.w3schools.com/jquery/traversing_siblings.asp
- W3Schools — jQuery next() Method – https://www.w3schools.com/jquery/traversing_next.asp
- W3Schools — jQuery nextAll() Method – https://www.w3schools.com/jquery/traversing_nextall.asp
- W3Schools — jQuery nextUntil() Method – https://www.w3schools.com/jquery/traversing_nextuntil.asp
- W3Schools — jQuery prev() Method – https://www.w3schools.com/jquery/traversing_prev.asp
- W3Schools — jQuery prevAll() Method – https://www.w3schools.com/jquery/traversing_prevall.asp
- W3Schools — jQuery prevUntil() Method – https://www.w3schools.com/jquery/traversing_prevuntil.asp
- Version 1.2 | jQuery API Documentation – https://api.jquery.com/category/version/1.2/
- Version 1.6 | jQuery API Documentation – https://api.jquery.com/category/version/1.6/