# jQuery to Vanilla JavaScript — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Migrating from jQuery to vanilla JavaScript is the systematic process of replacing jQuery's selectors, DOM manipulation helpers, event system, AJAX wrapper, effects, and utility functions with native browser APIs — `querySelector`, `classList`, `addEventListener`, `fetch`, CSS transitions, and standard built‑in methods — thereby removing the jQuery dependency and reducing bundle size while preserving behaviour.

**Technical Definition**
jQuery was created to normalise DOM APIs across a fragmented browser landscape. Modern evergreen browsers implement a consistent, powerful DOM API: `querySelector` and `querySelectorAll` for selection; `classList`, `textContent`, `innerHTML`, `append`, `prepend`, `insertAdjacentHTML`, `remove` for manipulation; `addEventListener` and `dispatchEvent` for events; `fetch` for HTTP; CSS transitions and the Web Animations API for animation; and `Array.prototype` / `Object` methods for utilities. The migration maps each jQuery method to its native equivalent, accounts for behavioural differences (e.g., `fetch` does not reject on HTTP 4xx/5xx; `querySelector` returns `null` rather than an empty jQuery object), and proceeds incrementally to keep production stable.

**Beginner‑Friendly Explanation**
jQuery is a convenience layer that makes web page scripting easier. But modern browsers now include almost everything jQuery was invented to provide. Migrating to vanilla JavaScript means learning the native equivalents — `querySelector` instead of `$()`, `classList.add()` instead of `.addClass()`, `addEventListener` instead of `.on()`, `fetch` instead of `$.ajax()` — and replacing jQuery calls one at a time. The benefit is a smaller, faster page and one fewer dependency to maintain and patch.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Direct API equivalents exist for most jQuery methods** | `querySelector`, `classList`, `addEventListener`, `fetch`, `textContent`, and array methods cover the majority of day‑to‑day jQuery usage. |
| **Behavioural differences require attention** | `fetch` does not reject on HTTP errors; `querySelector` returns `null`; `querySelectorAll` returns a static `NodeList`, not a jQuery object. |
| **Some jQuery features have no native equivalent** | Plugins, the chainable jQuery object, and the `fx` animation queue must be replaced with native patterns or small modules. |
| **Migration is incremental, not big‑bang** | Audit first, establish tests, replace one module or file at a time, and use feature flags. |
| **Helper scaffolding is acceptable during migration** | A thin `$` and `$$` helper can keep code readable; helpers are removed once migration completes. |
| **Tooling can enforce the migration** | `eslint-plugin-no-jquery` flags jQuery usage and suggests native replacements, preventing regressions. |

---

### Prerequisites

- Working knowledge of jQuery selectors, DOM manipulation, events, AJAX, and utility functions.
- Familiarity with modern JavaScript (`querySelector`, `classList`, `addEventListener`, `fetch`, `async/await`, `Promise`).
- Access to the codebase, a test suite, and a build tool that can remove the jQuery dependency.
- Awareness that some jQuery plugins have no direct native replacement and may require small modules or CSS‑only alternatives.

---

### Related Programming Areas

- **Legacy modernisation** — the broader discipline of updating old front‑end code safely.
- **DOM performance** — native APIs can be faster, but selector caching and batching still matter.
- **Security** — removing jQuery eliminates known jQuery CVEs and removes unsafe HTML sinks.
- **Build tooling** — bundlers and tree‑shaking can remove unused code once jQuery is gone.

---

### Core Concepts / Features (Index)

1. Selectors
2. DOM Manipulation
3. Events
4. AJAX
5. Effects
6. Utilities
7. Migration Patterns
8. Migration Tools
9. Migration Best Practices


## Core Concept 1: Selectors

### Definitions

**Core Definition**
Selector migration replaces jQuery's `$()` and `jQuery()` selection functions with native `document.querySelector()` and `document.querySelectorAll()`, and replaces traversal methods such as `.find()`, `.closest()`, `.parent()`, `.next()`, and `.prev()` with their native DOM counterparts.

**Technical Definition**
`document.querySelector(selector)` returns the **first** matching element or `null`. `document.querySelectorAll(selector)` returns a **static `NodeList`** of all matching elements; the `NodeList` supports `forEach` and can be converted to an array with `Array.from()` or spread syntax. jQuery's `$()` returns an empty jQuery object when no elements match, whereas `querySelector` returns `null` and `querySelectorAll` returns an empty `NodeList`. `getElementById`, `getElementsByClassName`, and `getElementsByTagName` are slightly faster but return **live** collections that change as the DOM changes; `querySelector*` is preferred unless a bottleneck has been measured. Descendant selection uses `element.querySelectorAll(selector)` instead of `.find()`, and ancestor traversal uses `element.closest(selector)` instead of `.closest()` .

