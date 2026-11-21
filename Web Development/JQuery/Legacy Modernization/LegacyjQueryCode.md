# Legacy jQuery Code — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Legacy jQuery code is jQuery-based JavaScript that has accumulated over time in a codebase — often written years ago, sometimes by developers who are no longer on the team — characterised by imperative DOM manipulation, implicit global state, tight coupling between markup and behaviour, outdated patterns, and a lack of test coverage, making it difficult and risky to modify, extend, or migrate.

**Technical Definition**
Legacy jQuery code exhibits a cluster of technical characteristics: **direct DOM mutation outside any rendering abstraction**, **global event delegation without namespacing**, **plugin side effects that are undocumented and untestable**, **string-concatenated HTML that bypasses encoding**, **implicit dependencies between scripts that rely on load order**, **deprecated API usage** (`.bind()`, `.live()`, `.delegate()`, `$.browser`), **Hungarian notation and `$`-prefixed variables without consistent conventions**, **deeply nested callbacks and duplicated selector queries**, and **event handlers that encode untested business rules**. The OWASP and migration literature describes these codebases as carrying **hidden behaviours** — event delegation bound at `document` level, DOM mutations performed by third-party plugins outside any framework render cycle, and accessibility shims implemented ad hoc in plugins whose internals nobody remembers — that a mechanical framework port would silently drop or need to explicitly reproduce. Modernising this code requires a combination of **refactoring** (improving structure without changing behaviour), **incremental migration** (replacing legacy surfaces with modern equivalents), and **Strangler Fig** discipline (growing the new system around the old one until the old one can be removed).

**Beginner‑Friendly Explanation**
Legacy jQuery code is the code that everyone is afraid to touch. It works — mostly — but nobody fully understands it. It is the code where a single click handler does five different things, where the same element is looked up ten times in a row, where variables leak into the global scope, where functions are nested eight levels deep, and where a small change causes a mysterious bug three screens away. It is not necessarily badly written; it is **old** . It was written when the team was smaller, when the requirements were different, and when jQuery was the obvious choice. Over time, patches, hotfixes, and new features accumulated on top of each other, and the structure eroded. Legacy jQuery code is a problem not because jQuery is bad, but because **the code has lost its shape**. The goal of legacy modernisation is to restore that shape — by refactoring, by introducing boundaries, and by gradually replacing the old code with something that is easier to reason about.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Imperative DOM manipulation** | Code directly finds elements and changes them; state lives in the DOM rather than in JavaScript. |
| **Global event delegation without namespacing** | `$(document).on('click', …)` catches events from nodes that no longer exist, causing memory leaks and duplicate handlers. |
| **Plugin side effects are undocumented** | Plugins mutate the DOM, set timers, and attach listeners outside any visible render cycle; their behaviour is invisible in the application code. |
| **String‑concatenated HTML** | `.html()` and `innerHTML` are used with untrusted data, creating XSS risk and making the code hard to sanitise. |
| **Deprecated API usage** | `.bind()`, `.live()`, `.delegate()`, `$.browser`, and `.context` are used, preventing upgrades to modern jQuery. |
| **Duplicate selector queries** | The same selector is evaluated repeatedly in the same function, multiplying DOM traversal cost. |
| **Hungarian jQuery and inconsistent naming** | `$`-prefixed variables are used inconsistently, and naming conventions do not distinguish jQuery objects from DOM nodes. |
| **Deeply nested callbacks** | Ajax chains, event handlers, and animations are nested several levels deep, making control flow hard to follow. |
| **No test coverage** | Business logic embedded in event handlers is untested, so any refactoring risks silent behaviour regressions. |
| **Implicit load-order dependencies** | Scripts depend on other scripts having run first, but this dependency is not declared anywhere. |

---

### Prerequisites

- Working knowledge of jQuery selection, DOM manipulation, events, and AJAX.
- Familiarity with modern JavaScript (ES2015+), modules, and at least one modern target (vanilla JS, React, Vue, Angular, Svelte, Alpine, htmx).
- Access to the legacy codebase and the ability to run it locally or in a staging environment.
- Awareness of the testing tools available for the project (Jest, Vitest, Playwright, Cypress).

---

### Related Programming Areas

- **Code smells and anti-patterns** — the diagnostic vocabulary for identifying legacy problems.
- **Refactoring** — the disciplined technique for improving structure without changing behaviour.
- **Incremental migration and the Strangler Fig pattern** — the strategy for replacing legacy code safely.
- **Modern JavaScript** — the target APIs and patterns that replace jQuery idioms.
- **Testing and characterisation tests** — the safety net that makes refactoring and migration possible.
- **Security** — legacy jQuery versions carry known CVEs (prototype pollution, XSS), and legacy code often uses unsafe HTML sinks.

---

### Core Concepts / Features (Index)

1. Legacy jQuery Code (General)
2. jQuery Spaghetti Code
3. jQuery Anti‑Patterns
4. jQuery Code Smells
5. jQuery Refactoring
6. jQuery Modernization
7. jQuery Migration
8. Legacy Modernization Best Practices


## Core Concept 1: Legacy jQuery Code (General)

### Definitions

**Core Definition**
Legacy jQuery code is jQuery-based front-end code that has become difficult to maintain, extend, or safely modify because its structure has eroded over time through accumulated patches, missing tests, outdated patterns, and implicit dependencies.

**Technical Definition**
Legacy jQuery code is defined not by its age but by its **maintainability deficit**. The code may still function correctly, but it exhibits a combination of structural problems — direct DOM mutation, global state, duplicated selectors, deprecated APIs, untested business logic, and undocumented plugin side effects — that make change risky. The migration literature describes the core challenge as **hidden behaviour**: "event delegation bound at document level, DOM mutations performed by third-party plugins outside any framework render cycle, and accessibility behaviour (focus trapping, ARIA attribute toggling) implemented ad hoc in a plugin nobody remembers the internals of". A legacy codebase is therefore not simply "old jQuery"; it is a codebase whose **behavioural surface is larger than its visible surface**, and whose implicit contracts must be inventoried before any refactoring or migration can safely begin.

