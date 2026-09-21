# jQuery Positional and Filtering Selectors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery positional and filtering selectors are expressions that filter a matched set of elements based on their index or position within the collection, or based on their position relative to sibling elements in the DOM tree.

**Technical Definition**
Positional and filtering selectors are appended to a base selector (e.g., `$("li:first")`) and narrow the matched set using either zero-based index filtering (jQuery extensions) or one-based child position filtering (CSS-compliant selectors). The index-based selectors (`:first`, `:last`, `:eq()`, `:lt()`, `:gt()`, `:even`, `:odd`) are **jQuery extensions** that filter the set of elements that have matched the preceding expressions. The child-position selectors (`:first-child`, `:last-child`, `:nth-child()`) are **CSS Selectors Level 3** standards that select elements based on their position among their parent's children.

**Important Deprecation Notice**: As of jQuery 3.4, the index-based pseudo-classes — `:first`, `:last`, `:eq()`, `:lt()`, `:gt()`, `:even`, and `:odd` — are **deprecated**. The official documentation recommends removing them from selectors and filtering results later using the corresponding methods: `.first()`, `.last()`, `.eq()`, `.slice()`, `.even()`, and `.odd()`. The child-position selectors (`:first-child`, `:last-child`, `:nth-child()`) are **not deprecated** and remain part of the CSS specification.

**Beginner-Friendly Explanation**
When you select elements with jQuery, you often get more elements than you need. Positional and filtering selectors let you narrow down the list. Some selectors work like a numbered list: ":first" picks the first item, ":eq(2)" picks the third item (because counting starts at 0), and ":even" picks every other item. Other selectors work by looking at the HTML structure: ":first-child" picks elements that are the first child of their parent, and ":nth-child(2)" picks the second child of each parent. The difference is that index-based selectors look at the whole collection jQuery found, while child-position selectors look at each parent's children separately.

### Key Characteristics

- **Two Distinct Categories**: Index-based selectors filter the jQuery collection (0-based), while child-position selectors filter based on DOM structure (1-based for CSS compliance).
- **Deprecation Status**: All index-based pseudo-selectors are deprecated in jQuery 3.4; child-position selectors are not deprecated.
- **Performance Consideration**: jQuery extension selectors cannot use native `querySelectorAll()`, making them slower. The recommended approach is to use pure CSS selectors followed by filtering methods.
- **Method Equivalents**: Each deprecated pseudo-selector has a modern method equivalent that performs better.
- **Composability**: These selectors can be combined with element, class, attribute, and hierarchical selectors.

### Prerequisites

- Basic understanding of jQuery selectors (element, ID, class, attribute).
- Familiarity with the DOM tree structure (parents, children, siblings).
- Awareness of zero-based vs one-based indexing.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **CSS Pseudo-Classes**: `:first-child`, `:last-child`, and `:nth-child()` are standard CSS selectors.
- **Array Indexing**: Index-based selectors mirror JavaScript array behaviour (0-based).
- **DOM Traversal Methods**: `.first()`, `.last()`, `.eq()`, `.slice()`, `.children()` provide programmatic alternatives.
- **Table Styling**: `:even` and `:odd` are commonly used for zebra-striping table rows.

### Core Concepts / Features

1. `:first` — First Matched Element
2. `:last` — Last Matched Element
3. `:eq()` — Element at Specific Index
4. `:lt()` — Elements Before an Index
5. `:gt()` — Elements After an Index
6. `:even` — Even-Indexed Elements
7. `:odd` — Odd-Indexed Elements
8. `:first-child` — First Child of Parent
9. `:last-child` — Last Child of Parent
10. `:nth-child()` — Nth Child of Parent

---

## Core Concept 1: `:first` — First Matched Element

### Definitions

**Core Definition**
The `:first` selector selects the first element in the matched set.

**Technical Definition**
`jQuery(":first")` filters the preceding selector's results to the single first element in the collection. It is equivalent to `:eq(0)` and `:lt(1)`. While this matches only a single element, `:first-child` can match more than one (one for each parent). As of jQuery 3.4, `:first` is deprecated; use `.first()` instead.

**Beginner-Friendly Explanation**
`:first` picks the very first element from everything you selected. If you select all paragraphs and add `:first`, you get only the first paragraph on the page. Only one element is ever returned.