**Beginner‑Friendly Explanation**
Selectors are how you find elements on the page. jQuery uses `$('.box')`; vanilla JavaScript uses `document.querySelectorAll('.box')`. The main differences are: `querySelector` gives you the first match, `querySelectorAll` gives you all matches in a list you can loop over, and if nothing matches, `querySelector` gives you `null` while jQuery gives you an empty object that does nothing. Traversal — finding a parent, sibling, or descendant — uses properties like `parentElement`, `nextElementSibling`, and methods like `closest()` and `querySelectorAll()` on the element itself.

---

### Purposes

- **To replace jQuery selection** with native APIs that are built into every modern browser.
- **To reduce the jQuery object overhead** of wrapping and chaining.
- **To make selection behaviour explicit** — `null` versus empty collection, static versus live collections.
- **To enable tree‑shaking** of jQuery from the bundle once selection is migrated.

---

### Syntax Rules and Structure

| jQuery | Vanilla JavaScript |
|---|---|
| `$('selector')` | `document.querySelectorAll('selector')` |
| `$('#id')` | `document.getElementById('id')` or `document.querySelector('#id')` |
| `$('.class')` | `document.querySelectorAll('.class')` |
| `$el.find('li')` | `el.querySelectorAll('li')` |
| `$el.closest(selector)` | `el.closest(selector)` |
| `$el.parent()` | `el.parentElement` |
| `$el.next()` | `el.nextElementSibling` |
| `$el.prev()` | `el.previousElementSibling` |
| `$el.siblings()` | `[...el.parentElement.children].filter(c => c !== el)` |

---

### Constraints and Limitations

- **`querySelector` returns `null`** when no element matches; always guard before using the result.
- **`querySelectorAll` returns a static `NodeList`** ; it does not update when the DOM changes, unlike live `HTMLCollection`s from `getElementsBy*`.
- **`getElementById` is faster** for ID lookups but returns a single element, not a collection.
- **jQuery's `:contains()` and other custom pseudo‑selectors have no native equivalent** and must be replaced with filtering logic.

---

### Annotated Code Example

```javascript
// ❌ jQuery
var $items = $('.item');
$items.each(function() {
  $(this).addClass('active');
});

// ✅ Vanilla JavaScript
document.querySelectorAll('.item').forEach(function(item) {
  item.classList.add('active');
});

// ✅ Scoped selection (replaces $el.find())
var container = document.querySelector('.container');
container.querySelectorAll('.item').forEach(function(item) {
  item.classList.add('active');
});

// ✅ Closest (replaces $el.closest())
var button = document.querySelector('.btn');
var card = button.closest('.card');
```

**Expected Output**
Both versions add the `active` class to every `.item` element. The vanilla version uses `querySelectorAll` and `forEach`; the scoped version restricts the search to `.container`; the `closest` version finds the nearest ancestor `.card`.

**Why this output occurs**: `querySelectorAll` returns a static `NodeList` that supports `forEach`. `classList.add` is the native equivalent of `.addClass()`. `closest` walks up the ancestor chain until it finds a match or reaches `null`.

---

### Real‑World Cases

- **Navigation menus**: Replacing `$('.nav a')` with `document.querySelectorAll('.nav a')`.
- **Form widgets**: Replacing `$el.find('input')` with `el.querySelectorAll('input')`.
- **Event delegation**: Replacing `$el.closest('.card')` with `event.target.closest('.card')`.

---

## Core Concept 2: DOM Manipulation

### Definitions

**Core Definition**
DOM manipulation migration replaces jQuery's `.addClass()`, `.removeClass()`, `.toggleClass()`, `.text()`, `.html()`, `.attr()`, `.css()`, `.append()`, `.prepend()`, `.remove()`, and related methods with native `classList`, `textContent`, `innerHTML`, `getAttribute`/`setAttribute`, `style`, `append`, `prepend`, `insertAdjacentHTML`, and `remove`.

**Technical Definition**
`classList` provides `add`, `remove`, `toggle`, and `contains` for class manipulation. `textContent` gets or sets text content and is the safe alternative to `.text()`. `innerHTML` gets or sets HTML content and carries the same XSS risk as `.html()`; untrusted data must be sanitised or rendered with `textContent`. `getAttribute` and `setAttribute` replace `.attr()`, and `dataset` accesses `data-*` attributes. `element.append()` and `element.prepend()` accept DOM nodes or strings (strings are inserted as text, not parsed as HTML); `insertAdjacentHTML()` parses HTML strings at a specified position. `element.remove()` replaces `.remove()`.

**Beginner‑Friendly Explanation**
DOM manipulation is how you change what is on the page — adding classes, changing text, inserting elements, removing elements. jQuery gives you methods like `.addClass()` and `.append()`; vanilla JavaScript gives you `classList.add()` and `append()`. The most important difference is safety: `.html()` and `innerHTML` parse strings as HTML, which can run scripts. If the data comes from a user, use `.text()` / `textContent` instead.

---

### Purposes

- **To replace jQuery's manipulation helpers** with native APIs.
- **To reduce the risk of XSS** by preferring `textContent` over `innerHTML` for untrusted data.
- **To improve performance** by avoiding jQuery object creation for simple changes.
- **To make DOM changes explicit** and easier to reason about.

---

