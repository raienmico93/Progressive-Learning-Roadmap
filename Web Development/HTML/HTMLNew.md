# HTML Comprehensive, Structured, and Progressive Learning Roadmap

## From Document Structure Foundations to Advanced Semantics, Accessibility, Forms, Media, and Production Web Engineering

HTML is best learned as more than "tags for text." The progression should cover **document structure → elements → attributes → semantics → text content → links → images → lists → tables → forms → media → metadata → accessibility → SEO → performance → modern HTML APIs → production web engineering**.

---

# I. HTML Foundations

- **1. What HTML Is**
  - HTML
  - HyperText Markup Language
  - HTML history
  - Tim Berners-Lee
  - HTML 1.0
  - HTML 2.0
  - HTML 3.2
  - HTML 4.01
  - XHTML
  - HTML5
  - HTML Living Standard
  - WHATWG
  - W3C
  - HTML philosophy
    - Structure
    - Semantics
    - Accessibility
    - Interoperability
    - Extensibility
  - HTML vs XML
  - HTML vs XHTML
  - HTML vs CSS
  - HTML vs JavaScript
  - HTML use cases
    - Web pages
    - Web applications
    - Email templates
    - Documentation
    - E-books
    - Progressive web apps
  - HTML in modern web development
  - HTML ecosystem
  - HTML specifications
  - HTML best practices

- **2. How HTML Works**
  - HTML documents
  - Markup
  - Elements
  - Tags
  - Attributes
  - Content
  - Nesting
  - DOM
  - Document Object Model
  - Parsing
  - Rendering
  - Rendering pipeline
  - HTML and CSS
  - HTML and JavaScript
  - HTML and accessibility
  - HTML best practices

- **3. HTML Document Structure**
  - `<!DOCTYPE html>`
  - `<html>`
  - `<head>`
  - `<body>`
  - Document structure
  - Document metadata
  - Document language
    - `lang` attribute
  - Document direction
    - `dir` attribute
  - Document encoding
    - `<meta charset="UTF-8">`
  - Document viewport
    - `<meta name="viewport" ...>`
  - Document title
    - `<title>`
  - Document structure best practices

- **4. HTML Syntax**
  - Elements
  - Tags
    - Opening tags
    - Closing tags
    - Self-closing tags
  - Attributes
    - Attribute names
    - Attribute values
    - Boolean attributes
  - Content
    - Text content
    - Element content
    - Mixed content
  - Comments
    - `<!-- -->`
  - Whitespace
  - Case sensitivity
  - Quoting
  - Escaping
  - Character references
    - Named references
    - Numeric references
    - Hexadecimal references
  - HTML syntax best practices

- **5. HTML Versions and Standards**
  - HTML5
  - HTML Living Standard
  - WHATWG
  - W3C
  - HTML validation
  - W3C Validator
  - Nu HTML Checker
  - Validation best practices

- **6. Setting Up HTML Development**
  - Code editors
    - VS Code
    - Sublime Text
    - Atom
    - WebStorm
    - Vim
    - Neovim
    - Emacs
  - Editor extensions
    - Emmet
    - Live Server
    - Prettier
    - ESLint
    - HTMLHint
  - Browsers
    - Chrome
    - Firefox
    - Safari
    - Edge
  - Developer tools
    - Chrome DevTools
    - Firefox Developer Tools
    - Safari Web Inspector
  - Local development
    - Live Server
    - Python HTTP server
    - Node.js HTTP server
    - BrowserSync
  - Version control
    - Git
    - GitHub
  - Project structure
  - Development best practices

---

# II. Elements

- [**7. Element Fundamentals**](/Web%20Development/HTML/Elements/Element.md)
  - Elements
  - Element types
    - Block-level elements
    - Inline elements
    - Inline-block elements
  - Element categories
    - Metadata content
    - Flow content
    - Sectioning content
    - Heading content
    - Phrasing content
    - Embedded content
    - Interactive content
    - Palpable content
    - Script-supporting content
  - Element nesting
  - Element content model
  - Element best practices

- [**8. Document Metadata Elements**](/Web%20Development/HTML/Elements/DocumentMetadata.md)
  - `<html>`
  - `<head>`
  - `<title>`
  - `<base>`
  - `<link>`
  - `<meta>`
  - `<style>`
  - `<script>`
  - `<noscript>`
  - Metadata element best practices

- [**9. Sectioning Elements**](/Web%20Development/HTML/Elements/Sectioning.md)
  - `<body>`
  - `<article>`
  - `<section>`
  - `<nav>`
  - `<aside>`
  - `<h1>` to `<h6>`
  - `<hgroup>`
  - `<header>`
  - `<footer>`
  - `<main>`
  - `<address>`
  - Sectioning element best practices

- [**10. Grouping Content Elements**](/Web%20Development/HTML/Elements/GroupingContent.md)
  - `<div>`
  - `<p>`
  - `<hr>`
  - `<pre>`
  - `<blockquote>`
  - `<ol>`
  - `<ul>`
  - `<menu>`
  - `<li>`
  - `<dl>`
  - `<dt>`
  - `<dd>`
  - `<figure>`
  - `<figcaption>`
  - Grouping content best practices

