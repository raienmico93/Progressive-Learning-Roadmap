# Link Types — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Link types are the categories of hyperlinks defined by the purpose of their destination and the URL scheme they use. HTML recognises navigational links (internal, external, anchor), action links (email, telephone, SMS), and download links. Each type has distinct syntax, user expectations, and security considerations.

**Technical Definition**
In the WHATWG HTML Living Standard, every `<a>` element with an `href` attribute creates a **hyperlink**. The `href` attribute accepts any URL scheme supported by browsers, including `http:`, `https:`, `mailto:`, `tel:`, and `sms:`. Link types are not formally enumerated as an attribute value; they are distinguished by the **scheme** and **target** of the `href` value, together with the `download` attribute and fragment identifiers. The `rel` attribute provides relationship metadata (e.g. `noopener`, `noreferrer`). The `target` attribute controls the browsing context in which the destination opens.

**Beginner-Friendly Explanation**
A link type is simply “what kind of link is this?” An **internal link** goes to another page on your own site. An **external link** goes to a different website. An **anchor link** jumps to a section of the same page (or another page). An **email link** opens the user’s email app. A **telephone link** starts a phone call. An **SMS link** opens the messaging app. A **download link** saves a file instead of navigating to it. Each type behaves differently, and each has its own best practices.

---

### Key Characteristics

- **Scheme-driven behaviour**: The URL scheme (`http:`, `https:`, `mailto:`, `tel:`, `sms:`) determines whether the link navigates, opens an email client, dials a number, or opens a messaging app.
- **Fragment-based navigation**: Anchor links use `#` followed by an element `id` to jump to a specific part of a document.
- **Download semantics**: The `download` attribute suggests saving the resource rather than navigating to it, but it only works reliably for same-origin URLs.
- **Security sensitivity**: External links with `target="_blank"` require `rel="noopener noreferrer"` to prevent opener-based attacks and referrer leakage.
- **Accessibility implications**: Link text must be descriptive; action links (email, phone, SMS) should indicate their behaviour so users are not surprised when an external application opens.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<a>`, `href`, and attributes.
2. **URL fundamentals**: understanding of absolute vs. relative URLs, fragments, and schemes.
3. **Browser behaviour awareness**: knowing that different schemes hand off to different applications.
4. **Accessibility awareness**: understanding WCAG 2.4.4 (Link Purpose) and the need for descriptive link text.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **SEO** | Internal and external link structures, anchor text, and `rel` attributes influence crawling and ranking. |
| **Accessibility (WCAG)** | Link purpose (2.4.4) and link text (2.4.9) apply to all link types. |
| **Security** | `noopener`, `noreferrer`, and `download` restrictions protect users. |
| **Internationalisation** | `tel:` and `sms:` URIs must use international format (`+` prefix) for cross-border compatibility. |
| **Mobile UX** | Telephone and SMS links are designed for mobile devices; desktop behaviour varies by installed applications. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Internal links** — same-site navigation
2. **External links** — cross-site navigation
3. **Anchor links** — fragment navigation
4. **Email links** (`mailto:`) — email client handoff
5. **Telephone links** (`tel:`) — phone call handoff
6. **SMS links** (`sms:`) — messaging handoff
7. **Download links** — file download
8. **Link type best practices** — quality, security, and accessibility guidelines

---

## 1. Internal Links — Same-Site Navigation

### Definitions

**Core Definition**
Internal links are hyperlinks that point to other pages, resources, or sections **within the same website**. They are the primary mechanism for site navigation and information architecture.

**Technical Definition**
An internal link uses an `href` value that resolves to a URL on the same origin as the current document. This may be a **root-relative URL** (`/about`), a **document-relative URL** (`../about` or `about.html`), or an **absolute URL** pointing to the same domain (`https://example.com/about`). Internal links are the backbone of a site’s crawlable structure and are used by search engines to discover and understand pages.

