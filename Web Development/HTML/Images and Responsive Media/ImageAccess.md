# HTML Image Accessibility: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML image accessibility is the practice of providing textual alternatives for images so that users who cannot see them — including screen reader users, users with low vision, and users with images disabled — can understand the information or function conveyed by the image.

**Technical Definition**

Image accessibility is governed by WCAG Success Criterion 1.1.1 (Non-text Content, Level A), which requires that "all non-text content that is presented to the user has a text alternative that serves the equivalent purpose" with specific exceptions for controls, time-based media, tests, sensory experiences, CAPTCHA, and pure decoration. In HTML, the primary mechanism for providing text alternatives is the `alt` attribute on the `<img>` element. The W3C Images Tutorial identifies five primary categories of images requiring different text alternative strategies: informative images, decorative images, functional images, images of text, and complex images.

**Beginner-Friendly Explanation**

Not everyone can see images on a webpage. Some people use screen readers that read text aloud, some have images turned off to save data, and some simply cannot see. Image accessibility means writing a short text description (called "alt text") for every image so that everyone can understand what the image shows or does. The description you write depends on what the image is for: a photo needs a description of what it shows, a decorative image needs nothing, and a button image needs to describe what it does.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`alt` is required** | Every `<img>` element must have an `alt` attribute, even if its value is empty |
| **Context determines content** | The same image may require different alt text depending on its purpose on the page |
| **Decorative images are hidden** | Empty `alt=""` tells assistive technology to ignore purely decorative images |
| **Functional images describe actions** | Images used as links or buttons must describe the function, not the visual appearance |
| **Complex images need two-part alternatives** | Graphs and diagrams require a short `alt` plus a longer textual equivalent |
| **WCAG 1.1.1 is Level A** | This is a minimum conformance requirement for all websites |

---

### Prerequisites

- Basic familiarity with the `<img>` element and its attributes
- Understanding of HTML document structure
- Awareness of screen readers and assistive technology
- Basic knowledge of WCAG guidelines (helpful but not required)

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Image accessibility is a cornerstone of accessible web design
- **Semantic HTML** – The `alt` attribute is a semantic mechanism for conveying meaning
- **SEO** – Alt text is used by search engines to index image content
- **Responsive Images** – `srcset` and `<picture>` interact with alt text
- **ARIA** – `aria-describedby` and `aria-label` supplement alt text for complex cases

---

## Core Concepts / Features

---

### 1. Alternative Text (The `alt` Attribute)

#### Definitions

**Core Definition**

Alternative text (alt text) is a brief textual description of an image, provided via the `alt` attribute on the `<img>` element, that serves as a replacement for the image when it cannot be seen.

**Technical Definition**

The `alt` attribute defines text that can replace the image in the page. Browsers do not always display images — for example, in non-visual browsers, when the user chooses not to display images, or when the image is invalid. In these cases, the browser may replace the image with the text in the `alt` attribute. Setting the `alt` attribute to an empty string (`alt=""`) indicates that the image is not a key part of the content (it is decoration or a tracking pixel), and non-visual browsers may omit it from rendering. The `alt` attribute is also used when copying and pasting the image to text, or saving a linked image to a bookmark. According to MDN, the most important reason to use `alt` is to support accessibility, as the alt text may be used by screen readers.

**Beginner-Friendly Explanation**

Alt text is a short description you write for an image. It‘s what a screen reader reads aloud to someone who can’t see the image. It‘s also what shows up if the image fails to load. You put it in the `alt` attribute: `<img src="photo.jpg" alt="A sunset over the ocean">`. Every image needs one — even if it’s empty.

#### Purposes

