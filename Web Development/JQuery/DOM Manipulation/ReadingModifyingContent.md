# jQuery Reading and Modifying Content: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery's content manipulation methods — `.text()`, `.html()`, and `.val()` — are the primary tools for reading from and writing to the content of DOM elements, covering plain text, HTML markup, and form field values respectively.

**Technical Definition**
These methods follow jQuery's dual getter/setter pattern. When called without arguments, they act as **getters** and return a value; when called with an argument, they act as **setters** and modify the matched elements, returning the jQuery object to preserve chainability. `.text()` maps to the DOM's `textContent` (or `innerText`) property, `.html()` maps to `innerHTML`, and `.val()` reads and writes the `value` property of form controls. Each method also accepts a callback function for computed assignments.

**Beginner-Friendly Explanation**
When you build a web page, you often need to read what is inside an element or change it. jQuery gives you three simple tools for this. `.text()` works with plain words, `.html()` works with HTML code (tags and all), and `.val()` works with form fields like text boxes and dropdowns. Each one can either **read** what is there or **replace** it with something new.

### Key Characteristics

- **Dual Getter/Setter Behaviour**: Called without arguments, each method retrieves a value; called with an argument, it sets the content of all matched elements.
- **Different Scopes on Read**: `.text()` returns the combined text of **all** matched elements, whereas `.html()` returns the content of only the **first** matched element. `.val()` returns the value of the **first** matched element.
- **Safety Profiles Differ**: `.text()` automatically escapes HTML, making it safe against XSS. `.html()` does **not** escape and is a known XSS sink. `.val()` sets a property value and does not parse HTML.
- **Callback Support**: All three methods accept a function that receives the element's index and its current value.
- **Complementary Use**: `.text()` is for non-form content, `.val()` is for form controls, and `.html()` is for trusted HTML markup.

### Prerequisites

- Basic understanding of HTML elements and attributes.
- Familiarity with jQuery selectors and the `$()` function.
- Awareness of form elements (`<input>`, `<select>`, `<textarea>`).
- A basic understanding of Cross-Site Scripting (XSS) as a security concept.

### Related Programming Areas

- **DOM Manipulation**: These methods are the most frequently used jQuery manipulation tools.
- **Form Handling**: `.val()` is essential for reading and resetting form data.
- **Security**: Understanding the difference between `.text()` and `.html()` is critical for preventing XSS.
- **Dynamic Content**: All three methods are used to inject or update content after page load.

### Core Concepts / Features

1. `.text()` — Get or Set Combined Text Contents
2. `.html()` — Get or Set HTML Contents
3. `.val()` — Get or Set Form Element Values
4. Reading vs. Replacing Element Content
5. Security Considerations of `.html()` (XSS Risks)

---

## Core Concept 1: `.text()` — Get or Set Combined Text Contents

### Definitions

**Core Definition**
`.text()` gets the combined text contents of each element in the set of matched elements, including their descendants, or sets the text contents of the matched elements.

**Technical Definition**
When called without arguments, `.text()` returns a string containing the combined text of all matched elements, including text and CDATA nodes as well as element nodes (as of jQuery 1.4). It uses the DOM's `textContent` property (or `innerText` depending on the browser). When called with a string argument, it sets the text content of **every** matched element, replacing all existing content (including any child elements). Internally, jQuery uses `document.createTextNode()`, which escapes the string so that it renders as literal text rather than being interpreted as HTML.

**Beginner-Friendly Explanation**
`.text()` deals only with the words inside an element, not the HTML tags. If you read it, you get all the visible text. If you set it, whatever you provide is treated as plain text — any `<` or `>` characters are shown as-is, not interpreted as tags. This makes it the safe choice for inserting user-provided content.

### Purposes

- To retrieve the combined, tag-free text content of a set of elements.
- To safely set the text content of elements, automatically escaping HTML characters.
- To insert user-provided content without risking XSS.
- To replace the entire content of an element with a plain text string.
- To compute new text values using a callback function.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter
$(selector).text();