**Beginner-Friendly Explanation**
Internal links are the links that keep visitors on your site. They point from one page of your website to another — like a menu item, a “related articles” link, or a “read more” button that goes to another page on the same site.

---

### Purposes

- **To enable site navigation** between pages and sections.
- **To establish information architecture** and topical relationships between pages.
- **To help search engines crawl and index the site** by providing a link graph.
- **To guide users to related content** and keep them engaged.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="/about">About Us</a>
<a href="contact.html">Contact</a>
<a href="../services/">Services</a>
```

**Component breakdown**

| URL form | Example | Resolution |
|---|---|---|
| Root-relative | `/about` | Resolves against the site root. |
| Document-relative | `contact.html` | Resolves against the current document’s directory. |
| Parent-relative | `../services/` | Resolves one directory up. |

#### Syntax Rules

1. **Use root-relative or document-relative URLs** for portability across environments.
2. **Include a trailing slash for directory links** when appropriate to avoid redirects.
3. **Use descriptive link text** that matches the target page’s content.

#### Constraints and Limitations

- **Document-relative links break when pages are moved** to different directory depths.
- **Absolute URLs to the same domain** work but are less portable between environments.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Navigation Menu

**Setup Guide**: Save as `internal-nav.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Internal Navigation</title>
</head>
<body>
  <nav>
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About Us</a></li>
      <li><a href="/services">Services</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</body>
</html>
```

**Expected output**: A navigation menu with four root-relative links. Clicking each link navigates to the corresponding page on the same site.

**Why this output occurs**: The `href` values are root-relative URLs, which resolve against the site’s root regardless of the current page’s location.

---

#### Example 2: A “Related Articles” Section

**Setup Guide**: Save as `internal-related.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Related Articles</title>
</head>
<body>
  <article>
    <h1>Understanding Link Types</h1>
    <p>Link types determine how a hyperlink behaves.</p>
  </article>

  <aside>
    <h2>Related Articles</h2>
    <ul>
      <li><a href="/articles/url-schemes">A Guide to URL Schemes</a></li>
      <li><a href="/articles/link-security">Link Security Best Practices</a></li>
    </ul>
  </aside>
</body>
</html>
```

**Expected output**: A related-articles list with descriptive internal links.

**Why this output occurs**: The links are descriptive and point to other pages within the site, helping both users and search engines understand the relationship between topics.

---

### Real-World Cases

1. **Site menus and breadcrumbs**: Primary navigation links.
2. **Cross-linking documentation**: Linking between related reference pages.
3. **Blog “read next” sections**: Guiding readers to related content.

---

### References

- MDN Web Docs — `<a>`: The Anchor element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a
- Google Search Central — Link best practices — https://developers.google.com/search/docs/crawling-indexing/links-crawlable

---

## 2. External Links — Cross-Site Navigation

### Definitions

**Core Definition**
External links are hyperlinks that point to resources on a **different website or origin**. They connect your content to the wider web.

**Technical Definition**
An external link uses an `href` value whose origin differs from the current document’s origin — typically an absolute `https://` URL to another domain. External links are often paired with `target="_blank"` to open in a new tab, and with `rel="noopener noreferrer"` to prevent opener-based attacks and referrer leakage. `rel="nofollow"` may be added for untrusted or user-generated external links.

**Beginner-Friendly Explanation**
External links take visitors away from your site to another website. They are useful for citing sources, linking to tools, or recommending resources — but they require security precautions.

---

### Purposes

- **To cite sources** and provide references.
- **To link to tools, partners, or authoritative resources**.
- **To open a new tab for cross-site navigation** without losing the user’s place.
- **To support the broader web of linked content**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Visit Example (opens in a new tab)
</a>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `href` | Absolute URL to the external resource. |
| `target="_blank"` | Opens in a new tab. |
| `rel="noopener"` | Prevents the new page from accessing `window.opener`. |
| `rel="noreferrer"` | Prevents the referrer from being sent. |
| `rel="nofollow"` | Suggests search engines do not follow the link. |

#### Syntax Rules

