# Link Accessibility — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Link accessibility is the practice of creating hyperlinks that are perceivable, operable, and understandable by everyone, including people who use screen readers, keyboard-only navigation, screen magnifiers, or other assistive technologies. It encompasses writing meaningful link text, providing mechanisms to bypass repeated content (skip links), ensuring visible keyboard focus indicators, and following established best practices for link behaviour and structure.

**Technical Definition**
In the context of the Web Content Accessibility Guidelines (WCAG), link accessibility is governed by several success criteria: **2.4.4 Link Purpose (In Context)** (Level A), which requires that the purpose of each link can be determined from the link text alone or together with its programmatically determined context; **2.4.9 Link Purpose (Link Only)** (Level AAA), which requires that the purpose be determinable from the link text alone; **2.4.1 Bypass Blocks** (Level A), which requires a mechanism to skip repeated blocks of content; and **2.4.7 Focus Visible** (Level AA), which requires that any keyboard-operable interface have a visible keyboard focus indicator. These criteria interact with the accessible name computation, the accessibility tree, and browser focus management.

**Beginner-Friendly Explanation**
Link accessibility means making links work for everyone. That means writing link text that clearly says where the link goes (not “click here”), giving keyboard users a way to skip past repeated navigation menus (a “skip link”), and making sure you can see which link is selected when you tab through a page (a visible focus outline). These practices help people with visual, motor, and cognitive disabilities use the web successfully.

---

### Key Characteristics

- **Link text is the accessible name**: Screen readers use the link’s text content (or the `alt` text of an image inside it) as the link’s name; vague text like “click here” is unusable when links are listed out of context.
- **Context matters for Level A**: WCAG 2.4.4 allows the surrounding sentence, paragraph, list item, or table cell to provide context, but best practice is to make the link text meaningful on its own.
- **Skip links satisfy Bypass Blocks**: A “Skip to main content” link at the top of each page allows keyboard and screen-reader users to bypass repeated navigation and header content.
- **Focus visibility is mandatory**: WCAG 2.4.7 requires that keyboard focus indicators be visible; removing `outline` without providing an alternative fails this criterion.
- **Keyboard operability**: Links must be reachable and activatable via keyboard (Tab to focus, Enter to activate) — native `<a href>` provides this automatically.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<a>`, `href`, and attributes.
2. **The accessibility tree**: understanding how browsers expose links to assistive technologies.
3. **Keyboard navigation basics**: understanding Tab order and focus management.
4. **WCAG fundamentals**: awareness of Level A, AA, and AAA success criteria.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **WCAG conformance** | 2.4.4, 2.4.9, 2.4.1, and 2.4.7 are the core link-related criteria. |
| **Screen readers** | Link lists and link navigation modes depend on meaningful link text. |
| **Keyboard accessibility** | Focus management and skip links are essential for keyboard-only users. |
| **CSS** | `:focus`, `:focus-visible`, and `outline` control focus indicator appearance. |
| **SEO** | Descriptive link text also benefits search-engine understanding. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Link text** — the accessible name
2. **Descriptive links** — meaningful, context-independent link text
3. **Skip links** — bypassing repeated content
4. **Focus states** — visible keyboard focus indicators
5. **Link accessibility best practices** — comprehensive guidelines

---

## 1. Link Text — The Accessible Name

### Definitions

**Core Definition**
Link text is the visible text content of a hyperlink, which also serves as its accessible name for assistive technologies. It is the primary information screen-reader users rely on to decide whether to follow a link.

**Technical Definition**
The accessible name of a link is computed from its content (text nodes and the `alt` text of embedded images), unless overridden by `aria-label` or `aria-labelledby`. Screen readers can present a list of all links on a page; when links are presented out of context, the link text alone must convey the purpose. WCAG 2.4.4 (Level A) allows the link text plus programmatically determined context (same sentence, paragraph, list item, or table cell); WCAG 2.4.9 (Level AAA) requires the link text alone to be sufficient.

**Beginner-Friendly Explanation**
Link text is the words you click on. Screen readers often read links out of context — in a list, for example — so the text must make sense on its own. “View pricing plans” works; “click here” does not.

---

### Purposes

- **To provide the accessible name** for screen-reader users.
- **To describe the link destination** so users can decide whether to follow it.
- **To satisfy WCAG 2.4.4 and 2.4.9** for link purpose.
- **To improve SEO** by giving search engines context about the linked resource.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="/pricing">View pricing plans</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| Content | Visible text that becomes the accessible name. |
| `alt` on images | If an image is the only content, its `alt` text becomes the link text. |
| `aria-label` | Optional override; use only when visible text is insufficient. |

#### Syntax Rules

1. **Link text must be meaningful out of context** — avoid “click here,” “read more,” and “learn more”.
2. **If an image is the only content**, its `alt` attribute describes the link purpose.
3. **If text and an image are both present**, the image may have `alt=""` if the text already describes the purpose.
4. **Do not use `aria-label` to override visible text** unless the visible text is genuinely insufficient.

#### Constraints and Limitations

- **Screen readers often provide a list of links out of context**; vague text is unusable in this mode.
- **Long URLs as link text** are poor for accessibility and usability.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Descriptive Link Text

**Setup Guide**: Save as `link-text-good.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Descriptive Link Text</title>
</head>
<body>
  <p>
    For more information, read our
    <a href="/shipping-policy">shipping policy</a>.
  </p>