- To provide a textual equivalent for screen reader users
- To display descriptive text when an image fails to load
- To enable users with speech input software to target images
- To improve SEO by giving search engines context about image content
- To satisfy WCAG Success Criterion 1.1.1

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="URL" alt="text alternative">
<img src="URL" alt="">  <!-- Decorative image -->
```

**Component Breakdown**

| Component | Description |
|---|---|
| `alt` | Attribute name |
| `"text alternative"` | A brief, descriptive text conveying the image‘s purpose or content |
| `""` | Empty string; indicates a decorative image |

**Syntax Rules**

- The `alt` attribute must be present on every `<img>` element
- The value must be appropriate for the image‘s purpose, not a description of the file
- For decorative images, use `alt=""`
- Alt text should be brief and concise — the W3C recommends keeping it short (e.g., around four-word length as a guideline)
- Do not include “image of” or “picture of” in the alt text

**Constraints and Limitations**

- Alt text cannot convey complex information like graphs or charts; use a two-part alternative for those
- The `title` attribute is not a substitute for `alt` and is not reliably announced by screen readers
- Overly long alt text is burdensome for screen reader users

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Alt Text**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Alt Text Demo</title>
</head>
<body>
    <!-- Informative image with descriptive alt text -->
    <img
        src="https://github.com/mdn/learning-area/blob/main/html/multimedia-and-embedding/tasks/images/images/blueberries.jpg?raw=true"
        alt="A pile of small blue berries"
        width="400"
        height="272"
    >
</body>
</html>
```

**Expected Output**

The image displays visually. When a screen reader encounters it, it announces “A pile of small blue berries, image.”

**Why This Output Occurs**

The `alt` attribute provides a textual equivalent of the image‘s content. The browser renders the image visually, and the screen reader uses the alt text to convey the image’s meaning to users who cannot see it.

---

**Example 2: Alt Text in a Functional Context**

```html
<p>
    <img src="search-icon.png" alt="Search" width="16" height="16">
    <input type="text" placeholder="Enter search term">
</p>
```

**Expected Output**

A small search icon appears next to a text input field. Screen readers announce “Search, image” followed by the input field.

**Why This Output Occurs**

The alt text “Search” describes the function of the icon, not its visual appearance. This helps screen reader users understand the purpose of the icon within the interface.

#### Real-World Cases

**Case 1: E-Commerce Product Images**

Product images use descriptive alt text: `alt="Red leather jacket with silver zipper and front pockets"`.

**Case 2: News Article Photographs**

News images use alt text that describes the scene: `alt="Protesters holding signs outside the city hall"`.

**Case 3: User Avatars**

Profile pictures use alt text like `alt="Profile picture of Jane Doe"`.

---

### 2. Decorative Images

#### Definitions

**Core Definition**

Decorative images are images that serve only a visual purpose — such as adding aesthetic appeal or separating content — and do not convey information that is essential to understanding the page.

**Technical Definition**

Decorative images are images that do not add information to the content of a page. Text values for these types of images would add audible clutter to screen reader output or could distract users if the topic is different from that in adjacent text. In these cases, a null (empty) alt text should be provided (`alt=""`) so that they can be ignored by assistive technologies, such as screen readers. The W3C recommends using CSS background images for decorative images rather than `<img>` elements.

**Beginner-Friendly Explanation**

A decorative image is just for looks — like a background pattern, a decorative divider, or a pretty photo that doesn‘t add any information. Since it doesn’t mean anything, you don‘t need to describe it. You tell screen readers to ignore it by using an empty `alt=""`.

#### Purposes

- To add visual appeal without conveying information
- To separate content sections visually
- To create atmosphere or mood
- To avoid cluttering screen reader output with unnecessary descriptions

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="decorative-pattern.png" alt="" width="100" height="100">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `alt=""` | Empty string; signals decorative image |
| `role="presentation"` | Optional; reinforces that the image is decorative |

**Syntax Rules**

- Use `alt=""` (empty string) for decorative images
- Do not omit the `alt` attribute entirely; that signals the image is informative but no alt text is available
- Consider using CSS background images instead of `<img>` for purely decorative images

**Constraints and Limitations**

- An empty `alt` attribute is not the same as a missing `alt` attribute
- Visual browsers hide the broken image icon when `alt=""` and the image fails to load
- Decorative images should not be given `role="presentation"` on `<img>` elements; the empty `alt` is sufficient

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Decorative Background Pattern**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Decorative Image Demo</title>
</head>
<body>
    <!-- Decorative divider image -->
    <img src="divider-pattern.png" alt="" width="600" height="20">

    <p>This is the main content of the page.</p>
</body>
</html>
```

**Expected Output**

The decorative divider displays visually, but screen readers skip it entirely — no announcement is made.

**Why This Output Occurs**

The empty `alt=""` tells assistive technology that the image is not a key part of the content. The browser still renders it visually, but the screen reader ignores it.

---

**Example 2: Decorative Icon in a Button**

```html
<button type="button">
    <img src="arrow-icon.png" alt="" width="16" height="16">
    Next
