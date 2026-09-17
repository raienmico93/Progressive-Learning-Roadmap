# CSS Attribute Selectors: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
CSS attribute selectors are patterns that match elements based on the presence, value, or substring content of their HTML attributes.

**Technical Definition**
According to MDN Web Docs, the CSS attribute selector "matches elements based on the element having a given attribute explicitly set, with options for defining an attribute value or substring value match". The W3C Selectors Level 4 specification defines attribute selectors as a category of simple selectors that allow "the representation of an element's attributes" and are "considered to match an element if that element has an attribute that matches the attribute represented by the attribute selector". Attribute selectors are part of the CSS Selectors specification and have been available across browsers since July 2015.

**Beginner-Friendly Explanation**
Imagine you have a box of assorted buttons. Some have labels, some are red, some are round, and some are both. Attribute selectors are like saying "give me all buttons with a label" or "all buttons whose label says 'Start'" or "all buttons whose label starts with 'S'." In HTML, attributes are the extra information you add to elements (like `href`, `type`, `class`, `data-*`), and attribute selectors let you pick elements based on that information.

---

### Key Characteristics

- **Nine selector types**: CSS provides nine attribute selector variants, from simple presence checks to complex substring matching with case-insensitivity modifiers.
- **Specificity**: Attribute selectors have a specificity of `0,1,0`, the same as class selectors, but lower than ID selectors.
- **Case sensitivity**: By default, attribute values are matched case-sensitively in XML and case-insensitively for certain HTML attributes. The `i` and `s` modifiers override this behavior.
- **No namespace by default**: Attribute selectors match attributes in no namespace unless a namespace prefix is specified.
- **Baseline widely available**: The core attribute selectors have been available across all browsers since July 2015. The case-insensitive `i` modifier has been available since July 2022.
- **Composable**: Attribute selectors can be combined with type selectors, class selectors, and other selectors to form compound selectors.

---

### Prerequisites

Before learning CSS attribute selectors, you should understand:

1. **HTML attributes**: How attributes like `href`, `type`, `class`, `data-*`, and `lang` are written and used.
2. **Basic CSS syntax**: How properties and values are written in a rule set.
3. **CSS selectors fundamentals**: Type, class, and ID selectors.
4. **The DOM**: How HTML elements and their attributes are represented in the document tree.
5. **CSS specificity basics**: How selector specificity determines which rule wins in a conflict.

---

### Related Programming Areas

- **CSS Class and ID Selectors**: Attribute selectors are related to `.class` and `#id` selectors, which are essentially shorthand for `[class~="..."]` and `[id="..."]`.
- **CSS Pseudo-classes**: `:checked`, `:disabled`, and `:required` can be combined with attribute selectors.
- **JavaScript DOM Selection**: `document.querySelectorAll('[data-active="true"]')` uses attribute selector syntax.
- **Form Styling**: Attribute selectors are essential for styling inputs based on `type`, `required`, `disabled`, and `checked`.
- **Data Attributes**: `data-*` attributes are commonly targeted with attribute selectors for state management and JavaScript hooks.
- **Internationalization**: `[lang|="en"]` is used for language-specific styling.

---

### Core Concepts / Features

The following core concepts are covered using the uniform structure required: **Definitions → Purposes → Syntax Rules and Structure → Multiple Annotated Code Examples → Real-World Cases**.

---

## 1. Presence Selector `[attr]`

### Definitions

**Core Definition**
The presence selector matches all elements that have the specified attribute, regardless of its value.

**Technical Definition**
According to MDN Web Docs, `[attr]` "represents elements with an attribute name of attr". The selector matches if the element has the attribute set, even if the attribute value is empty or boolean (e.g., `disabled`, `required`).

**Beginner-Friendly Explanation**
`[attr]` is like saying "find all elements that have this attribute, no matter what the attribute's value is." For example, `[disabled]` finds every element that is disabled, whether it is a button, input, or select.

---

### Purposes

- **To select elements that have a specific attribute** without caring about the attribute's value.
- **To style all elements with a boolean attribute** such as `disabled`, `required`, or `checked`.
- **To target elements with custom data attributes** such as `[data-tooltip]`.
- **To provide a broad selection** that can be narrowed with additional selectors.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The name of the attribute to match (e.g., `href`, `disabled`, `data-active`) |

