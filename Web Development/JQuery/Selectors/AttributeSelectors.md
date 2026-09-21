# jQuery Attribute Selectors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery attribute selectors are string expressions passed to the `$()` function that identify and retrieve DOM elements based on the presence, absence, or specific value patterns of their HTML attributes.

**Technical Definition**
Attribute selectors are a category within the jQuery Selectors API that match elements by examining their attributes in the DOM. When `$()` is called with an attribute selector string, jQuery parses the expression and — where possible — delegates to the browser's native `querySelectorAll()` method for standards-compliant selectors, or uses its internal Sizzle engine for jQuery-specific extensions. The selectors support exact matches, partial matches (prefix, suffix, substring), word-based matching, hyphen-separated prefix matching, and presence/absence checks.

**Beginner-Friendly Explanation**
HTML elements carry extra information in their attributes — like `href` on links, `src` on images, or custom `data-*` attributes. jQuery attribute selectors let you find elements based on that extra information. Instead of just saying "find all links," you can say "find all links whose `href` starts with `https`" or "find all inputs that have a `name` attribute." The attribute selector is written inside square brackets `[ ]`.

### Key Characteristics

- **CSS-Compatible**: Most attribute selectors follow the CSS Selectors Level 3 specification and are supported natively by modern browsers.
- **Pattern-Based Matching**: Supports exact, prefix, suffix, substring, word, and hyphen-prefix matching, providing fine-grained control over element selection.
- **jQuery Extensions**: The `[attribute!=value]` selector is a jQuery extension not found in the CSS specification.
- **Custom Data Attributes**: Frequently used with `data-*` attributes for application state and configuration.
- **Composability**: Can be combined with element, ID, and class selectors, and with other attribute selectors for precise targeting.

### Prerequisites

- Basic understanding of HTML attributes (`id`, `class`, `href`, `src`, `data-*`, etc.).
- Familiarity with jQuery basic selectors (element, ID, class).
- jQuery library included in the page via a `<script>` tag or CDN.
- A browser with developer tools for testing and inspection.

### Related Programming Areas

- **CSS Attribute Selectors**: The jQuery syntax mirrors CSS, so styling knowledge transfers.
- **Data Attributes**: Frequently used to store application state in `data-*` attributes.
- **Form Manipulation**: Selecting inputs by `type`, `name`, `value`, or `checked` status.
- **DOM Traversal**: Attribute selectors often serve as the starting point for `.filter()` and `.find()` chains.

### Core Concepts / Features

1. `[attribute]` — Has Attribute Selector
2. `[attribute=value]` — Attribute Equals Selector
3. `[attribute!=value]` — Attribute Not Equal Selector (jQuery extension)
4. `[attribute^=value]` — Attribute Starts With Selector
5. `[attribute$=value]` — Attribute Ends With Selector
6. `[attribute*=value]` — Attribute Contains Selector
7. `[attribute~=value]` — Attribute Contains Word Selector
8. `[attribute|=value]` — Attribute Contains Prefix Selector

---

## Core Concept 1: `[attribute]` — Has Attribute Selector

### Definitions

**Core Definition**
The `[attribute]` selector selects all elements that possess a specified attribute, regardless of the attribute's value.

**Technical Definition**
`jQuery("[attribute]")` matches every element in the document that has an attribute node with the given name. The attribute's value may be empty, and the element is still selected. This is the broadest attribute selector and is natively supported by `querySelectorAll()`.

**Beginner-Friendly Explanation**
This selector asks: "Does this element have this attribute?" It does not care what the value is — just that the attribute exists. For example, `$("[href]")` finds every element that has an `href` attribute, whether the link goes to `https://example.com` or `#section`.

### Purposes