- [**11. Text-Level Semantics**](/Web%20Development/HTML/Elements/TextLevel.md)
  - `<a>`
  - `<em>`
  - `<strong>`
  - `<small>`
  - `<s>`
  - `<cite>`
  - `<q>`
  - `<dfn>`
  - `<abbr>`
  - `<ruby>`
  - `<rt>`
  - `<rp>`
  - `<data>`
  - `<time>`
  - `<code>`
  - `<var>`
  - `<samp>`
  - `<kbd>`
  - `<sub>`
  - `<sup>`
  - `<i>`
  - `<b>`
  - `<u>`
  - `<mark>`
  - `<bdi>`
  - `<bdo>`
  - `<span>`
  - `<br>`
  - `<wbr>`
  - Text-level semantic best practices

- [**12. Embedded Content Elements**](/Web%20Development/HTML/Elements/Embedded.md)
  - `<img>`
  - `<picture>`
  - `<source>`
  - `<iframe>`
  - `<embed>`
  - `<object>`
  - `<param>`
  - `<video>`
  - `<audio>`
  - `<track>`
  - `<map>`
  - `<area>`
  - `<canvas>`
  - `<svg>`
  - `<math>`
  - Embedded content best practices

- [**13. Interactive Elements**](/Web%20Development/HTML/Elements/Interactive.md)
  - `<a>`
  - `<button>`
  - `<details>`
  - `<summary>`
  - `<dialog>`
  - `<form>`
  - `<input>`
  - `<label>`
  - `<select>`
  - `<option>`
  - `<optgroup>`
  - `<textarea>`
  - `<output>`
  - `<progress>`
  - `<meter>`
  - `<fieldset>`
  - `<legend>`
  - `<datalist>`
  - Interactive element best practices

- [**14. Scripting Elements**](/Web%20Development/HTML/Elements/Scripting.md)
  - `<script>`
  - `<noscript>`
  - `<template>`
  - `<slot>`
  - `<canvas>`
  - Scripting element best practices

- [**15. Deprecated Elements**](/Web%20Development/HTML/Elements/Deprecated.md)
  - `<applet>`
  - `<acronym>`
  - `<bgsound>`
  - `<big>`
  - `<blink>`
  - `<center>`
  - `<dir>`
  - `<font>`
  - `<frame>`
  - `<frameset>`
  - `<isindex>`
  - `<keygen>`
  - `<marquee>`
  - `<menuitem>`
  - `<nobr>`
  - `<noembed>`
  - `<noframes>`
  - `<plaintext>`
  - `<rb>`
  - `<rtc>`
  - `<spacer>`
  - `<strike>`
  - `<tt>`
  - `<xmp>`
  - Deprecated element alternatives
  - Deprecated element best practices

- [**16. Custom Elements**](/Web%20Development/HTML/Elements/Custom.md)
  - Custom elements
  - `<custom-element>`
  - Web Components
  - Custom element naming
  - Custom element lifecycle
  - Custom element best practices

---

# III. Attributes

- [**17. Attribute Fundamentals**](/Web%20Development/HTML/Attributes/Attribute.md)
  - Attributes
  - Global attributes
  - Element-specific attributes
  - Boolean attributes
  - Enumerated attributes
  - Attribute values
  - Attribute quoting
  - Attribute best practices

- [**18. Global Attributes**](/Web%20Development/HTML/Attributes/Global.md)
  - `accesskey`
  - `autocapitalize`
  - `autocorrect`
  - `autofocus`
  - `class`
  - `contenteditable`
  - `dir`
  - `draggable`
  - `enterkeyhint`
  - `exportparts`
  - `hidden`
  - `id`
  - `inert`
  - `inputmode`
  - `is`
  - `itemid`
  - `itemprop`
  - `itemref`
  - `itemscope`
  - `itemtype`
  - `lang`
  - `nonce`
  - `part`
  - `popover`
  - `role`
  - `slot`
  - `spellcheck`
  - `style`
  - `tabindex`
  - `title`
  - `translate`
  - `virtualkeyboardpolicy`
  - Global attribute best practices

- [**19. Data Attributes**](/Web%20Development/HTML/Attributes/Data.md)
  - `data-*`
  - Data attribute naming
  - Data attribute access
  - Data attribute usage
  - Data attribute best practices

- [**20. ARIA Attributes**](/Web%20Development/HTML/Attributes/ARIA.md)
  - ARIA
  - `role`
  - `aria-label`
  - `aria-labelledby`
  - `aria-describedby`
  - `aria-hidden`
  - `aria-live`
  - `aria-atomic`
  - `aria-busy`
  - `aria-checked`
  - `aria-disabled`
  - `aria-expanded`
  - `aria-haspopup`
  - `aria-pressed`
  - `aria-selected`
  - `aria-current`
  - `aria-controls`
  - `aria-owns`
  - `aria-activedescendant`
  - `aria-autocomplete`
  - `aria-multiline`
  - `aria-multiselectable`
  - `aria-orientation`
  - `aria-placeholder`
  - `aria-readonly`
  - `aria-required`
  - `aria-sort`
  - `aria-valuemax`
  - `aria-valuemin`
  - `aria-valuenow`
  - `aria-valuetext`
  - ARIA attribute best practices
  - ARIA pitfalls
  - First rule of ARIA

