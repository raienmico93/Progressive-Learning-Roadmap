# jQuery Properties: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery's `.prop()` and `.removeProp()` methods are the tools for reading, setting, and removing **DOM properties** — the dynamic, JavaScript-side state of an element — as opposed to the static HTML attributes written in the source markup.

**Technical Definition**
A **property** is a member of a DOM node object in JavaScript, representing its current state at runtime. Properties such as `checked`, `selected`, `disabled`, `value`, and `tagName` are accessed and modified directly on the DOM element object. jQuery's `.prop()` method provides a cross-browser, chainable interface for reading and writing these properties, and `.removeProp()` removes custom properties that were previously set via `.prop()`. Since jQuery 1.6, `.prop()` has been the recommended method for working with boolean attributes and form control state, while `.attr()` is reserved for HTML attributes.

**Beginner-Friendly Explanation**
Think of an HTML element as having two layers. The first layer is the **attribute** — the original settings written in the HTML code, like `checked` on a checkbox. The second layer is the **property** — the live, current state of the element as the user interacts with it. If a user clicks a checkbox, the **property** changes but the **attribute** does not. jQuery's `.prop()` works with the live layer, giving you the true current state. This is why `.prop()` is the right tool for checkboxes, radio buttons, and dropdowns.

### Key Characteristics

- **Dynamic State**: `.prop()` reads and writes the **current** state of an element, which can change as the user interacts with the page.
- **Boolean Type Support**: Properties like `checked`, `disabled`, and `selected` return and accept **boolean** values (`true`/`false`), unlike attributes which return strings.
- **Cross-Browser Consistency**: `.prop()` normalises differences in how browsers expose DOM properties.
- **Chainable Setters**: Like most jQuery methods, `.prop()` returns the jQuery object when used as a setter, preserving method chaining.
- **Custom Properties**: `.prop()` can store arbitrary JavaScript values on elements, though `.data()` is preferred for complex data due to memory leak risks in older browsers.
- **`.removeProp()` Caution**: Native properties should **not** be removed with `.removeProp()`; set them to `false` with `.prop()` instead.

### Prerequisites

- Basic understanding of HTML attributes (`checked`, `disabled`, `selected`, etc.).
- Familiarity with jQuery selectors and the `$()` function.
- Awareness of the difference between HTML attributes and DOM properties.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **Form Handling**: `.prop()` is essential for reading and setting checkbox, radio, and select states.
- **DOM Manipulation**: Properties represent the live state of the DOM tree.
- **Event Handling**: Form control state changes are often read via `.prop()` in event handlers.
- **Plugin Development**: Custom properties can be stored on elements, though `.data()` is generally preferred.

### Core Concepts / Features

1. `.prop()` — Get or Set DOM Properties
2. `.removeProp()` — Remove Custom Properties
3. Properties vs. Attributes
4. Managing Checkbox, Radio, and Select Form Control States

---

## Core Concept 1: `.prop()` — Get or Set DOM Properties

### Definitions

**Core Definition**
`.prop()` gets the value of a property for the first element in the set of matched elements, or sets one or more properties for every matched element.

**Technical Definition**
The `.prop()` method accesses the JavaScript **property** on a DOM element object. As a getter, it returns the property value for only the **first** element in the matched set, returning `undefined` if the property has not been set or if the set is empty. As a setter, it assigns the value to the property on **every** matched element and returns the jQuery object for chaining. The method accepts a single property name, a name-value pair, an object of multiple name-value pairs, or a callback function. It was introduced in jQuery 1.6 to provide explicit access to property values, separate from the `.attr()` method's attribute-focused behaviour.

**Beginner-Friendly Explanation**
`.prop()` is how you read or change the **live state** of an element. If you want to know whether a checkbox is currently ticked, you use `.prop("checked")` — it returns `true` or `false`. If you want to tick it, you use `.prop("checked", true)`. This is different from `.attr()`, which reads the original HTML setting.

### Purposes

