# jQuery Data Storage — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Data Storage is a mechanism for associating arbitrary JavaScript data with DOM elements, allowing developers to store and retrieve values without modifying the DOM or relying on global variables.

**Technical Definition:**
jQuery's data module provides methods to attach data to DOM elements via an internal cache (`jQuery.cache`) keyed by a unique expando property. The `.data()` and `$.data()` methods read from this cache, and from HTML5 `data-*` attributes as a fallback. The `.removeData()` and `$.removeData()` methods remove stored values, and `$.hasData()` checks whether an element has associated data. jQuery 3.x also supports the HTML5 `dataset` API natively. As of jQuery 3.0, the internal data storage mechanism was rewritten to avoid memory leaks and improve performance.

**Beginner-Friendly Explanation:**
jQuery Data Storage lets you attach information to page elements — like "this button belongs to user 42" or "this row has been selected" — without putting it in the HTML or using global variables. You store it with `.data()`, retrieve it with `.data()`, and remove it with `.removeData()`.

### Key Characteristics

- **Element-Associated:** Data is bound to specific DOM elements, not to selectors or documents.
- **Dual Source:** `.data()` reads from both jQuery's internal cache and HTML5 `data-*` attributes. The first read of a `data-*` attribute converts it to its JavaScript type (number, boolean, object, array, null, string).
- **Automatic JSON Parsing:** If a `data-*` attribute contains valid JSON, jQuery parses it automatically.
- **Caching:** Once a `data-*` attribute is read, its value is cached in jQuery's internal store. Subsequent reads use the cache.
- **Mutation Behavior:** Writing data with `.data(key, value)` updates the internal cache but does **not** update the DOM `data-*` attribute. Only `$.data()` and `.attr("data-key", value)` modify the DOM.
- **Version Sensitivity:** jQuery 1.6+ handles HTML5 data attributes; jQuery 3.0 rewrote the internal storage to eliminate memory leaks.

### Prerequisites

- Basic knowledge of HTML and the DOM
- Working knowledge of JavaScript objects and properties
- Familiarity with jQuery selectors and methods
- Understanding of HTML5 `data-*` attributes

### Related Programming Areas

- **HTML5 Data Attributes:** The standard mechanism for embedding custom data in HTML.
- **DOM Manipulation:** Data storage often accompanies element creation and removal.
- **Event Handling:** Storing state associated with event-bound elements.
- **Plugin Development:** Storing plugin state and options on elements.
- **Memory Management:** Proper cleanup of data when elements are removed.

### Core Concepts / Features

1. `.data()`
2. `.removeData()`
3. `$.data()`
4. `$.removeData()`
5. `$.hasData()`
6. Data Attributes
7. Data Storage
8. Data Retrieval
9. Data Performance
10. Data Best Practices

---

## 1. `.data()`

### Definitions

**Core Definition:** `.data()` is a jQuery method that stores or retrieves arbitrary data associated with the matched elements.

**Technical Definition:** `.data(key, value)` stores a value; `.data(key)` retrieves a value; `.data()` (no arguments) returns an object containing all data for the first element in the set. When reading, jQuery first checks the internal cache. If the key is not found, it reads the corresponding `data-*` HTML attribute and parses it (numbers become numbers, `"true"` becomes `true`, JSON becomes objects). The value is then cached for subsequent reads. When writing, `.data()` updates the internal cache only — the DOM attribute is not modified.

**Beginner-Friendly Explanation:** `.data()` is how you save and retrieve information on an element. You can save numbers, strings, arrays, or objects, and get them back later.

### Purposes

- To store state or metadata on DOM elements without global variables.
- To retrieve values from HTML5 `data-*` attributes with automatic type conversion.
- To associate application data with UI components.

### Syntax Rules and Structure

