# jQuery Deprecation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery deprecation is the lifecycle management process by which the jQuery team marks APIs as obsolete (“deprecated”), eventually removes them from the library (“removed”), and provides tooling — chiefly jQuery Migrate — to help developers identify, replace, and eliminate usage of those APIs before upgrading to a version where they no longer exist.

**Technical Definition**
jQuery maintains a **deprecation → removal pipeline** across major versions. An API is first marked **deprecated** (still functional, but discouraged and scheduled for future removal) and later **removed** (no longer present in the library; calling it throws an error unless a compatibility shim such as jQuery Migrate is loaded). The deprecation lifecycle is documented per method in the jQuery API documentation, which records the exact `version deprecated` and `version removed` for each API. Removal occurs almost exclusively in major releases (1.9, 3.0, 4.0), with rare exceptions for security‑driven changes in minor releases (e.g., `htmlPrefilter` in 3.5.0). The official migration path is **sequential**: 1.x → 2.x → 3.x → 4.x, using the matching jQuery Migrate version (1.x for jQuery 1‑2, 3.x for jQuery 3, 4.x for jQuery 4) at each step. jQuery Migrate restores removed APIs and logs `JQMIGRATE` warnings that distinguish **removed** items (must fix before removing Migrate) from **deprecated** items (should fix eventually but still work).

**Beginner‑Friendly Explanation**
jQuery has been evolving for two decades, and like any long‑lived software, some of its old features have been replaced by better alternatives. **Deprecation** is jQuery’s way of saying: “This feature still works, but we plan to remove it — please stop using it.” **Removal** is the point at which the feature is gone and your code will break if you have not migrated. The good news is that jQuery provides a helper called **jQuery Migrate** that brings the removed features back temporarily and prints warnings in the browser console every time your code uses something that has changed. Those warnings are your to‑do list: fix them one by one, then remove the helper. The migration is sequential — you move from 1.x to 2.x to 3.x to 4.x, not all at once — and the official upgrade guides list every change.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Deprecated ≠ removed** | Deprecated APIs still work (and may be safe to use short‑term); removed APIs throw errors without a shim. |
| **Removal is concentrated in major releases** | 1.9 removed `.live()`/`.die()`; 3.0 removed `.andSelf()`, `.selector`, `.context`, and `.size()`; 4.0 removed utility functions (`$.trim`, `$.isArray`, `$.parseJSON`, `$.now`, etc.). |
| **jQuery Migrate is the diagnostic bridge** | It restores removed APIs and logs `JQMIGRATE` warnings; version compatibility is strict. |
| **Warnings distinguish severity** | `JQMIGRATE` warnings prefixed with “is removed” are must‑fix; “is deprecated” are should‑fix. |
| **Security fixes can force minor‑release changes** | jQuery 3.5.0 changed `htmlPrefilter` to fix CVE‑2020‑11022/11023, breaking code that relied on self‑closing tag normalisation. |
| **Sequential migration is mandatory** | You cannot jump directly from 1.x to 4.x; the official path is 1.x → 2.x → 3.x → 4.x. |
| **Migrate is a development tool, not a production dependency** | Shipping Migrate long‑term hides incompatibilities and keeps removed, potentially vulnerable APIs alive. |

---

### Prerequisites

- A working knowledge of the existing jQuery codebase, including plugin dependencies and custom extensions.
- Access to the browser console (for `JQMIGRATE` warnings) and a staging environment.
- Familiarity with the target version’s upgrade guide (1.9, 3.0, 3.5, 4.0).
- A test suite or the ability to create one, so behavioural changes are caught.

---

### Related Programming Areas

- **jQuery version migration** — the upgrade process that deprecation makes necessary.
- **Dependency security** — outdated jQuery versions carry known CVEs.
- **Plugin compatibility** — plugins may depend on deprecated or removed APIs.
- **Legacy code modernisation** — deprecation cleanup is often the first step before framework migration.
- **Testing** — console‑warning triage and regression tests are the safety net.

---

### Core Concepts / Features (Index)

1. Deprecated Methods
2. Removed Methods
3. Deprecation Warnings
4. Migration Guides
5. Deprecation Best Practices


## Core Concept 1: Deprecated Methods

### Definitions

**Core Definition**
A deprecated jQuery method is a public API that the jQuery team has marked as obsolete and scheduled for removal in a future major release — it still works in the current version, but its use is discouraged and it will eventually stop functioning.

