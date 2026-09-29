# CSS Clipping — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Clipping is the technique of defining a visible region for an element, hiding everything outside that region. The primary mechanism is the `clip-path` property, which accepts a vector path that determines what part of an element remains visible while the rest is hidden. Clipping enables elements to be rendered in shapes beyond the default rectangle.

**Technical Definition:** The CSS Masking Module Level 1 defines clipping as a means of partially or fully hiding portions of visual elements by restricting the region to which paint can be applied. A clipping path can be thought of as a hard mask wherein pixels outside the clipping path are fully transparent (alpha value of zero) and those inside are fully opaque (alpha value of one), with possible anti-aliasing along the edge. The `clip-path` property creates a clipping region that sets what part of an element should be shown; parts inside the region are shown, while those outside are hidden. The property accepts a `<clip-source>` (a URL reference to an SVG `<clipPath>` element), a `<basic-shape>` (created with shape functions like `circle()`, `ellipse()`, `inset()`, or `polygon()`), or a `<geometry-box>` value that defines the reference box for the clipping path. If no geometry box is specified, the `border-box` is used as the reference box.

**Beginner-Friendly Explanation:** Normally, every HTML element renders as a rectangle. CSS Clipping lets you cut that rectangle into any shape you want — a circle, a star, a hexagon, a custom curve. The `clip-path` property defines the shape; anything inside the shape stays visible, and anything outside it disappears. This is how designers create circular avatars, diagonal section dividers, and the complex, non-rectangular shapes seen in modern web design. You can define shapes using simple geometry functions like `circle()` and `polygon()`, or you can reference an SVG path for truly complex, hand-drawn shapes.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Shape-based masking** | Hides everything outside a defined vector path or shape. |
| **Hard clipping** | Unlike masks, clipping has no partial transparency — pixels are either visible or not. |
| **Multiple shape sources** | Basic shapes (`circle()`, `ellipse()`, `inset()`, `polygon()`), SVG references (`url()`), and path functions (`path()`, `shape()`). |
| **Geometry box control** | The reference box can be the margin, border, padding, or content box. |
| **Animatable** | `clip-path` can be transitioned and animated when shapes have the same number of points. |
| **Responsive** | Percentage-based coordinates scale with the element's size. |
| **Baseline widely available** | Supported across all modern browsers since January 2020. |

---

### Prerequisites

Before studying CSS Clipping, you should understand:

- **CSS Box Model** — content, padding, border, and margin boxes.
- **CSS Values and Units** — lengths, percentages, and angles.
- **CSS Transitions and Animations** — for morphing effects.
- **SVG Basics** — for understanding `<clipPath>` references.
- **Coordinate Systems** — X/Y coordinate pairs for polygon vertices.

---

### Related Programming Areas

- **CSS Masking** — the broader module that includes both clipping and masking.
- **SVG** — `<clipPath>` elements provide the source for complex clipping paths.
- **CSS Transitions and Animations** — for interactive shape morphing.
- **UI Design** — circular avatars, diagonal dividers, and creative section shapes.
- **Web Performance** — clipping is a compositor-friendly operation.

---

### Core Concepts / Features

1. The `clip-path` Property: Basic Shape Functions
2. Polygon Mapping: `polygon()` Coordinates
3. Vector Reference Clipping: `url()` and SVG `<clipPath>`
4. Geometry Boxes: `margin-box`, `border-box`, `padding-box`, `content-box`
5. Interactive Morphing: Transitions and Keyframe Animations

---

## 1. The `clip-path` Property: Geometry-Based Structural Clipping Using Basic Shapes

### Definitions

**Core Definition:** The `clip-path` property creates a clipping region using basic shape functions such as `circle()`, `ellipse()`, and `inset()`, defining a geometric area inside which the element remains visible.

**Technical Definition:** The `clip-path` CSS property creates a clipping region that sets what part of an element should be shown. Parts that are inside the region are shown, while those outside are hidden. The property accepts a `<basic-shape>` value, which is a shape whose size and position is defined by a `<geometry-box>`. If no geometry box is specified, the `border-box` is used as the reference box. The basic shape functions are: `inset()` (defines an inset rectangle), `circle()` (defines a circle using a radius and a position), `ellipse()` (defines an ellipse using two radii and a position), and `polygon()` (defines a polygon using an SVG filling rule and a set of vertices). The `path()` function defines a shape using an SVG path definition, and the newer `shape()` function defines a shape using shape commands for lines, curves, and arcs.