</button>
```

**Expected Output**

The button displays with an arrow icon and the text “Next.” Screen readers announce “Next, button” and ignore the decorative icon.

**Why This Output Occurs**

The icon is decorative because the button‘s text “Next” already describes its purpose. Giving the icon alt text would cause screen readers to announce redundant information.

#### Real-World Cases

**Case 1: Background Textures**

Websites use decorative textures as backgrounds. These should be CSS background images or have `alt=""`.

**Case 2: Decorative Icons in Lists**

Bulleted lists with custom decorative icons use `alt=""` to avoid cluttering screen reader output.

**Case 3: Spacer Images**

Legacy spacer images (used for layout) should have `alt=""` or be replaced with CSS.

---

### 3. Informative Images

#### Definitions

**Core Definition**

Informative images are images that graphically represent concepts and information — typically pictures, photos, and illustrations — and convey meaning that is essential to understanding the surrounding content.

**Technical Definition**

Informative images are used to graphically illustrate concepts and information, typically pictures and illustrations. The text alternative needs to be at least a short description conveying the essential information presented by the image. The alt text should convey the essential information that the image provides, but does not need to describe every visual detail — only what is relevant in the context of the page.

**Beginner-Friendly Explanation**

An informative image is a picture that actually tells you something. A photo in a news article, an illustration in a textbook, or a product image on a shopping site — these all convey information. You need to describe what the image shows so that people who can‘t see it still get the information.

#### Purposes

- To convey information visually that is essential to understanding the content
- To provide a textual equivalent for screen reader users
- To ensure that all users have access to the same information
- To support users who browse with images disabled

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="URL" alt="concise description of essential information">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `alt` | The text alternative |
| `"concise description"` | A brief text conveying the essential information of the image |

**Syntax Rules**

- The alt text should be a short description conveying the essential information
- Include the most important information first
- Avoid redundancy with adjacent text
- The exact level of detail depends on context — a bird identification website needs more detail than a general park website

**Constraints and Limitations**

- Alt text cannot convey all the information of a complex image; use complex-image techniques for graphs and charts
- Overly detailed alt text is burdensome; focus on what matters in context

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Informative Photograph**

```html
<img
    src="eiffel-tower.jpg"
    alt="The Eiffel Tower at sunset, with the Paris skyline in the background"
    width="600"
    height="400"
>
```

**Expected Output**

The photograph displays visually. Screen readers announce the alt text, conveying the scene.

**Why This Output Occurs**

The alt text provides a concise description of the image‘s essential content — the Eiffel Tower at sunset with the Paris skyline — enabling users who cannot see the image to understand what it depicts.

---

**Example 2: Informative Illustration**

```html
<p>
    The water cycle describes how water moves through the environment.
    <img
        src="water-cycle.png"
        alt="Diagram showing evaporation, condensation, precipitation, and collection"
        width="500"
        height="300"
    >
</p>
```

**Expected Output**

The diagram displays, and screen readers announce the alt text describing the water cycle stages.

**Why This Output Occurs**

The alt text identifies the key stages shown in the diagram, conveying the essential information without describing every visual detail.

#### Real-World Cases

**Case 1: News Article Images**

News images use alt text that describes the scene, such as `alt="Firefighters battling a blaze at a warehouse"`.

**Case 2: Recipe Photos**

Recipe images use alt text like `alt="Golden-brown chocolate chip cookies on a wire rack"`.

**Case 3: Travel Photography**

Travel images use alt text that captures the mood and key features: `alt="Sunset over the Grand Canyon with vibrant orange and purple sky"`.

---

### 4. Functional Images

#### Definitions

**Core Definition**

Functional images are images used as links, buttons, or other interactive controls, where the image‘s purpose is to initiate an action rather than to convey information.

**Technical Definition**

Functional images are images which perform an action, e.g., a button, an icon to open a Word document, or a shopping cart. The text alternative of an image used as a link or as a button should describe the functionality of the link or button rather than the visual image. Examples of such images are a printer icon to represent the print function or a button to submit a form. The W3C alt decision tree states that if an image is used in a link or button and it would be hard or impossible to understand what the link or button does without the image, use the `alt` attribute to communicate the destination of the link or action taken.

**Beginner-Friendly Explanation**

A functional image is one you click on to do something — like a button with a printer icon that prints the page, or a shopping cart icon that goes to your cart. The alt text should describe what the image *does*, not what it looks like. So for a printer icon, use `alt="Print this page"`, not `alt="Printer"`.

#### Purposes

- To describe the function or destination of an interactive image
- To enable screen reader users to understand what a button or link does
- To support speech input users who say “click [function]”
- To satisfy WCAG Success Criterion 1.1.1 for functional content

#### Syntax Rules and Structure

**General Syntax**

```html
<a href="URL"><img src="icon.png" alt="Function description"></a>
<button><img src="icon.png" alt="Action description"></button>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `alt` | The text alternative describing the function |
| `"Function description"` | A verb or action phrase describing what happens |