**Syntax Rules**

- The attribute name is written without quotes.
- The selector matches any element that has the attribute set, regardless of its value.
- Attribute names are case-insensitive in HTML but case-sensitive in XML.
- Can be combined with a type selector: `a[href]` matches only `<a>` elements with an `href` attribute.

**Constraints and Limitations**

- **Boolean attributes**: Attributes like `disabled` or `checked` match `[attr]` even if written as `disabled=""` or just `disabled`.
- **Empty attributes**: An attribute set to an empty string (e.g., `alt=""`) still matches `[attr]`.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Styling All Links with `href`

**Setup Guide**

1. Create an HTML file named `index.html`.
2. Create a CSS file named `styles.css` and link it in the HTML `<head>`.
3. Paste the code below.

**HTML (`index.html`)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Presence Selector Example</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <a href="https://example.com">Link with href</a>
    <a>Link without href (not a real link)</a>
    <button disabled>Disabled Button</button>
    <button>Enabled Button</button>
</body>
</html>
```

**CSS (`styles.css`)**

```css
/* Style all elements with an href attribute */
[href] {
    color: #0d6efd;
    text-decoration: underline;
}

/* Style all elements with a disabled attribute */
[disabled] {
    opacity: 0.5;
    cursor: not-allowed;
}
```

**Expected Output**

- The first `<a>` element is blue and underlined.
- The second `<a>` element (without `href`) is unstyled by the `[href]` rule.
- The disabled button is semi-transparent and has a "not-allowed" cursor.

**Why This Output Occurs**

`[href]` matches any element with an `href` attribute. The first link has `href`, so it receives the styling. The second link does not have `href`, so it is not matched. `[disabled]` matches the disabled button, applying the opacity and cursor styles.

---

### Real-World Cases

**Form Styling**
`[disabled] { opacity: 0.5; }` and `[required] { border-left: 3px solid red; }` provide visual feedback for form states.

**Tooltip Indicators**
`[data-tooltip] { cursor: help; }` marks elements that have tooltips without needing a class.

**Link Styling**
`a[href] { color: blue; }` distinguishes real links from placeholder anchors.

**Accessibility**
`[aria-expanded="true"] { font-weight: bold; }` styles elements based on their ARIA state.

---

## 2. Exact Value Selector `[attr="value"]`

### Definitions

**Core Definition**
The exact value selector matches elements where the attribute value is exactly equal to the specified string.

**Technical Definition**
According to MDN Web Docs, `[attr=value]` "represents elements with an attribute name of attr whose value is exactly value". The value must match the entire attribute string.

**Beginner-Friendly Explanation**
`[attr="value"]` is like saying "find elements where this attribute is exactly this value." For example, `[type="text"]` finds all inputs where the `type` attribute is exactly "text".

---

### Purposes

- **To select elements with a specific attribute value** for precise targeting.
- **To style form inputs by type** (e.g., `[type="email"]`, `[type="checkbox"]`).
- **To target links by destination** (e.g., `[href="https://example.com"]`).
- **To match data attributes with exact values** (e.g., `[data-state="active"]`).

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr="value"] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The attribute name |
| `"value"` | The exact value to match (quotes are recommended) |

**Syntax Rules**

- The value can be quoted with single or double quotes.
- Quotes are required when the value contains spaces or special characters.
- Matching is case-sensitive for most attributes in HTML; however, the HTML specification defines certain attributes as case-insensitive (e.g., `type` on `<input>`).
- Can be combined with a type selector: `input[type="text"]`.

**Constraints and Limitations**

- **Case sensitivity varies**: HTML attributes like `type`, `rel`, and `dir` are case-insensitive, while others like `class` and `id` are case-sensitive.
- **Whitespace**: The value must match exactly, including any leading or trailing whitespace.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Styling Form Inputs by Type

**HTML**

```html
<form>
    <input type="text" placeholder="Text input">
    <input type="email" placeholder="Email input">
    <input type="password" placeholder="Password input">
    <input type="checkbox" id="check">
    <label for="check">Checkbox</label>
</form>
```

**CSS**

```css
/* Style all text inputs */
input[type="text"] {
    border: 2px solid #0d6efd;
    padding: 8px;
    border-radius: 4px;
}

