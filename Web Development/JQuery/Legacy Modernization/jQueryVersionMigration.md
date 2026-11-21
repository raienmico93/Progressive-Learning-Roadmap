# jQuery Version Migration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery version migration is the disciplined process of moving an application from one major jQuery release line to another — 1.x → 2.x, 2.x → 3.x, or 3.x → 4.x — by identifying breaking changes, replacing removed or deprecated APIs, using jQuery Migrate as a diagnostic bridge, and removing the compatibility shim once all warnings are resolved.

**Technical Definition**
jQuery maintains **parallel release lines** for different browser-support targets: the 1.x line supports IE 6/7/8, the 2.x line drops those browsers while keeping the same API, the 3.x line introduces Promises/A+ Deferreds, removes previously deprecated APIs, and fixes security vulnerabilities, and the 4.x line drops IE ≤ 10, removes long-deprecated utility functions, adds Trusted Types/CSP support, and migrates the source to ES modules. Migration is **not merely a version bump**; each major release changes documented behaviour, removes APIs, or alters default semantics in ways that require code changes. jQuery Migrate is the official **diagnostic tool**: it restores removed APIs and logs `JQMIGRATE`-prefixed warnings to the console, allowing developers to find and fix each incompatibility before removing the shim. The migration workflow is iterative: add Migrate → fix warnings → remove Migrate → repeat for the next major version.

**Beginner‑Friendly Explanation**
jQuery has been around for a long time, and it has changed significantly over the years. A version migration is the process of moving your code from an old version to a newer one. The tricky part is that major version upgrades (like 2.x → 3.x or 3.x → 4.x) are **not fully backwards‑compatible**: some functions are removed, some behaviours change, and some plugins stop working. jQuery Migrate is a helper tool that brings the old functions back and prints warnings in the browser console whenever your code uses something that has changed. You use those warnings as a to‑do list: fix each one, then remove the helper. Skipping this process is risky because the old functions may disappear entirely in the next major release.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Parallel release lines** | 1.x, 2.x, 3.x, and 4.x coexist; 1.x supports IE 6/7/8, 2.x drops them, 3.x drops IE 8 and introduces Promises/A+, 4.x drops IE ≤ 10 and removes deprecated utilities. |
| **Major versions contain breaking changes** | 3.0 removed `.andSelf()`, `.selector`, `.context`, and changed `.then()` semantics; 4.0 removed `jQuery.trim`, `jQuery.isArray`, `jQuery.parseJSON`, `jQuery.now`, `jQuery.isFunction`, and others. |
| **jQuery Migrate is the diagnostic bridge** | It restores removed APIs and logs `JQMIGRATE` warnings; version compatibility is strict (1.x → Migrate 1.x, 2.x → 1.x, 3.x → 3.x, 4.x → 4.x). |
| **Warnings distinguish “deprecated” from “removed”** | “Removed” items must be changed before the code works without Migrate; “deprecated” items still work but should be changed eventually. |
| **Security patches can be breaking** | jQuery 3.5.0 changed `htmlPrefilter` behaviour to fix CVE‑2020‑11022/11023, which can break code relying on self‑closing tag normalisation. |
| **Migration is sequential** | You cannot jump directly from 1.x to 4.x; the official path is 1.x → 2.x → 3.x → 4.x, using the matching Migrate version at each step. |

---

### Prerequisites

- A working knowledge of the existing jQuery codebase, including plugin dependencies and any custom jQuery extensions.
- Access to the browser console (for `JQMIGRATE` warnings) and a staging environment.
- Familiarity with the target jQuery version’s upgrade guide (1.9, 3.0, 3.5, 4.0).
- A test suite or the ability to create one, so that behavioural changes are caught.

---

### Related Programming Areas

- **Dependency security** — outdated jQuery versions carry known CVEs (prototype pollution, XSS).
- **Plugin compatibility** — plugins may depend on removed APIs and must be updated or replaced.
- **Legacy code modernisation** — version migration is often the first step before framework migration.
- **Testing** — characterisation tests and console‑warning triage are the safety net for migration.

---

### Core Concepts / Features (Index)

1. jQuery 1.x to 2.x
2. jQuery 2.x to 3.x
3. jQuery 3.x to 4.x
4. jQuery Migrate
5. Deprecation Warnings
6. Breaking Changes
7. Migration Best Practices


## Core Concept 1: jQuery 1.x to 2.x

### Definitions

**Core Definition**
The 1.x → 2.x migration is the transition from the IE 6/7/8‑compatible jQuery line to the modern‑browser line, which keeps the same public API but removes all legacy IE workarounds and drops support for old Internet Explorer.