**Syntax Rules**

- Describe the function, not the visual appearance
- Use action words like “Search,” “Print,” “Submit,” “Go to cart”
- If the image is the only content of a link, the alt text serves as the link text
- If the link also has text, the image may be decorative (`alt=""`)

**Constraints and Limitations**

- If the link text already describes the function, the image inside may need `alt=""` to avoid redundancy
- Functional images inside buttons may need `alt=""` if the button has text

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Functional Image as Link**

```html
<a href="/search">
    <img src="search-icon.svg" alt="Search" width="24" height="24">
</a>
```

**Expected Output**

The search icon displays as a clickable link. Screen readers announce “Search, link” and pressing Enter navigates to the search page.

**Why This Output Occurs**

The alt text “Search” describes the function of the link. The image is the only content of the link, so its alt text serves as the link text.

---

**Example 2: Functional Image in a Button**

```html
<button type="button" onclick="window.print()">
    <img src="print-icon.svg" alt="Print this page" width="20" height="20">
</button>
```

**Expected Output**

A print button displays with a printer icon. Screen readers announce “Print this page, button.”

**Why This Output Occurs**

The alt text describes the action that the button performs — printing the page.

---

**Example 3: Redundant Functional Image**

```html
<a href="/cart">
    <img src="cart-icon.svg" alt="" width="20" height="20">
    View Cart
</a>
```

**Expected Output**

The cart icon and text “View Cart” display together. Screen readers announce “View Cart, link” and skip the icon.

**Why This Output Occurs**

The link text “View Cart” already describes the function. The icon is decorative in this context, so `alt=""` prevents redundant announcements.

#### Real-World Cases

**Case 1: Social Media Share Buttons**

Share buttons use functional alt text: `alt="Share on Twitter"`.

**Case 2: Navigation Icons**

Hamburger menu icons use `alt="Open menu"` or `alt="Menu"`.

**Case 3: Form Submit Buttons**

Submit buttons with icons use `alt="Submit form"` or `alt="Search"`.

---

### 5. Complex-Image Descriptions

#### Definitions

**Core Definition**

Complex-image descriptions provide a two-part text alternative for images that contain substantial information — such as graphs, charts, diagrams, and maps — consisting of a short `alt` text plus a longer textual equivalent.

**Technical Definition**

Complex images contain substantial information — more than can be conveyed in a short phrase or sentence. These are typically graphs and charts, including flow charts and organizational charts; diagrams and illustrations where the page text relies on the user being able to understand the image; and maps showing locations or other information. In these situations a two-part text alternative is required. The first part is the short description to identify the image and, where appropriate, indicate the location of the long description. The second part is the long description — a textual representation of the essential information conveyed by the image.

**Beginner-Friendly Explanation**

Some images contain a lot of information — like a chart showing sales data or a diagram of a process. A short description isn‘t enough. You need to provide a longer explanation somewhere on the page or in a linked document. The `alt` attribute gives a brief summary, and then the longer description is available nearby.

#### Purposes

- To provide a full textual equivalent of complex visual information
- To enable screen reader users to access the data presented in charts and graphs
- To satisfy WCAG 1.1.1 for complex non-text content
- To ensure that information is not lost when images cannot be seen

#### Syntax Rules and Structure

**Method 1: `longdesc` Attribute**

```html
<img src="chart.png"
     alt="Bar chart showing monthly visitors for Q1 2014"
     longdesc="chart-description.html">
```

**Method 2: `aria-describedby`**

```html
<img src="chart.png"
     alt="Bar chart showing monthly visitors for Q1 2014"
     aria-describedby="chart-desc">
<div id="chart-desc">
    <p>Detailed description of the chart data...</p>
</div>
```

