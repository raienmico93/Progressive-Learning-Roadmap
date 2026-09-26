# HTML Figure Content: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML figure content is a semantic grouping mechanism that wraps self-contained content — such as images, diagrams, illustrations, code listings, quotations, or poems — into a single referenced unit, optionally accompanied by a visible caption.

**Technical Definition**

The `<figure>` element represents self-contained content, potentially with an optional caption, which is specified using the `<figcaption>` element. The figure, its caption, and its contents are referenced as a single unit. The `<figure>` element is categorised as flow content and palpable content. Its permitted content is either a `<figcaption>` element followed by flow content, or flow content followed by a `<figcaption>` element, or just flow content. Its DOM interface is `HTMLElement` . The `<figcaption>` element represents a caption or legend describing the rest of the contents of its parent `<figure>` element, providing the `<figure>` an accessible name. It must be the first or last child of the `<figure>` element .

**Beginner-Friendly Explanation**

Think of a `<figure>` like a photo in a magazine with a caption underneath. The photo and its caption belong together as one unit. You can move the whole thing to a different page and it still makes sense on its own. The `<figure>` tag wraps the content (the photo, diagram, or code), and the `<figcaption>` tag holds the caption or description. This tells browsers and screen readers that the content and its caption are connected.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Self-contained** | The figure can be moved to another part of the document without affecting the main flow |
| **Optional caption** | A `<figcaption>` may be placed as the first or last child of the `<figure>` |
| **Accessible name** | The `<figcaption>` provides the accessible name for the parent `<figure>` |
| **Versatile content** | Can contain images, diagrams, code snippets, quotations, poems, and more |
| **Semantic grouping** | Groups the content and its caption as a single referenced unit |
| **Global attributes only** | Both elements accept only global attributes |

---

### Prerequisites

- Basic familiarity with HTML document structure
- Understanding of the `<img>` element and its attributes
- Awareness of block-level and inline-level elements
- Basic knowledge of accessibility principles

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Figure captions provide accessible names for grouped content
- **Semantic HTML** – Figure content adds meaning and structure to visual and illustrative elements
- **Responsive Images** – Figures can contain `<picture>` elements and responsive images
- **Content Management Systems** – CMS platforms automatically wrap images in figures
- **Academic and Technical Writing** – Figures are standard for diagrams, charts, and code listings

---

## Core Concepts / Features

---

### 1. The `<figure>` Element

#### Definitions

**Core Definition**

The `<figure>` element represents self-contained content, optionally with a caption, that is typically referenced as a single unit from the main flow of the document.

**Technical Definition**

The `<figure>` HTML element represents self-contained content, potentially with an optional caption, which is specified using the `<figcaption>` element. It is categorised as flow content and palpable content. Its permitted content is a `<figcaption>` element followed by flow content, flow content followed by a `<figcaption>` element, or just flow content. Both the starting and ending tags are mandatory. It accepts only global attributes. Its DOM interface is `HTMLElement` .

**Beginner-Friendly Explanation**

The `<figure>` tag is a container for something that stands on its own — like a photo, a diagram, or a code example. You can move it around without breaking the meaning of the page. It‘s like a framed picture: the frame (the figure) holds the picture (the content) and any label (the caption) together as one unit.

#### Purposes

- To group self-contained content that is referenced as a single unit
- To associate a caption with an image, diagram, or other illustrative content
- To provide an accessible name for grouped content
- To allow content to be moved without affecting the document’s meaning

#### Syntax Rules and Structure

**General Syntax**

```html
<figure>
    <!-- content (image, code, diagram, etc.) -->
    <figcaption>Optional caption</figcaption>
</figure>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<figure>` | Opening tag; indicates the start of a figure |
| `Content` | Flow content; the self-contained content |
| `<figcaption>` | Optional caption; first or last child |
| `</figure>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- The `<figcaption>` must be the first or last child of the `<figure>`
- The `<figure>` element accepts only global attributes
- The element may contain any flow content

**Constraints and Limitations**

- A figure should not be used for content that is not self-contained
- The caption is optional; a figure without a caption is valid
- The `<figure>` element should not be nested inside another `<figure>` element

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Figure with Image and Caption**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Figure Demo</title>
</head>
<body>
    <figure>
        <img src="elephant.jpg" alt="Elephant at sunset" width="400" height="300">
        <figcaption>An elephant at sunset</figcaption>
    </figure>
</body>
</html>
```

**Expected Output**

A photograph of an elephant at sunset with the caption “An elephant at sunset” displayed below it.

**Why This Output Occurs**