- [**21. Microdata**](/Web%20Development/HTML/Attributes/Microdata.md)
  - Microdata
  - `itemscope`
  - `itemtype`
  - `itemprop`
  - `itemid`
  - `itemref`
  - Microdata best practices

- [**22. RDFa**](/Web%20Development/HTML/Attributes/RDFa.md)
  - RDFa
  - `vocab`
  - `typeof`
  - `property`
  - `resource`
  - `prefix`
  - RDFa best practices

- [**23. Custom Attributes**](/Web%20Development/HTML/Attributes/Custom.md)
  - Custom attributes
  - Attribute naming
  - Attribute usage
  - Custom attribute best practices

---

# IV. Text Content

- [**24. Headings**](/Web%20Development/HTML/Text%20Content/Headings.md)
  - `<h1>` to `<h6>`
  - Heading hierarchy
  - Heading best practices
  - Heading accessibility
  - Heading SEO
  - Heading pitfalls

- [**25. Paragraphs**](/Web%20Development/HTML/Text%20Content/Paragraphs.md)
  - `<p>`
  - Paragraph best practices
  - Paragraph spacing
  - Paragraph content

- [**26. Text Formatting**](/Web%20Development/HTML/Text%20Content/TextFormatting.md)
  - `<strong>`
  - `<em>`
  - `<b>`
  - `<i>`
  - `<u>`
  - `<mark>`
  - `<small>`
  - `<del>`
  - `<ins>`
  - `<s>`
  - `<sub>`
  - `<sup>`
  - Choosing semantic elements rather than purely visual elements
  - Text formatting best practices
  - Semantic vs presentational

- [**27. Quotations and References**](/Web%20Development/HTML/Text%20Content/QuotationsAndReferences.md)
  - `<blockquote>`
  - `<q>`
  - `<cite>`
  - `<abbr>`
  - `<dfn>`
  - `<address>`
  - `<time>`
  - Quotation best practices

- [**28. Code and Preformatted Text**](/Web%20Development/HTML/Text%20Content/CodeAndTechnicalText.md)
  - `<code>`
  - `<pre>`
  - `<kbd>`
  - `<samp>`
  - `<var>`
  - Formatting source code
  - Representing user input
  - Representing program output
  - Code best practices
  - Preformatted text best practices

- [**29. Line Breaks and Rules**](/Web%20Development/HTML/Text%20Content/LineBreaksAndRules.md)
  - `<br>`
  - `<hr>`
  - `<wbr>`
  - Line break best practices

- [**30. Time and Data**](/Web%20Development/HTML/Text%20Content/TimeAndData.md)
  - `<time>`
  - `<data>`
  - `datetime` attribute
  - `value` attribute
  - Time and data best practices

- [**31. Bidirectional Text**](/Web%20Development/HTML/Text%20Content/Bidirectional.md)
  - `<bdi>`
  - `<bdo>`
  - `dir` attribute
  - Bidirectional text best practices

- [**32. Ruby Annotations**](/Web%20Development/HTML/Text%20Content/RubyAnnotations.md)
  - `<ruby>`
  - `<rt>`
  - `<rp>`
  - Ruby annotation best practices

---

# V. Links

- [**33. Link Fundamentals**](/Web%20Development/HTML/Links/Link.md)
  - `<a>`
  - `href` attribute
  - Link text
  - Link targets
  - Link best practices

- [**34. Link Types**](/Web%20Development/HTML/Links/LinkTypes.md)
  - Internal links
  - External links
  - Anchor links
  - Email links
    - `mailto:`
  - Telephone links
    - `tel:`
  - SMS links
    - `sms:`
  - Download links
  - Link type best practices

- [**35. Link Attributes**](/Web%20Development/HTML/Links/LinkAttr.md)
  - `href`
  - `target`
    - `_self`
    - `_blank`
    - `_parent`
    - `_top`
  - `rel`
    - `noopener`
    - `noreferrer`
    - `nofollow`
    - `external`
    - `help`
    - `license`
    - `next`
    - `prev`
    - `search`
    - `tag`
  - `download`
  - `hreflang`
  - `type`
  - `ping`
  - `referrerpolicy`
  - Link attribute best practices

- [**36. Link Accessibility**](/Web%20Development/HTML/Links/LinkAccess.md)
  - Link text
  - Descriptive links
  - Skip links
  - Focus states
  - Link accessibility best practices

- [**37. Link SEO**](/Web%20Development/HTML/Links/LinkSEO.md)
  - Link SEO
  - Anchor text
  - Internal linking
  - External linking
  - Nofollow
  - Link SEO best practices

---

# VI. Images and Media

- [**41. Image Fundamentals**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/Images.md)
  - Images
  - `<img>`
  - `src` attribute
  - `alt` attribute
  - Image best practices

- [**42. Image Attributes**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/ImageAttr.md)
  - `src`
  - `alt`
  - `width`
  - `height`
  - `srcset`
  - `sizes`
  - `loading`
    - `lazy`
    - `eager`
  - `decoding`
    - `sync`
    - `async`
    - `auto`
  - `fetchpriority`
  - `ismap`
  - `usemap`
  - `crossorigin`
  - `referrerpolicy`
  - Image attribute best practices

