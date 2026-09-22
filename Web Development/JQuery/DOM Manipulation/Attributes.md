# jQuery Attributes and Custom Data: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery's attribute and data methods are the tools for reading, setting, and removing HTML attributes and for associating arbitrary JavaScript data with DOM elements in a way that is safe from memory leaks.

**Technical Definition**
The `.attr()` and `.removeAttr()` methods manipulate **HTML attributes** — the values written in the HTML source and exposed via the DOM's `getAttribute()`, `setAttribute()`, and `removeAttribute()` APIs. The `.data()` and `.removeData()` methods manipulate jQuery's **internal data cache**, a JavaScript-side store keyed to each DOM element via an expando property. The `.data()` method also performs a one-time read of HTML5 `data-*` attributes when a key is first accessed, automatically converting string values to JavaScript types (numbers, booleans, objects, arrays) where possible.

**Beginner-Friendly Explanation**
HTML elements can carry extra information in two ways. The first is **attributes** — things like `href`, `title`, or `data-user-id` that you write directly in the HTML. The second is **data** — information that JavaScript attaches to an element while the page is running, which does not appear in the HTML. jQuery gives you `.attr()` to work with attributes and `.data()` to work with the JavaScript-side data. They look similar but behave very differently, and knowing which one to use is essential for writing correct and maintainable code.

### Key Characteristics

- **Attributes are Visible; Data is Internal**: `.attr()` changes are reflected in the DOM and visible in developer tools. `.data()` changes are stored in jQuery's cache and do **not** modify the DOM.
- **Automatic Type Conversion in `.data()`**: When `.data()` reads a `data-*` attribute, it attempts to convert the string to a number, boolean, object, or array.
- **Boolean Attribute Ambiguity**: Attributes like `checked`, `disabled`, and `selected` have both an attribute and a property. jQuery 1.6+ recommends `.prop()` for these, not `.attr()`.
- **One-Time Initialisation**: `.data()` reads a `data-*` attribute **once** on first access. Subsequent changes to the attribute via `.attr()` do not affect the cached data value.
- **Chaining Preserved**: All four methods return the jQuery object when used as setters, preserving method chaining.

### Prerequisites

- Basic understanding of HTML attributes (`id`, `class`, `href`, `data-*`, etc.).
- Familiarity with jQuery selectors and the `$()` function.
- Awareness of the difference between HTML attributes and DOM properties.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **DOM Manipulation**: Attributes and data are fundamental to dynamic page behaviour.
- **Form Handling**: Boolean attributes (`checked`, `disabled`) are central to form state management.
- **Plugin Development**: `.data()` is the preferred mechanism for storing plugin state on elements.
- **HTML5 Custom Data**: `data-*` attributes provide a standard way to embed metadata in markup.

### Core Concepts / Features

1. `.attr()` — Get or Set Attribute Values
2. `.removeAttr()` — Remove Attributes
3. `.data()` — Get or Set Arbitrary Data
4. `.removeData()` — Remove Stored Data
5. Handling Boolean Attributes
6. `.attr()` vs. `.data()`: Attributes and the Data Cache

---

## Core Concept 1: `.attr()` — Get or Set Attribute Values

### Definitions

**Core Definition**
`.attr()` gets the value of an attribute for the first element in the set of matched elements or sets one or more attributes for every matched element.

**Technical Definition**
The `.attr()` method operates on HTML attributes. As a getter (called with only an attribute name), it returns the value of that attribute for the **first** matched element, or `undefined` if the attribute has not been set (as of jQuery 1.6). As a setter (called with a name and value, or an object of name-value pairs), it sets the attribute on **every** matched element and returns the jQuery object for chaining. The method uses the DOM's `getAttribute()` and `setAttribute()` APIs, with jQuery normalising cross-browser inconsistencies in how attribute values are reported.

**Beginner-Friendly Explanation**
`.attr()` is your tool for working with the values you can see in the HTML source. If you want to know what a link's `href` is, or change an image's `alt` text, or add a `title` attribute, you use `.attr()`. Reading returns a string; writing updates the DOM and the HTML source.

### Purposes

- To retrieve the value of an HTML attribute from the first matched element.
- To set one or more attributes on every matched element.
- To read or write standard attributes such as `href`, `src`, `title`, `alt`, and `id`.
- To work with custom `data-*` attributes when the goal is to modify the DOM.
- To compute attribute values dynamically using a callback function.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter
$(selector).attr(attributeName);

// Setter — single attribute
$(selector).attr(attributeName, value);
$(selector).attr(attributeName, function(index, oldValue) { ... });