- To select all elements carrying a particular attribute, regardless of its value.
- To identify elements that support a specific functionality (e.g., `[href]` for links, `[src]` for images).
- To find elements with custom `data-*` attributes for application logic.
- To serve as a broad starting point that can be narrowed with additional filters.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute]")
```

**Component Breakdown**

- `[` and `]` : Square brackets delimit the attribute selector.
- `attribute` : The name of the HTML attribute to check for (e.g., `href`, `name`, `data-id`).
- Returns: A jQuery object containing all elements that have the specified attribute.

**Syntax Rules**

1. The attribute name is written without quotes inside the brackets: `[href]`.
2. The selector matches regardless of the attribute's value, including empty values.
3. Attribute names are case-insensitive in HTML documents.
4. Can be combined with element selectors: `$("a[href]")` selects only `<a>` elements with an `href`.

**Constraints and Limitations**

- Does not filter by value; use more specific selectors for value-based matching.
- The attribute name must be a valid HTML attribute; custom attributes should use `data-*` naming.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting All Elements with an `href` Attribute**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Has Attribute Selector — href</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <a href="https://example.com">Link 1</a>
  <a href="#section">Link 2</a>
  <a>Link without href</a>
  <link rel="stylesheet" href="style.css">
  <div data-role="button">Custom button</div>

  <script>
    $(function () {
      // Step 1: Select all elements with an href attribute
      var $withHref = $("[href]");

      // Step 2: Log the count and tag names
      console.log("Elements with href:", $withHref.length); // 3

      $withHref.each(function () {
        console.log("Tag:", this.tagName, "| Value:", $(this).attr("href"));
      });

      // Step 3: Style them for visual confirmation
      $withHref.css("border", "2px solid blue");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two `<a>` elements with `href` and the `<link>` element receive a blue border.
- The `<a>` without `href` and the `<div>` are unchanged.
- Console output:
```
Elements with href: 3
Tag: A | Value: https://example.com
Tag: A | Value: #section
Tag: LINK | Value: style.css
```

**Why This Output Occurs**
`$("[href]")` matches any element that has an `href` attribute. The third `<a>` lacks the attribute and is excluded. The `<link>` element has an `href`, so it is included. The `<div>` with `data-role` does not match because it does not have an `href` attribute.

---

**Example 2: Selecting Elements with Custom Data Attributes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Has Attribute Selector — data-*</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button data-toggle="modal">Open Modal</button>
  <button data-toggle="dropdown">Open Dropdown</button>
  <button>No toggle</button>
  <div data-toggle="tooltip">Tooltip target</div>

  <script>
    $(function () {
      // Step 1: Select all elements with data-toggle
      var $toggles = $("[data-toggle]");
      console.log("Toggles found:", $toggles.length); // 3

      // Step 2: Log each element's data-toggle value
      $toggles.each(function () {
        console.log(this.tagName + " → " + $(this).attr("data-toggle"));
      });

      // Step 3: Highlight them
      $toggles.css("background-color", "lightgreen");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two buttons and the div with `data-toggle` receive a light green background.
- The button without `data-toggle` is unchanged.
- Console output:
```
Toggles found: 3
BUTTON → modal
BUTTON → dropdown
DIV → tooltip
```

**Why This Output Occurs**
`$("[data-toggle]")` matches any element with a `data-toggle` attribute, regardless of its value. The third button has no such attribute and is excluded. The `<div>` has `data-toggle="tooltip"` and is therefore selected.

### Real-World Cases

- **Finding All Links**: `$("a[href]")` selects only actual links, not placeholder anchors.
- **Image Preloading**: `$("img[src]")` finds all images that have a source set.
- **Form Field Detection**: `$("input[required]")` finds all required form fields.
- **Plugin Initialisation**: `$("[data-plugin]")` finds elements that have a plugin identifier.

### References

- Has Attribute Selector [name] – https://api.jquery.com/has-attribute-selector/
- Attribute | jQuery API Documentation – https://api.jquery.com/category/selectors/attribute-selectors/
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Core Concept 2: `[attribute=value]` — Attribute Equals Selector

### Definitions

**Core Definition**
The `[attribute=value]` selector selects all elements whose specified attribute has a value exactly equal to a given string.

**Technical Definition**
`jQuery("[attribute='value']")` matches elements where the attribute's value is identical to the provided string. The comparison is case-sensitive for attribute values in most browsers. This selector is part of the CSS Selectors Level 3 specification and is natively supported by `querySelectorAll()`.

**Beginner-Friendly Explanation**
This is the most precise attribute selector. It asks: "Does this element have this attribute with exactly this value?" For example, `$("input[type='text']")` finds only text inputs, not checkboxes or radio buttons.

### Purposes

- To select elements with an exact attribute value for precise targeting.
- To target form inputs by their `type`, `name`, or `value` attributes.
- To select links with a specific `target` or `rel` attribute.
- To filter elements by a specific `data-*` value.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute='value']")
```

**Component Breakdown**

- `attribute` : The attribute name (e.g., `type`, `name`, `data-id`).
- `=` : The equality operator.
- `'value'` : The exact value to match. Can be a quoted string or a valid CSS identifier.
- Returns: A jQuery object containing all elements with the exact attribute value.

**Syntax Rules**

1. The value should be enclosed in quotes if it contains spaces or special characters: `[name='first name']`.
2. Quotes are optional for simple identifiers: `[type=text]` is valid but `[type='text']` is recommended for clarity.
3. The comparison is exact — partial matches are not selected.
4. Case sensitivity depends on the attribute; `type` values are case-insensitive in HTML.

**Constraints and Limitations**