</body>
</html>
```

**Expected output**: The link text “shipping policy” clearly describes the destination, both in context and out of context.

**Why this output occurs**: The link text is descriptive and meaningful on its own. A screen reader user who encounters the link in a list of links knows exactly where it goes.

---

#### Example 2: Vague Link Text (Anti-Pattern)

**Setup Guide**: Save as `link-text-bad.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Vague Link Text</title>
</head>
<body>
  <p>
    For more information, <a href="/shipping-policy">click here</a>.
  </p>
</body>
</html>
```

**Expected output**: The link is announced as “click here, link” when read out of context, giving the user no information about the destination.

**Why this output occurs**: “Click here” is not descriptive; it fails WCAG 2.4.4 and 2.4.9 because the link purpose cannot be determined from the text alone.

---

#### Example 3: Image-Only Link with Alt Text

**Setup Guide**: Save as `link-image.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image Link</title>
</head>
<body>
  <a href="/">
    <img src="logo.svg" alt="Example Company homepage">
  </a>
</body>
</html>
```

**Expected output**: The link is announced as “Example Company homepage, link” because the image’s `alt` text becomes the accessible name.

**Why this output occurs**: When an image is the only content of a link, its `alt` attribute describes the link purpose.

---

### Real-World Cases

1. **Navigation menus**: Descriptive link text improves usability and SEO.
2. **Content links**: Inline links with descriptive text.
3. **Image links**: Logos and icons with meaningful `alt` text.

---

### References

- W3C — Technique H30: Providing link text that describes the purpose of a link — https://www.w3.org/WAI/WCAG21/Techniques/html/H30
- W3C — Understanding SC 2.4.4: Link Purpose (In Context) — https://w3c.github.io/wcag/understanding/link-purpose-in-context
- W3C — Understanding SC 2.4.9: Link Purpose (Link Only) — https://w3c.github.io/wcag/understanding/link-purpose-link-only

---

## 2. Descriptive Links — Meaningful, Context-Independent Link Text

### Definitions

**Core Definition**
Descriptive links are hyperlinks whose text clearly communicates the link’s destination or purpose, ideally without requiring surrounding context. They are the practical implementation of WCAG 2.4.4 and 2.4.9.

**Technical Definition**
Descriptive link text satisfies Technique H30, which states that the description should let a user distinguish the link from other links on the page and determine whether to follow it. Best practice goes beyond WCAG 2.4.4 (which allows contextual support) to meet the spirit of WCAG 2.4.9 (Level AAA), where the link text alone must be sufficient. Descriptive links also benefit users of screen readers, screen magnifiers, and cognitive disabilities.

**Beginner-Friendly Explanation**
A descriptive link tells you where it goes just by reading its text. Instead of “click here,” it says “download the annual report” or “view shipping policy.” This helps everyone, not just screen-reader users.

---

### Purposes

- **To make links understandable out of context**.
- **To satisfy WCAG 2.4.9 (Level AAA)** and exceed the minimum of 2.4.4.
- **To improve the experience for screen-reader users** who navigate by link lists.
- **To help all users scan and navigate** pages efficiently.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Describe the destination** | Use the page title or a clear descriptive phrase. |
| **Avoid vague phrases** | Do not use “click here,” “read more,” “info,” “more,” or “here”. |
| **Make links unique** | Different destinations should have different link text. |
| **Include file types when helpful** | Mention “(PDF)” or “(Excel)” when the link opens a file. |
| **Avoid long URLs** | Use human-readable text, not the raw URL. |

#### Syntax Rules

1. **Link text should describe the purpose** of the link.
2. **Use unique text for unique destinations**; repeated text like “read more” is confusing in a link list.
3. **If a generic phrase is unavoidable**, use `aria-label` to provide a unique accessible name.

#### Constraints and Limitations

- **Context can help at Level A**, but descriptive links aim for Level AAA independence.
- **Visible text and `aria-label` should agree**; mismatches confuse sighted and screen-reader users.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Descriptive Links in a List

**Setup Guide**: Save as `descriptive-list.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Descriptive Links</title>
</head>
<body>
  <ul>
    <li><a href="/reports/2025-annual.pdf">2025 Annual Report (PDF)</a></li>
    <li><a href="/reports/2024-annual.pdf">2024 Annual Report (PDF)</a></li>
    <li><a href="/budget/2025.xlsx">2025 Budget (Excel)</a></li>
  </ul>
