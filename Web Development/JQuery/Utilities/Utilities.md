# jQuery Utilities — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Utilities are a collection of static functions attached directly to the `jQuery` (or `$`) namespace that provide general-purpose helpers for iteration, type checking, array manipulation, string handling, deferred coordination, and other common programming tasks — independent of DOM manipulation.

**Technical Definition:**
`jQuery.*` utility functions are static methods on the `jQuery` object (not on jQuery instances). They include iteration helpers (`$.each()`, `$.map()`, `$.grep()`), type inspection (`$.type()`, `$.isArray()`, `$.isFunction()`, `$.isNumeric()`, `$.isPlainObject()`, `$.isEmptyObject()`, `$.isWindow()`, `$.isXMLDoc()`), array/object manipulation (`$.inArray()`, `$.merge()`, `$.extend()`, `$.uniqueSort()`), string and parsing utilities (`$.trim()`, `$.parseHTML()`, `$.parseJSON()`, `$.parseXML()`), function utilities (`$.proxy()`, `$.noop()`), time helpers (`$.now()`), DOM helpers (`$.contains()`, `$.escapeSelector()`), global evaluation (`$.globalEval()`), ready state management (`$.holdReady()`, `$.ready`), and asynchronous coordination (`$.when()`, `$.Deferred()`). Many of these mirror native ES5+ methods (`Array.prototype.forEach`, `Array.prototype.map`, `Array.prototype.filter`, `Array.isArray`, `JSON.parse`), which are now preferred in modern code.

**Beginner-Friendly Explanation:**
jQuery Utilities are like a Swiss Army knife of helper functions. Instead of writing your own loops, type checks, or array searches, you can use `$.each()`, `$.isArray()`, `$.inArray()`, and many more. They work on plain JavaScript data — not just DOM elements — and save you from writing repetitive code.

### Key Characteristics

- **Static Methods:** All utilities are called as `$.functionName()`, not on jQuery objects.
- **Native Equivalents:** Many utilities have native JavaScript equivalents introduced in ES5/ES6, making them redundant in modern code (`$.trim()` → `String.prototype.trim()`, `$.isArray()` → `Array.isArray()`, `$.parseJSON()` → `JSON.parse()`, `$.each()` → `Array.prototype.forEach()`).
- **Deprecation Status:** Several utilities are deprecated: `$.isArray()`, `$.isFunction()`, `$.isNumeric()`, `$.isWindow()`, `$.type()`, `$.trim()`, `$.parseJSON()`, `$.now()`, `$.proxy()`, `$.nodeName()`, `$.globalEval()`. They remain for backward compatibility but should be replaced with native equivalents.
- **Version Sensitivity:** `$.parseJSON()` was deprecated in jQuery 3.0 (use `JSON.parse()`); `$.isFunction()` and `$.isArray()` were deprecated in jQuery 3.3; `$.trim()` was deprecated in jQuery 3.5.
- **Cross-Cutting Concerns:** Utilities span iteration, type checking, parsing, function binding, and async coordination.
- **Independent of DOM:** Utilities operate on JavaScript values, not jQuery objects.

### Prerequisites

- Basic knowledge of JavaScript (objects, arrays, functions)
- Familiarity with jQuery syntax and the `$` namespace
- Understanding of JavaScript data types and type coercion
- Basic experience with browser developer tools (Console)

### Related Programming Areas

- **Native JavaScript APIs:** ES5+ methods (`Array.isArray`, `JSON.parse`, `Function.prototype.bind`) supersede many jQuery utilities.
- **Functional Programming:** `$.each()`, `$.map()`, and `$.grep()` are functional iteration helpers.
- **Asynchronous Programming:** `$.Deferred()` and `$.when()` are jQuery's Promise implementation.
- **Type Inspection:** Utilities like `$.type()` provide jQuery-specific type detection.
- **DOM Utilities:** `$.contains()`, `$.escapeSelector()`, and `$.parseHTML()` bridge JavaScript and the DOM.

---

## 1. `$.each()`

### Definitions

**Core Definition:** `$.each()` is a generic iterator function that loops over arrays and objects, executing a callback for each item.

**Technical Definition:** `jQuery.each(collection, callback)` iterates over arrays (by index) and objects (by key). The callback receives `(indexInArray, valueOfElement)` and `this` bound to the current value. Returning `false` from the callback breaks the loop, equivalent to `break` in a `for` loop. For arrays, `$.each()` iterates by numeric index; for objects, it iterates by property name.

**Beginner-Friendly Explanation:** `$.each()` is jQuery's version of a `for` loop. You give it a list or object, and it runs your function for each item.

### Purposes

- To iterate over arrays and objects with a consistent syntax.
- To perform an action on each item in a collection.
- To break out of a loop early by returning `false`.

### Syntax Rules and Structure

```javascript
$.each(collection, function(index, value) {
    // index: array index or object key
    // value: the item
    // this === value
    // return false to break
});
```

**Syntax Rules:**
1. Works on both arrays and objects.
2. Callback receives `(index, value)`.
3. `this` is bound to the current value.
4. Returning `false` breaks the loop.