/* Style all email inputs */
input[type="email"] {
    border: 2px solid #155724;
    padding: 8px;
    border-radius: 4px;
}

/* Style all password inputs */
input[type="password"] {
    border: 2px solid #e94560;
    padding: 8px;
    border-radius: 4px;
}
```

**Expected Output**

Each input type has a different border color: text inputs have blue borders, email inputs have green borders, and password inputs have red-pink borders.

**Why This Output Occurs**

`input[type="text"]` matches only `<input>` elements whose `type` attribute is exactly "text". The same applies to `email` and `password`. This provides precise control over form styling without adding classes.

---

### Real-World Cases

**Form Validation**
`input[type="email"]:invalid { border-color: red; }` styles email inputs that fail validation.

**Link Styling**
`a[href="https://example.com"] { font-weight: bold; }` highlights a specific link.

**Data Attributes**
`[data-role="admin"] { background-color: gold; }` styles elements with a specific data-role value.

**Button Variants**
`button[type="submit"] { background-color: green; }` distinguishes submit buttons from reset buttons.

---

## 3. Whitespace-Separated List Selector `[attr~="value"]`

### Definitions

**Core Definition**
The whitespace-separated list selector matches elements where the attribute value is a whitespace-separated list of words, one of which is exactly the specified value.

**Technical Definition**
According to MDN Web Docs, `[attr~=value]` "represents elements with an attribute name of attr whose value is a whitespace-separated list of words, one of which is exactly value". This is the selector that underlies the `.class` selector, as `class` attributes are whitespace-separated lists.

**Beginner-Friendly Explanation**
`[attr~="value"]` is like saying "find elements where this attribute has a list of words, and one of those words is exactly this value." For example, `[class~="btn"]` finds elements whose `class` attribute contains "btn" as one of its words.

---

### Purposes

- **To match one word within a space-separated attribute list** such as `class` or `rel`.
- **To style elements based on a specific class** without using the `.class` shorthand.
- **To target elements by one of multiple values** in an attribute like `rel="friend colleague"`.
- **To combine with type selectors** for more specific targeting (e.g., `a[rel~="external"]`).

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr~="value"] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The attribute name |
| `~=` | The whitespace-separated list operator |
| `"value"` | The word to match within the list |

**Syntax Rules**

- The attribute value is treated as a whitespace-separated list of words.
- The selector matches if one of the words is exactly equal to the specified value.
- The value cannot contain whitespace.
- Matching is case-sensitive for `class` and `id` attributes in HTML.

**Constraints and Limitations**

- **Exact word match**: The value must match an entire word, not a substring. `[class~="btn"]` matches `class="btn"` but not `class="btn-primary"`.
- **No partial matching**: For partial matching, use `[attr*="value"]` or `[attr^="value"]`.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Matching One Class in a List

**HTML**

```html
<button class="btn">Default</button>
<button class="btn primary">Primary</button>
<button class="btn primary large">Large Primary</button>
<div class="btn">A div with btn class</div>
```

**CSS**

```css
/* Style all elements with "btn" in their class list */
[class~="btn"] {
    padding: 10px 20px;
    border: 2px solid #333;
    border-radius: 6px;
    cursor: pointer;
    margin-right: 8px;
}

