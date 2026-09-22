# jQuery Inserting Elements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery insertion methods are a family of functions that place new content — elements, text, or HTML strings — into the DOM relative to a set of target elements, either **inside** the target (as a child) or **outside** it (as a sibling).

**Technical Definition**
jQuery's DOM insertion methods are organised into two categories based on the relationship between the inserted content and the target. **Inside placement** methods (`.append()`, `.appendTo()`, `.prepend()`, `.prependTo()`) insert content as a child of the target element, either at the end or the beginning of its existing children. **Outside placement** methods (`.before()`, `.insertBefore()`, `.after()`, `.insertAfter()`) insert content as a sibling of the target element, either before or after it in the parent's child list. Each method exists in two syntactic forms: a "content-first" form where the target precedes the method and the content is passed as an argument, and a "target-first" form where the content precedes the method and the target is passed as an argument.

**Beginner-Friendly Explanation**
When you want to add something new to a web page, you need to decide *where* it should go. Should it go **inside** an existing element — like adding a new item to the end of a list? Or should it go **outside** — like putting a new paragraph right before or after another one? jQuery gives you eight methods split into two groups: four for putting content inside a container, and four for putting content next to an element. Each pair does the same thing but reads in the opposite direction, so you can choose whichever is more natural for your code.

### Key Characteristics

- **Two Categories**: Inside placement (child relationships) vs. outside placement (sibling relationships).
- **Paired Methods**: Each operation has two equivalent methods with reversed syntax (e.g., `.append()` and `.appendTo()`).
- **Flexible Content**: Content can be a DOM element, text node, array of elements, HTML string, or jQuery object.
- **Chainable**: All methods return a jQuery object, enabling method chaining.
- **Move vs. Clone**: If the content is an existing element and there is a single target, it is **moved** (not cloned). With multiple targets, the element is **cloned** for all but the last target.
- **Function Support**: Since jQuery 1.4, all methods accept a function that returns the content to insert.
- **Script Execution Risk**: Methods that accept HTML strings can execute embedded scripts, creating an XSS risk with untrusted input.

### Prerequisites

- Basic understanding of the DOM tree (parent-child and sibling relationships).
- Familiarity with jQuery selectors and the `$()` function.
- Awareness of how to create elements dynamically with `$("<tag>")`.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **DOM Manipulation**: Insertion is the core of dynamic content generation.
- **Event Handling**: Events can be attached to inserted elements before or after insertion.
- **AJAX**: Server responses are often parsed and inserted into the page.
- **Templating**: Client-side templates produce HTML strings that are inserted with these methods.
- **Animation**: Elements are often created and inserted as part of animation sequences.

### Core Concepts / Features

1. Inside Placement: `.append()` & `.appendTo()`
2. Inside Placement: `.prepend()` & `.prependTo()`
3. Outside Placement: `.before()` & `.insertBefore()`
4. Outside Placement: `.after()` & `.insertAfter()`
5. Inside vs. Outside Placement: Key Differences

---

## Core Concept 1: `.append()` & `.appendTo()` — Insert at the End of the Target

### Definitions

**Core Definition**
`.append()` inserts content at the end of each target element's children; `.appendTo()` inserts the matched elements at the end of the target.

**Technical Definition**
The `.append()` method inserts the specified content as the **last child** of each element in the jQuery collection. The `.appendTo()` method performs the same task but reverses the syntax: the content precedes the method, and the target is passed as an argument. With `.append()`, the selector expression preceding the method is the **container** into which the content is inserted. With `.appendTo()`, the content precedes the method and is inserted into the **target container**.

**Beginner-Friendly Explanation**
`.append()` means "add this to the end of that." If you have a `<ul>` and you want to add a new `<li>` to the bottom of the list, you use `$("ul").append("<li>New item</li>")`. `.appendTo()` does the same thing but reads in the opposite direction: `$("<li>New item</li>").appendTo("ul")` means "take this new list item and put it at the end of the list." Both produce the same result — the choice is about which reads more naturally in your code.

### Purposes

- To add new content as the last child of one or more target elements.
- To move an existing element from one location to the end of another.
- To insert HTML strings, DOM elements, or jQuery objects at the end of a container.
- To append multiple pieces of content in a single call.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// .append() — target first, content as argument
$(target).append(content);
$(target).append(content1, content2, ...);
$(target).append(function(index, oldHtml) { ... });