**Technical Definition**
jQuery 2.0 was released alongside jQuery 1.9 with the explicit goal of providing a **smaller, faster library for modern browsers** by removing the code that supported IE 6/7/8. The 1.x line continued to receive critical patches (1.11.x, 1.12.x) for applications that still needed old IE support. The API surface of 2.x is essentially the same as 1.x, so the migration is primarily a **browser‑support decision** rather than a code‑change exercise; however, the 1.9 release that preceded 2.0 did contain breaking changes (`.live()`, `.die()`, `.toggle(fn, fn)`, `jQuery.browser()` removed, and `.andSelf()` replaced by `.addBack()`), so applications coming from 1.8 or earlier must address those first. The recommended strategy was to use **conditional loading**: serve jQuery 1.x to old IE and jQuery 2.x to modern browsers via conditional comments, so the same codebase could support both.

**Beginner‑Friendly Explanation**
jQuery 2.x is basically jQuery 1.x with the old‑Internet‑Explorer baggage removed. The functions you use are the same, but the library is smaller and faster because it no longer contains workarounds for IE 6, 7, and 8. The migration is mostly about deciding whether you still need to support those ancient browsers. If you do, you stay on 1.x (or use conditional loading to serve 2.x to modern browsers and 1.x to old ones). If you do not, you can move to 2.x with very few code changes — but you should first make sure your code does not use the APIs that were removed in 1.9, such as `.live()`, `.die()`, and `jQuery.browser()`.

---

### Purposes

- **To reduce library size and improve performance** by removing legacy IE code paths.
- **To align the codebase with modern browser capabilities** without changing the public API.
- **To make an explicit browser‑support decision** rather than carrying dead compatibility code indefinitely.
- **To prepare for the 3.x line**, which builds on the modern‑browser foundation of 2.x.

---

### Syntax Rules and Structure

| Decision | Action |
|---|---|
| **Still need IE 6/7/8** | Stay on the latest 1.x (1.12.x) or use conditional loading with 1.x for old IE and 2.x for modern browsers. |
| **Do not need IE 6/7/8** | Upgrade to the latest 2.x, then proceed to 3.x. |
| **Coming from 1.8 or earlier** | First address the 1.9 breaking changes (`.live()`, `.die()`, `.toggle(fn, fn)`, `jQuery.browser()`, `.andSelf()`). |

**Conditional Loading Pattern** (historical):
```html
<!--[if lt IE 9]>
<script src="jquery-1.12.4.js"></script>
<![endif]-->
<!--[if gte IE 9]><!-->
<script src="jquery-2.2.4.js"></script>
<!--<![endif]-->
```

---

### Constraints and Limitations

- **IE 9 and IE 10 in “Compatibility View”** may behave like old IE and are not supported by 2.x.
- **Third‑party plugins may still contain IE‑specific code**; test them on modern browsers before switching.
- **The 1.9 breaking changes are easy to miss** if the application has not been updated since 1.8.

---

### Annotated Code Example

```javascript
// ❌ REMOVED IN 1.9 (must be fixed before moving to 2.x)
$('.item').live('click', handler);        // .live() removed
$('.item').die('click');                  // .die() removed
if ($.browser.msie) { … }                 // jQuery.browser() removed
$('.x').toggle(fn1, fn2);                 // .toggle(fn, fn) removed

// ✅ MODERN EQUIVALENTS
$(document).on('click', '.item', handler); // delegated .on()
$(document).off('click', '.item');         // .off()
// Feature detection instead of $.browser
if ('msie' in document.documentMode) { … } // or use Modernizr
$('.x').on('click', function() { … });    // toggle via class/state
```

**Expected Output**
The modern equivalents preserve the behaviour without relying on APIs that were removed in 1.9 and are absent from 2.x.

**Why this output occurs**: `.live()` was replaced by delegated `.on()`; `.die()` by `.off()`; `jQuery.browser()` by feature detection; and `.toggle(fn, fn)` by explicit state toggling.

---

### Real‑World Cases

- **TYPO3 CMS**: Upgraded from jQuery 1.x to 2.x after dropping support for IE < 9, removing the “dirty workarounds” that 1.x carried.
- **SAPUI5**: Removed code that referenced IE8‑specific jQuery APIs before moving to the 2.x line.

---

## Core Concept 2: jQuery 2.x to 3.x

### Definitions

**Core Definition**
The 2.x → 3.x migration is the transition to the release line that introduced Promises/A+‑compliant Deferreds, removed long‑deprecated APIs, dropped IE 8 support, and changed several documented behaviours in ways that can break existing code.