**Beginner‑Friendly Explanation**
Legacy jQuery code is code that has lost its shape. It might have started clean and organised, but years of quick fixes, new requirements, and different developers have left it tangled. The code still works — the site still loads, the buttons still click — but nobody is quite sure why, and every change feels dangerous. The defining feature is not the jQuery syntax; it is the **fear** the code inspires. Legacy code is code where the cost of understanding exceeds the cost of rewriting, and where the risk of changing it feels higher than the risk of leaving it alone.

---

### Purposes

- **To establish a shared definition** of what makes jQuery code "legacy" so that teams can prioritise modernisation efforts.
- **To distinguish between code that is merely old and code that is genuinely difficult to maintain**.
- **To motivate the inventory of hidden behaviours** before any refactoring or migration begins.
- **To provide a diagnostic vocabulary** for discussing legacy problems with stakeholders.
- **To set the stage for the specific anti-patterns, code smells, and refactoring techniques** covered in the following sections.

---

### Syntax Rules and Structure

Legacy jQuery code does not have a formal syntax; it has **recognisable structural signatures**:

| Signature | Example |
|---|---|
| **Direct DOM mutation** | `$('#output').html(userInput)` |
| **Global event delegation** | `$(document).on('click', '.btn', fn)` |
| **Duplicated selector queries** | `$('.item').show(); $('.item').addClass('x')` |
| **Deeply nested callbacks** | `$.get(url, function(data) { $.each(data, function(i, item) { $.get(item.url, function(detail) { … }) }) })` |
| **Deprecated API usage** | `$('.x').bind('click', fn)` or `$.browser.msie` |
| **Implicit load-order dependency** | Script B assumes Script A has initialised a global |

---

### Constraints and Limitations

- **Legacy code is not necessarily bad code**; it may encode years of tested business rules that are not written down anywhere else.
- **Rewriting from scratch is usually the wrong first move**; it discards tested behaviour and concentrates risk.
- **The "legacy" label is contextual**; a small, well-tested jQuery file is not legacy, regardless of its age.

---

### Annotated Code Example

```javascript
// ❌ LEGACY SIGNATURE: direct DOM mutation, global delegation,
//    duplicated selectors, and untested business logic
$(document).on('click', '.add-to-cart', function() {
  var id = $(this).attr('id');                // unnecessary jQuery wrap
  $('.cart-count').text(parseInt($('.cart-count').text()) + 1);
  $.post('/cart/add', { id: id }, function() {
    $('.cart-message').html('<p>Added to cart</p>'); // XSS sink
    setTimeout(function() {
      $('.cart-message').fadeOut();
    }, 3000);
  });
});

// ✅ MODERN SIGNATURE: scoped delegation, cached selection,
//    safe text rendering, and explicit async handling
(function($) {
  'use strict';

  var $cartCount = $('.cart-count');
  var $cartMessage = $('.cart-message');

  $(document).on('click.cart', '.add-to-cart', function() {
    var id = this.id;                          // native DOM access
    var count = parseInt($cartCount.text(), 10) + 1;
    $cartCount.text(count);

    $.post('/cart/add', { id: id })
      .done(function() {
        $cartMessage.text('Added to cart');     // safe sink
        setTimeout(function() { $cartMessage.fadeOut(); }, 3000);
      })
      .fail(function() {
        $cartMessage.text('Could not add to cart');
      });
  });
})(jQuery);
```

**Expected Output**
Both versions add an item to the cart and display a confirmation message. The legacy version leaks an event handler to `document`, duplicates selector queries, and uses `.html()` with a string. The modernised version namespaces the event (`.cart`), caches the selections, uses `this.id` instead of `$(this).attr('id')`, renders with `.text()` instead of `.html()`, and handles the failure case explicitly.

**Why this output occurs**: The modernised version restores the code's shape without changing its visible behaviour. The business logic (add to cart, update count, show message) is identical; the structure is safer and easier to reason about.

---

### Real‑World Cases

- **Meteor Blaze**: jQuery was centralised in a single file (`dombackend.js`) behind a `DOMBackend` abstraction, allowing a surgical rewrite of the internals to native DOM APIs without changing the public API.
- **MediaWiki**: Still supports jQuery but explicitly warns that "it is slow and dated, and there are some functions and patterns to avoid," listing deprecated APIs and ES5/ES6 equivalents to replace.
- **Legacy banking platforms**: Strangler Fig migrations from jQuery monoliths to React component architectures, with zero downtime and incremental traffic routing.

---

## Core Concept 2: jQuery Spaghetti Code

### Definitions

**Core Definition**
jQuery spaghetti code is legacy jQuery code whose control flow is tangled and whose responsibilities are interleaved — event handlers that both read and write the DOM, callbacks that both fetch data and render UI, and selectors that both query and mutate — making it impossible to trace the effect of a change without reading the entire file.

**Technical Definition**
Spaghetti code in jQuery arises from the language's imperative, chainable API combined with the absence of structural conventions. A single `$(document).ready()` block may contain dozens of unrelated event handlers; a single click handler may perform validation, DOM updates, AJAX calls, and animation in sequence; and a single selector may be re‑evaluated at every step. The result is a **tangled control flow graph** where: **(1)** event handlers are anonymous and cannot be tested in isolation; **(2)** state is implicit in the DOM rather than explicit in variables; **(3)** AJAX callbacks are nested inside event handlers, which are nested inside other callbacks; and **(4)** error handling is either absent or duplicated. The code smells literature identifies "too many return statements," "similar blocks of code found in multiple locations," and "deeply nested control flow statements" as measurable symptoms of spaghetti structure.

**Beginner‑Friendly Explanation**
Spaghetti code is code that looks like a plate of noodles: everything is connected to everything else, and if you pull one strand, the whole plate moves. In jQuery, spaghetti happens when you write everything in one long `$(document).ready()` block — click handlers, AJAX calls, animations, validation, and DOM updates all mixed together. You cannot test one piece without the others, you cannot change one behaviour without breaking another, and you cannot understand the file without reading all of it. The opposite of spaghetti is **structured code**: small functions with clear responsibilities, explicit state, and testable behaviour.