- [**43. Responsive Images**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/ResponsiveImg.md)
  - Responsive images
  - `srcset`
  - `sizes`
  - `<picture>`
  - `<source>`
  - Art direction
  - Resolution switching
  - Format switching
  - Responsive image best practices

- [**44. Image Formats**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/ImageFormat.md)
  - JPEG
  - PNG
  - GIF
  - WebP
  - AVIF
  - SVG
  - HEIC
  - Format selection
  - Format best practices

- [**45. Image Accessibility**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/ImageAccess.md)
  - Alt text
  - Decorative images
  - Functional images
  - Complex images
  - Image accessibility best practices

- [**46. Image SEO**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/ImageSEO.md)
  - Image SEO
  - File names
  - Alt text
  - Image sitemaps
  - Image SEO best practices

- [**Figure Content**](/Web%20Development/HTML/Images%20and%20Responsive%20Media/FigureContent.md)
  - `<figure>`
  - `<figcaption>`
  - Images with captions
  - Diagrams
  - Illustrations
  - Code examples

- **47. Video**
  - `<video>`
  - `src` attribute
  - `controls`
  - `autoplay`
  - `muted`
  - `loop`
  - `preload`
  - `poster`
  - `width`
  - `height`
  - `playsinline`
  - `<source>`
  - `<track>`
  - Video best practices

- **48. Audio**
  - `<audio>`
  - `src` attribute
  - `controls`
  - `autoplay`
  - `muted`
  - `loop`
  - `preload`
  - `<source>`
  - Audio best practices

- **49. Media Accessibility**
  - Captions
  - Subtitles
  - Transcripts
  - Audio descriptions
  - Media accessibility best practices

- **50. Canvas**
  - `<canvas>`
  - Canvas context
  - 2D context
  - WebGL context
  - Canvas best practices

- **51. SVG**
  - SVG
  - Inline SVG
  - External SVG
  - SVG elements
  - SVG attributes
  - SVG accessibility
  - SVG best practices

- **52. Iframes**
  - `<iframe>`
  - `src`
  - `srcdoc`
  - `sandbox`
  - `allow`
  - `loading`
  - `referrerpolicy`
  - `title`
  - Iframe best practices
  - Iframe security

- **53. Embed and Object**
  - `<embed>`
  - `<object>`
  - `<param>`
  - Embed best practices

---

# VII. Lists

- [**54. Unordered Lists**](/Web%20Development/HTML/Lists/UnorderedLists.md)
  - `<ul>`
  - `<li>`
  - Nested unordered lists
  - Unordered list best practices

- [**55. Ordered Lists**](/Web%20Development/HTML/Lists/OrderedLists.md)
  - `<ol>`
  - `<li>`
  - `type` attribute
  - `start` attribute
  - `reversed` attribute
  - Ordered list best practices

- [**56. Description Lists**](/Web%20Development/HTML/Lists/DescriptionLists.md)
  - `<dl>`
  - `<dt>`
  - `<dd>`
  - Terms and definitions
  - Metadata-style lists
  - Description list best practices

- [**57. Menu**](/Web%20Development/HTML/Lists/Menu.md)
  - `<menu>`
  - `<li>`
  - Menu best practices

- [**58. Nested Lists**](/Web%20Development/HTML/Lists/NestedAndComplexLists.md)
  - Nested lists
  - Multi-level structures
  - Mixed list types
  - Semantic list usage
  - Navigation lists
  - Content lists
  - Nesting best practices

- [**59. List Accessibility**](/Web%20Development/HTML/Lists/ListAccess.md)
  - List accessibility
  - Screen reader support
  - List accessibility best practices

---

# VIII. Tables

- [**60. Table Fundamentals**](/Web%20Development/HTML/Tables/Table.md)
  - Tables
  - `<table>`
  - Table best practices

- [**61. Table Structure**](/Web%20Development/HTML/Tables/TableStructure.md)
  - `<caption>`
  - `<thead>`
  - `<tbody>`
  - `<tfoot>`
  - `<tr>`
  - `<th>`
  - `<td>`
  - `<colgroup>`
  - `<col>`
  - Table structure best practices

- [**62. Table Attributes**](/Web%20Development/HTML/Tables/TableAttr.md)
  - `colspan`
  - `rowspan`
  - `headers`
  - `scope`
    - `col`
    - `row`
    - `colgroup`
    - `rowgroup`
  - `abbr`
  - Table attribute best practices

- [**63. Table Accessibility**](/Web%20Development/HTML/Tables/TableAccess.md)
  - Table accessibility
  - Table headers
  - Table captions
  - Table summaries
  - Table accessibility best practices

- [**64. Responsive Tables**](/Web%20Development/HTML/Tables/ResponsiveTable.md)
  - Responsive tables
  - Horizontal scrolling
  - Stacked tables
  - Responsive table best practices

- [**Table Relationships**](/Web%20Development/HTML/Tables/TableRelationship.md)
  - `scope`
  - Row headers
  - Column headers
  - Complex header relationships
  - `colspan`
  - `rowspan`

- [**Accessible Data Tables**](/Web%20Development/HTML/Tables/AccessibleDataTables.md)
  - Proper header association
  - Table captions
  - Avoiding tables for layout
  - Complex-table accessibility