// Setter
$(selector).text(textString);
$(selector).text(function(index, oldText) { ... });
```

**Component Breakdown**

- `$(selector).text()` : Returns the combined text content as a string.
- `textString` : The text to set as the content of each matched element. Numbers and Booleans are converted to strings.
- `function(index, oldText)` : A callback returning the text to set. `index` is the element's position in the set; `oldText` is the current text.
- Returns: A string (getter) or a jQuery object (setter).

**Syntax Rules**

1. `.text()` takes **no arguments** when used as a getter.
2. As a setter, it replaces the content of **all** matched elements with the provided text.
3. The setter escapes HTML characters so the content renders as literal text.
4. `.text()` can be used in both XML and HTML documents.
5. As of jQuery 1.4, the setter accepts a callback function.

**Constraints and Limitations**

- `.text()` should **not** be used on form inputs or scripts. Use `.val()` for input values and `.html()` for script content.
- The text returned by the getter may vary in whitespace and newlines across browsers due to differences in HTML parsers.
- Setting `.text()` destroys any existing child elements.

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Setting Text**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.text() — Read and Set</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p class="demo">This is <b>bold</b> and <i>italic</i> text.</p>
  <p class="demo">Another paragraph.</p>
  <div id="output"></div>

  <script>
    $(function () {
      // Step 1: GET — read the combined text of all .demo elements
      var combinedText = $(".demo").text();
      console.log("Combined text:", combinedText);
      // "This is bold and italic text.Another paragraph."
      // Note: tags are stripped; text is concatenated

      // Step 2: SET — replace the text of the first paragraph only
      $(".demo").first().text("New plain text content");
      console.log("After set:", $(".demo").first().text());
      // "New plain text content"

      // Step 3: SET with a callback — append original text
      $(".demo").last().text(function (index, oldText) {
        return oldText + " (index " + index + ")";
      });
      console.log("With callback:", $(".demo").last().text());
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Combined text: This is bold and italic text.Another paragraph.
After set: New plain text content
With callback: Another paragraph. (index 1)
```

**Why This Output Occurs**
The getter concatenates the text of both paragraphs, stripping the `<b>` and `<i>` tags. The setter replaces the first paragraph's content with a plain string, destroying the `<b>` and `<i>` elements. The callback version appends the index to the original text.

---

