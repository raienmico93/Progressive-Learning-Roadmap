# Image Accessibility — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Image accessibility is the practice of providing text alternatives for images so that people who cannot see them — including screen-reader users, people with low vision, and users in low-bandwidth environments — can understand the information or function that the image conveys. It is governed primarily by WCAG Success Criterion 1.1.1 (Non-text Content, Level A), which requires that all non-text content presented to the user has a text alternative that serves the equivalent purpose.

**Technical Definition**
In the WHATWG HTML Living Standard and WCAG 2.2, image accessibility is implemented through the `alt` attribute on `<img>` elements, supplemented by ARIA attributes (`role="presentation"`, `aria-label`, `aria-labelledby`) and structural techniques (`<figure>` with `<figcaption>`, adjacent text descriptions, and links to long descriptions) for complex images. The W3C Images Tutorial defines a taxonomy of image purposes — **informative**, **decorative**, **functional**, **images of text**, **complex**, **groups of images**, and **image maps** — each with its own alt-text strategy. The `alt` attribute's value must convey the **meaning** of the image, not necessarily describe its visual characteristics.

**Beginner-Friendly Explanation**
Image accessibility means making sure every picture on your website has a text description that tells people who cannot see it what the picture is for. If the picture shows a product, the description says what the product is. If the picture is just decoration, you mark it so screen readers skip it. If the picture is a button, the description says what the button does. The goal is simple: no one should miss information or functionality just because they cannot see an image.

---

### Key Characteristics

- **Purpose-driven**: The alt text depends on the **purpose** of the image, not its visual content. A logo used as a link needs the link destination; a logo used decoratively needs nothing.
- **`alt` is mandatory (with narrow exceptions)**: Every meaningful `<img>` must have an `alt` attribute; decorative images must have `alt=""`, not a missing attribute.
- **Missing `alt` is worse than empty `alt`**: Screen readers may announce the filename or URL when `alt` is absent, which is useless and confusing.
- **Complex images require two parts**: A short `alt` to identify the image, plus a long description (adjacent text, a linked page, or a `<figure>` with a table) to convey the essential data.
- **Functional images describe actions, not objects**: A magnifying-glass icon that triggers search should say “Search,” not “magnifying glass”.
- **Images of text should be avoided**: Real text is more accessible, scalable, and localisable; when unavoidable, `alt` must contain the same words as the image.

---

### Prerequisites

1. **Basic HTML image syntax**: familiarity with `<img src alt>`.
2. **WCAG 2.2 awareness**: understanding of Success Criterion 1.1.1 (Non-text Content, Level A).
3. **Screen-reader basics**: awareness of how assistive technologies announce images and `alt` text.
4. **The W3C alt Decision Tree**: familiarity with the decision process for categorising image purposes.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **WCAG conformance** | 1.1.1 (Non-text Content) is the foundational criterion; other criteria (1.4.3 Contrast, 1.4.5 Images of Text) also apply. |
| **SEO** | Descriptive `alt` text makes images indexable and improves image search visibility. |
| **Speech input** | `alt` text enables voice-control users to activate buttons and links by name. |
| **Mobile and low-bandwidth** | `alt` text provides information when images are turned off or fail to load. |
| **Internationalisation** | Alt text should be written in the language of the surrounding content (`lang` attribute). |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Alt text** — the text alternative for images
2. **Decorative images** — images that add no information
3. **Functional images** — images that trigger actions
4. **Complex images** — images that convey substantial data
5. **Image accessibility best practices** — consolidated guidelines

---

## 1. Alt Text — The Text Alternative

### Definitions

**Core Definition**
Alt text (alternative text) is the value of the `alt` attribute on an `<img>` element. It provides a textual replacement for the image that screen readers announce and that browsers display when the image cannot load.