// Setter — multiple attributes
$(selector).attr({ attribute1: value1, attribute2: value2 });
```

**Component Breakdown**

- `attributeName` (String): The name of the attribute to get or set.
- `value` (String | Number | Boolean): The value to assign. Numbers and booleans are converted to strings.
- `function(index, oldValue)`: A callback returning the value to set. `index` is the element's position; `oldValue` is the current attribute value.
- `{ attribute1: value1, ... }` (Object): An object of key-value pairs for setting multiple attributes at once.
- Returns: A string (getter) or a jQuery object (setter).

**Syntax Rules**

1. As a getter, `.attr()` returns the value for the **first** element only. Use `.each()` or `.map()` to get values for all elements.
2. As of jQuery 1.6, `.attr()` returns `undefined` for attributes that have not been set.
3. `.attr()` should **not** be used on plain objects, arrays, the `window`, or the `document`.
4. For boolean attributes (`checked`, `selected`, `disabled`), use `.prop()` instead.
5. The callback function receives the element's index and the current attribute value.

**Constraints and Limitations**

- Attribute values are strings with a few exceptions (e.g., `value`, `tabindex`).
- `.attr()` cannot reliably retrieve properties like `selectedIndex`, `tagName`, `nodeName`, `nodeType`, `ownerDocument`, `defaultChecked`, or `defaultSelected` — these should use `.prop()`.
- Changing an attribute does not automatically update the corresponding DOM property (and vice versa).

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Setting a Single Attribute**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.attr() — Single Attribute</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <a id="link" href="https://example.com" title="Original title">Visit Example</a>
  <img id="photo" src="photo.jpg" alt="A photo">

  <script>
    $(function () {
      // Step 1: GET — read the href attribute of the first matched element
      var href = $("#link").attr("href");
      console.log("Href:", href); // "https://example.com"

      // Step 2: GET a non-existent attribute — returns undefined (jQuery 1.6+)
      var missing = $("#link").attr("data-nonexistent");
      console.log("Missing attr:", missing); // undefined

      // Step 3: SET — change the title attribute
      $("#link").attr("title", "Updated title");
      console.log("New title:", $("#link").attr("title")); // "Updated title"

      // Step 4: SET multiple attributes with an object
      $("#photo").attr({
        src: "new-photo.jpg",
        alt: "Updated alt text",
        width: "300"
      });
      console.log("Photo src:", $("#photo").attr("src")); // "new-photo.jpg"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Href: https://example.com
Missing attr: undefined
New title: Updated title
Photo src: new-photo.jpg
```

**Why This Output Occurs**
`.attr("href")` reads the `href` attribute from the first (and only) matched element. `.attr("data-nonexistent")` returns `undefined` because the attribute does not exist. The setter updates the DOM attribute, and a subsequent getter reads the updated value. The object form sets multiple attributes in a single call.

---