/* Style elements with "primary" in their class list */
[class~="primary"] {
    background-color: #0d6efd;
    color: white;
    border-color: #0d6efd;
}
```

**Expected Output**

- All four elements receive the base button styling.
- The second and third buttons have a blue background and white text (because they have `primary` in their class list).
- The first button and the div have the default styling.

**Why This Output Occurs**

`[class~="btn"]` matches any element whose `class` attribute contains "btn" as one of its whitespace-separated words. The third button has `class="btn primary large"`, which is a list of three words, one of which is "btn". The `[class~="primary"]` selector matches the second and third buttons because they both have "primary" in their class list.

---

### Real-World Cases

**Link Relationship Styling**
`a[rel~="external"] { background: url('external-icon.png') no-repeat right; }` marks external links.

**Tag Systems**
`[class~="tag"]` styles all elements tagged with the "tag" class, regardless of other classes.

**Category Filtering**
`[data-category~="tech"]` matches elements categorized as "tech" within a space-separated list.

---

## 4. Hyphen-Separated Prefix Selector `[attr|="value"]`

### Definitions

**Core Definition**
The hyphen-separated prefix selector matches elements where the attribute value is exactly the specified string or begins with that string immediately followed by a hyphen.

**Technical Definition**
According to MDN Web Docs, `[attr|=value]` "represents elements with an attribute name of attr whose value can be exactly value or can begin with value immediately followed by a hyphen, - (U+002D). It is often used for language subcode matches". This selector is commonly used with the `lang` attribute to match language and its subcodes.

**Beginner-Friendly Explanation**
`[attr|="value"]` is like saying "find elements where this attribute is exactly this value, or starts with this value followed by a hyphen." For example, `[lang|="en"]` matches `lang="en"`, `lang="en-US"`, and `lang="en-GB"`.

---

### Purposes

- **To match language codes and their regional variants** using the `lang` attribute.
- **To match attribute values that use hyphen-separated naming conventions**.
- **To target elements by a base category** where variants are separated by hyphens.
- **To style content based on language** for typographic or translation purposes.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr|="value"] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The attribute name |
| `|=` | The hyphen-separated prefix operator |
| `"value"` | The prefix string |

**Syntax Rules**

- Matches if the attribute value is exactly `value` or begins with `value-`.
- The hyphen must immediately follow the value (no spaces).
- Commonly used with `lang`, `hreflang`, and custom attributes with hyphenated conventions.
- Matching is case-insensitive for the `lang` attribute in HTML.

**Constraints and Limitations**

- **Only hyphen separator**: The selector only recognizes the hyphen (`-`) as the separator, not underscores or spaces.
- **Cannot be used for arbitrary prefixes**: For arbitrary prefixes without a hyphen separator, use `[attr^="value"]`.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Language-Specific Styling

**HTML**

```html
<p lang="en">This is English text.</p>
<p lang="en-US">This is American English text.</p>
<p lang="en-GB">This is British English text.</p>
<p lang="fr">This is French text.</p>
<p lang="zh-CN">This is Simplified Chinese text.</p>
```

**CSS**

```css
/* Style all English content (en, en-US, en-GB) */
[lang|="en"] {
    font-family: Georgia, serif;
    border-left: 4px solid #0d6efd;
    padding-left: 12px;
}

/* Style all Chinese content (zh, zh-CN, zh-TW) */
[lang|="zh"] {
    font-family: 'Noto Sans SC', sans-serif;
    border-left: 4px solid #e94560;
    padding-left: 12px;
}
```

**Expected Output**

- The first three paragraphs (English variants) have a blue left border and use Georgia.
- The French paragraph is unstyled.
- The Chinese paragraph has a red-pink left border and uses Noto Sans SC.

**Why This Output Occurs**

`[lang|="en"]` matches `lang="en"`, `lang="en-US"`, and `lang="en-GB"` because the attribute value is either exactly "en" or begins with "en-". The French paragraph (`lang="fr"`) does not match. The Chinese paragraph matches `[lang|="zh"]` because its `lang` value begins with "zh-".

---

### Real-World Cases

**Multilingual Typography**
`[lang|="ja"] { font-family: 'Noto Sans JP', sans-serif; }` applies Japanese-specific fonts to Japanese content.

**Quote Styling**
`[lang|="fr"] q { quotes: '« ' ' »'; }` applies French quotation marks.

**Hyphenated Categories**
`[data-type|="user"]` matches `data-type="user"` and `data-type="user-admin"`.

**Regional Pricing**
`[data-currency|="USD"]` matches all USD-denominated prices.

---

## 5. Prefix Selector `[attr^="value"]`

### Definitions

**Core Definition**
The prefix selector matches elements where the attribute value begins exactly with the specified string.

**Technical Definition**
According to MDN Web Docs, `[attr^=value]` "represents elements with an attribute name of attr whose value is prefixed (preceded) by value". The match is case-sensitive for most attributes unless the `i` modifier is used.

**Beginner-Friendly Explanation**
`[attr^="value"]` is like saying "find elements where this attribute starts with this string." For example, `[href^="https://"]` finds all links that start with "https://".

---

### Purposes

- **To select elements by URL scheme** (e.g., all HTTPS links).
- **To match elements with attributes that begin with a specific prefix** (e.g., `data-`, `box-`).
- **To style elements based on a naming convention** where the prefix indicates category.
- **To target all elements of a type** without listing each one individually.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr^="value"] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The attribute name |
| `^=` | The prefix operator |
| `"value"` | The prefix string to match |

**Syntax Rules**

- Matches if the attribute value begins with the specified string.
- The match is case-sensitive unless the `i` modifier is used.
- The value can be quoted or unquoted (quotes are recommended for clarity).
- Can be combined with type selectors: `a[href^="https"]`.

**Constraints and Limitations**

- **Case sensitivity**: By default, matching is case-sensitive for most attributes. Use `i` for case-insensitive matching.
- **Empty string**: `[attr^=""]` matches all elements that have the attribute, regardless of value.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Styling Secure Links

**HTML**

```html
<a href="https://example.com">Secure Link</a>
<a href="http://example.com">Insecure Link</a>
<a href="mailto:test@example.com">Email Link</a>
<a href="tel:+1234567890">Phone Link</a>
```

**CSS**

```css
/* Style all links that start with "https" */
a[href^="https"] {
    color: #155724;
    font-weight: bold;
}