- To retrieve the **current** boolean state of form controls (`checked`, `disabled`, `selected`).
- To set the state of form controls programmatically.
- To access DOM properties that have no attribute equivalent (`tagName`, `nodeName`, `selectedIndex`, `defaultChecked`).
- To store and retrieve custom properties on elements (with caution in older browsers).
- To compute property values dynamically using a callback function.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter
$(selector).prop(propertyName);

// Setter — single property
$(selector).prop(propertyName, value);
$(selector).prop(propertyName, function(index, oldValue) { ... });

// Setter — multiple properties
$(selector).prop({ property1: value1, property2: value2 });
```

**Component Breakdown**

- `propertyName` (String): The name of the DOM property to get or set.
- `value` (Any): The value to assign. Booleans are the most common for state properties.
- `function(index, oldValue)`: A callback returning the value to set. `index` is the element's position; `oldValue` is the current property value.
- `{ property1: value1, ... }` (Object): An object of key-value pairs for setting multiple properties.
- Returns: The property value (getter) or a jQuery object (setter).

**Syntax Rules**

1. As a getter, `.prop()` returns the value for the **first** element only.
2. As a setter, it applies the value to **all** matched elements.
3. For boolean properties, pass `true` or `false` — **not** strings like `"true"` or `"false"`.
4. Introduced in jQuery 1.6; prior to that, `.attr()` sometimes returned property values for boolean attributes.
5. The callback receives the element's index and the current property value.

**Constraints and Limitations**

- Attempting to change the `type` property of an input element in Internet Explorer 6, 7, or 8 throws an error.
- Setting a property to a non-primitive value (object or array) in IE < 9 can cause memory leaks unless `.removeProp()` is called before the element is removed.
- `.prop()` cannot be used to set properties on plain JavaScript objects, `window`, or `document`.

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Setting a Boolean Property**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prop() — Checkbox State</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="checkbox" id="agree" checked>
  <label for="agree">I agree to the terms</label>
  <button id="toggle">Toggle Agreement</button>
  <p id="status"></p>

  <script>
    $(function () {
      // Step 1: GET — read the current checked state (boolean)
      var isChecked = $("#agree").prop("checked");
      console.log("Initially checked:", isChecked); // true
      $("#status").text("Checked: " + isChecked);

      // Step 2: SET — uncheck the checkbox
      $("#agree").prop("checked", false);
      console.log("After uncheck:", $("#agree").prop("checked")); // false

      // Step 3: Toggle with a button
      $("#toggle").on("click", function () {
        var current = $("#agree").prop("checked");
        $("#agree").prop("checked", !current);
        $("#status").text("Checked: " + $("#agree").prop("checked"));
        console.log("Toggled to:", $("#agree").prop("checked"));
      });

      // Step 4: SET multiple properties with an object
      $("#agree").prop({
        checked: true,
        disabled: false
      });
      console.log("After object set — checked:", $("#agree").prop("checked")); // true
      console.log("After object set — disabled:", $("#agree").prop("disabled")); // false
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Initially checked: true
After uncheck: false
Toggled to: true
After object set — checked: true
After object set — disabled: false
```

**Why This Output Occurs**
`.prop("checked")` returns the boolean property value, not a string. Setting `.prop("checked", false)` updates the live state. The object form sets multiple properties at once. All operations return the jQuery object for chaining.

---