**Technical Definition**
Per WCAG 2.2 SC 1.1.1, every `<img>` element that conveys meaning must have an `alt` attribute whose value serves the **equivalent purpose** of the image. Technique H37 specifies that the alt text should be a **short text alternative** and that when the image contains words important to understanding the content, those words must be included in the alt text. The alt text does **not** need to describe the visual characteristics of the image; it must convey the same **meaning**. The value should be concise, neutral, accurate, and free of redundancy.

**Beginner-Friendly Explanation**
Alt text is the words you put in the `alt` attribute to describe the picture. It is what a screen reader reads aloud. It should tell the listener what the picture is **for**, not necessarily what it looks like. Keep it short — a sentence or less for most images.

---

### Purposes

- **To provide an accessible name** for the image to screen readers and other assistive technologies.
- **To display fallback text** when the image fails to load or is turned off.
- **To enable speech-input users** to identify and activate image-based links and buttons by name.
- **To make images indexable by search engines**.
- **To satisfy WCAG 1.1.1 (Non-text Content, Level A)**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="image.jpg" alt="Concise description of the image's purpose">
```

**Component breakdown**

| Component | Description |
|---|---|
| `src` | The image URL. |
| `alt` | **Required** for meaningful images. A short text alternative conveying the image's purpose. |

#### Syntax Rules (Six Core Rules)

1. **Describe the purpose, not the picture**: Say what the image communicates or does, not a literal visual inventory.
2. **Skip “image of” and “picture of”**: Screen readers already announce “image”; these phrases add noise.
3. **Keep it to a sentence or so**: Be concise; long alt text is rarely read in full.
4. **Do not repeat the caption**: If a visible caption already describes the image, the alt text should add new information or be empty.
5. **Include important words in the image**: If the image contains text essential to understanding, include those words in the alt text.
6. **Never use the filename**: Filenames like `IMG_1234.jpg` are meaningless to users.

#### Constraints and Limitations

- **Missing `alt` is not the same as empty `alt`**: A missing attribute may cause screen readers to announce the filename or URL.
- **Alt text cannot be empty for meaningful images**: Empty `alt` is reserved for decorative images only.
- **Alt text cannot contain markup**: It is plain text.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Meaningful Alt Text for an Informative Image

**Setup Guide**: Save as `alt-informative.html` and test with a screen reader.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Informative Image Alt Text</title>
</head>
<body>
  <p>
    Our new office features
    <img src="office.jpg" alt="An open-plan office with standing desks and large windows">
    and plenty of natural light.
  </p>
</body>
</html>
```

**Expected output**: A screen reader announces “An open-plan office with standing desks and large windows,” conveying the essential information about the office.

**Why this output occurs**: The `alt` text describes the **purpose** of the image (showing the office environment) in a single, descriptive sentence, following the six core rules.

---

#### Example 2: Alt Text That Includes Image Text

**Setup Guide**: Save as `alt-image-text.html`. This image contains the words “Free newsletter. Get free recipes, news, and more. Learn more.”

```html
<img src="newsletter.gif" alt="Free newsletter. Get free recipes, news, and more. Learn more.">
```

**Expected output**: The screen reader announces the full text content of the image, preserving the information the image was designed to convey.

**Why this output occurs**: Technique H37 requires that when an image contains words important to understanding the content, those words are included in the alt text.

---

#### Example 3: Alt Text Without “Image of”

**Setup Guide**: Save as `alt-no-image-of.html`.

```html
<!-- GOOD: describes the purpose directly -->
<img src="chart.png" alt="Revenue increased 20% from Q1 to Q4">

<!-- BAD: redundant "image of" prefix -->
<!-- <img src="chart.png" alt="Image of a chart showing revenue increased 20% from Q1 to Q4"> -->
```

**Expected output**: The good version announces the meaningful data; the bad version wastes the listener’s time with “Image of.”

**Why this output occurs**: The rule “Skip image of and picture of” reduces audible clutter, because screen readers already announce the element as an image.

---

### Real-World Cases