**Technical Definition**
jQuery 3.0 was the first major release after 2.x and contained a substantial set of breaking changes. The most consequential are: **Deferreds are now Promises/A+ compatible** (exceptions in `.then()` callbacks become rejection values, callbacks are always invoked asynchronously, and the resolution state is controlled by callbacks); **removed APIs** include `.andSelf()` (use `.addBack()`), `.selector` and `.context` properties, `jQuery.event.props`/`fixHooks`, and several undocumented internal APIs; **`jQuery.ajax()` special‑case Deferred methods were removed**; and **`jQuery.easing` additional‑parameter behaviour changed**. jQuery 3.0 also dropped IE 8 support (IE 9+ only). The official 3.0 Upgrade Guide and jQuery Migrate 3.x are the primary migration tools; Migrate restores removed behaviours and logs warnings so each incompatibility can be addressed.

**Beginner‑Friendly Explanation**
jQuery 3 is a bigger change than 2 was. The biggest difference is how “promises” work: the objects returned by AJAX calls and animations now behave like standard JavaScript promises, which means some code that relied on the old behaviour may break. Several old functions have been removed (like `.andSelf()`), and two properties (`.selector` and `.context`) that people used to read are gone. jQuery Migrate 3.x brings the old behaviours back and prints a warning whenever your code uses something that changed. You fix the warnings one by one, then remove Migrate. The official upgrade guide lists every change.

---

### Purposes

- **To adopt Promises/A+‑compliant Deferreds**, which behave consistently with native promises and `async/await`.
- **To remove deprecated APIs** that had been marked for removal since 1.9.
- **To drop IE 8 support** and reduce the library size.
- **To prepare for the 4.x line**, which builds on the 3.x API surface.

---

### Syntax Rules and Structure

| Breaking Change | jQuery 2.x | jQuery 3.x |
|---|---|---|
| **`.andSelf()`** | Available | Removed — use `.addBack()` |
| **`.selector` / `.context`** | Available | Removed |
| **`.then()` exceptions** | Bubble up, lock Deferred | Become rejection values |
| **`.then()` callback timing** | Sometimes synchronous | Always asynchronous |
| **`jQuery.ajax` Deferred special cases** | Present | Removed |
| **IE 8 support** | Yes (2.x) | No (IE 9+) |

---

### Constraints and Limitations

- **Promises/A+ changes can silently alter control flow**; test AJAX chains and animation sequences carefully.
- **Plugins that read `.selector` or `.context`** will receive `undefined` and may break.
- **The 3.x line has multiple minor releases** (3.0 → 3.5 → 3.7); each has its own upgrade notes, and 3.5.0 introduced a security‑driven change to `htmlPrefilter`.

---

### Annotated Code Example (Promises/A+ Change)

```javascript
// jQuery 2.x behaviour: an exception in a .then() callback
// bubbles up and locks the Deferred chain.
var d = $.Deferred();
d.then(function() { throw new Error('boom'); })
 .then(function() { console.log('not reached'); });

// jQuery 3.x behaviour: the exception becomes a rejection value,
// and the chain continues to the next rejection handler.
d.then(function() { throw new Error('boom'); })
 .catch(function(err) { console.log('caught:', err.message); });
```

**Expected Output**
In 3.x, the `.catch()` handler receives the error, and the chain is not locked. In 2.x, the exception would have bubbled up and the second `.then()` would not have run as expected.

**Why this output occurs**: jQuery 3.0 aligned Deferreds with the Promises/A+ specification, so thrown exceptions are converted into rejection values rather than escaping the chain.

---

### Real‑World Cases

- **SAPUI5**: Upgraded from jQuery 2.2.3 to 3.5.1 and used the jQuery 3.x compatibility layer to restore APIs that had changed incompatibly.
- **MediaWiki**: Upgraded to jQuery 3 in MediaWiki 1.29 and used jQuery Migrate 3 to temporarily restore removed features for a gradual transition.

---

## Core Concept 3: jQuery 3.x to 4.x

### Definitions

**Core Definition**
The 3.x → 4.x migration is the transition to the first major jQuery release in nearly a decade, which drops IE ≤ 10 support, removes long‑deprecated utility functions, adds Trusted Types/CSP support, migrates the source to ES modules, and changes several documented behaviours.