```javascript
// Store data
$(selector).data(key, value);

// Retrieve data
$(selector).data(key);

// Retrieve all data
$(selector).data();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `.data(key, value)` | Store `value` under `key`. |
| `.data(key)` | Retrieve the value for `key`. |
| `.data()` | Return all data as an object. |
| `.data(obj)` | Set multiple key/value pairs. |

**Syntax Rules:**
1. Keys are strings; values can be any JavaScript type.
2. Keys with dashes (`data-user-id`) become camelCase (`userId`) when accessed.
3. Reading a `data-*` attribute parses JSON automatically.
4. Writing does not update the DOM attribute.

**Constraints and Limitations:**
- `.data()` on a collection returns the value for the first element only (for reads).
- Writing to `.data()` does not affect the DOM, so `[data-*]` selectors won't reflect updates.
- Keys with `.` or `-` may cause unexpected behavior; use camelCase.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Storing and Retrieving Data**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.data() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="card" data-user-id="42" data-active="true">User Card</div>
    <div id="output"></div>
    <script>
        // Step 1: Read from data-* attributes (auto-parsed)
        var userId = $("#card").data("userId");
        var isActive = $("#card").data("active");
        $("#output").append("userId: " + userId + " (type: " + typeof userId + ")<br>");
        $("#output").append("active: " + isActive + " (type: " + typeof isActive + ")<br>");

        // Step 2: Store new data on the element
        $("#card").data("selected", true);
        $("#card").data("tags", ["vip", "active"]);

        // Step 3: Retrieve stored data
        $("#output").append("selected: " + $("#card").data("selected") + "<br>");
        $("#output").append("tags: " + $("#card").data("tags").join(", ") + "<br>");

        // Step 4: Get all data at once
        console.log("All data:", $("#card").data());
    </script>
</body>
</html>
```

**Expected Output:**
```
userId: 42 (type: number)
active: true (type: boolean)
selected: true
tags: vip, active
```
Console logs all data as an object.

**Why This Code Produces That Result:** `data-user-id="42"` is read as the key `userId` and parsed as a number (42). `data-active="true"` is parsed as a boolean. New data stored with `.data("selected", true)` and `.data("tags", [...])` is kept in jQuery's internal cache.

### Real-World Cases
- **UI State:** Storing selected rows, expanded panels, or active tabs.
- **Plugin Configuration:** Storing plugin options on initialized elements.
- **Component Metadata:** Attaching record IDs to rendered list items.

---

## 2. `.removeData()`

### Definitions

**Core Definition:** `.removeData()` removes data previously stored on elements using `.data()`.

**Technical Definition:** `.removeData(key)` removes the value for the specified key. `.removeData()` (no arguments) removes all data stored on the element. `.removeData([list])` removes multiple keys. Note that `.removeData()` does not remove `data-*` attributes; it only clears the internal cache. If a `data-*` attribute still exists, a subsequent `.data()` call will re-read it.

**Beginner-Friendly Explanation:** `.removeData()` deletes information you previously saved on an element.

### Purposes
- To clean up data when it is no longer needed.
- To prevent memory leaks in long-lived applications.
- To reset element state before re-rendering.

### Syntax Rules and Structure

```javascript
$(selector).removeData(key);
$(selector).removeData();
$(selector).removeData([key1, key2]);
```

**Syntax Rules:**
1. Removes from jQuery's cache, not the DOM.
2. Without arguments, removes all data.
3. `data-*` attributes remain in the DOM.