**Constraints and Limitations:**
- Slower than native `for` loops and `Array.prototype.forEach()`.
- Does not work on strings (use a `for` loop or split into an array).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Iterating Arrays and Objects**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.each() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Iterate an array
        var fruits = ["apple", "banana", "cherry"];
        $.each(fruits, function(index, value) {
            $("#output").append(index + ": " + value + "<br>");
        });

        // Step 2: Iterate an object
        var person = { name: "John", age: 30, city: "NYC" };
        $.each(person, function(key, value) {
            $("#output").append(key + " = " + value + "<br>");
        });

        // Step 3: Break early by returning false
        $.each(fruits, function(index, value) {
            if (value === "banana") return false;
            console.log("Before break:", value);
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
0: apple
1: banana
2: cherry
name = John
age = 30
city = NYC
```
Console logs `Before break: apple` only (loop breaks at "banana").

**Why This Code Produces That Result:** `$.each()` iterates arrays by index and objects by key. The callback receives the index/key and value. Returning `false` in the third loop stops iteration before "banana" is processed.

### Real-World Cases
- **Iterating API Responses:** Processing arrays of data.
- **Object Property Loops:** Iterating configuration objects.
- **Early Exit:** Breaking when a condition is met.

---

## 2. `$.map()`

### Definitions

**Core Definition:** `$.map()` translates all items in an array or object to a new array.

**Technical Definition:** `jQuery.map(arrayOrObject, callback)` returns a new array with the results of the callback. The callback receives `(value, indexOrKey)` and can return a value, `null`, or `undefined`. `null` and `undefined` results are removed from the output array; all other values (including arrays) are flattened into the result. As of jQuery 1.6, the callback may return an array, which is flattened.

**Beginner-Friendly Explanation:** `$.map()` takes a list, transforms each item with your function, and returns a new list of the results. If your function returns nothing, that item is skipped.

### Purposes

- To transform arrays into new arrays.
- To extract specific properties from objects.
- To filter and transform in one step (by returning `null` to exclude).

### Syntax Rules and Structure

```javascript
var newArray = $.map(collection, function(value, index) {
    return transformedValue; // or null to exclude
});
```

**Syntax Rules:**
1. Returns a new array; does not modify the original.
2. `null`/`undefined` results are excluded.
3. Arrays returned from the callback are flattened.
4. Works on arrays and objects.

**Constraints and Limitations:**
- Flattening behavior can be surprising.
- Use native `Array.prototype.map()` for arrays in modern code.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Transforming an Array**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.map() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Double each number
        var numbers = [1, 2, 3, 4];
        var doubled = $.map(numbers, function(value, index) {
            return value * 2;
        });
        $("#output").append("Doubled: " + doubled.join(", ") + "<br>");

        // Step 2: Filter by returning null
        var evens = $.map(numbers, function(value) {
            return value % 2 === 0 ? value : null;
        });
        $("#output").append("Evens: " + evens.join(", ") + "<br>");

        // Step 3: Extract properties from objects
        var users = [{ name: "John" }, { name: "Jane" }];
        var names = $.map(users, function(user) {
            return user.name;
        });
        $("#output").append("Names: " + names.join(", "));
    </script>
</body>
</html>
```

**Expected Output:**
```
Doubled: 2, 4, 6, 8
Evens: 2, 4
Names: John, Jane
```

**Why This Code Produces That Result:** `$.map()` applies the callback to each item and collects non-null results. Returning `null` for odd numbers excludes them. Extracting `user.name` produces an array of names.

### Real-World Cases
- **Data Transformation:** Converting API responses to display formats.
- **Property Extraction:** Pulling specific fields from objects.
- **Filtering + Mapping:** Combining operations in one pass.

---

## 3. `$.grep()`

### Definitions

**Core Definition:** `$.grep()` finds the elements of an array that satisfy a filter function.

**Technical Definition:** `jQuery.grep(array, function, invert)` returns a new array containing elements for which the filter function returns `true`. The callback receives `(element, index)`. If `invert` is `true`, the function returns elements for which the filter returns `false`.

**Beginner-Friendly Explanation:** `$.grep()` filters an array, keeping only the items that pass your test.

### Purposes

- To filter arrays based on a condition.
- To invert a filter with the `invert` parameter.
- To create subsets of data.

### Syntax Rules and Structure

```javascript
var filtered = $.grep(array, function(element, index) {
    return condition; // true to keep
}, invert);
```

**Syntax Rules:**
1. Returns a new array; does not modify the original.
2. `invert` defaults to `false`.
3. Callback receives `(element, index)`.

**Constraints and Limitations:**
- Use native `Array.prototype.filter()` for arrays in modern code.
- Does not work on objects.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Filtering an Array**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.grep() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var numbers = [1, 2, 3, 4, 5, 6];

        // Step 1: Keep only even numbers
        var evens = $.grep(numbers, function(n) {
            return n % 2 === 0;
        });
        $("#output").append("Evens: " + evens.join(", ") + "<br>");

        // Step 2: Invert to get odd numbers
        var odds = $.grep(numbers, function(n) {
            return n % 2 === 0;
        }, true);
        $("#output").append("Odds: " + odds.join(", "));
    </script>
</body>
</html>
```

**Expected Output:**
```
Evens: 2, 4, 6
Odds: 1, 3, 5
```

**Why This Code Produces That Result:** The filter function returns `true` for even numbers. With `invert: false` (default), even numbers are kept. With `invert: true`, the result is inverted, keeping odd numbers.

### Real-World Cases
- **Data Filtering:** Selecting items matching criteria.
- **Search Results:** Filtering by keyword.
- **Inverted Filters:** Excluding items.

---

## 4. `$.inArray()`

### Definitions

**Core Definition:** `$.inArray()` searches for a value in an array and returns its index, or `-1` if not found.

**Technical Definition:** `jQuery.inArray(value, array, fromIndex)` is similar to `Array.prototype.indexOf()`. It returns the index of the first occurrence of `value`, or `-1` if not found. `fromIndex` specifies the starting index.

**Beginner-Friendly Explanation:** `$.inArray()` tells you where an item is in an array, or `-1` if it's not there.

### Purposes

- To check if a value exists in an array.
- To find the position of an item.
- To implement membership tests.

### Syntax Rules and Structure

```javascript
var index = $.inArray(value, array);
if (index !== -1) { /* found */ }
```

**Syntax Rules:**
1. Returns `-1` if not found.
2. Uses strict equality (`===`).
3. `fromIndex` optional.

**Constraints and Limitations:**
- Use native `Array.prototype.indexOf()` or `Array.prototype.includes()` in modern code.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Checking Array Membership**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.inArray() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var colors = ["red", "green", "blue"];

        // Step 1: Check if a value exists
        var index = $.inArray("green", colors);
        $("#output").append("Green at index: " + index + "<br>"); // 1

        // Step 2: Check for a missing value
        var missing = $.inArray("yellow", colors);
        $("#output").append("Yellow at index: " + missing + "<br>"); // -1

        // Step 3: Conditional logic
        if ($.inArray("blue", colors) !== -1) {
            $("#output").append("Blue is in the array");
        }
    </script>
</body>
</html>
```

**Expected Output:**
```
Green at index: 1
Yellow at index: -1
Blue is in the array
```

**Why This Code Produces That Result:** `$.inArray()` returns the index of the found value or `-1`. Comparing to `-1` determines membership.

### Real-World Cases
- **Validation:** Checking if a value is in an allowed list.
- **Deduplication:** Checking if an item already exists.
- **Navigation:** Finding the position of the current page.

---

## 5. `$.isArray()`

### Definitions

**Core Definition:** `$.isArray()` determines whether the argument is a JavaScript array. **Deprecated in jQuery 3.3** — use `Array.isArray()`.

**Technical Definition:** `jQuery.isArray(obj)` returns `true` if `obj` is an array. It handles cross-frame arrays correctly.

**Beginner-Friendly Explanation:** `$.isArray()` checks if something is an array. In modern code, use `Array.isArray()`.

### Purposes
- To check if a value is an array before iterating.
- To distinguish arrays from objects.

### Syntax Rules and Structure

```javascript
$.isArray([]);        // true
$.isArray({});        // false
$.isArray("string");  // false
```

**Syntax Rules:**
1. Returns Boolean.
2. Deprecated; use `Array.isArray()`.

**Constraints and Limitations:**
- Deprecated since jQuery 3.3.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Array Detection**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isArray() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var values = [[], {}, "text", 42, null];
        $.each(values, function(i, val) {
            $("#output").append(
                JSON.stringify(val) + " isArray: " + $.isArray(val) + "<br>"
            );
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
[] isArray: true
{} isArray: false
"text" isArray: false
42 isArray: false
null isArray: false
```

**Why This Code Produces That Result:** Only the empty array returns `true`; all other values return `false`.

### Real-World Cases
- **Type Checking:** Validating API responses.
- **Branching Logic:** Handling arrays differently from objects.

---

## 6. `$.isFunction()`

### Definitions

**Core Definition:** `$.isFunction()` determines whether the argument is a JavaScript function. **Deprecated in jQuery 3.3** — use `typeof value === "function"`.

**Technical Definition:** `jQuery.isFunction(obj)` returns `true` if `obj` is a function.

**Beginner-Friendly Explanation:** `$.isFunction()` checks if something is a function.

### Purposes
- To check if a value is callable before invoking.
- To validate callback arguments.

### Syntax Rules and Structure

```javascript
$.isFunction(function() {}); // true
$.isFunction("text");        // false
```

**Syntax Rules:**
1. Returns Boolean.
2. Deprecated; use `typeof value === "function"`.

**Constraints and Limitations:**
- Deprecated since jQuery 3.3.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Function Detection**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isFunction() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var fn = function() {};
        var notFn = "hello";
        $("#output").append("fn isFunction: " + $.isFunction(fn) + "<br>");
        $("#output").append("notFn isFunction: " + $.isFunction(notFn));
    </script>
</body>
</html>
```

**Expected Output:**
```
fn isFunction: true
notFn isFunction: false
```

**Why This Code Produces That Result:** Only the actual function returns `true`.

### Real-World Cases
- **Plugin Options:** Checking if a callback option is a function.
- **Safe Invocation:** Verifying callability before calling.

---

## 7. `$.isNumeric()`

### Definitions

**Core Definition:** `$.isNumeric()` determines whether its argument represents a numeric value. **Deprecated in jQuery 3.3** — use `typeof value === "number"` or `Number.isFinite()`.

**Technical Definition:** `jQuery.isNumeric(value)` returns `true` if `value` is a number or a string that can be converted to a finite number. It returns `false` for `NaN`, `Infinity`, and non-numeric strings.

**Beginner-Friendly Explanation:** `$.isNumeric()` checks if something looks like a number, whether it's an actual number or a numeric string.

### Purposes
- To validate numeric input.
- To distinguish numeric strings from non-numeric values.

### Syntax Rules and Structure

```javascript
$.isNumeric(42);        // true
$.isNumeric("42");      // true
$.isNumeric("4.2");     // true
$.isNumeric("abc");     // false
$.isNumeric(Infinity);  // false
```

**Syntax Rules:**
1. Returns Boolean.
2. Accepts numbers and numeric strings.
3. Deprecated since jQuery 3.3.

**Constraints and Limitations:**
- Returns `false` for `Infinity` and `NaN`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Numeric Validation**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isNumeric() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var values = [42, "42", "4.2", "abc", Infinity, NaN, ""];
        $.each(values, function(i, val) {
            $("#output").append(
                String(val) + " isNumeric: " + $.isNumeric(val) + "<br>"
            );
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
42 isNumeric: true
42 isNumeric: true
4.2 isNumeric: true
abc isNumeric: false
Infinity isNumeric: false
NaN isNumeric: false
 isNumeric: false
```

**Why This Code Produces That Result:** `$.isNumeric()` returns `true` for numbers and numeric strings, `false` for non-numeric strings, `Infinity`, `NaN`, and empty strings.

### Real-World Cases
- **Form Validation:** Checking numeric input fields.
- **Data Parsing:** Validating numeric values from APIs.

---

## 8. `$.isPlainObject()`

### Definitions

**Core Definition:** `$.isPlainObject()` determines whether the argument is a plain object (created via `{}` or `new Object()`).

**Technical Definition:** `jQuery.isPlainObject(obj)` returns `true` if `obj` is an object created by the `Object` constructor or one with a `[[Prototype]]` of `null`. It returns `false` for DOM elements, arrays, functions, and objects from other realms.

**Beginner-Friendly Explanation:** `$.isPlainObject()` checks if something is a simple, ordinary object — not an array, DOM element, or class instance.

### Purposes
- To validate object arguments in plugins.
- To distinguish plain objects from arrays and DOM elements.
- To safely merge options.

### Syntax Rules and Structure

```javascript
$.isPlainObject({});            // true
$.isPlainObject(new Object());  // true
$.isPlainObject([]);            // false
$.isPlainObject(document);      // false
```

**Syntax Rules:**
1. Returns Boolean.
2. `true` for `{}` and `new Object()`.
3. `false` for arrays, DOM elements, functions, and class instances.

**Constraints and Limitations:**
- Does not detect objects from other JavaScript realms.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Plain Object Detection**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isPlainObject() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        $("#output").append("{}: " + $.isPlainObject({}) + "<br>");
        $("#output").append("new Object(): " + $.isPlainObject(new Object()) + "<br>");
        $("#output").append("[]: " + $.isPlainObject([]) + "<br>");
        $("#output").append("function: " + $.isPlainObject(function() {}) + "<br>");
        $("#output").append("document: " + $.isPlainObject(document));
    </script>
</body>
</html>
```

**Expected Output:**
```
{}: true
new Object(): true
[]: false
function: false
document: false
```

**Why This Code Produces That Result:** Only plain objects return `true`; arrays, functions, and DOM elements return `false`.

### Real-World Cases
- **Plugin Options:** Validating options objects.
- **Deep Merge:** Ensuring safe merging with `$.extend(true, ...)`.

---

## 9. `$.isEmptyObject()`

### Definitions

**Core Definition:** `$.isEmptyObject()` checks whether an object has any enumerable properties.

**Technical Definition:** `jQuery.isEmptyObject(obj)` returns `true` if `obj` has no enumerable own properties. It works on plain objects and arrays (an empty array returns `true`).

**Beginner-Friendly Explanation:** `$.isEmptyObject()` tells you if an object has nothing in it.

### Purposes
- To check if an object has any data.
- To validate API responses.
- To skip processing when no data is present.

### Syntax Rules and Structure

```javascript
$.isEmptyObject({});           // true
$.isEmptyObject({ a: 1 });     // false
$.isEmptyObject([]);           // true
$.isEmptyObject([1, 2]);       // false
```

**Syntax Rules:**
1. Returns Boolean.
2. Only checks enumerable own properties.

**Constraints and Limitations:**
- Inherited properties are not counted.
- DOM elements return `false`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Emptiness Check**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isEmptyObject() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        $("#output").append("{ }: " + $.isEmptyObject({}) + "<br>");
        $("#output").append("{ a: 1 }: " + $.isEmptyObject({ a: 1 }) + "<br>");
        $("#output").append("[ ]: " + $.isEmptyObject([]) + "<br>");
        $("#output").append("[1, 2]: " + $.isEmptyObject([1, 2]));
    </script>
</body>
</html>
```

**Expected Output:**
```
{ }: true
{ a: 1 }: false
[ ]: true
[1, 2]: false
```

**Why This Code Produces That Result:** Empty objects and arrays return `true`; non-empty ones return `false`.

### Real-World Cases
- **API Response Validation:** Checking if data was returned.
- **Form Data:** Checking if any fields have values.

---

## 10. `$.isWindow()`

### Definitions

**Core Definition:** `$.isWindow()` determines whether the argument is a `window` object. **Deprecated in jQuery 3.3**.

**Technical Definition:** `jQuery.isWindow(obj)` returns `true` if `obj` is a window object.

**Beginner-Friendly Explanation:** `$.isWindow()` checks if something is a browser window.

### Purposes
- To detect window objects in cross-frame code.
- To safely access window properties.

### Syntax Rules and Structure

```javascript
$.isWindow(window);  // true
$.isWindow(document); // false
$.isWindow({});      // false
```

**Syntax Rules:**
1. Returns Boolean.
2. Deprecated since jQuery 3.3.

**Constraints and Limitations:**
- Deprecated; use `obj != null && obj === obj.window`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Window Detection**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isWindow() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        $("#output").append("window: " + $.isWindow(window) + "<br>");
        $("#output").append("document: " + $.isWindow(document) + "<br>");
        $("#output").append("{}: " + $.isWindow({}));
    </script>
</body>
</html>
```

**Expected Output:**
```
window: true
document: false
{}: false
```

**Why This Code Produces That Result:** Only the actual `window` object returns `true`.

### Real-World Cases
- **Cross-Frame Scripts:** Detecting window objects from other frames.
- **Event Handling:** Distinguishing window from document targets.

---

## 11. `$.isXMLDoc()`

### Definitions

**Core Definition:** `$.isXMLDoc()` determines whether a DOM node is within an XML document.

**Technical Definition:** `jQuery.isXMLDoc(node)` returns `true` if the node is in an XML document (including SVG). It checks the `nodeType` and `documentElement` properties.

**Beginner-Friendly Explanation:** `$.isXMLDoc()` checks if a DOM node belongs to an XML document rather than an HTML document.

### Purposes
- To differentiate XML from HTML documents.
- To apply XML-specific handling.

### Syntax Rules and Structure

```javascript
$.isXMLDoc(document);          // false
$.isXMLDoc(document.createElement("div")); // false
$.isXMLDoc($.parseXML("<root/>"));  // true
```

**Syntax Rules:**
1. Returns Boolean.
2. `true` for XML and SVG documents.

**Constraints and Limitations:**
- Only works on DOM nodes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: XML Document Detection**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.isXMLDoc() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var xmlDoc = $.parseXML("<root><item>Test</item></root>");
        $("#output").append("XML doc: " + $.isXMLDoc(xmlDoc) + "<br>");
        $("#output").append("HTML doc: " + $.isXMLDoc(document));
    </script>
</body>
</html>
```

**Expected Output:**
```
XML doc: true
HTML doc: false
```

**Why This Code Produces That Result:** The parsed XML document returns `true`; the HTML `document` returns `false`.

### Real-World Cases
- **SVG Handling:** Detecting SVG documents.
- **XML Parsing:** Conditional logic for XML vs. HTML.

---

## 12. `$.type()`

### Definitions

**Core Definition:** `$.type()` determines the internal JavaScript `[[Class]]` of an object. **Deprecated in jQuery 3.3**.

**Technical Definition:** `jQuery.type(obj)` returns a string: `"undefined"`, `"null"`, `"boolean"`, `"number"`, `"string"`, `"function"`, `"array"`, `"date"`, `"regexp"`, `"object"`, or `"error"`.

**Beginner-Friendly Explanation:** `$.type()` tells you exactly what type of data something is — more precisely than `typeof`.

### Purposes
- To distinguish arrays from objects, dates from strings, etc.
- To perform type-specific branching.

### Syntax Rules and Structure

```javascript
$.type(undefined);  // "undefined"
$.type(null);       // "null"
$.type([]);         // "array"
$.type(new Date()); // "date"
$.type({});         // "object"
```

**Syntax Rules:**
1. Returns a lowercase string.
2. More precise than `typeof`.
3. Deprecated since jQuery 3.3.

**Constraints and Limitations:**
- Deprecated; use `Object.prototype.toString.call()` or native methods.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Type Detection**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.type() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var values = [undefined, null, true, 42, "text", function() {}, [], {}, new Date()];
        $.each(values, function(i, val) {
            $("#output").append(String(val) + " → " + $.type(val) + "<br>");
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
undefined → undefined
null → null
true → boolean
42 → number
text → string
function() {} → function
 → array
[object Object] → object
[object Date] → date
```

**Why This Code Produces That Result:** `$.type()` returns precise type strings, distinguishing arrays from objects and dates from objects.

### Real-World Cases
- **Type-Safe Branching:** Handling different data types.
- **Debugging:** Inspecting values.

---

## 13. `$.trim()`

### Definitions

**Core Definition:** `$.trim()` removes whitespace from the beginning and end of a string. **Deprecated in jQuery 3.5** — use `String.prototype.trim()`.

**Technical Definition:** `jQuery.trim(str)` returns the string with leading and trailing whitespace removed.

**Beginner-Friendly Explanation:** `$.trim()` cleans up extra spaces at the start and end of a string.

### Purposes
- To clean user input.
- To normalize strings before comparison.

### Syntax Rules and Structure

```javascript
$.trim("  hello  ");  // "hello"
```

**Syntax Rules:**
1. Returns a string.
2. Deprecated; use `str.trim()`.

**Constraints and Limitations:**
- Deprecated since jQuery 3.5.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Trimming Strings**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.trim() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var input = "   hello world   ";
        $("#output").append("Original: [" + input + "]<br>");
        $("#output").append("Trimmed: [" + $.trim(input) + "]");
    </script>
</body>
</html>
```

**Expected Output:**
```
Original: [   hello world   ]
Trimmed: [hello world]
```

**Why This Code Produces That Result:** `$.trim()` removes leading and trailing whitespace, preserving internal spaces.

### Real-World Cases
- **Form Validation:** Cleaning input before validation.
- **String Comparison:** Normalizing for equality checks.

---

## 14. `$.parseHTML()`

### Definitions

**Core Definition:** `$.parseHTML()` parses a string into an array of DOM nodes.

**Technical Definition:** `jQuery.parseHTML(data, context, keepScripts)` uses the native `DOMParser` or `document.implementation.createHTMLDocument()` to parse HTML. It returns an array of DOM nodes. By default, scripts are not included; set `keepScripts: true` to include them.

**Beginner-Friendly Explanation:** `$.parseHTML()` turns an HTML string into real DOM elements you can work with.

### Purposes
- To safely convert HTML strings to DOM nodes.
- To avoid XSS by not executing scripts by default.
- To process HTML before insertion.

### Syntax Rules and Structure

```javascript
var nodes = $.parseHTML("<div>Hello</div>");
$(nodes).appendTo("body");
```

**Syntax Rules:**
1. Returns an array of DOM nodes.
2. Scripts are excluded by default.
3. `context` sets the document context.

**Constraints and Limitations:**
- Does not sanitize HTML; untrusted input must still be sanitized.
- `keepScripts: true` is a security risk.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Parsing HTML Safely**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.parseHTML() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Parse HTML string into DOM nodes
        var nodes = $.parseHTML("<p>Hello</p><p>World</p>");
        console.log("Nodes:", nodes.length); // 2

        // Step 2: Append to the DOM
        $(nodes).appendTo("#output");

        // Step 3: Scripts are excluded by default
        var withScript = $.parseHTML("<div>Safe</div><script>alert('XSS')<\/script>");
        console.log("With script (excluded):", withScript.length); // 1
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Hello" and "World" paragraphs. Console logs 2 nodes for the first parse and 1 node for the second (script excluded).

**Why This Code Produces That Result:** `$.parseHTML()` parses the string into DOM nodes. By default, `<script>` tags are stripped, preventing XSS.

### Real-World Cases
- **Template Rendering:** Converting HTML strings to DOM.
- **Safe HTML Insertion:** Avoiding XSS by excluding scripts.

---

## 15. `$.parseJSON()`

### Definitions

**Core Definition:** `$.parseJSON()` parses a well-formed JSON string into a JavaScript object. **Deprecated in jQuery 3.0** — use `JSON.parse()`.

**Technical Definition:** `jQuery.parseJSON(json)` uses `JSON.parse()` internally. It throws an error for invalid JSON.

**Beginner-Friendly Explanation:** `$.parseJSON()` turns a JSON string into a JavaScript object.

### Purposes
- To parse JSON strings from older browsers (pre-IE8).
- To maintain backward compatibility.

### Syntax Rules and Structure

```javascript
var obj = $.parseJSON('{"name":"John"}');
```

**Syntax Rules:**
1. Deprecated since jQuery 3.0.
2. Use `JSON.parse()` in modern code.

**Constraints and Limitations:**
- Deprecated; no advantage over `JSON.parse()`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Parsing JSON**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.parseJSON() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var jsonString = '{"name":"John","age":30}';
        var obj = $.parseJSON(jsonString);
        $("#output").text(obj.name + " is " + obj.age);
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "John is 30".

**Why This Code Produces That Result:** `$.parseJSON()` converts the JSON string to an object, allowing property access.

### Real-World Cases
- **Legacy Code:** Maintaining older jQuery codebases.
- **Migration:** Replacing with `JSON.parse()`.

---

## 16. `$.parseXML()`

### Definitions

**Core Definition:** `$.parseXML()` parses a string into an XML document.

**Technical Definition:** `jQuery.parseXML(data)` uses the native `DOMParser` (or `ActiveXObject` in legacy IE) to parse XML. Returns an `XMLDocument`.

**Beginner-Friendly Explanation:** `$.parseXML()` turns an XML string into a document you can traverse.

### Purposes
- To parse XML responses from AJAX.
- To create XML documents for manipulation.

### Syntax Rules and Structure

```javascript
var xmlDoc = $.parseXML("<root><item>Test</item></root>");
$(xmlDoc).find("item").text(); // "Test"
```

**Syntax Rules:**
1. Returns an `XMLDocument`.
2. Throws on invalid XML.

**Constraints and Limitations:**
- XML must be well-formed.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Parsing XML**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.parseXML() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var xmlString = "<catalog><book id='1'><title>JS</title></book></catalog>";
        var xmlDoc = $.parseXML(xmlString);
        var $title = $(xmlDoc).find("title");
        $("#output").text("Title: " + $title.text());
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Title: JS".

**Why This Code Produces That Result:** `$.parseXML()` creates an XML document, and jQuery traversal methods extract the `<title>` text.

### Real-World Cases
- **RSS Feeds:** Parsing XML feeds.
- **SOAP APIs:** Working with XML responses.

---

## 17. `$.noop()`

### Definitions

**Core Definition:** `$.noop()` is an empty function that does nothing.

**Technical Definition:** `jQuery.noop()` returns `undefined` and performs no action. It is useful as a default callback.

**Beginner-Friendly Explanation:** `$.noop()` is a function that does nothing — useful when you need a placeholder.

### Purposes
- To provide a default no-op callback.
- To avoid `undefined` errors when a callback is expected.

### Syntax Rules and Structure

```javascript
$.noop();  // does nothing
```

**Syntax Rules:**
1. Takes no arguments.
2. Returns `undefined`.

**Constraints and Limitations:**
- No functional behavior.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using noop as Default Callback**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.noop() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        function doSomething(callback) {
            callback = callback || $.noop;
            callback();
            $("#output").text("Function executed with noop fallback");
        }
        doSomething(); // No error
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Function executed with noop fallback".

**Why This Code Produces That Result:** `$.noop` provides a safe default when no callback is passed.

### Real-World Cases
- **Plugin Defaults:** Default callback options.
- **Optional Callbacks:** Safely invoking undefined callbacks.

---

## 18. `$.now()`

### Definitions

**Core Definition:** `$.now()` returns the current timestamp in milliseconds. **Deprecated in jQuery 3.3** — use `Date.now()`.

**Technical Definition:** `jQuery.now()` returns `Date.now()` or `new Date().getTime()` in older browsers.

**Beginner-Friendly Explanation:** `$.now()` tells you the current time in milliseconds since 1970.

### Purposes
- To measure elapsed time.
- To generate unique timestamps.

### Syntax Rules and Structure

```javascript
var start = $.now();
// ... work ...
var elapsed = $.now() - start;
```

**Syntax Rules:**
1. Returns a number.
2. Deprecated; use `Date.now()`.

**Constraints and Limitations:**
- Deprecated since jQuery 3.3.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Measuring Execution Time**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.now() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var start = $.now();
        for (var i = 0; i < 100000; i++) { /* work */ }
        var elapsed = $.now() - start;
        $("#output").text("Elapsed: " + elapsed + "ms");
    </script>
</body>
</html>
```

**Expected Output:** The output div displays the elapsed time in milliseconds.

**Why This Code Produces That Result:** `$.now()` captures timestamps, and the difference gives the elapsed time.

### Real-World Cases
- **Performance Timing:** Measuring function execution.
- **Debouncing:** Tracking last execution time.

---

## 19. `$.proxy()`

### Definitions

**Core Definition:** `$.proxy()` takes a function and returns a new one with a specific `this` context. **Deprecated in jQuery 3.3** — use `Function.prototype.bind()`.

**Technical Definition:** `jQuery.proxy(fn, context)` returns a new function that, when called, invokes `fn` with `this` set to `context` and the original arguments.

**Beginner-Friendly Explanation:** `$.proxy()` locks a function's `this` value so it always refers to a specific object.

### Purposes
- To preserve `this` in callbacks.
- To bind event handlers to specific contexts.

### Syntax Rules and Structure

```javascript
var obj = { name: "John" };
var fn = function() { console.log(this.name); };
var bound = $.proxy(fn, obj);
bound(); // "John"
```

**Syntax Rules:**
1. Returns a new function.
2. Deprecated; use `fn.bind(context)`.

**Constraints and Limitations:**
- Deprecated since jQuery 3.3.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Binding Context**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.proxy() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var counter = {
            count: 0,
            increment: function() {
                this.count++;
                $("#output").text("Count: " + this.count);
            }
        };

        // Without proxy, `this` would be the button
        $("#output").on("click", $.proxy(counter.increment, counter));
    </script>
</body>
</html>
```

**Expected Output:** Each click on the output div increments and displays the count.

**Why This Code Produces That Result:** `$.proxy()` binds `this` to the `counter` object, so `this.count` refers to the correct property.

### Real-World Cases
- **Event Handlers:** Binding object methods to events.
- **Callbacks:** Preserving context in async callbacks.

---

## 20. `$.extend()`

### Definitions

**Core Definition:** `$.extend()` merges the contents of two or more objects into the first object.

**Technical Definition:** `jQuery.extend([deep], target, object1, ...)` merges properties from source objects into the target. If `deep` is `true`, the merge is recursive. If only one argument is provided, it extends the jQuery namespace. Returns the modified target.

**Beginner-Friendly Explanation:** `$.extend()` combines objects, copying properties from one to another.

### Purposes
- To merge default options with user options.
- To create shallow or deep copies of objects.
- To extend the jQuery namespace.

### Syntax Rules and Structure

```javascript
// Shallow merge
var merged = $.extend({}, defaults, options);

// Deep merge
var deepMerged = $.extend(true, {}, defaults, options);

// Extend jQuery namespace
$.extend({ myUtility: function() {} });
```

**Syntax Rules:**
1. Target is modified and returned.
2. Use `{}` as target for a new object.
3. `deep: true` enables recursive merge.

**Constraints and Limitations:**
- Deep merge can cause infinite loops with circular references.
- Prototype pollution risk with untrusted input.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Merging Options**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.extend() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var defaults = { color: "red", size: "medium", border: { width: 1 } };
        var options = { color: "blue", border: { style: "solid" } };

        // Step 1: Shallow merge
        var shallow = $.extend({}, defaults, options);
        $("#output").append("Shallow: " + JSON.stringify(shallow) + "<br>");

        // Step 2: Deep merge
        var deep = $.extend(true, {}, defaults, options);
        $("#output").append("Deep: " + JSON.stringify(deep));
    </script>
</body>
</html>
```

**Expected Output:**
```
Shallow: {"color":"blue","size":"medium","border":{"style":"solid"}}
Deep: {"color":"blue","size":"medium","border":{"width":1,"style":"solid"}}
```

**Why This Code Produces That Result:** Shallow merge replaces the `border` object entirely; deep merge recursively combines `border.width` and `border.style`.

### Real-World Cases
- **Plugin Options:** Merging user options with defaults.
- **Configuration:** Combining config objects.
- **Namespace Extension:** Adding custom utilities.

---

## 21. `$.merge()`

### Definitions

**Core Definition:** `$.merge()` merges the contents of two arrays into the first array.

**Technical Definition:** `jQuery.merge(first, second)` adds all elements of `second` to `first`, modifying `first` in place and returning it.

**Beginner-Friendly Explanation:** `$.merge()` combines two arrays into one.

### Purposes
- To concatenate arrays in place.
- To combine data from multiple sources.

### Syntax Rules and Structure

```javascript
var first = [1, 2];
var second = [3, 4];
$.merge(first, second); // first is now [1, 2, 3, 4]
```

**Syntax Rules:**
1. Modifies and returns the first array.
2. Does not create a new array.

**Constraints and Limitations:**
- Modifies the first array.
- Use `Array.prototype.concat()` for a new array.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Merging Arrays**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.merge() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var a = [1, 2, 3];
        var b = [4, 5, 6];
        $.merge(a, b);
        $("#output").text("Merged: " + a.join(", "));
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Merged: 1, 2, 3, 4, 5, 6".

**Why This Code Produces That Result:** `$.merge()` appends elements of `b` to `a`, modifying `a` in place.

### Real-World Cases
- **Data Combination:** Merging API results.
- **In-Place Concatenation:** When mutation is acceptable.

---

## 22. `$.uniqueSort()`

### Definitions

**Core Definition:** `$.uniqueSort()` sorts an array of DOM elements and removes duplicates. (Formerly `$.unique()`.)

**Technical Definition:** `jQuery.uniqueSort(array)` sorts an array of DOM elements in document order and removes duplicates. It only works on arrays of DOM elements, not strings or numbers.

**Beginner-Friendly Explanation:** `$.uniqueSort()` cleans up a list of DOM elements — sorting them and removing duplicates.

### Purposes
- To deduplicate DOM element arrays.
- To sort elements in document order.

### Syntax Rules and Structure

```javascript
var elements = [div2, div1, div2];
var unique = $.uniqueSort(elements); // [div1, div2]
```

**Syntax Rules:**
1. Only works on DOM elements.
2. Sorts and removes duplicates.
3. `$.unique()` is deprecated (renamed to `$.uniqueSort()` in jQuery 3.0).

**Constraints and Limitations:**
- Does not work on strings or numbers.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Deduplicating DOM Elements**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.uniqueSort() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="item" id="a">A</div>
    <div class="item" id="b">B</div>
    <div id="output"></div>
    <script>
        var items = document.querySelectorAll(".item");
        var duplicates = [items[0], items[1], items[0]];
        var unique = $.uniqueSort(duplicates);
        console.log("Unique count:", unique.length); // 2
        $(unique).appendTo("#output");
    </script>
</body>
</html>
```

**Expected Output:** The output div displays the two unique items (A and B) without duplication.

**Why This Code Produces That Result:** `$.uniqueSort()` removes the duplicate `items[0]` and sorts the elements.

### Real-World Cases
- **DOM Deduplication:** Combining selections from multiple queries.
- **Sorting Elements:** Ensuring document order.

---

## 23. `$.contains()`

### Definitions

**Core Definition:** `$.contains()` checks whether a DOM node is a descendant of another DOM node.

**Technical Definition:** `jQuery.contains(container, contained)` returns `true` if `contained` is a descendant of `container`, not including the container itself.

**Beginner-Friendly Explanation:** `$.contains()` tells you if one element is inside another.

### Purposes
- To check ancestry in the DOM.
- To validate event targets.

### Syntax Rules and Structure

```javascript
$.contains(document.body, document.getElementById("child")); // true
```

**Syntax Rules:**
1. Returns Boolean.
2. Does not include the container itself.

**Constraints and Limitations:**
- Both arguments must be DOM nodes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Checking Ancestry**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.contains() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="parent"><div id="child">Child</div></div>
    <div id="output"></div>
    <script>
        var parent = document.getElementById("parent");
        var child = document.getElementById("child");
        $("#output").append("Parent contains child: " + $.contains(parent, child) + "<br>");
        $("#output").append("Child contains parent: " + $.contains(child, parent));
    </script>
</body>
</html>
```

**Expected Output:**
```
Parent contains child: true
Child contains parent: false
```

**Why This Code Produces That Result:** `$.contains()` checks the DOM tree relationship; the child is inside the parent, but not vice versa.

### Real-World Cases
- **Event Delegation:** Checking if an event target is within a container.
- **DOM Traversal:** Validating relationships.

---

## 24. `$.escapeSelector()`

### Definitions

**Core Definition:** `$.escapeSelector()` escapes CSS special characters in a string so it can be safely used in a selector.

**Technical Definition:** `jQuery.escapeSelector(selector)` escapes characters that have special meaning in CSS selectors (`.`, `#`, `:`, `[`, `]`, etc.) by prefixing them with backslashes. This is the jQuery equivalent of the CSS.escape() method.

**Beginner-Friendly Explanation:** `$.escapeSelector()` makes weird characters in IDs or class names safe to use in jQuery selectors.

### Purposes
- To safely select elements with special characters in their IDs.
- To prevent selector syntax errors.

### Syntax Rules and Structure

```javascript
var escaped = $.escapeSelector("some.id");
$("#" + escaped);  // selects id="some.id"
```

**Syntax Rules:**
1. Returns an escaped string.
2. Does not include the `#` prefix.

**Constraints and Limitations:**
- Only escapes for CSS selector use.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Escaping Special Characters**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.escapeSelector() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="some.id">Dot ID</div>
    <div id="output"></div>
    <script>
        var escaped = $.escapeSelector("some.id");
        console.log("Escaped:", escaped); // "some\.id"
        var $el = $("#" + escaped);
        $("#output").text("Found: " + $el.text());
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Found: Dot ID".

**Why This Code Produces That Result:** `$.escapeSelector()` escapes the dot, making the selector valid and able to find the element.

### Real-World Cases
- **Server-Generated IDs:** IDs with dots, colons, or brackets.
- **CMS Content:** Dynamic IDs from content slugs.

---

## 25. `$.globalEval()`

### Definitions

**Core Definition:** `$.globalEval()` executes JavaScript code in the global context. **Deprecated** — use `window.eval()` or `Function()`.

**Technical Definition:** `jQuery.globalEval(code)` evaluates the code in the global scope, even when called from within a function.

**Beginner-Friendly Explanation:** `$.globalEval()` runs JavaScript code as if it were written at the top level of your page.

### Purposes
- To execute code in the global scope.
- To run scripts from AJAX responses.

### Syntax Rules and Structure

```javascript
$.globalEval("var x = 10;");
```

**Syntax Rules:**
1. Executes in global scope.
2. Security risk with untrusted code.

**Constraints and Limitations:**
- Deprecated; avoid due to XSS risk.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Global Evaluation**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.globalEval() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        $.globalEval("var globalVar = 'Hello from globalEval';");
        $("#output").text(window.globalVar);
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Hello from globalEval".

**Why This Code Produces That Result:** `$.globalEval()` defines `globalVar` in the global scope, accessible via `window.globalVar`.

### Real-World Cases
- **Legacy Script Execution:** Running scripts from old AJAX responses.
- **Security Warning:** Avoid with untrusted code.

---

## 26. `$.holdReady()`

### Definitions

**Core Definition:** `$.holdReady()` holds or releases the execution of jQuery's `ready` event.

**Technical Definition:** `jQuery.holdReady(hold)` delays the firing of `$(document).ready()` handlers. Call `$.holdReady(true)` to hold, and `$.holdReady(false)` to release. Useful for loading scripts or data before ready handlers run.

**Beginner-Friendly Explanation:** `$.holdReady()` pauses the "document is ready" event so you can load something first.

### Purposes
- To delay ready handlers until async resources load.
- To ensure dependencies are available before initialization.

### Syntax Rules and Structure

```javascript
$.holdReady(true);
$.getScript("plugin.js", function() {
    $.holdReady(false);
});
```

**Syntax Rules:**
1. Multiple holds stack; each `true` must be matched by a `false`.
2. Deprecated in jQuery 3.0? (Still available but less common.)

**Constraints and Limitations:**
- Can delay page initialization if not released.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Holding Ready for a Script**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.holdReady() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script>
        $.holdReady(true);
        // Simulate loading a script
        setTimeout(function() {
            console.log("Dependency loaded");
            $.holdReady(false);
        }, 1000);
    </script>
</head>
<body>
    <div id="output"></div>
    <script>
        $(document).ready(function() {
            $("#output").text("Ready fired after hold released");
            console.log("Ready handler fired");
        });
    </script>
</body>
</html>
```

**Expected Output:** After 1 second, the output div displays "Ready fired after hold released". Console logs "Dependency loaded" then "Ready handler fired".

**Why This Code Produces That Result:** `$.holdReady(true)` delays the ready event. After 1 second, `$.holdReady(false)` releases it, and the ready handler fires.

### Real-World Cases
- **Plugin Loading:** Loading dependencies before initialization.
- **Data Prefetching:** Fetching config before ready handlers.

---

## 27. `$.ready`

### Definitions

**Core Definition:** `$.ready` (or `$(document).ready()`) is a promise-like object that resolves when the DOM is fully loaded and ready for manipulation.

**Technical Definition:** `jQuery.ready` is a Promise that resolves when the document is ready. `$(handler)` and `$(document).ready(handler)` are equivalent ways to register ready callbacks. As of jQuery 3.0, `$.ready` is a Promise that can be used with `.then()`.

**Beginner-Friendly Explanation:** `$.ready` tells you when the page's HTML has finished loading and it's safe to manipulate the DOM.

### Purposes
- To run code after the DOM is ready.
- To avoid errors from manipulating elements that don't exist yet.

### Syntax Rules and Structure

```javascript
$(function() {
    // DOM is ready
});

// Or
$(document).ready(function() { /* ... */ });

// Or with Promise
$.ready.then(function() { /* ... */ });
```

**Syntax Rules:**
1. `$(function() {})` is shorthand for `$(document).ready()`.
2. `$.ready` is a Promise in jQuery 3.0+.
3. Ready handlers fire in registration order.

**Constraints and Limitations:**
- Does not wait for images or stylesheets to load (use `$(window).on("load")` for that).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using Ready**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.ready Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Register a ready handler
        $(function() {
            $("#output").append("Ready handler 1<br>");
        });

        // Step 2: Register another ready handler
        $(document).ready(function() {
            $("#output").append("Ready handler 2<br>");
        });

        // Step 3: Use $.ready as a Promise
        $.ready.then(function() {
            $("#output").append("Ready via Promise<br>");
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
Ready handler 1
Ready handler 2
Ready via Promise
```

**Why This Code Produces That Result:** All three handlers fire when the DOM is ready, in registration order.

### Real-World Cases
- **Page Initialization:** Running setup code after DOM load.
- **Plugin Initialization:** Ensuring elements exist before binding.

---

## 28. `$.when()`

### Definitions

**Core Definition:** `$.when()` provides a way to execute callback functions based on one or more Deferred objects or Promises.

**Technical Definition:** `jQuery.when(deferreds...)` returns a Promise that resolves when all input Deferreds resolve, or rejects when any rejects. Resolved values are passed as arguments in order. Each AJAX argument is `[data, textStatus, jqXHR]`.

**Beginner-Friendly Explanation:** `$.when()` waits for multiple things to finish before running your code.

### Purposes
- To coordinate multiple asynchronous operations.
- To wait for all AJAX requests to complete.
- To aggregate results from multiple sources.

### Syntax Rules and Structure

```javascript
$.when(promise1, promise2)
    .done(function(result1, result2) { /* ... */ })
    .fail(function(reason) { /* ... */ });
```

**Syntax Rules:**
1. Accepts any number of Promises.
2. Resolved values passed as separate arguments.
3. Rejects if any input rejects.

**Constraints and Limitations:**
- Rejects as soon as one fails; does not wait for others.
- Use `.apply()` for dynamic arrays.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Waiting for Multiple Requests**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.when() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        $.when(
            $.get("https://jsonplaceholder.typicode.com/posts/1"),
            $.get("https://jsonplaceholder.typicode.com/posts/2")
        ).done(function(post1, post2) {
            $("#output").append(
                "Post 1: " + post1[0].title + "<br>" +
                "Post 2: " + post2[0].title
            );
        }).fail(function() {
            $("#output").text("One request failed");
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
Post 1: sunt aut facere...
Post 2: qui est esse
```

**Why This Code Produces That Result:** Both requests run in parallel. When both succeed, `.done()` fires with both results.

### Real-World Cases
- **Dashboard Loading:** Waiting for multiple data sources.
- **Parallel Requests:** Fetching related resources simultaneously.

---

## 29. `$.Deferred()`

### Definitions

**Core Definition:** `$.Deferred()` creates a Deferred object, which is a chainable utility object that can register callbacks, invoke callbacks, and relay state.

**Technical Definition:** `jQuery.Deferred()` returns a Deferred object with methods for registering callbacks (`.done()`, `.fail()`, `.always()`, `.then()`), changing state (`.resolve()`, `.reject()`, `.notify()`), and querying state (`.state()`, `.promise()`). The `.promise()` method returns a restricted Promise view.

**Beginner-Friendly Explanation:** `$.Deferred()` creates a "promise" you can control — you decide when it succeeds or fails, and others can listen.

### Purposes
- To create custom asynchronous operations.
- To manage complex async workflows.
- To provide Promise interfaces for non-AJAX code.

### Syntax Rules and Structure

```javascript
var deferred = $.Deferred();
deferred.done(function(value) { /* ... */ });
deferred.resolve("value");
var promise = deferred.promise();
```

**Syntax Rules:**
1. Starts in "pending" state.
2. `.resolve()` / `.reject()` transitions permanently.
3. `.promise()` returns restricted view.

**Constraints and Limitations:**
- Pre-jQuery 3.0 not fully Promises/A+ compliant.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Custom Deferred**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.Deferred() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();

        deferred.done(function(value) {
            $("#output").text("Done: " + value);
        });

        setTimeout(function() {
            deferred.resolve("Operation complete");
        }, 1000);
    </script>
</body>
</html>
```

**Expected Output:** After 1 second, the output div displays "Done: Operation complete".

**Why This Code Produces That Result:** The Deferred is resolved after 1 second, firing the `.done()` callback.

### Real-World Cases
- **Custom Async Operations:** Wrapping WebSocket or timer-based operations.
- **Plugin APIs:** Returning Promises from plugin methods.

---

## 30. Utility Best Practices

### Definitions

**Core Definition:** jQuery Utility Best Practices are guidelines for using `jQuery.*` utility functions efficiently, favoring native JavaScript methods where available, and avoiding deprecated functions.

**Technical Definition:** Best practices encompass preferring native ES5+ equivalents (`Array.isArray()`, `JSON.parse()`, `String.prototype.trim()`, `Function.prototype.bind()`), avoiding deprecated utilities (`$.isArray()`, `$.isFunction()`, `$.trim()`, `$.parseJSON()`, `$.now()`, `$.proxy()`), using `$.extend(true, ...)` carefully to avoid prototype pollution, leveraging `$.when()` and `$.Deferred()` for async coordination, and using `$.escapeSelector()` for special characters. Note: `$.extend(true, {}, ...)` (deep merge with an empty object) is NOT vulnerable to prototype pollution; only versions before 3.4.0 were vulnerable when using `$.extend(true, ...)` without an empty target.

### Purposes

- To write modern, maintainable code using native APIs.
- To avoid deprecated jQuery utilities.
- To ensure security and performance.
- To leverage jQuery utilities where they add value.

### Syntax Rules and Structure

**Native Replacements for Deprecated Utilities:**

| jQuery Utility | Native Replacement | Deprecated Since |
|----------------|-------------------|------------------|
| `$.isArray()` | `Array.isArray()` | jQuery 3.3 |
| `$.isFunction()` | `typeof x === "function"` | jQuery 3.3 |
| `$.isNumeric()` | `typeof x === "number"` / `Number.isFinite()` | jQuery 3.3 |
| `$.isWindow()` | `x != null && x === x.window` | jQuery 3.3 |
| `$.type()` | `Object.prototype.toString.call()` | jQuery 3.3 |
| `$.trim()` | `String.prototype.trim()` | jQuery 3.5 |
| `$.parseJSON()` | `JSON.parse()` | jQuery 3.0 |
| `$.now()` | `Date.now()` | jQuery 3.3 |
| `$.proxy()` | `Function.prototype.bind()` | jQuery 3.3 |
| `$.nodeName()` | `element.nodeName` | jQuery 3.3 |
| `$.globalEval()` | Avoid; use `window.eval()` | — |

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Use native methods for type checking | `$.isArray(x)` | `Array.isArray(x)` |
| 2 | Use native JSON parsing | `$.parseJSON(s)` | `JSON.parse(s)` |
| 3 | Use native string trim | `$.trim(s)` | `s.trim()` |
| 4 | Use native function binding | `$.proxy(fn, ctx)` | `fn.bind(ctx)` |
| 5 | Use `Date.now()` | `$.now()` | `Date.now()` |
| 6 | Avoid `$.globalEval()` | `$.globalEval(code)` | `window.eval()` / `Function()` |
| 7 | Deep merge with empty target | `$.extend(true, defaults, opts)` | `$.extend(true, {}, defaults, opts)` |
| 8 | Use `$.when()` for parallel | Nested callbacks | `$.when(p1, p2)` |
| 9 | Use `$.escapeSelector()` | Manual escaping | `$.escapeSelector(id)` |
| 10 | Use `$.each()` only for objects | `$.each(array, fn)` | `array.forEach(fn)` |

**Syntax Rules:**

1. Prefer native JavaScript methods for type checking, parsing, and iteration.
2. Use `$.extend(true, {}, defaults, options)` with an empty target to avoid modifying defaults.
3. Avoid `$.globalEval()` with untrusted input.
4. Use `$.escapeSelector()` for dynamic IDs with special characters.
5. Use `$.when()` for coordinating parallel async operations.
6. Use `$.Deferred()` for custom asynchronous operations.

**Constraints and Limitations:**

- Native methods may not be available in very old browsers (IE8 and below).
- jQuery utilities remain useful for cross-browser compatibility in legacy code.
- Deprecated utilities still work but may be removed in future versions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Modern Replacements for Deprecated Utilities**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Utility Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // ✅ Native replacements for deprecated jQuery utilities
        var arr = [1, 2, 3];

        // Instead of $.isArray()
        console.log("isArray:", Array.isArray(arr)); // true

        // Instead of $.parseJSON()
        var obj = JSON.parse('{"name":"John"}');
        console.log("Parsed:", obj.name); // "John"

        // Instead of $.trim()
        var trimmed = "  hello  ".trim();
        console.log("Trimmed:", "[" + trimmed + "]"); // [hello]

        // Instead of $.now()
        var start = Date.now();

        // Instead of $.proxy()
        var context = { name: "Context" };
        var fn = function() { return this.name; }.bind(context);
        console.log("Bound:", fn()); // "Context"

        // ✅ jQuery utilities still useful
        // $.extend with empty target (safe deep merge)
        var defaults = { a: 1, nested: { x: 1 } };
        var options = { b: 2, nested: { y: 2 } };
        var merged = $.extend(true, {}, defaults, options);
        console.log("Merged:", JSON.stringify(merged));
        // {"a":1,"nested":{"x":1,"y":2},"b":2}

        // $.escapeSelector for special characters
        var escaped = $.escapeSelector("some.id");
        console.log("Escaped:", escaped); // "some\.id"

        $("#output").text("Best practices applied. Check console.");
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Best practices applied. Check console." Console logs the results of each utility replacement.

**Why This Code Produces That Result:** The code uses native methods (`Array.isArray`, `JSON.parse`, `.trim()`, `Date.now()`, `.bind()`) instead of deprecated jQuery utilities. It still uses `$.extend(true, {}, ...)` (safe form) and `$.escapeSelector()` where jQuery adds value.

### Real-World Cases

- **Modernization:** Migrating legacy jQuery code to native APIs.
- **Plugin Development:** Using `$.extend()` safely for options.
- **Security:** Avoiding `$.globalEval()` with untrusted input.

---

## References

- jQuery API Documentation — Utilities Category - https://api.jquery.com/category/utilities/
- jQuery API Documentation — jQuery.each() - https://api.jquery.com/jQuery.each/
- jQuery API Documentation — jQuery.map() - https://api.jquery.com/jQuery.map/
- jQuery API Documentation — jQuery.grep() - https://api.jquery.com/jQuery.grep/
- jQuery API Documentation — jQuery.inArray() - https://api.jquery.com/jQuery.inArray/
- jQuery API Documentation — jQuery.extend() - https://api.jquery.com/jQuery.extend/
- jQuery API Documentation — jQuery.when() - https://api.jquery.com/jQuery.when/
- jQuery API Documentation — jQuery.Deferred() - https://api.jquery.com/jQuery.Deferred/
- jQuery API Documentation — jQuery.parseHTML() - https://api.jquery.com/jQuery.parseHTML/
- jQuery API Documentation — jQuery.escapeSelector() - https://api.jquery.com/jQuery.escapeSelector/
- jQuery API Documentation — jQuery.holdReady() - https://api.jquery.com/jQuery.holdReady/
- jQuery API Documentation — jQuery.ready - https://api.jquery.com/jQuery.ready/
- jQuery 3.0 Upgrade Guide - https://jquery.com/upgrade-guide/3.0/
- jQuery 3.3 Upgrade Guide (Deprecations) - https://jquery.com/upgrade-guide/3.3/
- jQuery 3.5 Upgrade Guide (Deprecations) - https://jquery.com/upgrade-guide/3.5/
- jQuery 3.4.0 Release Notes (Prototype Pollution Fix) - https://blog.jquery.com/2019/04/10/jquery-3-4-0-released/
- MDN Web Docs — Array.isArray() - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray
- MDN Web Docs — JSON.parse() - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- MDN Web Docs — Function.prototype.bind() - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
- MDN Web Docs — Date.now() - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now
- MDN Web Docs — String.prototype.trim() - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/trim
- OWASP Web Frontend Security Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Web_Frontend_Security_Cheat_Sheet.html