**Technical Definition**
jQuery 4.0.0 (released January 2026) is a **cleanup release** that removes legacy code and APIs the team had wanted to eliminate for years. The major breaking changes are: **IE ≤ 10 support removed** (IE 11 remains for now; removal planned for 5.0); **deprecated utility functions removed**, including `jQuery.trim`, `jQuery.isArray`, `jQuery.parseJSON`, `jQuery.now`, `jQuery.isFunction`, `jQuery.isWindow`, `jQuery.camelCase`, `jQuery.nodeName`, `jQuery.type`, and `jQuery.isNumeric`; **`jQuery.cssNumber` and `jQuery.cssProps` removed**; **`toggleClass( Boolean | undefined )` signature removed**; **`jQuery.ajax` JSON‑to‑JSONP auto‑promotion removed**; **scripts are no longer auto‑executed without `dataType: "script"`** ; **`focusin`/`focusout` shimming stopped**; and **`jQuery.Deferred.getStackHook` removed**. jQuery 4.0 also adds **Trusted Types support** for CSP compatibility, uses `<script>` tags for most async requests to avoid CSP errors, and migrates the source from AMD to ES modules. The 4.x Migrate plugin does **not** warn about behaviours removed in earlier major versions (1.0, 2.0, 3.0); you must first complete the earlier migrations.

**Beginner‑Friendly Explanation**
jQuery 4 is the biggest cleanup in jQuery’s history. It removes a lot of old utility functions that most modern code no longer needs, because JavaScript itself now provides `Array.isArray()`, `JSON.parse()`, `String.prototype.trim()`, and `Date.now()`. It also drops support for Internet Explorer 10 and older, and it adds support for modern security features like Trusted Types. If your code uses any of the removed functions, it will break on jQuery 4 — but the migration is usually straightforward: replace `$.trim()` with `String.prototype.trim()`, `$.parseJSON()` with `JSON.parse()`, and so on. jQuery Migrate 4.x will warn you about any remaining incompatibilities.

---

### Purposes

- **To eliminate long‑deprecated APIs** and reduce the library’s surface area.
- **To drop IE ≤ 10** and remove legacy browser workarounds, reducing the gzipped size by over 3 kB.
- **To add Trusted Types/CSP support** for modern security policies.
- **To migrate the source to ES modules** for modern build tooling.
- **To prepare for jQuery 5.0**, which will remove IE 11 support.

---

### Syntax Rules and Structure

| Removed in 4.0 | Native Replacement |
|---|---|
| `jQuery.trim(str)` | `str.trim()` |
| `jQuery.isArray(val)` | `Array.isArray(val)` |
| `jQuery.parseJSON(str)` | `JSON.parse(str)` |
| `jQuery.now()` | `Date.now()` |
| `jQuery.isFunction(fn)` | `typeof fn === 'function'` |
| `jQuery.isNumeric(val)` | `Number.isFinite(Number(val))` or custom check |
| `jQuery.type(val)` | `typeof` / `Object.prototype.toString` |
| `jQuery.nodeName(el, name)` | `el.nodeName.toLowerCase() === name` |
| `jQuery.camelCase(str)` | Custom utility or `str.replace(/-([a-z])/g, …)` |
| `jQuery.isWindow(obj)` | `obj != null && obj === obj.window` |

---

### Constraints and Limitations

- **The 4.x Migrate plugin does not cover 1.x/2.x/3.x removals**; you must complete the earlier migrations first.
- **`toggleClass(Boolean)` removal** can break code that passed a boolean to force the toggle.
- **`focusin`/`focusout` shimming removal** means the native events are used directly, which may expose browser differences.
- **Trusted Types support requires CSP configuration**; it is not automatic.

---

### Annotated Code Example (Removed Utility Replacement)

```javascript
// ❌ jQuery 3.x — removed in 4.0
var name = $.trim(userInput);
var isArr = $.isArray(value);
var data = $.parseJSON(jsonString);
var ts = $.now();

// ✅ jQuery 4.x — native replacements
var name = userInput.trim();
var isArr = Array.isArray(value);
var data = JSON.parse(jsonString);
var ts = Date.now();
```

**Expected Output**
The native replacements produce the same results without relying on jQuery utilities that no longer exist in 4.x.

**Why this output occurs**: jQuery 4.0 removed utility functions that duplicate standard JavaScript APIs; the native methods are the intended replacements.

---

### Real‑World Cases

- **Uninett/nav**: Upgraded to jQuery 4.0.0 by replacing `$.parseJSON()` with `JSON.parse()` and `$.trim()` with `String.prototype.trim()`, and added jQuery Migrate 4.0.2 as a shim for legacy plugins.
- **SAPUI5**: Documented the jQuery 2.x → 3.x breaking changes and provided a compatibility layer; the same pattern applies to 3.x → 4.x.
- **Kendo UI**: Published a jQuery 4.0.0 support guide covering the removed APIs and their native replacements.

---

## Core Concept 4: jQuery Migrate

### Definitions

