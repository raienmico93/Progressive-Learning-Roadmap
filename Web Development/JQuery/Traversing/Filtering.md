# jQuery Filtering Collections: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery filtering methods are a set of jQuery functions that reduce a matched set of DOM elements to a smaller subset based on position, selector matching, or custom test functions.

**Technical Definition**
Filtering methods operate on an existing jQuery object and construct a new jQuery object containing a subset of the original elements. They differ from traversal methods in that they do not navigate the DOM tree; instead, they test each element in the current set against criteria (a selector, a function, a DOM element, or another jQuery object) and include or exclude elements based on the result. Filtering methods are part of jQuery's **Traversing** API and are essential for narrowing selections after an initial query.

**Beginner-Friendly Explanation**
When you select elements with jQuery, you often get more than you need. Filtering methods let you narrow down that list. Some filters work by position — like picking the first, last, or third item. Others work by rules — like "only keep the paragraphs that have the class 'intro'" or "remove all items that are hidden." They are like a sieve that lets only the elements you want through.

### Key Characteristics

- **Non-Navigational**: Filtering methods do not move through the DOM tree; they test the elements already in the set.
- **New jQuery Objects**: Each method returns a new jQuery object; the original selection is not modified.
- **Two Return Types**: Most filtering methods return a jQuery object; `.is()` returns a **Boolean** instead.
- **Selector or Function Criteria**: Most methods accept either a selector string or a callback function as the filtering criterion.
- **Performance-Varied**: Pure CSS selectors can leverage native `querySelectorAll()`; jQuery extension selectors and function-based filters are generally slower.

### Prerequisites

- Basic understanding of jQuery selectors and the `$()` function.
- Familiarity with jQuery objects and array-like collections.
- Awareness of zero-based indexing in JavaScript.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **DOM Traversal**: Filtering is one category of the broader jQuery traversing API.
- **Event Handling**: Filtering is often used to narrow event targets or detect element states with `.is()`.
- **Data Filtering**: `.filter()` and `.not()` are used to filter collections based on custom logic.
- **Performance Optimisation**: Choosing the right filtering method can significantly impact script performance.

### Core Concepts / Features

1. `.first()` — First Element in Set
2. `.last()` — Last Element in Set
3. `.eq()` — Element at Specific Index
4. `.slice()` — Subset by Index Range
5. `.filter()` — Elements Matching Selector or Test
6. `.not()` — Elements NOT Matching Selector or Test
7. `.has()` — Elements with Matching Descendant
8. `.is()` — Boolean Match Check

---

## Core Concept 1: `.first()` — First Element in Set

### Definitions