The `<figure>` element groups the image and its caption as a single unit. The `<figcaption>` element provides the visible caption and the accessible name for the figure .

---

**Example 2: Figure Without a Caption**

```html
<figure>
    <img src="diagram.png" alt="System architecture diagram" width="600" height="400">
</figure>
```

**Expected Output**

The diagram is displayed without a visible caption. Screen readers use the image’s `alt` text as the accessible description.

**Why This Output Occurs**

The `<figcaption>` is optional. When absent, the figure has no explicit accessible name, but the image’s `alt` text still provides a description.

#### Real-World Cases

**Case 1: Scientific Papers**

Scientific papers use `<figure>` for charts, graphs, and diagrams, each with a numbered caption such as “Figure 1: Experimental results.”

**Case 2: News Articles**

News articles use `<figure>` for photographs with descriptive captions and photo credits.

**Case 3: Documentation**

Technical documentation uses `<figure>` for screenshots with explanatory captions.

---

### 2. The `<figcaption>` Element

#### Definitions

**Core Definition**

The `<figcaption>` element represents a caption or legend describing the rest of the contents of its parent `<figure>` element.

**Technical Definition**

The `<figcaption>` HTML element represents a caption or legend describing the rest of the contents of its parent `<figure>` element, providing the `<figure>` an accessible name. It has no content categories. Its permitted content is flow content. Both the starting and ending tags are mandatory. Its permitted parent is a `<figure>` element, and it must be the first or last child. Its implicit ARIA role is “no corresponding role,” but it permits `group`, `none`, and `presentation` roles. Its DOM interface is `HTMLElement` .

**Beginner-Friendly Explanation**

The `<figcaption>` tag is the caption for your figure. It goes inside the `<figure>` tag, either at the very beginning or the very end. It describes what the figure shows. Browsers typically display it as a small block of text near the content.

#### Purposes

- To provide a visible caption for a figure
- To give the `<figure>` an accessible name for screen readers
- To describe the content of the figure for all users
- To provide context for the figure within the document flow

#### Syntax Rules and Structure

**General Syntax**

```html
<figure>
    <figcaption>Caption text</figcaption>
    <!-- content -->
</figure>

<!-- OR -->

<figure>
    <!-- content -->
    <figcaption>Caption text</figcaption>
</figure>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<figcaption>` | Opening tag; indicates the caption |
| `Caption text` | Flow content; the caption text |
| `</figcaption>` | Closing tag; required |

**Syntax Rules**

- The `<figcaption>` must be the first or last child of a `<figure>` element
- Both start and end tags are mandatory
- It permits only global attributes
- Only the first `<figcaption>` in a figure is presented as the caption

**Constraints and Limitations**

- A `<figcaption>` cannot be used outside a `<figure>` element
- It must be a direct child of the `<figure>`
- It should not be used as a substitute for the `alt` attribute on images

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Caption Before Content**

```html
<figure>
    <figcaption>Fig. 1 — A diagram of the water cycle</figcaption>
    <img src="water-cycle.png" alt="Diagram showing evaporation, condensation, and precipitation" width="600" height="400">
</figure>
```

**Expected Output**

The caption “Fig. 1 — A diagram of the water cycle” appears above the diagram.

**Why This Output Occurs**

The `<figcaption>` is placed as the first child of the `<figure>`, so the browser renders it before the image. Both placements are valid .

---

**Example 2: Caption After Content**

```html
<figure>
    <img src="chart.png" alt="Bar chart showing quarterly sales" width="600" height="400">
    <figcaption>Quarterly sales data for 2026</figcaption>
</figure>
```

**Expected Output**

The caption “Quarterly sales data for 2026” appears below the chart.

**Why This Output Occurs**

The `<figcaption>` is placed as the last child of the `<figure>`, so the browser renders it after the image.

#### Real-World Cases

**Case 1: Academic Figures**

Academic papers use `<figcaption>` for numbered captions like “Figure 3: Results of the experiment.”

**Case 2: Photo Galleries**

Photo galleries use `<figcaption>` for photo descriptions, dates, and credits.

**Case 3: Technical Documentation**

Documentation uses `<figcaption>` to describe screenshots, diagrams, and code examples.

---

### 3. Images with Captions

#### Definitions

**Core Definition**

Images with captions are photographs, illustrations, or graphics wrapped in a `<figure>` element with a `<figcaption>` providing a visible description or title.

**Technical Definition**

When an image is wrapped in a `<figure>` with a `<figcaption>`, the caption serves as the visible description and the accessible name for the figure. The image itself should still have an `alt` attribute for its content. The `<figcaption>` should describe the image or provide additional context, not simply repeat the `alt` text .