### Syntax Rules and Structure

| jQuery | Vanilla JavaScript |
|---|---|
| `$el.addClass('x')` | `el.classList.add('x')` |
| `$el.removeClass('x')` | `el.classList.remove('x')` |
| `$el.toggleClass('x')` | `el.classList.toggle('x')` |
| `$el.hasClass('x')` | `el.classList.contains('x')` |
| `$el.text('x')` | `el.textContent = 'x'` |
| `$el.html('<p>x</p>')` | `el.innerHTML = '<p>x</p>'` |
| `$el.attr('x', 'y')` | `el.setAttribute('x', 'y')` |
| `$el.data('x')` | `el.dataset.x` |
| `$el.css('color', 'red')` | `el.style.color = 'red'` |
| `$parent.append(node)` | `parent.append(node)` |
| `$parent.prepend(node)` | `parent.prepend(node)` |
| `$el.remove()` | `el.remove()` |
| `$el.append('<li>x</li>')` | `parent.insertAdjacentHTML('beforeend', '<li>x</li>')` |

---

### Constraints and Limitations

- **`innerHTML` is an XSS sink**; never pass untrusted data to it without sanitisation.
- **`append()` with a string inserts text**, not HTML; use `insertAdjacentHTML` to parse HTML.
- **`classList.toggle()` does not accept a second argument in older browsers**; modern browsers support the force parameter.

---

### Annotated Code Example

```javascript
// ❌ jQuery
$('.box').addClass('active');
$('.box').text('Hello');
$('.box').append('<span>World</span>');

// ✅ Vanilla JavaScript
document.querySelectorAll('.box').forEach(function(box) {
  box.classList.add('active');
  box.textContent = 'Hello';
});

// For HTML insertion, use insertAdjacentHTML with trusted HTML
document.querySelector('.box').insertAdjacentHTML('beforeend', '<span>World</span>');
```

**Expected Output**
Every `.box` receives the `active` class and the text “Hello”. The selected `.box` gains a `<span>World</span>` child.

**Why this output occurs**: `classList.add` adds the class; `textContent` sets the text safely; `insertAdjacentHTML` parses the trusted HTML string and inserts it at the end of the element.

---

### Real‑World Cases

- **Form feedback**: Setting `textContent` instead of `.html()` to avoid XSS.
- **List rendering**: Using `insertAdjacentHTML('beforeend', html)` to append rows efficiently.
- **Class toggling**: Replacing `.toggleClass('open')` with `classList.toggle('open')`.

---

## Core Concept 3: Events

### Definitions

**Core Definition**
Event migration replaces jQuery's `.on()`, `.off()`, `.click()`, `.submit()`, and related methods with `addEventListener`, `removeEventListener`, and `dispatchEvent`, and replaces jQuery's delegated `.on(selector, handler)` with native delegation using `event.target.closest()`.

**Technical Definition**
`element.addEventListener(type, handler, options)` binds an event handler. Unlike jQuery's `.on()`, native `addEventListener` does not support namespacing or automatic delegation. Delegation requires listening on a stable ancestor and using `event.target.closest(selector)` to determine whether the event originated from a matching descendant. `dispatchEvent(new Event('name'))` replaces `.trigger()`. jQuery's `.off()` has no direct native equivalent; the handler function must be stored and passed to `removeEventListener`. Passive listeners (`{ passive: true }`) can be used for scroll and touch events to improve performance.

**Beginner‑Friendly Explanation**
Events are how you respond to clicks, typing, and other user actions. jQuery uses `.on('click', handler)`; vanilla JavaScript uses `addEventListener('click', handler)`. The big difference is delegation: jQuery lets you say “listen on this parent for clicks on `.child` elements”, including future ones. Native JavaScript does not have that built in, but you can achieve the same effect by listening on the parent and checking `event.target.closest('.child')`.

---

### Purposes

- **To replace jQuery's event system** with the native DOM event API.
- **To implement delegation** using `closest()` for dynamic elements.
- **To use passive listeners** for scroll and touch events to improve performance.
- **To remove the jQuery event namespace dependency** and adopt explicit handler references.

---

### Syntax Rules and Structure

| jQuery | Vanilla JavaScript |
|---|---|
| `$el.on('click', fn)` | `el.addEventListener('click', fn)` |
| `$el.off('click', fn)` | `el.removeEventListener('click', fn)` |
| `$(parent).on('click', '.child', fn)` | `parent.addEventListener('click', e => { const t = e.target.closest('.child'); if (!t) return; fn.call(t, e); })` |
| `$el.trigger('event')` | `el.dispatchEvent(new Event('event'))` |
| `$el.click()` | `el.click()` (native method exists) |

---

### Constraints and Limitations

- **No namespacing**; store handler references to remove them later.
- **`closest()` may return `null`** ; always guard before acting.
- **Passive listeners cannot call `preventDefault()`** ; use `{ passive: false }` when prevention is required.

---

### Annotated Code Example