---

# IX. Forms

- [**65. Form Fundamentals**](/Web%20Development/HTML/Forms/Form.md)
  - Forms
  - `<form>`
  - Form attributes
    - `action`
    - `method`
    - `enctype`
    - `name`
    - `target`
    - `novalidate`
    - `autocomplete`
    - `accept-charset`
    - `rel`
  - GET requests
  - POST requests
  - Form submission lifecycle
  - Form best practices

- [**66. Input Elements**](/Web%20Development/HTML/Forms/InputElements.md)
  - `<input>`
  - Input types
    - `text`
    - `password`
    - `email`
    - `url`
    - `tel`
    - `number`
    - `range`
    - `date`
    - `time`
    - `datetime-local`
    - `month`
    - `week`
    - `color`
    - `file`
    - `checkbox`
    - `radio`
    - `hidden`
    - `submit`
    - `reset`
    - `button`
    - `image`
    - `search`
  - Input attributes
    - `type`
    - `name`
    - `value`
    - `placeholder`
    - `required`
    - `disabled`
    - `readonly`
    - `autocomplete`
    - `autofocus`
    - `multiple`
    - `accept`
    - `min`
    - `max`
    - `step`
    - `pattern`
    - `minlength`
    - `maxlength`
    - `size`
    - `list`
    - `form`
    - `formaction`
    - `formenctype`
    - `formmethod`
    - `formnovalidate`
    - `formtarget`
    - `dirname`
  - Input best practices

- [**67. Labels**](/Web%20Development/HTML/Forms/Labels.md)
  - `<label>`
  - `for` attribute
  - Implicit labels
  - Explicit labels
  - Label best practices

- [**68. Select Elements**](/Web%20Development/HTML/Forms/SelectElements.md)
  - `<select>`
  - `<option>`
  - `<optgroup>`
  - Select attributes
    - `multiple`
    - `size`
    - `name`
    - `required`
    - `disabled`
  - Option attributes
    - `value`
    - `selected`
    - `disabled`
    - `label`
  - Select best practices

- [**69. Textarea**](/Web%20Development/HTML/Forms/Textarea.md)
  - `<textarea>`
  - Textarea attributes
    - `name`
    - `rows`
    - `cols`
    - `placeholder`
    - `required`
    - `disabled`
    - `readonly`
    - `maxlength`
    - `minlength`
    - `wrap`
  - Textarea best practices

- [**70. Buttons**](/Web%20Development/HTML/Forms/Buttons.md)
  - `<button>`
  - Button types
    - `submit`
    - `reset`
    - `button`
  - Button attributes
    - `type`
    - `name`
    - `value`
    - `disabled`
    - `form`
    - `formaction`
    - `formenctype`
    - `formmethod`
    - `formnovalidate`
    - `formtarget`
  - Button best practices

- **71. Form Grouping**
  - `<fieldset>`
  - `<legend>`
  - Form grouping best practices

- **72. Form Validation**
  - HTML5 validation
  - Validation attributes
    - `required`
    - `pattern`
    - `min`
    - `max`
    - `minlength`
    - `maxlength`
    - `step`
    - `type`
  - Validation API
    - `checkValidity()`
    - `reportValidity()`
    - `setCustomValidity()`
  - Validation best practices
  - Validation accessibility

- **73. Form Accessibility**
  - Form accessibility
  - Labels
  - Fieldsets
  - Legends
  - Error messages
  - ARIA attributes
  - Form accessibility best practices

- **74. Advanced Form Elements**
  - `<datalist>`
  - `<output>`
  - `<progress>`
  - `<meter>`
  - Advanced form element best practices

- **75. Form Security**
  - Form security
  - CSRF
  - Input validation
  - Form security best practices

---

# X. Metadata and SEO

- **76. Metadata Fundamentals**
  - Metadata
  - `<meta>`
  - Metadata best practices

- **77. Meta Tags**
  - `charset`
  - `name="viewport"`
  - `name="description"`
  - `name="keywords"`
  - `name="author"`
  - `name="robots"`
  - `name="theme-color"`
  - `name="color-scheme"`
  - `http-equiv`
    - `content-type`
    - `default-style`
    - `refresh`
    - `x-ua-compatible`
    - `content-security-policy`
  - `property="og:*"`
    - Open Graph
  - `name="twitter:*"`
    - Twitter Cards
  - `name="apple-mobile-web-app-*"`
  - `name="application-name"`
  - Meta tag best practices

- **78. Open Graph**
  - Open Graph
  - `og:title`
  - `og:type`
  - `og:image`
  - `og:url`
  - `og:description`
  - `og:site_name`
  - `og:locale`
  - Open Graph best practices

- **79. Twitter Cards**
  - Twitter Cards
  - `twitter:card`
  - `twitter:site`
  - `twitter:creator`
  - `twitter:title`
  - `twitter:description`
  - `twitter:image`
  - Twitter Card best practices

- **80. Structured Data**
  - Structured data
  - JSON-LD
  - Microdata
  - RDFa
  - Schema.org
  - Schema types
    - Article
    - Product
    - Person
    - Organization
    - Event
    - Recipe
    - FAQ
    - Breadcrumb
  - Structured data best practices