/* Style all links that start with "http:" (but not https) */
a[href^="http:"] {
    color: #721c24;
    text-decoration: line-through;
}
```

**Expected Output**

- The secure link is green and bold.
- The insecure link is red and struck through.
- The email and phone links are unstyled.

**Why This Output Occurs**

`a[href^="https"]` matches only the first link because its `href` value begins with "https". The insecure link's `href` begins with "http:" (with a colon), so it matches `a[href^="http:"]`. The email and phone links begin with "mailto:" and "tel:" respectively, so they do not match either selector.

---

### Real-World Cases

**External Link Indicators**
`a[href^="http"]:not([href*="yourdomain.com"])` styles external links differently.

**Data Attribute Filtering**
`[data-*]` matches all elements with any data attribute, while `[data-user]` matches a specific data attribute.

**Icon Font Naming**
`[class^="icon-"]` styles all elements whose class begins with "icon-".

**File Type Indicators**
`a[href$=".pdf"]` marks PDF links with an icon.

---

## 6. Suffix Selector `[attr$="value"]`

### Definitions

**Core Definition**
The suffix selector matches elements where the attribute value ends exactly with the specified string.

**Technical Definition**
According to MDN Web Docs, `[attr$=value]` "represents elements with an attribute name of attr whose value is suffixed (followed) by value". The match is case-sensitive for most attributes unless the `i` modifier is used.

**Beginner-Friendly Explanation**
`[attr$="value"]` is like saying "find elements where this attribute ends with this string." For example, `[href$=".pdf"]` finds all links to PDF files.

---

### Purposes

- **To select elements by file extension** (e.g., all PDF links).
- **To match elements with attributes that end with a specific suffix** (e.g., `-active`, `-disabled`).
- **To style elements based on a naming convention** where the suffix indicates state or type.
- **To target all elements of a type** based on their URL ending.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr$="value"] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The attribute name |
| `$=` | The suffix operator |
| `"value"` | The suffix string to match |

**Syntax Rules**

- Matches if the attribute value ends with the specified string.
- The match is case-sensitive unless the `i` modifier is used.
- Can be combined with type selectors: `a[href$=".pdf"]`.

**Constraints and Limitations**

- **Case sensitivity**: By default, matching is case-sensitive for most attributes. Use `i` for case-insensitive matching.
- **Empty string**: `[attr$=""]` matches all elements that have the attribute, regardless of value.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Styling Links by File Type

**HTML**

```html
<a href="document.pdf">PDF Document</a>
<a href="report.docx">Word Document</a>
<a href="image.png">PNG Image</a>
<a href="archive.zip">ZIP Archive</a>
```

**CSS**

```css
/* Style all PDF links with a red border */
a[href$=".pdf"] {
    border: 2px solid #e94560;
    padding: 4px 8px;
    border-radius: 4px;
}

/* Style all image links with a green border */
a[href$=".png"],
a[href$=".jpg"],
a[href$=".gif"] {
    border: 2px solid #155724;
    padding: 4px 8px;
    border-radius: 4px;
}