**Beginner-Friendly Explanation**

A photo with a caption is like a picture in a book with a label underneath. The `<figure>` wraps the photo and the caption together. The `alt` text describes the photo for people who can‘t see it, and the `<figcaption>` provides a visible label for everyone.

#### Purposes

- To provide a visible caption for an image
- To give the image an accessible name via the caption
- To add context or credits to a photograph
- To group the image and its description as a single unit

#### Syntax Rules and Structure

**General Syntax**

```html
<figure>
    <img src="photo.jpg" alt="Description of the image content" width="400" height="300">
    <figcaption>Visible caption text</figcaption>
</figure>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<figure>` | Container for the image and caption |
| `<img>` | The image; requires `alt` attribute |
| `<figcaption>` | The visible caption |

**Syntax Rules**

- The image must have an `alt` attribute describing its content
- The `<figcaption>` should provide additional context or a title
- The `<figcaption>` must be the first or last child of the `<figure>`
- Do not use the `<figcaption>` as a replacement for the `alt` attribute

**Constraints and Limitations**

- The `<figcaption>` is not announced as the image’s alt text by all screen readers; the `alt` attribute is still required
- A figure with only an image and no caption is valid but does not provide a visible description

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Image with Alt and Figcaption**

```html
<figure>
    <img src="sunset.jpg" alt="A vibrant orange sunset over the ocean" width="600" height="400">
    <figcaption>Sunset over the Pacific Ocean, photographed from Santa Monica, California.</figcaption>
</figure>
```

**Expected Output**

The photograph displays with the caption below it. Screen readers announce the `alt` text when encountering the image and the `<figcaption>` content as the figure’s caption.

**Why This Output Occurs**

The `alt` attribute describes the image content for screen readers. The `<figcaption>` provides a visible caption with additional context (location). Both are necessary for full accessibility .

---

**Example 2: Figure with Multiple Images**

```html
<figure>
    <img src="orang-utan.jpg" alt="Baby orangutan hanging from a rope" width="300" height="200">
    <img src="macaque.jpg" alt="Macaque in the trees" width="300" height="200">
    <figcaption>A cheeky macaque, Lower Kintaganban River, Borneo.</figcaption>
</figure>
```

**Expected Output**

Two images are displayed side by side or stacked, with a single caption below them.

**Why This Output Occurs**

A `<figure>` can contain multiple images grouped under a single caption. The `<figcaption>` describes the group as a whole.

#### Real-World Cases

**Case 1: News Photography**

News websites use figures for article images with captions and photo credits.

**Case 2: E-Commerce Product Photos**

Product pages use figures for product images with descriptive captions.

**Case 3: Travel Blogs**

Travel blogs use figures for photos with location captions.

---

### 4. Diagrams and Illustrations

#### Definitions

**Core Definition**

Diagrams and illustrations are visual explanations — such as flowcharts, architectural diagrams, or scientific illustrations — wrapped in a `<figure>` with a `<figcaption>` describing their content.

**Technical Definition**

Diagrams and illustrations are self-contained visual content that is referenced from the main text. When wrapped in a `<figure>`, the caption provides the accessible name and visible description. For complex diagrams, a longer textual description may be needed in addition to the `alt` and `<figcaption>` .

**Beginner-Friendly Explanation**

A diagram is a picture that explains how something works — like a flowchart or a map. When you put it in a `<figure>`, you add a caption that tells readers what the diagram shows. This is especially important for people who can‘t see the diagram, because the caption gives them the information in text form.

#### Purposes

- To explain complex processes or structures visually
- To provide a caption that describes the diagram‘s purpose
- To group the diagram and its description as a single unit
- To make complex visual information accessible through text alternatives

#### Syntax Rules and Structure

**General Syntax**

```html
<figure>
    <img src="diagram.png" alt="Description of the diagram" width="600" height="400">
    <figcaption>Figure 1: Explanation of the diagram</figcaption>
</figure>
```

**Syntax Rules**

- The `alt` attribute should briefly describe the diagram
- The `<figcaption>` should provide the figure number and a descriptive title
- For complex diagrams, consider providing a longer description in the surrounding text or via `aria-describedby`

**Constraints and Limitations**

- A short `alt` and caption may not convey all the information in a complex diagram
- Complex diagrams may require a textual equivalent in addition to the figure

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Flowchart Diagram**

```html
<figure>
    <img src="user-flow.png" alt="User registration flow: sign up, verify email, create profile" width="800" height="500">
    <figcaption>Figure 2: The user registration flow from account creation to profile setup.</figcaption>
</figure>
```