**Core Definition**
jQuery Migrate is an official compatibility plugin that restores APIs removed in a major jQuery release and logs `JQMIGRATE` warnings to the browser console whenever deprecated or removed APIs are used, allowing developers to find and fix incompatibilities before removing the shim.

**Technical Definition**
jQuery Migrate is a **development and migration tool**, not a permanent dependency. When loaded after jQuery, it re‑adds removed behaviours and emits console warnings for each use. The development version logs detailed warnings (with stack traces); the production version is minified and does not generate warnings. Version compatibility is strict: jQuery 1.x uses Migrate 1.x, jQuery 2.x uses Migrate 1.x, jQuery 3.x uses Migrate 3.x, and jQuery 4.x uses Migrate 4.x. Migrate 4.x does **not** warn about behaviours removed in 1.0, 2.0, or 3.0; the official path is to complete each major migration sequentially (1.x → 2.x → 3.x → 4.x) using the matching Migrate version. All warnings start with `JQMIGRATE`; the full warning list is maintained in the project’s `warnings.md`.

**Beginner‑Friendly Explanation**
jQuery Migrate is a temporary helper that brings back the old functions your code still uses and prints a warning in the browser console every time you use one. Think of it as a checklist generator: every warning is a task you need to complete before you can safely remove the helper. The development version prints detailed warnings; the production version is silent, so you should use the development version during migration. You should never ship Migrate to production long‑term, because it keeps old, potentially vulnerable APIs alive and hides the fact that your code is not ready for the new version.

---

### Purposes

- **To restore removed APIs** so that legacy code continues to run during the migration.
- **To surface every incompatibility** through console warnings, turning a vague migration into a concrete to‑do list.
- **To provide a staged migration path** from one major version to the next.
- **To enable plugin compatibility** while plugins are updated or replaced.
- **To serve as a diagnostic tool**, not a permanent runtime dependency.

---

### Syntax Rules and Structure

#### Loading Migrate (Development)

```html
<script src="https://code.jquery.com/jquery-4.0.0.js"></script>
<script src="https://code.jquery.com/jquery-migrate-4.0.2.js"></script>
```

#### Inspecting Warnings Programmatically

```javascript
// All warnings are stored in jQuery.migrateMessages
console.log(jQuery.migrateMessages);
```

#### Warning Severity

| Warning Type | Meaning |
|---|---|
| **Removed** | Must be fixed before the code will work without Migrate. |
| **Deprecated** | Still works, but should be replaced; may be removed in a future version. |
| **Deprecated and removed** | Must be fixed before the code will work without Migrate. |

---

### Constraints and Limitations

- **Migrate is a development tool**, not a production dependency; shipping it long‑term defeats the purpose of the upgrade.
- **Version matching is mandatory**; using the wrong Migrate version produces misleading or missing warnings.
- **Migrate 4.x does not cover earlier removals**; you must migrate sequentially.
- **Migrate can hide security‑relevant API usage** (e.g., old HTML parsing behaviours) and keep vulnerable patterns alive.

---

### Annotated Code Example

```javascript
// Step 1: Load jQuery 3.x + Migrate 3.x in development
// Console output:
// JQMIGRATE: jQuery.fn.andSelf() is deprecated, use .addBack()

// Step 2: Fix the warning
// ❌ Old
$('.a').andSelf().addClass('x');
// ✅ New
$('.a').addBack().addClass('x');

// Step 3: Repeat until no warnings remain
// Step 4: Remove Migrate from the page
// Step 5: Verify the application still works
```

**Expected Output**
After fixing all warnings and removing Migrate, the console is clean and the application runs on the target jQuery version without the compatibility shim.

**Why this output occurs**: Each `JQMIGRATE` warning identifies a specific deprecated or removed API; replacing it with the recommended alternative eliminates the need for Migrate.

---

### Real‑World Cases

- **WordPress**: Loads jQuery Migrate next to jQuery and logs deprecation warnings for plugins that still use old shorthands like `.resize(fn)` and `.click(fn)`.
- **MediaWiki**: Used jQuery Migrate 3 to temporarily restore removed features during the jQuery 3 upgrade, with a gradual transition plan.
- **Uninett/nav**: Added jQuery Migrate 4.0.2 as a shim for legacy plugins (e.g., jquery-sparkline) during the jQuery 4 upgrade.

---

## Core Concept 5: Deprecation Warnings

### Definitions

**Core Definition**
Deprecation warnings are console messages emitted by jQuery Migrate (or by jQuery itself in some versions) that identify code using APIs that have been deprecated or removed, distinguishing between APIs that still work (“deprecated”) and APIs that no longer exist without the shim (“removed”).