**Method 3: Adjacent Text Description**

```html
<img src="chart.png" alt="Bar chart showing monthly visitors for Q1 2014">
<div>
    <h3>Chart Data: Monthly Visitors, Q1 2014</h3>
    <p>January: Site 1 had 500 visitors, Site 2 had 300, Site 3 had 200...</p>
</div>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `alt` | Short description identifying the image and its purpose |
| `longdesc` | URI of a page containing the long description |
| `aria-describedby` | ID of an element containing the long description |
| Adjacent text | A visible paragraph or section providing the full description |

**Syntax Rules**

- Provide a short `alt` that identifies the image and, where appropriate, indicates where the long description is located
- Provide the long description either on the same page or via a linked document
- The long description should include the essential information: scales, values, relationships, and trends
- Use `longdesc` for a separate page; use `aria-describedby` or adjacent text for same-page descriptions

**Constraints and Limitations**

- `longdesc` is not supported by all screen readers; `aria-describedby` has better support
- The long description should not simply repeat the alt text
- The description should be a textual representation of the information, not a visual description of the image

#### Annotated Complete Step-by-Step Code Examples

**Example 1: `longdesc` for a Separate Page**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Complex Image with longdesc</title>
</head>
<body>
    <h1>Quarterly Report</h1>

    <img src="q1-sales-chart.png"
         alt="Bar chart showing monthly and total visitors for the first quarter 2014 for sites 1 to 3"
         longdesc="q1-sales-description.html"
         width="600"
         height="400">

    <p>The chart above shows visitor statistics for three websites.</p>
</body>
</html>
```

**Expected Output**

The chart displays visually. Screen readers announce the short alt text and provide a way to access the linked long description.

**Why This Output Occurs**

The `longdesc` attribute provides a URI to a page containing the full textual description. The short alt identifies the image; the long description provides the data.

---

**Example 2: `aria-describedby` for Same-Page Description**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Complex Image with aria-describedby</title>
</head>
<body>
    <h1>Website Traffic Analysis</h1>

    <img src="traffic-chart.png"
         alt="Line chart showing daily traffic for January 2026"
         aria-describedby="traffic-description"
         width="600"
         height="400">

    <div id="traffic-description">
        <h2>Traffic Data: January 2026</h2>
        <p>Daily traffic ranged from 1,200 visitors on January 1st to a peak
        of 5,800 visitors on January 15th. Traffic declined steadily after
        the 15th, ending the month at approximately 2,100 visitors per day.</p>
    </div>
</body>
</html>
```

**Expected Output**

The chart displays, and screen readers announce the alt text followed by the long description from the referenced element.

**Why This Output Occurs**

The `aria-describedby` attribute creates a programmatic association between the image and the long description. Screen readers announce the alt text and then the description content.

---

**Example 3: Adjacent Text Description**

```html
<figure>
    <img src="organizational-chart.png"
         alt="Organizational chart for the company"
         width="600" height="400">
    <figcaption>
        <h2>Company Organizational Chart</h2>
        <p>The chart shows the CEO at the top, with three direct reports:
        the CFO, the CTO, and the COO. The CFO oversees Finance and Accounting.
        The CTO oversees Engineering and Product. The COO oversees Operations
        and Human Resources.</p>
    </figcaption>