---

### Purposes

- **To recognise the structural signatures of spaghetti code** so that it can be refactored.
- **To understand why spaghetti code is resistant to testing and change**.
- **To motivate the separation of concerns** (data fetching, state management, rendering, event handling).
- **To provide a diagnostic checklist** for code reviews and technical debt assessments.

---

### Syntax Rules and Structure

| Spaghetti Signature | Structured Alternative |
|---|---|
| One large `$(document).ready()` block | Multiple small modules with explicit exports |
| Anonymous handlers with inline logic | Named functions with single responsibilities |
| DOM as the source of truth | JavaScript variables or a state object |
| Nested AJAX callbacks | Promises, `async/await`, or a service layer |
| Repeated selectors inside handlers | Cached selectors at module scope |
| Duplicated error handling | Centralised error handler |

---

### Constraints and Limitations

- **Spaghetti code is not always fixable by refactoring alone**; if the code has no tests, characterisation tests must be added first.
- **The tangled structure often encodes hidden behaviour**; refactoring must preserve that behaviour, not just the visible output.
- **Perfect structure is not the goal**; the goal is **testability, readability, and safe change**.

---

### Annotated Code Example

```javascript
// ❌ SPAGHETTI: one block, mixed responsibilities, nested callbacks
$(document).ready(function() {
  $('#load').click(function() {
    $.get('/api/items', function(items) {
      var html = '';
      $.each(items, function(i, item) {
        html += '<li data-id="' + item.id + '">' + item.name + '</li>';
      });
      $('#list').html(html);
      $('#list li').click(function() {
        var id = $(this).data('id');
        $.get('/api/item/' + id, function(detail) {
          $('#detail').html('<h2>' + detail.name + '</h2><p>' + detail.desc + '</p>');
          $('#detail .close').click(function() {
            $('#detail').hide();
          });
        });
      });
    });
  });
});

// ✅ STRUCTURED: separated concerns, named functions, cached selectors
(function($) {
  'use strict';

  var $list = $('#list');
  var $detail = $('#detail');

  function renderList(items) {
    var html = items.map(function(item) {
      return '<li data-id="' + item.id + '">' + item.name + '</li>';
    }).join('');
    $list.html(html);
  }

  function renderDetail(detail) {
    $detail.html(
      '<h2>' + detail.name + '</h2><p>' + detail.desc + '</p>'
    );
  }

  function loadItems() {
    return $.getJSON('/api/items').then(renderList);
  }

  function loadDetail(id) {
    return $.getJSON('/api/item/' + id).then(renderDetail);
  }

  $(document).on('click.list', '#load', loadItems);
  $(document).on('click.list', '#list li', function() {
    loadDetail($(this).data('id'));
  });
  $(document).on('click.list', '#detail .close', function() {
    $detail.hide();
  });
})(jQuery);
```

**Expected Output**
Both versions load a list of items, render them, and display item details on click. The spaghetti version nests callbacks, re‑queries selectors, and attaches handlers directly to dynamically created elements. The structured version separates rendering (`renderList`, `renderDetail`) from data fetching (`loadItems`, `loadDetail`), uses promises for async flow, and uses delegated events that survive re‑rendering.

**Why this output occurs**: The structured version restores the code's shape: each function has a single responsibility, the async flow is linear, and the event handlers are delegated and namespaced. The behaviour is preserved, but the code is now testable and maintainable.

---

### Real‑World Cases

- **MediaWiki JavaScript**: The coding conventions explicitly recommend avoiding deeply nested control flow and using ES5/ES6 equivalents for jQuery utilities, precisely to prevent spaghetti accumulation.
- **CodeClimate analysis of jQuery-dependent projects**: Flags "too many return statements," "similar blocks of code found in multiple locations," and "deeply nested control flow" as measurable spaghetti symptoms.

---

## Core Concept 3: jQuery Anti‑Patterns

### Definitions

**Core Definition**
jQuery anti‑patterns are commonly used jQuery idioms that appear convenient but produce code that is slower, less secure, harder to maintain, or more fragile than the alternatives — patterns that "everyone does" but that experienced developers learn to avoid.

**Technical Definition**
Anti‑patterns are distinguished from mere mistakes by their **systemic nature**: they arise from jQuery's API design and are reinforced by tutorials, copy‑pasted snippets, and the library's historical reputation for convenience. The core anti‑patterns are: **(1) redundant selector queries** — re‑evaluating the same selector on every line instead of caching the result; **(2) unnecessary jQuery wrapping** — `$(this).attr('id')` instead of `this.id`; **(3) global scope pollution** — declaring variables without `var`/`let`/`const`, or attaching everything to `window`; **(4) global event delegation without namespacing** — `$(document).on('click', fn)` that catches events from nodes that no longer exist; **(5) using jQuery for everything** — array iteration, AJAX, and utility functions that have native equivalents; and **(6) unsafe HTML sinks** — passing untrusted data to `.html()`, `.append()`, or `$.parseHTML()`.

**Beginner‑Friendly Explanation**
An anti‑pattern is a habit that looks helpful but hurts you later. In jQuery, the classic anti‑pattern is writing `$('.item')` over and over again in the same function, because jQuery is so easy to use that it feels free — but every call searches the entire page. Another is `$(document).on('click', fn)`, which seems convenient because it catches everything, but it also catches events from elements you have already removed, causing leaks. Anti‑patterns are not rare; they are the **default** way people write jQuery, which is why legacy codebases are full of them.

---

### Purposes

- **To name and recognise the patterns** that degrade jQuery code quality.
- **To provide a checklist for code reviews** that catches anti‑patterns before they spread.
- **To motivate the use of modern JavaScript equivalents** for array iteration, AJAX, and utilities.
- **To reduce the security surface** by eliminating unsafe HTML sinks and global state.

---

### Syntax Rules and Structure