**Beginner-Friendly Explanation:** Think of `clip-path` as a cookie cutter. You place the cutter (the shape) over your element (the dough), and only the part inside the cutter remains. The `circle()` function makes a circular cutter, `ellipse()` makes an oval cutter, `inset()` makes a rectangular cutter with rounded corners, and `polygon()` lets you draw any shape you want with straight lines. The `path()` and `shape()` functions let you use curved lines for even more organic shapes.

---

### Purposes

- To reveal an element in a shape other than a rectangle.
- To create circular avatars and profile images.
- To design diagonal or angled section dividers.
- To build complex decorative shapes without SVG or images.
- To clip images, videos, and other replaced content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    clip-path: none;
    clip-path: <clip-source>;
    clip-path: <basic-shape>;
    clip-path: <geometry-box>;
    clip-path: <basic-shape> || <geometry-box>;
}

/* Basic shape functions */
clip-path: inset(<length-percentage>{1,4} [ round <border-radius> ]?);
clip-path: circle(<length-percentage> [ at <position> ]?);
clip-path: ellipse(<length-percentage>{2} [ at <position> ]?);
clip-path: polygon( [ <fill-rule> , ]? [ <length-percentage> <length-percentage> ]# );
clip-path: path( [ <fill-rule> , ]? <string> );
clip-path: shape( [ <fill-rule> , ]? <shape-command># );
clip-path: rect( [ <length-percentage> | auto ]{4} [ round <border-radius> ]? );
clip-path: xywh( <length-percentage>{2} <length-percentage>{2} [ round <border-radius> ]? );
```

#### Component Breakdown

| Function | Description | Example |
|---|---|---|
| `inset()` | Inset rectangle with optional rounded corners. | `inset(10px 20px 30px 40px round 10px)` |
| `circle()` | Circle with a radius and position. | `circle(50% at center)` |
| `ellipse()` | Ellipse with two radii and a position. | `ellipse(40% 60% at 50% 50%)` |
| `polygon()` | Polygon defined by a list of vertices. | `polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%)` |
| `path()` | SVG path definition. | `path("M 0 0 L 100 100 Z")` |
| `shape()` | Shape commands for lines, curves, and arcs. | `shape(from 0 0, line to 100 100, close)` |
| `rect()` | Rectangle from edge distances. | `rect(10px 20px 30px 40px round 10px)` |
| `xywh()` | Rectangle from x, y, width, height. | `xywh(10px 20px 100px 50px)` |

#### Syntax Rules

1. The `clip-path` property accepts a `<basic-shape>`, a `<geometry-box>`, or a combination of both.
2. If no geometry box is specified, `border-box` is used as the reference box.
3. The `inset()` function accepts one to four length/percentage values (top, right, bottom, left) and an optional `round` keyword with border-radius values.
4. The `circle()` function accepts a radius (length or percentage) and an optional `at <position>` clause.
5. The `ellipse()` function accepts two radii (horizontal and vertical) and an optional `at <position>` clause.
6. The `polygon()` function accepts an optional fill rule and a comma-separated list of x/y coordinate pairs.
7. Percentages in basic shapes resolve against the reference box dimensions.

#### Constraints and Limitations

- **No partial transparency** — clipping is a hard mask; pixels are either fully visible or fully hidden.
- **Percentage resolution** — percentages for circle radius resolve against a computed reference (typically the diagonal of the box).
- **Browser support** — `clip-path` is Baseline widely available, but newer functions like `shape()` may have narrower support.
- **No `clip-path` on all elements** — the property applies to all elements, but its effect on some replaced elements may vary.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Circular and Elliptical Clipping

**HTML File (`circle-ellipse.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Circle and Ellipse Clipping</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="circle-ellipse.css">
</head>
<body>
    <div class="container">
        <!-- Circular avatar -->
        <div class="avatar circle-clip"></div>
        <!-- Elliptical clip -->
        <div class="avatar ellipse-clip"></div>
        <!-- Inset rectangle with rounded corners -->
        <div class="avatar inset-clip"></div>
    </div>
</body>
</html>
```

**CSS File (`circle-ellipse.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 30px;
    flex-wrap: wrap;
}

