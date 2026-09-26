# HTML Link Targets: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Link targets are the destinations and browsing contexts that an `<a>` element‘s `href` attribute can point to, including same-page fragments, new browsing contexts, external websites, downloadable resources, email addresses, and telephone numbers.

**Technical Definition**

Link targets encompass the `href` attribute values (URLs, fragment identifiers, and URL schemes) and the `target` attribute values (named or keyword browsing contexts) that determine where and how a hyperlink’s destination is resolved and displayed. The `target` attribute, if specified, must contain a valid navigable target name or keyword, which specifies which navigable is to be used as the default when hyperlinks cause navigation. URL schemes such as `mailto:` and `tel:` initiate external applications, while the `download` attribute instructs the browser to download the linked resource rather than navigate to it.

**Beginner-Friendly Explanation**

A link target is simply “where the link goes” and “how it opens.” You can link to another page, a specific section of the same page, an email address, a phone number, or a file to download. You can also tell the browser whether to open the link in the same tab or a new one. This cheat sheet covers all the common target types you‘ll use.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`target` attribute** | Controls the browsing context in which the link opens (same tab, new tab, parent frame, top frame) |
| **Fragment identifiers** | Same-page navigation uses `#id` to scroll to a specific element |
| **URL schemes** | `mailto:` and `tel:` trigger external applications |
| **`download` attribute** | Prompts the browser to save the resource rather than navigate to it |
| **Security concerns** | `target="_blank"` requires `rel="noopener"` or `rel="noreferrer"` to prevent tabnabbing |
| **Accessibility** | Link text must be descriptive regardless of target type |

---

### Prerequisites

- Basic familiarity with HTML document structure
- Understanding of the `<a>` element and the `href` attribute
- Awareness of absolute, relative, and root-relative URLs

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Link purpose and target behaviour affect screen reader users
- **Web Security** – `rel="noopener"` prevents tabnabbing attacks
- **Mobile Web** – `tel:` and `mailto:` links enhance mobile user experience
- **SEO** – External link behaviour and `rel` attributes influence search engine crawling
- **Progressive Web Apps** – Downloadable resources and deep linking

---

## Core Concepts / Features

---

### 1. Same-Page Navigation (Fragment Identifiers)

#### Definitions

**Core Definition**

Same-page navigation uses a fragment identifier (`#id`) in the `href` attribute to scroll to a specific element within the current document.

**Technical Definition**

A fragment identifier is the part of a URL that follows the `#` symbol. For an HTML document, the indicated part is the element whose `id` attribute matches the fragment, or an `<a>` element with a `name` attribute matching the fragment (though the `name` attribute is deprecated in HTML5). When a user clicks a fragment link, the browser scrolls the target element into view without reloading the page.

**Beginner-Friendly Explanation**

Same-page navigation lets you create links that jump to a specific section of the same page — like a table of contents that scrolls down to a chapter when clicked. You give the target element an `id`, then link to it with `#id`.

#### Purposes

- To create table-of-contents links
- To provide “back to top” links
- To enable in-page navigation without page reloads
- To link to specific sections of a document

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="#section-id">Go to Section</a>
...
<h2 id="section-id">Section Title</h2>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `#` | Fragment identifier separator |
| `section-id` | The value of the target element‘s `id` attribute |

**Syntax Rules**

- The `id` value must be unique within the document
- The fragment identifier is case-sensitive
- Multiple elements must not share the same `id`
- The `id` must not contain spaces

**Constraints and Limitations**

- If no element matches the fragment, the browser does nothing (no scroll)
- Fragment identifiers are not sent to the server
- Using `name` attribute for fragment targets is deprecated

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Table of Contents**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Same-Page Navigation</title>
</head>
<body>
    <h1>Table of Contents</h1>
    <ul>
        <li><a href="#introduction">Introduction</a></li>
        <li><a href="#installation">Installation</a></li>
    </ul>

    <h2 id="introduction">Introduction</h2>
    <p>Welcome to this guide...</p>

    <h2 id="installation">Installation</h2>
    <p>To install, run...</p>

    <!-- Back to top link -->
    <p><a href="#top">Back to top</a></p>