**Core Definition**
`.first()` reduces the set of matched elements to the first one in the set.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.first()` method constructs a new jQuery object from the first element in that set. It is equivalent to `.eq(0)` and the deprecated `:first` selector. The method was added in jQuery 1.4. It does not accept any arguments.

**Beginner-Friendly Explanation**
`.first()` gives you the very first element from everything you selected. If you select all paragraphs, `.first()` gives you only the first paragraph on the page.

### Purposes

- To select the first element from a matched set for targeted manipulation.
- To apply a special style or behaviour to only the first item in a list.
- To serve as a performance-friendly alternative to the deprecated `:first` selector.
- To reduce a large collection to a single element for focused operations.

### Syntax Rules and Structure

```javascript
$(selector).first()
```

**Component Breakdown**
- `$(selector)` : The initial jQuery object.
- `.first()` : Takes no arguments; returns a new jQuery object containing the first element.
- Returns: A jQuery object containing exactly one element (or zero if the set is empty).

**Syntax Rules**
1. Takes **no arguments**.
2. Returns a jQuery object (unlike `.get(0)`, which returns a native DOM element).
3. Equivalent to `.eq(0)`.
4. Added in jQuery 1.4.

**Constraints and Limitations**
- Only returns one element regardless of how many parents exist.
- Use `.first-child` if you need the first child of each parent.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.first() — Highlight First List Item</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background-color: yellow; }
  </style>
</head>
<body>
  <ul>
    <li>Look:</li>
    <li>This is some text in a list.</li>
    <li>This is a note about it.</li>
    <li>This is another note about it.</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select all list items and reduce to the first
      $("ul li").first().addClass("highlight");
      // This is equivalent to: $("ul li").eq(0).addClass("highlight");

      // Step 2: Verify
      console.log("First item text:", $("ul li").first().text()); // "Look:"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first list item (“Look:”) receives a yellow background.
- Console output: `First item text: Look:`

**Why This Output Occurs**
`$("ul li").first()` reduces the collection of all `<li>` elements to the first one, which is the list item containing “Look:”. The `.addClass("highlight")` method then applies the yellow background to that single element.

### Real-World Cases
- **Highlighting the first row of a data table**.
- **Styling the first item in a navigation list differently**.
- **Selecting the first image in a gallery for a featured display**.

### References
- .first() – https://api.jquery.com/first/
- :first Selector (Deprecated) – https://api.jquery.com/first-selector/

---

## Core Concept 2: `.last()` — Last Element in Set

### Definitions

**Core Definition**
`.last()` reduces the set of matched elements to the last one in the set.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.last()` method constructs a new jQuery object from the last matching element in that set. It is equivalent to `.eq(-1)` and the deprecated `:last` selector.

**Beginner-Friendly Explanation**
`.last()` gives you the very last element from everything you selected. If you select all list items, `.last()` gives you only the final list item.

### Purposes

- To select the last element from a matched set for targeted manipulation.
- To apply a special style or behaviour to only the last item in a list.
- To serve as a performance-friendly alternative to the deprecated `:last` selector.
- To reduce a collection to a single element at the end.

### Syntax Rules and Structure

```javascript
$(selector).last()
```

**Component Breakdown**
- `$(selector)` : The initial jQuery object.
- `.last()` : Takes no arguments; returns a new jQuery object containing the last element.
- Returns: A jQuery object containing exactly one element (or zero if the set is empty).

**Syntax Rules**
1. Takes **no arguments**.
2. Returns a jQuery object.
3. Equivalent to `.eq(-1)`.
4. Added in jQuery 1.4.

**Constraints and Limitations**
- Only returns one element regardless of how many parents exist.
- Use `.last-child` if you need the last child of each parent.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.last() — Highlight Last List Item</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background-color: yellow; }
  </style>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4 (last)</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select all list items and reduce to the last
      $("ul li").last().addClass("highlight");

      // Step 2: Verify
      console.log("Last item text:", $("ul li").last().text()); // "Item 4 (last)"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The last list item (“Item 4 (last)”) receives a yellow background.
- Console output: `Last item text: Item 4 (last)`

**Why This Output Occurs**
`$("ul li").last()` reduces the collection to the final `<li>` element, which contains “Item 4 (last)”. The `.addClass("highlight")` method applies the yellow background.

### Real-World Cases
- **Styling the last item in a breadcrumb navigation**.
- **Adding a “last” class to the final element in a list**.
- **Selecting the most recently added element in a dynamic list**.

### References
- .last() – https://api.jquery.com/last/
- :last Selector (Deprecated) – https://api.jquery.com/last-selector/

---

## Core Concept 3: `.eq()` — Element at Specific Index

### Definitions

