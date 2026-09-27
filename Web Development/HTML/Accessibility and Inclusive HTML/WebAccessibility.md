# HTML Web Accessibility Fundamentals: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Web accessibility (often abbreviated as A11y) is the practice of designing and building web content so that it can be perceived, understood, navigated, and interacted with by everyone, including people with physical, sensory, cognitive, and situational disabilities.

**Technical Definition**

Web accessibility is defined by the W3C Web Accessibility Initiative (WAI) as the practice of ensuring that websites, tools, and technologies are designed and developed so that people with disabilities can use them. More specifically, people can perceive, understand, navigate, and interact with the Web, and they can contribute to the Web. It is governed by the Web Content Accessibility Guidelines (WCAG), an internationally recognised standard developed by the W3C. WCAG is built on four foundational principles, known by the acronym POUR: content must be **Perceivable**, **Operable**, **Understandable**, and **Robust**. The WHATWG HTML Living Standard and the WAI-ARIA (Accessible Rich Internet Applications) specification provide the technical foundation for implementing accessible markup.

**Beginner-Friendly Explanation**

Imagine trying to use a website if you couldn't see the screen, couldn't use a mouse, or couldn't read small text. Web accessibility is about making sure everyone — whether they have a permanent disability, a temporary injury, or are just in a difficult situation (bright sunlight, slow internet) — can still use your website. It's not just a nice-to-have; it's a fundamental requirement for building inclusive technology.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **POUR principles** | Perceivable, Operable, Understandable, Robust |
| **Assistive technology compatible** | Works with screen readers, magnifiers, voice control, and more |
| **Keyboard navigable** | All interactive elements operable via keyboard |
| **Semantically structured** | HTML elements used for their intended meaning |
| **WCAG compliance** | Conformance levels A, AA, AAA |
| **Legally mandated** | Required by laws like ADA, Section 508, EN 301 549 |
| **Universal benefit** | Improves usability for all users, not just those with disabilities |
| **Situational relevance** | Addresses temporary and situational limitations too |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Awareness of CSS and JavaScript basics
- Basic knowledge of how browsers render web pages
- Familiarity with forms and links (helpful but not required)

---

### Related Programming Areas

- **WCAG (Web Content Accessibility Guidelines)** – The international standard for web accessibility
- **ARIA (Accessible Rich Internet Applications)** – Attributes that supplement native HTML semantics
- **Semantic HTML** – The foundation of accessible markup
- **Screen Reader Technology** – JAWS, NVDA, VoiceOver, TalkBack
- **Keyboard Navigation** – Tab order, focus management, focus indicators
- **Assistive Technology** – Hardware and software that enables access
- **Legal Compliance** – ADA, Section 508, EN 301 549, AODA
- **Inclusive Design** – Designing for the full range of human diversity

---

## Core Concepts / Features

---

### 1. Accessibility Definition

#### Definitions

**Core Definition**

Web accessibility means designing and building web content so that it can be used by everyone, including people with physical, sensory, cognitive, or situational disabilities.

**Technical Definition**

The W3C WAI defines web accessibility as ensuring that websites, tools, and technologies are designed and developed so that people with disabilities can use them. More specifically, people can perceive, understand, navigate, and interact with the Web, and they can contribute to the Web. Web accessibility encompasses all disabilities that affect access to the Web, including auditory, cognitive, neurological, physical, speech, and visual disabilities. It also benefits people without disabilities, such as people using mobile phones, smart watches, smart TVs, and other devices with small screens, different input modes, and so on. The Web Accessibility Initiative (WAI) develops specifications, guidelines, techniques, and supporting resources.

**Beginner-Friendly Explanation**

Web accessibility means building websites that everyone can use. That includes people who are blind or have low vision, people who are deaf or hard of hearing, people who can‘t use a mouse, people with cognitive differences, and people who are temporarily injured or in a situation that limits their ability (like bright sunlight or a slow connection). If your website works for them, it works better for everyone.

#### Purposes

- To ensure equal access to information and functionality for all users
- To comply with legal requirements (ADA, Section 508, EN 301 549)
- To improve usability for all users, not just those with disabilities
- To expand the audience and reach of digital products
- To reduce the risk of discrimination and legal action
- To align with ethical and inclusive design principles