1. **News articles**: Every content image carries alt text describing the scene or the point the image illustrates.
2. **E-commerce**: Product images use alt text naming the product and its key visible attributes.
3. **Documentation**: Screenshots use alt text identifying the interface and the task being demonstrated.

---

### References

- W3C — Understanding SC 1.1.1: Non-text Content — https://w3c.github.io/wcag/understanding/non-text-content.html
- W3C — Technique H37: Using alt attributes on img elements — https://w3c.github.io/wcag/techniques/html/H37
- W3C — Images Tutorial — https://w3c.github.io/wai-website/tutorials/images/
- W3C — Tips and Tricks: Choosing appropriate text alternatives — https://www.w3.org/WAI/tips/writing/

---

## 2. Decorative Images — Images That Add No Information

### Definitions

**Core Definition**
A decorative image is an image that adds no information to the content of the page. It may be purely visual styling, “eye candy,” or supplementary to adjacent text that already conveys the information. Decorative images must be marked with `alt=""` (a null alt) so assistive technologies skip them.

**Technical Definition**
WCAG 2.2 SC 1.1.1 explicitly exempts pure decoration: if non-text content is pure decoration, used only for visual formatting, or not presented to users, it is implemented in a way that assistive technology can ignore. Techniques H67 and F39 define the correct implementation: `alt=""` with no `title` attribute allows AT to ignore the image, while a non-null alt like `alt="spacer"` or `alt="image"` is a failure. The judgment of whether an image is decorative belongs to the author, based on the reason for including it.

**Beginner-Friendly Explanation**
A decorative image is a picture that does not tell the reader anything they need to know. It might be a background flourish, a dividing line, or a photo that just makes the page look nice. You mark these with an empty `alt=""` so screen readers skip them entirely. If you leave the alt attribute out entirely, screen readers might read the filename, which is annoying and useless.

---

### Purposes

- **To prevent screen readers from announcing meaningless content** that would add audible clutter.
- **To satisfy WCAG 1.1.1** by implementing pure decoration in a way that AT can ignore.
- **To improve the listening experience** for screen-reader users by removing noise.
- **To distinguish decorative images from informative images** in an accessibility audit.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="decorative.png" alt="">
```

**Component breakdown**

| Component | Description |
|---|---|
| `alt=""` | A null (empty) alt attribute. |
| No `title` | The `title` attribute must be absent or empty. |
| Optional: `role="presentation"` | An alternative way to signal that the image should be ignored. |

#### Syntax Rules

1. **Use `alt=""` (empty string)**, not a missing `alt` attribute.
2. **Do not use a descriptive alt** like `alt="spacer"` or `alt="divider"` — this is a failure of SC 1.1.1.
3. **Do not use a `title` attribute** on decorative images; it can override the null alt.
4. **`role="presentation"` or `role="none"` may be used** as an alternative or supplement to `alt=""`.

#### Constraints and Limitations

- **Leaving `alt` off entirely is not acceptable**: Some screen readers will announce the filename.
- **Decorative vs. informative is a judgment call**: The author must decide whether the image contributes information.
- **Overusing `alt=""`** can hide images that actually convey information; do not use it as a shortcut.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Decorative Divider with `alt=""`

**Setup Guide**: Save as `decorative-divider.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Decorative Divider</title>
</head>
<body>
  <p>
    <img src="divider.png" alt="">
    Welcome to our site.
  </p>
</body>
</html>
```

**Expected output**: The decorative divider is completely ignored by screen readers; only “Welcome to our site” is announced.

**Why this output occurs**: The empty `alt=""` signals to assistive technology that the image can be safely skipped.

---

#### Example 2: Failure — Non-Null Alt on a Decorative Image

**Setup Guide**: Save as `decorative-failure.html`. An accessibility checker will flag this.

```html
<!-- FAILURE: alt="spacer" is announced by screen readers -->
<div>
  Tree type: <img src="spacer.gif" width="100" height="1" alt="spacer"> Cedrus deodara