/* Style all ZIP links with a blue border */
a[href$=".zip"] {
    border: 2px solid #0d6efd;
    padding: 4px 8px;
    border-radius: 4px;
}
```

**Expected Output**

- The PDF link has a red border.
- The PNG image link has a green border.
- The ZIP link has a blue border.
- The Word document link is unstyled.

**Why This Output Occurs**

`a[href$=".pdf"]` matches only the link whose `href` ends with ".pdf". The image selector uses a grouped list to match `.png`, `.jpg`, and `.gif` extensions. The ZIP link matches `a[href$=".zip"]`.

---

### Real-World Cases

**Download Indicators**
`a[href$=".pdf"]::after { content: " (PDF)"; }` adds a file type indicator to PDF links.

**State Classes**
`[class$="-active"]` matches classes like `tab-active`, `menu-active`, and `nav-active`.

**Email Domain Filtering**
`[href$="@company.com"]` styles email links to a specific domain.

**File Extension Styling**
`[href$=".zip"], [href$=".tar"], [href$=".gz"]` styles all archive download links.

---

## 7. Substring Selector `[attr*="value"]`

### Definitions

**Core Definition**
The substring selector matches elements where the attribute value contains the specified substring at least once.

**Technical Definition**
According to MDN Web Docs, `[attr*=value]` "represents elements with an attribute name of attr whose value contains at least one occurrence of value within the string". The match is case-sensitive for most attributes unless the `i` modifier is used.

**Beginner-Friendly Explanation**
`[attr*="value"]` is like saying "find elements where this attribute contains this string anywhere inside it." For example, `[class*="btn"]` finds any element whose class contains "btn" as a substring, whether it is `btn`, `btn-primary`, or `my-btn`.

---

### Purposes

- **To match elements based on a substring anywhere in the attribute value**.
- **To style elements with partial matches** in URLs or class names.
- **To target elements with dynamically generated attribute values** containing known substrings.
- **To combine with type selectors** for broad but targeted matching.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr*="value"] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `attr` | The attribute name |
| `*=` | The substring operator |
| `"value"` | The substring to search for |

**Syntax Rules**

- Matches if the attribute value contains the specified substring anywhere.
- The match is case-sensitive unless the `i` modifier is used.
- The substring can be of any length.
- Can be combined with type selectors: `input[class*="validate"]`.

**Constraints and Limitations**

- **Case sensitivity**: By default, matching is case-sensitive for most attributes. Use `i` for case-insensitive matching.
- **Performance**: Substring matching can be slower than prefix or suffix matching on large DOMs.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Matching Class Substrings

**HTML**

```html
<button class="btn">Default</button>
<button class="btn-primary">Primary</button>
<button class="btn-secondary">Secondary</button>
<div class="notification-btn">Notification</div>
<button class="submit">Submit</button>
```

**CSS**

```css
/* Style all elements whose class contains "btn" anywhere */
[class*="btn"] {
    padding: 8px 16px;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    margin-right: 8px;
}
```

**Expected Output**

- The first three buttons and the div all have the base styling.
- The submit button is unstyled.

**Why This Output Occurs**

`[class*="btn"]` matches any element whose `class` attribute contains "btn" as a substring. The first three buttons have `btn` in their class names. The div has `notification-btn`, which also contains "btn". The submit button has only `submit`, so it does not match.

---

### Real-World Cases

**URL Substring Matching**
`a[href*="example.com"]` matches all links to a specific domain, including subdomains.

**Dynamic Class Matching**
`[class*="state-"]` matches all elements with a class containing "state-".

**Search Result Highlighting**
`[data-highlight*="keyword"]` highlights elements containing a search keyword.

**Form Validation**
`input[class*="validate-"]` styles all inputs with a validation-related class.

---

## 8. Case-Insensitive Modifier `[attr="value" i]`

### Definitions

**Core Definition**
The case-insensitive modifier forces attribute value matching to ignore case differences within the ASCII range.

**Technical Definition**
According to MDN Web Docs, adding an `i` (or `I`) before the closing bracket "causes the value to be compared case-insensitively (for characters within the ASCII range)". This modifier can be used with any attribute selector operator (`=`, `~=`, `|=`, `^=`, `$=`, `*=`).

**Beginner-Friendly Explanation**
`[attr="value" i]` is like saying "match this attribute value, but do not worry about uppercase or lowercase letters." For example, `[type="text" i]` matches `type="text"`, `type="TEXT"`, and `type="Text"`.

---

### Purposes

- **To match attribute values regardless of letter case** for more forgiving styling.
- **To handle user-generated content** where casing may vary.
- **To style HTML attributes that are case-insensitive** according to the HTML specification.
- **To simplify selectors** when the exact case of an attribute value is unknown.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr="value" i] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `i` | The case-insensitivity modifier (placed before the closing bracket) |
| `attr` and `value` | The attribute and value to match |

**Syntax Rules**

- The `i` modifier can be used with any attribute selector operator.
- The modifier affects only ASCII characters (A–Z and a–z).
- The modifier must be separated from the value by whitespace.
- The modifier can also be written as uppercase `I`.

**Constraints and Limitations**

- **ASCII only**: The `i` modifier only affects characters within the ASCII range. Non-ASCII characters (e.g., accented letters) are still matched case-sensitively.
- **Browser support**: The `i` modifier has been available across browsers since July 2022.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Case-Insensitive Matching

**HTML**

```html
<input type="text" placeholder="lowercase">
<input type="TEXT" placeholder="UPPERCASE">
<input type="Text" placeholder="Mixed Case">
<input type="email" placeholder="email">
```

**CSS**

```css
/* Match text inputs regardless of case */
input[type="text" i] {
    border: 2px solid #0d6efd;
    padding: 8px;
}