#### The POUR Principles

| Principle | Description | Example |
|---|---|---|
| **Perceivable** | Information must be presentable in ways users can perceive | Alt text for images; captions for video |
| **Operable** | Interface components must be operable | Keyboard navigation; no seizure-inducing flashing |
| **Understandable** | Information and operation must be understandable | Clear language; consistent navigation |
| **Robust** | Content must work with current and future technologies | Valid HTML; ARIA for dynamic content |

**WCAG Conformance Levels**

| Level | Description |
|---|---|
| **A** | Minimum level; basic accessibility features |
| **AA** | Standard level; most commonly required by law |
| **AAA** | Highest level; not always achievable for all content |

**Syntax Rules**

- All non-text content must have a text alternative
- All functionality must be available via keyboard
- Content must be readable and understandable
- Content must be robust enough to work with assistive technologies

**Constraints and Limitations**

- Accessibility is a spectrum; 100% accessibility is an ideal, not always achievable
- Legal requirements vary by jurisdiction
- Testing with real users and assistive technology is essential

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Accessible vs. Inaccessible Markup**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Accessibility Demo</title>
</head>
<body>
    <!-- INACCESSIBLE: No alt text, no semantic structure -->
    <div onclick="submitForm()">Click here to submit</div>
    <img src="chart.png">

    <!-- ACCESSIBLE: Semantic HTML, alt text, keyboard support -->
    <button type="submit">Submit Form</button>
    <img src="chart.png" alt="Bar chart showing Q1 sales of $1.2M, Q2 of $1.5M, Q3 of $1.8M, and Q4 of $2.1M">
</body>
</html>
```

**Expected Output**

The accessible version is usable by screen readers and keyboard users. The inaccessible version is not.

**Why This Output Occurs**

The `<button>` element is natively focusable and announced as a button by screen readers. The `<div onclick>` is not focusable and not announced as interactive. The `alt` attribute provides a text alternative for the chart.

---

**Example 2: POUR Principles in Practice**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>POUR Demo</title>
</head>
<body>
    <!-- Perceivable: alt text -->
    <img src="logo.png" alt="Acme Corporation logo">

    <!-- Operable: keyboard-accessible link -->
    <a href="/products">View Products</a>

    <!-- Understandable: clear label -->
    <label for="email">Email Address:</label>
    <input type="email" id="email" name="email" required>

    <!-- Robust: semantic HTML -->
    <nav aria-label="Main navigation">
        <ul>
            <li><a href="/">Home</a></li>
        </ul>
    </nav>
</body>
</html>
```

**Expected Output**

The page is perceivable (alt text), operable (keyboard-accessible link), understandable (clear label), and robust (semantic HTML).

**Why This Output Occurs**

Each element follows a POUR principle: the `alt` attribute makes the image perceivable; the link is operable; the label is understandable; the semantic `<nav>` and `<ul>` are robust.

#### Real-World Cases

**Case 1: Government Websites**

Government websites are legally required to meet WCAG 2.1 AA (or higher) in many jurisdictions.

**Case 2: E-Commerce**

Accessible e-commerce sites reach a larger audience and avoid lawsuits.

**Case 3: Education**

Educational platforms must be accessible to students with disabilities under laws like the ADA and Section 508.

---

### 2. Assistive Technologies

#### Definitions

**Core Definition**

Assistive technologies (AT) are hardware and software tools that people with disabilities use to access and interact with digital content.

**Technical Definition**

Assistive technology is defined by the W3C as “hardware and/or software that acts as a user agent, or along with a mainstream user agent, to provide functionality to meet the requirements of users with disabilities that go beyond those offered by mainstream user agents”. Examples include screen readers, screen magnifiers, alternative keyboards, voice control software, braille displays, and switch devices. WCAG 2.1 defines assistive technology as “hardware and/or software that acts as a user agent, or along with a mainstream user agent, to provide functionality to meet the requirements of users with disabilities that go beyond those offered by mainstream user agents”. The purpose of assistive technology is to enable people with disabilities to use technology.

**Beginner-Friendly Explanation**