```javascript
// ❌ jQuery delegated handler
$('#list').on('click', '.item', function() {
  console.log('Clicked:', $(this).text());
});

// ✅ Native delegation with closest
document.getElementById('list').addEventListener('click', function(event) {
  var item = event.target.closest('.item');
  if (!item) return;
  console.log('Clicked:', item.textContent);
});
```

**Expected Output**
Clicking any `.item` inside `#list` logs its text, including items added after the listener was bound.

**Why this output occurs**: The listener is on `#list`; when a click bubbles up, `closest('.item')` finds the nearest matching ancestor (or the target itself). If no match is found, the handler returns early.

---

### Real‑World Cases

- **Dynamic lists**: Delegating clicks on items added by AJAX or rendering.
- **Form submission**: `addEventListener('submit', fn)` on the form.
- **Scroll performance**: `{ passive: true }` on scroll listeners.

---

## Core Concept 4: AJAX

### Definitions

**Core Definition**
AJAX migration replaces `$.ajax()`, `$.get()`, `$.post()`, and `$.getJSON()` with the native `fetch()` API, accounting for `fetch`'s different error‑handling model and its Promise‑based interface.

**Technical Definition**
`fetch(url, options)` returns a Promise that resolves to a `Response` object. **`fetch` does not reject on HTTP error statuses (4xx/5xx)** ; it only rejects on network failure. The response must be checked with `response.ok` (status 200–299) before parsing the body. jQuery's `$.ajax()` auto‑parses JSON and triggers `.fail()` on HTTP errors; `fetch` requires manual status checking and explicit `.json()` parsing. `fetch` also does not support request timeouts natively; `AbortController` is used for cancellation. CSRF tokens and credentials must be sent explicitly via headers or `credentials: 'same-origin'`.

**Beginner‑Friendly Explanation**
AJAX is how your page talks to the server without reloading. jQuery's `$.ajax()` is convenient: it parses JSON for you and treats HTTP errors as failures. `fetch` is the modern built‑in alternative, but it has two gotchas: it does not treat a 404 or 500 as an error (you have to check `response.ok` yourself), and you have to call `.json()` to read the response body. Once you know those differences, `fetch` with `async/await` is cleaner and more powerful than jQuery AJAX.

---

### Purposes

- **To replace jQuery AJAX** with the native `fetch` API.
- **To understand and handle the different error model** (HTTP errors versus network errors).
- **To use `async/await`** for readable asynchronous code.
- **To send CSRF tokens and credentials correctly** in frameworks such as Laravel and WordPress.

---

### Syntax Rules and Structure

```javascript
// ✅ fetch with error handling
async function getData(url) {
  const response = await fetch(url, {
    headers: { 'Accept': 'application/json' },
    credentials: 'same-origin'
  });
  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new Error(error.message || `HTTP ${response.status}`);
  }
  return response.json();
}
```

---

### Constraints and Limitations

- **`fetch` does not reject on HTTP 4xx/5xx**; check `response.ok`.
- **No built‑in timeout**; use `AbortController` with `setTimeout`.
- **Response body parsing is explicit**; call `.json()`, `.text()`, or `.blob()`.

---

### Annotated Code Example

```javascript
// ❌ jQuery AJAX
$.ajax({
  url: '/api/data',
  method: 'GET',
  dataType: 'json',
  success: function(data) { console.log(data); },
  error: function(xhr) { console.error(xhr.status); }
});

// ✅ fetch with async/await
async function loadData() {
  try {
    const response = await fetch('/api/data', {
      headers: { 'Accept': 'application/json' },
      credentials: 'same-origin'
    });
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error.message);
  }
}
```

**Expected Output**
Both versions fetch JSON data and log it on success. The jQuery version routes HTTP errors to `error`; the `fetch` version checks `response.ok` and throws manually, then catches both HTTP and network errors in the same `catch`.

**Why this output occurs**: `fetch` resolves on any HTTP response, so `response.ok` is required to distinguish success from failure. The `try/catch` handles both the manually thrown HTTP error and any network rejection.

---

### Real‑World Cases

- **Laravel CSRF**: Sending `X-XSRF-TOKEN` from the `XSRF-TOKEN` cookie with `credentials: 'same-origin'`.
- **WordPress admin‑ajax**: Using `FormData` with `fetch` and `credentials: 'same-origin'`.
- **API clients**: Centralising `fetch` calls in a small service layer with consistent error handling.

---

## Core Concept 5: Effects

### Definitions

**Core Definition**
Effects migration replaces jQuery's `.fadeIn()`, `.fadeOut()`, `.slideDown()`, `.slideUp()`, `.animate()`, and `.delay()` with CSS transitions, CSS animations, and the Web Animations API (WAAPI), which are GPU‑accelerated and synchronised with the browser's rendering pipeline.

**Technical Definition**
jQuery's `.animate()` historically used `setTimeout`/`setInterval` on the CPU and did not use `requestAnimationFrame` until jQuery 3.0. CSS transitions and animations are handled by the browser's compositor and can be GPU‑accelerated when animating `transform` and `opacity`. The Web Animations API (`element.animate(keyframes, options)`) provides a JavaScript interface to the browser's animation engine, with performance comparable to CSS animations and more control than jQuery's `.animate()`. `web.dev` recommends CSS for simple one‑shot transitions and WAAPI for complex motion with pause, reverse, and orchestration.