</div>
```

**Expected output**: A screen reader announces “spacer” between “Tree type:” and “Cedrus deodara,” adding meaningless noise.

**Why this output occurs**: Technique F39 defines this as a failure: the image is meant to be ignored, but its non-null alt text is announced.

---

#### Example 3: Decorative Image with `role="none"`

**Setup Guide**: Save as `decorative-role.html`.

```html
<p>Happy new year!</p>
<img src="fireworks.jpg" role="none" alt="">
```

**Expected output**: The fireworks image is ignored by assistive technologies.

**Why this output occurs**: The explicit `role="none"` (or `role="presentation"`) combined with `alt=""` tells AT to skip the image.

---

### Real-World Cases

1. **Design flourishes**: Background patterns, corner ornaments, and decorative borders.
2. **Spacing elements**: Images used to create blank space (though CSS is preferable).
3. **Supplementary “eye candy”**: Photos that make a page attractive but add no information beyond the adjacent text.

---

### References

- W3C — Method: Decorative Images — https://www.w3.org/WAI/GL/WCAG3/2022/methods/decorative-images/
- W3C — Technique H67: Using null alt text — https://www.w3.org/WAI/WCAG21/Techniques/html/H67
- W3C — Technique F39: Failure due to non-null alt — https://www.w3.org/WAI/WCAG22/Techniques/failures/F39
- W3C — Technique F38: Failure due to not marking up decorative images — https://w3c.github.io/wcag/techniques/failures/F38

---

## 3. Functional Images — Images That Trigger Actions

### Definitions

**Core Definition**
A functional image is an image used as a link, button, or other interactive control. Its text alternative must describe the **action** that the control performs, not the visual appearance of the image.

**Technical Definition**
The W3C Images Tutorial defines functional images as images used as links or buttons, where “the text alternative of an image used as a link or as a button should describe the functionality of the link or button rather than the visual image”. The text alternative should convey the action — “print this page” rather than “printer,” “search” rather than “magnifying lens,” “W3C home” rather than “W3C logo”. When a link or button contains one or more images and no text, the alt text must together convey the purpose of the link or button.

**Beginner-Friendly Explanation**
A functional image is a picture you click to do something. It might be a search icon, a print button, or a logo that links to the homepage. The alt text should say what **happens** when you click it — “Search,” “Print this page,” “Go to homepage” — not what the picture looks like.

---

### Purposes

- **To communicate the action** of an image-based control to screen-reader users.
- **To enable speech-input users** to activate controls by speaking their name.
- **To satisfy WCAG 1.1.1** for non-text content that functions as a control.
- **To prevent screen readers from announcing the filename or URL** of a functional image.

---

### Syntax Rules and Structure

#### Complete General Syntax (Image Link)

```html
<a href="https://www.w3.org/">
  <img src="w3c.png" alt="W3C home">
</a>
```

#### Complete General Syntax (Image Button)

```html
<button>
  <img src="magnifier.svg" alt="Search">
</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<a>` or `<button>` | The interactive control. |
| `<img>` | The functional image. |
| `alt` | Describes the **action**, not the object. |

#### Syntax Rules

1. **Describe the action**: “Search,” “Print,” “Download,” “W3C home” — not “magnifying glass,” “printer,” or “logo”.
2. **If the image is the only content of the link/button**, its alt text is the accessible name.
3. **If the link has both text and an image**, the image may have `alt=""` if the text already describes the action.
4. **Do not use the filename or URL as alt text**; this is unhelpful to users.

#### Constraints and Limitations

- **Missing or empty alt on a functional image is a severe failure**: The control becomes unusable to screen-reader users.
- **The alt text must match the visible label** if one is present, to avoid mismatches for sighted and screen-reader users.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Linked Logo with Descriptive Alt

**Setup Guide**: Save as `functional-logo.html`. The image is the only content of a link to the W3C homepage.

```html
<a href="https://www.w3.org/">
  <img src="w3c.png" alt="W3C home">