Assistive technology is any tool that helps someone with a disability use a computer or phone. A screen reader reads the screen aloud for someone who is blind. A screen magnifier zooms in for someone with low vision. A head pointer lets someone with limited mobility control the cursor. Voice control lets someone speak commands instead of typing. Building accessible websites means making sure these tools can understand and interact with your content.

#### Purposes

- To enable people with disabilities to access digital content
- To provide alternative input and output methods
- To support independence and autonomy
- To bridge the gap between user needs and technology capabilities

#### Common Assistive Technologies

| Technology | Description | Used By |
|---|---|---|
| **Screen reader** | Reads screen content aloud or outputs to braille | Blind, low vision users |
| **Screen magnifier** | Enlarges portions of the screen | Low vision users |
| **Voice control** | Allows control via spoken commands | Motor impairments, RSI |
| **Alternative keyboard** | Specialised keyboards (e.g., one-handed, large keys) | Motor impairments |
| **Switch device** | Single or dual-switch input | Severe motor impairments |
| **Braille display** | Tactile output of screen content | Deaf-blind users |
| **Captioning** | Text representation of audio | Deaf, hard of hearing |
| **Text-to-speech** | Reads text aloud | Cognitive, learning disabilities |

**Syntax Rules**

- Use semantic HTML so AT can interpret content correctly
- Provide text alternatives for non-text content
- Ensure keyboard operability for all interactive elements
- Use ARIA to supplement native semantics when necessary
- Test with actual assistive technologies

**Constraints and Limitations**

- AT support varies by browser, operating system, and version
- Some ARIA attributes have inconsistent support
- Testing with real AT is essential; automated tools catch only ~30% of issues

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Screen Reader-Friendly Form**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Screen Reader Form</title>
</head>
<body>
    <form action="/submit" method="post">
        <!-- Label explicitly associated with input -->
        <label for="name">Full Name:</label>
        <input type="text" id="name" name="name" required
               aria-describedby="name-hint">
        <p id="name-hint">Please enter your first and last name.</p>

        <!-- Fieldset groups related controls -->
        <fieldset>
            <legend>Preferred Contact Method</legend>
            <label>
                <input type="radio" name="contact" value="email" checked>
                Email
            </label>
            <label>
                <input type="radio" name="contact" value="phone">
                Phone
            </label>
        </fieldset>

        <button type="submit">Submit</button>
    </form>
</body>
</html>
```

**Expected Output**

A screen reader announces: "Full Name, edit text, required. Please enter your first and last name." Then "Preferred Contact Method, group. Email, radio button, checked. Phone, radio button."

**Why This Output Occurs**

The `<label for>` associates the label with the input. `aria-describedby` links the hint. The `<fieldset>` and `<legend>` group the radio buttons and provide a group name. The `required` attribute is announced as "required."

---

**Example 2: Accessible Custom Widget**

```html
<button aria-expanded="false" aria-controls="menu" id="menu-button">
    Menu
</button>
<ul id="menu" hidden>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
</ul>

<script>
    const button = document.getElementById('menu-button');
    const menu = document.getElementById('menu');

    button.addEventListener('click', () => {
        const expanded = button.getAttribute('aria-expanded') === 'true';
        button.setAttribute('aria-expanded', !expanded);
        menu.hidden = expanded;
    });