- Only matches elements where the attribute value is **exactly** equal; no partial matching.
- If the attribute does not exist on an element, that element is not selected.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting Inputs by Type**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Equals — Input Types</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="text" value="Name">
  <input type="email" value="email@example.com">
  <input type="text" value="Username">
  <input type="checkbox">

  <script>
    $(function () {
      // Step 1: Select all inputs with type="text"
      var $textInputs = $("input[type='text']");
      console.log("Text inputs:", $textInputs.length); // 2

      // Step 2: Style them
      $textInputs.css("border", "2px solid green");

      // Step 3: Select a specific value
      var $specific = $("input[value='Username']");
      console.log("Specific input found:", $specific.length); // 1
      $specific.css("background-color", "lightyellow");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two text inputs receive a green border.
- The input with `value="Username"` receives a light yellow background.
- Console output:
```
Text inputs: 2
Specific input found: 1
```

**Why This Output Occurs**
`$("input[type='text']")` matches only elements whose `type` attribute is exactly `"text"`. The email and checkbox inputs are excluded. The second query matches only the input whose `value` attribute is exactly `"Username"`.

---

**Example 2: Selecting Links with a Specific Target**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Equals — Target</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <a href="https://example.com" target="_blank">External</a>
  <a href="/internal" target="_self">Internal</a>
  <a href="https://other.com" target="_blank">Another External</a>

  <script>
    $(function () {
      // Select all links that open in a new tab
      $("a[target='_blank']").css("color", "red");

      console.log("New-tab links:",
        $("a[target='_blank']").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two links with `target="_blank"` turn red.
- The internal link remains unchanged.
- Console output: `New-tab links: 2`

**Why This Output Occurs**
`$("a[target='_blank']")` matches only `<a>` elements whose `target` attribute is exactly `"_blank"`. The internal link has `target="_self"` and is excluded.

### Real-World Cases

- **Form Validation**: `$("input[type='email']")` selects all email fields for validation.
- **Button Detection**: `$("button[type='submit']")` finds submit buttons for form handling.
- **External Link Marking**: `$("a[target='_blank']").addClass("external")` marks links that open in new tabs.
- **Data Attribute Filtering**: `$("[data-status='active']")` finds elements with a specific state.

### References

- Attribute Equals Selector [name=”value”] – https://api.jquery.com/attribute-equals-selector/
- CSS Selectors Level 3 — Attribute selectors – https://www.w3.org/TR/css3-selectors/#attribute-selectors
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Core Concept 3: `[attribute!=value]` — Attribute Not Equal Selector

### Definitions

**Core Definition**
The `[attribute!=value]` selector selects elements that either do not have the specified attribute, or have it but with a value that is not equal to the given string.

**Technical Definition**
`jQuery("[attribute!='value']")` is a jQuery extension not part of the CSS specification. It is functionally equivalent to `:not([attribute='value'])`. Because it is not a standard CSS selector, it cannot use the browser's native `querySelectorAll()` and relies on jQuery's Sizzle engine, which may be slower.

**Beginner-Friendly Explanation**
This selector asks: "Give me elements that do **not** have this attribute with this exact value." That includes two groups: elements without the attribute at all, and elements that have the attribute but with a different value.

### Purposes

- To exclude elements that have a specific attribute value.
- To select a broad set of elements and then filter out those matching a particular condition.
- To target form inputs that do not have a particular name or type.
- To find elements that are "not" something, as a complement to equality selectors.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute!='value']")
```

**Component Breakdown**

- `attribute` : The attribute name to check.
- `!=` : The not-equal operator (jQuery extension).
- `'value'` : The value to exclude.
- Returns: A jQuery object containing elements that either lack the attribute or have a different value.

**Syntax Rules**

1. This selector is a **jQuery extension** and is not part of the CSS specification.
2. It is equivalent to `:not([attribute='value'])`.
3. For better performance, use `$("pure-css-selector").not("[attribute='value']")` instead.
4. The value can be a quoted string or a valid identifier.

**Constraints and Limitations**

- **Performance**: Cannot use native `querySelectorAll()`; slower than standard selectors.
- **Deprecated Recommendation**: jQuery recommends using `.not()` for better performance.
- **Browser Compatibility**: Works in all browsers via jQuery, but not natively.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting Inputs Without a Specific Name**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Not Equal — Inputs</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="radio" name="newsletter" value="Hot Fuzz">
  <input type="radio" value="Cold Fusion">
  <input type="radio" name="accept" value="Evil Plans">

  <script>
    $(function () {
      // Step 1: Select all inputs that do NOT have name="newsletter"
      var $notNewsletter = $("input[name!='newsletter']");
      console.log("Not newsletter:", $notNewsletter.length); // 2

      // Step 2: Append text to the next sibling of each
      $notNewsletter.next().append("<b> (not newsletter)</b>");

      // Step 3: Highlight for clarity
      $notNewsletter.css("outline", "2px solid orange");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The second and third radio inputs (which either lack `name` or have a different `name`) receive an orange outline.
- Console output: `Not newsletter: 2`

**Why This Output Occurs**
`$("input[name!='newsletter']")` excludes only the first input, which has `name="newsletter"`. The second input has no `name` attribute, and the third has `name="accept"`. Both match the selector because they are "not equal" to the excluded value.

---

**Example 2: Performance Alternative Using `.not()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Not Equal — .not() Alternative</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div data-status="active">Active</div>
  <div data-status="inactive">Inactive</div>
  <div data-status="pending">Pending</div>
  <div>No status</div>

  <script>
    $(function () {
      // Recommended approach: pure CSS selector + .not()
      var $notActive = $("div[data-status]").not("[data-status='active']");
      console.log("Not active (has status):", $notActive.length); // 2

      $notActive.css("border", "2px solid purple");

      // This excludes the div without data-status entirely
      var $allNotActive = $("div").not("[data-status='active']");
      console.log("All divs not active:", $allNotActive.length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output**
- The "inactive" and "pending" divs receive a purple border.
- The div without `data-status` also matches the second query.
- Console output:
```
Not active (has status): 2
All divs not active: 3
```

**Why This Output Occurs**
`$("div[data-status]")` first selects all divs with a `data-status` attribute, then `.not("[data-status='active']")` excludes the active one. The second query starts with all divs and excludes the active one, so the div without a status is included.

### Real-World Cases

- **Form Field Exclusion**: `$("input[name!='csrf_token']")` processes all fields except the CSRF token.
- **Navigation Filtering**: `$("a[href!='#']").addClass("real-link")` marks links that are not placeholders.
- **State Filtering**: `$("[data-state!='resolved']")` finds all items that are not resolved.
- **Element Exclusion**: `$("div[class!='hidden']")` selects divs that are not hidden by class.

### References

- Attribute Not Equal Selector [name!=”value”] – https://api.jquery.com/attribute-not-equal-selector/
- jQuery API — Attribute Selectors – https://api.jquery.com/category/selectors/attribute-selectors/

---

## Core Concept 4: `[attribute^=value]` — Attribute Starts With Selector

### Definitions

**Core Definition**
The `[attribute^=value]` selector selects elements whose specified attribute value begins exactly with a given string.

**Technical Definition**
`jQuery("[attribute^='value']")` matches elements where the attribute's value starts with the provided prefix. The comparison is case-sensitive. This selector is part of the CSS Selectors Level 3 specification and is natively supported by `querySelectorAll()`. It is useful for identifying elements with systematically generated IDs or values.

**Beginner-Friendly Explanation**
This selector asks: "Does this element's attribute value begin with this text?" For example, `$("input[name^='news']")` finds all inputs whose `name` attribute starts with `"news"` — like `"newsletter"`, `"newsboy"`, or `"news-item"`.

### Purposes

- To select elements whose attribute value starts with a known prefix.
- To identify elements generated by server-side frameworks with systematic IDs.
- To group elements by a common attribute prefix for batch operations.
- To filter form fields by a naming convention (e.g., all fields starting with `"user_"`).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute^='value']")
```

**Component Breakdown**

- `attribute` : The attribute name to check.
- `^=` : The "starts with" operator.
- `'value'` : The prefix string to match.
- Returns: A jQuery object containing elements whose attribute value begins with the prefix.

**Syntax Rules**

1. The comparison is case-sensitive: `[name^='News']` will not match `name="newsletter"`.
2. The value can be quoted or a valid identifier.
3. If the attribute value is exactly equal to the prefix, the element is also selected (a string starts with itself).
4. Can be combined with element selectors: `$("input[name^='news']")`.

**Constraints and Limitations**

- Case-sensitive matching may require normalization if case-insensitive behaviour is needed.
- Performance is slower than class selectors; the official documentation recommends using classes where possible.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting Inputs by Name Prefix**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Starts With — Input Names</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input name="newsletter">
  <input name="milkman">
  <input name="newsboy">
  <input name="newmilk">

  <script>
    $(function () {
      // Step 1: Select inputs whose name starts with "news"
      var $newsInputs = $("input[name^='news']");
      console.log("News inputs:", $newsInputs.length); // 2

      // Step 2: Set their values
      $newsInputs.val("news here!");

      // Step 3: Log the values
      $newsInputs.each(function () {
        console.log(this.name + " → " + this.value);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `newsletter` and `newsboy` inputs receive the value `"news here!"`.
- Console output:
```
News inputs: 2
newsletter → news here!
newsboy → news here!
```

**Why This Output Occurs**
`$("input[name^='news']")` matches inputs whose `name` attribute begins with `"news"`. `"newsletter"` and `"newsboy"` both start with `"news"`. `"milkman"` and `"newmilk"` do not start with `"news"` (the latter starts with `"new"` but not `"news"`), so they are excluded.

---

**Example 2: Selecting Links by Protocol**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Starts With — Protocol</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <a href="https://secure.com">Secure</a>
  <a href="http://insecure.com">Insecure</a>
  <a href="mailto:user@example.com">Email</a>
  <a href="/relative">Relative</a>

  <script>
    $(function () {
      // Select all links with HTTPS protocol
      $("a[href^='https']").css("color", "green");

      // Select all links with mailto: protocol
      $("a[href^='mailto']").css("color", "blue");

      console.log("HTTPS links:", $("a[href^='https']").length); // 1
      console.log("Mailto links:", $("a[href^='mailto']").length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- The HTTPS link turns green.
- The mailto link turns blue.
- Console output:
```
HTTPS links: 1
Mailto links: 1
```

**Why This Output Occurs**
`$("a[href^='https']")` matches the first link because its `href` starts with `"https"`. The `http://` link does not start with `"https"`. The `mailto:` link starts with `"mailto"` and matches the second selector. The relative link matches neither.

### Real-World Cases

- **Form Field Grouping**: `$("input[name^='shipping_']")` finds all shipping-related form fields.
- **Protocol Detection**: `$("a[href^='https']").addClass("secure")` marks secure links.
- **ID Prefix Matching**: `$("[id^='product-']")` finds all elements with IDs starting with `"product-"`.
- **Data Attribute Prefixing**: `$("[data-user^='admin']")` finds elements where `data-user` starts with `"admin"`.

### References

- Attribute Starts With Selector [name^=”value”] – https://api.jquery.com/attribute-starts-with-selector/
- CSS Selectors Level 3 — Attribute selectors – https://www.w3.org/TR/css3-selectors/#attribute-selectors
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Core Concept 5: `[attribute$=value]` — Attribute Ends With Selector

### Definitions

**Core Definition**
The `[attribute$=value]` selector selects elements whose specified attribute value ends exactly with a given string.

**Technical Definition**
`jQuery("[attribute$='value']")` matches elements where the attribute's value ends with the provided suffix. The comparison is case-sensitive. This selector is part of the CSS Selectors Level 3 specification and is natively supported by `querySelectorAll()`.

**Beginner-Friendly Explanation**
This selector asks: "Does this element's attribute value end with this text?" For example, `$("a[href$='.pdf']")` finds all links to PDF files, because their `href` ends with `".pdf"`.

### Purposes

- To select elements whose attribute value ends with a known suffix.
- To identify file types by their extension (e.g., `.pdf`, `.jpg`, `.zip`).
- To filter elements by a consistent naming suffix in IDs or classes.
- To group elements by a common attribute ending for batch operations.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute$='value']")
```

**Component Breakdown**

- `attribute` : The attribute name to check.
- `$=` : The "ends with" operator.
- `'value'` : The suffix string to match.
- Returns: A jQuery object containing elements whose attribute value ends with the suffix.

**Syntax Rules**

1. The comparison is case-sensitive.
2. The value can be quoted or a valid identifier.
3. If the attribute value is exactly equal to the suffix, the element is also selected (a string ends with itself).
4. Can be combined with element selectors: `$("a[href$='.pdf']")`.

**Constraints and Limitations**

- Case-sensitive matching may require lowercase normalization for case-insensitive checks.
- The suffix must match exactly; `[href$='pdf']` will not match `href="file.PDF"` if case differs.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting Links by File Extension**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Ends With — File Types</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <a href="document.pdf">PDF Document</a>
  <a href="image.jpg">JPEG Image</a>
  <a href="archive.zip">ZIP Archive</a>
  <a href="report.pdf">Another PDF</a>
  <a href="page.html">HTML Page</a>

  <script>
    $(function () {
      // Step 1: Select all links to PDF files
      var $pdfLinks = $("a[href$='.pdf']");
      console.log("PDF links:", $pdfLinks.length); // 2

      // Step 2: Mark them
      $pdfLinks.addClass("pdf-link").css("color", "red");

      // Step 3: Select all links to images
      $("a[href$='.jpg']").css("border", "2px solid orange");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two PDF links turn red and receive the class `pdf-link`.
- The JPEG link receives an orange border.
- Console output: `PDF links: 2`

**Why This Output Occurs**
`$("a[href$='.pdf']")` matches links whose `href` attribute ends with `".pdf"`. The two PDF links match; the ZIP, JPEG, and HTML links do not. The second selector matches only the JPEG link.

---

**Example 2: Selecting Elements by ID Suffix**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Ends With — ID Suffix</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="user-123">User 123</div>
  <div id="user-456">User 456</div>
  <div id="admin-789">Admin 789</div>
  <div id="user-abc">User ABC</div>

  <script>
    $(function () {
      // Select all elements whose id ends with a digit
      var $numericIds = $("[id$='3'], [id$='6'], [id$='9']");
      console.log("Numeric-suffix IDs:", $numericIds.length); // 3

      $numericIds.css("font-weight", "bold");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The three elements with IDs ending in `3`, `6`, or `9` become bold.
- The element with ID `user-abc` remains unchanged.
- Console output: `Numeric-suffix IDs: 3`

**Why This Output Occurs**
The multiple selector combines three suffix matches: `[id$='3']`, `[id$='6']`, and `[id$='9']`. Each matches one element, and the combined jQuery object contains three elements.

### Real-World Cases

- **File Type Detection**: `$("a[href$='.pdf']")` identifies PDF links for special handling.
- **Download Links**: `$("a[href$='.zip'], a[href$='.tar.gz']")` finds archive download links.
- **ID Convention Matching**: `$("[id$='-container']")` finds all container elements following a naming convention.
- **Email Domain Filtering**: `$("a[href$='@example.com']")` finds mailto links to a specific domain.

### References

- Attribute Ends With Selector [name$=”value”] – https://api.jquery.com/attribute-ends-with-selector/
- jQuery [attribute$=value] Selector – https://raw.githubusercontent.com/T-manuel/W3Schools/main/jquery/sel_attribute_end_value.html
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Core Concept 6: `[attribute*=value]` — Attribute Contains Selector

### Definitions

**Core Definition**
The `[attribute*=value]` selector selects elements whose specified attribute value contains a given substring anywhere within it.

**Technical Definition**
`jQuery("[attribute*='value']")` matches elements where the attribute's value contains the provided substring. This is the most permissive of the value-matching attribute selectors. It is part of the CSS Selectors Level 3 specification and is natively supported by `querySelectorAll()`. It should be compared with `[attribute~=value]`, which requires a word match rather than a substring match.

**Beginner-Friendly Explanation**
This selector asks: "Does this element's attribute value contain this text anywhere?" For example, `$("input[name*='man']")` finds all inputs whose `name` contains `"man"` — like `"milkman"`, `"letterman2"`, or `"man-news"`.

### Purposes

- To select elements where the attribute value contains a substring, regardless of position.
- To perform broad pattern matching when the exact position of the substring is unknown.
- To filter elements by partial values in `data-*` attributes.
- To serve as a broad selection that can be narrowed with additional filters.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute*='value']")
```

**Component Breakdown**

- `attribute` : The attribute name to check.
- `*=` : The "contains" operator.
- `'value'` : The substring to search for within the attribute value.
- Returns: A jQuery object containing elements whose attribute value contains the substring.

**Syntax Rules**

1. The comparison is case-sensitive.
2. The substring can appear anywhere in the attribute value: beginning, middle, or end.
3. The value can be quoted or a valid identifier.
4. This is the most generous jQuery attribute selector that matches against a value.

**Constraints and Limitations**

- Case-sensitive; `[name*='Man']` will not match `name="milkman"`.
- May produce a larger result set than more specific selectors like `^=` or `$=`.
- For word-level matching, use `[attribute~=value]` instead.

### Multiple Annotated Complete Code Examples

**Example 1: Broad Substring Matching**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Contains — Input Names</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input name="man-news">
  <input name="milkman">
  <input name="letterman2">
  <input name="newmilk">

  <script>
    $(function () {
      // Step 1: Select inputs whose name contains "man"
      var $manInputs = $("input[name*='man']");
      console.log("Contains 'man':", $manInputs.length); // 3

      // Step 2: Set their values
      $manInputs.val("has man in it!");

      // Step 3: Log names
      $manInputs.each(function () {
        console.log(this.name + " → " + this.value);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `man-news`, `milkman`, and `letterman2` inputs receive the value `"has man in it!"`.
- The `newmilk` input is unchanged.
- Console output:
```
Contains 'man': 3
man-news → has man in it!
milkman → has man in it!
letterman2 → has man in it!
```

**Why This Output Occurs**
`$("input[name*='man']")` matches any input whose `name` attribute contains the substring `"man"`. `"man-news"` starts with `"man"`, `"milkman"` ends with `"man"`, and `"letterman2"` contains `"man"` in the middle. `"newmilk"` does not contain `"man"` and is excluded.

---

**Example 2: Comparing `*=` with `~=`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Contains — vs Word Selector</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div data-tags="javascript jquery">Both</div>
  <div data-tags="javascript only">JavaScript only</div>
  <div data-tags="jquery only">jQuery only</div>
  <div data-tags="javascript-advanced">Hyphenated</div>

  <script>
    $(function () {
      // *="jquery" matches any substring occurrence
      var $contains = $("[data-tags*='jquery']");
      console.log("Contains 'jquery':", $contains.length); // 2

      // ~="jquery" matches whole words only
      var $word = $("[data-tags~='jquery']");
      console.log("Word 'jquery':", $word.length); // 2

      // *="java" matches substring
      var $substring = $("[data-tags*='java']");
      console.log("Contains 'java':", $substring.length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output**
- Console output:
```
Contains 'jquery': 2
Word 'jquery': 2
Contains 'java': 3
```

**Why This Output Occurs**
`[data-tags*='jquery']` matches any element whose `data-tags` value contains the substring `"jquery"`. Two elements contain it. `[data-tags~='jquery']` matches only elements where `"jquery"` appears as a complete word delimited by spaces. The same two elements match. `[data-tags*='java']` matches three elements because `"java"` appears as a substring in `"javascript jquery"`, `"javascript only"`, and `"javascript-advanced"`.

### Real-World Cases

- **Search Filtering**: `$("input[name*='search']")` finds search-related form fields.
- **URL Pattern Matching**: `$("a[href*='docs']")` finds links containing `"docs"` in their URL.
- **Data Attribute Searching**: `$("[data-description*='sale']")` finds elements whose description mentions "sale".
- **Class-Like Matching**: `$("[class*='btn-']")` finds elements with class names containing `"btn-"`.

### References

- Attribute Contains Selector [name*=”value”] – https://api.jquery.com/attribute-contains-selector/
- jQuery [attribute*=value] Selector – https://coursera.w3schools.com/jquery/sel_attribute_contains_value.asp
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Core Concept 7: `[attribute~=value]` — Attribute Contains Word Selector

### Definitions

**Core Definition**
The `[attribute~=value]` selector selects elements whose specified attribute value contains a given word, where the word is a complete token delimited by whitespace.

**Technical Definition**
`jQuery("[attribute~='value']")` matches elements where the attribute's value contains the provided word as a **standalone token**, separated from other words by spaces. The comparison is case-sensitive. This selector is part of the CSS Selectors Level 3 specification and is natively supported by `querySelectorAll()`. It is more precise than `[attribute*=value]` because it requires a whole-word match.

**Beginner-Friendly Explanation**
This selector asks: "Does this element's attribute value contain this word as a separate word?" For example, `$("[data-tags~='jquery']")` matches `data-tags="javascript jquery"` but not `data-tags="jquery-ui"` because `"jquery"` is not a standalone word in the latter case.

### Purposes

- To select elements by a specific word within a space-separated attribute value.
- To filter elements by tags, categories, or keywords stored in attributes.
- To distinguish between whole-word matches and substring matches.
- To work with attributes that contain lists of values, such as `class` or custom `data-*` lists.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute~='value']")
```

**Component Breakdown**

- `attribute` : The attribute name to check.
- `~=` : The "contains word" operator.
- `'value'` : The word to match as a complete token.
- Returns: A jQuery object containing elements whose attribute value contains the word.

**Syntax Rules**

1. The comparison is case-sensitive.
2. The word must be delimited by whitespace to match; `"jquery-ui"` does not match `[~='jquery']`.
3. The value can be quoted or a valid identifier.
4. This selector is equivalent to the CSS `[attr~=val]` selector.

**Constraints and Limitations**

- Does not match hyphenated compounds (e.g., `"jquery-ui"`).
- Case-sensitive; `[~='JQuery']` will not match `"jquery"`.
- For substring matching regardless of word boundaries, use `[attribute*=value]` instead.

### Multiple Annotated Complete Code Examples

**Example 1: Whole-Word Matching**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Contains Word — Tags</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div data-tags="javascript jquery">Both</div>
  <div data-tags="javascript only">JavaScript only</div>
  <div data-tags="jquery only">jQuery only</div>
  <div data-tags="jquery-ui">Hyphenated</div>
  <div data-tags="myjquery">Prefix</div>

  <script>
    $(function () {
      // Step 1: Select elements where "jquery" is a whole word
      var $wholeWord = $("[data-tags~='jquery']");
      console.log("Whole word 'jquery':", $wholeWord.length); // 2

      $wholeWord.css("border", "2px solid blue");

      // Step 2: Compare with substring matching
      var $substring = $("[data-tags*='jquery']");
      console.log("Substring 'jquery':", $substring.length); // 4
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first and third divs (where `"jquery"` is a whole word) receive a blue border.
- The hyphenated `"jquery-ui"` and prefixed `"myjquery"` divs are not selected by `~=`.
- Console output:
```
Whole word 'jquery': 2
Substring 'jquery': 4
```

**Why This Output Occurs**
`[data-tags~='jquery']` requires `"jquery"` to be a complete token delimited by spaces. The first div (`"javascript jquery"`) and third div (`"jquery only"`) match. The fourth div (`"jquery-ui"`) and fifth div (`"myjquery"`) do not. The substring selector `*=` matches all four because `"jquery"` appears as a substring in each.

### Real-World Cases

- **Tag Filtering**: `$("[data-tags~='featured']")` finds elements tagged as "featured".
- **Category Selection**: `$("[data-categories~='sale']")` finds products in the "sale" category.
- **Role-Based Selection**: `$("[data-roles~='admin']")` finds elements where "admin" is one of the roles.
- **Class-Like Matching**: `$("[class~='active']")` is equivalent to `$(".active")` but uses attribute selector syntax.

### References

- Attribute Contains Word Selector [name~=”value”] – https://api.jquery.com/attribute-contains-word-selector/
- jQuery [attribute~=value] Selector – https://www.w3schools.com/JQuery/sel_attribute_contains_value.asp
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Core Concept 8: `[attribute|=value]` — Attribute Contains Prefix Selector

### Definitions

**Core Definition**
The `[attribute|=value]` selector selects elements whose specified attribute value is either exactly equal to a given string, or starts with that string followed by a hyphen (`-`).

**Technical Definition**
`jQuery("[attribute|='value']")` matches elements where the attribute's value equals the provided string or begins with the string immediately followed by a hyphen. This selector was introduced into the CSS specification to handle language attributes such as `hreflang="en"` and `hreflang="en-US"`. It is natively supported by `querySelectorAll()`.

**Beginner-Friendly Explanation**
This selector asks: "Is this attribute value exactly this, or does it start with this followed by a hyphen?" For example, `$("[hreflang|='en']")` matches both `hreflang="en"` and `hreflang="en-US"`, but not `hreflang="english"`.

### Purposes

- To select elements by language attributes, matching both base language and regional variants.
- To match attribute values that follow a hyphen-separated naming convention.
- To select elements where the attribute value is a prefix followed by a hyphen (e.g., `en-US`, `zh-CN`).
- To handle language-specific content selection in multilingual applications.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("[attribute|='value']")
```

**Component Breakdown**

- `attribute` : The attribute name to check.
- `|=` : The "contains prefix" operator (equals or starts with value + hyphen).
- `'value'` : The prefix string to match.
- Returns: A jQuery object containing elements whose attribute value equals the prefix or starts with prefix + hyphen.

**Syntax Rules**

1. Matches **exactly** the value or **starts with** value followed by a hyphen (`-`).
2. The comparison is case-sensitive.
3. The value can be quoted or a valid identifier.
4. This selector is designed for language attributes but works with any hyphen-separated attribute value.

**Constraints and Limitations**

- Does not match values where the prefix is followed by anything other than a hyphen.
- Case-sensitive; `[hreflang|='EN']` will not match `hreflang="en-US"`.
- Less commonly used than other attribute selectors; primarily for language and locale attributes.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting Language Variants**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Contains Prefix — Language</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <a href="page.html" hreflang="en">English</a>
  <a href="page.html" hreflang="en-UK">English (UK)</a>
  <a href="page.html" hreflang="en-US">English (US)</a>
  <a href="page.html" hreflang="fr">French</a>
  <a href="page.html" hreflang="english">Not a valid language code</a>

  <script>
    $(function () {
      // Step 1: Select all links with English language variants
      var $english = $("a[hreflang|='en']");
      console.log("English links:", $english.length); // 3

      // Step 2: Style them
      $english.css("border", "3px dotted green");

      // Step 3: Log each match
      $english.each(function () {
        console.log("hreflang:", $(this).attr("hreflang"));
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The links with `hreflang="en"`, `hreflang="en-UK"`, and `hreflang="en-US"` receive a green dotted border.
- The French link and the invalid `"english"` link are unchanged.
- Console output:
```
English links: 3
hreflang: en
hreflang: en-UK
hreflang: en-US
```

**Why This Output Occurs**
`[hreflang|='en']` matches values that are exactly `"en"` or start with `"en-"`. The `"en"`, `"en-UK"`, and `"en-US"` values all match. The `"english"` value does not match because it does not start with `"en-"`; it starts with `"en"` followed by `"g"`, not a hyphen.

---

**Example 2: Hyphen-Separated Naming Conventions**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Contains Prefix — Naming</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div data-section="news">News</div>
  <div data-section="news-sports">Sports News</div>
  <div data-section="news-weather">Weather News</div>
  <div data-section="newspaper">Newspaper</div>

  <script>
    $(function () {
      // Select all elements with data-section="news" or "news-*"
      var $news = $("[data-section|='news']");
      console.log("News sections:", $news.length); // 3

      $news.css("background-color", "lightblue");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `"news"`, `"news-sports"`, and `"news-weather"` divs receive a light blue background.
- The `"newspaper"` div is unchanged.
- Console output: `News sections: 3`

**Why This Output Occurs**
`[data-section|='news']` matches values that equal `"news"` or start with `"news-"`. The `"news"`, `"news-sports"`, and `"news-weather"` values match. `"newspaper"` does not match because it starts with `"news"` followed by `"p"`, not a hyphen.

### Real-World Cases

- **Language Selection**: `$("[hreflang|='en']")` selects all English language links.
- **Locale Filtering**: `$("[data-locale|='en']")` finds elements set to English locales.
- **Section Navigation**: `$("[data-section|='products']")` finds all product-related sections.
- **Version Matching**: `$("[data-version|='v2']")` matches version 2 and its sub-versions.

### References

- Attribute Contains Prefix Selector [name|=”value”] – https://api.jquery.com/attribute-contains-prefix-selector/
- jQuery [attribute|=value] Selector – https://www.w3schools.com/jquery/sel_attribute_contains_value.asp
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors

---

## Summary Table: jQuery Attribute Selectors at a Glance

| Selector | Syntax | Matches | Native CSS? | Example |
|---|---|---|---|---|
| Has Attribute | `[attribute]` | Any value or no value | Yes | `$("[href]")` |
| Equals | `[attribute='value']` | Exact match | Yes | `$("input[type='text']")` |
| Not Equal | `[attribute!='value']` | Missing or different value | No (jQuery extension) | `$("input[name!='csrf']")` |
| Starts With | `[attribute^='value']` | Begins with string | Yes | `$("a[href^='https']")` |
| Ends With | `[attribute$='value']` | Ends with string | Yes | `$("a[href$='.pdf']")` |
| Contains | `[attribute*='value']` | Contains substring | Yes | `$("input[name*='man']")` |
| Contains Word | `[attribute~='value']` | Contains whole word | Yes | `$("[data-tags~='jquery']")` |
| Contains Prefix | `[attribute\|='value']` | Equals or starts with value + hyphen | Yes | `$("[hreflang\|='en']")` |

---

## General References

- jQuery API — Attribute Selectors – https://api.jquery.com/category/selectors/attribute-selectors/
- Has Attribute Selector [name] – https://api.jquery.com/has-attribute-selector/
- Attribute Equals Selector [name=”value”] – https://api.jquery.com/attribute-equals-selector/
- Attribute Not Equal Selector [name!=”value”] – https://api.jquery.com/attribute-not-equal-selector/
- Attribute Starts With Selector [name^=”value”] – https://api.jquery.com/attribute-starts-with-selector/
- Attribute Ends With Selector [name$=”value”] – https://api.jquery.com/attribute-ends-with-selector/
- Attribute Contains Selector [name*=”value”] – https://api.jquery.com/attribute-contains-selector/
- Attribute Contains Word Selector [name~=”value”] – https://api.jquery.com/attribute-contains-word-selector/
- Attribute Contains Prefix Selector [name|=”value”] – https://api.jquery.com/attribute-contains-prefix-selector/
- CSS Selectors Level 3 — Attribute selectors – https://www.w3.org/TR/css3-selectors/#attribute-selectors
- MDN Web Docs — Attribute selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors
- W3Schools — jQuery Selectors – https://www.w3schools.com/jquery/jquery_selectors.asp