**Technical Definition**
Deprecation is recorded in the jQuery API documentation with a `version deprecated` field. A deprecated method remains callable and behaves as documented, but the team has announced its intention to remove it. Deprecated APIs fall into several categories: **event methods superseded by `.on()`/`.off()`** (`.bind()`, `.delegate()`, `.unbind()`, `.undelegate()`); **utility functions duplicated by native JavaScript** (`jQuery.parseJSON()`, `jQuery.unique()`, `jQuery.proxy()`); **animation internals** (`jQuery.fx.interval`); **selector internals** (`jQuery.expr[":"]`, `jQuery.expr.filters`); and **Deferred helpers** (`deferred.pipe()`). Deprecation does **not** mean the API is broken; it means the API is on the removal path. jQuery Migrate logs deprecated APIs with `console.info` (not `console.warn`), signalling lower urgency than removed APIs.

**Beginner‑Friendly Explanation**
A deprecated method is like a road that is still open but has a “closed for construction soon” sign. You can still drive on it today, but you should start planning an alternative route. In jQuery, `.bind()` is deprecated — it still works, but the modern `.on()` is preferred and `.bind()` will eventually disappear. Deprecated methods are not urgent, but they are technical debt: the longer you leave them, the more of them you accumulate, and the harder the eventual migration becomes.

---

### Purposes

- **To signal that an API is on the removal path** without breaking existing code immediately.
- **To provide a migration window** during which developers can replace deprecated APIs at their own pace.
- **To distinguish low‑urgency changes** (deprecated) from high‑urgency changes (removed).
- **To guide plugin authors** toward the modern API surface so their plugins remain compatible with future jQuery versions.
- **To reduce the long‑term maintenance burden** by gradually shrinking the library’s public surface.

---

### Syntax Rules and Structure

| Deprecated API | Version Deprecated | Modern Replacement |
|---|---|---|
| `.bind()` | 1.12 / 2.2 | `.on()` |
| `.delegate()` | 1.12 / 2.2 | `.on()` with selector |
| `.unbind()` | 1.12 / 2.2 | `.off()` |
| `.undelegate()` | 1.12 / 2.2 | `.off()` with selector |
| `jQuery.parseJSON()` | 3.0 | `JSON.parse()` |
| `jQuery.unique()` | 3.0 | `jQuery.uniqueSort()` |
| `jQuery.proxy()` | 3.3 | `Function.prototype.bind()` |
| `jQuery.isFunction()` | 3.3 | `typeof fn === "function"` |
| `deferred.pipe()` | 1.8 | `deferred.then()` |
| `jQuery.fx.interval` | 3.0 | No replacement (use `requestAnimationFrame`) |

---

### Constraints and Limitations

- **Deprecated methods still work**, but they may be slower or less safe than their replacements.
- **Deprecation does not guarantee a specific removal version**; the timeline is announced in upgrade guides.
- **Plugins may still depend on deprecated APIs**; updating the plugin is usually the best fix.

---

### Annotated Code Example

```javascript
// ❌ DEPRECATED: .bind() (still works in 3.x, removed in 4.0)
$('.btn').bind('click', handler);

// ✅ MODERN: .on()
$('.btn').on('click', handler);

// ❌ DEPRECATED: jQuery.parseJSON() (deprecated in 3.0, removed in 4.0)
var data = $.parseJSON(jsonString);

// ✅ MODERN: JSON.parse()
var data = JSON.parse(jsonString);
```

**Expected Output**
The deprecated versions still function in jQuery 3.x, but the modern versions are the supported API and will continue to work in jQuery 4.x.

**Why this output occurs**: Deprecated APIs are retained for a migration window; removed APIs are not. The modern replacements are the intended long‑term path.

---

### Real‑World Cases

- **WordPress plugins**: Many plugins still use `.bind()` and `.delegate()`; jQuery Migrate warns about these so plugin authors can update before jQuery 4 becomes the default.
- **MediaWiki gadgets**: Deprecated APIs such as `.size()` and `.andSelf()` are flagged by linting rules and Migrate warnings, prompting gadget authors to modernise.

---

## Core Concept 2: Removed Methods

### Definitions

