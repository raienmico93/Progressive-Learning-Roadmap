# HTML Comments

HTML comments are annotations in the source code that are **ignored by the browser** and **not rendered** on the page. They exist purely for developers — to document, explain, or temporarily disable code.

---

## 1. Comment Syntax

### Basic Syntax

```html
<!-- This is a comment -->
```

A comment begins with `<!--` and ends with `-->`. Everything between these markers is ignored by the browser.

### Examples

```html
<!-- This is a single-line comment -->

<!--
  This is a
  multi-line comment
-->

<p>Visible text</p>
<!-- <p>This paragraph is commented out</p> -->
<p>More visible text</p>
```

### Inline Comments

Comments can appear anywhere in the document — between elements, inside elements, or in the `<head>`:

```html
<head>
  <meta charset="UTF-8">
  <!-- Viewport meta is required for responsive design -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Page</title>
</head>
<body>
  <header>
    <!-- Logo will go here -->
    <h1>Site Title</h1>
  </header>
</body>
```

### Rules and Restrictions

| Rule | Explanation |
|---|---|
| **No nesting** | Comments cannot contain `-->` inside them |
| **No `--` inside** | Double hyphens inside comments are invalid in strict HTML |
| **Cannot contain `>` before `-->`** | In HTML5, comments end at the first `-->` |
| **Not rendered** | Comments never appear on the page |
| **Visible in source** | Anyone can view them via "View Source" |

### Invalid Comment Examples

```html
<!-- WRONG: comment inside a comment -->
<!-- This is <!-- nested --> and breaks -->

<!-- WRONG: double hyphen inside -->
<!-- This -- is invalid -->

<!-- WRONG: doesn't close -->
<!-- This comment never ends
```

### Conditional Comments (Legacy)

Older versions of Internet Explorer supported **conditional comments** to target specific IE versions:

```html
<!--[if IE 9]>
  <link rel="stylesheet" href="ie9.css">
<![endif]-->
```

These are **obsolete** and should not be used in modern development. Feature detection and polyfills have replaced them.

### The `<!--[if ...]>` Trap

The sequence `<!--[if` is sometimes parsed specially by old browsers. In modern HTML, it's just a comment. But avoid using `<!--[` at the start of a comment to prevent any legacy parsing oddities.

---

## 2. When to Use Comments

Comments serve several legitimate purposes:

### 1. Explaining Non-Obvious Code

```html
<!-- The aria-live region is updated by JavaScript when the cart changes -->
<div id="cart-status" aria-live="polite"></div>
```

### 2. Marking Sections

```html
<!-- ==================== HEADER ==================== -->
<header>...</header>

<!-- ==================== MAIN CONTENT ==================== -->
<main>...</main>

<!-- ==================== FOOTER ==================== -->
<footer>...</footer>
```

### 3. Documenting Decisions

```html
<!-- Using <section> instead of <div> for semantic meaning -->
<section class="features">...</section>
```

### 4. Noting TODOs and FIXMEs

```html
<!-- TODO: Replace placeholder image with final asset -->
<img src="placeholder.jpg" alt="Product photo">

<!-- FIXME: This form doesn't validate on Safari -->
<form action="/submit">...</form>
```

### 5. Temporarily Disabling Code

```html
<!-- Temporarily hidden during redesign
<nav class="old-nav">
  <a href="/">Home</a>
</nav>
-->
```

### 6. Providing Context for Collaborators

```html
<!-- This ID is referenced by analytics.js — do not rename -->
<div id="conversion-tracker"></div>
```

### 7. Legal and Licensing Notices

```html
<!--
  Copyright 2026 MyCompany, Inc.
  Licensed under the MIT License.
  https://opensource.org/licenses/MIT
-->
```

---

## 3. Documentation

Comments are a form of **inline documentation** — they explain the *why* behind the code, not just the *what*.

### Documenting Structure

```html
<!-- Primary navigation — appears on all pages -->
<nav aria-label="Main">
  ...
</nav>

<!-- Breadcrumb trail — only on inner pages -->
<nav aria-label="Breadcrumb">
  ...
</nav>
```

### Documenting Complex Components