**Core Definition**
`.eq()` reduces the set of matched elements to the one at the specified zero-based index.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.eq()` method constructs a new jQuery object from one element within that set. The supplied index identifies the position of this element in the set. Note that the supplied index is **zero-based** and refers to the position of the element within the jQuery object, not within the DOM tree. Providing a negative number indicates a position starting from the end of the set. If an element cannot be found at the specified index, the method constructs a new jQuery object with an empty set and a `length` property of `0`.

**Beginner-Friendly Explanation**
`.eq()` is like array indexing. `.eq(0)` gives you the first element, `.eq(1)` gives the second, and so on. Negative numbers count from the end: `.eq(-1)` is the last element.

### Purposes

- To select a single element at a known position within a matched set.
- To access elements by their zero-based index in a collection.
- To select elements counting backward from the end using negative indices.
- To serve as a performance-friendly alternative to the deprecated `:eq()` selector.

### Syntax Rules and Structure

```javascript
$(selector).eq(index)
$(selector).eq(-index)
```

**Component Breakdown**
- `.eq(index)` : Zero-based index of the element to match.
- `.eq(-index)` : Zero-based index counting backwards from the last element (added in jQuery 1.4).
- Returns: A jQuery object containing zero or one element.

**Syntax Rules**
1. Index is **zero-based**: `.eq(0)` is the first element, `.eq(1)` is the second.
2. Negative indices count from the end: `.eq(-1)` is the last element.
3. Added in jQuery 1.1.2; negative index support added in 1.4.
4. Returns an empty jQuery object if the index is out of bounds.

**Constraints and Limitations**
- Only selects a single element.
- Use `:nth-child()` to select the nth child of each parent.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.eq() — Highlight Third Div</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    div { width: 60px; height: 60px; margin: 10px; float: left;
          border: 2px solid blue; }
    .blue { background: blue; }
  </style>
</head>
<body>
  <div></div><div></div><div></div>
  <div></div><div></div><div></div>

  <script>
    $(function () {
      // Step 1: Find all divs and reduce to index 2 (the third div)
      $("body").find("div").eq(2).addClass("blue");

      // Step 2: Select the second-to-last div
      $("body").find("div").eq(-2).css("border-color", "red");

      // Step 3: Verify
      console.log("Div at index 2 has blue class:",
        $("body").find("div").eq(2).hasClass("blue")); // true
    });
  </script>
</body>
</html>
```

**Expected Output**
- The third `<div>` (index 2) receives a blue background.
- The second-to-last `<div>` receives a red border.
- Console output: `Div at index 2 has blue class: true`

**Why This Output Occurs**
`$("body").find("div").eq(2)` selects the element at index 2 (the third `<div>`) from the collection of all `<div>` elements. `.eq(-2)` counts backward from the end, selecting the second-to-last `<div>`.

### Real-World Cases
- **Selecting a specific column in a table for styling**.
- **Accessing a particular item in a dynamically generated list**.
- **Highlighting the nth element in a search result set**.

### References
- .eq() – https://api.jquery.com/eq/
- :eq() Selector (Deprecated) – https://api.jquery.com/eq-selector/

---

## Core Concept 4: `.slice()` — Subset by Index Range

### Definitions