**Example 2: Using a Callback to Set Properties**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prop() — Callback Function</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="checkbox" class="item" value="1"> Item 1<br>
  <input type="checkbox" class="item" value="2"> Item 2<br>
  <input type="checkbox" class="item" value="3"> Item 3<br>
  <button id="checkEven">Check Even Items</button>

  <script>
    $(function () {
      // Set the disabled property using a callback
      $(".item").prop("disabled", function (index, oldValue) {
        // Disable the second checkbox (index 1)
        return index === 1;
      });

      // Log the disabled states
      $(".item").each(function (i) {
        console.log("Item " + i + " disabled:", $(this).prop("disabled"));
      });

      // Use a callback to check even-indexed items
      $("#checkEven").on("click", function () {
        $(".item").prop("checked", function (index) {
          return index % 2 === 0; // Check items at indices 0 and 2
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Item 0 disabled: false
Item 1 disabled: true
Item 2 disabled: false
```

**Why This Output Occurs**
The callback receives each element's index and returns `true` for index 1, disabling only the second checkbox. The second callback returns `true` for even indices, checking the first and third checkboxes when the button is clicked.

### Real-World Cases

- **Select All Checkbox**: `$("#selectAll").prop("checked", true)` checks all child checkboxes.
- **Form Validation**: `$("#submit").prop("disabled", !isValid)` enables or disables the submit button.
- **Radio Button Reset**: `$("input[name=choice]").prop("checked", false)` clears all radio selections.
- **Read-Only Toggle**: `$("#field").prop("readonly", true)` makes a field read-only.

### References

- jQuery API — .prop() – https://api.jquery.com/prop/
- W3Schools — jQuery prop() Method – https://www.w3schools.com/jquery/html_prop.asp
- jQuery 1.6 Released — .attr() and .prop() – https://blog.jquery.com/2011/05/03/jquery-16-released/

---

## Core Concept 2: `.removeProp()` — Remove Custom Properties

### Definitions

**Core Definition**
`.removeProp()` removes a property set by the `.prop()` method for the set of matched elements.

**Technical Definition**
The `.removeProp()` method deletes a property from a DOM element object. It is intended **only** for removing custom properties that were previously set via `.prop()`. The official jQuery documentation explicitly warns that it should **not** be used to remove built-in (native) properties such as `checked`, `disabled`, `selected`, or others, because doing so removes the property completely and it cannot be added back to the element. For native properties, the correct approach is to use `.prop()` to set the property to `false`.

**Beginner-Friendly Explanation**
`.removeProp()` is the cleanup tool for custom properties you have added to an element yourself. It is **not** for removing standard properties like `checked` or `disabled` — for those, you should set them to `false` instead. Using `.removeProp()` on a native property can break the element permanently.

### Purposes

- To remove custom properties that were previously set via `.prop()`.
- To clean up temporary properties before removing an element from the DOM.
- To avoid memory leaks in older browsers (IE < 9) when non-primitive values were stored via `.prop()`.
- To reset an element's custom property state.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).removeProp(propertyName);
```

**Component Breakdown**

- `propertyName` (String): The name of the property to remove.
- Returns: A jQuery object (for chaining).

**Syntax Rules**

1. Removes the property from **every** matched element.
2. Should **only** be used on custom properties set via `.prop()`.
3. Should **not** be used on native properties (`checked`, `disabled`, `selected`, etc.).
4. For native boolean properties, use `.prop(propertyName, false)` instead.
5. Added in jQuery 1.6.

**Constraints and Limitations**

- Removing a native property completely breaks the element's ability to use that property again.
- In IE < 9, using `.prop()` to set a non-primitive value without later calling `.removeProp()` can cause memory leaks.
- After removal, a subsequent `.prop(propertyName)` returns `undefined`.

### Multiple Annotated Complete Code Examples

**Example 1: Setting and Removing a Custom Property**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeProp() — Custom Property</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p id="demo"></p>
  <button id="remove">Remove Custom Property</button>

  <script>
    $(function () {
      // Step 1: Set a custom property
      $("#demo").prop("luggageCode", 1234);
      $("#demo").append("The secret luggage code is: " +
        String($("#demo").prop("luggageCode")) + ". ");

      // Step 2: Remove the custom property
      $("#remove").on("click", function () {
        $("#demo").removeProp("luggageCode");
        $("#demo").append("Now the secret luggage code is: " +
          String($("#demo").prop("luggageCode")) + ". ");
        console.log("After removal:", $("#demo").prop("luggageCode")); // undefined
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The paragraph initially displays “The secret luggage code is: 1234.”
- After clicking the button, it appends “Now the secret luggage code is: undefined.”
- Console output: `After removal: undefined`

**Why This Output Occurs**
`.prop("luggageCode", 1234)` stores the number `1234` as a custom property. `.removeProp("luggageCode")` deletes it, and a subsequent `.prop()` call returns `undefined`.

---

**Example 2: Why Native Properties Should Not Be Removed**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeProp() — Native Property Warning</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="checkbox" id="box" checked>

  <script>
    $(function () {
      // --- INCORRECT: Removing a native property ---
      // This will remove the "checked" property completely
      // $("#box").removeProp("checked");
      // After this, $("#box").prop("checked") returns undefined,
      // and the checkbox can never be checked again!

      // --- CORRECT: Set the native property to false ---
      $("#box").prop("checked", false);
      console.log("After prop(false):", $("#box").prop("checked")); // false

      // The checkbox can still be re-checked
      $("#box").prop("checked", true);
      console.log("Re-checked:", $("#box").prop("checked")); // true
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
After prop(false): false
Re-checked: true
```

**Why This Output Occurs**
Setting `.prop("checked", false)` changes the boolean value without removing the property. The property remains on the element, so it can be set back to `true` later. If `.removeProp("checked")` were used instead, the property would be permanently removed and subsequent `.prop("checked")` calls would return `undefined`.

### Real-World Cases

- **Temporary State**: `$("#temp").prop("processing", true)` followed by `$("#temp").removeProp("processing")` after the operation completes.
- **Memory Leak Prevention**: In IE < 9, `$("#el").removeProp("largeObject")` before removing the element from the DOM.
- **Plugin Cleanup**: `$("#widget").removeProp("widgetState")` during plugin teardown.
- **Never for Native Properties**: Use `.prop("checked", false)` instead of `.removeProp("checked")`.

### References

- jQuery API — .removeProp() – https://api.jquery.com/removeprop/
- W3Schools — jQuery removeProp() Method – https://www.w3schools.com/jquery/html_removeprop.asp

---

## Core Concept 3: Properties vs. Attributes

### Definitions

**Core Definition**
Attributes are the values written in the HTML source and accessed via `.attr()`; properties are the dynamic state of the DOM element object at runtime, accessed via `.prop()`.

**Technical Definition**
An **attribute** is a piece of information defined in the HTML markup (e.g., `checked="checked"`) and stored in the DOM's attribute list. It represents the **initial** or **default** state. A **property** is a member of the DOM element's JavaScript object (e.g., `elem.checked`), representing the **current** state. For boolean attributes, the attribute's presence sets the initial property value, but subsequent user interaction changes only the property, not the attribute. Some properties (e.g., `selectedIndex`, `tagName`, `nodeName`, `nodeType`, `ownerDocument`, `defaultChecked`) have **no corresponding attribute** and can only be accessed via `.prop()`.

**Beginner-Friendly Explanation**
An **attribute** is like a sticky note on a box that says “Fragile” — it was written when the box was packed and does not change. A **property** is like a switch on the box that can be flipped on and off — it shows the current state. If someone flips the switch, the sticky note still says “Fragile,” but the switch shows the real, current condition. jQuery's `.attr()` reads the sticky note; `.prop()` reads the switch.

### Purposes

- To understand why `.attr("checked")` and `.prop("checked")` can return different values.
- To choose the correct method for the desired operation: DOM-visible changes require `.attr()`; current state requires `.prop()`.
- To avoid the pre-1.6 inconsistency where `.attr()` sometimes returned property values.
- To correctly handle boolean attributes (`checked`, `disabled`, `selected`).
- To access properties that have no attribute equivalent.

### Syntax Rules and Structure

**Comparison Table**

| Aspect | `.attr()` | `.prop()` |
|---|---|---|
| What it reads/writes | HTML attribute | DOM property |
| Data type returned | String (usually) | Any (boolean, number, string) |
| Reflects current state | No | Yes |
| Visible in DOM inspector | Yes | No (in some browsers) |
| Boolean attribute type | String (`"checked"`) | Boolean (`true`/`false`) |
| Changes with user interaction | No | Yes |
| Recommended for | `href`, `src`, `title`, `alt`, `data-*` | `checked`, `disabled`, `selected`, `value`, `tagName` |

**Syntax Rules**

1. Use `.attr()` for attributes that represent **static** information: `href`, `src`, `title`, `alt`, `id`, `class`, `data-*`.
2. Use `.prop()` for properties that represent **dynamic** state: `checked`, `disabled`, `selected`, `value`, `readonly`, `multiple`.
3. Use `.prop()` for properties with no attribute equivalent: `tagName`, `nodeName`, `nodeType`, `selectedIndex`, `defaultChecked`, `ownerDocument`.
4. As of jQuery 1.6, `.attr()` returns `undefined` for attributes that have not been set.
5. For boolean attributes, the **presence** of the attribute sets the initial property to `true`, regardless of the attribute's value.

**Constraints and Limitations**

- `.attr("checked")` returns `"checked"` (jQuery 1.6–3.x) or `""` (jQuery 4.0+) as a string, never a boolean.
- `.attr("checked")` reflects only the **initial** state, not the current state.
- Changing the attribute does not automatically update the property, and vice versa.

### Multiple Annotated Complete Code Examples

**Example 1: The Checkbox Discrepancy**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Properties vs Attributes — Checkbox</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="checkbox" id="box" checked>
  <button id="uncheck">Uncheck</button>

  <script>
    $(function () {
      // Step 1: Initial state — both agree
      console.log("Initial .attr('checked'):", $("#box").attr("checked")); // "checked"
      console.log("Initial .prop('checked'):", $("#box").prop("checked")); // true

      // Step 2: Uncheck via .prop()
      $("#uncheck").on("click", function () {
        $("#box").prop("checked", false);
        console.log("After uncheck — .attr:", $("#box").attr("checked")); // "checked" (stale!)
        console.log("After uncheck — .prop:", $("#box").prop("checked")); // false (correct!)
      });

      // Step 3: The attribute is unchanged, but the property reflects reality
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
```

**Why This Output Occurs**
The `checked` attribute is present in the HTML, so `.attr("checked")` returns `"checked"` both before and after the user unchecks the box. The `checked` property, however, changes from `true` to `false` when the box is unchecked. This is the fundamental difference: `.attr()` reads the initial state; `.prop()` reads the current state.

---

**Example 2: Properties Without Attribute Equivalents**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Properties Without Attributes</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <select id="mySelect">
    <option>Option 1</option>
    <option selected>Option 2</option>
    <option>Option 3</option>
  </select>

  <script>
    $(function () {
      // --- Properties that have NO attribute equivalent ---
      console.log("tagName:", $("#mySelect").prop("tagName"));       // "SELECT"
      console.log("nodeName:", $("#mySelect").prop("nodeName"));     // "SELECT"
      console.log("nodeType:", $("#mySelect").prop("nodeType"));     // 1
      console.log("selectedIndex:", $("#mySelect").prop("selectedIndex")); // 1

      // These return undefined via .attr()
      console.log("tagName via .attr():", $("#mySelect").attr("tagName"));       // undefined
      console.log("selectedIndex via .attr():", $("#mySelect").attr("selectedIndex")); // undefined

      // --- Changing selectedIndex via .prop() ---
      $("#mySelect").prop("selectedIndex", 2);
      console.log("New selectedIndex:", $("#mySelect").prop("selectedIndex")); // 2
      console.log("Selected value:", $("#mySelect").val()); // "Option 3"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
tagName: SELECT
nodeName: SELECT
nodeType: 1
selectedIndex: 1
tagName via .attr(): undefined
selectedIndex via .attr(): undefined
New selectedIndex: 2
Selected value: Option 3
```

**Why This Output Occurs**
Properties like `tagName`, `nodeName`, `nodeType`, and `selectedIndex` exist only as DOM properties and have no corresponding HTML attribute. `.attr()` returns `undefined` for all of them. `.prop("selectedIndex", 2)` changes the selection to the third option, and `.val()` confirms the new value.

### Real-World Cases

- **Form Validation**: Use `.prop("disabled", true)` to disable a submit button, not `.attr("disabled", true)`.
- **Checkbox State**: Use `.prop("checked")` to read the current state, not `.attr("checked")`.
- **Link URLs**: Use `.attr("href")` to read or set the link destination, not `.prop("href")`.
- **Image Sources**: Use `.attr("src")` to change an image source, not `.prop("src")`.
- **Element Identification**: Use `.prop("tagName")` to get an element's tag name — there is no attribute for this.

### References

- jQuery API — .attr() (Attributes vs. Properties) – https://api.jquery.com/attr/
- jQuery API — .prop() (Attributes vs. Properties) – https://api.jquery.com/prop/
- jQuery 1.6 Released — .attr() and .prop() – https://blog.jquery.com/2011/05/03/jquery-16-released/
- W3C — HTML Forms Specification: Boolean Attributes – https://www.w3.org/TR/html401/interact/forms.html#h-17.4
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/

---

## Core Concept 4: Managing Checkbox, Radio, and Select Form Control States

### Definitions

**Core Definition**
Managing form control states means using `.prop()` to accurately read and set the `checked`, `selected`, and `disabled` properties of checkboxes, radio buttons, and select elements, reflecting the user's current interaction with the form.

**Technical Definition**
Form controls such as checkboxes, radio buttons, and select elements have **boolean properties** (`checked`, `selected`, `disabled`) that reflect their current state. These properties are updated dynamically as the user interacts with the page. jQuery recommends using `.prop()` for these properties, because `.attr()` returns the **initial** state (the value from the HTML source) rather than the current state. For select elements, the `selected` property applies to individual `<option>` elements, while `selectedIndex` (accessed via `.prop()`) reflects the index of the currently selected option. The `.val()` method provides a higher-level interface for getting and setting selected values in select elements.

**Beginner-Friendly Explanation**
Checkboxes, radio buttons, and dropdowns all change as the user interacts with them. To know **right now** whether a checkbox is ticked, you use `.prop("checked")`. To tick it, you use `.prop("checked", true)`. For dropdowns, `.val()` is usually easier — it gives you the selected value directly. Using `.attr()` for these tasks gives you the **starting** state, not the current one, which is almost never what you want.

### Purposes

- To accurately read the current checked state of checkboxes and radio buttons.
- To programmatically check or uncheck checkboxes and radio buttons.
- To read and set the selected option in a select element.
- To enable or disable form controls based on validation or application state.
- To reset form controls to their default states.

### Syntax Rules and Structure

**Complete General Syntax**

**Checkboxes:**
```javascript
$(selector).prop("checked");              // get (boolean)
$(selector).prop("checked", true/false);  // set
```

**Radio Buttons:**
```javascript
$("input[name=group]:checked").val();     // get selected value
$("input[name=group]").prop("checked", false); // clear all
$("input[name=group][value='x']").prop("checked", true); // select specific
```

**Select Elements:**
```javascript
$(selector).val();                        // get selected value
$(selector).val("value");                 // set selected value
$(selector).prop("selectedIndex");        // get index of selected option
$("option[value='x']").prop("selected", true); // select specific option
```

**Component Breakdown**

- `.prop("checked")` : Returns `true` if a checkbox or radio is checked.
- `.prop("checked", true/false)` : Sets the checked state of all matched elements.
- `.val()` : Returns the value of the selected option in a `<select>`.
- `.prop("selectedIndex")` : Returns the zero-based index of the selected option.
- `.prop("selected", true)` : Selects a specific `<option>` element.

**Syntax Rules**

1. Use `.prop("checked", true/false)` for checkboxes and radio buttons, **not** `.attr("checked", ...)`.
2. Use `.val()` to get the value of the selected option in a `<select>`.
3. For multi-select elements, `.val()` returns an **array** of selected values.
4. To select a specific option, use `.prop("selected", true)` on the `<option>`.
5. To clear all radio buttons in a group, use `.prop("checked", false)`.
6. As of jQuery 3.0, `.removeAttr("checked")` no longer sets the `checked` property to `false`.

**Constraints and Limitations**

- `.attr("checked")` returns a string, not a boolean, and does not reflect current state.
- `.val()` on a select returns `null` if no option is selected (single select).
- For multi-select elements, `.val()` returns an empty array if no options are selected.

### Multiple Annotated Complete Code Examples

**Example 1: Checkboxes — Reading, Setting, and Counting**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Checkbox State with .prop()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <h3>Select your hobbies:</h3>
  <label><input type="checkbox" name="hobby" value="reading"> Reading</label><br>
  <label><input type="checkbox" name="hobby" value="gaming"> Gaming</label><br>
  <label><input type="checkbox" name="hobby" value="cooking"> Cooking</label><br>
  <label><input type="checkbox" name="hobby" value="travel"> Travel</label><br>
  <button id="selectAll">Select All</button>
  <button id="clearAll">Clear All</button>
  <p id="count"></p>

  <script>
    $(function () {
      // Step 1: COUNT checked checkboxes
      function updateCount() {
        var count = $("input[name='hobby']:checked").length;
        $("#count").text(count + " selected");
        console.log("Selected count:", count);
      }

      // Step 2: Listen for changes
      $("input[name='hobby']").on("change", updateCount);
      updateCount(); // initial count

      // Step 3: SELECT ALL
      $("#selectAll").on("click", function () {
        $("input[name='hobby']").prop("checked", true);
        updateCount();
      });

      // Step 4: CLEAR ALL
      $("#clearAll").on("click", function () {
        $("input[name='hobby']").prop("checked", false);
        updateCount();
      });

      // Step 5: GET selected values
      $("input[name='hobby']").on("change", function () {
        var selected = $("input[name='hobby']:checked").map(function () {
          return this.value;
        }).get();
        console.log("Selected values:", selected);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking “Select All” checks all four checkboxes and updates the count to “4 selected”.
- Clicking “Clear All” unchecks all and updates the count to “0 selected”.
- Console output shows the count and the array of selected values.

**Why This Output Occurs**
`.prop("checked", true)` sets the checked property on all matched checkboxes, which updates their visual state and triggers the `change` event. The `:checked` selector and `.length` count how many are currently checked. `.map()` and `.get()` collect the values of the selected checkboxes into an array.

---

**Example 2: Radio Buttons — Selecting and Reading**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Radio Button State with .prop()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <h3>Choose a payment method:</h3>
  <label><input type="radio" name="payment" value="credit"> Credit Card</label><br>
  <label><input type="radio" name="payment" value="debit"> Debit Card</label><br>
  <label><input type="radio" name="payment" value="paypal"> PayPal</label><br>
  <button id="selectPaypal">Select PayPal</button>
  <p id="selected"></p>

  <script>
    $(function () {
      // Step 1: READ the selected radio value
      function showSelected() {
        var value = $("input[name='payment']:checked").val();
        $("#selected").text("Selected: " + (value || "none"));
        console.log("Selected payment:", value);
      }

      // Step 2: Listen for changes
      $("input[name='payment']").on("change", showSelected);
      showSelected();

      // Step 3: SELECT a specific radio programmatically
      $("#selectPaypal").on("click", function () {
        $("input[name='payment'][value='paypal']").prop("checked", true);
        showSelected();
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking “Select PayPal” selects the PayPal radio button and updates the paragraph to “Selected: paypal”.
- Console output shows the selected payment method.

**Why This Output Occurs**
`.prop("checked", true)` on the specific radio button selects it. Because radio buttons in the same group share a `name` attribute, selecting one automatically deselects the others. The `:checked` selector retrieves the currently selected radio, and `.val()` returns its value.

---

**Example 3: Select Elements — Reading and Setting**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Select State with .val() and .prop()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <h3>Choose a country:</h3>
  <select id="country">
    <option value="">-- Select --</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
  </select>

  <h3>Choose languages (multi-select):</h3>
  <select id="langs" multiple>
    <option value="js">JavaScript</option>
    <option value="py">Python</option>
    <option value="java">Java</option>
    <option value="go">Go</option>
  </select>

  <button id="preset">Preset Choices</button>
  <p id="output"></p>

  <script>
    $(function () {
      // Step 1: READ single select value
      $("#country").on("change", function () {
        console.log("Country:", $(this).val()); // e.g., "us"
        $("#output").text("Country: " + $(this).val());
      });

      // Step 2: READ multi-select values (array)
      $("#langs").on("change", function () {
        var selected = $(this).val();
        console.log("Languages:", selected); // e.g., ["js", "py"]
      });

      // Step 3: SET values programmatically
      $("#preset").on("click", function () {
        // Set single select
        $("#country").val("ca");

        // Set multi-select (array of values)
        $("#langs").val(["js", "go"]);

        console.log("Country after preset:", $("#country").val()); // "ca"
        console.log("Languages after preset:", $("#langs").val()); // ["js", "go"]

        // Read selectedIndex via .prop()
        console.log("Country selectedIndex:",
          $("#country").prop("selectedIndex")); // 3
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Country: us
Languages: ["js", "py"]
Country after preset: ca
Languages after preset: ["js", "go"]
Country selectedIndex: 3
```

**Why This Output Occurs**
`.val()` on a single select returns the selected option's value as a string. On a multi-select, it returns an array of selected values. Setting `.val("ca")` selects the matching option, and `.val(["js", "go"])` selects multiple options. `.prop("selectedIndex")` returns the zero-based index of the selected option in the single select.

### Real-World Cases

- **Select All / Clear All**: `$(".item").prop("checked", true)` for bulk checkbox operations.
- **Dynamic Radio Groups**: `$("input[name=size][value='large']").prop("checked", true)` to pre-select a radio option.
- **Multi-Select Tags**: `$("#tags").val(["js", "css", "html"])` to pre-select multiple tags.
- **Form Validation**: `if ($("input[name=payment]:checked").length === 0) { alert("Please select a payment method"); }`.
- **Disabled State Management**: `$("#submit").prop("disabled", !isValid)` to enable/disable submission.

### References

- jQuery API — .prop() – https://api.jquery.com/prop/
- jQuery API — .val() – https://api.jquery.com/val/
- jQuery API — :checked Selector – https://api.jquery.com/checked-selector/
- jQuery API — :selected Selector – https://api.jquery.com/selected-selector/
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/

---

## Summary Table: jQuery Properties at a Glance

| Method | Operates On | Getter Scope | Setter Effect | Type Returned | Key Use |
|---|---|---|---|---|---|
| `.prop()` | DOM properties | First matched element | All matched elements | Any (boolean, number, string) | Form state, custom properties |
| `.removeProp()` | DOM properties | N/A | Removes from all matched elements | N/A | Custom property cleanup only |

### Decision Guide: `.prop()` vs. `.attr()` vs. `.val()`

| Goal | Use | Example |
|---|---|---|
| Read/write checkbox or radio state | `.prop()` | `$("#box").prop("checked")` |
| Read/write select selected state | `.prop()` or `.val()` | `$("#sel").val()` |
| Enable/disable a form control | `.prop()` | `$("#btn").prop("disabled", true)` |
| Get an element's tag name | `.prop()` | `$("#el").prop("tagName")` |
| Read/write an HTML attribute (`href`, `src`) | `.attr()` | `$("a").attr("href")` |
| Read/write form field value (text) | `.val()` | `$("#name").val()` |
| Store/retrieve custom data | `.data()` | `$("#el").data("key", value)` |

---

## General References

- jQuery API — .prop() – https://api.jquery.com/prop/
- jQuery API — .removeProp() – https://api.jquery.com/removeprop/
- jQuery API — .attr() – https://api.jquery.com/attr/
- jQuery API — .val() – https://api.jquery.com/val/
- jQuery API — Attributes Category – https://api.jquery.com/category/attributes/
- jQuery 1.6 Released — .attr() and .prop() – https://blog.jquery.com/2011/05/03/jquery-16-released/
- W3Schools — jQuery prop() Method – https://www.w3schools.com/jquery/html_prop.asp
- W3Schools — jQuery removeProp() Method – https://www.w3schools.com/jquery/html_removeprop.asp
- W3C — HTML Forms Specification: Boolean Attributes – https://www.w3.org/TR/html401/interact/forms.html#h-17.4
- WHATWG DOM — Boolean Interface for Properties – https://lists.w3.org/Archives/Public/public-webapps-github/2017May/1398.html
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/
- Stack Overflow — Why use .prop() instead of .attr() for boolean attributes? – https://stackoverflow.com/questions/5874652/