</body>
</html>
```

**Expected Output**

Clicking “Introduction” scrolls to the Introduction heading. Clicking “Back to top” scrolls to the top of the page.

**Why This Output Occurs**

The fragment identifier `#introduction` matches the `id="introduction"` on the `<h2>` element. The browser scrolls that element into view. The `#top` fragment scrolls to the top of the document (browsers have a built-in `top` fragment target).

---

**Example 2: Linking to a Specific Element**

```html
<p>Read the <a href="#disclaimer">disclaimer</a> before proceeding.</p>
<!-- ... long content ... -->
<p id="disclaimer"><strong>Disclaimer:</strong> This information is for educational purposes only.</p>
```

**Expected Output**

Clicking “disclaimer” scrolls directly to the disclaimer paragraph.

**Why This Output Occurs**

The `id="disclaimer"` provides a fragment target for the link. The browser scrolls the paragraph into view and updates the URL with `#disclaimer`.

#### Real-World Cases

**Case 1: Documentation Tables of Contents**

Technical documentation uses fragment identifiers for chapter and section navigation.

**Case 2: FAQ Pages**

FAQ pages use “Back to top” links to return users to the question list.

**Case 3: Long-Form Articles**

News and magazine articles use fragment identifiers for internal cross-references.

---

### 2. New Browsing Context (`target="_blank"`)

#### Definitions

**Core Definition**

The `target="_blank"` attribute value opens the linked document in a new browsing context (tab or window).

**Technical Definition**

The `target` attribute specifies the browsing context to use. The keyword `_blank` indicates that the link should be opened in a new browsing context. According to the HTML specification, if the `target` attribute is specified with `_blank`, the user agent must open the link in a new, unnamed browsing context. When using `target="_blank"`, the `rel="noopener"` or `rel="noreferrer"` attribute should be added to prevent the new page from accessing the `window.opener` property.

**Beginner-Friendly Explanation**

Adding `target="_blank"` to a link makes it open in a new tab (or window, depending on browser settings). This is useful for external links so users don‘t leave your site. However, it introduces a security risk: the new page can control your page via `window.opener`. Always add `rel="noopener"` to prevent this.

#### Purposes

- To open external links without navigating away from the current page
- To keep users on the original page while viewing referenced content
- To open downloadable resources in a new context
- To compare two pages side by side

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="https://example.com" target="_blank" rel="noopener">External Link</a>
```

**Target Attribute Values**

| Value | Description |
|---|---|
| `_self` | Opens in the same browsing context (default) |
| `_blank` | Opens in a new browsing context |
| `_parent` | Opens in the parent browsing context |
| `_top` | Opens in the top-level browsing context |

**Syntax Rules**

- The `target` attribute can be a keyword (`_blank`) or a named browsing context
- When `target="_blank"`, always add `rel="noopener"` or `rel="noreferrer"`
- The `rel` attribute value is a space-separated list of link types

**Constraints and Limitations**

- Some browsers may block pop-ups from `target="_blank"` links
- Without `rel="noopener"`, the new page can access the original page’s `window` object via `window.opener`
- Adding `rel="noopener"` prevents this access and ensures the new page runs in a separate process
- `rel="noreferrer"` has the same effect and also prevents the `Referer` header from being sent

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Secure External Link**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>New Tab Link</title>
</head>
<body>
    <!-- Secure external link: target="_blank" + rel="noopener" -->
    <p>
        Visit the
        <a href="https://developer.mozilla.org/" target="_blank" rel="noopener">MDN Web Docs</a>
        for documentation.
    </p>
</body>
</html>
```

**Expected Output**

Clicking the link opens MDN Web Docs in a new tab. The original page remains open.

**Why This Output Occurs**

The `target="_blank"` attribute tells the browser to open the link in a new browsing context. The `rel="noopener"` attribute prevents the new page from accessing the original page’s `window.opener` property.

---

**Example 2: Unsafe vs. Safe Link**