### Purposes

- To select the first element from a matched set for targeted manipulation.
- To apply a special style or behaviour to only the first item in a list.
- To serve as an alternative to `.first()` when writing selector strings.
- To reduce a large collection to a single element for focused operations.

### Syntax Rules and Structure

```javascript
$("selector:first")
```

**Component Breakdown**
- `"selector:first"` : A valid base selector followed by `:first`.
- Returns: A jQuery object containing exactly one element (or zero if the set is empty).

**Syntax Rules**
1. The `:first` pseudo-class must be appended to a base selector: `$("p:first")`.
2. It selects only the first element in the entire matched set, not the first child of each parent.
3. Equivalent to `:eq(0)` and `:lt(1)`.
4. **Deprecated in jQuery 3.4**: Use `.first()` method instead.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`; slower than method alternatives.
- Only returns one element regardless of how many parents exist.
- Use `:first-child` when you need the first child of each parent.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:first Selector — Table Row Highlight</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <table border="1">
    <tr><td>First Row</td></tr>
    <tr><td>Middle Row</td></tr>
    <tr><td>Last Row</td></tr>
  </table>

  <script>
    $(function () {
      // Step 1: Select the first table row using :first
      $("tr:first").css("background-color", "yellow");

      // Step 2: Verify only one row was selected
      console.log("Rows selected:", $("tr:first").length); // 1

      // Step 3: Modern alternative using .first()
      // $("tr").first().css("background-color", "yellow");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first table row receives a yellow background.
- Console output: `Rows selected: 1`

**Why This Output Occurs**
`$("tr:first")` filters all `<tr>` elements to the first one in the collection. Only the first row is selected and styled.

### Real-World Cases
- Highlighting the first row of a data table.
- Styling the first item in a navigation list differently.
- Selecting the first image in a gallery for a featured display.

### References
- :first Selector – https://api.jquery.com/first-selector/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 2: `:last` — Last Matched Element

### Definitions

**Core Definition**
The `:last` selector selects the last element in the matched set.

**Technical Definition**
`jQuery(":last")` filters the preceding selector's results to the single last element in the collection. It selects a single element by filtering the current jQuery collection and matching the last element within it. As of jQuery 3.4, `:last` is deprecated; use `.last()` instead.

**Beginner-Friendly Explanation**
`:last` picks the very last element from everything you selected. If you select all list items and add `:last`, you get only the final list item.

### Purposes

- To select the last element from a matched set for targeted manipulation.
- To apply a special style or behaviour to only the last item in a list.
- To serve as an alternative to `.last()` when writing selector strings.
- To reduce a collection to a single element at the end.

### Syntax Rules and Structure

```javascript
$("selector:last")
```

**Component Breakdown**
- `"selector:last"` : A valid base selector followed by `:last`.
- Returns: A jQuery object containing exactly one element (or zero if the set is empty).

**Syntax Rules**
1. The `:last` pseudo-class must be appended to a base selector: `$("li:last")`.
2. It selects only the last element in the entire matched set, not the last child of each parent.
3. **Deprecated in jQuery 3.4**: Use `.last()` method instead.
4. Use `:last-child` when you need the last child of each parent.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`; slower than method alternatives.
- Only returns one element regardless of how many parents exist.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:last Selector — Last List Item</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select the last list item using :last
      $("li:last").css("font-weight", "bold");

      // Step 2: Modern alternative using .last()
      // $("li").last().css("font-weight", "bold");

      console.log("Last item:", $("li:last").text()); // "Item 3"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The last list item (“Item 3”) becomes bold.
- Console output: `Last item: Item 3`

**Why This Output Occurs**
`$("li:last")` filters all `<li>` elements to the last one in the collection, which is “Item 3”.

### Real-World Cases
- Styling the last item in a breadcrumb navigation.
- Adding a “last” class to the final element in a list.
- Selecting the most recently added element in a dynamic list.

### References
- :last Selector – https://api.jquery.com/last-selector/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 3: `:eq()` — Element at Specific Index

### Definitions

**Core Definition**
The `:eq()` selector selects the element at a specific zero-based index within the matched set.

**Technical Definition**
`jQuery(":eq(index)")` selects the element at index `index` within the matched set. A negative index counts backward from the last element (added in jQuery 1.8). Because JavaScript arrays use 0-based indexing, `$(".myclass:eq(1)")` selects the second element with class `myclass`. As of jQuery 3.4, `:eq()` is deprecated; use `.eq()` instead.