/* Match email inputs regardless of case */
input[type="EMAIL" i] {
    border: 2px solid #155724;
    padding: 8px;
}
```

**Expected Output**

All three text inputs (lowercase, uppercase, and mixed case) have blue borders. The email input has a green border.

**Why This Output Occurs**

`input[type="text" i]` matches `type="text"`, `type="TEXT"`, and `type="Text"` because the `i` modifier makes the comparison case-insensitive. Similarly, `input[type="EMAIL" i]` matches `type="email"` because the case is ignored.

---

### Real-World Cases

**User Input Validation**
`[data-status="active" i]` matches "active", "Active", and "ACTIVE" in user-generated data.

**Cross-Browser Consistency**
For HTML attributes with inconsistent casing across browsers, the `i` modifier ensures consistent matching.

**International Content**
`[lang="en" i]` matches language codes regardless of case.

**Dynamic Data**
When attribute values come from external sources with unpredictable casing, the `i` modifier ensures matches.

---

## 9. Case-Sensitive Modifier `[attr="value" s]`

### Definitions

**Core Definition**
The case-sensitive modifier forces attribute value matching to be case-sensitive within the ASCII range.

**Technical Definition**
According to MDN Web Docs, adding an `s` (or `S`) before the closing bracket "causes the value to be compared case-sensitively (for characters within the ASCII range)". This modifier is useful in environments like HTML where certain attributes are case-insensitive by default.

**Beginner-Friendly Explanation**
`[attr="value" s]` is like saying "match this attribute value exactly, paying attention to uppercase and lowercase letters." For example, `[data-code="ABC" s]` matches only `data-code="ABC"`, not `data-code="abc"`.

---

### Purposes

- **To enforce case-sensitive matching** for attributes that are case-insensitive by default in HTML.
- **To distinguish between values that differ only in case**.
- **To ensure precise matching** in XML documents and HTML documents with mixed-case attribute values.
- **To override the default case-insensitivity** of certain HTML attributes.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
[attr="value" s] {
    /* declarations */
}
```

**Breakdown**

| Component | Meaning |
|-----------|---------|
| `s` | The case-sensitivity modifier (placed before the closing bracket) |
| `attr` and `value` | The attribute and value to match |

**Syntax Rules**

- The `s` modifier can be used with any attribute selector operator.
- The modifier affects only ASCII characters (A–Z and a–z).
- The modifier must be separated from the value by whitespace.
- The modifier can also be written as uppercase `S`.
- The `s` modifier is useful in HTML where attributes like `type` are case-insensitive by default.

**Constraints and Limitations**