**Expected Output**

The flowchart is displayed with a caption identifying it as “Figure 2” and describing its purpose.

**Why This Output Occurs**

The `<figure>` groups the diagram with its caption. The `alt` provides a brief summary, and the `<figcaption>` provides the figure number and title.

---

**Example 2: Scientific Illustration**

```html
<figure>
    <img src="cell-structure.png" alt="Diagram of a plant cell showing the cell wall, nucleus, chloroplasts, and vacuole" width="600" height="400">
    <figcaption>Figure 3: Cross-section of a plant cell with labelled organelles.</figcaption>
</figure>
```

**Expected Output**

The cell diagram is displayed with a numbered caption.

**Why This Output Occurs**

The `alt` text describes the key structures shown in the diagram. The `<figcaption>` provides the figure number and a formal title.

#### Real-World Cases

**Case 1: Academic Textbooks**

Textbooks use figures for diagrams with numbered captions like “Figure 4.2: The structure of DNA.”

**Case 2: Technical Manuals**

Manuals use figures for assembly diagrams with step-by-step captions.

**Case 3: Medical Illustrations**

Medical content uses figures for anatomical diagrams with descriptive captions.

---

### 5. Code Examples

#### Definitions

**Core Definition**

Code examples are snippets of programming code wrapped in a `<figure>` element, with a `<figcaption>` providing a description, title, or explanation.

**Technical Definition**

Code listings can be self-contained content that is referenced from the main text. When wrapped in a `<figure>`, the `<figcaption>` provides a caption for the code example. The code itself is typically marked up with `<pre>` and `<code>` elements inside the figure. This is explicitly supported by the HTML specification, which states that figures can be used to annotate code listings .

**Beginner-Friendly Explanation**

A code example is a snippet of programming code that you show to readers. You put it in a `<figure>` with a `<figcaption>` that explains what the code does. This is useful in tutorials and documentation because it keeps the code and its explanation together.

#### Purposes

- To display code examples with descriptive captions
- To group code with its title or explanation
- To provide a semantic container for code listings
- To enable code examples to be referenced from the main text

#### Syntax Rules and Structure

**General Syntax**

```html
<figure>
    <pre><code>
    <!-- code here -->
    </code></pre>
    <figcaption>Description of the code example</figcaption>
</figure>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<figure>` | Container for the code and caption |
| `<pre>` | Preserves whitespace and line breaks |
| `<code>` | Marks the content as code |
| `<figcaption>` | Caption describing the code |

**Syntax Rules**

- The code should be wrapped in `<pre><code>` for proper formatting
- The `<figcaption>` should describe what the code does or its purpose
- The `<figcaption>` must be the first or last child of the `<figure>`

**Constraints and Limitations**

- Code inside `<pre>` must have `<` and `>` escaped as `&lt;` and `&gt;`
- The `<figcaption>` is not a substitute for code comments

#### Annotated Complete Step-by-Step Code Examples

**Example 1: JavaScript Function with Caption**

```html
<figure>
    <pre><code>
function greet(name) {
    return "Hello, " + name + "!";
}
    </code></pre>
    <figcaption>A simple JavaScript function that greets a user by name.</figcaption>
</figure>
```

**Expected Output**

The code is displayed in a monospaced font with preserved formatting, and the caption appears below it.

**Why This Output Occurs**

The `<pre>` element preserves whitespace and line breaks, and `<code>` marks the content as code. The `<figcaption>` provides a description of the code’s purpose.

---

**Example 2: HTML Example with Caption**

```html
<figure>
    <pre><code>
&lt;figure&gt;
    &lt;img src="photo.jpg" alt="A photo"&gt;
    &lt;figcaption&gt;A photo with a caption&lt;/figcaption&gt;
&lt;/figure&gt;
    </code></pre>
    <figcaption>Basic HTML markup for a figure with an image and caption.</figcaption>
</figure>
```

**Expected Output**

The HTML code is displayed as text with the angle brackets visible, and the caption explains what the code does.

**Why This Output Occurs**

The `<` and `>` characters are escaped using `&lt;` and `&gt;` so the browser displays them as text rather than interpreting them as HTML tags. The `<figcaption>` describes the code example.

#### Real-World Cases

**Case 1: Programming Tutorials**

Tutorials use figures for code examples with captions explaining the code’s purpose.

**Case 2: API Documentation**

API docs use figures for code snippets with captions describing the endpoint or function.

**Case 3: Technical Blog Posts**

Blog posts use figures for code examples with explanatory captions.

---

### 6. Accessibility Considerations

#### Definitions