</body>
</html>
```

**Expected output**: Each link clearly describes its destination and file type, making the list usable even when read out of context.

**Why this output occurs**: The link text includes the document name and file type, satisfying Technique H30 and providing unique, descriptive labels for each destination.

---

#### Example 2: Using `aria-label` for a Generic Phrase

**Setup Guide**: Save as `descriptive-aria.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Descriptive with ARIA</title>
</head>
<body>
  <article>
    <h2>How to Build Accessible Forms</h2>
    <p>Learn the fundamentals of accessible form design.</p>
    <a href="/articles/accessible-forms" aria-label="Read more about How to Build Accessible Forms">
      Read more
    </a>
  </article>
</body>
</html>
```

**Expected output**: Sighted users see “Read more,” but screen readers announce “Read more about How to Build Accessible Forms, link,” providing context that the visible text alone lacks.

**Why this output occurs**: The `aria-label` overrides the visible text with a unique, descriptive accessible name, addressing the problem of repeated “Read more” links in a list.

---

### Real-World Cases

1. **E-commerce**: Product links with descriptive names (e.g. “Wireless Headphones — $99”).
2. **Documentation**: Links that include the page title or section name.
3. **News sites**: Article links with the headline as the link text.

---

### References

- W3C — Technique H30: Providing link text that describes the purpose of a link — https://www.w3.org/WAI/WCAG21/Techniques/html/H30
- W3C — Understanding SC 2.4.9: Link Purpose (Link Only) — https://w3c.github.io/wcag/understanding/link-purpose-link-only
- Digital Agency Design System — Link text accessibility — https://design.digital.go.jp/dads/foundations/link-text/accessibility/

---

## 3. Skip Links — Bypassing Repeated Content

### Definitions

**Core Definition**
A skip link (also called a “skip navigation” or “skip to main content” link) is a hyperlink placed at or near the beginning of a page that allows keyboard and screen-reader users to bypass repeated blocks of content — such as navigation menus, headers, and advertising frames — and jump directly to the main content.

**Technical Definition**
Skip links satisfy **WCAG 2.4.1 Bypass Blocks (Level A)** , which requires a mechanism to bypass blocks of content that are repeated on multiple web pages. Technique G123 describes adding a link at the beginning of a repeated block that moves focus past the block. The skip link targets an element with a matching `id` (typically `<main id="main-content">`) and must be operable by keyboard. It should be visible on focus so that sighted keyboard users can discover it.

**Beginner-Friendly Explanation**
A skip link is a hidden link at the top of the page that appears when you press Tab. It says “Skip to main content” and, when activated, jumps you past the navigation menu straight to the important content. This saves keyboard and screen-reader users from tabbing through dozens of menu links on every page.

---

### Purposes

- **To satisfy WCAG 2.4.1 (Bypass Blocks)** .
- **To save keyboard users from tabbing through repeated navigation** on every page.
- **To let screen-reader users skip header content** and reach the main content quickly.
- **To improve efficiency** for users of screen magnifiers and cognitive disabilities.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="#main-content" class="skip-link">Skip to main content</a>
...
<main id="main-content">
  …
</main>
```