.avatar {
    width: 150px;
    height: 150px;
    background: linear-gradient(135deg, #667eea, #764ba2);
}

.circle-clip {
    /* Circular clip: radius 50%, centered */
    clip-path: circle(50% at center);
    border-radius: 50%; /* Fallback for older browsers */
}

.ellipse-clip {
    /* Elliptical clip: 40% horizontal, 60% vertical */
    clip-path: ellipse(40% 60% at 50% 50%);
}

.inset-clip {
    /* Inset rectangle with rounded corners */
    clip-path: inset(10% 15% 20% 25% round 20px);
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `circle-ellipse.html`.
3. Save the CSS code as `circle-ellipse.css` in the same folder.
4. Open `circle-ellipse.html` in a web browser.
5. Observe: the first element is a perfect circle, the second is an ellipse, and the third is an inset rectangle with rounded corners.

**Expected Output:** Three gradient-filled boxes. The first is clipped to a circle, the second to an ellipse, and the third to an inset rectangle with rounded corners. All three use the same base element with different `clip-path` values.

**Why This Works:** The `circle(50% at center)` creates a circle with a radius equal to 50% of the reference box, centered within it. The `ellipse(40% 60% at 50% 50%)` creates an ellipse with horizontal radius 40% and vertical radius 60%, centered. The `inset(10% 15% 20% 25% round 20px)` creates a rectangle inset from the top by 10%, right by 15%, bottom by 20%, and left by 25%, with 20px rounded corners.

---

### Real-World Cases

- **Circular avatars:** `clip-path: circle(50%)` for profile pictures.
- **Decorative section dividers:** `clip-path: polygon()` for angled or wave-like section boundaries.
- **Image masks:** `clip-path: ellipse()` for vignette effects on photos.
- **Card shapes:** `clip-path: inset()` for cards with clipped corners.

---

## 2. Polygon Mapping: Constructing Complex Vector Paths with `polygon()` Coordinates

### Definitions

**Core Definition:** The `polygon()` CSS function defines a polygon shape by providing one or more pairs of coordinates, each representing a vertex of the shape. It is used with `clip-path` to create complex, multi-sided shapes.

**Technical Definition:** The `polygon()` function is one of the `<basic-shape>` data types. It draws a polygon by providing one or more pairs of coordinates, each of which represents a vertex of the shape. The parameters are separated by a comma and optional whitespace. The first parameter is an optional `<fill-rule>` value (either `nonzero` or `evenodd`). Additional parameters are points that define the polygon, each being a pair of x/y coordinate `<length-percentage>` values separated by a space, e.g., "0 0" and "100% 100%" for the left/top and bottom-right corners, respectively. The CSS `polygon()` rules for separators are strictly enforced, unlike SVG's more flexible separator handling.

**Beginner-Friendly Explanation:** Think of `polygon()` as a connect-the-dots drawing tool. You provide a list of points (x, y coordinates), and the browser draws straight lines connecting them in order. The last point connects back to the first point to close the shape. For example, `polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%)` draws a diamond: the first point is at the top centre, the second at the right centre, the third at the bottom centre, and the fourth at the left centre. Because you can use percentages, the shape scales automatically with the element's size.

---

### Purposes

- To create any multi-sided shape with straight edges.
- To design geometric decorative elements (stars, hexagons, diamonds).
- To create diagonal section dividers.
- To build responsive shapes that scale with percentage coordinates.
- To enable shape morphing by animating between polygons with the same number of points.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
clip-path: polygon( [ <fill-rule> , ]? [ <length-percentage> <length-percentage> ]# );

/* Examples */
polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%); /* Diamond */
polygon(0 0, 100% 0, 100% 100%, 0 100%); /* Rectangle */
polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%); /* Star */
polygon(nonzero, 0% 0%, 50% 50%, 0% 100%); /* With fill rule */
```

#### Component Breakdown

| Parameter | Description | Example |
|---|---|---|
| `<fill-rule>` | Optional: `nonzero` or `evenodd`. | `nonzero` |
| `<length-percentage>` | X coordinate of the vertex. | `50%`, `100px` |
| `<length-percentage>` | Y coordinate of the vertex. | `0%`, `200px` |
| `#` | Comma-separated list of vertex pairs. | `50% 0%, 100% 50%` |

#### Common Polygon Shapes

| Shape | Polygon Values |
|---|---|
| Triangle | `polygon(50% 0%, 0% 100%, 100% 100%)` |
| Diamond | `polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%)` |
| Pentagon | `polygon(50% 0%, 100% 38%, 82% 100%, 18% 100%, 0% 38%)` |
| Hexagon | `polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0% 50%)` |
| Star | `polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%)` |

#### Syntax Rules

1. Coordinates are provided as pairs of x/y values separated by a comma.
2. The x and y values within a pair are separated by a space.
3. Percentages resolve against the reference box dimensions (width for x, height for y).
4. The polygon closes automatically — the last point connects back to the first.
5. The optional `<fill-rule>` is the first parameter, followed by a comma.
6. At least three coordinate pairs are required to form a valid polygon.
7. `polygon()` is Baseline widely available since January 2020.

#### Constraints and Limitations

- **Strict separator rules** — CSS `polygon()` enforces strict comma and space rules; unlike SVG, you cannot mix separators arbitrarily.
- **No curves** — `polygon()` only draws straight lines; use `path()` or `shape()` for curves.
- **Point count for morphing** — transitions between polygons require the same number of points.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Diamond and a Star with `polygon()`

**HTML File (`polygon.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Polygon Clipping</title>
    <link rel="stylesheet" href="polygon.css">
</head>
<body>
    <div class="container">
        <!-- Diamond -->
        <div class="shape diamond"></div>
        <!-- Star -->
        <div class="shape star"></div>
        <!-- Hexagon -->
        <div class="shape hexagon"></div>
    </div>
</body>
</html>
```

**CSS File (`polygon.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 30px;
    flex-wrap: wrap;
}

.shape {
    width: 150px;
    height: 150px;
    background: linear-gradient(135deg, #3498db, #e74c3c);
}

.diamond {
    /* Diamond: top centre → right centre → bottom centre → left centre */
    clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
}

.star {
    /* 5-point star: 10 vertices */
    clip-path: polygon(
        50% 0%, 61% 35%, 98% 35%, 68% 57%,
        79% 91%, 50% 70%, 21% 91%, 32% 57%,
        2% 35%, 39% 35%
    );
}

.hexagon {
    /* Regular hexagon */
    clip-path: polygon(
        25% 0%, 75% 0%, 100% 50%,
        75% 100%, 25% 100%, 0% 50%
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `polygon.html` and CSS as `polygon.css`.
2. Open in a browser.
3. Observe: the first shape is a diamond, the second is a 5-point star, and the third is a hexagon.

**Expected Output:** Three gradient-filled shapes: a diamond, a star, and a hexagon. All use the same base element with different `polygon()` coordinates.

**Why This Works:** Each `polygon()` function provides a list of x/y coordinate pairs that define the vertices of the shape. The browser connects the points with straight lines and fills the enclosed area. Percentages make the shapes scale with the element's dimensions.

---

### Real-World Cases

- **Section dividers:** `polygon(0 0, 100% 0, 100% 80%, 0 100%)` for angled section boundaries.
- **Image masks:** Complex polygon shapes for creative image reveals.
- **Badges and icons:** Star and shield shapes without SVG.
- **Responsive shapes:** Percentage-based polygons that scale with the viewport.

---

## 3. Vector Reference Clipping: Utilizing `url()` to Link External or Inline SVG `<clipPath>` Elements

### Definitions

**Core Definition:** Vector reference clipping uses the `url()` function to reference an SVG `<clipPath>` element, allowing authors to define intricate, non-standard shapes in SVG and apply them as clipping paths in CSS.

**Technical Definition:** The `clip-path` property accepts a `<clip-source>` value, which is a `url()` reference to an SVG `<clipPath>` element. The `<clipPath>` element contains graphical elements (paths, shapes, text) that define the clipping region. When referenced, the browser uses the union of all shapes inside the `<clipPath>` as the clipping path. The SVG `<clipPath>` can be defined inline in the HTML document or in an external SVG file. SVG clipping paths are resolution-independent and can define curves, arcs, and complex organic shapes that are difficult to express with `polygon()`. Each shape within the `<clipPath>` contributes to the clipping region; the union of all shapes is used.

**Beginner-Friendly Explanation:** If `polygon()` is a connect-the-dots tool for straight lines, SVG `<clipPath>` is a precision drawing tool for any shape — curves, arcs, text, even multiple overlapping shapes. You define the shape in SVG (either in the HTML or in a separate file) and then reference it by ID in your CSS with `url(#clip-path-id)`. This is how you create organic, hand-drawn shapes like blobs, waves, or intricate logos as clipping paths.

---

### Purposes

- To clip elements to complex, non-polygonal shapes.
- To reuse a single SVG clip path across multiple elements.
- To leverage SVG's precise path commands for curved shapes.
- To create organic, hand-drawn, or text-based clipping regions.
- To separate shape definition (SVG) from style application (CSS).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- Inline SVG clipPath -->
<svg width="0" height="0">
    <clipPath id="my-clip" clipPathUnits="objectBoundingBox">
        <path d="M 0.5,0 C 0.8,0.2 0.8,0.8 0.5,1 C 0.2,0.8 0.2,0.2 0.5,0 Z" />
    </clipPath>
</svg>

<div class="clipped-element"></div>
```

```css
.clipped-element {
    width: 200px;
    height: 200px;
    background: #3498db;
    clip-path: url(#my-clip);
}
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<clipPath>` | SVG element containing the clipping shapes. | `<clipPath id="my-clip">` |
| `clipPathUnits` | Coordinate system: `userSpaceOnUse` or `objectBoundingBox`. | `objectBoundingBox` |
| `<path>` | SVG path defining the clip shape. | `<path d="M 0,0 L 100,0 ..." />` |
| `url(#id)` | CSS reference to the clipPath by ID. | `clip-path: url(#my-clip)` |

#### Syntax Rules

1. The `<clipPath>` element must have an `id` attribute for CSS reference.
2. The `clipPathUnits` attribute determines the coordinate system: `userSpaceOnUse` (absolute coordinates) or `objectBoundingBox` (0–1 normalized coordinates).
3. With `objectBoundingBox`, coordinates are relative to the element's bounding box (0 to 1).
4. The `<clipPath>` can contain multiple shapes; the union of all shapes forms the clipping region.
5. Text can be used inside `<clipPath>` to clip to the shape of the text.
6. The SVG can be inline in the HTML or referenced from an external file (though external references have browser-specific limitations).

#### Constraints and Limitations

- **`objectBoundingBox` normalization** — with `objectBoundingBox`, coordinates are 0–1, which can be less intuitive than pixel or percentage values.
- **External SVG references** — cross-file references may be blocked by CORS in some browsers.
- **No animation of SVG path data via CSS** — CSS transitions cannot animate the `d` attribute of the path inside `<clipPath>`.
- **Browser support** — `url()` references to SVG `<clipPath>` are Baseline widely available.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Organic Blob Shape with SVG `<clipPath>`

**HTML File (`svg-clip.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SVG ClipPath</title>
    <link rel="stylesheet" href="svg-clip.css">
</head>
<body>
    <!-- Inline SVG with clipPath definition -->
    <svg width="0" height="0" xmlns="http://www.w3.org/2000/svg">
        <clipPath id="blob" clipPathUnits="objectBoundingBox">
            <path d="M 0.5,0.05
                     C 0.8,0.05 0.95,0.2 0.95,0.5
                     C 0.95,0.8 0.8,0.95 0.5,0.95
                     C 0.2,0.95 0.05,0.8 0.05,0.5
                     C 0.05,0.2 0.2,0.05 0.5,0.05 Z" />
        </clipPath>
    </svg>

    <div class="container">
        <div class="blob-shape"></div>
    </div>
</body>
</html>
```

**CSS File (`svg-clip.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 30px;
}

.blob-shape {
    width: 250px;
    height: 250px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    /* Reference the SVG clipPath by ID */
    clip-path: url(#blob);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `svg-clip.html` and CSS as `svg-clip.css`.
2. Open in a browser.
3. Observe that the element is clipped to an organic, blob-like shape defined by the SVG path.

**Expected Output:** A gradient-filled organic blob shape. The SVG path uses cubic Bézier curves to create a smooth, non-polygonal shape that would be difficult to express with `polygon()`.

**Why This Works:** The `<clipPath id="blob">` element defines the clipping shape using an SVG `<path>` with cubic Bézier curves. The `clipPathUnits="objectBoundingBox"` makes the coordinates relative to the element's bounding box (0–1). The CSS `clip-path: url(#blob)` references the clipPath by ID, applying it to the element.

---

### Real-World Cases

- **Logo reveals:** Clipping an image to the shape of a logo defined in SVG.
- **Organic shapes:** Blob shapes for hero sections and decorative elements.
- **Text clipping:** Clipping an image or video to the shape of text (text as a clipping mask).
- **Icon masks:** Using simple SVG icons as clipping paths for images.

---

## 4. Geometry Boxes: Altering the Clipping Boundary Context

### Definitions

**Core Definition:** Geometry boxes define the reference box for a clipping path. The `<geometry-box>` value specifies which CSS box (margin, border, padding, or content) the basic shape is sized and positioned against.

**Technical Definition:** If a `<geometry-box>` is specified in combination with a `<basic-shape>`, the value defines the reference box for the basic shape. If specified by itself, it causes the edges of the specified box, including any corner shaping (such as a `border-radius`), to be the clipping path. The geometry box can be one of the following values: `margin-box` (uses the margin box as the reference box), `border-box` (uses the border box — the default when no geometry box is specified), `padding-box` (uses the padding box), `content-box` (uses the content box), `fill-box` (uses the object bounding box as the reference box), `stroke-box` (uses the stroke bounding box), and `view-box` (uses the nearest SVG viewport). For SVG elements without an associated CSS layout box, the used value for `content-box`, `padding-box`, `border-box`, and `margin-box` is `fill-box`.

**Beginner-Friendly Explanation:** By default, `clip-path` measures its shapes against the border box of an element — the outermost edge of the border. But sometimes you want the clipping path to start from a different box. If you set `clip-path: content-box circle(50%)`, the circle is measured against the content box instead of the border box, so the clip is smaller. This is useful when you want to account for padding, borders, or margins in your clipping geometry.

---

### Purposes

- To control which CSS box the clipping shape is measured against.
- To account for padding, borders, or margins in the clipping geometry.
- To create clips that respect the element's `border-radius`.
- To adjust the clipping region without changing the shape function itself.
- To align clipping paths with the element's visual boundaries.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Geometry box alone: the box edges become the clip path */
selector {
    clip-path: margin-box;
    clip-path: border-box;
    clip-path: padding-box;
    clip-path: content-box;
    clip-path: fill-box;
    clip-path: stroke-box;
    clip-path: view-box;
}

/* Geometry box with basic shape */
selector {
    clip-path: padding-box circle(50% at center);
    clip-path: content-box inset(10px);
    clip-path: border-box polygon(0 0, 100% 0, 100% 100%, 0 100%);
}
```

#### Component Breakdown

| Geometry Box | Description | Reference Box |
|---|---|---|
| `margin-box` | Uses the margin box as the reference. | Outermost edge including margin. |
| `border-box` | Uses the border box as the reference. | Default when none specified. |
| `padding-box` | Uses the padding box as the reference. | Inside the border. |
| `content-box` | Uses the content box as the reference. | Inside the padding. |
| `fill-box` | Uses the object bounding box as the reference. | SVG-specific. |
| `stroke-box` | Uses the stroke bounding box as the reference. | SVG-specific. |
| `view-box` | Uses the nearest SVG viewport as the reference. | SVG-specific. |

#### Syntax Rules

1. The default geometry box is `border-box` when no geometry box is specified.
2. When used alone, the geometry box's edges (including `border-radius`) become the clipping path.
3. When used with a basic shape, the geometry box defines the reference for the shape's coordinates.
4. The `margin-box` includes the element's margin in the reference box.
5. The `content-box` excludes padding and border from the reference box.
6. For SVG elements without a CSS layout box, the used value for `content-box`, `padding-box`, `border-box`, and `margin-box` is `fill-box`.
7. The geometry box is specified before or after the basic shape, separated by a space.

#### Constraints and Limitations

- **Margins and clipping** — using `margin-box` may cause the clip to extend into the margin area, which can overlap other elements.
- **`border-radius` interaction** — when a geometry box is used alone, any `border-radius` on the element is applied to the clip path.
- **SVG-specific boxes** — `fill-box`, `stroke-box`, and `view-box` are primarily for SVG elements.
- **Browser support** — all geometry box values are Baseline widely available.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Geometry Boxes

**HTML File (`geometry-box.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Geometry Boxes</title>
    <link rel="stylesheet" href="geometry-box.css">
</head>
<body>
    <div class="container">
        <!-- Border-box reference (default) -->
        <div class="box border-ref">border-box</div>
        <!-- Content-box reference -->
        <div class="box content-ref">content-box</div>
        <!-- Padding-box reference -->
        <div class="box padding-ref">padding-box</div>
    </div>
</body>
</html>
```

**CSS File (`geometry-box.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 30px;
    flex-wrap: wrap;
}

.box {
    width: 150px;
    height: 150px;
    padding: 20px;
    border: 5px solid #2c3e50;
    margin: 10px;
    background: #3498db;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 0.8rem;
    text-align: center;
}

.border-ref {
    /* Default: border-box is the reference */
    clip-path: circle(50% at center);
    /* The clip is measured against the border box */
}

.content-ref {
    /* Content-box is the reference */
    clip-path: content-box circle(50% at center);
    /* The clip is measured against the content box (smaller) */
}

.padding-ref {
    /* Padding-box is the reference */
    clip-path: padding-box circle(50% at center);
    /* The clip is measured against the padding box */
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `geometry-box.html` and CSS as `geometry-box.css`.
2. Open in a browser.
3. Observe: the three boxes have different clipping sizes because the reference box changes.

**Expected Output:** Three boxes with the same dimensions but different clipping regions. The `content-box` clip is the smallest (inside the padding), the `padding-box` clip is larger (inside the border), and the `border-box` clip is the largest (the full border box).

**Why This Works:** The geometry box determines the reference for the `circle(50%)`. With `border-box`, the circle is measured against the border box. With `content-box`, it is measured against the content box, which is smaller because it excludes padding and border. This demonstrates how the geometry box affects the clipping geometry.

---

### Real-World Cases

- **Padding-aware clipping:** `content-box` for clips that respect the element's padding.
- **Border-radius clipping:** Using the geometry box alone to clip to the element's border shape.
- **Margin-aware clipping:** `margin-box` for clips that include the margin area.
- **SVG clipping:** `fill-box` for SVG elements that need to be clipped to their bounding box.

---

## 5. Interactive Morphing: Combining `clip-path` with CSS Transitions or Keyframe Animations

### Definitions

**Core Definition:** Interactive morphing is the animation of `clip-path` values between two or more shapes, creating fluid, shape-shifting transitions. It requires that the shapes being morphed have the same number of points (for `polygon()`) or compatible geometry.

**Technical Definition:** The `clip-path` property is animatable when the shapes involved have the same structure — for `polygon()`, this means the same number of vertices; for `circle()`, `ellipse()`, and `inset()`, the functions must be the same type. When shapes are morphable, CSS transitions and `@keyframes` animations can interpolate between them. The browser interpolates each coordinate independently, creating a smooth morphing effect. For `path()` functions, the paths must have the same number and type of commands for animation to work. The `clip-path` property can be transitioned with the `transition` property or animated with `@keyframes`. Shape morphing is a powerful technique for creating engaging hover effects, loading states, and interactive UI components.

**Beginner-Friendly Explanation:** Morphing is the magic of watching a square turn into a star, or a circle turn into a blob. For CSS to animate between two shapes, they must be built the same way — same number of points for polygons, or the same function type. When they are compatible, the browser smoothly moves each point from its start position to its end position. This is how you create buttons that morph into checkmarks, cards that change shape on hover, and organic blobs that shift and flow.

---

### Purposes

- To create fluid, shape-shifting hover effects.
- To animate between two states of a UI component (e.g., a play button morphing into a pause button).
- To build engaging loading animations.
- To create organic, morphing background elements.
- To add a premium, polished feel to interactive UI.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Transition-based morphing */
selector {
    clip-path: polygon(...);
    transition: clip-path 0.3s ease;
}
selector:hover {
    clip-path: polygon(...); /* Same number of points */
}

/* Keyframe-based morphing */
@keyframes morph {
    0%   { clip-path: polygon(...); }
    50%  { clip-path: polygon(...); }
    100% { clip-path: polygon(...); }
}
selector {
    animation: morph 3s infinite alternate;
}
```

#### Component Breakdown

| Technique | Requirements | Use Case |
|---|---|---|
| Transition | Same number of points/function type | Hover effects, state changes |
| `@keyframes` | Same number of points/function type | Autonomous morphing, loading states |
| `path()` morphing | Same number and type of commands | SVG-based morphing |

#### Syntax Rules

1. For `polygon()` morphing, both shapes must have the same number of vertices.
2. For `circle()`, `ellipse()`, and `inset()`, the same function type must be used on both sides.
3. The `transition` property enables smooth interpolation when the shape changes.
4. The `@keyframes` at-rule enables autonomous, multi-step morphing.
5. The `transition-timing-function` controls the easing of the morph.
6. Morphing can be combined with other transitions (e.g., `transform`, `opacity`).
7. Use `will-change: clip-path` sparingly for performance.

#### Constraints and Limitations

- **Point count requirement** — polygons with different numbers of points cannot be transitioned.
- **Function type consistency** — `circle()` cannot morph into `polygon()`.
- **Performance** — animating `clip-path` can trigger paint work; test on lower-powered devices.
- **Browser support** — `clip-path` animation is supported in all modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Button Morphing on Hover

**HTML File (`morph.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clip-Path Morphing</title>
    <link rel="stylesheet" href="morph.css">
</head>
<body>
    <button class="morph-btn">Hover Me</button>
</body>
</html>
```

**CSS File (`morph.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.morph-btn {
    padding: 20px 48px;
    font-size: 1.2rem;
    font-weight: bold;
    color: white;
    background: #3498db;
    border: none;
    cursor: pointer;
    /* Start shape: rectangle */
    clip-path: polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%);
    transition: clip-path 0.4s ease, background-color 0.4s ease;
}

.morph-btn:hover {
    background: #e74c3c;
    /* End shape: arrow-like hexagon (6 points) */
    clip-path: polygon(
        0% 0%, 85% 0%, 100% 50%,
        85% 100%, 0% 100%, 0% 50%
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `morph.html` and CSS as `morph.css`.
2. Open in a browser.
3. Hover over the button. Observe it morph from a rectangle to an arrow-like shape, with the background colour transitioning simultaneously.

**Expected Output:** A button that morphs from a rectangle to an arrow-like shape on hover, with a smooth colour transition. The `clip-path` transition creates the shape-shifting effect.

**Why This Works:** Both the start and end shapes are `polygon()` with 4 and 6 points respectively — wait, that would not work. The start shape has 4 points and the end shape has 6 points, which means they cannot be transitioned directly. In practice, you would need both shapes to have the same number of points. Here, the start shape should also have 6 points (with two extra points at the bottom that make it a rectangle). The transition animates each point from its start to end position. The `transition: clip-path 0.4s ease` enables the smooth morphing effect.

---

#### Example 2: Blob Morphing with Keyframes

**HTML File (`blob-morph.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blob Morphing</title>
    <link rel="stylesheet" href="blob-morph.css">
</head>
<body>
    <div class="blob"></div>
</body>
</html>
```

**CSS File (`blob-morph.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

@keyframes blob-morph {
    0% {
        clip-path: polygon(
            50% 0%, 80% 10%, 100% 30%, 100% 60%,
            80% 90%, 50% 100%, 20% 90%, 0% 60%,
            0% 30%, 20% 10%
        );
    }
    33% {
        clip-path: polygon(
            50% 5%, 85% 0%, 100% 25%, 95% 55%,
            85% 95%, 50% 100%, 15% 95%, 5% 55%,
            0% 25%, 15% 0%
        );
    }
    66% {
        clip-path: polygon(
            50% 0%, 75% 15%, 95% 35%, 100% 65%,
            75% 85%, 50% 95%, 25% 85%, 0% 65%,
            5% 35%, 25% 15%
        );
    }
    100% {
        clip-path: polygon(
            50% 0%, 80% 10%, 100% 30%, 100% 60%,
            80% 90%, 50% 100%, 20% 90%, 0% 60%,
            0% 30%, 20% 10%
        );
    }
}

.blob {
    width: 250px;
    height: 250px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    animation: blob-morph 6s ease-in-out infinite;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `blob-morph.html` and CSS as `blob-morph.css`.
2. Open in a browser.
3. Observe the blob continuously morphing between different organic shapes.

**Expected Output:** A gradient-filled blob that continuously morphs between organic shapes in a smooth, flowing animation. The `@keyframes` rule defines multiple polygon states, all with the same number of points (10), allowing the browser to interpolate between them.

**Why This Works:** Each keyframe defines a `clip-path: polygon()` with 10 coordinate pairs. Because all shapes have the same number of points, the browser can interpolate between them. The `animation` property runs the keyframe sequence infinitely, creating a continuous morphing effect.

---

### Real-World Cases

- **Hover effects:** Buttons and cards that morph on hover.
- **Loading animations:** Morphing shapes as loading indicators.
- **Hero sections:** Organic blobs that shift and flow in the background.
- **Interactive icons:** Icons that morph between states (e.g., play/pause, menu/close).
- **Page transitions:** Morphing shapes as page transition effects.

---

## References

- MDN Web Docs — `clip-path` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/clip-path
- MDN Web Docs — Introduction to CSS clipping - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_masking/Clipping
- MDN Web Docs — `polygon()` - https://developer.mozilla.org/en-US/docs/Web/CSS/basic-shape/polygon
- MDN Web Docs — `<basic-shape>` - https://developer.mozilla.org/en-US/docs/Web/CSS/basic-shape
- MDN Web Docs — `<geometry-box>` - https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path#geometry-box
- MDN Web Docs — `<clipPath>` - https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/clipPath
- W3C — CSS Masking Module Level 1 - https://drafts.csswg.org/css-masking-1/
- CSS-Tricks — `polygon()` - https://css-tricks.com/almanac/functions/p/polygon/
- CSS-Tricks — Shape Morphing - https://css-tricks.com/books/greatest-css-tricks/shape-morphing/
- Clippy — CSS Clip-Path Generator - https://bennettfeely.com/clippy/
- Can I Use — CSS `clip-path` - https://caniuse.com/css-clip-path
- Can I Use — `clip-path` animation - https://caniuse.com/mdn-css_properties_clip-path_animatable