**Constraints and Limitations:**
- Does not remove `data-*` attributes.
- If the attribute still exists, `.data()` will re-read it.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Removing Stored Data**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.removeData() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" data-role="container">Box</div>
    <div id="output"></div>
    <script>
        // Step 1: Store data
        $("#box").data("status", "active");
        $("#box").data("count", 5);
        $("#output").append("Before: status=" + $("#box").data("status") + "<br>");

        // Step 2: Remove specific key
        $("#box").removeData("status");
        $("#output").append("After remove status: " + $("#box").data("status") + "<br>");

        // Step 3: Remove all data
        $("#box").removeData();
        $("#output").append("After remove all: count=" + $("#box").data("count") + "<br>");

        // Step 4: data-* attribute still readable
        $("#output").append("data-role still exists: " + $("#box").data("role"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Before: status=active
After remove status: undefined
After remove all: count=undefined
data-role still exists: container
```

**Why This Code Produces That Result:** `.removeData("status")` clears the cached value; `.data("status")` returns `undefined`. `.removeData()` clears all cached data. However, the `data-role` attribute is still in the DOM, so `.data("role")` reads it again.

### Real-World Cases
- **Component Teardown:** Cleaning up data before removing elements.
- **State Reset:** Clearing selection or expansion state.
- **SPA Navigation:** Removing data when views are destroyed.

---

## 3. `$.data()`

### Definitions

**Core Definition:** `$.data()` is a low-level static method for storing and retrieving data on a DOM element.

**Technical Definition:** `$.data(element, key, value)` stores `value` on the DOM element. `$.data(element, key)` retrieves the value. Unlike `.data()`, `$.data()` does not read `data-*` attributes; it only accesses jQuery's internal store.

**Beginner-Friendly Explanation:** `$.data()` is the low-level version of `.data()` — it works directly with DOM elements and does not read HTML attributes.

### Purposes
- To store/retrieve data when you have a raw DOM element, not a jQuery object.
- To avoid the automatic `data-*` attribute reading of `.data()`.

### Syntax Rules and Structure

```javascript
$.data(element, key, value);
var value = $.data(element, key);
```

**Syntax Rules:**
1. First argument is a DOM element.
2. Does not read `data-*` attributes.
3. Returns the stored value or `undefined`.

**Constraints and Limitations:**
- Requires a DOM element, not a selector.
- Does not parse `data-*` attributes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using `$.data()` with a DOM Element**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.data() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="el" data-initial="from-html">Element</div>
    <div id="output"></div>
    <script>
        var el = document.getElementById("el");

        // Step 1: Store via $.data()
        $.data(el, "custom", "stored-value");

        // Step 2: Retrieve via $.data()
        $("#output").append("Custom: " + $.data(el, "custom") + "<br>");

        // Step 3: $.data does NOT read data-* attributes
        $("#output").append("Initial (via $.data): " + $.data(el, "initial") + "<br>");

        // Step 4: .data() DOES read data-* attributes
        $("#output").append("Initial (via .data): " + $(el).data("initial"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Custom: stored-value
Initial (via $.data): undefined
Initial (via .data): from-html
```

**Why This Code Produces That Result:** `$.data()` accesses only jQuery's internal cache, so `data-initial` is not read (returns `undefined`). `.data()` reads the HTML attribute and returns `"from-html"`.

### Real-World Cases
- **Native DOM Integration:** Storing data on elements referenced by native DOM APIs.
- **Performance:** Avoiding attribute parsing when only internal data is needed.

---

## 4. `$.removeData()`

### Definitions

**Core Definition:** `$.removeData()` is a low-level static method for removing data from a DOM element.

**Technical Definition:** `$.removeData(element, key)` removes the value for `key` from the element's internal cache. `$.removeData(element)` removes all data. Like `$.data()`, it does not touch `data-*` attributes.

**Beginner-Friendly Explanation:** `$.removeData()` is the low-level version of `.removeData()` — it works directly with DOM elements.

### Purposes
- To remove data from raw DOM elements.
- To clean up internal cache without touching HTML attributes.

### Syntax Rules and Structure

```javascript
$.removeData(element, key);
$.removeData(element);
```

**Syntax Rules:**
1. First argument is a DOM element.
2. Does not modify `data-*` attributes.

**Constraints and Limitations:**
- Requires a DOM element.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Removing Data with `$.removeData()`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.removeData() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="el">Element</div>
    <div id="output"></div>
    <script>
        var el = document.getElementById("el");
        $.data(el, "temp", "value");
        $("#output").append("Before: " + $.data(el, "temp") + "<br>");

        $.removeData(el, "temp");
        $("#output").append("After: " + $.data(el, "temp"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Before: value
After: undefined
```

**Why This Code Produces That Result:** `$.removeData()` clears the internal cache entry, so subsequent reads return `undefined`.

### Real-World Cases
- **Native DOM Cleanup:** Removing data from elements managed by native code.

---

## 5. `$.hasData()`

### Definitions

**Core Definition:** `$.hasData()` determines whether an element has any data associated with it in jQuery's internal cache.

**Technical Definition:** `jQuery.hasData(element)` returns `true` if the element has data stored via jQuery, `false` otherwise. It does not count `data-*` attributes that have not yet been read.

**Beginner-Friendly Explanation:** `$.hasData()` checks if an element has any jQuery-stored data.

### Purposes
- To check if an element has associated data before cleanup.
- To avoid unnecessary `.removeData()` calls.
- To debug data storage.

### Syntax Rules and Structure

```javascript
$.hasData(element); // true or false
```

**Syntax Rules:**
1. Requires a DOM element.
2. Does not count unread `data-*` attributes.

**Constraints and Limitations:**
- Only checks jQuery's internal cache.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Checking for Data**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.hasData() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="el" data-attr="html-value">Element</div>
    <div id="output"></div>
    <script>
        var el = document.getElementById("el");

        // Step 1: Before reading, no internal data
        $("#output").append("Before read: " + $.hasData(el) + "<br>");

        // Step 2: Read the data-* attribute (caches it)
        $(el).data("attr");
        $("#output").append("After read: " + $.hasData(el) + "<br>");

        // Step 3: Store custom data
        $(el).data("custom", "value");
        $("#output").append("After store: " + $.hasData(el));
    </script>
</body>
</html>
```

**Expected Output:**
```
Before read: false
After read: true
After store: true
```

**Why This Code Produces That Result:** `$.hasData()` returns `false` before any data is cached. Reading the `data-*` attribute caches it, making `$.hasData()` return `true`.

### Real-World Cases
- **Cleanup Logic:** Checking if data exists before removal.
- **Debugging:** Verifying data storage.

---

## 6. Data Attributes

### Definitions

**Core Definition:** HTML5 `data-*` attributes are custom attributes that store extra information on standard HTML elements without requiring a specific schema.

**Technical Definition:** Attributes of the form `data-*` are part of the HTML5 specification. jQuery's `.data()` reads these attributes and converts their values to JavaScript types: `"true"`/`"false"` → boolean, numeric strings → number, JSON strings → objects/arrays, and other strings remain strings.

**Beginner-Friendly Explanation:** `data-*` attributes let you embed custom data directly in your HTML, and jQuery reads them easily.

### Purposes
- To embed data directly in HTML markup.
- To avoid extra AJAX requests for initial data.
- To provide semantic hooks for JavaScript.

### Syntax Rules and Structure

```html
<div data-user-id="42" data-active="true" data-tags='["vip","new"]'></div>
```

```javascript
$("div").data("userId");  // 42 (number)
$("div").data("active");  // true (boolean)
$("div").data("tags");    // ["vip", "new"] (array)
```

**Syntax Rules:**
1. Attributes must begin with `data-`.
2. Dashed names convert to camelCase: `data-user-id` → `userId`.
3. Values are parsed as JSON when possible.
4. Access via `.data("key")` or `.attr("data-key")`.

**Constraints and Limitations:**
- Values are strings in the DOM; jQuery parses them.
- Case-sensitive conversion: `data-user-id` → `userId`, not `user-id`.
- Writing with `.data()` does not update the attribute.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading Various Data Attribute Types**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Data Attributes</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="el"
         data-name="John"
         data-age="30"
         data-active="true"
         data-score="4.5"
         data-tags='["a","b"]'
         data-config='{"theme":"dark"}'>
    </div>
    <div id="output"></div>
    <script>
        var $el = $("#el");
        $("#output").append("name: " + $el.data("name") + " (" + typeof $el.data("name") + ")<br>");
        $("#output").append("age: " + $el.data("age") + " (" + typeof $el.data("age") + ")<br>");
        $("#output").append("active: " + $el.data("active") + " (" + typeof $el.data("active") + ")<br>");
        $("#output").append("score: " + $el.data("score") + " (" + typeof $el.data("score") + ")<br>");
        $("#output").append("tags: " + $el.data("tags").join(",") + " (" + typeof $el.data("tags") + ")<br>");
        $("#output").append("config.theme: " + $el.data("config").theme);
    </script>
</body>
</html>
```

**Expected Output:**
```
name: John (string)
age: 30 (number)
active: true (boolean)
score: 4.5 (number)
tags: a,b (object)
config.theme: dark
```

**Why This Code Produces That Result:** jQuery parses `data-*` values: numeric strings become numbers, `"true"` becomes boolean, and JSON arrays/objects are parsed. The `typeof` checks confirm the JavaScript types.

### Real-World Cases
- **Server-Rendered Data:** Embedding initial state in HTML.
- **Configuration:** Passing options to plugins via attributes.
- **Analytics:** Storing tracking metadata on elements.

---

## 7. Data Storage

### Definitions

**Core Definition:** Data storage in jQuery refers to the act of persisting values in jQuery's internal cache associated with DOM elements.

**Technical Definition:** jQuery stores data in an internal `jQuery.cache` object keyed by a unique expando property (`jQuery.expando`) added to each element. This avoids polluting the DOM with custom attributes and allows storing any JavaScript value (not just strings).

**Beginner-Friendly Explanation:** Data storage is how jQuery remembers values on elements without putting them in the HTML.

### Purposes
- To store non-string values (objects, arrays, functions) on elements.
- To keep data out of the DOM for cleanliness and performance.

### Syntax Rules and Structure

```javascript
$(el).data("key", value);           // jQuery object
$.data(element, "key", value);      // DOM element
```

**Syntax Rules:**
1. Stored in `jQuery.cache`, not the DOM.
2. Any JavaScript value can be stored.
3. Element gets an expando property for lookup.

**Constraints and Limitations:**
- Data is lost when the element is removed without cleanup.
- Cache is not serialized.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Storing Complex Values**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Data Storage</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="widget">Widget</div>
    <div id="output"></div>
    <script>
        // Step 1: Store a complex object
        $("#widget").data("settings", {
            theme: "dark",
            size: "large",
            callbacks: {
                onClose: function() { console.log("Closed"); }
            }
        });

        // Step 2: Retrieve and use it
        var settings = $("#widget").data("settings");
        $("#output").text("Theme: " + settings.theme + ", Size: " + settings.size);

        // Step 3: Call a stored callback
        settings.callbacks.onClose();
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Theme: dark, Size: large". Console logs "Closed".

**Why This Code Produces That Result:** The complex object (including nested functions) is stored in jQuery's cache and retrieved intact. The callback is invoked successfully.

### Real-World Cases
- **Plugin State:** Storing options and state objects.
- **Event Handlers:** Storing callback references for later invocation.

---

## 8. Data Retrieval

### Definitions

**Core Definition:** Data retrieval is the process of reading values previously stored on elements or defined in `data-*` attributes.

**Technical Definition:** `.data(key)` checks jQuery's internal cache first. If not found, it reads the `data-*` attribute, parses it (JSON), and caches the result. Subsequent reads use the cache. `.data()` with no arguments returns all data for the first element.

**Beginner-Friendly Explanation:** Data retrieval is how you get back information you stored on an element.

### Purposes
- To access stored state and configuration.
- To read initial data from HTML.
- To retrieve all data for debugging.

### Syntax Rules and Structure

```javascript
$(el).data("key");        // single value
$(el).data();             // all data (object)
$.data(element, "key");   // DOM element access
```

**Syntax Rules:**
1. Cache is checked first.
2. `data-*` attributes are read on cache miss.
3. `.data()` returns all data for the first element.

**Constraints and Limitations:**
- `.data()` on a collection returns the first element's value for key reads.
- Cached values take precedence over attributes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Cache vs. Attribute Retrieval**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Data Retrieval</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="el" data-value="from-html">Element</div>
    <div id="output"></div>
    <script>
        var $el = $("#el");

        // Step 1: First read — from attribute, cached
        $("#output").append("1st: " + $el.data("value") + "<br>");

        // Step 2: Change the attribute directly
        $el.attr("data-value", "changed-in-dom");

        // Step 3: .data() still returns cached value
        $("#output").append("2nd (cached): " + $el.data("value") + "<br>");

        // Step 4: .attr() reflects the DOM
        $("#output").append("attr: " + $el.attr("data-value"));
    </script>
</body>
</html>
```

**Expected Output:**
```
1st: from-html
2nd (cached): from-html
attr: changed-in-dom
```

**Why This Code Produces That Result:** The first `.data("value")` read the attribute and cached it. Changing the attribute with `.attr()` does not update the cache, so `.data()` still returns the cached value. `.attr()` reads the current DOM attribute.

### Real-World Cases
- **Initial State:** Reading server-rendered data.
- **Debugging:** Inspecting all data on an element.

---

## 9. Data Performance

### Definitions

**Core Definition:** Data performance refers to the efficiency of storing, retrieving, and removing element-associated data, and the impact of these operations on memory and execution speed.

**Technical Definition:** jQuery's data cache avoids DOM attribute manipulation, which is slower than object property access. However, caching values indefinitely can cause memory leaks if elements are removed without cleanup. Reading `data-*` attributes triggers JSON parsing, which is slower than cache reads. jQuery 3.0 improved the internal data structure to reduce memory overhead.

**Beginner-Friendly Explanation:** Storing data in jQuery's cache is fast, but if you don't clean up when elements are removed, memory can grow. Reading from HTML attributes is slower than reading from the cache.

### Purposes
- To understand the performance implications of data operations.
- To avoid memory leaks in long-lived applications.
- To choose the right storage mechanism.

### Syntax Rules and Structure

```javascript
// Fast: cache read
$(el).data("cached");  // internal cache

// Slower: attribute read + parse (first time only)
$(el).data("fromHtml");  // reads data-* and caches

// Cleanup: prevent leaks
$(el).removeData().off();
```

**Syntax Rules:**
1. Cache reads are faster than attribute reads.
2. Removing elements with `.remove()` cleans up data automatically.
3. Use `.removeData()` for elements that remain in the DOM.

**Constraints and Limitations:**
- Retained data on removed elements can leak memory.
- JSON parsing of attributes has a cost.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Cache vs. Attribute Performance**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Data Performance</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="el" data-config='{"a":1,"b":2,"c":3}'>Element</div>
    <div id="output"></div>
    <script>
        // Step 1: First read — parses JSON (slower)
        console.time("first read");
        var config1 = $("#el").data("config");
        console.timeEnd("first read");

        // Step 2: Second read — from cache (faster)
        console.time("second read");
        var config2 = $("#el").data("config");
        console.timeEnd("second read");

        $("#output").text("a=" + config2.a + ", b=" + config2.b);

        // Step 3: Clean up when done
        $("#el").removeData();
        console.log("Data removed");
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "a=1, b=2". Console shows the first read takes longer (JSON parsing) than the second (cache hit).

**Why This Code Produces That Result:** The first `.data("config")` reads the attribute and parses the JSON, which is slower. The second read retrieves the cached object, which is fast.

### Real-World Cases
- **High-Frequency Access:** Caching data for repeated reads.
- **Memory Audits:** Cleaning up data to prevent leaks.

---

## 10. Data Best Practices

### Definitions

**Core Definition:** Data Best Practices are guidelines for using jQuery's data storage effectively, safely, and performantly.

**Technical Definition:** Best practices encompass preferring `.data()` for element-associated state, cleaning up with `.removeData()` or `.remove()`, avoiding storing large objects, being aware that `.data()` writes do not update the DOM, using `data-*` attributes for initial server-rendered data, and avoiding keys that conflict with jQuery internals.

### Purposes

- To prevent memory leaks.
- To maintain consistent data access patterns.
- To optimize performance.
- To avoid common pitfalls with `data-*` attributes.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Use `.data()` for element state | Global variables | `$(el).data("state", ...)` |
| 2 | Clean up when removing elements | `.detach()` without cleanup | `.remove()` (auto-cleans) |
| 3 | Use `data-*` for initial data | AJAX for small configs | `data-config='{...}'` |
| 4 | Avoid huge objects in data | Storing megabytes | Store references or IDs |
| 5 | Remember writes don't update DOM | Expecting `[data-x]` to match | Use `.attr()` if DOM update needed |
| 6 | Use camelCase keys | `data("user-id")` | `data("userId")` |
| 7 | Remove data before re-render | Leaving stale data | `.removeData()` then re-render |
| 8 | Use `$.hasData()` before cleanup | Blind removal | Check first |
| 9 | Don't store functions in HTML | `data-fn="..."` | Store functions via `.data()` |
| 10 | Use `$.data()` for raw elements | `$(el).data()` in hot loops | `$.data(el, key)` |

**Syntax Rules:**
1. Use `.data()` for application state associated with elements.
2. Clean up with `.removeData()` or `.remove()` to prevent leaks.
3. Store initial data in `data-*` attributes; access via `.data()`.
4. Avoid storing large objects; store IDs or references instead.
5. Use camelCase keys consistently.
6. Remember that `.data()` writes do not update the DOM.

**Constraints and Limitations:**
- `data-*` attributes are strings in the DOM; jQuery parses them.
- Cached data can become stale if attributes are changed externally.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Data Best Practices**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Data Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <ul id="list">
        <li data-id="1" data-status="active">Item 1</li>
        <li data-id="2" data-status="inactive">Item 2</li>
    </ul>
    <button id="clear">Clear Data</button>
    <div id="output"></div>
    <script>
        // ✅ BEST PRACTICE: Use data-* for initial data
        $("#list li").each(function() {
            var $li = $(this);
            var id = $li.data("id");       // parsed as number
            var status = $li.data("status"); // string
            $("#output").append("ID: " + id + ", Status: " + status + "<br>");
        });

        // ✅ BEST PRACTICE: Store app state with .data()
        $("#list li").on("click", function() {
            var $li = $(this);
            $li.data("selected", !$li.data("selected"));
            $li.toggleClass("selected");
            console.log("Selected:", $li.data("selected"));
        });

        // ✅ BEST PRACTICE: Clean up data when done
        $("#clear").on("click", function() {
            $("#list li").each(function() {
                // ✅ Check before removing
                if ($.hasData(this)) {
                    $(this).removeData();
                }
            });
            $("#output").append("All data cleared<br>");
        });

        // ✅ BEST PRACTICE: Use .remove() which auto-cleans data
        // $("#list li:first").remove();
    </script>
</body>
</html>
```

**Expected Output:** The output div lists each item's ID and status. Clicking items toggles selection state. Clicking "Clear Data" removes all stored data and appends "All data cleared".

**Why This Code Produces That Result:** The code uses `data-*` attributes for initial data, `.data()` for application state, `$.hasData()` before cleanup, and `.removeData()` to prevent leaks. This demonstrates clean, performant data management.

### Real-World Cases

- **List Components:** Storing item IDs and selection state.
- **Plugins:** Storing options and state on initialized elements.
- **SPA Views:** Cleaning up data on route changes.

---

## References

- jQuery API Documentation — .data() - https://api.jquery.com/data/
- jQuery API Documentation — .removeData() - https://api.jquery.com/removeData/
- jQuery API Documentation — jQuery.data() - https://api.jquery.com/jQuery.data/
- jQuery API Documentation — jQuery.removeData() - https://api.jquery.com/jQuery.removeData/
- jQuery API Documentation — jQuery.hasData() - https://api.jquery.com/jQuery.hasData/
- jQuery Learning Center — Data Methods - https://learn.jquery.com/using-jquery-core/data-methods/
- MDN Web Docs — Using data-* attributes - https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes
- MDN Web Docs — HTMLElement.dataset - https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset
- W3C — HTML5 Data Attributes Specification - https://html.spec.whatwg.org/multipage/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes
- jQuery Blog — jQuery 3.0 Final Released - https://blog.jquery.com/2016/06/09/jquery-3-0-final-released/