</script>
```

**Expected Output**

Screen readers announce: "Menu, button, collapsed." After clicking: "Menu, button, expanded."

**Why This Output Occurs**

`aria-expanded` communicates the state of the menu. `aria-controls` links the button to the menu. The `hidden` attribute hides the menu visually and from AT when collapsed.

#### Real-World Cases

**Case 1: Screen Reader Users**

Blind users navigate the web using screen readers like JAWS, NVDA, or VoiceOver.

**Case 2: Screen Magnifier Users**

Low-vision users zoom in on portions of the screen using tools like ZoomText or built-in OS magnifiers.

**Case 3: Voice Control Users**

People with motor impairments use voice control (Dragon NaturallySpeaking, Voice Control) to navigate and interact.

---

### 3. Keyboard Access

#### Definitions

**Core Definition**

Keyboard access means ensuring that all interactive elements and functionality can be operated using only a keyboard, without requiring a mouse.

**Technical Definition**

WCAG Success Criterion 2.1.1 (Keyboard, Level A) requires that all functionality of the content be operable through a keyboard interface without requiring specific timings for individual keystrokes, except where the underlying function requires input that depends on the path of the user‘s movement and not just the endpoints. Native HTML interactive elements (`<a>`, `<button>`, `<input>`, `<select>`, `<textarea>`) are focusable and keyboard-operable by default. Custom interactive elements must be given `tabindex="0"` to be focusable and must handle keyboard events (`Enter`, `Space`, arrow keys). WCAG Success Criterion 2.4.3 (Focus Order) requires a logical tab order, and 2.4.7 (Focus Visible) requires a visible focus indicator.

**Beginner-Friendly Explanation**

Some people can‘t use a mouse — they navigate the web with the Tab key, activate links with Enter, and toggle checkboxes with Space. Keyboard access means making sure everything on your page can be reached and used with just the keyboard. Native HTML elements work with keyboards automatically. Custom elements need extra code.

#### Purposes

- To enable users who cannot use a mouse to navigate and interact
- To support users with motor impairments, repetitive strain injuries, or temporary injuries
- To comply with WCAG Success Criterion 2.1.1
- To improve usability for power users who prefer keyboards
- To ensure focus is visible and logical

#### Syntax Rules and Structure

**Standard Keyboard Interactions**

| Element | Key | Action |
|---|---|---|
| Links (`<a>`) | Enter | Activate link |
| Buttons (`<button>`) | Enter, Space | Activate button |
| Checkboxes | Space | Toggle |
| Radio buttons | Arrow keys | Move between options |
| Select | Arrow keys, Enter | Navigate and select |
| Text inputs | Tab | Focus next field |
| Modals | Escape | Close modal |

**Tabindex Values**

| Value | Behaviour |
|---|---|
| `tabindex="0"` | Focusable in natural tab order |
| `tabindex="-1"` | Focusable programmatically only |
| `tabindex="1+"` | Focusable in custom order (avoid) |

**Focus Visible CSS**

```css
:focus-visible {
    outline: 3px solid #005fcc;
    outline-offset: 2px;
}
```

**Syntax Rules**

- Use native HTML interactive elements whenever possible
- Custom interactive elements need `tabindex="0"` and keyboard event handlers
- Never use `tabindex` values greater than 0 (they disrupt natural order)
- Always provide a visible focus indicator
- Ensure the tab order follows the visual order

**Constraints and Limitations**

- Custom widgets must replicate all keyboard behaviours of native elements
- Focus indicators must have sufficient contrast (WCAG 1.4.11)
- Some assistive technologies have their own keyboard shortcuts that may conflict

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Native Keyboard Access**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Keyboard Access</title>
    <style>
        :focus-visible {
            outline: 3px solid #005fcc;
            outline-offset: 2px;
        }
    </style>
</head>
<body>
    <!-- Native elements are keyboard-accessible by default -->
    <a href="/">Home</a>
    <button type="button">Click Me</button>
    <input type="checkbox" id="agree">
    <label for="agree">I agree</label>
    <select name="country">
        <option>Choose...</option>
        <option>USA</option>
        <option>UK</option>
    </select>
</body>
</html>
```

**Expected Output**

Tab moves focus between each element. Enter activates links and buttons. Space toggles the checkbox. Arrow keys navigate the select.

**Why This Output Occurs**

Native HTML elements have built-in keyboard support. The `:focus-visible` CSS provides a visible focus indicator.

---

**Example 2: Custom Widget with Keyboard Support**

```html
<div role="button" tabindex="0" id="customBtn"
     aria-pressed="false">
    Toggle
</div>

<script>
    const btn = document.getElementById('customBtn');

    btn.addEventListener('click', toggle);
    btn.addEventListener('keydown', (event) => {
        if (event.key === 'Enter' || event.key === ' ') {
            event.preventDefault();
            toggle();
        }
    });

    function toggle() {
        const pressed = btn.getAttribute('aria-pressed') === 'true';
        btn.setAttribute('aria-pressed', !pressed);
        btn.textContent = pressed ? 'Off' : 'On';
    }
</script>
```

**Expected Output**

Tab focuses the custom button. Enter or Space toggles it. Screen readers announce "Toggle, button, not pressed" and then "Toggle, button, pressed."