- **81. SEO Fundamentals**
  - SEO
  - Semantic HTML
  - Heading hierarchy
  - Meta descriptions
  - Canonical URLs
  - Sitemaps
  - Robots.txt
  - SEO best practices

- **82. Canonical URLs**
  - Canonical URLs
  - `<link rel="canonical">`
  - Canonical URL best practices

- **83. Sitemaps**
  - XML sitemaps
  - Sitemap structure
  - Sitemap best practices

- **84. Robots.txt**
  - Robots.txt
  - Robots meta tag
  - Robots.txt best practices

---

# XI. Accessibility

- **85. Accessibility Fundamentals**
  - Accessibility
  - A11y
  - WCAG
  - POUR principles
    - Perceivable
    - Operable
    - Understandable
    - Robust
  - Accessibility best practices

- **86. Semantic HTML**
  - Semantic HTML
  - Semantic elements
  - Semantic HTML benefits
  - Semantic HTML best practices

- **87. ARIA**
  - ARIA
  - ARIA roles
  - ARIA states
  - ARIA properties
  - ARIA labels
  - ARIA live regions
  - ARIA best practices
  - ARIA pitfalls
  - First rule of ARIA

- **88. Keyboard Accessibility**
  - Keyboard accessibility
  - Focus management
  - Tab order
  - `tabindex`
  - Focus indicators
  - Skip links
  - Keyboard accessibility best practices

- **89. Screen Readers**
  - Screen readers
  - NVDA
  - JAWS
  - VoiceOver
  - TalkBack
  - Screen reader testing
  - Screen reader best practices

- **90. Color and Contrast**
  - Color contrast
  - WCAG contrast requirements
  - Color blindness
  - Color and contrast best practices

- **91. Accessible Forms**
  - Accessible forms
  - Labels
  - Fieldsets
  - Legends
  - Error messages
  - Accessible form best practices

- **92. Accessible Images**
  - Accessible images
  - Alt text
  - Decorative images
  - Complex images
  - Accessible image best practices

- **93. Accessible Media**
  - Accessible media
  - Captions
  - Subtitles
  - Transcripts
  - Audio descriptions
  - Accessible media best practices

- **94. Accessibility Testing**
  - Accessibility testing
  - Automated testing
  - Manual testing
  - Lighthouse
  - axe
  - WAVE
  - Accessibility testing best practices

---

# XII. Modern HTML APIs

- **95. Web Components**
  - Web Components
  - Custom Elements
  - Shadow DOM
  - HTML Templates
  - Slots
  - Web Component best practices

- **96. Shadow DOM**
  - Shadow DOM
  - Shadow root
  - Shadow host
  - Encapsulation
  - Styling shadow DOM
  - Shadow DOM best practices

- **97. Templates**
  - `<template>`
  - `<slot>`
  - Template content
  - Template cloning
  - Template best practices

- **98. Custom Elements**
  - Custom elements
  - Custom element lifecycle
    - `connectedCallback()`
    - `disconnectedCallback()`
    - `adoptedCallback()`
    - `attributeChangedCallback()`
  - Custom element best practices

- **99. HTML Templates and Slots**
  - Templates
  - Slots
  - Named slots
  - Default slots
  - Slot fallback
  - Template and slot best practices

- **100. Popover API**
  - Popover API
  - `popover` attribute
  - `popovertarget`
  - `popovertargetaction`
  - Popover API best practices

- **101. Dialog Element**
  - `<dialog>`
  - `showModal()`
  - `show()`
  - `close()`
  - Dialog best practices

- **102. Details and Summary**
  - `<details>`
  - `<summary>`
  - Details and summary best practices

- **103. Lazy Loading**
  - Lazy loading
  - `loading="lazy"`
  - Lazy loading best practices

- **104. Responsive Images API**
  - Responsive images
  - `srcset`
  - `sizes`
  - `<picture>`
  - Responsive image API best practices

- **105. Drag and Drop API**
  - Drag and Drop API
  - `draggable`
  - Drag events
  - Drop events
  - Drag and drop best practices

- **106. File API**
  - File API
  - File input
  - File reading
  - File upload
  - File API best practices

- **107. Geolocation API**
  - Geolocation API
  - `navigator.geolocation`
  - Geolocation best practices

- **108. Notifications API**
  - Notifications API
  - Notification permissions
  - Notification best practices

- **109. Clipboard API**
  - Clipboard API
  - Clipboard events
  - Clipboard best practices

- **110. Intersection Observer**
  - Intersection Observer
  - `IntersectionObserver`
  - Intersection Observer best practices

- **111. Mutation Observer**
  - Mutation Observer
  - `MutationObserver`
  - Mutation Observer best practices

- **112. Resize Observer**
  - Resize Observer
  - `ResizeObserver`
  - Resize Observer best practices

- **113. Web Storage**
  - Web Storage
  - `localStorage`
  - `sessionStorage`
  - Web Storage best practices

- **114. IndexedDB**
  - IndexedDB
  - IndexedDB best practices

- **115. Web Workers**
  - Web Workers
  - Web Worker best practices

- **116. Service Workers**
  - Service Workers
  - Service Worker best practices

- **117. WebSockets**
  - WebSockets
  - WebSocket best practices