```html
<!--
  Accordion component
  - Uses <details>/<summary> for native accessibility
  - Styled with CSS in components/accordion.css
  - Enhanced with JavaScript for smooth animation
-->
<details class="accordion">
  <summary>Section 1</summary>
  <p>Content...</p>
</details>
```

### Documenting Data Attributes

```html
<!--
  data-analytics: tracking category
  data-placement: where the ad appears
  data-refresh: seconds between content refreshes
-->
<div
  class="ad-slot"
  data-analytics="homepage-banner"
  data-placement="top"
  data-refresh="30"
>
</div>
```

### Documenting Dependencies

```html
<!-- Requires: normalize.css, grid.css, components/button.css -->
<link rel="stylesheet" href="css/main.css">

<!-- Requires: vendor/lodash.min.js -->
<script src="js/app.js" defer></script>
```

### What Good Documentation Looks Like

**Good — explains why:**

```html
<!-- Using role="status" so screen readers announce updates politely -->
<div role="status">Item added to cart</div>
```

**Bad — restates the obvious:**

```html
<!-- This is a div with role status -->
<div role="status">Item added to cart</div>
```

### What to Document

| Document | Don't Document |
|---|---|
| Non-obvious decisions | Obvious structure |
| Workarounds and their reasons | What the tag is |
| Dependencies and requirements | Basic HTML syntax |
| TODOs with context | Vague "fix this later" |
| Accessibility considerations | Redundant descriptions |
| Legal/licensing info | Personal notes |

---

## 4. Temporary Development Notes

Comments are useful for **temporary** notes during development, but they should be cleaned up before deployment.

### Common Temporary Uses

#### Placeholder Markup

```html
<!-- PLACEHOLDER: Replace with actual hero image -->
<div class="hero-placeholder"></div>
```

#### Debugging

```html
<!-- DEBUG: Checking if this section renders correctly -->
<section class="debug-section">
  <p>Test content</p>
</section>
```

#### Work-in-Progress Notes

```html
<!--
  WIP: Footer redesign in progress
  - Need to add social icons
  - Need to add newsletter signup
  - Waiting on legal for privacy policy link
-->
<footer>
  <p>&copy; 2026 MyCompany</p>
</footer>
```

#### Commenting Out Code Temporarily

```html
<!-- Temporarily disabled during A/B test
<button class="cta-old">Sign Up</button>
-->
<button class="cta-new">Get Started Free</button>
```

#### Marking Unfinished Work

```html
<!-- TODO: Add form validation (ticket #1234) -->
<!-- FIXME: Button alignment breaks on mobile (ticket #1235) -->
<!-- HACK: Temporary workaround for Safari flexbox bug -->
```

### Best Practices for Temporary Comments

1. **Include a ticket number** so the note can be tracked:

   ```html
   <!-- TODO(#1234): Replace with real API endpoint -->
   ```

2. **Date your notes** if they might linger:

   ```html
   <!-- TODO (2026-01-15): Refactor this section -->
   ```

3. **Use consistent prefixes** to make notes searchable:
   - `TODO:` — something to do
   - `FIXME:` — something broken
   - `HACK:` — a workaround
   - `NOTE:` — important context
   - `REVIEW:` — needs review

4. **Clean up before deployment** — temporary comments shouldn't ship to production.

### The Difference Between TODO and FIXME

| Prefix | Meaning | Action |
|---|---|---|
| `TODO:` | Feature or improvement to add | Add to backlog |
| `FIXME:` | Known bug or broken behavior | Fix before release |
| `HACK:` | Fragile workaround | Replace with proper solution |
| `NOTE:` | Context for future readers | Keep indefinitely |
| `REVIEW:` | Needs a second opinion | Request review |

---

## 5. Avoiding Unnecessary Comments

Comments add value only when they explain something **non-obvious**. Over-commenting clutters the code and becomes noise.

### The Golden Rule

> **Comment the *why*, not the *what*.**

### Examples of Unnecessary Comments

```html
<!-- BAD: restates the obvious -->

<!-- This is the header -->
<header>
  <!-- This is the navigation -->
  <nav>
    <!-- This is a link to the home page -->
    <a href="/">Home</a>
    <!-- This is a link to the about page -->
    <a href="/about">About</a>
  </nav>
</header>

<!-- This closes the header -->
</header>
```