**Why This Output Occurs**

`tabindex="0"` makes the div focusable. The `keydown` handler responds to Enter and Space. `role="button"` and `aria-pressed` communicate the role and state.

#### Real-World Cases

**Case 1: Government Websites**

Government websites must be fully keyboard-accessible to comply with Section 508 and WCAG.

**Case 2: Power Users**

Many developers and power users navigate primarily with the keyboard.

**Case 3: Assistive Technology Users**

Screen reader and switch device users rely on keyboard access.

---

### 4. Screen Readers

#### Definitions

**Core Definition**

A screen reader is an assistive technology that converts digital text and interface elements into spoken language or braille output for users who are blind or have low vision.

**Technical Definition**

A screen reader is a software application that attempts to identify and interpret what is being displayed on the screen. This interpretation is then re-presented to the user with text-to-speech, sound icons, or a braille output device. Screen readers are a form of assistive technology (AT). The browser‘s accessibility tree, built from the DOM and the HTML Accessibility API Mappings (HTML-AAM), is what screen readers consume. Semantic HTML provides the structure that screen readers use to announce roles, names, states, and values. WCAG guidelines require that content be perceivable and understandable by screen readers.

**Beginner-Friendly Explanation**

A screen reader is software that reads a webpage aloud. Blind users rely on it to hear what’s on the screen — the text, the headings, the links, the buttons, the form fields. It also announces the role of each element ("link," "button," "heading level 2") and its state ("checked," "expanded"). To make your site work with screen readers, you need semantic HTML and proper labelling.

#### Purposes

- To provide access to digital content for blind and low-vision users
- To announce the role, name, state, and value of interface elements
- To enable navigation by headings, landmarks, lists, and links
- To output content in braille for deaf-blind users
- To satisfy WCAG requirements for perceivability

#### Screen Reader Announcements

| Element | Announcement |
|---|---|
| `<h1>` | "Heading level 1, [text]" |
| `<a href>` | "[text], link" |
| `<button>` | "[text], button" |
| `<img alt="...">` | "Image, [alt text]" |
| `<input required>` | "Edit text, required" |
| `<nav>` | "Navigation landmark" |
| `<ul>` | "List, N items" |
| `<fieldset>` | "[legend text], group" |

**Common Screen Readers**

| Screen Reader | Platform | Cost |
|---|---|---|
| **JAWS** | Windows | Commercial |
| **NVDA** | Windows | Free, open source |
| **VoiceOver** | macOS, iOS | Built-in |
| **TalkBack** | Android | Built-in |
| **Narrator** | Windows | Built-in |
| **Orca** | Linux | Free |

**Syntax Rules**

- Use semantic HTML so screen readers announce correct roles
- Provide text alternatives for non-text content (`alt`, `aria-label`)
- Use headings to create a navigable outline
- Use landmarks (`<nav>`, `<main>`, `<header>`, `<footer>`) for navigation
- Ensure form fields have associated labels
- Use `aria-live` for dynamic content updates

**Constraints and Limitations**

- Screen reader support for ARIA varies
- Some screen readers handle complex widgets differently
- Testing with real screen readers is essential
- Automated tools cannot fully test screen reader experience

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Semantic Structure for Screen Readers**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Screen Reader Structure</title>
</head>
<body>
    <header>
        <h1>Site Title</h1>
        <nav aria-label="Main">
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/about">About</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <h2>Article Title</h2>
            <p>Article content.</p>
        </article>
    </main>

    <footer>
        <p>© 2026</p>
    </footer>
</body>
</html>
```

**Expected Output**

A screen reader announces: "Site Title, heading level 1. Main navigation, navigation landmark. Home, link. About, link. Main, main landmark. Article Title, heading level 2."

**Why This Output Occurs**

Semantic elements (`<header>`, `<nav>`, `<main>`, `<article>`, `<footer>`) create a structure that screen readers can interpret and navigate.

---

**Example 2: Live Region for Dynamic Content**

```html
<button id="load">Load Message</button>
<div id="status" aria-live="polite"></div>

<script>
    document.getElementById('load').addEventListener('click', () => {
        document.getElementById('status').textContent = 'Data loaded successfully.';
    });