**Component breakdown**

| Component | Description |
|---|---|
| `href="#main-content"` | Target the main content region. |
| `.skip-link` | CSS class for visually hiding until focused. |
| `<main id="main-content">` | The target element. |

#### Syntax Rules

1. **Place the skip link at the very beginning of the `<body>`** .
2. **Target the `<main>` element** (or the first element of the main content) with a matching `id`.
3. **The skip link must be keyboard-operable** and become visible on focus.
4. **Use consistent labels** across all pages (e.g. always “Skip to main content”).

#### Constraints and Limitations

- **Skip links that never become visible** are useless to sighted keyboard users.
- **Targeting an element that is not focusable** may require `tabindex="-1"` on the target to ensure focus moves correctly.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Standard Skip Link

**Setup Guide**: Save as `skip-link.html`. Press Tab on page load to reveal the skip link.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Skip Link</title>
  <style>
    .skip-link {
      position: absolute;
      top: -40px;
      left: 0;
      background: #000;
      color: #fff;
      padding: 8px;
      z-index: 100;
    }
    .skip-link:focus {
      top: 0;
    }
  </style>
</head>
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header>
    <h1>Site Title</h1>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <main id="main-content">
    <h2>Main Content</h2>
    <p>This is the primary content of the page.</p>
  </main>
</body>
</html>
```

**Expected output**: When the page loads, the skip link is off-screen. Pressing Tab reveals it at the top of the page. Activating it moves focus to the `<main>` element, bypassing the header and navigation.

**Why this output occurs**: The skip link satisfies WCAG 2.4.1 by providing a mechanism to bypass the repeated navigation block. The CSS hides the link until it receives focus, making it discoverable for keyboard users.

---

#### Example 2: Skip Link Targeting a Focusable Main Region

**Setup Guide**: Save as `skip-link-focus.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Skip Link with Focus Target</title>
</head>
<body>
  <a href="#main" class="skip-link">Skip to main content</a>

  <nav>
    <a href="/">Home</a>
    <a href="/products">Products</a>
    <a href="/contact">Contact</a>
  </nav>

  <main id="main" tabindex="-1">
    <h1>Welcome</h1>
    <p>Main content starts here.</p>
  </main>