**Beginner‑Friendly Explanation**
Effects are how you make things move, fade, or slide. jQuery's `.fadeIn()` and `.animate()` are convenient but run on the CPU and can be choppy. CSS transitions and the Web Animations API let the browser's GPU do the work, which is smoother and uses less battery. For simple fades and slides, CSS is easiest. For complex sequences — bounce, pause, reverse — WAAPI gives you the control without jQuery's performance penalty.

---

### Purposes

- **To replace jQuery effects** with CSS transitions and WAAPI.
- **To improve animation performance** by using GPU‑accelerated properties (`transform`, `opacity`).
- **To gain more control** over playback (pause, reverse, speed) with WAAPI.
- **To remove jQuery's animation queue** and simplify timing.

---

### Syntax Rules and Structure

| jQuery | Vanilla CSS/WAAPI |
|---|---|
| `$el.fadeIn()` | `el.style.opacity = 1` with CSS `transition: opacity 300ms` |
| `$el.slideDown()` | Toggle a class with CSS `transition: max-height` or `transform` |
| `$el.animate({ left: '100px' }, 300)` | `el.animate([{ left: '0' }, { left: '100px' }], 300)` |
| `$el.delay(500).fadeOut()` | `setTimeout(() => { el.style.opacity = 0 }, 500)` |

---

### Constraints and Limitations

- **CSS transitions are two‑state**; WAAPI supports multi‑keyframe sequences.
- **WAAPI animations are not automatically cancelled** on element removal; call `.cancel()` in cleanup.
- **Animating layout properties** (`width`, `height`, `top`, `left`) triggers reflow; prefer `transform` and `opacity`.

---

### Annotated Code Example

```javascript
// ❌ jQuery
$('.box').fadeOut(300);

// ✅ CSS transition
.box {
  transition: opacity 300ms ease;
}
.box.hidden {
  opacity: 0;
}
document.querySelector('.box').classList.add('hidden');

// ✅ Web Animations API (equivalent to jQuery .animate())
document.querySelector('.box').animate(
  [{ opacity: 1 }, { opacity: 0 }],
  { duration: 300, easing: 'ease', fill: 'forwards' }
);
```

**Expected Output**
The box fades to zero opacity over 300 ms in all three versions. The CSS version uses the browser's transition engine; the WAAPI version uses the browser's animation engine.

**Why this output occurs**: CSS transitions and WAAPI are handled by the browser's rendering pipeline, which can composite `opacity` changes on the GPU. jQuery's `.fadeOut()` historically drove the animation on the CPU with timers.

---

### Real‑World Cases

- **Modal transitions**: CSS `opacity` and `transform` transitions.
- **Complex sequences**: WAAPI for multi‑step animations with pause and reverse.
- **Scroll‑linked animation**: `requestAnimationFrame` for fine‑grained control.

---

## Core Concept 6: Utilities

### Definitions

**Core Definition**
Utility migration replaces jQuery's `$.each()`, `$.map()`, `$.grep()`, `$.inArray()`, `$.trim()`, `$.extend()`, `$.isArray()`, `$.parseJSON()`, and `$.now()` with native `Array.prototype.forEach()`, `map()`, `filter()`, `indexOf()`/`includes()`, `String.prototype.trim()`, `Object.assign()`/spread, `Array.isArray()`, `JSON.parse()`, and `Date.now()`.

**Technical Definition**
jQuery's utility functions predate standardised ES5 array methods. `$.each(array, fn)` maps to `array.forEach(fn)`; `$.each(object, fn)` requires `Object.entries(object).forEach()` or `Object.keys(object).forEach()`. `$.map()` maps to `Array.prototype.map()` but differs in null handling and flattening; `$.grep()` maps to `filter()`; `$.inArray()` maps to `indexOf()` or `includes()`. `$.trim()` is `String.prototype.trim()`. `$.extend()` deep‑merge behaviour must be replaced with `Object.assign()` (shallow) or a small deep‑merge utility. jQuery 4.0 removed several of these helpers, making native replacements mandatory for future upgrades.

**Beginner‑Friendly Explanation**
Utilities are small helper functions for looping, searching, and merging data. jQuery's `$.each()` and `$.map()` were useful before JavaScript had `forEach()` and `map()`. Now every modern browser has them built in, so you can replace `$.each(arr, fn)` with `arr.forEach(fn)`, `$.grep(arr, fn)` with `arr.filter(fn)`, and `$.trim(str)` with `str.trim()`. This removes the jQuery dependency for data manipulation and makes the code more standard.

---

### Purposes

- **To replace jQuery utility functions** with standard JavaScript methods.
- **To prepare for jQuery 4.0**, which removes several of these helpers.
- **To make data manipulation more readable** and consistent with modern JavaScript.
- **To reduce the surface area of jQuery usage** in the codebase.