| Anti‑Pattern | Problematic Code | Better Alternative |
|---|---|---|
| **Redundant selectors** | `$('.item').show(); $('.item').addClass('x')` | `var $items = $('.item'); $items.show().addClass('x')` |
| **Unnecessary jQuery wrap** | `$(this).attr('id')` | `this.id` |
| **Global scope pollution** | `name = 'x'` (no declaration) | `var name = 'x'` or `const name = 'x'` |
| **Global delegation** | `$(document).on('click', fn)` | `$(document).on('click.myModule', '.btn', fn)` |
| **jQuery for everything** | `$.each(arr, fn)` | `arr.forEach(fn)` |
| **Unsafe HTML sinks** | `$(el).html(userInput)` | `$(el).text(userInput)` or DOMPurify |
| **Deprecated APIs** | `.bind()`, `.live()`, `.delegate()` | `.on()`, `.off()` |

---

### Constraints and Limitations

- **Anti‑patterns are context‑dependent**; caching a selector used only once adds a variable without benefit.
- **Some anti‑patterns are performance‑only**; others (like unsafe HTML) are security‑critical.
- **Modern jQuery versions (3.x/4.x) reduce some performance gaps**, but the structural and security anti‑patterns remain valid.

---

### Annotated Code Example

```javascript
// ❌ ANTI-PATTERNS: redundant selectors, unnecessary wrap,
//    global delegation, unsafe HTML, deprecated APIs
$(document).ready(function() {
  $('.title').show();                         // selector query
  $('.title').addClass('visible');            // selector query again
  $('.btn').bind('click', function() {        // deprecated .bind
    var id = $(this).attr('id');              // unnecessary wrap
    $('.output').html('<p>' + id + '</p>');   // unsafe HTML sink
  });
  $(document).on('click', '.dynamic', fn);    // global delegation
});

// ✅ REFACTORED: cached selectors, native access, safe sinks,
//    namespaced delegation, modern API
(function($) {
  'use strict';

  $(function() {
    var $title = $('.title');                  // cache once
    $title.show().addClass('visible');

    $(document).on('click.buttons', '.btn', function() {
      var id = this.id;                       // native DOM access
      $('.output').text(id);                   // safe sink
    });

    $(document).on('click.dynamic', '.dynamic', fn);
  });
})(jQuery);
```

**Expected Output**
Both versions show the title, handle button clicks, and bind a delegated handler. The anti‑pattern version performs three selector queries where one would suffice, wraps `this` unnecessarily, uses the deprecated `.bind()`, renders with `.html()`, and leaves the delegated handler unnamed and unremovable. The refactored version caches the selection, uses `this.id`, uses `.text()`, namespaces both delegated handlers, and uses the modern `.on()` API.

**Why this output occurs**: Each refactoring step addresses a specific anti‑pattern. The behaviour is identical; the structure and safety are improved.

---

### Real‑World Cases

- **MediaWiki**: Explicitly lists deprecated jQuery functions to avoid (`.bind`, `.unbind`, `.delegate`, `.undelegate`, `.browser`, etc.) and recommends ES5/ES6 equivalents for `.each`, `.grep`, `.inArray`, `.map`, `.trim`, `.extend`.
- **jQuery Framework Rules**: A best‑practices checklist that explicitly calls out "Don't use jQuery for everything," "Don't pollute global scope," and "Cache jQuery selectors".
- **Stack Overflow jQuery best‑practices discussions**: The canonical anti‑patterns — redundant selectors, unnecessary wrapping, and passing non‑DOM nodes as context — are documented and consistently recommended against.

---

## Core Concept 4: jQuery Code Smells

### Definitions

**Core Definition**
jQuery code smells are surface‑level symptoms in jQuery code that suggest a deeper structural or design problem — patterns that are not bugs in themselves but that often correlate with maintainability, performance, or security issues.

**Technical Definition**
Code smells in jQuery are detected through both **static analysis** and **expert heuristics**. The measurable smells include: **long functions** (e.g., `toggleClass` exceeding 25 lines), **deeply nested control flow**, **duplicated code blocks**, **inconsistent return statements**, and **excessive function length**. The jQuery‑specific smells include: **Hungarian jQuery** (inconsistent `$` prefixing that confuses jQuery objects with DOM nodes), **mid‑chain reference leaks** (methods like `.find()`, `.filter()`, `.children()` that select new elements from an existing set and can leak memory if not cleaned up), **selector repetition**, and **implicit state in the DOM**. The term "Hungarian jQuery" was coined to describe the smell where developers prefix every jQuery variable with `$` without a consistent convention, making the code harder to read rather than easier.

**Beginner‑Friendly Explanation**
A code smell is a warning sign — not a bug, but something that often leads to bugs. In jQuery, a smell might be a function that is 200 lines long, or a variable named `$el` that sometimes holds a jQuery object and sometimes holds a DOM node. Another smell is a chain like `$(this).find('.x').filter('.y').children()` followed by no cleanup, which can leak memory. Smells are useful because they are **easy to spot** and they point to deeper problems. Fixing a smell does not always fix the underlying issue, but it makes the code easier to work with and often prevents the issue from getting worse.

---

### Purposes

- **To provide a diagnostic vocabulary** for code reviews and technical debt assessments.
- **To catch maintainability problems early**, before they become bugs.
- **To guide refactoring priorities** by identifying the most damaging smells.
- **To distinguish cosmetic smells** (Hungarian jQuery) from **structural smells** (deep nesting, duplication) and **security smells** (unsafe HTML sinks).

---

### Syntax Rules and Structure

| Smell Category | Signature | Example |
|---|---|---|
| **Structural** | Long functions, deep nesting, duplication | 40‑line `toggleClass`; nested `if/else` inside `$.each` |
| **jQuery‑specific** | Hungarian jQuery, selector repetition | `$el`, `$this`, `$item` used inconsistently |
| **Memory** | Mid‑chain reference leaks | `$(this).find('.x').data('key', largeObject)` without cleanup |
| **Security** | Unsafe HTML sinks | `.html(userInput)` or string concatenation into `innerHTML` |
| **Lifecycle** | Global delegation without namespacing | `$(document).on('click', fn)` with no `.off()` counterpart |