</body>
</html>
```

**Expected output**: Activating the skip link moves focus to the `<main>` element, and the browser scrolls it into view.

**Why this output occurs**: The `tabindex="-1"` on `<main>` makes the element programmatically focusable, ensuring that focus actually lands on the main content region when the skip link is activated.

---

### Real-World Cases

1. **Government and public-sector sites**: Skip links are a common requirement for WCAG compliance.
2. **Content-heavy sites**: News sites and documentation with extensive navigation menus.
3. **Single-page applications**: Skip links can target the main content region of the current route.

---

### References

- W3C — Understanding SC 2.4.1: Bypass Blocks — https://w3c.github.io/wcag/understanding/bypass-blocks
- W3C — Technique G123: Adding a link at the beginning of a block of repeated content — https://www.w3.org/WAI/WCAG21/Techniques/general/G123
- Province of British Columbia — Navigable content: Skip links — https://digital.gov.bc.ca/accessible-web-products/navigable-content/

---

## 4. Focus States — Visible Keyboard Focus Indicators

### Definitions

**Core Definition**
A focus state (or focus indicator) is the visual change applied to an element when it receives keyboard focus. It tells sighted keyboard users which element will respond to their next keypress.

**Technical Definition**
WCAG **2.4.7 Focus Visible (Level AA)** requires that any keyboard-operable user interface have a mode of operation where the keyboard focus indicator is visible. The focus indicator is typically implemented as an outline or border, and it must not be time-limited. Focus indicators are also subject to WCAG 1.4.11 Non-text Contrast (Level AA), requiring a 3:1 contrast ratio against adjacent colors. The `:focus-visible` pseudo-class allows authors to show focus indicators only when needed (e.g. keyboard navigation), while suppressing them for mouse clicks.

**Beginner-Friendly Explanation**
When you press Tab to move through links on a page, a visible outline appears around the currently focused link. That outline is the focus state. It tells you where you are. Never remove it with CSS unless you replace it with something equally visible.

---

### Purposes

- **To satisfy WCAG 2.4.7 (Focus Visible)** .
- **To let keyboard users know which link will activate** when they press Enter.
- **To help users with attention, memory, or executive-function limitations** track their position.
- **To support screen-magnifier users** who rely on visual focus cues.

---

### Syntax Rules and Structure

#### Complete General Syntax (CSS)

```css
a:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

**Component breakdown**

| Component | Description |
|---|---|
| `:focus-visible` | Applies styles when focus is visible (typically keyboard). |
| `outline` | The focus indicator. |
| `outline-offset` | Separates the outline from the element. |

#### Syntax Rules

1. **Never remove the default focus indicator** without providing a replacement.
2. **Use `:focus-visible`** to show indicators for keyboard users while suppressing them for mouse clicks.
3. **Ensure the indicator has sufficient contrast** (3:1 against adjacent colors).
4. **Do not time-limit the focus indicator**; it must remain visible while focus is held.

#### Constraints and Limitations

- **`:focus-visible` support is broad but not universal**; provide a fallback `:focus` style for older browsers.
- **Custom focus styles must be at least as visible** as the browser default.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Focus Indicator with `:focus-visible`

**Setup Guide**: Save as `focus-visible.html`. Tab through the links to see the focus indicator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Focus Visible</title>
  <style>
    a:focus-visible {
      outline: 3px solid #e63946;
      outline-offset: 2px;
      border-radius: 2px;
    }
    /* Fallback for older browsers */
    a:focus {
      outline: 3px solid #e63946;
      outline-offset: 2px;
    }
  </style>
</head>
<body>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
    <a href="/contact">Contact</a>
  </nav>
</body>
</html>
```

**Expected output**: When tabbing through the links, each focused link receives a prominent red outline. Clicking with the mouse does not show the outline (in browsers that support `:focus-visible`).

**Why this output occurs**: `:focus-visible` applies the focus style only when the browser determines that focus should be visible (typically for keyboard navigation), satisfying WCAG 2.4.7 while avoiding distracting outlines for mouse users.

---

#### Example 2: Removing Focus Indicator (Anti-Pattern)

**Setup Guide**: Save as `focus-bad.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>No Focus Indicator</title>
  <style>
    /* ANTI-PATTERN: removes focus indicator entirely */
    a:focus {
      outline: none;
    }
  </style>
</head>
<body>
  <a href="/">Home</a>
  <a href="/about">About</a>