**Core Definition**
A removed jQuery method is a public API that no longer exists in a given jQuery version — calling it throws a `TypeError` (`$(...).method is not a function`) unless a compatibility shim such as jQuery Migrate is loaded.

**Technical Definition**
Removal is the terminal stage of the deprecation pipeline. The jQuery API documentation records the exact `version removed` for each method. Removal is concentrated in major releases: **1.9** removed `.live()`, `.die()`, `.toggle(fn, fn)`, and `jQuery.browser()`; **3.0** removed `.andSelf()`, `.selector`, `.context`, `.size()`, `.load()`/`.unload()`/`.error()` shorthands, and the special‑case Deferred methods on `$.ajax`; **3.5.0** changed `htmlPrefilter` (a security‑driven removal of behaviour); **4.0** removed `jQuery.trim`, `jQuery.isArray`, `jQuery.parseJSON`, `jQuery.now`, `jQuery.isFunction`, `jQuery.isWindow`, `jQuery.isNumeric`, `jQuery.type`, `jQuery.camelCase`, `jQuery.nodeName`, `jQuery.cssNumber`, `jQuery.cssProps`, `jQuery.fx.interval`, `jQuery.Deferred.getStackHook`, `jQuery.event.global`, `jQuery.fn.push/sort/splice`, and the `toggleClass(Boolean)` signature. Removed methods require code changes; Migrate can temporarily restore them but should not be shipped to production long‑term.

**Beginner‑Friendly Explanation**
A removed method is a road that is no longer there. Your GPS (the code) still tries to take it, and you get an error. In jQuery, if you call `.live()` on jQuery 3.x, you get `$(...).live is not a function` because `.live()` was removed in 1.9. The fix is to replace it with the modern equivalent — in this case, delegated `.on()`. jQuery Migrate can bring removed methods back temporarily, but you must fix the code before you remove Migrate, or the application will break.

---

### Purposes

- **To eliminate dead APIs** and reduce the library’s size and maintenance surface.
- **To remove APIs that have insecure or inconsistent behaviour** (e.g., `htmlPrefilter` normalisation).
- **To align with native JavaScript standards** by removing utility functions that duplicate `Array.isArray`, `JSON.parse`, `Date.now`, etc.
- **To signal a hard cutover** that requires code changes rather than optional cleanup.
- **To enable future major releases** (e.g., jQuery 5.0 will remove IE 11 support).

---

### Syntax Rules and Structure

| Removed API | Version Removed | Replacement |
|---|---|---|
| `.live()` / `.die()` | 1.9 | `.on()` / `.off()` |
| `.toggle(fn, fn)` | 1.9 | Explicit state toggling |
| `jQuery.browser()` | 1.9 | Feature detection |
| `.andSelf()` | 3.0 | `.addBack()` |
| `.selector` / `.context` | 3.0 | Store selector in a variable |
| `.size()` | 3.0 | `.length` |
| `.load()` / `.unload()` / `.error()` shorthands | 3.0 | `.on("load", fn)` etc. |
| `jQuery.trim()` | 4.0 | `String.prototype.trim()` |
| `jQuery.isArray()` | 4.0 | `Array.isArray()` |
| `jQuery.parseJSON()` | 4.0 | `JSON.parse()` |
| `jQuery.now()` | 4.0 | `Date.now()` |
| `jQuery.isFunction()` | 4.0 | `typeof fn === "function"` |
| `jQuery.isNumeric()` | 4.0 | Custom check |
| `jQuery.type()` | 4.0 | `typeof` / `Object.prototype.toString` |
| `jQuery.nodeName()` | 4.0 | `el.nodeName.toLowerCase()` |
| `jQuery.camelCase()` | 4.0 | Custom utility |
| `jQuery.cssNumber` / `jQuery.cssProps` | 4.0 | Native CSS properties |
| `jQuery.fx.interval` | 4.0 | No replacement (uses rAF) |
| `jQuery.Deferred.getStackHook` | 4.0 | `jQuery.Deferred.getErrorHook` |
| `jQuery.event.global` | 4.0 | Remove usage |
| `jQuery.fn.push/sort/splice` | 4.0 | `.add()`, `.toArray()`, `.slice()` |
| `toggleClass(Boolean)` signature | 4.0 | `classList.toggle()` |

---

### Constraints and Limitations