---

### Constraints and Limitations

- **Smells are heuristics, not proofs**; a long function may be appropriate in some contexts.
- **Static analysis tools flag smells in vendor libraries**; scope scans to application code.
- **Fixing a smell does not guarantee the underlying issue is resolved**; refactoring must be paired with tests.

---

### Annotated Code Example (Hungarian jQuery Smell)

```javascript
// ❌ HUNGARIAN JQUERY SMELL: inconsistent $ prefixing
var $this = this;                // DOM node with $ prefix
var el = $(this);                // jQuery object without $ prefix
var $item = $('.item');          // jQuery object with $ prefix
var item = document.getElementById('item'); // DOM node without $ prefix

// ✅ CONSISTENT CONVENTION: $ for jQuery objects, no $ for DOM nodes
var thisEl = this;               // DOM node
var $thisEl = $(this);           // jQuery object
var $item = $('.item');          // jQuery object
var itemEl = document.getElementById('item'); // DOM node
```

**Expected Output**
The behaviour is identical, but the consistent convention makes it immediately clear which variables are jQuery objects and which are raw DOM nodes. This reduces the risk of calling jQuery methods on DOM nodes (or vice versa).

**Why this output occurs**: The smell is not the `$` prefix itself — it is the **inconsistency**. A consistent convention is a readability aid; an inconsistent one is a trap.

---

### Real‑World Cases

- **CodeClimate**: Flags long functions, duplicated code, and deep nesting in jQuery files, providing a quantified smell report.
- **James Padolsey's "jQuery Code Smells"**: A widely cited article that catalogues the patterns experienced developers learn to avoid.
- **eslint-plugin-no-jquery**: An ESLint plugin that discourages jQuery functions and encourages maintaining state in JavaScript rather than querying the DOM.

---

## Core Concept 5: jQuery Refactoring

### Definitions

**Core Definition**
jQuery refactoring is the disciplined process of improving the internal structure of jQuery code — without changing its external behaviour — by eliminating duplication, reducing nesting, extracting named functions, caching selections, namespacing events, and replacing deprecated APIs with modern equivalents.

**Technical Definition**
Refactoring is distinguished from rewriting by its **behaviour‑preserving** constraint. The refactoring workflow for jQuery code follows the standard discipline: **(1)** establish characterisation tests that capture the current behaviour; **(2)** identify a specific smell or anti‑pattern; **(3)** apply a targeted transformation (e.g., extract function, cache selector, replace `.bind()` with `.on()`); **(4)** run the tests to confirm behaviour is unchanged; **(5)** repeat. The common jQuery refactoring transformations include: **selector caching** (replacing repeated `$('.x')` calls with a single variable); **event namespacing** (replacing `.on('click', fn)` with `.on('click.module', fn)` and `.off('.module')`); **delegation replacement** (replacing direct binding on dynamic elements with delegated `.on()`) ; **deprecated API replacement** (`.bind()` → `.on()`, `.live()` → `.on()`, `.delegate()` → `.on()`); **IIFE encapsulation** (wrapping code in `(function($) { … })(jQuery)` to avoid global pollution); and **promise adoption** (replacing nested AJAX callbacks with `.done()`, `.fail()`, or `async/await`).

**Beginner‑Friendly Explanation**
Refactoring is cleaning up the code without changing what it does. It is like reorganising a messy room: the furniture stays the same, but now you can find things and move around. In jQuery, refactoring means taking a long, tangled function and breaking it into smaller named functions; taking repeated `$('.item')` calls and storing the result in a variable; taking `.bind()` and replacing it with `.on()`; and taking global variables and wrapping them in an IIFE so they do not leak. The key rule is: **behaviour must not change**. If the tests still pass, the refactoring is safe.

---

### Purposes

- **To improve readability and maintainability** without changing behaviour.
- **To eliminate performance anti‑patterns** (redundant selectors, unnecessary wrapping).
- **To eliminate security anti‑patterns** (unsafe HTML sinks, deprecated APIs with known issues).
- **To prepare the code for migration** by making its behaviour explicit and testable.
- **To reduce the risk of future changes** by restoring structure.

---

### Syntax Rules and Structure

| Refactoring | Before | After |
|---|---|---|
| **Selector caching** | `$('.x').show(); $('.x').addClass('y')` | `var $x = $('.x'); $x.show().addClass('y')` |
| **Event namespacing** | `$(el).on('click', fn)` | `$(el).on('click.module', fn); $(el).off('.module')` |
| **Delegation** | `$('.dynamic').click(fn)` | `$(document).on('click.dynamic', '.dynamic', fn)` |
| **Deprecated API** | `$(el).bind('click', fn)` | `$(el).on('click', fn)` |
| **IIFE encapsulation** | `var x = 1;` (global) | `(function($) { var x = 1; })(jQuery)` |
| **Promise adoption** | `$.get(url, function(data) { $.get(data.url, …) })` | `$.getJSON(url).then(function(data) { return $.getJSON(data.url); })` |

---

### Constraints and Limitations

- **Refactoring without tests is risky**; characterisation tests must be added first.
- **Some refactorings are purely cosmetic** (Hungarian jQuery) and do not fix structural problems.
- **Refactoring does not replace migration**; a perfectly refactored jQuery codebase is still a jQuery codebase.

---

### Annotated Code Example

```javascript
// BEFORE: repeated selectors, deprecated API, global scope
$('.item').show();
$('.item').addClass('active');
$('.item').on('click', function() { … });

// AFTER: cached selection, modern API, namespaced event, IIFE scope
(function($) {
  'use strict';

  $(function() {
    var $items = $('.item');
    $items.show().addClass('active');
    $items.on('click.items', function() { … });
    // later: $items.off('.items');
  });
})(jQuery);
```

**Expected Output**
The behaviour is identical: the items are shown, the active class is added, and the click handler is bound. The refactored version performs one selector query instead of three, uses the modern `.on()` API, namespaces the event for clean unbinding, and confines its variables to the IIFE scope.