**Core Definition**
`.slice()` reduces the set of matched elements to a subset specified by a range of indices.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.slice()` method constructs a new jQuery object containing a subset of the elements specified by the `start` and, optionally, `end` argument. The supplied `start` index identifies the position of one of the elements in the set; if `end` is omitted, all elements after this one will be included in the result. The method is patterned after the JavaScript `.slice()` method for arrays, including support for negative indices that indicate offsets from the end of the set.

**Beginner-Friendly Explanation**
`.slice()` lets you cut out a section of your selection, like slicing a loaf of bread. `.slice(2, 4)` gives you elements 3 and 4 (because counting starts at 0). Negative numbers count from the end.

### Purposes

- To select a contiguous range of elements from a collection.
- To skip the first N elements and take the rest.
- To take only the first N elements.
- To serve as a performance-friendly alternative to the deprecated `:lt()` and `:gt()` selectors.

### Syntax Rules and Structure

```javascript
$(selector).slice(start)
$(selector).slice(start, end)
```

**Component Breakdown**
- `start` (Integer) : Zero-based position at which elements begin to be selected. If negative, it indicates an offset from the end.
- `end` (Integer, Optional) : Zero-based position at which elements stop being selected (exclusive). If negative, it indicates an offset from the end.
- Returns: A jQuery object containing the subset of elements.

**Syntax Rules**
1. The `start` index is **inclusive**; the `end` index is **exclusive**.
2. If `end` is omitted, the range extends to the end of the set.
3. Negative indices count from the end of the set.
4. Added in jQuery 1.1.4.

**Constraints and Limitations**
- The `end` index is not included in the result.
- If `start` is greater than the set length, an empty set is returned.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.slice() — Select a Range of Items</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4</li>
    <li>Item 5</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select items from index 2 onward
      $("li").slice(2).css("background-color", "red");
      // Items 3, 4, 5 are selected

      // Step 2: Select items from index 2 to 4 (exclusive)
      $("li").slice(2, 4).css("border", "2px solid green");
      // Items 3 and 4 are selected

      // Step 3: Use negative indices
      $("li").slice(-2, -1).css("font-weight", "bold");
      // Item 4 is selected (second from end, up to one from end)

      console.log("Slice(2,4) count:", $("li").slice(2, 4).length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 3, 4, and 5 receive a red background.
- Items 3 and 4 receive a green border.
- Item 4 becomes bold.
- Console output: `Slice(2,4) count: 2`

**Why This Output Occurs**
`.slice(2)` selects elements from index 2 to the end (items 3, 4, 5). `.slice(2, 4)` selects indices 2 and 3 (items 3 and 4), stopping before index 4. `.slice(-2, -1)` selects from two positions from the end up to one position from the end, which is only item 4.

### Real-World Cases
- **Pagination**: Showing a specific range of items per page.
- **Chunking large lists**: Breaking a large selection into smaller chunks for processing.
- **Skipping header rows**: `.slice(1)` skips the first row of a table and selects the rest.

### References
- .slice() – https://api.jquery.com/slice/
- :lt() Selector (Deprecated) – https://api.jquery.com/lt-selector/
- :gt() Selector (Deprecated) – https://api.jquery.com/gt-selector/

---

## Core Concept 5: `.filter()` — Elements Matching Selector or Test

### Definitions

**Core Definition**
`.filter()` reduces the set of matched elements to those that match a selector or pass a function's test.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.filter()` method constructs a new jQuery object from a subset of the matching elements. The supplied selector is tested against each element; all elements matching the selector will be included in the result. The second form allows filtering elements against a function rather than a selector. For each element, if the function returns `true` (or a "truthy" value), the element will be included in the filtered set; otherwise, it will be excluded. Within the filter function, `this` refers to each DOM element in turn. The method can also accept a DOM element or an existing jQuery object as its argument (added in jQuery 1.4).

**Beginner-Friendly Explanation**
`.filter()` is like a sieve. You give it a rule — either a selector like “.intro” or a custom function — and it keeps only the elements that match the rule, discarding the rest.

### Purposes

- To narrow a collection to elements that match a specific selector.
- To filter elements based on custom logic using a callback function.
- To remove elements that do not meet specific criteria from a selection.
- To serve as a powerful alternative to complex selector strings when custom logic is needed.

### Syntax Rules and Structure

```javascript
$(selector).filter(selector)
$(selector).filter(function(index))
$(selector).filter(element)
$(selector).filter(jQueryObject)
```

**Component Breakdown**
- `selector` : A string containing a selector expression to match the current set against.
- `function(index)` : A function used as a test for each element in the set. `this` is the current DOM element. The function receives the element's index as its argument.
- `element` : One or more DOM elements to match the current set against (added in jQuery 1.4).
- `jQueryObject` : An existing jQuery object to match the current set against (added in jQuery 1.4).
- Returns: A jQuery object containing the matching elements.

**Syntax Rules**
1. Elements matching the selector or passing the function's test are **included** in the result.
2. The callback function receives the element's index and should return `true` or `false`.
3. Added in jQuery 1.0; DOM element and jQuery object signatures added in 1.4.