- **Removed APIs throw errors** if called without Migrate.
- **Migrate only restores behaviour**; it does not make removed APIs good practice.
- **Some removals are security‑critical** and should not be reversed (e.g., `htmlPrefilter` normalisation).
- **Plugins may still depend on removed APIs**; they must be updated or replaced.

---

### Annotated Code Example

```javascript
// ❌ REMOVED in jQuery 3.0: .size()
var count = $('.item').size();

// ✅ MODERN: .length
var count = $('.item').length;

// ❌ REMOVED in jQuery 4.0: jQuery.trim()
var name = $.trim(userInput);

// ✅ MODERN: String.prototype.trim()
var name = userInput.trim();
```

**Expected Output**
The removed versions throw a `TypeError` in the target jQuery version; the modern versions work correctly.

**Why this output occurs**: Removed APIs are absent from the library; the replacement APIs are part of the core language or the supported jQuery surface.

---

### Real‑World Cases

- **Drupal 12 / jQuery 4**: Sites upgrading to jQuery 4 must replace `$.trim()` and `$.parseJSON()` with native equivalents, or the forms break.
- **Uninett/nav**: Replaced `$.parseJSON()` with `JSON.parse()` and `$.trim()` with `String.prototype.trim()` when upgrading to jQuery 4.0.0.

---

## Core Concept 3: Deprecation Warnings

### Definitions

**Core Definition**
Deprecation warnings are console messages emitted by jQuery Migrate (and in some cases by jQuery itself) that identify code using deprecated or removed APIs, with a stack trace pointing to the call site and a remediation hint.

**Technical Definition**
All jQuery Migrate warnings start with the string **`JQMIGRATE`** . The development (uncompressed) build logs warnings; the production build does not. Warnings are classified by severity: **“is removed”** (printed with `console.warn`) means the API no longer exists without Migrate and **must** be fixed before Migrate is removed; **“is deprecated”** (printed with `console.info`) means the API still works but should be replaced eventually. The full list of warnings is maintained in `warnings.md`. Developers can inspect the `jQuery.migrateMessages` array programmatically. Warnings are generated only when the deprecated code **actually executes**, so static searches complement runtime detection.

**Beginner‑Friendly Explanation**
Deprecation warnings are the browser console messages that say things like `JQMIGRATE: jQuery.trim() is removed; use String.prototype.trim`. They are your migration to‑do list. A “removed” warning is urgent — your code will break when Migrate is removed. A “deprecated” warning is less urgent but still worth fixing. The warnings only appear when the code actually runs, so you need to exercise your application’s features to see all of them.

---

### Purposes

- **To pinpoint the exact call site** of deprecated or removed APIs with a stack trace.
- **To distinguish must‑fix (“removed”) from should‑fix (“deprecated”)** changes.
- **To track migration progress** as the number of warnings decreases.
- **To catch incompatibilities that static searches might miss** (e.g., dynamic property access).
- **To provide remediation instructions** for each warning.

---

### Syntax Rules and Structure

| Warning Prefix | Severity | Action |
|---|---|---|
| `JQMIGRATE: … is removed` | Warning (`console.warn`) | Must fix before removing Migrate. |
| `JQMIGRATE: … is deprecated` | Info (`console.info`) | Should fix; still works. |
| `JQMIGRATE: Migrate is installed` | Info | Normal startup message. |

---

### Constraints and Limitations

- **Warnings only fire when the code executes**; static searches are required for dormant code paths.
- **Third‑party plugins may trigger warnings**; investigate the stack trace to find the source.
- **The production build is silent**; use the development build during migration.

---

### Annotated Code Example

```text
JQMIGRATE: jQuery.trim() is removed; use String.prototype.trim
  at legacyPlugin.js:42
JQMIGRATE: jQuery.fn.bind() is deprecated; use .on()
  at myScript.js:88
```

**Expected Output**
The first warning is must‑fix (`.trim()` removed in 4.0); the second is should‑fix (`.bind()` deprecated). The stack traces identify the call sites.

**Why this output occurs**: Migrate distinguishes removed APIs (which require the shim to function) from deprecated APIs (which still function but are discouraged).

---

### Real‑World Cases

- **MediaWiki gadget developers**: Use `JQMIGRATE` warnings to update gadgets that use `.size()` and `.andSelf()` before those APIs are removed.
- **WordPress plugin authors**: Use warnings to identify plugins that still use deprecated shorthands like `.click(fn)` and `.resize(fn)`.