</a>
```

**Expected output**: The link is announced as “W3C home, link,” telling the user where the link goes.

**Why this output occurs**: The alt text describes the **function** of the image (navigating to the homepage), not the visual content (“W3C logo”).

---

#### Example 2: Search Button with Icon

**Setup Guide**: Save as `functional-search.html`.

```html
<button type="button">
  <img src="magnifier.svg" alt="Search">
</button>
```

**Expected output**: The button is announced as “Search, button,” enabling the user to activate it.

**Why this output occurs**: The alt text describes the action (“Search”), not the visual object (“magnifying glass”).

---

#### Example 3: Icon with Additional Link Text

**Setup Guide**: Save as `functional-icon-text.html`. The icon conveys “opens in new window” alongside the link text.

```html
<a href="https://www.w3.org/" target="_blank">
  W3C Homepage
  <img src="external.svg" alt="Opens in new window">
</a>
```

**Expected output**: The link is announced as “W3C Homepage, Opens in new window, link,” conveying both the destination and the behaviour.

**Why this output occurs**: The icon’s alt text adds essential information (new-window behaviour) that the link text alone does not convey.

---

### Real-World Cases

1. **Social media icons**: Each icon links to a profile; alt text should say “Visit our Instagram page,” not “Instagram logo”.
2. **Navigation icons**: A home icon should say “Home,” not “house.”
3. **Action buttons**: Print, download, share, and search icons should describe their actions.

---

### References

- W3C — Method: Functional Images — https://www.w3.org/WAI/GL/WCAG3/2020/methods/functional-images/
- W3C — Images Tutorial: Functional Images — https://www.w3.org/WAI/tutorials/images/functional/
- W3C — Technique H37: Using alt attributes — https://w3c.github.io/wcag/techniques/html/H37

---

## 4. Complex Images — Images That Convey Substantial Data

### Definitions

**Core Definition**
A complex image is an image that contains substantial information — more than can be conveyed in a short phrase or sentence. Examples include graphs, charts, diagrams, flowcharts, and illustrations that the page text depends on for understanding.

**Technical Definition**
The W3C defines complex images as those requiring a **two-part text alternative**: first, a **short description** (in the `alt` attribute) to identify the image and, where appropriate, indicate the location of the second part; and second, a **long description** that provides a textual representation of the essential information conveyed by the image. The long description must include scales, values, relationships, and trends for data visualisations. Several approaches are recommended: a link to a separate long-description page, a text link adjacent to the image, a `<figure>` element enclosing both the image and a data table, or an `alt` attribute that points to the location of the long description.

**Beginner-Friendly Explanation**
A complex image is a picture that contains a lot of information — like a chart, a graph, or a detailed diagram. You cannot fit all that information into a short alt text. So you provide two things: a short alt that says what the image is and where to find more, and a longer description — either nearby, on a linked page, or in a table — that gives all the important data.

---

### Purposes

- **To convey the essential data** of charts, graphs, and diagrams to users who cannot see them.
- **To satisfy WCAG 1.1.1** for non-text content that conveys substantial information.
- **To provide equal access to data-driven content** in reports, dashboards, and educational materials.
- **To support users who prefer text** or who are using low-bandwidth connections.

---

### Syntax Rules and Structure

#### Complete General Syntax (Short Alt + Adjacent Long Description)

```html
<p>
  <img src="chart.png" alt="2013 first quarter example.com website visitors bar chart">
  <br>
  <a href="2013first_qtr_text.html">Example.com Site visitors Jan to March 2013 text description of the bar chart</a>