**Technical Definition**
jQuery Migrate classifies warnings into two categories. **“Removed”** warnings (printed with `console.warn`) indicate that the API has been removed from the current jQuery version; the code will not work without Migrate and must be changed before Migrate is removed. **“Deprecated”** warnings (printed with `console.info`) indicate that the API still works but is no longer considered good practice and may be removed in a future version; the code should be updated eventually but does not require immediate change. All warnings start with the string `JQMIGRATE` and are stored in the `jQuery.migrateMessages` array. The full list of warnings is maintained in the project’s `warnings.md`.

**Beginner‑Friendly Explanation**
Not all warnings are equally urgent. A “removed” warning means your code is actually broken without the helper and must be fixed now. A “deprecated” warning means your code still works but is using something that will eventually disappear, so you should fix it when you have time. The warnings appear in the browser console while you use the development version of Migrate. You can think of them as a prioritised to‑do list: fix all the red “removed” items first, then work through the yellow “deprecated” items.

---

### Purposes

- **To prioritise migration work** by distinguishing must‑fix incompatibilities from should‑fix deprecations.
- **To identify the exact API and call site** that needs to change.
- **To track progress** toward a Migrate‑free codebase.
- **To avoid silent breakage** when Migrate is eventually removed.

---

### Syntax Rules and Structure

| Warning Prefix | Severity | Action |
|---|---|---|
| `JQMIGRATE: … is deprecated` | Informational | Plan to replace; not urgent. |
| `JQMIGRATE: … is removed` | Warning | Must fix before removing Migrate. |
| `JQMIGRATE: Migrate is installed` | Info | Normal startup message. |

---

### Constraints and Limitations

- **Warnings are only generated by the development version of Migrate**; the production version is silent.
- **Some warnings may be triggered by third‑party plugins**, not your own code; investigate the stack trace to find the source.
- **Deprecated warnings can accumulate**; triage them regularly rather than letting them pile up.

---

### Annotated Code Example

```text
JQMIGRATE: jQuery.fn.andSelf() is deprecated, use .addBack()
  at myScript.js:42
JQMIGRATE: jQuery.fn.live() is removed, use .on()
  at legacyPlugin.js:12
```

**Expected Output**
The first warning is informational (`.andSelf()` still works); the second is a must‑fix (`.live()` will not work without Migrate). The stack traces identify the call sites.

**Why this output occurs**: Migrate distinguishes removed APIs (which require the shim to function) from deprecated APIs (which still function but should be replaced).

---

### Real‑World Cases

- **WordPress plugin authors**: Use Migrate warnings to update plugins that still use `.resize(fn)` and `.click(fn)` shorthands.
- **MediaWiki gadget developers**: Use the distinction between “deprecated” and “deprecated and removed” to prioritise updates.

---

## Core Concept 6: Breaking Changes

### Definitions

**Core Definition**
Breaking changes are modifications in a new jQuery release that alter documented behaviour, remove public APIs, or change default semantics in ways that can cause existing code to stop working or behave differently without modification.

**Technical Definition**
Breaking changes are documented in each major release’s upgrade guide. The most consequential breaking changes across jQuery’s history are: **1.9** — `.live()`, `.die()`, `.toggle(fn, fn)`, and `jQuery.browser()` removed; `.andSelf()` replaced by `.addBack()`. **3.0** — Deferreds become Promises/A+ compliant; `.andSelf()`, `.selector`, and `.context` removed; `jQuery.ajax` special‑case Deferred methods removed; IE 8 support dropped. **3.5.0** — `htmlPrefilter` no longer normalises self‑closing tags, which can break code relying on the old behaviour; this was a security fix for CVE‑2020‑11022/11023. **4.0** — IE ≤ 10 support removed; utility functions (`$.trim`, `$.isArray`, `$.parseJSON`, `$.now`, `$.isFunction`, `$.isWindow`, `$.camelCase`, `$.nodeName`, `$.type`, `$.isNumeric`) removed; `toggleClass(Boolean)` signature removed; JSON‑to‑JSONP auto‑promotion removed; scripts no longer auto‑executed without `dataType: "script"`; `focusin`/`focusout` shimming stopped.

**Beginner‑Friendly Explanation**
A breaking change is a change that breaks code that used to work. jQuery tries to keep these to major releases, but sometimes a security fix (like the 3.5.0 `htmlPrefilter` change) forces a breaking change in a minor release. The official upgrade guides list every breaking change, and jQuery Migrate warns you when your code hits one. The most important thing to know is that breaking changes are **not optional** — they are the reason the migration exists. You cannot ignore them and expect the new version to work.

---