1. **Use absolute URLs** for external links.
2. **Pair `target="_blank"` with `rel="noopener noreferrer"`** for security.
3. **Warn users when a link opens a new tab or downloads a file**.

#### Constraints and Limitations

- **`_blank` without `noopener`** allows the new page to navigate the opener tab.
- **`nofollow` is a hint**, not a guarantee; search engines may still follow the link.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Secure External Link

**Setup Guide**: Save as `external-secure.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Secure External Link</title>
</head>
<body>
  <p>
    Read the official documentation at
    <a href="https://developer.mozilla.org/"
       target="_blank"
       rel="noopener noreferrer">
      MDN Web Docs (opens in a new tab)
    </a>.
  </p>
</body>
</html>
```

**Expected output**: The link opens MDN in a new tab, and the new page cannot access `window.opener` or receive the referrer.

**Why this output occurs**: `target="_blank"` opens a new browsing context; `rel="noopener noreferrer"` prevents opener-based attacks and referrer leakage.

---

#### Example 2: An External Link with `nofollow`

**Setup Guide**: Save as `external-nofollow.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>External Nofollow</title>
</head>
<body>
  <p>
    User-submitted link:
    <a href="https://untrusted.example.com" rel="nofollow noopener noreferrer">
      Untrusted Site
    </a>
  </p>
</body>
</html>
```

**Expected output**: The link opens the untrusted site in the same tab (no `target`), and `rel="nofollow"` suggests that search engines should not follow it.

**Why this output occurs**: `rel="nofollow"` is a hint to search engines for user-generated or untrusted links; `noopener noreferrer` would be added if `target="_blank"` were also used.

---

### Real-World Cases

1. **Citations and references**: Linking to authoritative sources.
2. **Partner and sponsor links**: With appropriate `rel` values.
3. **User-generated content**: With `rel="nofollow"` to discourage spam.

---

### References

- web.dev — Links to cross-origin destinations are unsafe — https://web.dev/articles/unsafe-cross-origin-destinations
- Google Search Central — Qualify outbound links — https://developers.google.com/search/docs/crawling-indexing/qualify-outbound-links

---

## 3. Anchor Links — Fragment Navigation

### Definitions

**Core Definition**
Anchor links (also called fragment links or jump links) navigate to a **specific element within a document** identified by its `id` attribute. They use the `#` character followed by the element’s ID.

**Technical Definition**
An anchor link’s `href` value contains a fragment identifier (`#id`). When activated, the browser scrolls the matching element into view and updates the URL’s fragment. Anchor links can target elements on the same page or, with a full URL, on another page (`page.html#section`). The target element must have an `id` attribute that exactly matches the fragment.

**Beginner-Friendly Explanation**
Anchor links jump to a specific section of a page — like a table of contents that takes you directly to a chapter. They are useful for long pages and for linking to a specific section of a documentation page.

---

### Purposes

- **To navigate within a long page** without scrolling manually.
- **To create tables of contents and “skip to content” links**.
- **To link to a specific section of another page**.
- **To support deep linking** to a particular part of a document.

---

### Syntax Rules and Structure

#### Complete General Syntax (Same Page)

```html
<a href="#section-two">Jump to Section Two</a>
...
<h2 id="section-two">Section Two</h2>
```

#### Complete General Syntax (Cross-Page)

```html
<a href="/docs/page.html#section-two">Read Section Two</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| `#` | Fragment identifier prefix. |
| `section-two` | The target element’s `id`. |

#### Syntax Rules

1. **The fragment must match an existing `id`** in the target document.
2. **IDs must be unique within a document**.
3. **For cross-page anchors, use the full URL followed by `#id`**.

#### Constraints and Limitations

- **Broken fragments do nothing** (the browser stays at the top).
- **IDs are case-sensitive** in HTML5.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Table of Contents