</p>
```

#### Complete General Syntax (Figure with Data Table)

```html
<figure role="group">
  <img src="chart.png" alt="2013 first quarter example.com website visitors bar chart">
  <table>
    <caption>Example.com Site visitors Jan to March 2013</caption>
    <tr><th>Month</th><th>Visitors</th></tr>
    <tr><td>January</td><td>12,000</td></tr>
    <tr><td>February</td><td>15,000</td></tr>
    <tr><td>March</td><td>18,000</td></tr>
  </table>
</figure>
```

**Component breakdown**

| Component | Description |
|---|---|
| `alt` | Short description identifying the image. |
| Long description | Adjacent text, linked page, or data table providing the essential data. |
| `<figure role="group">` | Structurally associates the image with its long description. |

#### Syntax Rules

1. **The short alt identifies the image** and, where appropriate, points to the long description.
2. **The long description provides the essential information** — values, trends, relationships, and scales.
3. **Use `<figure>` with a data table** for charts generated from tabular data.
4. **Make the long description available to all users**, not just screen-reader users.
5. **The long description must serve the same purpose** as the image; otherwise it fails SC 1.1.1.

#### Constraints and Limitations

- **A short alt alone is insufficient** for complex images; it cannot convey the data.
- **The `longdesc` attribute has limited browser support**; a regular link or `<figure>` is more reliable.
- **The long description must be accurate and complete**; an inadequate description is a failure technique.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Chart with Adjacent Text Link

**Setup Guide**: Save as `complex-chart.html`. Provide a chart image and a linked description page.

```html
<p>
  <img src="chart.png" alt="2013 first quarter example.com website visitors bar chart">
  <br>
  <a href="2013first_qtr_text.html">
    Example.com Site visitors Jan to March 2013 text description of the bar chart
  </a>
</p>
```

**Expected output**: The screen reader announces the short alt identifying the chart, and the user can follow the link to read the full text description.

**Why this output occurs**: The two-part approach provides a short identifier and a path to the complete data, satisfying the complex-image requirement.

---

#### Example 2: Chart with Inline Data Table in `<figure>`

**Setup Guide**: Save as `complex-figure.html`.

```html
<figure role="group">
  <img src="chart.png" alt="2013 first quarter example.com website visitors bar chart">
  <table>
    <caption>Example.com Site visitors Jan to March 2013</caption>
    <thead>
      <tr><th>Month</th><th>Visitors</th></tr>
    </thead>
    <tbody>
      <tr><td>January</td><td>12,000</td></tr>
      <tr><td>February</td><td>15,000</td></tr>
      <tr><td>March</td><td>18,000</td></tr>
    </tbody>
  </table>
</figure>
```

**Expected output**: The screen reader announces the image and the associated data table, giving the user direct access to the underlying numbers.

**Why this output occurs**: The `<figure>` element with `role="group"` structurally associates the image with its data table, providing the long description inline.

---

#### Example 3: Alt Text Pointing to On-Page Description

**Setup Guide**: Save as `complex-pointer.html`.

```html
<p>
  <img src="chart.png" alt="2013 first quarter example.com website visitors bar chart. Described under the heading Site visitors full text">