**Core Definition**

Accessibility considerations for figure content involve ensuring that the figure, its caption, and any images within it are properly understood by screen readers and other assistive technologies.

**Technical Definition**

The `<figcaption>` element provides the `<figure>` with an accessible name. When an image inside a `<figure>` lacks an `alt` attribute, the HTML Accessibility API Mappings (AAM) indicate that the image should get its accessible name from the `<figcaption>` content. However, the `alt` attribute is still required for the image itself to describe its content. The `<figure>` element may also be given a `role="group"` and an `aria-labelledby` attribute referencing the `<figcaption>`’s `id` for enhanced screen reader support .

**Beginner-Friendly Explanation**

Screen readers need to know what a figure contains and what it‘s for. The `<figcaption>` tells the screen reader the name of the figure. The image inside still needs an `alt` attribute to describe what it shows. If you want to be extra careful, you can add a `role="group"` and an `aria-labelledby` to make the relationship between the figure and its caption even clearer.

#### Purposes

- To ensure screen readers announce the figure and its caption
- To provide an accessible name for the figure
- To describe the content of images within the figure
- To satisfy WCAG Success Criterion 1.1.1 (Non-text Content)

#### Syntax Rules and Structure

**Enhanced Accessibility Markup**

```html
<figure role="group" aria-labelledby="fig-caption">
    <img src="diagram.png" alt="Description of the diagram content">
    <figcaption id="fig-caption">Figure 1: The diagram caption</figcaption>
</figure>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `role="group"` | Groups the figure and caption as a single unit |
| `aria-labelledby` | References the `id` of the `<figcaption>` |
| `id` on `<figcaption>` | Creates a target for `aria-labelledby` |

**Syntax Rules**

- The `<figcaption>` provides the accessible name for the `<figure>` by default
- The `role="group"` and `aria-labelledby` attributes are optional enhancements
- The image inside the figure must still have an `alt` attribute

**Constraints and Limitations**

- Not all screen readers handle figure captions identically
- The `role="group"` approach is recommended by some accessibility guidelines 

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Enhanced Figure Accessibility**

```html
<figure role="group" aria-labelledby="chart-caption">
    <img src="sales-chart.png" alt="Bar chart showing quarterly sales for 2026" width="600" height="400">
    <figcaption id="chart-caption">Figure 1: Quarterly sales data for 2026.</figcaption>
</figure>
```

**Expected Output**

Screen readers announce the figure as a group with the caption “Figure 1: Quarterly sales data for 2026” and the image’s `alt` text describing the chart.

**Why This Output Occurs**

The `role="group"` and `aria-labelledby` create an explicit programmatic relationship between the figure and its caption, enhancing screen reader support .

---

**Example 2: Figure with Caption as Accessible Name**

```html
<figure>
    <img src="elephant.jpg" alt="Elephant at sunset" width="400" height="300">
    <figcaption>An elephant at sunset</figcaption>
</figure>
```

**Expected Output**

The `<figcaption>` provides the accessible name for the `<figure>`. Screen readers may announce the figure with the caption as its name.

**Why This Output Occurs**

The `<figcaption>` is the first or last child of the `<figure>`, so it is mapped to the figure’s accessible name .

#### Real-World Cases

**Case 1: Government Accessibility Compliance**

Government websites use enhanced figure markup to meet WCAG 2.1 Success Criterion 1.1.1.

**Case 2: Screen Reader Testing**

Accessibility testers verify that figures and captions are announced correctly by screen readers.

**Case 3: Educational Content**

Educational platforms use figures with captions for accessible diagrams and illustrations.

---

## References

- MDN Web Docs – `<figure>`: The Figure with Optional Caption element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/figure
- MDN Web Docs – `<figcaption>`: The Figure Caption element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/figcaption
- WHATWG HTML Living Standard – The figure element – https://html.spec.whatwg.org/multipage/grouping-content.html#the-figure-element
- WHATWG HTML Living Standard – The figcaption element – https://html.spec.whatwg.org/multipage/grouping-content.html#the-figcaption-element
- W3C – HTML5: The figure element – https://dev.w3.org/html5/spec-author-view/grouping-content.html#the-figure-element
- W3C – HTML Accessibility API Mappings (HTML-AAM) – https://w3c.github.io/html-aam/
- W3C – WCAG 2.1 Understanding Success Criterion 1.1.1: Non-text Content – https://www.w3.org/WAI/WCAG21/Understanding/non-text-content.html
- WebAIM – Alternative Text – https://webaim.org/techniques/alttext/
- MDN Web Docs – ARIA: figure role – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles/figure_role