---

## Core Concept 4: Migration Guides

### Definitions

**Core Definition**
jQuery migration guides are the official documentation — the upgrade guides for 1.9, 3.0, 3.5, and 4.0 — that list every breaking change, removed API, and behavioural modification in a release, along with the recommended replacements and the corresponding jQuery Migrate version.

**Technical Definition**
Each major release has a dedicated upgrade guide that serves as the authoritative reference for deprecation and removal. The guides are structured by module (Ajax, Attributes, Core, Deferred, Event, Manipulation, Selector, Traversing, etc.) and each entry is labelled as a **breaking change**, **feature**, or **deprecation**. The **3.0 Upgrade Guide** is particularly important because it documents the Promises/A+ change, the removal of `.andSelf()`, `.selector`, `.context`, and `.size()`, and the deprecation of `.bind()` and `.delegate()`. The **4.0 Upgrade Guide** documents the removal of utility functions, the IE ≤ 10 support drop, the `htmlPrefilter` change, and the `toggleClass(Boolean)` removal. The guides also specify the matching jQuery Migrate version: 1.x for jQuery 1‑2, 3.x for jQuery 3, 4.x for jQuery 4.

**Beginner‑Friendly Explanation**
Migration guides are the instruction manuals for upgrading jQuery. Each guide lists everything that changed in a release, including what was removed and what you should use instead. The guides are organised by topic, so you can look up the specific API you are using. They also tell you which version of jQuery Migrate to use. You do not need to memorise the guides; you use them as a reference when a warning or an error points you to a specific API.

---

### Purposes

- **To provide a complete, authoritative list of changes** for each major release.
- **To explain the rationale** behind each breaking change (security, standards alignment, native replacement).
- **To specify the correct jQuery Migrate version** for each upgrade path.
- **To guide sequential migration** (1.x → 2.x → 3.x → 4.x).
- **To serve as a reference during code reviews and migration planning**.

---

### Syntax Rules and Structure

| Guide | URL |
|---|---|
| jQuery 1.9 Upgrade Guide | `https://jquery.com/upgrade-guide/1.9/` |
| jQuery 3.0 Upgrade Guide | `https://jquery.com/upgrade-guide/3.0/` |
| jQuery 3.5 Upgrade Guide | `https://jquery.com/upgrade-guide/3.5/` |
| jQuery 4.0 Upgrade Guide | `https://jquery.com/upgrade-guide/4.0/` |
| jQuery Migrate warnings list | `https://github.com/jquery/jquery-migrate/blob/main/warnings.md` |

---

### Constraints and Limitations

- **The guides are large**; do not read them end‑to‑end unless you are planning a full migration.
- **Some changes are documented only in the blog post**, not the guide (e.g., minor behavioural changes).
- **Third‑party plugins may lag behind** the official guides; plugin documentation is also required.

---

### Annotated Code Example (Consulting the Guide for `.andSelf()`)

```text
From the jQuery 3.0 Upgrade Guide:
Breaking change: .andSelf() removed, use .addBack()
```

```javascript
// ❌ Removed in 3.0
$('.a').andSelf().addClass('x');

// ✅ Replacement per the guide
$('.a').addBack().addClass('x');
```

**Expected Output**
The guide provides the exact replacement, and the code works in jQuery 3.x+.

**Why this output occurs**: The upgrade guide is the authoritative source for the replacement API.

---

### Real‑World Cases

- **SAPUI5**: Uses the jQuery 3.0 upgrade guide to document the differences between its jQuery 2.2.3 variant and jQuery 3.5.1, including the `htmlPrefilter` and `jQuery.easing` changes.
- **MediaWiki**: References the upgrade guide and Migrate warnings to manage the jQuery 3 transition.

---

## Core Concept 5: Deprecation Best Practices

### Definitions

**Core Definition**
jQuery deprecation best practices are the consolidated conventions — migrate sequentially, match the Migrate version to the jQuery version, fix “removed” warnings first, test after each step, and remove Migrate only when the console is clean — that make the upgrade safe, traceable, and complete.