**Setup Guide**: Save as `anchor-toc.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Table of Contents</title>
</head>
<body>
  <nav>
    <h2>Contents</h2>
    <ul>
      <li><a href="#introduction">Introduction</a></li>
      <li><a href="#link-types">Link Types</a></li>
      <li><a href="#best-practices">Best Practices</a></li>
    </ul>
  </nav>

  <section id="introduction">
    <h2>Introduction</h2>
    <p>…</p>
  </section>

  <section id="link-types">
    <h2>Link Types</h2>
    <p>…</p>
  </section>

  <section id="best-practices">
    <h2>Best Practices</h2>
    <p>…</p>
  </section>
</body>
</html>
```

**Expected output**: Clicking a table-of-contents link scrolls to the corresponding section and updates the URL fragment.

**Why this output occurs**: Each `href="#…"` matches an `id` on a section element, and the browser scrolls to that element.

---

#### Example 2: A Skip Link

**Setup Guide**: Save as `anchor-skip.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Skip Link</title>
</head>
<body>
  <a href="#main-content">Skip to main content</a>

  <header>
    <h1>Site Title</h1>
    <nav>…</nav>
  </header>

  <main id="main-content">
    <h2>Main Content</h2>
    <p>…</p>
  </main>
</body>
</html>
```

**Expected output**: The skip link moves focus to the `<main>` element, bypassing the header and navigation.

**Why this output occurs**: The `href="#main-content"` targets the `id="main-content"` on `<main>`, and the browser scrolls to and focuses that element.

---

### Real-World Cases

1. **Documentation tables of contents**: Navigating long reference pages.
2. **Skip links**: Bypassing repeated navigation for keyboard users.
3. **Cross-page deep links**: Linking to a specific section of a documentation page.

---

### References

- MDN Web Docs — `<a>`: Linking to an element on the same page — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#linking_to_an_element_on_the_same_page
- W3C — HTML5: Fragment identifiers — https://www.w3.org/TR/html5/browsers.html#scroll-to-fragid

---

## 4. Email Links — `mailto:` Scheme

### Definitions

**Core Definition**
An email link uses the `mailto:` URL scheme to open the user’s default email client with a new message pre-addressed to the specified email address.

**Technical Definition**
The `mailto:` URI scheme is defined in RFC 6068. It identifies an Internet mailbox and may include optional headers such as `subject`, `body`, `cc`, and `bcc`. The `href` value takes the form `mailto:address@example.com` or, with parameters, `mailto:address@example.com?subject=Hello&body=World`. Browsers hand off `mailto:` URLs to the operating system’s registered email handler.

**Beginner-Friendly Explanation**
An email link opens the user’s email app with a new message ready to send. Instead of copying an email address, the user clicks the link and starts writing.

---

### Purposes

- **To make it easy to contact the site owner** without copying an email address.
- **To pre-fill subject and body** for structured enquiries.
- **To support multiple recipients** via `cc` or `bcc`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="mailto:user@example.com">Email us</a>
```

#### Complete General Syntax (with Subject and Body)

```html
<a href="mailto:user@example.com?subject=Enquiry&body=Hello%20team">Email us</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| `mailto:` | The scheme. |
| `user@example.com` | The recipient address. |
| `?subject=…` | Optional subject header. |
| `&body=…` | Optional body header (URL-encoded). |
| `&cc=…` | Optional CC recipient. |
| `&bcc=…` | Optional BCC recipient. |

#### Syntax Rules

1. **Use the `mailto:` scheme followed by a valid email address**.
2. **URL-encode special characters** in subject and body (e.g. spaces as `%20`).
3. **Separate parameters with `&`** and the first parameter with `?`.

#### Constraints and Limitations

- **User must have an email client configured**; otherwise, the link may do nothing.
- **Mobile devices may prompt to choose an email app**.
- **Some users prefer not to have their email client opened** unexpectedly.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Email Link

**Setup Guide**: Save as `mailto-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Email Link</title>
</head>
<body>
  <p>
    Contact us at
    <a href="mailto:hello@example.com">hello@example.com</a>.
  </p>
</body>
</html>
```