---

### Syntax Rules and Structure

| jQuery | Vanilla JavaScript |
|---|---|
| `$.each(arr, fn)` | `arr.forEach(fn)` |
| `$.each(obj, fn)` | `Object.entries(obj).forEach(([k, v]) => fn(k, v))` |
| `$.map(arr, fn)` | `arr.map(fn)` |
| `$.grep(arr, fn)` | `arr.filter(fn)` |
| `$.inArray(val, arr)` | `arr.indexOf(val)` or `arr.includes(val)` |
| `$.trim(str)` | `str.trim()` |
| `$.isArray(val)` | `Array.isArray(val)` |
| `$.parseJSON(str)` | `JSON.parse(str)` |
| `$.extend(target, source)` | `Object.assign(target, source)` or `{ ...target, ...source }` |

---

### Constraints and Limitations

- **`$.map` flattens arrays and removes `null`** ; `Array.prototype.map` does neither. Use `.flatMap()` or `.filter(Boolean)` if needed.
- **`$.extend(true, …)` is a deep merge**; `Object.assign` is shallow. A small deep‑merge utility is required for nested objects.
- **`$.each` on objects** requires `Object.entries` or `Object.keys`.

---

### Annotated Code Example

```javascript
// ❌ jQuery
$.each(items, function(index, item) {
  console.log(index, item.name);
});
var names = $.map(items, function(item) { return item.name; });
var active = $.grep(items, function(item) { return item.active; });

// ✅ Vanilla JavaScript
items.forEach(function(item, index) {
  console.log(index, item.name);
});
var names = items.map(function(item) { return item.name; });
var active = items.filter(function(item) { return item.active; });
```

**Expected Output**
Both versions iterate over `items`, extract names, and filter active items. The native versions use the standard array methods that are built into every modern browser.

**Why this output occurs**: `forEach`, `map`, and `filter` are ES5 array methods that provide the same iteration, transformation, and filtering behaviour as jQuery's utilities, without the jQuery dependency.

---

### Real‑World Cases

- **Data processing**: Replacing `$.grep` with `filter` in list rendering.
- **Object iteration**: Replacing `$.each(obj, fn)` with `Object.entries(obj).forEach()`.
- **JSON parsing**: Replacing `$.parseJSON` with `JSON.parse`.

---

## Core Concept 7: Migration Patterns

### Definitions

**Core Definition**
Migration patterns are the structural approaches used to move from jQuery to vanilla JavaScript safely — auditing usage, establishing a utility layer, replacing leaf modules first, using feature flags, and removing jQuery only after all usage is migrated.

**Technical Definition**
The recommended pattern is **incremental strangling**: audit every jQuery call, create a small utility layer (`src/utils/dom.js`, `events.js`, `http.js`) that mimics the jQuery API using native methods, migrate leaf modules (functions with no dependents) first, run the old and new implementations in parallel behind feature flags, and remove the utility layer and jQuery once all call sites are migrated. The `kokil.com.np` guide describes phases: dual‑load behind a feature flag, rewrite one module as an ES module, replace leaf nodes first, and keep helpers as scaffolding until migration completes. A **bounded rewrite** of a small, isolated surface is preferable when the legacy code is fundamentally incompatible with incremental coexistence.

**Beginner‑Friendly Explanation**
Migration patterns are the “how” of the migration. You do not delete jQuery on day one. You start by listing every place jQuery is used. Then you build a small set of native helper functions that look like jQuery, so the rest of the code can be migrated one function at a time. You start with the functions that nothing else depends on (the leaves), test them, and move upward. You keep the old code running behind a feature flag until the new code is proven. When everything is migrated, you remove the helpers and the jQuery script tag.

---

### Purposes

- **To keep production stable** while the migration is in progress.
- **To make the migration measurable** by tracking the percentage of jQuery calls removed.
- **To allow rollback** at the module or feature‑flag level.
- **To avoid a big‑bang rewrite** that freezes feature development.

---

### Syntax Rules and Structure

| Phase | Activity |
|---|---|
| **Audit** | `grep` for `$`, `jQuery`, `.ajax`, `.on`, `.fn`; document each usage. |
| **Utility layer** | Create `dom.js`, `events.js`, `http.js` with native implementations. |
| **Leaf first** | Migrate functions with no dependents. |
| **Parallel run** | Old and new code behind a feature flag. |
| **Remove scaffolding** | Delete helpers and jQuery once all call sites are migrated. |

---

### Constraints and Limitations

- **The utility layer must not become a permanent jQuery clone**; remove it after migration.
- **Feature flags add complexity**; remove them once the migration is stable.
- **Some plugins cannot be migrated line‑for‑line**; isolate them behind a dynamic import until they are replaced.

---

### Annotated Code Example

```javascript
// ✅ Utility layer (temporary scaffolding)
const $ = (selector, context = document) => context.querySelector(selector);
const $$ = (selector, context = document) => [...context.querySelectorAll(selector)];

// Migrate a leaf module
function toggleMenu() {
  $$('.menu-item').forEach(item => {
    item.classList.toggle('open');
  });
}

// Once all call sites use native APIs, delete $ and $$ and remove jQuery
```