```html
<!-- UNSAFE: target="_blank" without rel="noopener" -->
<a href="https://untrusted.example.com" target="_blank">Unsafe Link</a>

<!-- SAFE: target="_blank" with rel="noopener" -->
<a href="https://untrusted.example.com" target="_blank" rel="noopener">Safe Link</a>
```

**Expected Output**

Both open in a new tab, but only the safe link prevents the new page from controlling the original page.

**Why This Output Occurs**

Without `rel="noopener"`, the new page can use `window.opener.location = "https://phishing-site.com"` to redirect the original page. The `rel="noopener"` attribute blocks this access.

#### Real-World Cases

**Case 1: Social Media Sharing**

Share links use `target="_blank"` to open sharing dialogs without leaving the current page.

**Case 2: Documentation References**

Documentation links to external specifications with `target="_blank"` and `rel="noopener"`.

**Case 3: E-Commerce External Links**

Product pages link to manufacturer websites in new tabs.

---

### 3. External Websites

#### Definitions

**Core Definition**

External websites are destinations on different domains, typically linked using absolute URLs.

**Technical Definition**

An external link points to a resource on a different origin (domain, protocol, or port). External links use absolute URLs containing the protocol and domain name. The `rel` attribute can specify relationships such as `external`, `nofollow`, or `noopener`. The `target` attribute controls the browsing context.

**Beginner-Friendly Explanation**

An external link goes to a different website. You use the full address (like `https://example.com`) in the `href` attribute. It‘s common to open external links in a new tab so users don’t leave your site, but you must add `rel="noopener"` for security.

#### Purposes

- To reference sources and further reading
- To link to partner or affiliate websites
- To cite external documentation or specifications
- To provide social media or external service links

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="https://external-site.com/page" target="_blank" rel="noopener">Visit External Site</a>
```

**Syntax Rules**

- Use absolute URLs with the protocol (`https://`)
- Add `target="_blank"` to open in a new tab
- Add `rel="noopener"` for security
- Optionally add `rel="external"` to indicate the link is external
- Use `rel="nofollow"` for sponsored or untrusted links

**Constraints and Limitations**

- External links may be broken if the destination changes
- Without `rel="noopener"`, external pages can manipulate the original page
- Some browsers block `target="_blank"` pop-ups

#### Annotated Complete Step-by-Step Code Examples

**Example 1: External Reference Link**

```html
<p>
    According to the
    <a href="https://www.w3.org/WAI/" target="_blank" rel="noopener">W3C Web Accessibility Initiative</a>,
    accessibility is essential for developers and organisations.
</p>
```

**Expected Output**

The link opens the W3C WAI website in a new tab.

**Why This Output Occurs**

The absolute URL points to an external domain. The `target="_blank"` opens it in a new tab, and `rel="noopener"` prevents the external page from accessing the original page.

---

**Example 2: Affiliate Link with `nofollow`**

```html
<a href="https://affiliate.example.com/product" target="_blank" rel="noopener nofollow sponsored">
    Buy Product
</a>
```

**Expected Output**

The link opens in a new tab and is marked as sponsored/nofollow for search engines.

**Why This Output Occurs**

The `rel` attribute contains multiple link types: `noopener` (security), `nofollow` (SEO), and `sponsored` (indicates paid link).

#### Real-World Cases

**Case 1: News Article Citations**

News articles link to external sources with `target="_blank"` and `rel="noopener"`.

**Case 2: Affiliate Marketing**

Affiliate links use `rel="sponsored nofollow"` to comply with search engine guidelines.

**Case 3: Social Media Profiles**

Company websites link to their social media profiles in new tabs.

---

### 4. Downloadable Resources

#### Definitions

**Core Definition**

Downloadable resources are files that the browser saves to the user‘s device rather than displaying in the browser, triggered by the `download` attribute on an `<a>` element.

**Technical Definition**

The `download` attribute, when specified on an `<a>` or `<area>` element, indicates that the browser should download the linked resource rather than navigate to it. The attribute can optionally contain a value specifying the filename to use when saving. In cross-origin situations, the `download` attribute must be combined with the `Content-Disposition: attachment` HTTP header to avoid user warnings.