### Purposes

- **To understand exactly what will break** before upgrading, so that the migration can be planned.
- **To distinguish breaking changes from mere deprecations**, which do not require immediate action.
- **To use the upgrade guides and Migrate warnings as the authoritative source** for what needs to change.
- **To avoid shipping code that silently relies on removed behaviour**.

---

### Syntax Rules and Structure

| Release | Breaking Change | Required Action |
|---|---|---|
| **1.9** | `.live()`, `.die()`, `.toggle(fn, fn)`, `$.browser()` removed | Use `.on()`, `.off()`, class toggling, feature detection |
| **3.0** | `.andSelf()` removed; `.selector`/`.context` removed; Deferreds Promises/A+ | Use `.addBack()`; store selectors in variables; test promise chains |
| **3.5.0** | `htmlPrefilter` no longer normalises self‑closing tags | Use correct HTML markup; Migrate 3.4.0+ can restore old behaviour |
| **4.0** | IE ≤ 10 removed; utility functions removed; `toggleClass(Boolean)` removed | Use native APIs (`trim`, `Array.isArray`, `JSON.parse`, `Date.now`); use `classList.toggle` |

---

### Constraints and Limitations

- **Breaking changes in minor releases (e.g., 3.5.0)** are rare but can happen for security reasons.
- **Plugins may depend on removed APIs**; they must be updated or replaced.
- **Some breaking changes are intentional bug fixes** (e.g., `htmlPrefilter`) and the old behaviour should not be restored long‑term.

---

### Annotated Code Example (3.5.0 `htmlPrefilter`)

```javascript
// ❌ jQuery < 3.5.0: self-closing tags were normalised
$('#container').html('<div/><span>text</span>');
// Old behaviour: <div></div><span>text</span>

// ✅ jQuery 3.5.0+: self-closing tags are not normalised
$('#container').html('<div><span>text</span></div>');
// Use correct HTML markup; the browser parses it according to the HTML spec.
```

**Expected Output**
The 3.5.0+ behaviour follows the HTML specification: non‑void elements are not self‑closing, and the browser nests subsequent elements accordingly. Code that relied on the old normalisation must use explicit closing tags.

**Why this output occurs**: jQuery 3.5.0 removed the `htmlPrefilter` regex that rewrote self‑closing tags, as part of the fix for CVE‑2020‑11022/11023.

---

### Real‑World Cases

- **SAPUI5**: Documented the `htmlPrefilter` change and the `jQuery.easing` change as breaking differences between its jQuery 2.2.3 variant and jQuery 3.5.1.
- **Uninett/nav**: Replaced removed jQuery 4 utility functions with native APIs and added Migrate 4.0.2 for legacy plugins.

---

## Core Concept 7: Migration Best Practices

### Definitions

**Core Definition**
jQuery version migration best practices are the consolidated conventions — migrate sequentially, use the matching Migrate version, fix “removed” warnings first, test after each step, and remove Migrate only when the console is clean — that make the upgrade safe, traceable, and complete.

**Technical Definition**
Best practices synthesise the individual techniques: **(1) migrate sequentially** — 1.x → 2.x → 3.x → 4.x; you cannot jump directly from 1.x to 4.x. **(2) Use the matching Migrate version** — 1.x/2.x → Migrate 1.x, 3.x → Migrate 3.x, 4.x → Migrate 4.x. **(3) Fix “removed” warnings before “deprecated” warnings** — the former are must‑fix, the latter are should‑fix. **(4) Test after each step** — upgrade, add Migrate, fix warnings, remove Migrate, run tests, then move to the next major version. **(5) Update plugins** — later plugin versions are usually the most compatible with recent jQuery versions. **(6) Do not ship Migrate to production long‑term** — it hides incompatibilities and keeps old, potentially vulnerable APIs alive. **(7) Use the official upgrade guides** (1.9, 3.0, 3.5, 4.0) as the authoritative reference for breaking changes.

**Beginner‑Friendly Explanation**
Migration best practices are the habits that keep a version upgrade from becoming a disaster. Move one major version at a time — do not try to jump from jQuery 1 to 4. Use the right Migrate version for the version you are on. Fix the “removed” warnings first, because those are the ones that will actually break your code. Test after each step, so you know which change caused which problem. Update your plugins, because old plugins often depend on removed APIs. And do not leave Migrate in production forever — it is a temporary tool, not a permanent solution.

---

### Purposes