### Examples of Useful Comments

```html
<!-- GOOD: explains why -->

<!-- aria-label added because the icon has no visible text -->
<button aria-label="Search">
  <svg>...</svg>
</button>

<!-- data-* attributes consumed by analytics.js for event tracking -->
<a href="/pricing" data-track="cta-pricing">View Pricing</a>

<!-- Wrapping in <section> instead of <div> for landmark navigation -->
<section class="testimonials">...</section>
```

### Anti-Patterns to Avoid

#### 1. Commented-Out Code Left Behind

```html
<!-- BAD: dead code cluttering the file -->
<!-- <div class="old-banner">
  <p>Old promotion text</p>
</div> -->
```

**Why it's bad:** Version control (Git) already tracks history. If you need it back, retrieve it from Git — don't leave it in the file.

#### 2. Changelog Comments

```html
<!-- BAD: version history in comments -->
<!--
  2026-01-01: Added hero section
  2026-01-05: Changed hero image
  2026-01-10: Removed hero CTA
-->
```

**Why it's bad:** Use Git commit messages instead.

#### 3. Author Tags

```html
<!-- BAD: personal tags -->
<!-- Created by Jane -->
<!-- Last modified by Bob -->
```

**Why it's bad:** Git blame provides this info more accurately.

#### 4. Redundant Section Markers

```html
<!-- BAD: marker adds nothing -->
<div>...</div>
<!-- End div -->
```

**Why it's bad:** The closing tag already tells you what it closes.

#### 5. Outdated Comments

```html
<!-- BAD: comment contradicts code -->
<!-- Displays user's email -->
<p class="username">JohnDoe</p>
```

**Why it's bad:** Worse than no comment — actively misleading.

### When to Delete a Comment

Delete a comment when:

- It restates what the code already says
- It's outdated or contradicts the code
- It's commented-out dead code
- It's a personal note or changelog
- It's a "note to self" that no longer applies

### When to Keep a Comment

Keep a comment when:

- It explains a **non-obvious decision**
- It documents a **workaround** and why it exists
- It **warns** about a gotcha
- It references an **external resource** (ticket, spec, doc)
- It's a **legal notice** (license, copyright)

### The "Future You" Test

Before writing a comment, ask:

> *"Will this help someone (including future me) understand the code better?"*

If the answer is no — **don't write it**.

---

## 6. Comments and Security

### Comments Are Visible to Everyone

HTML comments are sent to the browser and visible via **View Source** or **DevTools**. **Never put sensitive information in comments.**

```html
<!-- DANGER: never do this -->
<!--
  Staging API key: sk_test_abc123
  Admin password: hunter2
  TODO: Remove this before launch
-->
```

### What NOT to Put in Comments

| Don't Include | Why |
|---|---|
| Passwords or API keys | Visible to anyone |
| Internal URLs | Reveals infrastructure |
| Employee names | Privacy concerns |
| Unreleased features | Competitive intelligence leak |
| Security vulnerabilities | Highlights attack vectors |
| Database schemas | Aids attackers |
| Comments about users | Privacy/legal risk |

### Comments vs. Build-Time Comments

Some build tools (like server-side includes, templating engines, or bundlers) support **build-time comments** that are stripped from the output:

```html
<!--#include virtual="/header.html" -->
```

These are processed on the server and never sent to the browser. But standard HTML comments **are** sent — treat them as public.

### Minification Removes Comments (Sometimes)

Build tools like HTML minifiers often strip comments from production output. But:

- **Don't rely on this** — you may deploy unminified
- **Some comments are preserved** — like conditional comments or license headers
- **Assumption of removal is risky** — always assume comments are visible

---

## 7. Comments in Different Contexts

### Comments in `<head>`

```html
<head>
  <meta charset="UTF-8">
  <!-- Preconnect to improve font loading performance -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="stylesheet" href="css/main.css">
</head>
```

### Comments in `<body>`

```html
<body>
  <!-- Skip link for keyboard users -->
  <a href="#main" class="skip-link">Skip to content</a>

  <header>...</header>

  <main id="main">
    <!-- Content sections -->
  </main>
</body>
```

### Comments Around Scripts