**Expected output**: Clicking the link opens the user’s default email client with a new message addressed to `hello@example.com`.

**Why this output occurs**: The `mailto:` scheme instructs the browser to hand off the URL to the operating system’s email handler.

---

#### Example 2: Email Link with Subject and Body

**Setup Guide**: Save as `mailto-params.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Email with Subject</title>
</head>
<body>
  <p>
    <a href="mailto:support@example.com?subject=Support%20Request&body=Please%20describe%20your%20issue.">
      Email Support
    </a>
  </p>
</body>
</html>
```

**Expected output**: The email client opens with the subject “Support Request” and the body pre-filled with “Please describe your issue.”

**Why this output occurs**: The `subject` and `body` query parameters are parsed by the email client and used to populate the new message.

---

### Real-World Cases

1. **Contact pages**: “Email us” links.
2. **Support requests**: Pre-filled subject and body for structured enquiries.
3. **Newsletter sign-ups**: Email links with a specific subject line.

---

### References

- MDN Web Docs — Email links — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Creating_links#email_links
- RFC 6068 — The 'mailto' URI Scheme — https://datatracker.ietf.org/doc/html/rfc6068

---

## 5. Telephone Links — `tel:` Scheme

### Definitions

**Core Definition**
A telephone link uses the `tel:` URL scheme to initiate a phone call to the specified number when activated on a device that supports telephony.

**Technical Definition**
The `tel:` URI scheme is defined in RFC 3966. It describes resources identified by telephone numbers, which may be global (starting with `+` and the country code) or local. The `href` value takes the form `tel:+15551234567`. On cellular devices, the number is dialled; on desktop systems, a registered calling application (e.g. Skype, FaceTime) may handle the URI.

**Beginner-Friendly Explanation**
A telephone link lets users tap a phone number on a mobile device to start a call. On a computer, it may open a calling app.

---

### Purposes

- **To enable one-tap calling** on mobile devices.
- **To reduce friction** for customer support and sales contact.
- **To provide a clickable phone number** in contact information.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="tel:+15551234567">+1 (555) 123-4567</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| `tel:` | The scheme. |
| `+` | International dialling prefix. |
| `15551234567` | Country code and number, without spaces or punctuation. |

#### Syntax Rules

1. **Use the international format** (`+` followed by country code and number) for cross-border compatibility.
2. **Omit spaces, hyphens, and parentheses** from the `href` value; the visible text may include formatting.
3. **Use the `tel:` scheme**; do not use `callto:` or other non-standard schemes.

#### Constraints and Limitations

- **Behaviour depends on the device**: cellular devices dial; desktops may open a calling app or do nothing.
- **Not all desktop environments have a registered `tel:` handler**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Telephone Link

**Setup Guide**: Save as `tel-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Telephone Link</title>
</head>
<body>
  <p>
    Call us:
    <a href="tel:+15551234567">+1 (555) 123-4567</a>
  </p>
</body>
</html>
```

**Expected output**: On a mobile device, tapping the link initiates a call to +1 (555) 123-4567. On a desktop, a registered calling application may open.

**Why this output occurs**: The `tel:` scheme hands the number to the device’s telephony or calling application.

---

#### Example 2: Telephone Link with International Format

**Setup Guide**: Save as `tel-international.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>International Telephone Link</title>
</head>
<body>
  <p>
    International: <a href="tel:+442071234567">+44 20 7123 4567</a>
  </p>
</body>
</html>
```

**Expected output**: The link initiates a call to the UK number +44 20 7123 4567.

**Why this output occurs**: The `href` uses the international format with `+44` (UK country code), ensuring the call routes correctly from any country.

---

### Real-World Cases

1. **Contact pages**: Click-to-call phone numbers.
2. **Customer support**: One-tap calling on mobile.
3. **Local business listings**: Phone numbers in structured data.

---

### References

- MDN Web Docs — `<a>`: Linking to telephone numbers — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#linking_to_telephone_numbers
- RFC 3966 — The tel URI for Telephone Numbers — https://datatracker.ietf.org/doc/html/rfc3966