**Why this output occurs**: Each transformation is behaviour‑preserving. The selector caching reduces DOM queries; the namespacing enables precise cleanup; the IIFE prevents global pollution.

---

### Real‑World Cases

- **Single‑page application resource cleanup**: A checklist that recommends namespacing events (`click.myFeature`) and using `.off('.myFeature')` for precise teardown, plus cancelling pending AJAX requests on route change.
- **MediaWiki**: Recommends replacing deprecated jQuery APIs and using ES5/ES6 equivalents for jQuery utilities, as part of its coding conventions.
- **jQuery plugin modularisation**: Using IIFEs to create private scopes and avoid global variable pollution in plugin code.

---

## Core Concept 6: jQuery Modernization

### Definitions

**Core Definition**
jQuery modernization is the process of bringing a legacy jQuery codebase up to current standards — upgrading jQuery itself to a supported, patched version; replacing deprecated APIs; adopting modern JavaScript features; and introducing build tooling, linting, and testing — without necessarily replacing jQuery with another framework.

**Technical Definition**
Modernization is distinct from migration: migration replaces jQuery with a different technology, while modernization **improves the jQuery codebase in place**. The core modernization tasks are: **(1) upgrade jQuery** to a supported version (3.7.x or 4.0.0), which removes legacy browser support, drops deprecated APIs, and aligns event behaviour with modern standards; **(2) remove deprecated APIs** (`.bind()`, `.live()`, `.delegate()`, `$.browser`, `.context`, etc.) and replace them with their modern equivalents; **(3) adopt modern JavaScript** (`let`/`const`, arrow functions, template literals, promises, `async/await`, `fetch`) alongside jQuery; **(4) introduce tooling** (ESLint with `eslint-plugin-no-jquery` or `eslint-config-wikimedia`, a bundler such as Rollup or Vite, and a test runner such as Jest or Vitest); and **(5) remove reliance on jQuery Migrate**, which is a temporary shim that keeps deprecated APIs alive but can quietly keep the codebase on a vulnerable jQuery version.

**Beginner‑Friendly Explanation**
Modernizing jQuery is like renovating an old house without moving out. You upgrade the wiring (jQuery version), replace the old fixtures (deprecated APIs), add safety features (linting and tests), and bring in modern appliances (ES6+ features). You do not tear the house down; you make it safer and more comfortable to live in. Modernization is often the right first step before migration, because it makes the codebase easier to understand and test, and it eliminates the known security vulnerabilities in old jQuery versions.

---

### Purposes

- **To eliminate known security vulnerabilities** in outdated jQuery versions (prototype pollution, XSS).
- **To make the codebase compatible with modern tooling** (bundlers, linters, test runners).
- **To reduce the cost of future migration** by first making the codebase clean and testable.
- **To align with modern browser standards** and remove legacy browser workarounds.
- **To remove jQuery Migrate** and the false sense of safety it provides.

---

### Syntax Rules and Structure

| Modernization Task | Before | After |
|---|---|---|
| **jQuery version** | jQuery 1.x / 2.x | jQuery 3.7.x or 4.0.0 |
| **Deprecated API** | `.bind()`, `.live()`, `.delegate()` | `.on()`, `.off()` |
| **jQuery utilities** | `$.each()`, `$.map()`, `$.trim()` | `Array.prototype.forEach()`, `.map()`, `String.prototype.trim()` |
| **AJAX** | `$.ajax()` with callbacks | `fetch()` with `async/await` (or jQuery promises) |
| **Migrate shim** | jQuery Migrate in production | Remove after deprecation warnings are fixed |
| **Linting** | None | ESLint with `eslint-plugin-no-jquery` |

---

### Constraints and Limitations

- **Upgrading to jQuery 4.0.0 introduces breaking changes**; deprecated APIs are removed, and plugins may need updates.
- **jQuery Migrate is a temporary tool**, not a permanent solution; relying on it can keep the codebase on a vulnerable version.
- **Modernization does not replace migration**; it improves the legacy code but does not change its fundamental architecture.

---

### Annotated Code Example

```javascript
// BEFORE: jQuery 1.x style, deprecated APIs, Migrate dependency
$(document).ready(function() {
  $('.btn').bind('click', function() {
    $.get('/api/data', function(data) {
      $('.output').html(data);
    });
  });
});

// AFTER: jQuery 3.7.x/4.0, modern APIs, no Migrate, linted
$(function() {
  $('.btn').on('click', function() {
    $.getJSON('/api/data')
      .done(function(data) {
        $('.output').text(data);        // safe sink
      })
      .fail(function() {
        $('.output').text('Error loading data');
      });
  });
});
```

**Expected Output**
Both versions load data and display it. The modernized version uses `.on()` instead of `.bind()`, `.getJSON()` with `.done()`/`.fail()` instead of nested callbacks, and `.text()` instead of `.html()`, eliminating the XSS risk. It does not require jQuery Migrate.

**Why this output occurs**: Modernization removes deprecated APIs, adopts promise‑based async handling, and replaces unsafe HTML sinks. The behaviour is preserved; the security and maintainability are improved.

---

### Real‑World Cases

- **jQuery 4.0.0 release**: Removes legacy browser support, drops deprecated APIs, aligns event behaviour with modern standards, and reduces the build size by ~3 kB gzipped.
- **MediaWiki**: Uses `eslint-config-wikimedia` to lint jQuery code and explicitly recommends replacing deprecated APIs and ES5/ES6 equivalents.
- **SAP**: Provides a rule for "Usage of deprecated jQuery API" and strongly recommends removing/refactoring all deprecated usage to stay future‑proof.

---

## Core Concept 7: jQuery Migration

### Definitions

**Core Definition**
jQuery migration is the process of replacing jQuery — partially or entirely — with a different technology: vanilla JavaScript, a component framework (React, Vue, Angular, Svelte), a lightweight reactive library (Alpine.js), or a hypermedia‑driven tool (htmx), while maintaining business continuity and preserving tested behaviour.