// .appendTo() — content first, target as argument
$(content).appendTo(target);
```

**Component Breakdown**

- `.append(content)` : Inserts `content` at the end of each matched element.
- `.appendTo(target)` : Inserts the matched set at the end of `target`.
- `content` : DOM element, text node, array of elements, HTML string, or jQuery object.
- `target` : A selector, element, HTML string, array of elements, or jQuery object.
- Returns: A jQuery object (the original set for `.append()`, the inserted set for `.appendTo()`).

**Syntax Rules**

1. `.append()` and `.appendTo()` perform the same task; the difference is the placement of content and target.
2. `.append()` inserts the content as the **last child** of each target.
3. If the content is an existing element and there is one target, it is **moved** (not cloned).
4. With multiple targets, clones are created for each target except the last.
5. Since jQuery 1.4, a function can be passed to compute the content to insert.

**Constraints and Limitations**

- Inserting HTML strings from untrusted sources creates an XSS vulnerability.
- `.appendTo()` returns the inserted set, not the target; this affects chain behaviour.

### Multiple Annotated Complete Code Examples

**Example 1: Appending to Multiple Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.append() — Multiple Targets</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Append the same content to all .inner divs
      $(".inner").append("<p>Test</p>");

      // Step 2: Each .inner div now has the new <p> as its last child
      console.log("Test paragraphs:", $(".inner p").length); // 2

      // Step 3: Verify the structure
      $(".inner").each(function () {
        console.log($(this).html());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Both `.inner` divs receive a `<p>Test</p>` element at the end of their content.
- Console output shows two test paragraphs and the updated HTML of each div.

**Why This Output Occurs**
`.append("<p>Test</p>")` inserts the HTML string as the last child of each matched `.inner` div. Both divs receive the content.

---

**Example 2: Moving an Existing Element**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.append() — Moving an Element</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <h2>Greetings</h2>
  <div class="container">
    <div class="inner">Hello</div>
  </div>

  <script>
    $(function () {
      // Step 1: Move the <h2> into the container
      $(".container").append($("h2"));

      // Step 2: Verify the <h2> is now inside the container
      console.log("H2 in container:", $(".container h2").length); // 1

      // Step 3: The <h2> is moved, not cloned
      console.log("Total h2 elements:", $("h2").length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `<h2>` element is moved from before `.container` to inside it.
- Console output confirms the move (still only one `<h2>`).

**Why This Output Occurs**
`.append($("h2"))` takes the existing `<h2>` element and moves it into the container. Because there is only one target, the element is moved rather than cloned.

### Real-World Cases

- **Todo Lists**: Appending a new `<li>` to the end of a list when a user adds a task.
- **Chat Applications**: Appending a new message `<div>` to the end of a conversation.
- **Data Tables**: Appending a new `<tr>` to the end of a table body after an AJAX response.
- **Dynamic Forms**: Appending a new input field to the end of a form section.

### References

- jQuery API — .append() – https://api.jquery.com/append/
- jQuery API — .appendTo() – https://api.jquery.com/appendTo/
- jQuery API — DOM Insertion, Inside – https://api.jquery.com/category/manipulation/dom-insertion-inside/

---

## Core Concept 2: `.prepend()` & `.prependTo()` — Insert at the Beginning of the Target

### Definitions

**Core Definition**
`.prepend()` inserts content at the beginning of each target element's children; `.prependTo()` inserts the matched elements at the beginning of the target.

**Technical Definition**
The `.prepend()` method inserts the specified content as the **first child** of each element in the jQuery collection. The `.prependTo()` method performs the same task but reverses the syntax, with the content preceding the method and the target passed as an argument. With `.prepend()`, the selector expression preceding the method is the container. With `.prependTo()`, the content precedes the method and is inserted into the target container.

**Beginner-Friendly Explanation**
`.prepend()` means "add this to the beginning of that." If you have a list and you want to add a new item to the top, you use `$("ul").prepend("<li>First item</li>")`. `.prependTo()` reads in the opposite direction: `$("<li>First item</li>").prependTo("ul")` means "take this new list item and put it at the top of the list."

### Purposes

- To add new content as the first child of one or more target elements.
- To insert a new item at the beginning of a list, container, or section.
- To move an existing element to the beginning of another.
- To prepend multiple pieces of content in a single call.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// .prepend() — target first, content as argument
$(target).prepend(content);
$(target).prepend(content1, content2, ...);
$(target).prepend(function(index, oldHtml) { ... });

// .prependTo() — content first, target as argument
$(content).prependTo(target);
```