**Beginner-Friendly Explanation**
`:eq()` is like array indexing. `:eq(0)` gives you the first element, `:eq(1)` gives the second, and so on. Negative numbers count from the end: `:eq(-1)` is the last element.

### Purposes

- To select a single element at a known position within a matched set.
- To access elements by their zero-based index in a collection.
- To select elements counting backward from the end using negative indices.
- To serve as an alternative to `.eq()` when writing selector strings.

### Syntax Rules and Structure

```javascript
$("selector:eq(index)")
$("selector:eq(-index)")
```

**Component Breakdown**
- `:eq(index)` : Zero-based index of the element to match.
- `:eq(-index)` : Zero-based index counting backwards from the last element (added in jQuery 1.8).
- Returns: A jQuery object containing zero or one element.

**Syntax Rules**
1. Index is **zero-based**: `:eq(0)` is the first element, `:eq(1)` is the second.
2. Negative indices count from the end: `:eq(-1)` is the last element.
3. **Deprecated in jQuery 3.4**: Use `.eq()` method instead.
4. In contrast, `:nth-child(n)` uses **1-based indexing** to conform to the CSS specification.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`.
- Only selects a single element; use `:nth-child()` to select the nth child of each parent.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:eq() Selector — Table Cell Highlight</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <table border="1">
    <tr><td>TD #0</td><td>TD #1</td><td>TD #2</td></tr>
    <tr><td>TD #3</td><td>TD #4</td><td>TD #5</td></tr>
  </table>

  <script>
    $(function () {
      // Step 1: Select the third td (index 2) using :eq()
      $("td:eq(2)").css("color", "red");

      // Step 2: Select the last td using a negative index
      $("td:eq(-1)").css("background-color", "lightblue");

      // Step 3: Modern alternative
      // $("td").eq(2).css("color", "red");

      console.log("TD #2 text:", $("td:eq(2)").text()); // "TD #2"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The third `<td>` (“TD #2”) turns red.
- The last `<td>` (“TD #5”) receives a light blue background.
- Console output: `TD #2 text: TD #2`

**Why This Output Occurs**
`$("td:eq(2)")` selects the element at index 2 (the third `<td>`) from the collection of all `<td>` elements. `$("td:eq(-1)")` counts backward from the end, selecting the last `<td>`.

### Real-World Cases
- Selecting a specific column in a table for styling.
- Accessing a particular item in a dynamically generated list.
- Highlighting the nth element in a search result set.

### References
- :eq() Selector – https://api.jquery.com/eq-selector/
- .eq() Method – https://api.jquery.com/eq/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 4: `:lt()` — Elements Before an Index

### Definitions

**Core Definition**
The `:lt()` selector selects all elements at an index **less than** the specified index within the matched set.

**Technical Definition**
`jQuery(":lt(index)")` selects all elements whose zero-based index is less than `index`. A negative index counts backward from the last element (added in jQuery 1.8). `:lt(3)` can be replaced with a call to `.slice(0, 3)`. As of jQuery 3.4, `:lt()` is deprecated; use `.slice()` instead.

**Beginner-Friendly Explanation**
`:lt()` means “less than.” `:lt(3)` selects elements with indices 0, 1, and 2 — the first three elements.

### Purposes

- To select all elements before a certain position in a collection.
- To apply styles or behaviours to a subset of elements from the beginning.
- To serve as an alternative to `.slice(0, index)` when writing selector strings.
- To create “top N” selections from a larger set.

### Syntax Rules and Structure

```javascript
$("selector:lt(index)")
$("selector:lt(-index)")
```

**Component Breakdown**
- `:lt(index)` : Zero-based index; selects elements with index < `index`.
- `:lt(-index)` : Zero-based index counting backwards from the last element.
- Returns: A jQuery object containing zero or more elements.