**Technical Definition**
Migration is the **replacement** of the legacy technology, not just its improvement. The technical challenge is **DOM ownership conflict**: jQuery assumes it can read and mutate the DOM at any time, while modern frameworks assume exclusive ownership of their subtrees and reconcile against a virtual or compiled representation. The migration strategies are therefore centred on **coexistence** (running old and new code side by side), **boundaries** (isolating jQuery inside containers the framework does not touch), and **incremental replacement** (the Strangler Fig pattern). The target selection depends on the application: **vanilla JavaScript** for simple DOM interactions, **React/Vue/Angular/Svelte** for component‑based applications, **Alpine.js** for declarative interactivity without a build step, and **htmx** for hypermedia‑driven server interactions. The migration order is typically: **AJAX first** (htmx or `fetch`), then **event handling** (native or framework events), then **DOM manipulation** (framework templates), and finally **plugin replacement**.

**Beginner‑Friendly Explanation**
Migration is moving out of the jQuery house and into a new one. You can move everything at once (a full rewrite), or you can move one room at a time (incremental migration). Most teams choose incremental migration because it is safer: you keep living in the old house while the new one is being built, and you move when each room is ready. The new house might be made of vanilla JavaScript (simple and lightweight), React or Vue (component‑based), Alpine (declarative and lightweight), or htmx (server‑driven). The important part is to choose the right house for your needs and to move deliberately, with tests and rollback plans.

---

### Purposes

- **To replace jQuery with a modern technology** that better fits the application's needs.
- **To eliminate the jQuery dependency** and its historical security vulnerabilities.
- **To adopt a declarative, component‑based, or hypermedia‑driven architecture**.
- **To improve maintainability and testability** by moving away from imperative DOM manipulation.
- **To enable incremental migration** without freezing feature development or discarding business logic.

---

### Syntax Rules and Structure

| Migration Target | When to Choose | Key Pattern |
|---|---|---|
| **Vanilla JavaScript** | Simple DOM interactions, no framework needed | Replace `$()` with `querySelector`, `.on()` with `addEventListener`, `$.ajax` with `fetch` |
| **React** | Component‑based UI, large team, rich ecosystem | Wrapper components with `useRef`/`useEffect` |
| **Vue** | Progressive adoption, reactive state | Directives and wrapper components with `onMounted`/`onBeforeUnmount` |
| **Angular** | Enterprise applications, strong typing | Model‑driven components, `HttpClient`, no direct DOM access |
| **Svelte** | Compiled reactivity, minimal runtime | `bind:this` and `onMount` for plugin integration |
| **Alpine.js** | Lightweight declarative interactivity, no build step | `x-data`, `x-init`, `x-ref`, `$nextTick` |
| **htmx** | Server‑rendered HTML, hypermedia‑driven | `hx-get`, `hx-target`, `hx-swap` replace `$.ajax` + `.html()` |

---

### Constraints and Limitations

- **Full rewrites are usually the wrong first move**; they freeze feature development and concentrate risk.
- **DOM ownership conflicts** must be resolved with boundaries (refs, `v-once`, `wire:ignore`, `bind:this`).
- **Shared auth/session models** must be solved before coexistence begins.
- **The choice of target depends on the application**; there is no universal "best" framework.

---

### Annotated Code Example (Incremental Migration to htmx)

```html
<!-- BEFORE: jQuery AJAX + .html() -->
<script>
  $('#search-btn').on('click', function() {
    $.get('/posts/', $('#search-form').serialize(), function(html) {
      $('#post-list').html(html);
    });
  });
</script>

<!-- AFTER: htmx attributes, no JavaScript -->
<form id="search-form" hx-get="/posts/" hx-target="#post-list" hx-swap="innerHTML">
  <button type="submit" id="search-btn">Search</button>
</form>
<div id="post-list"></div>
```

**Expected Output**
Submitting the form fetches `/posts/` and inserts the response into `#post-list`. No JavaScript is required.

**Why this output occurs**: htmx processes the `hx-get`, `hx-target`, and `hx-swap` attributes at page load. The form submission is intercepted, the request is sent, and the response is swapped into the target element. The jQuery AJAX code is no longer needed.

---

### Real‑World Cases

- **Meteor Blaze**: Replaced jQuery with native DOM APIs in `dombackend.js`, using `addEventListener`, `closest()`, and `<template>` parsing, while keeping the same `DOMBackend` API.
- **Drupal core**: A proposal to use htmx to replace jQuery AJAX behaviours.
- **Magento Hyvä**: Replaced the Luma jQuery/Knockout/RequireJS stack with Alpine.js and Tailwind CSS, reducing the JavaScript payload by ~90%.
- **Legacy banking platforms**: Strangler Fig migrations from jQuery monoliths to React, with zero downtime.

---

## Core Concept 8: Legacy Modernization Best Practices

### Definitions

**Core Definition**
Legacy jQuery modernization best practices are the consolidated conventions — inventory before changing, test before refactoring, use the Strangler Fig pattern, migrate in risk order, maintain dual‑stack discipline, and document the transition — that make the process of modernising legacy jQuery code safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual strategies: **(1) inventory hidden behaviours** — event delegation, plugin side effects, accessibility shims, and unsanitised HTML — before any migration plan commits to a boundary; **(2) establish characterisation tests** before refactoring or migrating, so that behaviour regressions are caught immediately; **(3) default to the Strangler Fig pattern**, scoped by seams (routes, modules, iframes), and avoid full rewrites unless the surface is small and isolated; **(4) sequence migrations business‑risk‑first** — low‑risk surfaces (internal admin pages, isolated widgets) before high‑risk surfaces (checkout, authentication); **(5) maintain backward compatibility** at every step and run old and new paths in parallel; **(6) define rollback procedures** for each phase; **(7) monitor for regressions** continuously and use feature flags for gradual rollout; and **(8) document the migration decisions** and the new architecture.