**Example 2: `.text()` Safely Escapes HTML**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.text() — HTML Escaping</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="container"></div>
  <div id="safe"></div>

  <script>
    $(function () {
      // Simulate untrusted user input containing HTML
      var userInput = '<img src=x onerror="alert(\'XSS\')">';

      // DANGEROUS: .html() would interpret this as markup and execute the script
      // $("#container").html(userInput);

      // SAFE: .text() escapes the angle brackets and treats it as literal text
      $("#safe").text(userInput);
      console.log("Safe content:", $("#safe").html());
      // "&lt;img src=x onerror="alert('XSS')"&gt;"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#safe` div displays the literal text `<img src=x onerror="alert('XSS')">`.
- No alert is triggered.
- Console output shows the escaped HTML entities.

**Why This Output Occurs**
`.text()` uses `document.createTextNode()`, which converts `<` to `&lt;` and `>` to `&gt;`. The browser renders these as visible characters rather than interpreting them as tags, preventing the script from executing.

### Real-World Cases

- **Displaying User Comments**: `$("#comment").text(userComment)` safely shows user input without executing scripts.
- **Updating Labels**: `$("label[for='email']").text("Email Address (required)")` changes label text.
- **Reading Content for Logic**: `if ($("#status").text() === "Active") { ... }` checks displayed status text.

### References

- jQuery API — .text() – https://api.jquery.com/text/
- W3Schools — jQuery text() Method – https://www.w3schools.com/jquery/html_text.asp

---

## Core Concept 2: `.html()` — Get or Set HTML Contents

### Definitions

**Core Definition**
`.html()` gets the HTML contents of the first element in the set of matched elements or sets the HTML contents of every matched element.

**Technical Definition**
When called without arguments, `.html()` returns the HTML content (including tags) of the **first** matched element, using the browser's `innerHTML` property. When called with a string argument, it sets the HTML content of **every** matched element by parsing the string as HTML and replacing the existing content. As of jQuery 1.4, the setter accepts a callback function. `.html()` is not available on XML documents.

**Beginner-Friendly Explanation**
`.html()` is like `.text()`, but it works with HTML code instead of just plain text. If you read it, you get the tags too. If you set it, jQuery treats whatever you provide as actual HTML — it creates elements, not just displays characters. This is powerful but also dangerous if the content comes from an untrusted source.

### Purposes

- To retrieve the full HTML markup (including tags) of the first matched element.
- To replace the content of elements with new HTML markup.
- To dynamically build complex DOM structures from HTML strings.
- To compute new HTML content using a callback function.
- To access the content of `<script>` elements (for which `.text()` should not be used).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter
$(selector).html();

// Setter
$(selector).html(htmlString);
$(selector).html(function(index, oldHtml) { ... });
```

**Component Breakdown**

- `$(selector).html()` : Returns the HTML content of the **first** matched element as a string.
- `htmlString` : An HTML string to set as the content of each matched element.
- `function(index, oldHtml)` : A callback returning the HTML to set. `index` is the element's position; `oldHtml` is the current HTML.
- Returns: A string (getter) or a jQuery object (setter).

**Syntax Rules**

1. As a getter, `.html()` returns only the content of the **first** matched element.
2. As a setter, it replaces the content of **all** matched elements.
3. The setter parses the string as HTML, creating new DOM elements.
4. `.html()` is not available on XML documents.
5. As of jQuery 1.4, the setter accepts a callback function.

**Constraints and Limitations**

- **Security Critical**: `.html()` does **not** escape HTML. Passing untrusted input directly to `.html()` creates an XSS vulnerability.
- The returned HTML may not exactly match the source due to browser `innerHTML` normalisation.
- Setting `.html()` destroys all existing child elements and their event handlers.

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Setting HTML**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.html() — Read and Set</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="demo-container">
    <div class="demo-box">Demonstration Box</div>
  </div>

  <script>
    $(function () {
      // Step 1: GET — retrieve the HTML content of the first match
      var htmlContent = $("div.demo-container").html();
      console.log("HTML content:", htmlContent);
      // "<div class="demo-box">Demonstration Box</div>"

      // Step 2: SET — replace the content with new HTML
      $("div.demo-container").html("<p>New <b>HTML</b> content</p>");
      console.log("After set:", $("div.demo-container").html());
      // "<p>New <b>HTML</b> content</p>"

      // Step 3: SET with a callback
      $("div.demo-container").html(function (index, oldHtml) {
        return "<strong>Replaced:</strong> " + oldHtml;
      });
      console.log("With callback:", $("div.demo-container").html());
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
HTML content: <div class="demo-box">Demonstration Box</div>
After set: <p>New <b>HTML</b> content</p>
With callback: <strong>Replaced:</strong> <p>New <b>HTML</b> content</p>
```

**Why This Output Occurs**
The getter returns the inner HTML of the first `.demo-container` div. The setter replaces that inner HTML with a new paragraph element. The callback receives the current HTML and returns modified markup.

---

**Example 2: The XSS Danger of `.html()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.html() — XSS Risk</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="vulnerable"></div>

  <script>
    $(function () {
      // Simulate untrusted user input
      var userInput = '<img src=x onerror="alert(\'XSS Attack!\')">';

      // DANGEROUS: .html() interprets the input as HTML
      // The onerror handler will execute when the image fails to load
      $("#vulnerable").html(userInput);

      console.log("Injected HTML:", $("#vulnerable").html());
      // The alert will fire in the browser
    });
  </script>
</body>
</html>
```

**Expected Output**
- An alert box displaying “XSS Attack!” appears in the browser.
- Console output shows the injected HTML markup.

**Why This Output Occurs**
`.html()` parses the input string as HTML, creating an `<img>` element with an `onerror` attribute. When the image source `x` fails to load, the `onerror` JavaScript executes. This demonstrates why `.html()` must never be used with untrusted input.

### Real-World Cases

- **Template Rendering**: `$("#result").html(compiledTemplate(data))` injects pre-sanitised HTML from a trusted template engine.
- **Rich Text Display**: Rendering HTML from a trusted CMS after server-side sanitisation.
- **Reading Script Content**: `$("script#config").html()` retrieves the text inside a script tag (where `.text()` should not be used).

### References

- jQuery API — .html() – https://api.jquery.com/html/
- jQuery API — Security Notes – https://api.jquery.com/html/#html-htmlString
- W3Schools — jQuery html() Method – https://www.w3schools.com/jquery/html_html.asp

---

## Core Concept 3: `.val()` — Get or Set Form Element Values

### Definitions

**Core Definition**
`.val()` gets the current value of the first element in the set of matched elements or sets the value of every matched element, and is primarily designed for form controls.

**Technical Definition**
The `.val()` method reads and writes the `value` property of form elements: `<input>`, `<select>`, and `<textarea>`. When called on a `<select multiple>` element, it returns an array of selected values. As of jQuery 3.0, if no options are selected in a multi-select, it returns an empty array. For checkboxes and radio buttons, it returns the value of the checked element. When used as a setter, it accepts a string, number, or array, and as of jQuery 1.4, a callback function.

**Beginner-Friendly Explanation**
`.val()` is the tool for working with form fields. If you need to know what a user typed into a text box, or which option they chose in a dropdown, you use `.val()`. If you want to fill in a form field programmatically, you also use `.val()`. It does not work on regular elements like `<div>` or `<p>` — those use `.text()` or `.html()`.

### Purposes

- To read the current value of a form field (text input, select, textarea).
- To programmatically set the value of form fields.
- To retrieve an array of values from a multiple-select element.
- To reset or clear form fields.
- To read checkbox and radio button values based on their checked state.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter
$(selector).val();

// Setter
$(selector).val(value);
$(selector).val(function(index, oldValue) { ... });
```

**Component Breakdown**

- `$(selector).val()` : Returns the value of the **first** matched element. For multi-selects, returns an array.
- `value` : A string, number, or array to set as the value.
- `function(index, oldValue)` : A callback returning the value to set.
- Returns: A string, number, or array (getter) or a jQuery object (setter).

**Syntax Rules**

1. `.val()` is used **only** on form elements; it does not work on `<div>`, `<span>`, etc.
2. As a getter, it returns the value of the **first** matched element.
3. As a setter, it sets the value of **all** matched elements.
4. On a `<select multiple>`, the getter returns an array of selected values.
5. As of jQuery 3.0, `.val()` on an empty collection returns `undefined`; on a multi-select with no selections, it returns `[]`.

**Constraints and Limitations**

- Using `.val()` on non-form elements returns `undefined` or empty results.
- For `<textarea>`, `.val()` strips carriage return characters from the browser-reported value.
- `.val()` does not trigger change events; use `.trigger("change")` if needed.

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Setting Form Values**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.val() — Read and Set Form Values</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="text" id="name" value="John">
  <select id="single">
    <option>Single</option>
    <option>Single2</option>
  </select>
  <select id="multiple" multiple="multiple">
    <option selected="selected">Multiple</option>
    <option>Multiple2</option>
    <option selected="selected">Multiple3</option>
  </select>
  <button id="btn">Set Values</button>

  <script>
    $(function () {
      // Step 1: GET — read values
      console.log("Text input:", $("#name").val());          // "John"
      console.log("Single select:", $("#single").val());     // "Single"

      // Step 2: GET — multi-select returns an array
      var multiVals = $("#multiple").val();
      console.log("Multi-select:", multiVals);               // ["Multiple", "Multiple3"]

      // Step 3: SET — change values programmatically
      $("#btn").on("click", function () {
        $("#name").val("Jane");
        $("#single").val("Single2");
        $("#multiple").val(["Multiple2"]);  // Set a single selection in multi-select
        console.log("After set — name:", $("#name").val()); // "Jane"
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Text input: John
Single select: Single
Multi-select: ["Multiple", "Multiple3"]
After set — name: Jane
```

**Why This Output Occurs**
`.val()` reads the `value` property of each form control. For the multi-select, it returns an array of all selected option values. The setter updates the underlying `value` property, and the input field's displayed value changes accordingly.

---

**Example 2: Checkboxes and Radios with `.val()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.val() — Checkboxes and Radios</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <input type="checkbox" name="fruit" value="apple" checked> Apple
  <input type="checkbox" name="fruit" value="banana"> Banana
  <input type="radio" name="color" value="red" checked> Red
  <input type="radio" name="color" value="blue"> Blue

  <script>
    $(function () {
      // Step 1: Get the value of the checked checkbox
      var checkedFruit = $("input[name='fruit']:checked").val();
      console.log("Checked fruit:", checkedFruit); // "apple"

      // Step 2: Get the value of the checked radio button
      var checkedColor = $("input[name='color']:checked").val();
      console.log("Checked color:", checkedColor); // "red"

      // Step 3: Check a different radio and read again
      $("input[name='color'][value='blue']").prop("checked", true);
      console.log("New checked color:",
        $("input[name='color']:checked").val()); // "blue"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Checked fruit: apple
Checked color: red
New checked color: blue
```

**Why This Output Occurs**
`.val()` on a checkbox or radio returns the `value` attribute of the element. By combining with the `:checked` selector, you retrieve the value of the currently selected control. Changing the `checked` property and re-querying returns the new value.

### Real-World Cases

- **Form Reset**: `$("form input").val("")` clears all text inputs.
- **Form Population**: `$("#email").val(user.email)` fills a form field with data from an API.
- **Multi-Select Processing**: `$("#tags").val()` retrieves all selected tags as an array.
- **Conditional Logic**: `if ($("#country").val() === "US") { showStateField(); }`.

### References

- jQuery API — .val() – https://api.jquery.com/val/
- jQuery API — Forms – https://api.jquery.com/category/forms/
- W3Schools — jQuery val() Method – https://www.w3schools.com/jquery/html_val.asp

---

## Core Concept 4: Reading vs. Replacing Element Content

### Definitions

**Core Definition**
Reading content means retrieving the current text, HTML, or value from an element without changing it; replacing content means overwriting the existing content of one or more elements with new content.

**Technical Definition**
jQuery's content methods follow the **getter/setter** pattern. Called without arguments, they read the current state. Called with an argument, they write new content, **destroying** any existing child nodes, text nodes, and event handlers attached to those children. `.text()` and `.html()` replace the **inner content** of the element (the element itself remains), while `.replaceWith()` and `.remove()` replace or remove the element itself.

**Beginner-Friendly Explanation**
Reading is asking “what is inside this element right now?” Replacing is saying “throw away what is inside and put this new thing there instead.” When you replace, anything that was inside — including child elements and their behaviours — is gone. The container element itself stays; only its contents change.

### Purposes

- To retrieve current content for validation, comparison, or display in another location.
- To update page content dynamically based on user interaction or data changes.
- To reset or clear content before inserting new data.
- To build dynamic interfaces where content changes without a page reload.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// READING
var textContent = $(selector).text();   // returns String
var htmlContent = $(selector).html();   // returns String
var formValue   = $(selector).val();    // returns String/Number/Array

// REPLACING
$(selector).text(newText);              // replaces ALL matched elements
$(selector).html(newHtml);              // replaces ALL matched elements
$(selector).val(newValue);              // replaces ALL matched elements
```

**Component Breakdown**

- **Reading** uses the method with no arguments; the return type depends on the method.
- **Replacing** uses the method with an argument; the return value is the jQuery object.
- All three setter methods accept a callback function for computed replacements.

**Syntax Rules**

1. Reading methods return values; they break the jQuery chain.
2. Replacing methods return the jQuery object; they preserve the chain.
3. Setters replace the content of **all** matched elements.
4. Replacing content destroys existing child elements and their event handlers.

**Constraints and Limitations**

- Reading `.html()` returns only the first element's content; `.text()` returns all.
- Replacing content removes event handlers on child elements unless delegated events are used.
- Replacing does not affect the matched element itself, only its inner content.

### Multiple Annotated Complete Code Examples

**Example 1: Reading vs. Replacing**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Reading vs Replacing</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="content">
    <p>Original <b>bold</b> text</p>
  </div>

  <script>
    $(function () {
      // READING — retrieve content without changing it
      var htmlRead = $("#content").html();
      var textRead = $("#content").text();
      console.log("Read HTML:", htmlRead);
      // "<p>Original <b>bold</b> text</p>"
      console.log("Read text:", textRead);
      // "Original bold text"

      // REPLACING — overwrite the content
      $("#content").html("<p>New <i>italic</i> content</p>");
      console.log("After replace HTML:", $("#content").html());
      // "<p>New <i>italic</i> content</p>"

      // The original <p> and <b> elements are destroyed
      console.log("Bold count:", $("#content b").length); // 0
      console.log("Italic count:", $("#content i").length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Read HTML: <p>Original <b>bold</b> text</p>
Read text: Original bold text
After replace HTML: <p>New <i>italic</i> content</p>
Bold count: 0
Italic count: 1
```

**Why This Output Occurs**
Reading retrieves the current content. Replacing with `.html()` destroys the original `<p>` and `<b>` elements and creates a new `<p>` with an `<i>` element. The counts confirm that the original `<b>` is gone and the new `<i>` exists.

### Real-World Cases

- **Live Search**: Reading input values on each keystroke and replacing result container content.
- **Form Validation**: Reading field values and replacing error message content.
- **Content Editors**: Reading the current HTML, modifying it, and replacing it.
- **Chat Applications**: Reading message input values and replacing chat container content with new messages.

### References

- jQuery API — Manipulation – https://api.jquery.com/category/manipulation/
- jQuery API — DOM Replacement – https://api.jquery.com/category/manipulation/dom-replacement/
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/

---

## Core Concept 5: Security Considerations of `.html()` (Cross-Site Scripting / XSS Risks)

### Definitions

**Core Definition**
Cross-Site Scripting (XSS) is a security vulnerability in which an attacker injects malicious scripts into content that is then executed by a victim's browser; jQuery's `.html()` method is a common **sink** for such attacks when used with untrusted input.

**Technical Definition**
A **sink** is a function or method that can execute or render untrusted data in a dangerous context. jQuery's `.html()`, `.append()`, `.before()`, `.after()`, `.prepend()`, and `$()` constructors are all XSS sinks because they parse their arguments as HTML. If the HTML string contains `<script>` tags, `onerror` attributes, `javascript:` URLs, or other executable content, that code runs in the context of the user's session. CVE-2020-11022 and CVE-2020-11023 affected jQuery versions before 3.5.0, where even sanitised HTML containing `<option>` elements could execute untrusted code.

**Beginner-Friendly Explanation**
XSS is like someone slipping a dangerous note into a pile of papers you are about to read aloud. If you use `.html()` to put user input onto the page, an attacker can include a hidden script that steals passwords, cookies, or does other harmful things. The `.text()` method is safe because it treats everything as plain words, never as code. The `.html()` method is dangerous because it treats everything as code.

### Purposes

- To understand why `.html()` must never be used with untrusted input.
- To recognise the difference between safe sinks (`.text()`, `.val()`) and dangerous sinks (`.html()`, `.append()`).
- To learn mitigation strategies such as DOMPurify and jQuery's built-in fixes.
- To identify when a jQuery version is vulnerable to known XSS CVEs.
- To apply defence-in-depth by sanitising on both the server and the client.

### Syntax Rules and Structure

**Safe vs. Dangerous Sinks**

| Method | XSS Risk | Reason |
|---|---|---|
| `.text(value)` | Safe | Escapes HTML; creates text nodes |
| `.val(value)` | Safe | Sets property value; does not parse HTML |
| `.html(value)` | **Dangerous** | Parses value as HTML; executes scripts |
| `.append(value)` | **Dangerous** | Parses value as HTML |
| `.prepend(value)` | **Dangerous** | Parses value as HTML |
| `.before(value)` | **Dangerous** | Parses value as HTML |
| `.after(value)` | **Dangerous** | Parses value as HTML |

**Mitigation Syntax**

```javascript
// SAFE: Use .text() for untrusted content
$(selector).text(untrustedInput);

// SAFE: Sanitise with DOMPurify before .html()
var clean = DOMPurify.sanitize(untrustedInput, { SAFE_FOR_JQUERY: true });
$(selector).html(clean);

// SAFE: Parse and filter with $.parseHTML()
var safeNodes = $.parseHTML(untrustedInput);
$(selector).empty().append(safeNodes);
```

**Syntax Rules**

1. Never pass untrusted input directly to `.html()`, `.append()`, `.prepend()`, `.before()`, or `.after()`.
2. Use `.text()` for any content that should be displayed as plain text.
3. If HTML is required, sanitise it with a library like DOMPurify using the `SAFE_FOR_JQUERY` option.
4. Upgrade to jQuery 3.5.0 or later to mitigate CVE-2020-11022 and CVE-2020-11023.
5. Server-side sanitisation is not sufficient on its own; client-side sanitisation is also required.

**Constraints and Limitations**

- jQuery versions before 3.5.0 are vulnerable to XSS even with sanitised HTML containing `<option>` elements.
- No client-side sanitiser is perfect; defence-in-depth is essential.
- `.text()` cannot be used when actual HTML rendering is required.

### Multiple Annotated Complete Code Examples

**Example 1: Dangerous vs. Safe Content Insertion**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>XSS — Dangerous vs Safe</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="dangerous"></div>
  <div id="safe"></div>

  <script>
    $(function () {
      // Simulated untrusted user input with an XSS payload
      var userInput = '<img src=x onerror="alert(\'XSS!\')">';

      // DANGEROUS: .html() executes the onerror handler
      // $("#dangerous").html(userInput);  // Uncomment to see the alert

      // SAFE: .text() escapes the HTML and displays it as literal text
      $("#safe").text(userInput);
      console.log("Safe output HTML:", $("#safe").html());
      // "&lt;img src=x onerror="alert('XSS!')"&gt;"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#safe` div displays the literal text `<img src=x onerror="alert('XSS!')">`.
- No alert appears.
- Console output shows the escaped entities.

**Why This Output Occurs**
`.text()` escapes `<` and `>` to `&lt;` and `&gt;`, so the browser renders the string as visible text rather than parsing it as an HTML element. The `onerror` handler never runs because no `<img>` element is created.

---

**Example 2: Sanitising with DOMPurify**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>XSS — DOMPurify Sanitisation</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/dompurify/3.0.0/purify.min.js"></script>
</head>
<body>
  <div id="output"></div>

  <script>
    $(function () {
      // Untrusted input with a script tag
      var userInput = '<p>Hello</p><script>alert("XSS")<\/script>';

      // SANITISE with DOMPurify using SAFE_FOR_JQUERY
      var clean = DOMPurify.sanitize(userInput, {
        SAFE_FOR_JQUERY: true
      });

      // Now it is safe to use .html()
      $("#output").html(clean);

      console.log("Sanitised output:", $("#output").html());
      // "<p>Hello</p>" — the script tag is removed
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#output` div displays “Hello” without executing any script.
- Console output shows the `<script>` tag has been removed.

**Why This Output Occurs**
DOMPurify with the `SAFE_FOR_JQUERY` option removes dangerous elements and attributes, including `<script>` tags and event handlers. The sanitised HTML is safe to pass to `.html()`.

### Real-World Cases

- **User Comments**: Always use `.text()` when displaying user-submitted comments.
- **Rich Text Editors**: Sanitise HTML output from rich text editors with DOMPurify before rendering.
- **URL Parameters**: Never pass `location.search` or `location.hash` directly to `.html()`.
- **AJAX Responses**: Sanitise HTML returned from external APIs before injecting it into the DOM.

### References

- jQuery API — Security Notes on .html() – https://api.jquery.com/html/#html-htmlString
- CVE-2020-11022 — NVD – https://nvd.nist.gov/vuln/detail/CVE-2020-11022
- CVE-2020-11023 — NVD – https://nvd.nist.gov/vuln/detail/CVE-2020-11023
- jQuery 3.5.0 Upgrade Guide – https://jquery.com/upgrade-guide/3.5/
- DOMPurify — GitHub – https://github.com/cure53/DOMPurify
- OWASP — Cross Site Scripting (XSS) – https://owasp.org/www-community/attacks/xss/
- OWASP — DOM based XSS Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- Snyk — jQuery XSS Vulnerability – https://security.snyk.io/package/npm/jquery

---

## Summary Table: Reading and Modifying Content at a Glance

| Method | Target | Getter Scope | Safe? | Escapes HTML? |
|---|---|---|---|---|
| `.text()` | Element content | All matched elements | Safe | Yes |
| `.html()` | Element HTML | First matched element | **Dangerous** | No |
| `.val()` | Form values | First matched element | Safe | N/A (property) |

### Key Differences

| Aspect | `.text()` | `.html()` | `.val()` |
|---|---|---|---|
| What it gets | Combined text | HTML of first element | Value of first form field |
| What it sets | Text content | HTML content | Form field value |
| HTML escaping | Yes (safe) | No (dangerous) | N/A |
| Works on forms | No | No | Yes |
| Returns array | No | No | Yes (multi-select) |

---

## General References

- jQuery API — .text() – https://api.jquery.com/text/
- jQuery API — .html() – https://api.jquery.com/html/
- jQuery API — .val() – https://api.jquery.com/val/
- jQuery API — Manipulation – https://api.jquery.com/category/manipulation/
- jQuery API — Security Notes – https://api.jquery.com/html/#html-htmlString
- jQuery 3.5.0 Upgrade Guide – https://jquery.com/upgrade-guide/3.5/
- W3Schools — jQuery Set Content and Attributes – https://www.w3schools.com/jquery/jquery_dom_set.asp
- OWASP — Cross Site Scripting (XSS) – https://owasp.org/www-community/attacks/xss/
- OWASP — DOM based XSS Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- DOMPurify — GitHub – https://github.com/cure53/DOMPurify
- Learning jQuery 3 (Fifth Edition) — O'Reilly – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/