**Syntax Rules**
1. Index is **zero-based**: `:lt(3)` selects indices 0, 1, and 2.
2. Negative indices count from the end: `:lt(-2)` selects all elements except the last two.
3. **Deprecated in jQuery 3.4**: Use `.slice(0, index)` instead.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`.
- Negative indices were added in jQuery 1.8.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:lt() Selector — First Three Items</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 0</li>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select the first three items (indices 0, 1, 2)
      $("li:lt(3)").css("background-color", "lightgreen");

      // Step 2: Modern alternative using .slice()
      // $("li").slice(0, 3).css("background-color", "lightgreen");

      console.log("Items before index 3:", $("li:lt(3)").length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first three list items (Item 0, Item 1, Item 2) receive a light green background.
- The last two items remain unchanged.
- Console output: `Items before index 3: 3`

**Why This Output Occurs**
`$("li:lt(3)")` selects all `<li>` elements with an index less than 3, which includes indices 0, 1, and 2.

### Real-World Cases
- Styling the first N items in a “top results” list.
- Applying special formatting to elements before a certain point.
- Creating “recent items” sections from a longer list.

### References
- :lt() Selector – https://api.jquery.com/lt-selector/
- .slice() Method – https://api.jquery.com/slice/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 5: `:gt()` — Elements After an Index

### Definitions

**Core Definition**
The `:gt()` selector selects all elements at an index **greater than** the specified index within the matched set.

**Technical Definition**
`jQuery(":gt(index)")` selects all elements whose zero-based index is greater than `index`. A negative index counts backward from the last element (added in jQuery 1.8). `:gt(3)` can be replaced with a call to `.slice(4)` (the provided index needs to be increased by one). As of jQuery 3.4, `:gt()` is deprecated; use `.slice()` instead.

**Beginner-Friendly Explanation**
`:gt()` means “greater than.” `:gt(2)` selects elements with indices 3, 4, 5, and so on — everything after the first three elements.

### Purposes

- To select all elements after a certain position in a collection.
- To apply styles or behaviours to a subset of elements from the end of a set.
- To skip the first N elements and operate on the rest.
- To serve as an alternative to `.slice(index + 1)` when writing selector strings.

### Syntax Rules and Structure

```javascript
$("selector:gt(index)")
$("selector:gt(-index)")
```

**Component Breakdown**
- `:gt(index)` : Zero-based index; selects elements with index > `index`.
- `:gt(-index)` : Zero-based index counting backwards from the last element.
- Returns: A jQuery object containing zero or more elements.

**Syntax Rules**
1. Index is **zero-based**: `:gt(2)` selects indices 3, 4, 5, etc.
2. Negative indices count from the end: `:gt(-3)` selects all elements except the last two.
3. **Deprecated in jQuery 3.4**: Use `.slice(index + 1)` instead.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`.
- Negative indices were added in jQuery 1.8.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:gt() Selector — Skip First Three</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 0</li>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Select items after index 2 (indices 3 and 4)
      $("li:gt(2)").css("background-color", "lightyellow");

      // Step 2: Modern alternative using .slice()
      // $("li").slice(3).css("background-color", "lightyellow");

      console.log("Items after index 2:", $("li:gt(2)").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- The last two list items (Item 3 and Item 4) receive a light yellow background.
- The first three items remain unchanged.
- Console output: `Items after index 2: 2`

**Why This Output Occurs**
`$("li:gt(2)")` selects all `<li>` elements with an index greater than 2, which includes indices 3 and 4.

### Real-World Cases
- Skipping the first few rows in a table and styling the rest.
- Selecting “remaining” items after a certain point in a list.
- Applying different formatting to items beyond a threshold.

### References
- :gt() Selector – https://api.jquery.com/gt-selector/
- .slice() Method – https://api.jquery.com/slice/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 6: `:even` — Even-Indexed Elements

### Definitions

**Core Definition**
The `:even` selector selects all elements with an even index (0, 2, 4, etc.) within the matched set.

**Technical Definition**
`jQuery(":even")` selects elements at even indices, zero-indexed. Counter-intuitively, `:even` selects the first element, third element, and so on within the matched set because indexing starts at 0. As of jQuery 3.4, `:even` is deprecated; use `.even()` (available in jQuery 3.5.0 or newer) or `.filter(":even")` instead.

**Beginner-Friendly Explanation**
`:even` does not mean “the 2nd, 4th, 6th elements.” Because counting starts at 0, it means the 1st, 3rd, 5th elements — the elements at positions 0, 2, 4, and so on.

### Purposes

- To create zebra-striping effects in tables by selecting alternating rows.
- To apply styles to every other element in a collection.
- To serve as an alternative to `.filter(":even")` or `.even()` when writing selector strings.

### Syntax Rules and Structure

```javascript
$("selector:even")
```

**Component Breakdown**
- `:even` : No arguments; selects elements at even zero-based indices.
- Returns: A jQuery object containing elements at indices 0, 2, 4, etc.

**Syntax Rules**
1. Indexing is **zero-based**: `:even` selects indices 0, 2, 4, etc.
2. This means the **first** element is selected (index 0 is even).
3. **Deprecated in jQuery 3.4**: Use `.even()` (jQuery 3.5.0+) or `.filter(":even")`.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`.
- Counter-intuitive for beginners: “even” selects the 1st, 3rd, 5th elements.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:even Selector — Zebra Striping</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <table border="1">
    <tr><td>Row 0 (even)</td></tr>
    <tr><td>Row 1 (odd)</td></tr>
    <tr><td>Row 2 (even)</td></tr>
    <tr><td>Row 3 (odd)</td></tr>
  </table>

  <script>
    $(function () {
      // Step 1: Select even-indexed rows (0 and 2)
      $("tr:even").css("background-color", "#f0f0f0");

      // Step 2: Modern alternative
      // $("tr").filter(":even").css("background-color", "#f0f0f0");

      console.log("Even rows:", $("tr:even").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- Rows 0 and 2 (the 1st and 3rd rows) receive a light gray background.
- Console output: `Even rows: 2`

**Why This Output Occurs**
`$("tr:even")` selects `<tr>` elements at even indices (0 and 2). Because indexing starts at 0, this means the first and third rows.

### Real-World Cases
- Zebra-striping table rows for improved readability.
- Applying alternating background colours to list items.
- Selecting every other element in a grid layout.

### References
- :even Selector – https://api.jquery.com/even-selector/
- .even() Method – https://api.jquery.com/even/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 7: `:odd` — Odd-Indexed Elements

### Definitions

**Core Definition**
The `:odd` selector selects all elements with an odd index (1, 3, 5, etc.) within the matched set.

**Technical Definition**
`jQuery(":odd")` selects elements at odd indices, zero-indexed. Because indexing starts at 0, `:odd` selects the second element, fourth element, and so on within the matched set. As of jQuery 3.4, `:odd` is deprecated; use `.odd()` (available in jQuery 3.5.0 or newer) or `.filter(":odd")` instead.

**Beginner-Friendly Explanation**
`:odd` means the 2nd, 4th, 6th elements — the elements at positions 1, 3, 5, and so on, because counting starts at 0.

### Purposes

- To create zebra-striping effects in tables by selecting alternating rows (the complement of `:even`).
- To apply styles to every other element in a collection.
- To serve as an alternative to `.filter(":odd")` or `.odd()` when writing selector strings.

### Syntax Rules and Structure

```javascript
$("selector:odd")
```

**Component Breakdown**
- `:odd` : No arguments; selects elements at odd zero-based indices.
- Returns: A jQuery object containing elements at indices 1, 3, 5, etc.

**Syntax Rules**
1. Indexing is **zero-based**: `:odd` selects indices 1, 3, 5, etc.
2. This means the **second** element is selected (index 1 is odd).
3. **Deprecated in jQuery 3.4**: Use `.odd()` (jQuery 3.5.0+) or `.filter(":odd")`.

**Constraints and Limitations**
- Cannot use native `querySelectorAll()`.
- Counter-intuitive for beginners: “odd” selects the 2nd, 4th, 6th elements.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:odd Selector — Zebra Striping Complement</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <table border="1">
    <tr><td>Row 0 (even)</td></tr>
    <tr><td>Row 1 (odd)</td></tr>
    <tr><td>Row 2 (even)</td></tr>
    <tr><td>Row 3 (odd)</td></tr>
  </table>

  <script>
    $(function () {
      // Step 1: Select odd-indexed rows (1 and 3)
      $("tr:odd").css("background-color", "#e0e0e0");

      // Step 2: Modern alternative
      // $("tr").filter(":odd").css("background-color", "#e0e0e0");

      console.log("Odd rows:", $("tr:odd").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- Rows 1 and 3 (the 2nd and 4th rows) receive a slightly darker gray background.
- Console output: `Odd rows: 2`

**Why This Output Occurs**
`$("tr:odd")` selects `<tr>` elements at odd indices (1 and 3). Because indexing starts at 0, this means the second and fourth rows.

### Real-World Cases
- Completing zebra-striping patterns alongside `:even`.
- Applying alternating styles to every other element.
- Selecting the complement of `:even` selections.

### References
- :odd Selector – https://api.jquery.com/odd-selector/
- .odd() Method – https://api.jquery.com/odd/
- Deprecated 3.4 – https://api.jquery.com/category/deprecated/deprecated-3.4/

---

## Core Concept 8: `:first-child` — First Child of Parent

### Definitions

**Core Definition**
The `:first-child` selector selects all elements that are the first child of their parent.

**Technical Definition**
`jQuery(":first-child")` matches all elements that are the first child of their parent. While `.first()` matches only a single element, `:first-child` can match more than one: one for each parent. This is equivalent to `:nth-child(1)`. Unlike the deprecated index-based selectors, `:first-child` is **not deprecated** and is part of the CSS specification.

**Beginner-Friendly Explanation**
`:first-child` looks at each parent and finds its first child. If you have three lists, `:first-child` finds the first item in each list — three elements in total. This is different from `:first`, which finds only the very first element overall.

### Purposes

- To select the first child of every parent element in the document.
- To apply styles to the first item in each list, each container, or each section.
- To work with the CSS specification-compliant selector that is not deprecated.
- To target structural elements based on their position among siblings.

### Syntax Rules and Structure

```javascript
$("selector:first-child")
```

**Component Breakdown**
- `:first-child` : No arguments; matches elements that are the first child of their parent.
- Returns: A jQuery object containing one element per parent (that matches the base selector).

**Syntax Rules**
1. Matches elements that are the **first child** of their parent, regardless of element type.
2. Can match **multiple elements** — one for each parent.
3. Equivalent to `:nth-child(1)`.
4. **Not deprecated** — part of the CSS Selectors Level 3 specification.

**Constraints and Limitations**
- The element must be the first **child node**, not just the first element of its type. Text nodes and comments are not counted as children for this selector.
- Use `:first-of-type` if you need the first element of a specific type among siblings.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:first-child Selector — First Span in Each Div</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <span>First span in div 1</span>
    <span>Second span in div 1</span>
  </div>
  <div>
    <span>First span in div 2</span>
    <span>Second span in div 2</span>
  </div>

  <script>
    $(function () {
      // Step 1: Select the first span in each div
      $("div span:first-child").css("font-weight", "bold");

      console.log("First spans:", $("div span:first-child").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first span in each div becomes bold.
- Console output: `First spans: 2`

**Why This Output Occurs**
`$("div span:first-child")` selects `<span>` elements that are the first child of a `<div>`. Because there are two divs, two spans are selected — one per parent.

### Real-World Cases
- Styling the first item in each navigation list differently.
- Applying a top border to the first row of each table section.
- Selecting the first paragraph in each article section.

### References
- :first-child Selector – https://api.jquery.com/first-child-selector/
- Child Filter | jQuery API Documentation – https://api.jquery.com/category/selectors/child-filter-selectors/
- MDN Web Docs — :first-child – https://developer.mozilla.org/en-US/docs/Web/CSS/:first-child

---

## Core Concept 9: `:last-child` — Last Child of Parent

### Definitions

**Core Definition**
The `:last-child` selector selects all elements that are the last child of their parent.

**Technical Definition**
`jQuery(":last-child")` matches all elements that are the last child of their parent. While `.last()` matches only a single element, `:last-child` can match more than one: one for each parent. This selector is **not deprecated** and is part of the CSS specification.

**Beginner-Friendly Explanation**
`:last-child` looks at each parent and finds its last child. If you have three lists, `:last-child` finds the last item in each list — three elements in total.

### Purposes

- To select the last child of every parent element in the document.
- To apply styles to the last item in each list, each container, or each section.
- To work with the CSS specification-compliant selector that is not deprecated.
- To target structural elements based on their position among siblings.

### Syntax Rules and Structure

```javascript
$("selector:last-child")
```

**Component Breakdown**
- `:last-child` : No arguments; matches elements that are the last child of their parent.
- Returns: A jQuery object containing one element per parent (that matches the base selector).

**Syntax Rules**
1. Matches elements that are the **last child** of their parent, regardless of element type.
2. Can match **multiple elements** — one for each parent.
3. **Not deprecated** — part of the CSS Selectors Level 3 specification.

**Constraints and Limitations**
- The element must be the last **child node**, not just the last element of its type.
- Use `:last-of-type` if you need the last element of a specific type among siblings.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:last-child Selector — Last Span in Each Div</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <span>First span in div 1</span>
    <span>Last span in div 1</span>
  </div>
  <div>
    <span>First span in div 2</span>
    <span>Last span in div 2</span>
  </div>

  <script>
    $(function () {
      // Step 1: Select the last span in each div
      $("div span:last-child").css("color", "red");

      console.log("Last spans:", $("div span:last-child").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- The last span in each div turns red.
- Console output: `Last spans: 2`

**Why This Output Occurs**
`$("div span:last-child")` selects `<span>` elements that are the last child of a `<div>`. Because there are two divs, two spans are selected.

### Real-World Cases
- Styling the last item in each navigation group.
- Adding a bottom border to the last row of each table section.
- Removing margin from the last paragraph in each section.

### References
- :last-child Selector – https://api.jquery.com/last-child-selector/
- Child Filter | jQuery API Documentation – https://api.jquery.com/category/selectors/child-filter-selectors/
- MDN Web Docs — :last-child – https://developer.mozilla.org/en-US/docs/Web/CSS/:last-child

---

## Core Concept 10: `:nth-child()` — Nth Child of Parent

### Definitions

**Core Definition**
The `:nth-child()` selector selects all elements that are the nth child of their parent, using 1-based indexing as specified by CSS.

**Technical Definition**
`jQuery(":nth-child(index/even/odd/equation)")` selects all elements that are the nth child of their parent. Because jQuery's implementation of `:nth-` selectors is strictly derived from the CSS specification, the value of `n` is **1-indexed**, meaning counting starts at 1. The argument can be a number, the strings `even` or `odd`, or an equation (e.g., `4n`). The `:nth-child(n)` pseudo-class is easily confused with `.eq(n)`, but they can result in dramatically different matched elements.

**Beginner-Friendly Explanation**
`:nth-child()` looks at each parent and picks its nth child. `:nth-child(1)` picks the first child of each parent. `:nth-child(2)` picks the second child of each parent. You can also use formulas: `:nth-child(even)` picks even-numbered children, and `:nth-child(3n)` picks every third child.

### Purposes

- To select elements based on their position among their parent's children.
- To create complex alternating patterns using formulas (e.g., `3n+1`).
- To style specific columns or positions within each parent.
- To use the CSS specification-compliant selector that is not deprecated.

### Syntax Rules and Structure

```javascript
$("selector:nth-child(index)")
$("selector:nth-child(even)")
$("selector:nth-child(odd)")
$("selector:nth-child(equation)")
```

**Component Breakdown**
- `:nth-child(index)` : 1-based index of the child to match.
- `:nth-child(even)` : Selects even-numbered children (2nd, 4th, etc.).
- `:nth-child(odd)` : Selects odd-numbered children (1st, 3rd, etc.).
- `:nth-child(equation)` : An equation such as `4n` or `3n+1`.
- Returns: A jQuery object containing matched elements across all parents.

**Syntax Rules**
1. Indexing is **1-based** (CSS-compliant): `:nth-child(1)` selects the first child.
2. The `even` and `odd` keywords are **1-based**: `:nth-child(even)` selects the 2nd, 4th, 6th children.
3. Formulas use `n` as a counter: `:nth-child(3n)` selects every third child.
4. **Not deprecated** — part of the CSS Selectors Level 3 specification.

**Key Difference from `:eq()`**
- `:nth-child(n)` counts **all children** of each parent, regardless of type, and selects the element only if it matches the attached selector.
- `.eq(n)` counts only the **matched set** (the elements jQuery found), not limited to children of any other element.

**Constraints and Limitations**
- Text nodes and comments are not counted as children for this selector.
- The element must be the nth child among **all siblings**, not just siblings of the same type.
- Use `:nth-of-type()` when you need to count only siblings of the same element type.

### Annotated Complete Code Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>:nth-child() Selector — Second List Item in Each List</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>
    <ul>
      <li>John</li>
      <li>Karl</li>
      <li>Brandon</li>
    </ul>
  </div>
  <div>
    <ul>
      <li>Glen</li>
      <li>Tane</li>
      <li>Ralph</li>
      <li>David</li>
    </ul>
  </div>

  <script>
    $(function () {
      // Step 1: Select the second li in each ul
      $("ul li:nth-child(2)").append("<span> - 2nd!</span>");

      // Step 2: Log how many were matched
      console.log("Second items:", $("ul li:nth-child(2)").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- “Karl” and “Tane” (the second items in each list) receive the appended text “ - 2nd!”.
- Console output: `Second items: 2`

**Why This Output Occurs**
`$("ul li:nth-child(2)")` selects `<li>` elements that are the second child of their parent `<ul>`. Each `<ul>` has a second child, so two elements are selected.

### Real-World Cases
- Zebra-striping table rows using `:nth-child(even)`.
- Styling every third item in a grid using `:nth-child(3n)`.
- Selecting specific columns in a table using `:nth-child()`.
- Creating alternating row colours in dynamically generated lists.

### References
- :nth-child() Selector – https://api.jquery.com/nth-child-selector/
- Child Filter | jQuery API Documentation – https://api.jquery.com/category/selectors/child-filter-selectors/
- MDN Web Docs — :nth-child() – https://developer.mozilla.org/en-US/docs/Web/CSS/:nth-child
- W3C — Selectors Level 3 – https://www.w3.org/TR/css3-selectors/#nth-child-pseudo

---

## Summary Table: jQuery Positional and Filtering Selectors at a Glance

| Selector | Index Base | Deprecated? | Method Alternative | Matches |
|---|---|---|---|---|
| `:first` | 0-based | Yes (3.4) | `.first()` | First element in set (1 element) |
| `:last` | 0-based | Yes (3.4) | `.last()` | Last element in set (1 element) |
| `:eq(n)` | 0-based | Yes (3.4) | `.eq(n)` | Element at index n |
| `:lt(n)` | 0-based | Yes (3.4) | `.slice(0, n)` | Elements with index < n |
| `:gt(n)` | 0-based | Yes (3.4) | `.slice(n + 1)` | Elements with index > n |
| `:even` | 0-based | Yes (3.4) | `.even()` / `.filter(":even")` | Indices 0, 2, 4, … |
| `:odd` | 0-based | Yes (3.4) | `.odd()` / `.filter(":odd")` | Indices 1, 3, 5, … |
| `:first-child` | 1-based (CSS) | **No** | N/A | First child of each parent |
| `:last-child` | 1-based (CSS) | **No** | N/A | Last child of each parent |
| `:nth-child(n)` | 1-based (CSS) | **No** | N/A | nth child of each parent |

---

## General References

- Basic Filter | jQuery API Documentation – https://api.jquery.com/category/selectors/basic-filter-selectors/
- Child Filter | jQuery API Documentation – https://api.jquery.com/category/selectors/child-filter-selectors/
- Deprecated 3.4 | jQuery API Documentation – https://api.jquery.com/category/deprecated/deprecated-3.4/
- :first Selector – https://api.jquery.com/first-selector/
- :last Selector – https://api.jquery.com/last-selector/
- :eq() Selector – https://api.jquery.com/eq-selector/
- :lt() Selector – https://api.jquery.com/lt-selector/
- :gt() Selector – https://api.jquery.com/gt-selector/
- :even Selector – https://api.jquery.com/even-selector/
- :odd Selector – https://api.jquery.com/odd-selector/
- :first-child Selector – https://api.jquery.com/first-child-selector/
- :last-child Selector – https://api.jquery.com/last-child-selector/
- :nth-child() Selector – https://api.jquery.com/nth-child-selector/
- .first() Method – https://api.jquery.com/first/
- .last() Method – https://api.jquery.com/last/
- .eq() Method – https://api.jquery.com/eq/
- .slice() Method – https://api.jquery.com/slice/
- .even() Method – https://api.jquery.com/even/
- .odd() Method – https://api.jquery.com/odd/
- MDN Web Docs — CSS Pseudo-classes – https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes
- W3C — Selectors Level 3 – https://www.w3.org/TR/css3-selectors/
- jQuery 3.4.0 Release Notes – https://blog.jquery.com/2019/04/10/jquery-3-4-0-released/