**Beginner-Friendly Explanation**

The `download` attribute tells the browser “don’t open this file — save it to the user‘s computer.” You can also suggest a filename. It only works for files hosted on the same domain as your website.

#### Purposes

- To provide files for download (PDFs, images, documents)
- To specify a custom filename for downloaded files
- To create “Download” buttons
- To bundle resources for offline use

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="file.pdf" download>Download PDF</a>
<a href="file.pdf" download="custom-name.pdf">Download with Custom Name</a>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `download` | Boolean attribute; presence triggers download |
| `download="filename"` | Optional value; specifies the saved filename |

**Syntax Rules**

- The `download` attribute only works for same-origin resources
- The filename value is a suggestion; the `Content-Disposition` header can override it
- For cross-origin resources, the server must send `Content-Disposition: attachment`

**Constraints and Limitations**

- The `download` attribute does not work for cross-origin resources without the `Content-Disposition` header
- Users can still choose to open the file rather than save it, depending on browser settings
- The attribute has no effect when the user enters the URL directly

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Download Link**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Download Demo</title>
</head>
<body>
    <!-- Download with original filename -->
    <p><a href="/files/report.pdf" download>Download Annual Report (PDF)</a></p>

    <!-- Download with custom filename -->
    <p><a href="/files/report.pdf" download="2026-annual-report.pdf">Download with Custom Name</a></p>
</body>
</html>
```

**Expected Output**

Clicking the link downloads the PDF file instead of opening it in the browser.

**Why This Output Occurs**

The `download` attribute instructs the browser to save the resource. The optional value specifies the filename.

---

**Example 2: Download with `target="_blank"`**

```html
<a href="document.pdf" download="document.pdf" target="_blank">Download PDF</a>
```

**Expected Output**

The file downloads in a new tab (or the download starts without navigating away).

**Why This Output Occurs**

The `target="_blank"` ensures the current page is not navigated away from during the download.

#### Real-World Cases

**Case 1: Software Downloads**

Software websites use `download` for installer files.

**Case 2: Resume/CV Downloads**

Personal websites use `download` for PDF resumes.

**Case 3: E-Book Distribution**

Publishers use `download` for e-book files.

---

### 5. Email Links (`mailto:`)

#### Definitions

**Core Definition**

An email link uses the `mailto:` URL scheme to open the user’s default email client with a new message pre-addressed.

**Technical Definition**

The `mailto:` scheme is a URL scheme that creates a hyperlink to an email address. The syntax is `mailto:recipient@example.com`. Additional parameters can be appended using a query string (`?`) to pre-fill the subject (`subject=`), CC (`cc=`), BCC (`bcc=`), and body (`body=`) fields. Multiple recipients are separated by commas.

**Beginner-Friendly Explanation**

A `mailto:` link opens the user‘s email app when clicked, with the “To” field already filled in. You can also pre-fill the subject and message body. It’s a quick way to let users contact you without typing your email address.

#### Purposes

- To provide a one-click way to send an email
- To pre-fill subject and body for user convenience
- To include multiple recipients (CC, BCC)
- To reduce friction for contact forms

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="mailto:recipient@example.com">Email Us</a>
```

**Full Syntax with Parameters**

```html
<a href="mailto:info@example.com?subject=Subject&cc=cc@example.com&body=Message%20body">Email</a>
```

**Parameter Breakdown**

| Parameter | Description |
|---|---|
| `mailto:` | Recipient email address (required) |
| `?subject=` | Email subject line (URL-encoded) |
| `&cc=` | Carbon copy recipient(s) |
| `&bcc=` | Blind carbon copy recipient(s) |
| `&body=` | Email body text (URL-encoded) |

**Syntax Rules**

- The first parameter is separated by `?`; subsequent parameters by `&`
- Spaces and special characters must be URL-encoded (`%20` for spaces)
- Line breaks in the body should be encoded as `%0A`
- Multiple recipients are separated by commas

**Constraints and Limitations**

- `mailto:` links depend on the user having an email client configured
- Webmail users (Gmail, Outlook.com) may not have a default mail client
- Long bodies can be truncated by some email clients

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Email Link**