- **ASCII only**: The `s` modifier only affects characters within the ASCII range.
- **Browser support**: The `s` modifier has been available across browsers since July 2022.
- **Less commonly needed**: In HTML, most attribute values are already case-sensitive (e.g., `class`, `id`), so the `s` modifier is primarily useful for attributes that are case-insensitive by default.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Forcing Case-Sensitive Matching

**HTML**

```html
<input type="text" placeholder="lowercase type">
<input type="TEXT" placeholder="uppercase type">
<input data-code="ABC" placeholder="ABC">
<input data-code="abc" placeholder="abc">
```

**CSS**

```css
/* Match only lowercase "text" type */
input[type="text" s] {
    border: 2px solid #0d6efd;
    padding: 8px;
}

/* Match only uppercase "ABC" data-code */
[data-code="ABC" s] {
    background-color: #fff3cd;
    border: 2px solid #856404;
}
```

**Expected Output**

- Only the first input (with `type="text"`) gets the blue border. The second input (with `type="TEXT"`) does not match because the `s` modifier forces case-sensitive matching.
- Only the third input (with `data-code="ABC"`) gets the amber background. The fourth input (with `data-code="abc"`) does not match.

**Why This Output Occurs**

`input[type="text" s]` matches only the exact lowercase value "text". `type="TEXT"` does not match because of the `s` modifier. Similarly, `[data-code="ABC" s]` matches only the uppercase "ABC" value, not the lowercase "abc".

---

### Real-World Cases

**Code Matching**
`[data-code="USD" s]` distinguishes between currency codes with different casing.

**Language Codes**
`[lang="en" s]` matches only the exact lowercase "en" language code.

**Product Identifiers**
`[data-sku="ABC123" s]` ensures exact matching for product SKUs that are case-sensitive.

**Version Strings**
`[data-version="v2" s]` distinguishes between "v2" and "V2" version tags.

---

## Constraints, Limitations, and Version-Specific Notes

- **Specificity**: Attribute selectors have a specificity of `0,1,0`, the same as class selectors. They are less specific than ID selectors (`1,0,0`) and more specific than type selectors (`0,0,1`).
- **Case sensitivity varies by attribute**: In HTML, attributes like `class` and `id` are case-sensitive, while attributes like `type`, `rel`, and `dir` are case-insensitive. The `i` and `s` modifiers override this behavior.
- **ASCII-only modifiers**: The `i` and `s` modifiers only affect ASCII characters; non-ASCII characters are matched case-sensitively.
- **Browser support**: The core attribute selectors have been available since July 2015. The `i` and `s` modifiers have been available since July 2022.
- **Namespace handling**: Attribute selectors match attributes in no namespace by default. To match namespaced attributes, a namespace prefix is required.
- **Performance**: Substring matching (`*=`, `^=`, `$=`) can be slower than exact matching (`=`, `~=`, `|=`) on large DOMs.
- **Quoting**: Values containing spaces or special characters must be quoted. Quotes are recommended even when optional, for clarity.
- **Empty values**: `[attr=""]` matches elements where the attribute is set to an empty string. `[attr^=""]`, `[attr$=""]`, and `[attr*=""]` match all elements that have the attribute.
- **Boolean attributes**: Attributes like `disabled`, `checked`, and `required` match `[attr]` even when written without a value.

---

## References

- MDN Web Docs — Attribute Selectors Reference - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/Attribute_selectors
- MDN Web Docs — Attribute Selectors Tutorial - https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Attribute_selectors
- MDN Web Docs — CSS Selectors Guide - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Selectors
- W3C — Selectors Level 4 Specification - https://www.w3.org/TR/selectors-4/
- W3C — Selectors Level 3 Specification - https://www.w3.org/TR/selectors-3/
- W3C — CSS 2.1 Specification, Section 5.8: Attribute Selectors - https://www.w3.org/TR/CSS21/selector.html#attribute-selectors
- MDN Web Docs — Specificity - https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity
- Web Platform Features Explorer — Case-insensitive Attribute Selector - https://web-platform-dx.github.io/web-features-explorer/features/case-insensitive-attribute-selector/
- CSS-Tricks — The Skinny on CSS Attribute Selectors - https://css-tricks.com/attribute-selectors/