</p>
<h4>Site visitors full text</h4>
<p>January: 12,000 visitors. February: 15,000. March: 18,000.</p>
```

**Expected output**: The alt text tells the user what the image is and where to find the full description on the same page.

**Why this output occurs**: This approach is acceptable when the location of the long description can be accurately pinpointed by description.

---

### Real-World Cases

1. **Financial reports**: Bar charts and line graphs with data tables as long descriptions.
2. **Scientific papers**: Diagrams and flowcharts with detailed textual explanations.
3. **Dashboards**: Interactive charts with accessible data summaries.

---

### References

- W3C — Complex Images (Draft) — https://www.w3.org/WAI/EO/Drafts/app-notes/images/complex
- W3C — Technique G73: Linking complex images to a long description — https://wcagrules.com/techniques/G73
- W3C — Technique F67: Failure due to inadequate long descriptions — https://www.w3.org/WAI/WCAG22/Techniques/failures/F67
- W3C — Images Tutorial: Complex Images — https://www.w3.org/WAI/tutorials/images/complex/

---

## 5. Image Accessibility Best Practices

### Definitions

**Core Definition**
Image accessibility best practices are the consolidated guidelines for providing appropriate text alternatives for every image on a page — covering informative, decorative, functional, complex, and text-containing images — so that no information or functionality is lost to users who cannot see the image.

**Technical Definition**
Best practices are informed by WCAG 2.2 SC 1.1.1, the W3C Images Tutorial, and the alt Decision Tree. They encompass purpose-driven alt text, null alt for decorative images, action-oriented alt for functional images, two-part descriptions for complex images, avoidance of images of text, and colour-contrast requirements for graphical information.

**Beginner-Friendly Explanation**
Best practices are the habits that make every image accessible: describe what the image is for, skip purely decorative images, describe what functional images do, give complex images a full text description somewhere, and avoid putting important text inside images.

---

### Purposes

- **To satisfy WCAG 1.1.1** across all image types.
- **To ensure no information is lost** for screen-reader, low-vision, or low-bandwidth users.
- **To improve usability for speech-input and mobile users**.
- **To support SEO** through descriptive alt text.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Provide `alt` on every meaningful image** | Mandatory for informative, functional, and complex images. |
| **Use `alt=""` for decorative images** | Never omit the attribute; use an empty string. |
| **Describe the purpose, not the picture** | Focus on what the image communicates or does. |
| **Skip “image of” and “picture of”** | Screen readers already announce the element type. |
| **Include important image text in `alt`** | If the image contains words essential to understanding, include them. |
| **Use action-oriented alt for functional images** | “Search,” “Print,” “W3C home” — not the visual object. |
| **Provide two-part descriptions for complex images** | Short alt + long description (adjacent, linked, or in a `<figure>`). |
| **Avoid images of text** | Use real text; when unavoidable, alt text must match the words in the image. |
| **Ensure 3:1 contrast for graphical information** | Lines, icons, and chart elements that convey meaning must meet contrast requirements. |

#### Syntax Rules

1. **Purpose first**: Alt text reflects the image's purpose in context, not its visual content.
2. **Null alt for decoration**: Empty string, not a missing attribute.
3. **Action-oriented for controls**: Functional images describe what happens, not what is pictured.
4. **Two-part for data**: Complex images need a short identifier and a long description.
5. **Real text over text images**: Avoid embedding essential text in images.

#### Constraints and Limitations

- **Alt text cannot convey complex data**: Charts and graphs require long descriptions beyond the alt attribute.
- **Decorative vs. informative is a judgment call**: The author must decide, and can err in either direction.
- **Colour contrast for graphics**: Meaningful graphical elements need sufficient contrast, not just text.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Page with All Image Types

**Setup Guide**: Save as `best-practice-images.html`. Test with a screen reader and an accessibility checker.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Images</title>
</head>
<body>
  <!-- Informative image -->
  <img src="team.jpg" alt="The customer support team smiling at their desks" width="800" height="600">

  <!-- Decorative image -->
  <img src="divider.png" alt="">

  <!-- Functional image -->
  <a href="/donate">
    <img src="donate-button.png" alt="Donate now">
  </a>

  <!-- Complex image with adjacent long description -->
  <img src="revenue-chart.png" alt="Revenue bar chart for Q1–Q4 2025">
  <a href="revenue-description.html">Full text description of the revenue chart</a>

  <!-- Image of text (avoid if possible) -->
  <img src="logo.png" alt="Example Company">
</body>
</html>
```

**Expected output**: A screen reader announces the team photo, skips the divider, announces the “Donate now” link, announces the revenue chart with its description link, and announces the company logo.

**Why this output occurs**: Each image is marked up according to its purpose: informative images get descriptive alt, decorative images get `alt=""`, functional images get action-oriented alt, complex images get a short alt plus a long-description link, and the logo gets its text equivalent.