**Expected Output**
The `toggleMenu` function uses the temporary helpers to select and toggle classes. After all modules are migrated, the helpers and jQuery are removed.

**Why this output occurs**: The helpers provide a familiar syntax during the transition, but they are thin wrappers over native APIs, so removing them at the end is straightforward.

---

### Real‑World Cases

- **Laravel/Blade sites**: Dual‑load jQuery and native helpers behind a flag, migrate one Blade partial at a time.
- **WordPress themes**: Replace `admin-ajax.php` calls with `fetch` and `FormData`, then remove jQuery.
- **Single‑page applications**: Migrate leaf components first, then shared utilities, then the root.

---

## Core Concept 8: Migration Tools

### Definitions

**Core Definition**
Migration tools are the linters, codemods, audit scripts, and test harnesses that help identify jQuery usage, suggest native replacements, automate mechanical transformations, and verify that behaviour is preserved.

**Technical Definition**
`eslint-plugin-no-jquery` is the primary linter: it disallows jQuery functions and provides native equivalents in suggestions. Its configs include `recommended`, `deprecated` (all known deprecated and removed code), `slim` (excludes AJAX and effects), and `all` (all methods and utilities, for full removal). The `variablePattern` rule enforces `$`‑prefixed variables for jQuery objects, which is required for the method rules to work. A `grep` audit script searches for `$`, `jQuery`, `.ajax`, `.fn`, and plugin registrations. Codemods can automate simple replacements such as `.bind()` → `.on()` or `.trim()` → `String.prototype.trim()`. Tests (unit, integration, visual regression) are the safety net that confirms the migration preserved behaviour.

**Beginner‑Friendly Explanation**
Migration tools are the helpers that make the migration faster and safer. The linter (`eslint-plugin-no-jquery`) flags every jQuery call and tells you what to use instead. A `grep` script finds all the places jQuery is used so you can make a plan. Codemods can automatically replace simple patterns like `.bind()` with `.on()`. Tests confirm that nothing broke. Together, these tools turn a scary manual migration into a guided, verifiable process.

---

### Purposes

- **To find every jQuery usage** so that no call site is missed.
- **To prevent regressions** by flagging new jQuery usage during the migration.
- **To automate mechanical replacements** and reduce manual effort.
- **To verify behaviour** with tests before, during, and after the migration.

---

### Syntax Rules and Structure

```bash
# Audit: find all jQuery usage
grep -rE '\$\(|jQuery\(|\.ajax|\.fn\.' resources/js public/js --include='*.js'

# Lint: disallow jQuery and suggest native equivalents
npm install eslint-plugin-no-jquery --save-dev
# .eslintrc: "extends": "plugin:no-jquery/all"
```

---

### Constraints and Limitations

- **Linters only flag static usage**; dynamic property access (e.g., `$[method]()`) may be missed.
- **Codemods are not universally safe**; always review generated changes and run tests.
- **Tests are mandatory**; without them, the migration is guesswork.

---

### Annotated Code Example

```javascript
// .eslintrc.json
{
  "extends": "plugin:no-jquery/all",
  "plugins": ["no-jquery"]
}

// Terminal output:
// error  Avoid using jQuery's addClass()  no-jquery/no-add-class
//       Use classList.add() instead
// error  Avoid using jQuery's ajax()  no-jquery/no-ajax
//       Use fetch() instead
```

**Expected Output**
ESLint reports each jQuery usage with a suggested native replacement, preventing new jQuery code from being added and guiding the migration.

**Why this output occurs**: The `no-jquery/all` config enables rules for every jQuery method and utility. Each rule maps the method to its native equivalent and provides a fix suggestion.

---

### Real‑World Cases

- **CI gates**: Failing the build on new jQuery usage prevents regressions.
- **Migration dashboards**: Counting remaining `no-jquery` errors to measure progress.
- **Codemod-assisted refactoring**: Automating `.bind()` → `.on()` across a large codebase.

---

## Core Concept 9: Migration Best Practices

### Definitions

**Core Definition**
Migration best practices are the consolidated conventions — audit before changing, test before refactoring, migrate leaf modules first, use a temporary helper layer, run old and new code in parallel, and remove jQuery only after all call sites are migrated — that make the migration safe, measurable, and maintainable.

**Technical Definition**
Best practices synthesise the individual patterns: **(1) audit comprehensively** with `grep` and a spreadsheet of file, method, native equivalent, risk, and owner; **(2) establish tests** (characterisation tests, integration tests) before touching code; **(3) build a temporary utility layer** to keep the code readable during the transition; **(4) migrate leaf modules first** and work upward; **(5) run old and new code in parallel** behind feature flags; **(6) handle behavioural differences explicitly** (`fetch` error model, `querySelector` null, `querySelectorAll` static NodeList); **(7) use `textContent` over `innerHTML`** for untrusted data; **(8) remove the utility layer and jQuery** once all call sites are migrated; and **(9) enforce the end state with `eslint-plugin-no-jquery/all`** to prevent regressions.