---

## 6. SMS Links — `sms:` Scheme

### Definitions

**Core Definition**
An SMS link uses the `sms:` URL scheme to open the device’s messaging application with a new text message pre-addressed to the specified phone number or numbers.

**Technical Definition**
The `sms:` URI scheme is defined in RFC 5724. It specifies one or more recipients for an SMS message. The `href` value takes the form `sms:+15551234567` or, for multiple recipients, `sms:+15551234567,+15559876543`. Some platforms support an optional `?body=` parameter to pre-fill the message body, though support varies.

**Beginner-Friendly Explanation**
An SMS link opens the user’s texting app with a new message ready to send to the specified number. It is useful for mobile contact options.

---

### Purposes

- **To enable one-tap texting** on mobile devices.
- **To pre-address a message** to a support or sales number.
- **To support multiple recipients** for group messaging.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="sms:+15551234567">Text us</a>
```

#### Complete General Syntax (with Body)

```html
<a href="sms:+15551234567?body=Hello">Text us</a>
```

**Component breakdown**

| Component | Description |
|---|---|
| `sms:` | The scheme. |
| `+15551234567` | Recipient number in international format. |
| `?body=…` | Optional pre-filled message body (platform-dependent). |

#### Syntax Rules

1. **Use the `sms:` scheme** followed by the recipient number.
2. **Use international format** (`+` and country code) for reliability.
3. **Separate multiple recipients with commas**.
4. **The `body` parameter support varies**; do not rely on it universally.

#### Constraints and Limitations

- **Desktop support is limited**; SMS links are primarily for mobile devices.
- **The `body` parameter is not universally supported**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple SMS Link

**Setup Guide**: Save as `sms-basic.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SMS Link</title>
</head>
<body>
  <p>
    Text us:
    <a href="sms:+15551234567">+1 (555) 123-4567</a>
  </p>
</body>
</html>
```

**Expected output**: On a mobile device, tapping the link opens the messaging app with a new message to +1 (555) 123-4567.

**Why this output occurs**: The `sms:` scheme hands the recipient number to the device’s messaging application.

---

#### Example 2: SMS Link with Pre-filled Body

**Setup Guide**: Save as `sms-body.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SMS with Body</title>
</head>
<body>
  <p>
    <a href="sms:+15551234567?body=Hi%2C%20I%20need%20help">
      Text Support
    </a>
  </p>
</body>
</html>
```

**Expected output**: The messaging app opens with the body pre-filled with “Hi, I need help” (on platforms that support the `body` parameter).

**Why this output occurs**: The `body` query parameter is parsed by the messaging app; support is platform-dependent.

---

### Real-World Cases

1. **Customer support**: “Text us” links on mobile sites.
2. **Appointment reminders**: Pre-filled SMS links.
3. **Two-factor authentication**: SMS-based verification links.

---

### References

- RFC 5724 — URI Scheme for GSM SMS — https://datatracker.ietf.org/doc/html/rfc5724
- MDN Web Docs — `<a>`: Supported URL schemes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#href

---

## 7. Download Links — File Download

### Definitions

**Core Definition**
A download link uses the `download` attribute on an `<a>` element to suggest that the browser should save the linked resource as a file rather than navigating to it.

**Technical Definition**
The `download` attribute is a boolean (or string) attribute on `<a>`. When present, it causes the browser to treat the linked URL as a download. If a value is provided, it suggests the filename. However, `download` only works reliably for **same-origin URLs**, same-origin `blob:` URLs, and `data:` URLs. For cross-origin URLs, the attribute is ignored, and navigation occurs normally (unless the server sends `Content-Disposition: attachment`).

**Beginner-Friendly Explanation**
A download link lets users save a file — like a PDF, image, or CSV — to their computer instead of opening it in the browser. You add the `download` attribute to the link, and the browser saves the file.

---

### Purposes

- **To let users save files** (documents, images, data exports).
- **To provide a clear “download” action** distinct from navigation.
- **To suggest a filename** for the saved file.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<a href="/files/report.pdf" download="annual-report.pdf">Download Report</a>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `href` | URL of the resource to download. |
| `download` | Boolean or filename string. |
| `download="filename"` | Suggests the saved filename. |

#### Syntax Rules

1. **`download` only works for same-origin URLs**, `blob:`, and `data:` schemes.
2. **The `download` value suggests a filename**; the browser may adjust it.
3. **The server may override the filename** with `Content-Disposition`.

#### Constraints and Limitations

- **Cross-origin downloads require `Content-Disposition: attachment`** from the server.
- **Browser behaviour varies**: some browsers prompt, others save automatically.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Same-Origin Download

**Setup Guide**: Save as `download-same-origin.html` and serve from the same origin as the file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Download Link</title>
</head>
<body>
  <a href="/files/report.pdf" download="annual-report.pdf">
    Download Annual Report (PDF)
  </a>
</body>
</html>
```