---

#### Example 2: Common Failures

**Setup Guide**: Save as `image-failures.html`. An accessibility checker will flag each issue.

```html
<!-- FAILURE: missing alt -->
<img src="photo.jpg">

<!-- FAILURE: non-null alt on decorative image -->
<img src="spacer.gif" alt="spacer">

<!-- FAILURE: filename as alt -->
<img src="IMG_1234.jpg" alt="IMG_1234.jpg">

<!-- FAILURE: functional image without alt -->
<button><img src="search.svg"></button>

<!-- FAILURE: complex image with only a short alt -->
<img src="chart.png" alt="Chart">
```

**Expected output**: Each failure degrades accessibility: missing alt may cause the filename to be announced, non-null alt on decorative images adds noise, filenames are meaningless, functional images become unusable, and complex images lose their data.

**Why this output occurs**: These are the most common image accessibility failures identified by WCAG Techniques F39, F38, and H37.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.1.1 compliance is legally required; image accessibility is audited.
2. **E-commerce**: Product images with descriptive alt text improve both accessibility and SEO.
3. **News and media**: Article images require meaningful alt text and, for charts, long descriptions.

---

### References

- W3C — Understanding SC 1.1.1: Non-text Content — https://w3c.github.io/wcag/understanding/non-text-content.html
- W3C — Images Tutorial — https://w3c.github.io/wai-website/tutorials/images/
- W3C — alt Decision Tree — https://www.w3.org/WAI/tutorials/images/decision-tree/
- W3C — Technique H37: Using alt attributes — https://w3c.github.io/wcag/techniques/html/H37
- W3C — Technique H67: Using null alt text — https://www.w3.org/WAI/WCAG21/Techniques/html/H67
- W3C — Technique F39: Failure due to non-null alt — https://www.w3.org/WAI/WCAG22/Techniques/failures/F39
- W3C — Technique F38: Failure due to not marking up decorative images — https://w3c.github.io/wcag/techniques/failures/F38
- W3C — Method: Decorative Images — https://www.w3.org/WAI/GL/WCAG3/2022/methods/decorative-images/
- W3C — Method: Functional Images — https://www.w3.org/WAI/GL/WCAG3/2020/methods/functional-images/

---

## Summary Table of Image Accessibility Concepts

| Concept | WCAG Criterion | Key Rule | Common Failure |
|---|---|---|---|
| **Alt text** | 1.1.1 (A) | Describe the purpose, not the picture | Missing alt, filename as alt, “image of” prefix |
| **Decorative images** | 1.1.1 (A) | `alt=""` with no `title` | Non-null alt like “spacer,” or missing alt |
| **Functional images** | 1.1.1 (A) | Describe the action, not the object | Missing alt, describing the visual object |
| **Complex images** | 1.1.1 (A) | Short alt + long description | Short alt alone with no data |
| **Best practices** | Multiple | Purpose-driven, concise, accurate | Overly long alt, keyword stuffing, images of text |

---

## Final Notes

- **Purpose is the organising principle**: Every alt-text decision flows from the image's purpose — informative, decorative, functional, or complex.
- **`alt=""` is not the same as no `alt`**: A missing attribute can cause filenames to be announced; an empty attribute tells AT to skip the image.
- **Functional images describe actions**: “Search,” “Print,” “W3C home” — not “magnifying glass” or “logo”.
- **Complex images need two parts**: A short identifier and a long description that conveys the essential data.
- **Images of text should be avoided**: Real text is more accessible, scalable, and localisable.
- **Test with real assistive technologies**: Automated checkers catch missing `alt` and null-alt misuse, but only screen-reader testing verifies that the alt text actually conveys the right meaning.

---

This cheat sheet has covered image accessibility — alt text, decorative images, functional images, complex images, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate image accessibility against WCAG 2.2 Success Criterion 1.1.1 (Non-text Content) and the W3C Images Tutorial taxonomy.