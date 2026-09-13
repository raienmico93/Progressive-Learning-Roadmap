Here's a set of study notes based on your outline for **"1. Introduction to HTML"**, expanded with explanations and examples to make the concepts clearer and more useful for learning or teaching.

---

# 1. Introduction to HTML

## 1.1 Definition of HTML

**HTML** stands for **HyperText Markup Language**.

- **HyperText**: Text that contains links (hyperlinks) to other documents or resources, allowing non-linear navigation.
- **Markup Language**: A system for annotating a document so that the structure and meaning of its content are identifiable. It uses **tags** (e.g., `<p>`, `<h1>`, `<a>`) to label content.

**Key point:** HTML is the **standard markup language for structuring documents on the Web**. Every web page you visit is, at its core, an HTML document interpreted by a browser.

**Example:**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>My First Page</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <p>This is a paragraph of text.</p>
  </body>
</html>
```

---

## 1.2 Purpose of HTML

HTML exists to give **meaning and structure** to content, not to control how it looks (that's CSS) or how it behaves (that's JavaScript).

| Purpose | Description | Example Element |
|---|---|---|
| Structuring content | Organizes text, images, and media into a logical document | `<header>`, `<section>`, `<article>` |
| Defining semantic meaning | Communicates what content *is*, not just how it looks | `<nav>`, `<main>`, `<em>`, `<strong>` |
| Connecting documents and resources | Links pages and embeds external files | `<a href="...">`, `<link>`, `<script>` |
| Embedding media and interactive elements | Adds images, audio, video, forms, canvas | `<img>`, `<video>`, `<form>`, `<canvas>` |

**Semantic vs. non-semantic example:**
```html
<!-- Non-semantic -->
<div class="header">Site Title</div>

<!-- Semantic -->
<header>Site Title</header>
```
Semantic elements improve accessibility, SEO, and code readability.

---

## 1.3 HTML versus Programming Languages

HTML is **not a programming language**. It's a markup language.

| Aspect | HTML (Markup) | Programming Language (e.g., Python, JavaScript) |
|---|---|---|
| Nature | Describes structure and content | Describes logic, computation, and behavior |
| Logic | No variables, loops, or conditionals | Supports variables, loops, conditionals |
| Execution | Parsed and rendered by the browser | Executed by an engine/runtime |
| Purpose | Presentation of content | Problem-solving and control flow |

### Relationship with CSS and JavaScript

The three core web technologies work together:

- **HTML** → Structure (the skeleton, the content)
- **CSS** → Presentation (layout, colors, typography)
- **JavaScript** → Behavior (interactivity, dynamic updates)

**Analogy:** HTML is the frame of a house, CSS is the paint and interior design, and JavaScript is the plumbing and electricity.

**Example of all three together:**
```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      /* CSS */
      p { color: blue; }
    </style>
  </head>
  <body>
    <!-- HTML -->
    <p id="message">Click the button</p>
    <button onclick="changeText()">Click me</button>

    <script>
      // JavaScript
      function changeText() {
        document.getElementById("message").textContent = "You clicked!";
      }
    </script>
  </body>
</html>
```

---

## 1.4 Evolution of HTML

HTML has evolved significantly since its creation.

### Early HTML (1991–1994)
- Created by **Tim Berners-Lee** at CERN.
- Very limited set of tags (headings, paragraphs, links, lists).
- Browser-specific extensions caused inconsistency.

### HTML 4 (1997–1999)
- Standardized by the **W3C**.
- Introduced **CSS** support, tables, forms, and scripting.
- Separated structure from presentation (partially).

### XHTML (2000–2002)
- Reformulated HTML as an **XML application**.
- Required strict syntax: all tags closed, lowercase, quoted attributes.
- Meant to be more rigorous, but was too strict for many authors.

### HTML5 (2008–2014)
- Major overhaul driven by **WHATWG** and later standardized by W3C.
- Added semantic elements (`<article>`, `<nav>`, `<header>`).
- Native support for audio, video, canvas, and offline storage.
- Simplified syntax (`<!DOCTYPE html>` instead of long DTD declarations).

### Living-Standard Development
- HTML is no longer versioned (no "HTML6").
- It's a **living standard** that continuously evolves.
- Maintained by **WHATWG** with input from browser vendors and the community.

**Evolution summary timeline:**
```
1991 → HTML (Tim Berners-Lee)
1997 → HTML 4.0 (W3C)
2000 → XHTML 1.0
2008 → HTML5 draft (WHATWG)
2014 → HTML5 Recommendation (W3C)
Today → Living Standard (WHATWG)
```

---

## 1.5 HTML Specifications

### WHATWG HTML Standard
- The **Web Hypertext Application Technology Working Group (WHATWG)** maintains the HTML Living Standard.
- It's the authoritative, continuously updated specification.
- URL: [https://html.spec.whatwg.org](https://html.spec.whatwg.org)

### Browser Implementation
- Browsers (Chrome, Firefox, Safari, Edge) implement the spec, but with slight variations and sometimes delays.
- Features may be "shipping" (implemented), "in development," or "experimental."

### Standards Compliance
- **Validation**: Tools like the [W3C Markup Validation Service](https://validator.w3.org/) check documents against the spec.
- **Compliance benefits**:
  - Cross-browser consistency
  - Better accessibility
  - Improved SEO
  - Future-proofing

**Tip:** Writing valid HTML doesn't guarantee identical rendering across all browsers, but it greatly improves reliability.

---

## Key Takeaways

1. **HTML is a markup language**, not a programming language — it structures content.
2. Its core purposes are **structuring, semantic labeling, linking, and embedding**.
3. HTML works alongside **CSS (presentation)** and **JavaScript (behavior)**.
4. HTML evolved from simple early versions → HTML 4 → XHTML → HTML5 → **Living Standard**.
5. The **WHATWG** maintains the spec; browsers implement it with varying degrees of compliance.

---

Would you like me to expand any section further — for example, with more code examples, a quiz, or a diagram of the HTML/CSS/JS relationship?