</body>
</html>
```

**Expected output**: Keyboard users see no indication of which link is focused, making the page effectively unusable without a mouse.

**Why this output occurs**: Removing `outline` without providing an alternative fails WCAG 2.4.7 and harms keyboard accessibility.

---

### Real-World Cases

1. **All interactive pages**: Focus indicators are required for keyboard accessibility.
2. **Design systems**: Focus styles are codified into component libraries.
3. **Government sites**: WCAG 2.4.7 compliance is audited.

---

### References

- W3C — Understanding SC 2.4.7: Focus Visible — https://w3c.github.io/wcag/understanding/focus-visible
- MDN Web Docs — `:focus-visible` — https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible
- Oregon State University — Focus Visible — https://accessibility.oregonstate.edu/focus-visible

---

## 5. Link Accessibility Best Practices

### Definitions

**Core Definition**
Link accessibility best practices are guidelines for creating hyperlinks that are usable by everyone — covering descriptive text, keyboard operability, visible focus, skip mechanisms, and appropriate warnings for non-standard behaviour.

**Technical Definition**
Best practices are informed by WCAG 2.4.4, 2.4.9, 2.4.1, and 2.4.7, as well as Techniques H30, G123, and G165. They encompass meaningful link text, unique labels for unique destinations, skip links for repeated content, visible focus indicators, warnings for new tabs or downloads, and appropriate use of `aria-label` when visible text is insufficient.

**Beginner-Friendly Explanation**
Best practices help you build links that work for everyone: describe where the link goes, give keyboard users a way to skip menus, make focus visible, and warn users when a link opens a new tab or downloads a file.

---

### Purposes

- **To ensure WCAG conformance** (2.4.4, 2.4.9, 2.4.1, 2.4.7).
- **To support screen-reader and keyboard users** effectively.
- **To improve usability for all users** through clear, predictable link behaviour.
- **To avoid common accessibility failures** such as vague text and invisible focus.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Write descriptive link text** | Describe the destination or purpose; avoid “click here”. |
| **Make links unique** | Different destinations should have different text. |
| **Provide skip links** | Include a “Skip to main content” link at the top of each page. |
| **Never remove focus indicators** | Use `:focus-visible` with a visible outline. |
| **Warn about new tabs and downloads** | Indicate when a link opens a new tab or downloads a file. |
| **Use `aria-label` for generic phrases** | Provide a unique accessible name when visible text is insufficient. |
| **Combine image and text links** | Use `alt=""` on images when adjacent text already describes the link. |
| **Avoid long URLs as link text** | Use human-readable text instead. |

#### Syntax Rules

1. **Accessibility first**: Link text must be meaningful; focus must be visible; repeated content must be skippable.
2. **Consistency**: Use the same skip-link label across pages.
3. **Predictability**: Warn users when a link behaves non-standardly (new tab, download, external app).

#### Constraints and Limitations

- **`aria-label` overrides visible text**; use it only when necessary.
- **Skip links must be keyboard-operable** and visible on focus.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Fully Accessible Link Set

**Setup Guide**: Save as `best-practice-links.html`. Tab through the page to verify focus and skip-link behaviour.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Links</title>
  <style>
    .skip-link {
      position: absolute;
      top: -40px;
      left: 0;
      background: #000;
      color: #fff;
      padding: 8px;
    }
    .skip-link:focus {
      top: 0;
    }
    a:focus-visible {
      outline: 3px solid #e63946;
      outline-offset: 2px;
    }
  </style>
</head>
<body>
  <a href="#main" class="skip-link">Skip to main content</a>

  <header>
    <nav>
      <a href="/">Home</a>
      <a href="/products">Products</a>
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <main id="main">
    <h1>Welcome</h1>
    <p>
      Read our
      <a href="/shipping-policy">shipping policy</a>
      or
      <a href="/annual-report.pdf" download>
        download the annual report (PDF)
      </a>.
    </p>
  </main>
</body>
</html>
```