```html
<a href="mailto:info@example.com">Contact Us</a>
```

**Expected Output**

Clicking opens the default email client with “info@example.com” in the “To” field.

**Why This Output Occurs**

The `mailto:` scheme tells the browser to hand off the email address to the operating system‘s default email application.

---

**Example 2: Full-Featured Email Link**

```html
<a href="mailto:info@example.com?subject=Website%20Inquiry&cc=manager@example.com&body=Hello%2C%0A%0AI%20have%20a%20question%20about...">
    Send Email
</a>
```

**Expected Output**

Opens the email client with the recipient, subject, CC, and body pre-filled.

**Why This Output Occurs**

The query string parameters pre-fill the email fields. `%20` represents spaces, and `%0A` represents line breaks.

#### Real-World Cases

**Case 1: Contact Pages**

Company contact pages use `mailto:` links for direct email contact.

**Case 2: Feedback Forms**

“Send Feedback” links use `mailto:` with a pre-filled subject and body.

**Case 3: Support Emails**

Help documentation includes `mailto:` links for support requests.

---

### 6. Telephone Links (`tel:`)

#### Definitions

**Core Definition**

A telephone link uses the `tel:` URL scheme to initiate a phone call when clicked on a device with telephony capability.

**Technical Definition**

The `tel:` scheme is a URL scheme that creates a hyperlink to a telephone number. On devices with phone capabilities, clicking the link initiates a call (usually with a confirmation prompt). On desktop browsers without voice call support, the link may open the default telephony application. The syntax is `tel:+[country code]-[area code]-[number]`.

**Beginner-Friendly Explanation**

A `tel:` link lets users tap a phone number to call it directly. It‘s especially useful on mobile devices. Always use the international format (`+1-555-123-4567`) so the number works from anywhere.

#### Purposes

- To enable click-to-call on mobile devices
- To provide one-tap dialling for customer support numbers
- To improve mobile user experience for contact pages
- To reduce friction for users needing to call

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="tel:+1-555-123-4567">Call +1 (555) 123-4567</a>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `tel:` | URL scheme for telephone numbers |
| `+` | International dialling prefix |
| `1` | Country code (e.g., 1 for US/Canada) |
| `555-123-4567` | Area code and local number |

**Syntax Rules**

- Always use the international dialling format: `+` + country code + area code + number
- Separate segments with hyphens for readability
- The visible link text can be formatted with spaces and parentheses
- On devices without phone capability, the browser may open a telephony app

**Constraints and Limitations**

- Requires a device with telephony capability (mobile phone, VoIP app)
- Desktop browsers may open a default telephony app
- Some browsers automatically detect phone numbers; use `<meta name="format-detection" content="telephone=no">` to disable this

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Telephone Link**

```html
<a href="tel:+1-555-123-4567">Call +1 (555) 123-4567</a>
```

**Expected Output**

On a mobile device, tapping the link prompts to call the number. On desktop, it may open a telephony app.

**Why This Output Occurs**

The `tel:` scheme signals to the device that the number should be dialled. The international format ensures the call connects regardless of the user’s location.

---

**Example 2: Telephone Link with Country Code**

```html
<a href="tel:+44-20-7946-0958">Call +44 20 7946 0958</a>
```

**Expected Output**

Initiates a call to the UK number.

**Why This Output Occurs**

The `+44` country code identifies the UK, and the rest of the number is the London area code and local number.

#### Real-World Cases

**Case 1: Restaurant Websites**

Restaurants use `tel:` links for reservation phone numbers.

**Case 2: Emergency Contact Pages**

Organisations use `tel:` for emergency contact numbers.

**Case 3: Mobile E-Commerce**

Checkout pages use `tel:` for customer support call links.

---

### 7. Choosing Semantic Elements Rather Than Purely Visual Elements

#### Definitions

**Core Definition**

Choosing the correct link target type means selecting the appropriate URL scheme, `target` value, and `rel` attributes based on the content and user intent, rather than on visual appearance.

**Technical Definition**