**Expected output**: Clicking the link saves `report.pdf` as `annual-report.pdf` (or prompts the user to save it).

**Why this output occurs**: The `download` attribute is honoured because the file is same-origin, and the value suggests the filename.

---

#### Example 2: A Cross-Origin Download (Server-Side Header Required)

**Setup Guide**: Save as `download-cross-origin.html`. The server must send `Content-Disposition: attachment`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Cross-Origin Download</title>
</head>
<body>
  <a href="https://cdn.example.com/files/report.pdf" download>
    Download Report (PDF)
  </a>
</body>
</html>
```

**Expected output**: Without the `Content-Disposition` header, the browser navigates to the PDF instead of downloading it. With the header, the file is downloaded.

**Why this output occurs**: For cross-origin URLs, the `download` attribute is ignored; only the server’s `Content-Disposition` header can force a download.

---

### Real-World Cases

1. **Document downloads**: PDFs, spreadsheets, and reports.
2. **Data exports**: CSV or JSON exports from web apps.
3. **Media downloads**: Images and audio files.

---

### References

- MDN Web Docs — `<a>`: download — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#download
- WHATWG — Downloading resources — https://html.spec.whatwg.org/multipage/links.html#downloading-resources

---

## 8. Link Type Best Practices

### Definitions

**Core Definition**
Link type best practices are guidelines for choosing the correct link type, writing descriptive link text, securing external links, and ensuring that links behave predictably for all users.

**Technical Definition**
Best practices are informed by WCAG 2.4.4 (Link Purpose), Google’s link best-practices documentation, and security guidance from web.dev. They encompass descriptive anchor text, correct URL schemes, safe `target` handling, `rel` attribute usage, and explicit warnings for non-standard link behaviour (new tabs, downloads, external apps).

**Beginner-Friendly Explanation**
Best practices help you create links that are safe, clear, and usable by everyone — including screen-reader users and people who are surprised when a link opens a new app or downloads a file.

---

### Purposes

- **To ensure accessibility** through descriptive link text and predictable behaviour.
- **To protect security** with `noopener` and `noreferrer` on external links.
- **To improve SEO** through meaningful anchor text and correct `rel` values.
- **To maintain user trust** by warning about new tabs, downloads, and external apps.

---

### Syntax Rules and Structure

| Link Type | Best Practice |
|---|---|
| **Internal** | Use root-relative URLs; descriptive anchor text; cross-link related content. |
| **External** | Use `target="_blank"` with `rel="noopener noreferrer"`; warn users about new tabs. |
| **Anchor** | Ensure the fragment matches an existing `id`; use for long pages and skip links. |
| **Email** (`mailto:`) | Use a valid address; URL-encode subject and body parameters. |
| **Telephone** (`tel:`) | Use international format (`+` and country code). |
| **SMS** (`sms:`) | Use international format; be aware that `body` support varies. |
| **Download** | Use `download` for same-origin files; rely on `Content-Disposition` for cross-origin files. |

#### Syntax Rules

1. **Descriptive link text**: Avoid “click here” and “read more”.
2. **Security**: Always pair `target="_blank"` with `rel="noopener noreferrer"`.
3. **Predictability**: Warn users when a link opens a new tab, downloads a file, or opens an external app.
4. **Correct scheme**: Use `mailto:`, `tel:`, or `sms:` only for their intended actions.

#### Constraints and Limitations

- **`nofollow` is a hint**, not a directive.
- **`download` is subject to same-origin restrictions**.
- **`body` on `sms:` has limited support**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Well-Formed Set of Links

**Setup Guide**: Save as `best-practice-links.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Links</title>
</head>
<body>
  <!-- Internal link: descriptive, root-relative -->
  <p>Read our <a href="/shipping-policy">shipping policy</a>.</p>

  <!-- External link: secure, warns about new tab -->
  <p>
    Visit
    <a href="https://example.com"
       target="_blank"
       rel="noopener noreferrer">
      Example (opens in a new tab)
    </a>.
  </p>

  <!-- Email link: international, descriptive -->
  <p>
    Email <a href="mailto:support@example.com">support@example.com</a>.
  </p>

  <!-- Telephone link: international format -->
  <p>
    Call <a href="tel:+15551234567">+1 (555) 123-4567</a>.
  </p>