**Technical Definition**
Best practices synthesise the individual techniques: **(1) migrate sequentially** — 1.x → 2.x → 3.x → 4.x; do not jump from 1.x to 4.x. **(2) Use the matching Migrate version** — 1.x/2.x → Migrate 1.x, 3.x → Migrate 3.x, 4.x → Migrate 4.x. **(3) Fix “removed” warnings before “deprecated” warnings** — the former are must‑fix. **(4) Test after each major version** — upgrade, add Migrate, fix warnings, remove Migrate, run tests, then proceed. **(5) Update plugins** — later plugin versions are usually the most compatible. **(6) Do not ship Migrate to production long‑term** — it hides incompatibilities and keeps removed APIs alive. **(7) Use static searches** (`grep` for `$.trim`, `$.isArray`, `.bind()`, etc.) to find dormant code paths that Migrate cannot detect. **(8) Consult the official upgrade guides** for each release.

**Beginner‑Friendly Explanation**
Best practices are the habits that keep a jQuery upgrade from becoming a disaster. Move one major version at a time. Use the right Migrate version. Fix the urgent “removed” warnings first. Test after each step. Update your plugins. Do not leave Migrate in production forever. And search your code for old APIs even if the warnings do not show up, because some code may not run during your testing. These habits turn a risky upgrade into a controlled, manageable process.

---

### Purposes

- **To provide a single actionable checklist** for jQuery deprecation management.
- **To prevent the most common upgrade failures** (jumping versions, wrong Migrate version, ignoring warnings).
- **To make the migration traceable and testable** at each step.
- **To ensure the final state is Migrate‑free and runs on the target version.**

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Migration path** | Sequential: 1.x → 2.x → 3.x → 4.x | Jumping from 1.x to 4.x |
| **Migrate version** | Matching the jQuery version | Wrong Migrate version |
| **Warning triage** | Fix “removed” warnings first | Ignoring warnings |
| **Testing** | Test after each major version | Testing only at the end |
| **Static search** | `grep` for deprecated APIs | Relying on Migrate alone |
| **Production** | Remove Migrate when console is clean | Shipping Migrate long‑term |
| **Reference** | Official upgrade guides | Relying on memory |

---

### Constraints and Limitations

- **Sequential migration takes longer** than a direct jump, but it is the supported path.
- **Some plugins may never be updated**; they must be replaced or isolated.
- **Migrate cannot detect code that never runs**; static searches are required.

---

### Annotated Code Example (Sequential Migration Checklist)

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

- Removed | jQuery API Documentation — https://api.jquery.com/category/removed/
- Deprecated 3.0 | jQuery API Documentation — https://api.jquery.com/category/deprecated/deprecated-3.0/
- jQuery Core 3.0 Upgrade Guide — https://jquery.com/upgrade-guide/3.0/
- jQuery Core 4.0 Upgrade Guide — https://jquery.com/upgrade-guide/4.0/
- jQuery Migrate Plugin - Warning Messages | GitHub — https://github.com/jquery/jquery-migrate/blob/713d1c653775481faba718c8700451e0e34cde03/warnings.md
- jquery/jquery-migrate | GitHub — https://github.com/jquery/jquery-migrate/
- Using jQuery Migrate for Safer Upgrades | OpenReplay — http://blog.openreplay.com.s3-website-eu-west-1.amazonaws.com/jquery-migrate-safer-upgrades/
- jQuery 4.0 is out: a catch-up guide for people who stopped at 1.x | lilting.ch — https://lilting.ch/en/articles/jquery-4-catchup
- jQuery 4.0.0 Support: What’s New + Kendo UI Has You Covered | Telerik — https://www.telerik.com/blogs/jquery-4-0-0-support-whats-new-kendo-ui-has-you-covered
- .size() | jQuery API Documentation — https://api.jquery.com/size/
- jQuery 4.0 Released: Less Legacy, More Modern Web Standards | heise — https://www.heise.de
- jQuery 1.9 Upgrade Guide | jQuery — https://jquery.com/upgrade-guide/1.9/
- jQuery 3.5 Upgrade Guide | jQuery — https://jquery.com/upgrade-guide/3.5/
- Developer Guide | SAP — https://help.sap.com
- ResourceLoader/Migration guide (users) | MediaWiki — https://www.mediawiki.org/wiki/ResourceLoader/Migration_guide_(users)
- Upgrading jQuery: Working Towards a Healthy Web | Official jQuery Blog — https://blog.jquery.com/2024/04/17/upgrading-jquery-working-towards-a-healthy-web/