</script>
```

**Expected Output**

After clicking, the screen reader announces "Data loaded successfully."

**Why This Output Occurs**

`aria-live="polite"` tells the screen reader to announce changes to the live region.

#### Real-World Cases

**Case 1: Blind Users**

Blind users rely entirely on screen readers to browse the web.

**Case 2: Deaf-Blind Users**

Deaf-blind users use screen readers with braille displays.

**Case 3: Cognitive Disabilities**

Some users with cognitive disabilities use screen readers to supplement reading.

---

### 5. Semantic Structure

#### Definitions

**Core Definition**

Semantic structure is the practice of using HTML elements according to their intended meaning, so that assistive technologies can correctly interpret the type, purpose, and relationships of content.

**Technical Definition**

Semantic HTML is the use of HTML elements for their intended purpose. The WHATWG HTML Living Standard defines the meaning and content model of each element. Semantic elements (`<article>`, `<aside>`, `<details>`, `<figcaption>`, `<figure>`, `<footer>`, `<header>`, `<main>`, `<mark>`, `<nav>`, `<section>`, `<summary>`, `<time>`) carry intrinsic meaning that browsers and assistive technologies expose through the accessibility tree. The HTML Accessibility API Mappings (HTML-AAM) define how HTML elements map to platform accessibility APIs. WCAG Success Criterion 1.3.1 (Info and Relationships) requires that information, structure, and relationships conveyed through presentation can be programmatically determined.

**Beginner-Friendly Explanation**

Semantic HTML means using the right tag for the right job. Don‘t use a `<div>` when you should use a `<button>`. Don’t use `<b>` when you mean `<strong>`. Don‘t use a table for layout. Semantic elements tell the browser and screen readers what things are. This is the foundation of web accessibility.

#### Purposes

- To communicate the meaning and structure of content to assistive technology
- To enable screen reader navigation by landmarks, headings, and lists
- To satisfy WCAG Success Criterion 1.3.1
- To improve SEO by providing clear content signals
- To make code more maintainable and understandable

#### Semantic Elements Overview

| Element | Meaning | ARIA Role |
|---|---|---|
| `<header>` | Introductory content | `banner` |
| `<nav>` | Navigation links | `navigation` |
| `<main>` | Main content | `main` |
| `<article>` | Self-contained composition | `article` |
| `<section>` | Thematic grouping | `region` (if labelled) |
| `<aside>` | Tangentially related content | `complementary` |
| `<footer>` | Footer for nearest ancestor | `contentinfo` |
| `<h1>`–`<h6>` | Headings | `heading` |
| `<ul>`, `<ol>` | Lists | `list` |
| `<button>` | Button | `button` |
| `<a href>` | Link | `link` |
| `<table>` | Tabular data | `table` |

**Syntax Rules**

- Use the most semantic element available for the content
- Use headings to create a logical outline
- Use landmarks to identify major page regions
- Use `<button>` for actions and `<a>` for navigation
- Use `<table>` only for tabular data
- Use `<ul>`, `<ol>`, and `<dl>` for lists

**Constraints and Limitations**

- Overuse of `<div>` and `<span>` creates "divitis" and harms accessibility
- ARIA can supplement but not replace native semantics
- The first rule of ARIA is to use native HTML

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Semantic vs. Non-Semantic Markup**

```html
<!-- NON-SEMANTIC: Divs and spans everywhere -->
<div class="header">
    <div class="title">Site Title</div>
    <div class="nav">
        <div class="nav-item" onclick="navigate('/')">Home</div>
    </div>
</div>
<div class="main">
    <div class="article">
        <div class="heading">Article Title</div>
        <div class="content">Article content.</div>
    </div>
</div>

<!-- SEMANTIC: Proper HTML elements -->
<header>
    <h1>Site Title</h1>
    <nav>
        <ul>
            <li><a href="/">Home</a></li>
        </ul>
    </nav>
</header>
<main>
    <article>
        <h2>Article Title</h2>
        <p>Article content.</p>
    </article>