- **118. Server-Sent Events**
  - Server-Sent Events
  - SSE best practices

- **119. Fetch API**
  - Fetch API
  - Fetch best practices

- **120. Canvas API**
  - Canvas API
  - Canvas best practices

- **121. WebGL**
  - WebGL
  - WebGL best practices

- **122. WebAssembly**
  - WebAssembly
  - WASM best practices

---

# XIII. HTML Performance

- **123. Performance Fundamentals**
  - Performance
  - Rendering performance
  - Load performance
  - Performance metrics
  - Core Web Vitals
    - Largest Contentful Paint
    - First Input Delay
    - Cumulative Layout Shift
    - Interaction to Next Paint
  - Performance best practices

- **124. HTML Optimization**
  - Minification
  - Compression
  - Whitespace
  - Comments
  - HTML optimization best practices

- **125. Critical Rendering Path**
  - Critical rendering path
  - Render-blocking resources
  - Critical CSS
  - Async scripts
  - Defer scripts
  - Critical rendering path best practices

- **126. Resource Hints**
  - `preload`
  - `prefetch`
  - `preconnect`
  - `dns-prefetch`
  - `prerender`
  - `modulepreload`
  - Resource hint best practices

- **127. Lazy Loading**
  - Lazy loading
  - Images
  - Iframes
  - Videos
  - Lazy loading best practices

- **128. Image Optimization**
  - Image optimization
  - Responsive images
  - Modern formats
  - Compression
  - Image optimization best practices

- **129. Font Optimization**
  - Font optimization
  - Font loading
  - Font display
  - Font subsetting
  - Font optimization best practices

- **130. Script Loading**
  - Script loading
  - `async`
  - `defer`
  - Module scripts
  - Script loading best practices

- **131. HTML Validation and Parsing**
  - HTML validation
  - HTML parsing
  - Parse errors
  - Error handling
  - Validation best practices

---

# XIV. HTML Projects by Difficulty

## Beginner Projects

- **1. Personal Profile Page**
  - Document structure
  - Headings
  - Paragraphs
  - Images
  - Links

- **2. Recipe Page**
  - Semantic HTML
  - Lists
  - Images
  - Tables
  - Forms

- **3. Blog Post**
  - Semantic HTML
  - Headings
  - Paragraphs
  - Images
  - Links

- **4. Contact Form**
  - Forms
  - Inputs
  - Labels
  - Validation
  - Accessibility

- **5. Photo Gallery**
  - Images
  - Responsive images
  - `<picture>`
  - `srcset`
  - `sizes`

---

## Intermediate Projects

- **6. Multi-Page Website**
  - Document structure
  - Semantic HTML
  - Navigation
  - Links
  - Accessibility

- **7. Product Page**
  - Semantic HTML
  - Tables
  - Forms
  - Images
  - Structured data

- **8. Blog with Comments**
  - Semantic HTML
  - Forms
  - Accessibility
  - Structured data

- **9. Landing Page**
  - Semantic HTML
  - Forms
  - Media
  - SEO
  - Accessibility

- **10. Documentation Site**
  - Semantic HTML
  - Navigation
  - Code blocks
  - Tables
  - Accessibility

---

## Advanced Projects

- **11. Accessible Form System**
  - Forms
  - Validation
  - ARIA
  - Error messages
  - Accessibility

- **12. SEO-Optimized Website**
  - Semantic HTML
  - Meta tags
  - Open Graph
  - Structured data
  - Sitemaps
  - Robots.txt

- **13. Progressive Web App**
  - HTML structure
  - Service workers
  - Web app manifest
  - Offline support
  - Performance

- **14. Web Components Library**
  - Custom elements
  - Shadow DOM
  - Templates
  - Slots
  - Accessibility

- **15. Email Template**
  - HTML email
  - Tables
  - Inline CSS
  - Email clients
  - Accessibility

---

## Expert Projects

- **16. Design System HTML**
  - Semantic HTML
  - Accessibility
  - Components  - Documentation
  - Testing

- **17. E-Commerce Platform**
  - Semantic HTML
  - Structured data
  - SEO
  - Accessibility
  - Performance

- **18. Accessibility-First Website**
  - Semantic HTML
  - ARIA
  - Keyboard accessibility
  - Screen reader support
  - Testing

- **19. High-Performance Website**
  - Critical rendering path
  - Resource hints
  - Lazy loading
  - Image optimization
  - Core Web Vitals

- **20. Production Web Platform**
  - Semantic HTML
  - Accessibility
  - SEO
  - Performance
  - Security
  - Testing
  - Documentation

---

# XV. Progressive HTML Learning Sequence

## Level 1 — HTML Fundamentals

- Master:
  - Document structure
  - Elements
  - Attributes
  - Text content
  - Links
  - Images

## Level 2 — Semantic HTML

- Master:
  - Sectioning elements
  - Grouping content
  - Text-level semantics
  - Semantic HTML best practices

## Level 3 — Lists and Tables

- Master:
  - Unordered lists
  - Ordered lists
  - Description lists
  - Tables
  - Table accessibility

## Level 4 — Forms

- Master:
  - Form fundamentals
  - Input elements
  - Labels
  - Select elements
  - Textarea
  - Buttons
  - Form grouping
  - Form validation
  - Form accessibility