**Beginner‑Friendly Explanation**
Best practices are the habits that keep the migration from turning into a disaster. Start by understanding what you have. Write tests before you change anything. Move one small piece at a time and test it. Keep the old code running until the new code is proven. Watch out for the differences between jQuery and native APIs — especially `fetch` error handling and `querySelector` returning `null`. Use `textContent` for user data to avoid XSS. When everything is migrated, delete the helpers and the jQuery script tag, and add a linter rule to make sure nobody adds it back.

---

### Purposes

- **To provide a single actionable checklist** for jQuery‑to‑vanilla migration.
- **To prevent the most common migration failures** (untested refactoring, behavioural regressions, incomplete migration).
- **To balance migration speed with production stability**.
- **To establish a baseline for code reviews and migration planning**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Audit** | `grep` + spreadsheet + risk levels | Starting without scope |
| **Tests** | Characterisation tests before refactoring | Refactoring untested code |
| **Order** | Leaf modules first | Starting with shared utilities |
| **Helpers** | Temporary `$`/`$$` scaffolding | Permanent jQuery clone |
| **Parallel run** | Feature flags | Big‑bang cutover |
| **Error handling** | Check `response.ok` with `fetch` | Assuming `fetch` rejects on 4xx/5xx |
| **Security** | `textContent` for untrusted data | `innerHTML` with user input |
| **Completion** | Remove helpers and jQuery; enforce with lint | Leaving scaffolding forever |

---

### Constraints and Limitations

- **Best practices are context‑dependent**; a small internal tool may not need feature flags.
- **The helper layer adds overhead**; remove it as soon as possible.
- **Some jQuery plugins have no native equivalent**; a small module or a retained isolated dependency may be the pragmatic choice.

---

### Annotated Code Example (Consolidated)

```javascript
// ✅ 1. Audit: grep for jQuery usage
// ✅ 2. Tests: add characterisation tests
// ✅ 3. Helper layer (temporary)
const $ = (sel, ctx = document) => ctx.querySelector(sel);
const $$ = (sel, ctx = document) => [...ctx.querySelectorAll(sel)];

// ✅ 4. Migrate a leaf module
function renderList(items) {
  const list = $('#list');
  list.innerHTML = '';                       // trusted template, not user data
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item.name;              // safe for untrusted data
    list.append(li);
  });
}

// ✅ 5. AJAX with fetch and error handling
async function loadItems() {
  const response = await fetch('/api/items', { credentials: 'same-origin' });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

// ✅ 6. Remove helpers and jQuery when done
// ✅ 7. Enforce with eslint-plugin-no-jquery/all
```

**Expected Output**
The list renders safely, AJAX handles HTTP errors explicitly, and the code uses temporary helpers that will be removed at the end of the migration.

**Why this output occurs**: Each step corresponds to a best practice: audit, test, helper layer, leaf‑first migration, explicit error handling, safe DOM rendering, and enforcement.

---

### Real‑World Cases

- **Laravel/Blade migration**: Audit with `grep`, migrate form partials first, use `fetch` with CSRF headers, remove jQuery when all partials are migrated.
- **WordPress theme modernisation**: Replace `admin-ajax.php` calls with `fetch` and `FormData`, migrate leaf widgets first, remove jQuery from the enqueue list.
- **CI enforcement**: `eslint-plugin-no-jquery/all` fails the build on new jQuery usage, preventing regression.

---

## References

- Migrate jQuery to Vanilla JavaScript | Aviator Runbooks — https://github.com/aviator-co/runbooks-library/blob/master/templates/migrate_jquery_to_vanilla_javascript.md
- You (Might) Don't Need jQuery | camsong — https://github.com/camsong/You-Dont-Need-jQuery
- jQuery to JavaScript Cheat Sheet | Zurkon — https://github.com/Zurkon/jquery-to-javascript-cheatsheet
- jQuery to Vanilla JS Migration Guide: Step‑by‑Step 2026 | Kokil Thapa — https://kokil.com.np/blog/jquery-to-vanilla-js-migration-guide
- jQuery to Modern JavaScript Migration Guide | techdebt.fail — https://techdebt.fail/migration-guides/jquery-to-modern/
- eslint-plugin-no-jquery | npm — https://www.npmjs.com/package/eslint-plugin-no-jquery
- CSS versus JavaScript animations | web.dev — https://web.dev/articles/css-vs-javascript
- Fetch vs jQuery AJAX error handling differences | Volcengine — https://www.volcengine.com/article/1677148
- jQuery 4.0 removed utility helpers | Telerik — https://www.telerik.com/blogs/jquery-4-0-0-support-whats-new-kendo-ui-has-you-covered
- DOM API for jQuery users | makandracards — https://makandracards.com/makandra/47447-dom-api-for-jquery-users
- Event delegation without jQuery | makandracards — https://makandracards.com/makandra/47673-event-delegation-without-jquery