</body>
</html>
```

**Expected output**: All links are descriptive, secure, and behave predictably.

**Why this output occurs**: Each link follows the corresponding best practice: internal links are descriptive and root-relative; the external link is secure and warns about the new tab; the email and phone links use the correct schemes and international formats.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG compliance requires descriptive link text and secure external links.
2. **Documentation sites**: Consistent link types and warnings for downloads and external tabs.
3. **E-commerce**: Secure external payment links and clear download links for invoices.

---

### References

- W3C — Understanding SC 2.4.4: Link Purpose (In Context) — https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html
- Google Search Central — Link best practices — https://developers.google.com/search/docs/crawling-indexing/links-crawlable
- web.dev — Links to cross-origin destinations are unsafe — https://web.dev/articles/unsafe-cross-origin-destinations
- MDN Web Docs — `<a>`: Accessibility — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a#accessibility

---

## Summary Table of Link Types

| Link Type | Scheme / Mechanism | Primary Purpose | Key Security / Best Practice |
|---|---|---|---|
| **Internal** | `https://` (same origin) or relative URL | Same-site navigation | Descriptive text; root-relative URLs. |
| **External** | `https://` (different origin) | Cross-site navigation | `target="_blank"` + `rel="noopener noreferrer"`; warn users. |
| **Anchor** | `#fragment` | In-page or cross-page section navigation | Fragment must match an existing `id`. |
| **Email** | `mailto:` | Open email client | Valid address; URL-encode subject/body. |
| **Telephone** | `tel:` | Initiate phone call | International format (`+` and country code). |
| **SMS** | `sms:` | Open messaging app | International format; `body` support varies. |
| **Download** | `download` attribute | Save file instead of navigating | Same-origin only; cross-origin needs `Content-Disposition`. |

---

## Final Notes

- **Link types are defined by scheme and purpose**: Internal and external links use `https:`; action links use `mailto:`, `tel:`, or `sms:`; downloads use the `download` attribute.
- **Security is non-negotiable for external links**: Always use `rel="noopener noreferrer"` with `target="_blank"`.
- **Accessibility depends on descriptive text**: Every link type must have meaningful link text that describes its purpose.
- **Test on real devices**: Telephone and SMS links behave differently on desktop and mobile; verify the experience on both.
- **Validate and audit**: Use link checkers and accessibility tools to catch broken fragments, missing `rel` attributes, and vague anchor text.

---

This cheat sheet has covered link types — internal, external, anchor, email (`mailto:`), telephone (`tel:`), SMS (`sms:`), and download — plus best practices, with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate link markup against the WHATWG HTML Living Standard, WCAG 2.4.4, and web security guidance.