## Level 5 — Media

- Master:
  - Images
  - Responsive images
  - Video
  - Audio
  - Canvas
  - SVG
  - Iframes
  - Media accessibility

## Level 6 — Metadata and SEO

- Master:
  - Metadata
  - Meta tags
  - Open Graph
  - Twitter Cards
  - Structured data
  - SEO fundamentals
  - Canonical URLs
  - Sitemaps
  - Robots.txt

## Level 7 — Accessibility

- Master:
  - Accessibility fundamentals
  - Semantic HTML
  - ARIA
  - Keyboard accessibility
  - Screen readers
  - Color and contrast
  - Accessible forms
  - Accessible images
  - Accessible media
  - Accessibility testing

## Level 8 — Modern HTML APIs

- Master:
  - Web Components
  - Shadow DOM
  - Templates
  - Custom Elements
  - Popover API
  - Dialog Element
  - Details and Summary
  - Lazy Loading
  - Responsive Images API
  - Drag and Drop API
  - File API
  - Geolocation API
  - Notifications API
  - Clipboard API
  - Intersection Observer
  - Mutation Observer
  - Resize Observer
  - Web Storage
  - IndexedDB
  - Web Workers
  - Service Workers
  - WebSockets
  - Server-Sent Events
  - Fetch API
  - Canvas API
  - WebGL
  - WebAssembly

## Level 9 — Performance

- Master:
  - Performance fundamentals
  - HTML optimization
  - Critical rendering path
  - Resource hints
  - Lazy loading
  - Image optimization
  - Font optimization
  - Script loading
  - HTML validation and parsing

## Level 10 — Production Engineering

- Master:
  - Semantic HTML
  - Accessibility
  - SEO
  - Performance
  - Security
  - Testing
  - Documentation
  - Production best practices

---

# XVI. Final HTML Competency Map

- **Foundations**

  - HTML history
  - How HTML works
  - Document structure
  - HTML syntax
  - HTML versions
  - Development setup

- **Elements**

  - Element fundamentals
  - Document metadata
  - Sectioning elements
  - Grouping content
  - Text-level semantics
  - Embedded content
  - Interactive elements
  - Scripting elements
  - Deprecated elements
  - Custom elements

- **Attributes**

  - Attribute fundamentals
  - Global attributes
  - Data attributes
  - ARIA attributes
  - Microdata
  - RDFa
  - Custom attributes

- **Text Content**

  - Headings
  - Paragraphs
  - Text formatting
  - Quotations
  - Abbreviations
  - Code and preformatted text
  - Line breaks and rules
  - Address
  - Time and data
  - Bidirectional text
  - Ruby annotations
  - Edits

- **Links**

  - Link fundamentals
  - Link types
  - Link attributes
  - Link accessibility
  - Link SEO

- **Images and Media**

  - Image fundamentals
  - Image attributes
  - Responsive images
  - Image formats
  - Image accessibility
  - Image SEO
  - Video
  - Audio
  - Media accessibility
  - Canvas
  - SVG
  - Iframes
  - Embed and object

- **Lists**

  - Unordered lists
  - Ordered lists
  - Description lists
  - Menu
  - Nested lists
  - List accessibility

- **Tables**

  - Table fundamentals
  - Table structure
  - Table attributes
  - Table accessibility
  - Responsive tables

- **Forms**

  - Form fundamentals
  - Input elements
  - Labels
  - Select elements
  - Textarea
  - Buttons
  - Form grouping
  - Form validation
  - Form accessibility
  - Advanced form elements
  - Form security

- **Metadata and SEO**

  - Metadata
  - Meta tags
  - Open Graph
  - Twitter Cards
  - Structured data
  - SEO fundamentals
  - Canonical URLs
  - Sitemaps
  - Robots.txt

- **Accessibility**

  - Accessibility fundamentals
  - Semantic HTML
  - ARIA
  - Keyboard accessibility
  - Screen readers
  - Color and contrast
  - Accessible forms
  - Accessible images
  - Accessible media
  - Accessibility testing

- **Modern HTML APIs**

  - Web Components
  - Shadow DOM
  - Templates
  - Custom Elements
  - Popover API
  - Dialog Element
  - Details and Summary
  - Lazy Loading
  - Responsive Images API
  - Drag and Drop API
  - File API
  - Geolocation API
  - Notifications API
  - Clipboard API
  - Intersection Observer
  - Mutation Observer
  - Resize Observer
  - Web Storage
  - IndexedDB
  - Web Workers
  - Service Workers
  - WebSockets
  - Server-Sent Events
  - Fetch API
  - Canvas API
  - WebGL
  - WebAssembly

- **Performance**

  - Performance fundamentals
  - HTML optimization
  - Critical rendering path
  - Resource hints
  - Lazy loading
  - Image optimization
  - Font optimization
  - Script loading
  - HTML validation and parsing

- **Production**

  - Semantic HTML
  - Accessibility
  - SEO
  - Performance
  - Security
  - Testing
  - Documentation

---

## Recommended Overall Progression

**HTML Fundamentals → Semantic HTML → Lists and Tables → Forms → Media → Metadata and SEO → Accessibility → Modern HTML APIs → Performance → Production Engineering**