**Beginner‑Friendly Explanation**
Best practices are the rules that keep a modernisation project from turning into a disaster. Start by understanding what you have: list every jQuery plugin, every global event handler, every piece of business logic hidden in a click callback. Write tests before you change anything, so you know when you break something. Move one small piece at a time, using the Strangler Fig pattern, and keep the old system running until the new one is proven. Migrate the easy, low‑risk stuff first. Have a rollback plan. Watch for errors. And write down what you did. These habits turn a risky, stressful modernisation into a controlled, manageable process.

---

### Purposes

- **To provide a single actionable checklist** for legacy jQuery modernisation.
- **To prevent the most common modernisation failures** (full rewrites, untested refactoring, feature freezes).
- **To balance modernisation speed with operational stability**.
- **To establish a baseline for planning and executing the modernisation**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Inventory** | List all jQuery usage, plugins, and global state | Starting without knowing the scope |
| **Tests** | Characterisation tests before refactoring | Refactoring untested code |
| **Pattern** | Strangler Fig, scoped by seams | Big‑bang rewrite |
| **Sequencing** | Low‑risk surfaces first | Starting with the highest‑risk flow |
| **Coexistence** | Old and new paths run in parallel | Cutting over without verification |
| **Rollback** | Defined rollback procedure per phase | No way to undo a failed migration |
| **Monitoring** | Continuous regression monitoring | Assuming it works without checking |
| **Documentation** | Document decisions and new architecture | Leaving the next team guessing |

---

### Constraints and Limitations

- **Best practices must be adapted to context**; a small internal tool has different risk tolerances than a customer‑facing checkout.
- **Strangler migrations require sustained dual‑stack capacity**; if the team cannot maintain two stacks, the sequencing must change.
- **Some legacy plugins are fundamentally incompatible** with modern frameworks; a bounded rewrite of that specific surface may be the only viable option.

---

### Annotated Code Example (Consolidated Modernisation Checklist)

```text
✅ 1. Inventory: list $(, $.ajax, $.get, $.post, plugins, global state
✅ 2. Tests: add characterisation tests for surfaces to be modernised
✅ 3. Seams: identify route, module, and iframe boundaries
✅ 4. Strangler: set up routing layer; new routes → new app, old routes → legacy
✅ 5. Sequence: modernise low-risk routes/widgets first
✅ 6. Coexist: run old and new paths in parallel; compare behaviour
✅ 7. Rollback: feature flags and rollback procedures for each phase
✅ 8. Monitor: regression suites, error rates, adoption metrics
✅ 9. Decommission: remove legacy code paths; document new architecture
```

**Expected Output**
A structured, risk‑managed modernisation that proceeds in verifiable increments and ends with the legacy system fully modernised or replaced.

**Why this output occurs**: Each step addresses a specific modernisation risk: scope uncertainty (inventory), behaviour regression (tests), concentrated risk (Strangler), operational disruption (coexistence and rollback), and incomplete modernisation (decommission and documentation).

---

### Real‑World Cases

- **Legacy banking platform**: Strangler Fig migration from jQuery to React, with zero downtime and incremental traffic routing.
- **Meteor Blaze**: Surgical replacement of jQuery internals with native DOM APIs, preserving the public `DOMBackend` API.
- **MediaWiki**: Incremental modernisation of jQuery usage through linting rules, deprecated‑API warnings, and ES5/ES6 equivalents.

---

## References

- Manual:Coding conventions/JavaScript | MediaWiki — https://www.mediawiki.org/w/index.php?title=Manual:Coding_conventions/JavaScript&curid=75214&diff=6861511&oldid=6861506
- jQuery Framework Rules | GitHub — https://github.com/hivellm/rulebook/blob/9f46bd9f41ce1eead578087e735701ac285df673/templates/frameworks/JQUERY.md
- Jquery Bad practices | Stack Overflow (archived) — https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/2070097/jquery-bad-practices
- 分享｜单页应用中 jQuery 资源回收清单 | LeetCode — https://leetcode.cn/discuss/post/3838559/dan-ye-ying-yong-zhong-jquery-zi-yuan-hu-z96d/
- vanguard-frontier-agentic/skills/frontend/legacy-jquery-to-modern-framework-review/SKILL.md | GitHub — https://github.com/VincentChuWaiChow/vanguard-frontier-agentic/blob/master/skills/frontend/legacy-jquery-to-modern-framework-review/SKILL.md
- Remove jQuery dependency from Blaze — Native DOM rewrite of dombackend.js | GitHub Issue #490 — https://github.com/meteor/blaze/issues/490
- Stop using Hungarian jQuery | Aaron Powell — https://github.com/aaronpowell/aaronpowell.github.io/blob/9e7dd8d5a51ccfcd6df2dee0f292a7204e928fc7/src/content/posts/2012-06-27-hungarian-jquery.md
- jQuery 4.0.0 Support: What’s New + Kendo UI Has You Covered | Telerik — https://www.telerik.com/blogs/jquery-4-0-0-support-whats-new-kendo-ui-has-you-covered
- jQuery Migrate: A Practical Security Guide | Safeguard — https://safeguard.sh/resources/blog/jquery-migrate
- Modernizing a Legacy Banking Platform: How a Strangler Fig Approach Enabled Zero-Downtime Migration from jQuery Monoliths to a Scalable React Component Architecture | JISEM — https://www.jisem-journal.com
- jQuery插件如何实现模块化设计 | Yisu — https://www.yisu.com/jc/1058819.html
- Identification and Refactoring of Exception Handling Code Smells in JavaScript Based on jQuery and AngularJS Libraries | UiTM — http://ndltd.ncl.edu.tw/handle/e3j2he
- CodeClimate: rove issues — https://codeclimate.com
- DeepSource: JS-0045 — https://app.deepsource.com
- Developer Guide | SAP — https://help.sap.com
- jQuery 4.0 veröffentlicht: Weniger Legacy, mehr moderne Webstandards | heise — https://www.heise.de
- ResourceLoader/Migration guide (users)/zh | MediaWiki — https://www.mediawiki.org/wiki/ResourceLoader/Migration_guide_(users)/zh