</figure>
```

**Expected Output**

The chart displays with a visible caption containing the full description. Screen readers announce the alt text and then the caption content.

**Why This Output Occurs**

The `<figure>` and `<figcaption>` elements create a semantic association between the image and its description. The description is visible on the page and programmatically associated with the image.

#### Real-World Cases

**Case 1: Financial Reports**

Bar charts and line graphs in financial reports use `aria-describedby` or `longdesc` to provide the underlying data.

**Case 2: Scientific Papers**

Diagrams and flow charts in scientific papers use adjacent text descriptions or linked long descriptions.

**Case 3: Government Data Visualizations**

Government websites use `aria-describedby` to provide accessible alternatives for data visualizations.

---

### 6. Choosing the Right Approach

#### Definitions

**Core Definition**

Choosing the right approach for image accessibility means selecting the appropriate alt text strategy — empty, descriptive, functional, or complex — based on the image‘s purpose in context.

**Technical Definition**

The W3C alt decision tree provides a systematic method for determining the appropriate `alt` attribute value based on the image’s role: whether it contains text, is used in a link or button, contributes meaning to the page, or is purely decorative. The decision depends on the image‘s purpose, not its visual appearance.

**Beginner-Friendly Explanation**

Not every image needs the same kind of description. Ask yourself: What is this image for? If it’s just decoration, use `alt=""`. If it shows information, describe it. If it‘s a button, describe what it does. If it’s a chart, provide a longer description. The W3C decision tree helps you decide.

#### Purposes

- To ensure the correct alt text strategy for each image
- To avoid common accessibility mistakes
- To provide consistent, appropriate text alternatives across a website
- To satisfy WCAG 1.1.1 for all image types

#### Syntax Rules and Structure

**Decision Tree Summary**

| Question | If Yes | If No |
|---|---|---|
| Does the image contain text? | Depends on context (see below) | Continue |
| Is the image used in a link/button and needs to convey function? | Use functional alt text | Continue |
| Does the image contribute meaning? | Is it simple? Use informative alt. Is it complex? Use two-part alt | Continue |
| Is the image purely decorative? | Use `alt=""` | Review context |

**Syntax Rules**

- Use empty `alt=""` for decorative images and redundant images
- Use descriptive alt text for informative images
- Use action-oriented alt text for functional images
- Use two-part alternatives for complex images
- Always consider context; the same image may need different alt text on different pages

**Constraints and Limitations**

- The decision tree does not cover every possible case
- Context is critical; there is no one-size-fits-all alt text
- When in doubt, consult the W3C Images Tutorial for detailed guidance

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Applying the Decision Tree**

```html
<!-- Decorative divider: no meaning -->
<img src="divider.png" alt="" width="600" height="20">

<!-- Informative photo: conveys information -->
<img src="mountain.jpg" alt="Snow-capped mountain peak at sunrise" width="600" height="400">

<!-- Functional icon: initiates action -->
<a href="/print"><img src="printer.png" alt="Print this page" width="24" height="24"></a>

<!-- Complex chart: needs two-part alternative -->
<img src="sales-chart.png"
     alt="Bar chart showing quarterly sales figures"
     aria-describedby="sales-data">
<div id="sales-data">
    <p>Q1: $1.2M, Q2: $1.5M, Q3: $1.8M, Q4: $2.1M</p>
</div>
```

**Expected Output**

Each image receives appropriate treatment based on its purpose.

**Why This Output Occurs**

The decorative divider uses `alt=""`. The photo uses descriptive alt text. The print icon uses functional alt text. The chart uses a short alt plus a long description.

#### Real-World Cases

**Case 1: Government Websites**

Government accessibility guidelines require applying the decision tree to every image on a page.

**Case 2: Content Management Systems**

CMS platforms provide alt text fields and guidance based on image purpose.

**Case 3: Design Systems**

Design systems document alt text conventions for common image patterns.

---

## References

- W3C – Understanding Success Criterion 1.1.1: Non-text Content – https://w3c.github.io/wcag/understanding/non-text-content
- W3C – Images Tutorial – https://raw.githubusercontent.com/w3c/wai-tutorial-images/master/content/index.md
- W3C – Decorative Images – https://www.w3.org/WAI/tutorials/images/decorative/
- W3C – Informative Images – https://www.w3.org/WAI/tutorials/images/informative/
- W3C – Functional Images – https://www.w3.org/WAI/tutorials/images/functional/
- W3C – Complex Images – https://www.w3.org/WAI/tutorials/images/complex/
- W3C – An alt Decision Tree – https://www.w3.org/WAI/tutorials/images/decision-tree/
- MDN Web Docs – `<img>`: The Image Embed element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img
- MDN Web Docs – HTMLImageElement: alt property – https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/alt
- MDN Web Docs – HTML images – https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Images_in_HTML
- WebAIM – Alternative Text – https://webaim.org/techniques/alttext/
- W3C – WCAG 2.1 Understanding Success Criterion 1.1.1: Non-text Content – https://www.w3.org/WAI/WCAG21/Understanding/non-text-content.html
- W3C – H37: Using alt attributes on img elements – https://www.w3.org/WAI/WCAG21/Techniques/html/H37
- W3C – G95: Providing short text alternatives that provide a brief description of the non-text content – https://www.w3.org/WAI/WCAG21/Techniques/general/G95