- **To provide a single actionable checklist** for jQuery version migration.
- **To prevent the most common migration failures** (jumping versions, using the wrong Migrate version, ignoring warnings).
- **To make the migration traceable and testable** at each step.
- **To ensure that the final state is Migrate‑free and runs on the target version.**

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Migration path** | Sequential: 1.x → 2.x → 3.x → 4.x | Jumping from 1.x to 4.x |
| **Migrate version** | Matching the jQuery version | Wrong Migrate version |
| **Warning triage** | Fix “removed” warnings first | Ignoring warnings |
| **Testing** | Test after each major version | Testing only at the end |
| **Plugins** | Update to latest compatible versions | Keeping old plugins |
| **Production** | Remove Migrate when console is clean | Shipping Migrate long‑term |
| **Reference** | Official upgrade guides (1.9, 3.0, 3.5, 4.0) | Relying on memory or guesswork |

---

### Constraints and Limitations

- **Sequential migration takes longer** than a direct jump, but it is the only supported path.
- **Some plugins may never be updated**; they must be replaced or isolated.
- **Migrate cannot fix code that relies on undocumented behaviour**; manual investigation is required.

---

### Annotated Code Example (Sequential Migration)

```text
✅ Step 1: Upgrade to latest 1.x or 2.x
✅ Step 2: Add jQuery Migrate 1.x
✅ Step 3: Fix all JQMIGRATE warnings
✅ Step 4: Remove Migrate 1.x
✅ Step 5: Upgrade to latest 3.x
✅ Step 6: Add jQuery Migrate 3.x
✅ Step 7: Fix all JQMIGRATE warnings
✅ Step 8: Remove Migrate 3.x
✅ Step 9: Upgrade to 4.x
✅ Step 10: Add jQuery Migrate 4.x
✅ Step 11: Fix all JQMIGRATE warnings
✅ Step 12: Remove Migrate 4.x
✅ Step 13: Run the full test suite
```

**Expected Output**
The application moves through each major version in sequence, with all incompatibilities resolved before moving to the next, and ends on jQuery 4.x without Migrate.

**Why this output occurs**: Each step addresses the breaking changes introduced by that specific major release, using the matching Migrate version as the diagnostic tool.

---

### Real‑World Cases

- **MediaWiki**: Upgraded to jQuery 3 in MediaWiki 1.29 and used jQuery Migrate 3 to temporarily restore removed features, with a plan to remove the shim once warnings were resolved.
- **WordPress**: Loads jQuery Migrate alongside jQuery and uses the warnings to identify plugins that still use deprecated shorthands.
- **Uninett/nav**: Upgraded to jQuery 4.0.0 by replacing removed utility functions and adding Migrate 4.0.2 for legacy plugins.

---

## References

- jQuery 1.9 final, jQuery 2.0 beta, Migrate final released | Official jQuery Blog — https://blog.jquery.com/2013/01/15/jquery-1-9-final-jquery-2-0-beta-migrate-final-released/
- jQuery 3.0 Release Candidate…Released! | Official jQuery Blog — http://blog.jquery.com/2016/05/20/jquery-3-0-release-candidate-released/
- jQuery Core 3.0 Upgrade Guide — https://jquery.com/upgrade-guide/3.0/
- jQuery 3.5.0 Released! | Official jQuery Blog — https://blog.jquery.com/2020/04/10/jquery-3-5-0-released/
- jQuery 4.0.0 | Official jQuery Blog — https://blog.jquery.com/2026/01/17/jquery-4-0-0/
- jQuery Core 4.0 Upgrade Guide — https://jquery.com/upgrade-guide/4.0/
- jQuery Migrate | GitHub — https://github.com/jquery/jquery-migrate/
- jQuery Migrate Plugin - Warning Messages — https://github.com/jquery/jquery-migrate/blob/main/warnings.md
- Changes of Note in jQuery 1.9 — https://jquery.com/upgrade-guide/1.9/
- jQuery 3.5 Upgrade Guide — https://jquery.com/upgrade-guide/3.5/
- jQuery 4.0.0 Support: What’s New + Kendo UI Has You Covered | Telerik — https://www.telerik.com/blogs/jquery-4-0-0-support-whats-new-kendo-ui-has-you-covered
- jQuery Migrate: A Practical Security Guide | Safeguard — https://safeguard.sh/resources/blog/jquery-migrate
- Upgrading jQuery: Working Towards a Healthy Web | Official jQuery Blog — https://blog.jquery.com/2024/04/17/upgrading-jquery-working-towards-a-healthy-web/
- jQuery 1.9 Upgrade Guide (Wikimedia) — https://lists.wikimedia.org/pipermail/wikitech-l/2013-December/073324.html
- SAPUI5: Upgrading from a Version Below 1.82 — https://help.sap.com/docs/SAPUI5/upgrading-from-a-version-below-1-82