```html
<!-- Load analytics only in production -->
<script>
  if (location.hostname !== 'localhost') {
    // analytics code
  }
</script>
```

### Comments in Templates

In templating languages (Handlebars, Jinja, EJS), HTML comments coexist with template comments:

```html
<!-- This is an HTML comment (appears in output) -->
{{! This is a Handlebars comment (stripped) }}
{# This is a Jinja comment (stripped) #}
```

**Important:** HTML comments in templates **will appear in the final output**. Use template-specific comments to keep notes out of production.

### Comments in Framework Components

In React, Vue, and similar frameworks, HTML-like comments behave differently:

```jsx
// React: use JSX comments
{/* This is a JSX comment */}
```

```vue
<!-- Vue: HTML comments work in templates -->
<!-- But they're stripped in production builds by default -->
```

---

## 8. Complete Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- Viewport meta enables responsive layout on mobile -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Comment Demo</title>

  <!-- Preconnect improves font load performance -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="stylesheet" href="css/main.css">

  <!--
    Copyright 2026 MyCompany, Inc.
    Licensed under the MIT License.
  -->
</head>
<body>
  <!-- Skip link for keyboard and screen reader users -->
  <a href="#main" class="skip-link">Skip to content</a>

  <!-- ==================== HEADER ==================== -->
  <header class="site-header">
    <h1 class="logo" translate="no">MyBrand</h1>

    <!-- aria-label distinguishes this nav from others on the page -->
    <nav aria-label="Main">
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  </header>

  <!-- ==================== MAIN ==================== -->
  <main id="main">
    <!--
      Hero section
      - Background image is set in CSS (hero.css)
      - data-parallax enables scroll animation via app.js
    -->
    <section class="hero" data-parallax="true">
      <h2>Welcome</h2>
      <p>Discover something new.</p>

      <!-- TODO(#482): Replace placeholder CTA text with final copy -->
      <a href="/signup" class="btn btn-primary">Get Started</a>
    </section>

    <!--
      Testimonials
      - Using <blockquote> for semantic meaning
      - cite attribute links to original source
    -->
    <section class="testimonials">
      <blockquote cite="https://example.com/review">
        <p>This product changed my workflow.</p>
        <footer>— <cite>Jane D.</cite></footer>
      </blockquote>
    </section>
  </main>

  <!-- ==================== FOOTER ==================== -->
  <footer class="site-footer">
    <p>&copy; 2026 MyCompany</p>
  </footer>

  <!-- Loaded at end of body to avoid blocking rendering -->
  <script src="js/app.js" defer></script>
</body>
</html>
```

---

## 9. Quick Reference Table

| Aspect | Guideline |
|---|---|
| **Syntax** | `<!-- comment -->` |
| **Nesting** | Not allowed |
| **Rendering** | Never displayed |
| **Visibility** | Visible in page source |
| **Purpose** | Explain *why*, not *what* |
| **Documentation** | Non-obvious decisions, dependencies |
| **Temporary notes** | Use `TODO:`, `FIXME:`, `NOTE:` prefixes |
| **Cleanup** | Remove temporary notes before deployment |
| **Security** | Never put secrets in comments |
| **Avoid** | Restating obvious code, dead code, changelogs |
| **Keep** | Workarounds, warnings, legal notices, TODOs with tickets |

---

## Key Takeaways

1. **Syntax:** `<!-- comment -->` — ignored by browsers, visible in source.
2. **No nesting** and **no `--` inside** comments.
3. Comment the **why**, not the **what**.
4. Use comments for **documentation**, **context**, and **temporary notes**.
5. Prefix temporary notes with **`TODO:`, `FIXME:`, `HACK:`, `NOTE:`** for searchability.
6. **Avoid** restating obvious code, dead code, changelogs, and author tags.
7. **Never put secrets** in comments — they're public.
8. **Clean up** temporary comments before deployment.
9. Use **Git**, not comments, for version history and authorship.
10. A good comment **helps the future reader** — including future you.

---

Would you like me to continue with **"6. HTML Text Content"** (headings, paragraphs, `<br>`, `<hr>`, `<pre>`, `<blockquote>`), or expand on any part of this section (e.g., comments in templating engines, or HTML minification behavior)?