**Expected output**: The skip link appears on focus; all links have visible focus outlines; the download link includes a clear label and file type.

**Why this output occurs**: The skip link satisfies 2.4.1; the descriptive link text satisfies 2.4.4 and 2.4.9; the visible focus outlines satisfy 2.4.7; and the download warning improves usability.

---

#### Example 2: Common Accessibility Failures

**Setup Guide**: Save as `link-failures.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Link Failures</title>
  <style>
    /* FAILURE: removes focus indicator */
    a:focus { outline: none; }
  </style>
</head>
<body>
  <!-- FAILURE: vague link text -->
  <p><a href="/report.pdf">Click here</a> for the report.</p>

  <!-- FAILURE: long URL as link text -->
  <p><a href="https://example.com/very/long/path">https://example.com/very/long/path</a></p>

  <!-- FAILURE: no skip link on a page with repeated navigation -->
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
    <a href="/contact">Contact</a>
  </nav>
</body>
</html>
```

**Expected output**: The page fails multiple WCAG criteria: no visible focus (2.4.7), vague link text (2.4.4/2.4.9), and no skip mechanism (2.4.1).

**Why this output occurs**: Each failure violates a specific accessibility requirement, making the page difficult or impossible to use for keyboard and screen-reader users.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG compliance is legally required; link accessibility is audited.
2. **E-commerce**: Descriptive product links and accessible focus states improve the shopping experience.
3. **Content management systems**: Templates enforce skip links and descriptive link text.

---

### References

- W3C — Technique H30: Providing link text that describes the purpose of a link — https://www.w3.org/WAI/WCAG21/Techniques/html/H30
- W3C — Technique G123: Adding a link at the beginning of a block of repeated content — https://www.w3.org/WAI/WCAG21/Techniques/general/G123
- W3C — Understanding SC 2.4.1: Bypass Blocks — https://w3c.github.io/wcag/understanding/bypass-blocks
- W3C — Understanding SC 2.4.7: Focus Visible — https://w3c.github.io/wcag/understanding/focus-visible
- W3C — Understanding SC 2.4.4: Link Purpose (In Context) — https://w3c.github.io/wcag/understanding/link-purpose-in-context
- W3C — Understanding SC 2.4.9: Link Purpose (Link Only) — https://w3c.github.io/wcag/understanding/link-purpose-link-only

---

## Summary Table of Link Accessibility Concepts

| Concept | WCAG Criterion | Key Requirement | Best Practice |
|---|---|---|---|
| **Link text** | 2.4.4 (A), 2.4.9 (AAA) | Purpose determinable from text or context | Descriptive, unique text |
| **Descriptive links** | 2.4.9 (AAA) | Purpose from text alone | Avoid vague phrases; include file types |
| **Skip links** | 2.4.1 (A) | Mechanism to bypass repeated blocks | “Skip to main content” at page start |
| **Focus states** | 2.4.7 (AA) | Visible keyboard focus indicator | `:focus-visible` with high-contrast outline |
| **Best practices** | Multiple | Combine all of the above | Descriptive text, skip links, visible focus, warnings |

---

## Final Notes

- **Link text is the foundation**: Descriptive, meaningful link text benefits screen-reader users, keyboard users, and search engines alike.
- **Skip links are a Level A requirement**: Every page with repeated navigation should provide a mechanism to bypass it.
- **Focus visibility is non-negotiable**: Removing focus outlines without replacement fails WCAG 2.4.7 and breaks keyboard navigation.
- **Test with real assistive technologies**: Automated checkers catch some issues, but screen-reader and keyboard testing is essential.
- **Use `aria-label` judiciously**: It is a powerful tool for fixing generic link text, but it should not override clear visible text.

---

This cheat sheet has covered link accessibility — link text, descriptive links, skip links, focus states, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate link accessibility against the WCAG 2.2 success criteria 2.4.1, 2.4.4, 2.4.7, and 2.4.9.