**Component Breakdown**

- `.prepend(content)` : Inserts `content` as the first child of each matched element.
- `.prependTo(target)` : Inserts the matched set as the first child of `target`.
- Returns: A jQuery object.

**Syntax Rules**

1. `.prepend()` and `.prependTo()` perform the same task; the difference is the placement of content and target.
2. `.prepend()` inserts the content as the **first child** of each target.
3. Move vs. clone behaviour is the same as `.append()`: single target moves, multiple targets clone for all but the last.
4. Since jQuery 1.4, a function can be passed to compute the content.

**Constraints and Limitations**

- Inserting untrusted HTML creates an XSS risk.
- The returned set differs between the two forms.

### Multiple Annotated Complete Code Examples

**Example 1: Prepending to Multiple Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.prepend() — Multiple Targets</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Prepend content to all .inner divs
      $(".inner").prepend("<p>Test</p>");

      // Step 2: Each .inner div now has the new <p> as its first child
      console.log("Test paragraphs:", $(".inner p").length); // 2

      // Step 3: Verify the first child is the new paragraph
      $(".inner").each(function () {
        console.log("First child:", $(this).children().first().text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Both `.inner` divs receive a `<p>Test</p>` element at the beginning of their content.
- Console output shows “First child: Test” for each div.

**Why This Output Occurs**
`.prepend("<p>Test</p>")` inserts the HTML string as the first child of each matched `.inner` div. The existing content (“Hello” and “Goodbye”) is pushed after the new paragraph.

### Real-World Cases

- **Notifications**: Prepending a new notification to the top of a notification list.
- **Comments**: Prepending the newest comment to the top of a comment thread.
- **Activity Feeds**: Adding the latest activity to the beginning of a feed.
- **Breadcrumb Navigation**: Prepending a new step to the beginning of a breadcrumb trail.

### References

- jQuery API — .prepend() – https://api.jquery.com/prepend/
- jQuery API — .prependTo() – https://api.jquery.com/prependTo/
- jQuery API — DOM Insertion, Inside – https://api.jquery.com/category/manipulation/dom-insertion-inside/

---

## Core Concept 3: `.before()` & `.insertBefore()` — Insert Before the Target

### Definitions

**Core Definition**
`.before()` inserts content before each target element as a sibling; `.insertBefore()` inserts the matched elements before the target.

**Technical Definition**
The `.before()` method inserts the specified content **before** each element in the set of matched elements, as a sibling in the same parent. The `.insertBefore()` method performs the same task but reverses the syntax: the content precedes the method and the target is passed as an argument. With `.before()`, the content to be inserted comes from the method's argument: `$(target).before(contentToBeInserted)`. With `.insertBefore()`, the content precedes the method and is inserted before the target: `$(contentToBeInserted).insertBefore(target)`.

**Beginner-Friendly Explanation**
`.before()` means "put this right before that." If you have a paragraph and you want to add a new paragraph just before it, you use `$("p").before("<p>New</p>")`. `.insertBefore()` reads in the opposite direction: `$("<p>New</p>").insertBefore("p")` means "take this new paragraph and put it before the existing paragraph." The new element becomes a sibling of the target, not a child.

### Purposes

- To insert content immediately before a target element in the same parent.
- To add a label or heading before a form field or section.
- To insert a new sibling element at a specific position.
- To move an existing element to a position before another.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// .before() — target first, content as argument
$(target).before(content);
$(target).before(content1, content2, ...);
$(target).before(function(index) { ... });

// .insertBefore() — content first, target as argument
$(content).insertBefore(target);
```

**Component Breakdown**

- `.before(content)` : Inserts `content` before each matched element.
- `.insertBefore(target)` : Inserts the matched set before `target`.
- Returns: A jQuery object.

**Syntax Rules**

1. `.before()` and `.insertBefore()` perform the same task; the difference is the placement of content and target.
2. The inserted content becomes a **sibling** of the target, not a child.
3. Move vs. clone behaviour applies: single target moves, multiple targets clone.
4. Since jQuery 1.4, a function can be passed to compute the content.

**Constraints and Limitations**

- The target must have a parent; inserting before the `<html>` element is not possible.
- Inserting untrusted HTML creates an XSS risk.

### Multiple Annotated Complete Code Examples

**Example 1: Inserting Before Multiple Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.before() — Multiple Targets</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Insert a paragraph before each .inner div
      $(".inner").before("<p>Test</p>");

      // Step 2: Each .inner div now has a <p> before it as a sibling
      console.log("Test paragraphs:", $(".container > p").length); // 2

      // Step 3: Verify the structure — the <p> is a sibling, not a child
      console.log("Inner children:", $(".inner").first().children().length); // 0
    });
  </script>
</body>
</html>
```

**Expected Output**
- Two `<p>Test</p>` elements are inserted before each `.inner` div, as siblings.
- Console output confirms the paragraphs are siblings (inside `.container`, not inside `.inner`).

**Why This Output Occurs**
`.before("<p>Test</p>")` inserts the HTML string as a sibling immediately preceding each `.inner` div. The new paragraphs are children of `.container`, not of `.inner`.

### Real-World Cases

- **Form Labels**: Inserting a `<label>` before an `<input>` field.
- **Section Headings**: Inserting a heading before a content block.
- **Navigation Items**: Inserting a new link before an existing navigation item.
- **Dividers**: Inserting an `<hr>` before a section for visual separation.

### References

- jQuery API — .before() – https://api.jquery.com/before/
- jQuery API — .insertBefore() – https://api.jquery.com/insertBefore/
- jQuery API — DOM Insertion, Outside – https://api.jquery.com/category/manipulation/dom-insertion-outside/

---

## Core Concept 4: `.after()` & `.insertAfter()` — Insert After the Target

### Definitions

**Core Definition**
`.after()` inserts content after each target element as a sibling; `.insertAfter()` inserts the matched elements after the target.

**Technical Definition**
The `.after()` method inserts the specified content **after** each element in the set of matched elements, as a sibling in the same parent. The `.insertAfter()` method performs the same task but reverses the syntax: the content precedes the method and the target is passed as an argument. With `.after()`, the content comes from the method's argument: `$(target).after(contentToBeInserted)`. With `.insertAfter()`, the content precedes the method and is inserted after the target: `$(contentToBeInserted).insertAfter(target)`.

**Beginner-Friendly Explanation**
`.after()` means "put this right after that." If you have a paragraph and you want to add a new paragraph just after it, you use `$("p").after("<p>New</p>")`. `.insertAfter()` reads in the opposite direction: `$("<p>New</p>").insertAfter("p")` means "take this new paragraph and put it after the existing paragraph."

### Purposes

- To insert content immediately after a target element in the same parent.
- To add a note or supplementary information after a form field.
- To insert a new sibling element at a specific position.
- To move an existing element to a position after another.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// .after() — target first, content as argument
$(target).after(content);
$(target).after(content1, content2, ...);
$(target).after(function(index, oldHtml) { ... });

// .insertAfter() — content first, target as argument
$(content).insertAfter(target);
```

**Component Breakdown**

- `.after(content)` : Inserts `content` after each matched element.
- `.insertAfter(target)` : Inserts the matched set after `target`.
- Returns: A jQuery object.

**Syntax Rules**

1. `.after()` and `.insertAfter()` perform the same task; the difference is the placement of content and target.
2. The inserted content becomes a **sibling** of the target, not a child.
3. Move vs. clone behaviour applies: single target moves, multiple targets clone.
4. Since jQuery 1.4, a function can be passed to compute the content.

**Constraints and Limitations**

- The target must have a parent.
- Inserting untrusted HTML creates an XSS risk.

### Multiple Annotated Complete Code Examples

**Example 1: Inserting After Multiple Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.after() — Multiple Targets</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Insert a paragraph after each .inner div
      $(".inner").after("<p>Test</p>");

      // Step 2: Each .inner div now has a <p> after it as a sibling
      console.log("Test paragraphs:", $(".container > p").length); // 2

      // Step 3: Verify the <p> is a sibling, not a child
      console.log("Inner children:", $(".inner").first().children().length); // 0
    });
  </script>
</body>
</html>
```

**Expected Output**
- Two `<p>Test</p>` elements are inserted after each `.inner` div, as siblings.
- Console output confirms the paragraphs are siblings.

**Why This Output Occurs**
`.after("<p>Test</p>")` inserts the HTML string as a sibling immediately following each `.inner` div.

### Real-World Cases

- **Error Messages**: Inserting an error message after an invalid form field.
- **Help Text**: Adding help text after a label or input.
- **Dividers**: Inserting an `<hr>` after a section for visual separation.
- **Action Buttons**: Adding a button after a form field for submission.

### References

- jQuery API — .after() – https://api.jquery.com/after/
- jQuery API — .insertAfter() – https://api.jquery.com/insertAfter/
- jQuery API — DOM Insertion, Outside – https://api.jquery.com/category/manipulation/dom-insertion-outside/

---

## Core Concept 5: Inside vs. Outside Placement — Key Differences

### Definitions

**Core Definition**
The fundamental difference is that **inside** placement methods insert content as a **child** of the target, while **outside** placement methods insert content as a **sibling** of the target.

**Technical Definition**
Inside placement methods (`.append()`, `.appendTo()`, `.prepend()`, `.prependTo()`) modify the **innerHTML** of the target element, inserting content into its child node list. Outside placement methods (`.before()`, `.insertBefore()`, `.after()`, `.insertAfter()`) insert content into the **parent's child node list** at a position adjacent to the target, without modifying the target's own children. This distinction determines the DOM structure, CSS inheritance, event bubbling behaviour, and the result of subsequent traversal operations.

**Beginner-Friendly Explanation**
Think of an element as a box. **Inside placement** means putting something **into** the box — it becomes one of the box's contents. **Outside placement** means putting something **next to** the box — it stays on the shelf beside the box, not inside it. If you later move the box, the inside content moves with it, but the outside content stays where it is.

### Purposes

- To choose the correct DOM structure for the intended design.
- To understand how CSS selectors (descendant vs. child) will match the inserted content.
- To predict event bubbling behaviour (events on inside content bubble to the target; events on outside content do not).
- To select the appropriate method for a given insertion task.

### Syntax Rules and Structure

**Comparison Table**

| Aspect | Inside Placement | Outside Placement |
|---|---|---|
| Methods | `.append()`, `.appendTo()`, `.prepend()`, `.prependTo()` | `.before()`, `.insertBefore()`, `.after()`, `.insertAfter()` |
| Relationship | Content becomes a **child** of the target | Content becomes a **sibling** of the target |
| DOM modification | Target's `innerHTML` is modified | Parent's child list is modified |
| CSS descendant selector | Matches the inserted content | Does not match |
| Event bubbling | Events bubble to the target | Events do not bubble to the target |
| Traversal from target | `.children()` finds the content | `.siblings()` finds the content |
| If target is moved | Content moves with it | Content stays in place |

**Syntax Rules**

1. Inside methods are used when the content logically belongs **within** the target (e.g., list items inside a list).
2. Outside methods are used when the content should be a **peer** of the target (e.g., a label before an input).
3. The paired syntax forms (`.append()` vs. `.appendTo()`) do not change the DOM result, only the code readability.

**Constraints and Limitations**

- Inside placement cannot insert content before or after the target within the same parent.
- Outside placement cannot insert content as the first or last child of the target.

### Multiple Annotated Complete Code Examples

**Example 1: Inside vs. Outside — Structural Comparison**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inside vs Outside Placement</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="inside-target">Target</div>
  <div id="outside-target">Target</div>

  <script>
    $(function () {
      // --- INSIDE: Append content as a child ---
      $("#inside-target").append("<span>Inside content</span>");

      // --- OUTSIDE: Insert content as a sibling ---
      $("#outside-target").after("<span>Outside content</span>");

      // Step 1: Check the DOM structure
      console.log("Inside target children:",
        $("#inside-target").children().length); // 1
      console.log("Outside target children:",
        $("#outside-target").children().length); // 0

      // Step 2: Check sibling relationships
      console.log("Outside target next sibling:",
        $("#outside-target").next().text()); // "Outside content"

      // Step 3: CSS descendant selector matches inside content only
      console.log("Descendant spans in body:",
        $("body span").length); // 2
      console.log("Spans inside #inside-target:",
        $("#inside-target span").length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output**
- `#inside-target` contains a `<span>` child.
- `#outside-target` has a `<span>` sibling (not a child).
- Console output confirms the structural differences.

**Why This Output Occurs**
`.append()` inserts the `<span>` as a child of `#inside-target`. `.after()` inserts the `<span>` as a sibling of `#outside-target`. The DOM structure differs fundamentally, affecting traversal and CSS matching.

### Real-World Cases

- **Lists**: Use `.append()` to add items **inside** a `<ul>`.
- **Form Labels**: Use `.before()` to add a label **before** an input (outside, as a sibling).
- **Cards**: Use `.append()` to add content **inside** a card container.
- **Dividers**: Use `.after()` to insert an `<hr>` **after** a section (outside).

### References

- jQuery API — Manipulation Category – https://api.jquery.com/category/manipulation/
- jQuery API — DOM Insertion, Inside – https://api.jquery.com/category/manipulation/dom-insertion-inside/
- jQuery API — DOM Insertion, Outside – https://api.jquery.com/category/manipulation/dom-insertion-outside/
- Stack Overflow — When to use append vs after in jQuery? – https://stackoverflow.com/questions/5034184/

---

## Summary Table: jQuery Insertion Methods at a Glance

| Method | Placement | Content Position | Target Position | Relationship |
|---|---|---|---|---|
| `.append(content)` | Inside | As argument | Before method | Content becomes last child |
| `.appendTo(target)` | Inside | Before method | As argument | Content becomes last child |
| `.prepend(content)` | Inside | As argument | Before method | Content becomes first child |
| `.prependTo(target)` | Inside | Before method | As argument | Content becomes first child |
| `.before(content)` | Outside | As argument | Before method | Content becomes preceding sibling |
| `.insertBefore(target)` | Outside | Before method | As argument | Content becomes preceding sibling |
| `.after(content)` | Outside | As argument | Before method | Content becomes following sibling |
| `.insertAfter(target)` | Outside | Before method | As argument | Content becomes following sibling |

### Key Behaviour Notes

| Behaviour | Inside Placement | Outside Placement |
|---|---|---|
| Single target | Content is moved (not cloned) | Content is moved (not cloned) |
| Multiple targets | Cloned for all but last | Cloned for all but last |
| Return value (content-first) | Target set | Inserted set |
| CSS descendant match | Yes | No |
| Event bubbling to target | Yes | No |

---

## Security Note

By design, any jQuery constructor or method that accepts an HTML string — `jQuery()`, `.append()`, `.after()`, etc. — can potentially execute code. This can occur by injection of script tags or use of HTML attributes that execute code (for example, `<img onload="">`). Do not use these methods to insert strings obtained from untrusted sources such as URL query parameters, cookies, or form inputs. Doing so can introduce cross-site-scripting (XSS) vulnerabilities. Remove or escape any user input before adding content to the document.

---

## General References

- jQuery API — .append() – https://api.jquery.com/append/
- jQuery API — .appendTo() – https://api.jquery.com/appendTo/
- jQuery API — .prepend() – https://api.jquery.com/prepend/
- jQuery API — .prependTo() – https://api.jquery.com/prependTo/
- jQuery API — .before() – https://api.jquery.com/before/
- jQuery API — .insertBefore() – https://api.jquery.com/insertBefore/
- jQuery API — .after() – https://api.jquery.com/after/
- jQuery API — .insertAfter() – https://api.jquery.com/insertAfter/
- jQuery API — DOM Insertion, Inside – https://api.jquery.com/category/manipulation/dom-insertion-inside/
- jQuery API — DOM Insertion, Outside – https://api.jquery.com/category/manipulation/dom-insertion-outside/
- jQuery API — Manipulation Category – https://api.jquery.com/category/manipulation/
- W3Schools — jQuery Add Elements – https://www.w3schools.com/jquery/jquery_dom_add.asp
- Stack Overflow — When to use append vs after in jQuery? – https://stackoverflow.com/questions/5034184/
- Stack Overflow — Difference between append and prepend – https://stackoverflow.com/questions/6636835/
- Stack Overflow — jQuery append vs appendTo – https://stackoverflow.com/questions/5034184/