**Example 2: Using a Callback to Set Attributes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.attr() — Callback Function</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <img class="thumb" src="img1.jpg" alt="Thumbnail">
  <img class="thumb" src="img2.jpg" alt="Thumbnail">
  <img class="thumb" src="img3.jpg" alt="Thumbnail">

  <script>
    $(function () {
      // Set the alt attribute using a callback
      $(".thumb").attr("alt", function (index, oldValue) {
        // index is the position in the set; oldValue is the current alt
        return oldValue + " " + (index + 1);
      });

      // Log the results
      $(".thumb").each(function () {
        console.log("Alt:", $(this).attr("alt"));
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Alt: Thumbnail 1
Alt: Thumbnail 2
Alt: Thumbnail 3
```

**Why This Output Occurs**
The callback receives the element's index and its current `alt` value. Returning `oldValue + " " + (index + 1)` produces "Thumbnail 1", "Thumbnail 2", and "Thumbnail 3" for the three images.

### Real-World Cases

- **Link Management**: `$("a").attr("target", "_blank")` makes all links open in new tabs.
- **Image Alt Text**: `$("img").attr("alt", "Product image")` sets descriptive alt text for accessibility.
- **Dynamic URLs**: `$("a.download").attr("href", "/files/" + fileId + ".pdf")` builds download links dynamically.
- **Custom Data Attributes**: `$("#user").attr("data-role", "admin")` sets a `data-*` attribute visible in the DOM.

### References

- jQuery API — .attr() – https://api.jquery.com/attr/
- jQuery API — Attributes Category – https://api.jquery.com/category/attributes/
- W3Schools — jQuery attr() Method – https://www.w3schools.com/jquery/html_attr.asp

---

## Core Concept 2: `.removeAttr()` — Remove Attributes

### Definitions

**Core Definition**
`.removeAttr()` removes a specified attribute from each element in the set of matched elements.

**Technical Definition**
The `.removeAttr()` method uses the JavaScript `removeAttribute()` function, but it has the advantage of being callable directly on a jQuery object and accounts for different attribute naming across browsers. As of jQuery 3.0, `.removeAttr()` on a boolean attribute such as `checked`, `selected`, or `readonly` **no longer sets** the corresponding property to `false`; it simply removes the attribute.

**Beginner-Friendly Explanation**
`.removeAttr()` is the opposite of `.attr()` — it takes an attribute away entirely. If an element has a `disabled` attribute or a `title` attribute, `.removeAttr()` removes it from the HTML.

### Purposes

- To remove a specific attribute from all matched elements.
- To strip attributes that are no longer needed after a state change.
- To clean up dynamically generated attributes.
- To remove multiple attributes at once (space-separated).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).removeAttr(attributeName);
$(selector).removeAttr("attr1 attr2 attr3");
```

**Component Breakdown**

- `attributeName` (String): The name of the attribute to remove. Multiple attributes can be specified as a space-separated string (added in jQuery 1.7).
- Returns: A jQuery object (for chaining).

**Syntax Rules**

1. Removes the attribute from **every** matched element.
2. As of jQuery 1.7, multiple attributes can be removed by separating their names with spaces.
3. As of jQuery 3.0, removing a boolean attribute no longer sets its underlying property to `false`.
4. Removing a non-existent attribute does nothing and throws no error.

**Constraints and Limitations**

- Removing an attribute does not necessarily reset the corresponding DOM property to its default.
- For boolean attributes, prefer `.prop("checked", false)` over `.removeAttr("checked")` in jQuery 3+.

### Multiple Annotated Complete Code Examples

**Example 1: Removing a Single Attribute**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeAttr() — Single Attribute</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p class="demo" style="color: red;">Styled paragraph</p>
  <p class="demo" style="font-weight: bold;">Bold paragraph</p>

  <script>
    $(function () {
      // Step 1: Verify the style attribute exists
      console.log("Before:", $(".demo:first").attr("style")); // "color: red;"

      // Step 2: Remove the style attribute from all .demo elements
      $(".demo").removeAttr("style");

      // Step 3: Verify removal
      console.log("After:", $(".demo:first").attr("style")); // undefined
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Before: color: red;
After: undefined
```

**Why This Output Occurs**
`.removeAttr("style")` removes the `style` attribute from both paragraphs. The subsequent `.attr("style")` returns `undefined` because the attribute no longer exists.

---

**Example 2: Removing Multiple Attributes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeAttr() — Multiple Attributes</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="text" id="name" disabled="disabled" readonly="readonly"
         placeholder="Enter name" value="John">

  <script>
    $(function () {
      // Step 1: Remove multiple attributes with a space-separated string
      $("#name").removeAttr("disabled readonly placeholder");

      // Step 2: Verify they are gone
      console.log("Disabled:", $("#name").attr("disabled"));       // undefined
      console.log("Readonly:", $("#name").attr("readonly"));       // undefined
      console.log("Placeholder:", $("#name").attr("placeholder")); // undefined
      console.log("Value (still present):", $("#name").attr("value")); // "John"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Disabled: undefined
Readonly: undefined
Placeholder: undefined
Value (still present): John
```

**Why This Output Occurs**
The space-separated string `"disabled readonly placeholder"` tells jQuery to remove all three attributes. The `value` attribute was not listed and remains intact.

### Real-World Cases

- **Form Unlocking**: `$("input").removeAttr("disabled")` re-enables all form fields after a state change.
- **Cleanup After Validation**: `$(".error").removeAttr("data-error")` removes validation markers.
- **Dynamic Content Reset**: `$(".template").removeAttr("data-processed")` resets templates for re-processing.

### References

- jQuery API — .removeAttr() – https://api.jquery.com/removeAttr/
- W3Schools — jQuery removeAttr() Method – https://www.w3schools.com/jquery/html_removeattr.asp

---

## Core Concept 3: `.data()` — Get or Set Arbitrary Data

### Definitions

**Core Definition**
`.data()` stores arbitrary data associated with the matched elements or returns the value at the named data store for the first element in the set of matched elements.

**Technical Definition**
The `.data()` method attaches data of **any type** to DOM elements in a way that is safe from circular references and therefore from memory leaks. It maintains an internal JavaScript cache keyed to each element. When a key is first accessed, if the element has a corresponding `data-*` attribute, that value is read **once**, converted to the appropriate JavaScript type (number, boolean, object, or array where possible), and stored in the cache. Subsequent `.data()` calls return the cached value, even if the `data-*` attribute is later changed via `.attr()`. Using `.data()` to **set** a value updates the cache only — it does **not** modify the DOM.

**Beginner-Friendly Explanation**
`.data()` is like a hidden storage box attached to an element. You can put anything in it — numbers, strings, objects, arrays — and get it back later. The first time you ask for a value, jQuery checks if there is a matching `data-*` attribute in the HTML and uses that as the starting value. But after that, the box is separate from the HTML. Changing the HTML attribute later does not change what is in the box, and putting something new in the box does not change the HTML.

### Purposes

- To store arbitrary JavaScript data associated with DOM elements.
- To retrieve values from HTML5 `data-*` attributes with automatic type conversion.
- To associate complex objects (arrays, plain objects) with elements without serialising them.
- To cache data on elements for plugin state or application logic.
- To store multiple values at once using an object.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter — single key
$(selector).data(key);

// Getter — all data as an object
$(selector).data();

// Setter — single key-value pair
$(selector).data(key, value);

// Setter — multiple key-value pairs
$(selector).data({ key1: value1, key2: value2 });
```

**Component Breakdown**

- `key` (String): The name of the data to get or set. Hyphenated keys are camelCased on read (e.g., `data-my-name` → `myName`).
- `value` (Any): The value to store. Can be any JavaScript type except `undefined`.
- `{ key1: value1, ... }` (Object): An object of key-value pairs to set multiple values at once.
- Returns: The stored value (getter) or a jQuery object (setter).

**Syntax Rules**

1. As a getter, `.data(key)` returns the value for the **first** matched element.
2. `.data()` with no arguments returns all data as a JavaScript object.
3. Setting data via `.data()` does **not** modify `data-*` attributes in the DOM.
4. Reading a `data-*` attribute via `.data()` happens **once**; subsequent `.attr()` changes do not affect the cached value.
5. `.data()` converts HTML5 `data-*` values to JavaScript types: `"1"` → `1`, `"true"` → `true`, JSON strings → objects/arrays.
6. As of jQuery 3, hyphenated keys are converted to camelCase on read and write.

**Constraints and Limitations**

- `.data()` cannot be used on `<object>`, `<applet>`, or `<embed>` elements (unless the `<object>` is a Flash plugin).
- `undefined` is not recognised as a data value; `.data("key", undefined)` returns the jQuery object without setting anything.
- The `data-*` attribute is read only once; changing it later via `.attr()` does not update the cache.

### Multiple Annotated Complete Code Examples

**Example 1: Storing and Retrieving Data**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.data() — Store and Retrieve</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="user" data-user-id="42" data-role="admin" data-active="true"></div>

  <script>
    $(function () {
      // Step 1: Read data-* attributes via .data() with type conversion
      console.log("User ID:", $("#user").data("userId"));   // 42 (number)
      console.log("Role:", $("#user").data("role"));        // "admin" (string)
      console.log("Active:", $("#user").data("active"));    // true (boolean)

      // Step 2: Store a complex object
      $("#user").data("preferences", {
        theme: "dark",
        notifications: true
      });

      console.log("Theme:", $("#user").data("preferences").theme); // "dark"

      // Step 3: Get all data as an object
      var allData = $("#user").data();
      console.log("All data:", allData);
      // { userId: 42, role: "admin", active: true, preferences: {...} }
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
User ID: 42
Role: admin
Active: true
Theme: dark
All data: { userId: 42, role: "admin", active: true, preferences: { theme: "dark", notifications: true } }
```

**Why This Output Occurs**
`.data("userId")` reads the `data-user-id` attribute and converts `"42"` to the number `42`. `.data("active")` converts `"true"` to the boolean `true`. Setting `.data("preferences", {...})` stores a plain object in jQuery's cache. The bulk getter returns everything as a single object.

---

**Example 2: The One-Time Read Behaviour**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.data() — One-Time Read</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" data-value="initial"></div>

  <script>
    $(function () {
      // Step 1: First .data() call reads the attribute and caches it
      console.log("Initial data:", $("#box").data("value")); // "initial"

      // Step 2: Change the data-* attribute via .attr()
      $("#box").attr("data-value", "updated");

      // Step 3: .data() still returns the cached value
      console.log("After .attr():", $("#box").data("value")); // "initial"

      // Step 4: .attr() returns the new attribute value
      console.log("Via .attr():", $("#box").attr("data-value")); // "updated"

      // Step 5: Setting via .data() does not change the attribute
      $("#box").data("value", "from data()");
      console.log("After .data() set:", $("#box").data("value"));     // "from data()"
      console.log("Attr unchanged:", $("#box").attr("data-value"));    // "updated"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Initial data: initial
After .attr(): initial
Via .attr(): updated
After .data() set: from data()
Attr unchanged: updated
```

**Why This Output Occurs**
The first `.data("value")` call reads the `data-value` attribute once and caches `"initial"`. Subsequent changes to the attribute via `.attr()` do not affect the cache. Setting via `.data()` updates the cache only; the attribute remains `"updated"`. This demonstrates the **decoupling** between the attribute and the data cache.

### Real-World Cases

- **Plugin State**: `$("#slider").data("slider-state", { value: 50, max: 100 })` stores plugin configuration.
- **User Preferences**: `$("#user").data("theme", "dark")` caches user preferences without polluting the DOM.
- **Cached API Data**: `$("#product").data("details", apiResponse)` caches fetched data to avoid repeated requests.
- **HTML5 Configuration**: `<div data-config='{"mode":"live"}'>` read via `.data("config")` returns a parsed object.

### References

- jQuery API — .data() – https://api.jquery.com/data/
- jQuery Learning Center — Data Methods – https://learn.jquery.com/using-jquery-core/data-methods/
- Microsoft Learn — Understanding jQuery's Data API – https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/september/jquery-understanding-jquery-s-data-apis

---

## Core Concept 4: `.removeData()` — Remove Stored Data

### Definitions

**Core Definition**
`.removeData()` removes a previously-stored piece of data from the matched elements.

**Technical Definition**
The `.removeData()` method removes values that were previously set using `.data()`. When called with the name of a key, it deletes that particular value; when called with no arguments, it removes **all** values. Critically, `.removeData()` only removes data from jQuery's internal `.data()` cache — any corresponding `data-*` attributes on the element are **not** removed. A later call to `.data("key")` will therefore re-retrieve the value from the `data-key` attribute.

**Beginner-Friendly Explanation**
`.removeData()` is the way to take something out of the hidden storage box attached to an element. If you remove a key, the next time you ask for it, jQuery will look at the HTML `data-*` attribute again (if one exists) and use that as the value. To completely remove the data, you need to use both `.removeData()` and `.removeAttr()`.

### Purposes

- To remove specific stored data keys from jQuery's cache.
- To clear all stored data from an element.
- To reset plugin state stored on elements.
- To prevent stale data from persisting after state changes.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Remove a single key
$(selector).removeData(keyName);

// Remove multiple keys
$(selector).removeData(["key1", "key2"]);
$(selector).removeData("key1 key2");

// Remove all data
$(selector).removeData();
```

**Component Breakdown**

- `keyName` (String): The name of the data key to remove.
- `list` (Array | String): An array or space-separated string of keys to remove (added in jQuery 1.7).
- Returns: A jQuery object (for chaining).

**Syntax Rules**

1. Removes data from **jQuery's internal cache only**; `data-*` attributes are untouched.
2. Called with no arguments, it removes all stored data for the element.
3. As of jQuery 1.7, multiple keys can be removed with an array or space-separated string.
4. After `.removeData("key")`, a subsequent `.data("key")` will re-read the `data-key` attribute if it exists.

**Constraints and Limitations**

- Does not remove `data-*` attributes from the DOM; use `.removeAttr()` for that.
- The bulk `.data()` getter reads `data-*` attributes only once; `.removeData()` does not reset that behaviour.

### Multiple Annotated Complete Code Examples

**Example 1: Removing a Specific Data Key**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeData() — Single Key</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" data-test1="from-attribute" data-test2="persistent"></div>

  <script>
    $(function () {
      // Step 1: Set data values
      $("#box").data("test1", "VALUE-1");
      $("#box").data("test2", "VALUE-2");
      console.log("Before removal — test1:", $("#box").data("test1")); // "VALUE-1"
      console.log("Before removal — test2:", $("#box").data("test2")); // "VALUE-2"

      // Step 2: Remove test1 from the data cache
      $("#box").removeData("test1");

      // Step 3: test1 now falls back to the data-* attribute
      console.log("After removal — test1:", $("#box").data("test1")); // "from-attribute"
      console.log("After removal — test2:", $("#box").data("test2")); // "VALUE-2"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Before removal — test1: VALUE-1
Before removal — test2: VALUE-2
After removal — test1: from-attribute
After removal — test2: VALUE-2
```

**Why This Output Occurs**
Before removal, `.data("test1")` returns the cached value `"VALUE-1"`. After `.removeData("test1")`, the cache entry is deleted. The next `.data("test1")` call re-reads the `data-test1` attribute and returns `"from-attribute"`. The `test2` key was not removed and remains in the cache.

---

**Example 2: Removing All Data and the Attribute**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeData() — All Data</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="item" data-category="electronics" data-stock="5"></div>

  <script>
    $(function () {
      // Step 1: Set additional data and verify
      $("#item").data("price", 99.99);
      console.log("Category:", $("#item").data("category")); // "electronics"
      console.log("Price:", $("#item").data("price"));       // 99.99

      // Step 2: Remove ALL data from the cache
      $("#item").removeData();

      // Step 3: Attribute-sourced data is re-read; cache-only data is gone
      console.log("Category after:", $("#item").data("category")); // "electronics"
      console.log("Price after:", $("#item").data("price"));       // undefined

      // Step 4: To fully remove, also remove the attributes
      $("#item").removeAttr("data-category data-stock");
      console.log("Category after attr removal:",
        $("#item").data("category")); // undefined
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Category: electronics
Price: 99.99
Category after: electronics
Price after: undefined
Category after attr removal: undefined
```

**Why This Output Occurs**
`.removeData()` with no arguments clears the entire data cache. However, because `data-category` and `data-stock` attributes still exist in the DOM, the next `.data("category")` call re-reads the attribute and returns `"electronics"`. The `price` value, which was set only via `.data()`, is permanently gone. Removing the attributes with `.removeAttr()` completes the cleanup.

### Real-World Cases

- **Plugin Teardown**: `$("#widget").removeData()` clears all plugin state before re-initialisation.
- **State Reset**: `$("#form").removeData("validation")` resets validation state after submission.
- **Memory Management**: `$("#cache-target").removeData()` frees cached data when the element is no longer needed.
- **Full Cleanup**: `$("#item").removeData().removeAttr("data-category")` removes both cache and attribute.

### References

- jQuery API — .removeData() – https://api.jquery.com/removedata/
- W3Schools — jQuery removeData() Method – https://www.w3schools.com/jquery/misc_removedata.asp

---

## Core Concept 5: Handling Boolean Attributes

### Definitions

**Core Definition**
Boolean attributes are HTML attributes whose presence indicates `true` and whose absence indicates `false`; they have corresponding DOM **properties** that are updated dynamically as the element's state changes.

**Technical Definition**
Attributes such as `checked`, `selected`, `disabled`, and `readonly` are defined by the HTML specification as boolean attributes. The presence of the attribute in the HTML sets the initial state, but the **property** (`elem.checked`, `elem.disabled`) reflects the **current** state and changes as the user interacts with the element. Before jQuery 1.6, `.attr()` sometimes returned property values for these attributes, causing inconsistent behaviour. Since jQuery 1.6, `.attr()` retrieves the **attribute** (the initial state), while `.prop()` retrieves the **property** (the current state).

**Beginner-Friendly Explanation**
A checkbox has two things: an **attribute** in the HTML (`checked`) that says whether it started checked, and a **property** in JavaScript (`checked`) that says whether it is checked **right now**. If a user clicks the checkbox, the property changes but the attribute does not. jQuery's `.attr()` sees the starting state; `.prop()` sees the current state. For checkboxes, radio buttons, and disabled fields, you almost always want `.prop()`.

### Purposes

- To correctly read and set the current state of form controls.
- To avoid the pre-1.6 inconsistency where `.attr()` returned property values.
- To understand the difference between initial HTML state and dynamic JavaScript state.
- To choose the right method (`.prop()` vs. `.attr()`) for form element state.

### Syntax Rules and Structure

**Boolean Attribute Comparison**

| Method | Returns | Changes with State? | Use For |
|---|---|---|---|
| `elem.checked` | Boolean | Yes | Current checkbox state |
| `$(elem).prop("checked")` | Boolean | Yes | Current checkbox state |
| `$(elem).attr("checked")` (1.6+) | String (`"checked"` or `undefined`) | No | Initial HTML state |
| `$(elem).attr("checked")` (pre-1.6) | Boolean | Yes | (Deprecated behaviour) |

**Syntax Rules**

1. Use `.prop()` for boolean attributes: `checked`, `selected`, `disabled`, `readonly`, `multiple`, `autofocus`, `required`, `hidden`, `open`.
2. Use `.attr()` only for string attributes: `id`, `class`, `href`, `src`, `title`, `alt`, `data-*`.
3. As of jQuery 3.0, `.removeAttr("checked")` no longer sets the `checked` property to `false`.
4. Use `.is(":checked")` as an alternative for checking checkbox state.

**Constraints and Limitations**

- `.attr("checked")` returns `"checked"` (string) in jQuery 1.6–3.x, and `""` (empty string) in jQuery 4.0+; it never returns a boolean.
- `.attr("checked")` reflects only the initial HTML state, not the current checkbox state.

### Multiple Annotated Complete Code Examples

**Example 1: `.attr()` vs. `.prop()` for Checkboxes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Boolean Attributes — attr vs prop</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="checkbox" id="agree" checked>
  <label for="agree">I agree</label>
  <button id="toggle">Toggle</button>

  <script>
    $(function () {
      // Step 1: Initial state — both attr and prop return truthy values
      console.log("Initial .attr('checked'):", $("#agree").attr("checked")); // "checked"
      console.log("Initial .prop('checked'):", $("#agree").prop("checked")); // true

      // Step 2: Simulate a user click (changes the property, not the attribute)
      $("#agree").prop("checked", false);

      // Step 3: The property has changed, but the attribute has not
      console.log("After uncheck — .attr:", $("#agree").attr("checked")); // "checked" (unchanged)
      console.log("After uncheck — .prop:", $("#agree").prop("checked")); // false (changed)

      // Step 4: Toggle back
      $("#toggle").on("click", function () {
        var current = $("#agree").prop("checked");
        $("#agree").prop("checked", !current);
        console.log("Toggled to:", $("#agree").prop("checked"));
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Initial .attr('checked'): checked
Initial .prop('checked'): true
After uncheck — .attr: checked
After uncheck — .prop: false
Toggled to: true
```

**Why This Output Occurs**
`.attr("checked")` returns the string `"checked"` because the attribute is present in the HTML. `.prop("checked")` returns the boolean `true` because the checkbox is checked. After `.prop("checked", false)`, the property changes to `false` but the attribute remains `"checked"`. This demonstrates why `.prop()` is the correct choice for reading current state.

---

**Example 2: Disabled Attribute**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Boolean Attributes — Disabled</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="text" id="field" disabled>
  <button id="enable">Enable</button>

  <script>
    $(function () {
      // Step 1: Check initial state
      console.log("Disabled attr:", $("#field").attr("disabled")); // "" or "disabled"
      console.log("Disabled prop:", $("#field").prop("disabled")); // true

      // Step 2: Enable the field (correct way)
      $("#enable").on("click", function () {
        $("#field").prop("disabled", false);
        console.log("After enable — prop:", $("#field").prop("disabled")); // false
        console.log("After enable — attr:", $("#field").attr("disabled")); // undefined
      });

      // Step 3: Disable via prop
      $("#field").prop("disabled", true);
      console.log("Re-disabled — prop:", $("#field").prop("disabled")); // true
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Disabled attr: 
Disabled prop: true
After enable — prop: false
After enable — attr: undefined
Re-disabled — prop: true
```

**Why This Output Occurs**
`.attr("disabled")` returns an empty string (the attribute is present). `.prop("disabled")` returns `true`. After `.prop("disabled", false)`, the property is `false` and the attribute is removed entirely (returns `undefined` via `.attr()`).

### Real-World Cases

- **Select All Checkbox**: `$("#selectAll").prop("checked", true)` checks all child checkboxes.
- **Form Enable/Disable**: `$("#submit").prop("disabled", isValid)` enables or disables the submit button based on validation.
- **Radio Button Selection**: `$("input[name=color]").prop("checked", false)` clears all radio selections.
- **Readonly Toggle**: `$("#field").prop("readonly", true)` makes a field read-only without removing it.

### References

- jQuery API — .prop() – https://api.jquery.com/prop/
- jQuery API — .attr() (Boolean Attributes section) – https://api.jquery.com/attr/#attributes-vs-properties
- W3C — HTML Forms Specification: Boolean Attributes – https://www.w3.org/TR/html401/interact/forms.html#h-17.4
- jQuery Blog — jQuery 1.6 Released: .attr() and .prop() – https://blog.jquery.com/2011/05/03/jquery-16-released/

---

## Core Concept 6: `.attr()` vs. `.data()`: Attributes and the Data Cache

### Definitions

**Core Definition**
The key difference is that `.attr()` reads and writes **HTML attributes** (visible in the DOM), while `.data()` reads and writes **jQuery's internal data cache** (invisible in the DOM), with a one-time initialisation from `data-*` attributes.

**Technical Definition**
`.data()` is **not** an accessor for `data-*` attributes. It is an accessor for jQuery's data cache on the element. That cache is initialised from `data-*` attributes if any are present, but `.data()` **never writes** to the attributes, and changing the attribute **does not change** the data cache after initialisation. If you need to use the attributes (both reading and setting them), use `.attr()`, not `.data()`.`.attr()` is an accessor for attributes.

**Beginner-Friendly Explanation**
Think of an element as having two notebooks. The **attribute notebook** is written in pencil on the outside of the element — everyone can see it, and it is part of the HTML. The **data notebook** is a private journal kept by JavaScript — nobody can see it in the HTML, and it can hold anything. `.attr()` writes in the pencil notebook. `.data()` writes in the private journal. The first time you open the private journal, jQuery copies anything from the pencil notebook that starts with `data-`. After that, the two notebooks are completely separate.

### Purposes

- To understand why changing a `data-*` attribute with `.attr()` does not affect `.data()`.
- To choose the correct method for the desired outcome: DOM-visible changes require `.attr()`; JavaScript-side storage requires `.data()`.
- To avoid the common mistake of expecting `.data()` to update the DOM.
- To correctly synchronise attribute and data when both must change.

### Syntax Rules and Structure

**Behaviour Comparison Table**

| Action | `.attr("data-x", "value")` | `.data("x", "value")` |
|---|---|---|
| Updates DOM attribute | Yes | No |
| Updates data cache | No | Yes |
| Visible in developer tools | Yes | No |
| Accessible via CSS selectors | Yes | No |
| Persists across page serialisation | Yes | No |
| Returns data-* on first `.data()` read | Yes (source) | Yes (initial value) |

**Syntax Rules**

1. `.data()` reads `data-*` attributes **once** on first access; after that, the cache is independent.
2. `.attr()` changes to `data-*` attributes do **not** update the `.data()` cache.
3. `.data()` changes do **not** modify `data-*` attributes.
4. To synchronise both, use both methods: `.attr("data-x", "value").data("x", "value")`.
5. `.data()` automatically converts `data-*` string values to JavaScript types; `.attr()` always returns strings.

**Constraints and Limitations**

- There is no automatic synchronisation between attributes and the data cache.
- If you need the data to survive page serialisation (e.g., for form submission), use `.attr()`.
- If you need to store complex objects, use `.data()` — attributes can only hold strings.

### Multiple Annotated Complete Code Examples

**Example 1: Demonstrating the Decoupling**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.attr() vs .data() — Decoupling</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="item" data-price="10" data-name="Widget"></div>

  <script>
    $(function () {
      // --- Initial read via .data() initialises the cache ---
      console.log("Initial .data('price'):", $("#item").data("price")); // 10 (number)
      console.log("Initial .attr('data-price'):", $("#item").attr("data-price")); // "10" (string)

      // --- Change the attribute via .attr() ---
      $("#item").attr("data-price", "25");

      // --- The attribute changed, but the data cache did not ---
      console.log("After .attr() — .data('price'):", $("#item").data("price")); // 10 (stale)
      console.log("After .attr() — .attr('data-price'):", $("#item").attr("data-price")); // "25"

      // --- Change the data via .data() ---
      $("#item").data("price", 50);

      // --- The cache changed, but the attribute did not ---
      console.log("After .data() — .data('price'):", $("#item").data("price")); // 50
      console.log("After .data() — .attr('data-price'):", $("#item").attr("data-price")); // "25" (unchanged)

      // --- To synchronise, use both ---
      $("#item").attr("data-price", 99).data("price", 99);
      console.log("Synchronised — .data:", $("#item").data("price")); // 99
      console.log("Synchronised — .attr:", $("#item").attr("data-price")); // "99"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Initial .data('price'): 10
Initial .attr('data-price'): 10
After .attr() — .data('price'): 10
After .attr() — .attr('data-price'): 25
After .data() — .data('price'): 50
After .data() — .attr('data-price'): 25
Synchronised — .data: 99
Synchronised — .attr: 99
```

**Why This Output Occurs**
The initial `.data("price")` reads `"10"` from the `data-price` attribute and caches it as the number `10`. When `.attr("data-price", "25")` changes the attribute, the cache remains `10`. When `.data("price", 50)` changes the cache, the attribute remains `"25"`. Only using **both** methods keeps them in sync. This demonstrates the fundamental decoupling between attributes and the data cache.

---

**Example 2: Choosing the Right Method**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.attr() vs .data() — Choosing</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="product" data-sku="ABC-123" data-stock="42"></div>
  <span id="display"></span>

  <script>
    $(function () {
      // --- SCENARIO 1: Update the DOM for CSS/selectors ---
      // Use .attr() because the change must be visible in the DOM
      $("#product").attr("data-stock", "10");
      // Now CSS like [data-stock="0"] { opacity: 0.5; } would match
      console.log("DOM attribute:", $("#product").attr("data-stock")); // "10"

      // --- SCENARIO 2: Store complex application state ---
      // Use .data() because the object cannot be an attribute
      var productState = {
        lastFetched: Date.now(),
        priceHistory: [10, 12, 11],
        isPromoted: true
      };
      $("#product").data("state", productState);
      console.log("State price history:",
        $("#product").data("state").priceHistory); // [10, 12, 11]

      // --- SCENARIO 3: Read a configuration value from HTML ---
      // Use .data() for automatic type conversion
      var stock = $("#product").data("stock"); // number, not string
      console.log("Stock type:", typeof stock); // "number"

      // --- SCENARIO 4: Store a simple string for a plugin ---
      // Use .data() to avoid polluting the DOM
      $("#product").data("pluginId", "carousel-7");
      console.log("Plugin ID:", $("#product").data("pluginId")); // "carousel-7"
      // Note: no data-plugin-id attribute was added to the DOM
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
DOM attribute: 10
State price history: [10, 12, 11]
Stock type: number
Plugin ID: carousel-7
```

**Why This Output Occurs**
`.attr("data-stock", "10")` updates the DOM attribute, which is appropriate for CSS-driven state. `.data("state", {...})` stores a complex object that cannot be represented as an attribute. `.data("stock")` reads the `data-stock` attribute and converts `"42"` to the number `42`. `.data("pluginId", "carousel-7")` stores a value in the cache without adding anything to the DOM.

### Real-World Cases

- **CSS-Driven State**: Use `.attr("data-state", "loading")` so CSS selectors like `[data-state="loading"]` can style the element.
- **Plugin Configuration**: Use `.data("options", {...})` to store plugin settings as an object.
- **Form Serialisation**: Use `.attr()` for values that must be included in form submission or page serialisation.
- **Caching API Responses**: Use `.data("response", apiData)` to avoid re-fetching data.

### References

- jQuery API — .data() (Additional Notes) – https://api.jquery.com/data/#data-html5
- jQuery API — .attr() (Attributes vs. Properties) – https://api.jquery.com/attr/#attributes-vs-properties
- Stack Overflow — jQuery .data() does not work, but .attr() does – https://stackoverflow.com/questions/4514949/
- Stack Overflow — Difference between .attr() and .data() – https://stackoverflow.com/questions/7261619/

---

## Summary Table: Attributes and Custom Data at a Glance

| Method | Operates On | Getter Scope | Setter Effect | Visible in DOM? | Type Conversion |
|---|---|---|---|---|---|
| `.attr()` | HTML attributes | First matched element | All matched elements | Yes | No (always string) |
| `.removeAttr()` | HTML attributes | N/A | Removes from all matched elements | Yes (attribute removed) | N/A |
| `.data()` | jQuery data cache | First matched element | All matched elements (cache only) | No | Yes (automatic) |
| `.removeData()` | jQuery data cache | N/A | Removes from all matched elements (cache only) | No | N/A |

### Key Decision Guide

| Goal | Use |
|---|---|
| Read/write an HTML attribute (`href`, `src`, `title`) | `.attr()` |
| Read/write form state (`checked`, `disabled`) | `.prop()` |
| Store complex JavaScript data on an element | `.data()` |
| Update a `data-*` attribute for CSS/selectors | `.attr("data-*", ...)` |
| Read a `data-*` attribute with automatic type conversion | `.data()` |
| Remove an HTML attribute | `.removeAttr()` |
| Remove cached data | `.removeData()` |
| Fully clean up both attribute and data | `.removeData().removeAttr()` |

---

## General References

- jQuery API — Attributes Category – https://api.jquery.com/category/attributes/
- jQuery API — .attr() – https://api.jquery.com/attr/
- jQuery API — .removeAttr() – https://api.jquery.com/removeAttr/
- jQuery API — .data() – https://api.jquery.com/data/
- jQuery API — .removeData() – https://api.jquery.com/removedata/
- jQuery API — .prop() – https://api.jquery.com/prop/
- jQuery Learning Center — Data Methods – https://learn.jquery.com/using-jquery-core/data-methods/
- W3C — HTML Forms Specification: Boolean Attributes – https://www.w3.org/TR/html401/interact/forms.html#h-17.4
- HTML Living Standard — Custom Data Attributes – https://html.spec.whatwg.org/multipage/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes
- Microsoft Learn — Understanding jQuery's Data API – https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/september/jquery-understanding-jquery-s-data-apis
- Stack Overflow — Difference between .attr() and .data() – https://stackoverflow.com/questions/7261619/
- Stack Overflow — jQuery .data() does not work, but .attr() does – https://stackoverflow.com/questions/4514949/
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/