Semantic link markup requires using the correct `href` scheme (`mailto:`, `tel:`, `#fragment`, absolute URL) for the content type, the correct `target` attribute for the desired browsing context, and the correct `rel` attributes for security and SEO. The HTML specification defines these behaviours explicitly, and WCAG guidelines require that link purpose be determinable from link text.

**Beginner-Friendly Explanation**

Use the right link type for the right job: `#id` for same-page jumps, `mailto:` for email, `tel:` for phone numbers, and absolute URLs for external sites. Always add `rel="noopener"` when using `target="_blank"`. Don‘t use JavaScript to fake a phone link — use `tel:`.

#### Purposes

- To ensure correct browser behaviour
- To maintain security with `rel="noopener"` on `target="_blank"` links
- To improve accessibility with meaningful link text
- To enhance mobile user experience with `tel:` and `mailto:` links
- To comply with SEO best practices for external and sponsored links

#### Correct vs. Incorrect Patterns

| If you want to… | Use… | Not… |
|---|---|---|
| Link to another page | `<a href="page.html">` | `<a href="javascript:void(0)">` |
| Link to a section | `<a href="#id">` | `<a href="javascript:scrollTo()">` |
| Open in new tab | `target="_blank" rel="noopener"` | `target="_blank"` alone |
| Email link | `<a href="mailto:...">` | `<a href="javascript:email()">` |
| Phone link | `<a href="tel:...">` | `<a href="javascript:call()">` |
| Download file | `<a href="file.pdf" download>` | JavaScript download hacks |

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct vs. Incorrect Link Types**

```html
<!-- CORRECT: Semantic link types -->
<p><a href="mailto:info@example.com">Email</a></p>
<p><a href="tel:+15551234567">Call</a></p>
<p><a href="#section">Jump to Section</a></p>
<p><a href="https://example.com" target="_blank" rel="noopener">External</a></p>

<!-- INCORRECT: Non-semantic or unsafe patterns -->
<p><a href="javascript:void(0)" onclick="sendEmail()">Email</a></p>
<p><a href="javascript:void(0)" onclick="makeCall()">Call</a></p>
<p><a href="https://example.com" target="_blank">External (unsafe)</a></p>
```

**Expected Output**

The correct links work across all devices and browsers. The incorrect links may fail, behave unpredictably, or introduce security risks.

**Why This Output Occurs**

The `mailto:` and `tel:` schemes are handled natively by browsers and operating systems. JavaScript-based alternatives may not work on all devices and break accessibility. The `rel="noopener"` attribute prevents the security vulnerability introduced by `target="_blank"`.

#### Real-World Cases

**Case 1: Government Accessibility Compliance**

Government websites must use semantic link types and `rel="noopener"` for security.

**Case 2: Mobile-First Design**

Mobile-first websites prioritise `tel:` and `mailto:` links for contact actions.

**Case 3: SEO Best Practices**

External links use `rel="noopener"` and optionally `rel="nofollow"` or `rel="sponsored"`.

---

## References

- MDN Web Docs – `<a>`: The Anchor element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a
- WHATWG HTML Living Standard – The a element – https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-a-element
- WHATWG HTML Living Standard – Links – https://html.spec.whatwg.org/multipage/links.html
- MDN Web Docs – `target` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#target
- Chrome for Developers – Links to cross-origin destinations are unsafe – https://developer.chrome.com/docs/lighthouse/best-practices/external-anchors-use-rel-noopener
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.4: Link Purpose (In Context) – https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html
- web.dev – Click to call – https://web.dev/articles/click-to-call
- Yoast – The Full mailto Link Syntax – https://yoast.com/developer-blog/guide-mailto-links/
- MDN Web Docs – `mailto:` URL scheme – https://developer.mozilla.org/en-US/docs/Web/URI/Schemes/mailto
- MDN Web Docs – `tel:` URL scheme – https://developer.mozilla.org/en-US/docs/Web/URI/Schemes/tel
- MDN Web Docs – `download` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#download
- W3C – HTML 5: Links – https://dev.w3.org/html5/spec-author-view/links.html