</main>
```

**Expected Output**

The semantic version is navigable by screen readers using landmarks, headings, and lists. The non-semantic version is just a series of divs with no meaning.

**Why This Output Occurs**

Semantic elements carry intrinsic meaning that browsers expose through the accessibility tree. Divs and spans carry no meaning.

---

**Example 2: Accessible Data Table**

```html
<table>
    <caption>Quarterly Sales</caption>
    <thead>
        <tr>
            <th scope="col">Region</th>
            <th scope="col">Q1</th>
            <th scope="col">Q2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">North</th>
            <td>$45,000</td>
            <td>$52,000</td>
        </tr>
        <tr>
            <th scope="row">South</th>
            <td>$38,000</td>
            <td>$41,000</td>
        </tr>
    </tbody>
</table>
```

**Expected Output**

A screen reader announces: "Quarterly Sales, table, 3 columns, 2 rows. Region, column header. Q1, column header. North, row header. $45,000."

**Why This Output Occurs**

The `<caption>` provides the table name. `<thead>` and `<th scope>` define headers. `<tbody>` contains the data. The screen reader uses this structure to announce header-data relationships.

#### Real-World Cases

**Case 1: Screen Reader Navigation**

Screen reader users navigate by landmarks, headings, lists, and tables. Semantic HTML makes this possible.

**Case 2: Search Engine Optimization**

Search engines use semantic HTML to understand content structure and hierarchy.

**Case 3: Government Compliance**

Government websites must use semantic HTML to comply with WCAG 1.3.1.

---

### 6. Choosing the Right Accessibility Approach

#### Definitions

**Core Definition**

Choosing the right accessibility approach means selecting the appropriate combination of semantic HTML, ARIA attributes, keyboard support, and testing to meet the needs of all users.

**Technical Definition**

The choice depends on the content type and interaction. Native HTML elements provide built-in accessibility semantics and keyboard support. ARIA should be used only when native HTML cannot express the required semantics or behaviour. The first rule of ARIA is to use native HTML; the second is to not change native semantics unless necessary; the third is to make all interactive controls keyboard-accessible.

#### Decision Guide

| Scenario | Recommended Approach |
|---|---|
| Standard link | `<a href="...">` |
| Standard button | `<button>` |
| Icon-only button | `<button aria-label="...">` |
| Custom dropdown | Native `<select>` or ARIA `role="listbox"` with keyboard support |
| Modal dialog | `<dialog>` or ARIA `role="dialog"` with focus management |
| Tabs | ARIA `role="tablist"`, `role="tab"`, `role="tabpanel"` |
| Live updates | `aria-live="polite"` or `role="status"` |
| Error messages | `aria-describedby` + `aria-invalid` |
| Form groups | `<fieldset>` + `<legend>` |
| Data tables | `<table>`, `<caption>`, `<th scope>`, `<thead>`, `<tbody>` |

---

## References

- W3C – Introduction to Web Accessibility – https://www.w3.org/WAI/fundamentals/accessibility-intro/
- W3C – Web Content Accessibility Guidelines (WCAG) 2.1 – https://www.w3.org/TR/WCAG21/
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.1.1: Keyboard – https://www.w3.org/WAI/WCAG21/Understanding/keyboard.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.7: Focus Visible – https://www.w3.org/WAI/WCAG21/Understanding/focus-visible.html
- MDN Web Docs – Accessibility – https://developer.mozilla.org/en-US/docs/Web/Accessibility
- MDN Web Docs – HTML: A good basis for accessibility – https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML
- MDN Web Docs – ARIA – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA
- W3C – WAI-ARIA Authoring Practices Guide – https://www.w3.org/WAI/ARIA/apg/
- WHATWG – HTML Living Standard – https://html.spec.whatwg.org/multipage/
- W3C – HTML Accessibility API Mappings (HTML-AAM) – https://w3c.github.io/html-aam/
- WebAIM – Introduction to Web Accessibility – https://webaim.org/intro/
- WebAIM – Keyboard Accessibility – https://webaim.org/techniques/keyboard/
- WebAIM – Screen Reader User Survey – https://webaim.org/projects/screenreadersurvey/
- The A11Y Project – Checklist – https://www.a11yproject.com/checklist/
- GOV.UK – Accessibility – https://www.gov.uk/help/accessibility
- Section508.gov – https://www.section508.gov/
- Deque – axe-core – https://www.deque.com/axe/