**Constraints and Limitations**
- Function-based filtering is generally slower than selector-based filtering.
- jQuery extension selectors (e.g., `:even`) cannot use native `querySelectorAll()`.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.filter() — Selector and Function</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li><strong>list</strong> item 1 - one strong tag</li>
    <li><strong>list</strong> item <strong>2</strong> - two <span>strong tags</span></li>
    <li>list item 3</li>
    <li>list item 4</li>
    <li>list item 5</li>
    <li>list item 6</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Filter with a selector — only items with index 2n
      $("li").filter(":nth-child(2n)").css("background-color", "red");
      // Items 2, 4, 6 are selected

      // Step 2: Filter with a function — only items with exactly one <strong>
      $("li").filter(function (index) {
        return $("strong", this).length === 1;
      }).css("border", "2px solid blue");
      // Only the first list item is selected

      // Step 3: Verify
      console.log("Filtered by function:", $("li").filter(function () {
        return $("strong", this).length === 1;
      }).length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 2, 4, and 6 receive a red background.
- The first list item receives a blue border.
- Console output: `Filtered by function: 1`

**Why This Output Occurs**
`$("li").filter(":nth-child(2n)")` uses a jQuery extension selector to match even-indexed list items. The function-based filter tests each `<li>` for the number of `<strong>` descendants; only the first item contains exactly one `<strong>` tag.

### Real-World Cases
- **Search Filtering**: `$(".product").filter(":contains('sale')")` filters products containing the word “sale”.
- **Form Validation**: `$("input").filter(function() { return $(this).val() === ""; })` finds empty inputs.
- **Dynamic Lists**: Filtering items based on user-selected categories or tags.

### References
- .filter() – https://api.jquery.com/filter/
- :not() Selector – https://api.jquery.com/not-selector/

---

## Core Concept 6: `.not()` — Elements NOT Matching Selector or Test

### Definitions

**Core Definition**
`.not()` removes elements from the set of matched elements that match a selector, DOM element, or pass a function's test.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.not()` method constructs a new jQuery object from a subset of the matching elements. The supplied selector is tested against each element; the elements that **don't match** the selector will be included in the result. As of jQuery 1.4, the `.not()` method can take a function as its argument in the same way that `.filter()` does. Elements for which the function returns `true` are **excluded** from the filtered set; all other elements are included. Note: When a CSS selector string is passed to `.not()`, text and comment nodes will always be removed from the resulting jQuery object during the filtering process.

**Beginner-Friendly Explanation**
`.not()` is the opposite of `.filter()`. Instead of saying “keep only these,” you say “throw these away.” It removes elements that match your criteria and keeps everything else.

### Purposes

- To exclude elements that match a specific selector from a collection.
- To remove elements based on custom logic using a callback function.
- To select a broad set of elements and then filter out those matching a particular condition.
- To serve as a more readable alternative to complex `:not()` selector strings.

### Syntax Rules and Structure

```javascript
$(selector).not(selector)
$(selector).not(function(index))
$(selector).not(element)
$(selector).not(jQueryObject)
```

**Component Breakdown**
- `selector` : A string containing a selector expression to match elements against.
- `function(index, element)` : A function used as a test for each element. `this` is the current DOM element. Elements for which the function returns `true` are excluded.
- `element` : A DOM element to remove from the set.
- `jQueryObject` : An existing jQuery object whose elements should be removed from the set.
- Returns: A jQuery object containing the elements that do not match.

**Syntax Rules**
1. Elements matching the selector or passing the function's test are **excluded** from the result.
2. The callback function receives the element's index and the DOM element.
3. Added in jQuery 1.0; function signature added in 1.4.
4. Text and comment nodes are always removed when a CSS selector string is used.

**Constraints and Limitations**
- The `.not()` method is the opposite of `.filter()`.
- Function-based filtering is generally slower than selector-based filtering.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.not() — Remove Even Items</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>list item 1</li>
    <li>list item 2</li>
    <li>list item 3</li>
    <li>list item 4</li>
    <li>list item 5</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Remove even-indexed items
      $("li").not(":nth-child(2n)").css("background-color", "red");
      // Items 1, 3, 5 are selected (the ones NOT matching 2n)

      // Step 2: Remove a specific DOM element
      var thirdItem = document.getElementById("notli");
      // $("li").not(thirdItem).css("background-color", "red");

      // Step 3: Use a function to exclude items with a class
      $("li").not(function () {
        return $(this).hasClass("special");
      }).css("border", "2px solid blue");

      console.log("Not even count:", $("li").not(":nth-child(2n)").length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output**
- Items 1, 3, and 5 receive a red background.
- Items without the class “special” receive a blue border.
- Console output: `Not even count: 3`

**Why This Output Occurs**
`$("li").not(":nth-child(2n)")` excludes the even-indexed list items (2, 4) and keeps the odd-indexed ones (1, 3, 5). The function-based `.not()` excludes any `<li>` that has the class “special”.

### Real-World Cases
- **Form Field Exclusion**: `$("input").not("[type='hidden']")` selects all inputs except hidden fields.
- **Navigation Filtering**: `$("a").not("[href^='#']")` selects links that are not internal anchors.
- **List Cleanup**: Removing specific items from a list before processing.

### References
- .not() – https://api.jquery.com/not/
- :not() Selector – https://api.jquery.com/not-selector/

---

## Core Concept 7: `.has()` — Elements with Matching Descendant

### Definitions

**Core Definition**
`.has()` reduces the set of matched elements to those that have a descendant that matches a selector or DOM element.

**Technical Definition**
Given a jQuery object that represents a set of DOM elements, the `.has()` method constructs a new jQuery object from a subset of the matching elements. The supplied selector is tested against the **descendants** of the matching elements; the element will be included in the result if any of its descendant elements matches the selector. The method can also accept a DOM element as its argument (added in jQuery 1.4). The `.has()` method is **not** the same as the `:has()` selector; `.has()` is a method that filters the current set, while `:has()` is a selector that can be used within a selector string. The `:has()` selector is a jQuery extension and cannot take advantage of `querySelectorAll()`; for better performance, use `$( "your-pure-css-selector" ).has( selector/DOMElement )` instead.

**Beginner-Friendly Explanation**
`.has()` keeps only the elements that contain something inside them. If you have a list of `<div>` elements and you want only those that contain a `<p>`, `.has("p")` gives you exactly that.

### Purposes

- To select elements that contain a specific type of descendant.
- To filter containers based on their contents.
- To avoid traversing into descendants when only the container is needed.
- To serve as a performance-friendly alternative to the `:has()` selector.

### Syntax Rules and Structure

```javascript
$(selector).has(selector)
$(selector).has(containedElement)
```

**Component Breakdown**
- `selector` (String) : A selector expression to match against descendants.
- `containedElement` (Element) : A DOM element to match against descendants.
- Returns: A jQuery object containing the elements that have a matching descendant.

**Syntax Rules**
1. Tests **descendants** (at any depth) for a match.
2. Returns the **container** elements, not the descendants themselves.
3. Added in jQuery 1.4.
4. The `:has()` selector is a jQuery extension; use the `.has()` method for better performance.

**Constraints and Limitations**
- The `:has()` selector cannot use native `querySelectorAll()`.
- Performance depends on the depth and breadth of the descendant search.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.has() — Select List Items Containing a Nested List</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .full { border: 1px solid red; }
  </style>
</head>
<body>
  <ul>
    <li>list item 1</li>
    <li>list item 2
      <ul>
        <li>list item 2-a</li>
        <li>list item 2-b</li>
      </ul>
    </li>
    <li>list item 3</li>
    <li>list item 4</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select list items that contain a nested <ul>
      $("li").has("ul").css("background-color", "red");

      // Step 2: Check if a UL contains an LI
      $("ul").append("<li>" +
        ($("ul").has("li").length ? "Yes" : "No") +
        "</li>");

      // Step 3: Add a class to all ULs that have LIs
      $("ul").has("li").addClass("full");

      console.log("Items with nested ul:", $("li").has("ul").length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- The second list item (which contains a nested `<ul>`) receives a red background.
- A new list item “Yes” is appended to the `<ul>`, confirming it contains `<li>` elements.
- Console output: `Items with nested ul: 1`

**Why This Output Occurs**
`$("li").has("ul")` tests each `<li>` for a descendant `<ul>`. Only the second list item contains a nested list, so only it is selected and styled. The second call `$("ul").has("li")` checks whether any `<ul>` contains `<li>` descendants, which is true, so “Yes” is appended.

### Real-World Cases
- **Table Filtering**: `$("tr").has("td.error")` selects rows that contain error cells.
- **Form Section Detection**: `$("fieldset").has("input:required")` finds fieldsets with required fields.
- **Navigation Menus**: `$("li").has("ul")` selects menu items that have dropdown sub-menus.
- **Content Sections**: `$("section").has("img")` finds sections that contain images.

### References
- .has() – https://api.jquery.com/has/
- :has() Selector – https://api.jquery.com/has-selector/

---

## Core Concept 8: `.is()` — Boolean Match Check

### Definitions

**Core Definition**
`.is()` checks the current matched set of elements against a selector, element, or jQuery object and returns `true` if at least one element matches.

**Technical Definition**
Unlike other filtering methods, `.is()` does **not** create a new jQuery object. Instead, it allows us to test the contents of a jQuery object without modification. It returns a **Boolean** value (`true` or `false`). The method accepts a selector expression, a DOM element, or a jQuery object as its argument. It also accepts a function (added in jQuery 1.6); for each element, if the function returns `true`, `.is()` returns `true` as well. This is useful for testing whether an element has a particular class, is hidden, or matches any other selector.

**Beginner-Friendly Explanation**
`.is()` is a yes/no question. It asks: “Does at least one of these elements match this description?” You can ask things like “Is this element hidden?” or “Does this element have the class 'active'?” It always gives you `true` or `false`, not a collection of elements.

### Purposes

- To test whether an element matches a selector without modifying the collection.
- To check element state (e.g., visible, hidden, checked, disabled).
- To validate conditions before performing an action (e.g., in an `if` statement).
- To check whether a jQuery object contains a specific DOM element.

### Syntax Rules and Structure

```javascript
$(selector).is(selector)
$(selector).is(function(index, element))
$(selector).is(element)
$(selector).is(jQueryObject)
```

**Component Breakdown**
- `selector` : A selector expression to match the current set against.
- `function(index, element)` : A function used as a test. For each element, if the function returns `true`, `.is()` returns `true`.
- `element` : A DOM element to match against.
- `jQueryObject` : An existing jQuery object to match against.
- Returns: **Boolean** (`true` if at least one element matches; otherwise `false`).

**Syntax Rules**
1. Returns a **Boolean**, not a jQuery object.
2. Does **not** modify the original collection.
3. If the set is empty, `.is()` returns `false`.
4. Added in jQuery 1.0; function signature added in 1.6.

**Constraints and Limitations**
- Returns `true` if **at least one** element matches, not if all elements match.
- Performance depends on the specificity of the selector; broad selectors are slower.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.is() — Check Element State</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="myDiv" class="pretty awesome">Content</div>
  <div id="hiddenDiv" style="display:none;">Hidden</div>
  <input type="checkbox" id="myCheck" checked>
  <p id="output"></p>

  <script>
    $(function () {
      // Step 1: Check if #myDiv has the class "pretty"
      var hasPretty = $("#myDiv").is(".pretty");
      console.log("Has 'pretty' class:", hasPretty); // true

      // Step 2: Check if #hiddenDiv is hidden
      var isHidden = $("#hiddenDiv").is(":hidden");
      console.log("Hidden div is hidden:", isHidden); // true

      // Step 3: Check if the checkbox is checked
      var isChecked = $("#myCheck").is(":checked");
      console.log("Checkbox is checked:", isChecked); // true

      // Step 4: Use in a conditional
      if ($("#myDiv").is(".pretty.awesome")) {
        $("#output").text("It's pretty and awesome!");
      }

      // Step 5: Check multiple conditions
      console.log("Has either class:",
        $("#myDiv").is(".pretty, .nonexistent")); // true
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#output` div displays “It's pretty and awesome!”.
- Console output:
```
Has 'pretty' class: true
Hidden div is hidden: true
Checkbox is checked: true
Has either class: true
```

**Why This Output Occurs**
`.is(".pretty")` checks whether `#myDiv` matches the `.pretty` selector and returns `true`. `.is(":hidden")` checks the hidden div’s visibility and returns `true`. `.is(":checked")` checks the checkbox’s state. The multiple selector `.pretty, .nonexistent` returns `true` because at least one selector matches.

### Real-World Cases
- **Form Validation**: `if ($("input").is(":invalid")) { ... }` checks for invalid fields.
- **Navigation State**: `if ($("nav li").is(".active")) { ... }` checks for active menu items.
- **Visibility Checks**: `if ($("#modal").is(":visible")) { $("#modal").fadeOut(); }`.
- **Element Type Detection**: `if ($(this).is("button")) { ... }` checks whether a clicked element is a button.

### References
- .is() – https://api.jquery.com/is/
- :hidden Selector – https://api.jquery.com/hidden-selector/
- :visible Selector – https://api.jquery.com/visible-selector/

---

## Summary Table: jQuery Filtering Methods at a Glance

| Method | Return Type | Accepts | Includes/Excludes | Added | Key Use |
|---|---|---|---|---|---|
| `.first()` | jQuery object | No args | Includes first | 1.4 | Get first element |
| `.last()` | jQuery object | No args | Includes last | 1.4 | Get last element |
| `.eq()` | jQuery object | Integer index | Includes one | 1.1.2 | Get element at index |
| `.slice()` | jQuery object | start, [end] | Includes range | 1.1.4 | Get subset range |
| `.filter()` | jQuery object | Selector, fn, element, jQuery | **Includes** matches | 1.0 | Keep matching elements |
| `.not()` | jQuery object | Selector, fn, element, jQuery | **Excludes** matches | 1.0 | Remove matching elements |
| `.has()` | jQuery object | Selector, element | Includes containers | 1.4 | Elements with descendants |
| `.is()` | **Boolean** | Selector, fn, element, jQuery | N/A (test only) | 1.0 | Check for a match |

### Key Differences

| Aspect | `.filter()` | `.not()` | `.has()` | `.is()` |
|---|---|---|---|---|
| Purpose | Keep matching | Remove matching | Keep containers | Test match |
| Return | jQuery object | jQuery object | jQuery object | Boolean |
| Tests | The element itself | The element itself | The element's descendants | The element itself |
| Function support | Yes | Yes | No | Yes (1.6+) |

---

## General References

- Filtering | jQuery API Documentation – https://api.jquery.com/category/traversing/filtering/
- .first() – https://api.jquery.com/first/
- .last() – https://api.jquery.com/last/
- .eq() – https://api.jquery.com/eq/
- .slice() – https://api.jquery.com/slice/
- .filter() – https://api.jquery.com/filter/
- .not() – https://api.jquery.com/not/
- .has() – https://api.jquery.com/has/
- .is() – https://api.jquery.com/is/
- jQuery Learning Center — Traversing – https://learn.jquery.com/using-jquery-core/traversing/
- W3Schools — jQuery Traversing Filtering – https://www.w3schools.com/jquery/jquery_traversing_filtering.asp
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/
- jQuery Performance Best Practices – https://api.jquery.com/category/selectors/