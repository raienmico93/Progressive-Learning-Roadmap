# Graphics & Graphics2D — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `Graphics` is the abstract base class in `java.awt` that provides the drawing primitives used by every Swing component. It exposes methods for drawing lines, rectangles, ovals, arcs, polygons, strings, and images, and for setting the current colour and font. `Graphics2D` extends `Graphics` and adds the modern 2D drawing API: strokes, paints (including gradients), anti-aliasing rendering hints, transforms, compositing, clipping, and shape-based drawing.

**Technical Definition.** `public abstract class Graphics extends Object`. `public abstract class Graphics2D extends Graphics`. A `Graphics` object encapsulates the state needed for basic rendering: the component to draw on, the origin translation, the clipping region, the current colour, the current font, the paint mode, and the current XOR alternation colour. `Graphics2D` adds the graphics state for rendering: the `Paint`, the `Stroke`, the `Composite`, the `AffineTransform`, the clip shape, and the `RenderingHints`. `Graphics` objects are **transient** — they are provided by the painting system to `paintComponent(Graphics g)` and are only valid for the duration of that method. They must never be stored or used after the method returns. All on-screen painting is performed on the Event Dispatch Thread (EDT); the `Graphics` object is not thread-safe.

**Beginner-Friendly Explanation.** `Graphics` is the **paintbrush** that Swing hands to your `paintComponent` method. It knows how to draw lines, rectangles, ovals, arcs, polygons, text, and images onto a component. `Graphics2D` is a **more capable brush** that adds smooth edges (anti-aliasing), gradient fills, dashed and thick lines, transparency, rotation, scaling, and more. You almost never create a `Graphics` object yourself; Swing provides one every time your component is painted. You use it to draw, and then you discard it — do not keep a reference to it. Think of `Graphics` as a stateful drawing cursor: it has a current colour, a current font, and a current clip, and every drawing call uses the current state. `Graphics2D` adds even more state: a stroke, a paint, a transform, a composite, and rendering hints. Modifying that state affects subsequent drawing, which is why it is best practice to work on a **copy** of the `Graphics` object so your changes do not leak into other painting code.

---

### Key Characteristics

1. **Abstract, transient, and provided by Swing.** `Graphics` and `Graphics2D` are abstract classes. You never instantiate them; Swing provides an instance to `paintComponent(Graphics g)`. It must not be stored, reused, or accessed from another thread.
2. **Stateful drawing cursor.** Every drawing method uses the current colour, font, clip, paint, stroke, and transform. Changing state affects subsequent calls.
3. **Coordinate system.** The origin `(0, 0)` is the upper-left corner of the component; `x` increases to the right and `y` increases downward.
4. **Baseline-based text.** `drawString` positions text relative to its **baseline**, not its top-left corner.
5. **Angles in degrees.** `drawArc` and `fillArc` use angles measured in degrees, with 0° at 3 o'clock and positive angles going **counterclockwise**.
6. **Integer and float overloads.** `Graphics` uses integer coordinates; `Graphics2D` provides float and `Shape`-based overloads for precision.
7. **`Graphics2D` extends `Graphics`.** All `Graphics` methods are available on `Graphics2D`; the reverse is not true.
8. **Copy safely.** Use `(Graphics2D) g.create()` to obtain a private copy when modifying state, and `dispose()` it when done.
9. **Not thread-safe.** All painting happens on the EDT; never share a `Graphics` object across threads.
10. **Rendering quality is configurable.** `Graphics2D` exposes `RenderingHints` for anti-aliasing, interpolation, text rendering, and more.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, anonymous inner classes, and lambda expressions.
- **Swing component basics:** `JComponent`, `JPanel`, `JFrame`, and the painting pipeline (`paintComponent`, `paintBorder`, `paintChildren`).
- **The `paintComponent(Graphics g)` override** and the `super.paintComponent(g)` convention.
- **Basic geometry:** `Point`, `Rectangle`, `Dimension`, and coordinate conventions.
- **The `java.awt` package structure:** `Color`, `Font`, `Image`, `Polygon`, `Shape`.
- **A JDK (version 8 or later).** All graphics classes are in `java.awt` and `java.awt.geom`.

---

### Related Programming Areas

- **`java.awt.Graphics` and `java.awt.Graphics2D`:** The drawing APIs.
- **`java.awt.Color`:** The colour state used by `setColor` and `setPaint`.
- **`java.awt.Font`:** The font state used by `setFont` and `drawString`.
- **`java.awt.Stroke` and `java.awt.BasicStroke`:** The line style state used by `setStroke`.
- **`java.awt.Paint` and its implementations (`Color`, `GradientPaint`, `LinearGradientPaint`, `RadialGradientPaint`, `TexturePaint`):** The fill source used by `setPaint`.
- **`java.awt.RenderingHints`:** The quality settings used by `setRenderingHint`.
- **`java.awt.Image`, `java.awt.image.BufferedImage`, `javax.swing.ImageIcon`:** The image types drawn by `drawImage`.
- **`java.awt.Polygon` and `java.awt.geom.Path2D`:** Shape representations used by `drawPolygon` and `fillPolygon`.
- **`java.awt.geom.Shape`:** The shape abstraction used by `Graphics2D.draw` and `Graphics2D.fill`.
- **`javax.swing.Timer`:** The standard mechanism for repeated `repaint()` calls in animation.
- **`javax.swing.RepaintManager`:** The infrastructure that schedules painting and performs double buffering.

---

## Core Concepts and Features

---

## 1. Graphics — The Abstract Drawing Context

### Definitions

**Core Definition.** `Graphics` is the abstract class that encapsulates the basic drawing state (colour, font, clip, paint mode) and provides primitive drawing methods for lines, rectangles, ovals, arcs, polygons, strings, and images.

**Technical Definition.** `public abstract class Graphics extends Object`. Instances are provided by the painting system. The class declares `setColor`, `getColor`, `setFont`, `getFont`, `setClip`, `getClip`, `getClipBounds`, `drawLine`, `drawRect`, `drawOval`, `drawRoundRect`, `drawArc`, `drawPolygon`, `drawPolyline`, `fillRect`, `fillOval`, `fillRoundRect`, `fillArc`, `fillPolygon`, `drawString`, `drawImage`, `clearRect`, `copyArea`, `translate`, `create`, and `dispose`. It also provides `getFontMetrics` for measuring text.

**Beginner-Friendly Explanation.** `Graphics` is the **basic paintbrush** provided to your paint method. It draws shapes, text, and images using its current colour and font.

---

### Purposes

- To provide a uniform drawing API for components, images, and printers.
- To encapsulate drawing state (colour, font, clip) so callers do not manage it manually.
- To serve as the base class for the more capable `Graphics2D`.
- To provide primitive drawing operations for simple shapes and text.
- To support component painting through the `paintComponent(Graphics g)` contract.

---

### Syntax Rules and Structure

#### Complete General Syntax (Working with a Graphics Object)

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    g.setColor(Color.BLUE);
    g.setFont(new Font("SansSerif", Font.BOLD, 16));
    g.drawLine(10, 10, 200, 10);
    g.drawRect(10, 20, 150, 80);
    g.drawString("Hello", 10, 130);
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `Graphics g` | The drawing context provided by the painting system. |
| `setColor(Color)` | Sets the current colour for subsequent drawing. |
| `setFont(Font)` | Sets the current font for subsequent text. |
| `drawLine(...)` | Draws a line using the current colour. |
| `drawString(...)` | Draws text using the current colour and font. |

#### Syntax Rules

- The `Graphics` object is only valid during the current paint method. Do not store it.
- Every drawing call uses the current colour, font, and clip.
- Coordinate `(0, 0)` is the upper-left corner of the component.
- Use `getFontMetrics(font)` to measure text.

#### Constraints and Limitations

- `Graphics` uses integer coordinates only; use `Graphics2D` for float precision.
- `Graphics` does not support strokes, gradients, anti-aliasing, or transforms; those are `Graphics2D` features.
- The `Graphics` object may be reused internally by Swing; do not assume you own it.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Drawing with Graphics

```java
import javax.swing.*;
import java.awt.*;

public class BasicGraphicsPanel extends JPanel {
    public BasicGraphicsPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                                   // 1. Clear
        g.setColor(Color.BLUE);                                    // 2. Colour
        g.drawLine(20, 20, 380, 20);                               // 3. Line
        g.drawRect(20, 40, 160, 100);                              // 4. Rect
        g.setColor(Color.RED);
        g.drawOval(220, 40, 160, 100);                             // 5. Oval
        g.setColor(Color.BLACK);
        g.setFont(new Font("SansSerif", Font.BOLD, 18));
        g.drawString("Graphics Demo", 20, 200);                    // 6. Text
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Basic Graphics");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BasicGraphicsPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white 400×300 panel with a blue horizontal line at the top, a blue rectangle, a red oval, and the black text “Graphics Demo” near the bottom.

**Why the code produces this result.** The `Graphics` object draws each shape using the current colour and font. Changing the colour between calls affects subsequent drawing.

---

### Real-World Cases

- **Simple custom components** that draw a few shapes and labels.
- **Basic chart panels** that draw bars and axes.
- **Status indicators** that draw simple icons.

---

## 2. Graphics2D — The Modern Drawing API

### Definitions

**Core Definition.** `Graphics2D` extends `Graphics` with the modern 2D rendering API: shapes, strokes, paints, transforms, compositing, clipping, and rendering hints.

**Technical Definition.** `public abstract class Graphics2D extends Graphics`. It adds `setStroke`, `getStroke`, `setPaint`, `getPaint`, `setComposite`, `getComposite`, `setTransform`, `getTransform`, `transform`, `translate`, `rotate`, `scale`, `shear`, `clip`, `setClip`, `getClip`, `draw(Shape)`, `fill(Shape)`, `drawString(String, float, float)`, `drawString(AttributedCharacterIterator, float, float)`, `setRenderingHint`, `getRenderingHint`, `setRenderingHints`, `getRenderingHints`, `setBackground`, `getBackground`, `create`, and `dispose`.

**Beginner-Friendly Explanation.** `Graphics2D` is the **professional paintbrush**. It adds smooth edges, gradients, thick and dashed lines, transparency, rotation, scaling, and high-quality text. Cast `g` to `Graphics2D` to use it.

---

### Purposes

- To draw anti-aliased, high-quality shapes and text.
- To fill shapes with gradients, textures, and alpha blends.
- To draw lines with custom widths, caps, joins, and dash patterns.
- To apply transforms (translate, rotate, scale, shear) to drawing operations.
- To draw arbitrary `Shape` implementations and `Path2D` paths.
- To control rendering quality through `RenderingHints`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();                       // 1. Copy
    try {
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                            RenderingHints.VALUE_ANTIALIAS_ON);    // 2. Quality
        g2.setPaint(new GradientPaint(0, 0, Color.BLUE,
                                      getWidth(), getHeight(), Color.CYAN));
        g2.setStroke(new BasicStroke(4f, BasicStroke.CAP_ROUND,
                                     BasicStroke.JOIN_ROUND));
        g2.translate(100, 100);                                    // 3. Transform
        g2.rotate(Math.toRadians(30));
        g2.fill(new java.awt.geom.RoundRectangle2D.Double(
            -50, -25, 100, 50, 20, 20));                            // 4. Shape
    } finally {
        g2.dispose();                                              // 5. Dispose
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `(Graphics2D) g` | Casts the graphics context to the 2D API. |
| `g.create()` | Creates a private copy so state changes do not affect other painting. |
| `setRenderingHint(...)` | Configures anti-aliasing and other quality settings. |
| `setPaint(Paint)` | Sets the fill source (Color, GradientPaint, etc.). |
| `setStroke(Stroke)` | Sets the line style (width, caps, joins, dashes). |
| `setTransform(...)` / `translate` / `rotate` / `scale` | Modifies the coordinate transform. |
| `draw(Shape)` / `fill(Shape)` | Draws or fills arbitrary shapes. |
| `dispose()` | Releases the copy. |

#### Syntax Rules

- Casting `Graphics` to `Graphics2D` is always safe in Swing painting.
- Use `g2.create()` when modifying state; dispose the copy.
- `Shape`-based drawing is the modern, precise approach.
- Rendering hints affect quality and performance.

#### Constraints and Limitations

- Anti-aliasing and alpha compositing have performance costs.
- `GradientPaint` requires distinct start and end points.
- `Graphics2D` state is not restored automatically; work on a copy or restore manually.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Gradient Rounded Rectangle with Anti-Aliasing

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.RoundRectangle2D;

public class GradientPanel extends JPanel {
    public GradientPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();                   // 1. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON); // 2. AA
            g2.setPaint(new GradientPaint(
                50, 50, Color.BLUE, 300, 250, Color.CYAN));        // 3. Gradient
            g2.fill(new RoundRectangle2D.Double(50, 50, 250, 200, 30, 30)); // 4. Fill
        } finally {
            g2.dispose();                                          // 5. Dispose
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Gradient Panel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new GradientPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A smooth, anti-aliased rounded rectangle filled with a blue-to-cyan gradient.

**Why the code produces this result.** Anti-aliasing smooths the edges. `GradientPaint` fills the shape with a linear gradient. `RoundRectangle2D` defines the rounded shape.

---

#### Example 2: Rotated Text with Transform

```java
import javax.swing.*;
import java.awt.*;

public class RotatedTextPanel extends JPanel {
    public RotatedTextPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setFont(new Font("SansSerif", Font.BOLD, 24));
            g2.setColor(Color.DARK_GRAY);
            g2.translate(200, 150);                                // 1. Origin
            g2.rotate(Math.toRadians(-30));                        // 2. Rotate
            g2.drawString("Rotated Text", -80, 0);                  // 3. Draw
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Rotated Text");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new RotatedTextPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text “Rotated Text” drawn at a -30° angle, centered at (200, 150).

**Why the code produces this result.** `translate` moves the origin to the centre, `rotate` rotates the coordinate system, and `drawString` draws relative to the new origin.

---

### Real-World Cases

- **Charts and diagrams** with anti-aliased shapes and gradients.
- **Custom buttons and panels** with rounded corners and themed gradients.
- **Game canvases** with transformed sprites.
- **Custom renderers** with high-quality text.

---

## 3. drawLine()

### Definitions

**Core Definition.** `drawLine(int x1, int y1, int x2, int y2)` draws a line from `(x1, y1)` to `(x2, y2)` using the current colour and stroke.

**Technical Definition.** `public abstract void drawLine(int x1, int y1, int x2, int y2)`. In `Graphics`, the line has a 1-pixel width. In `Graphics2D`, the line is drawn using the current `Stroke`, so widths, caps, joins, and dashes are respected. The line is clipped to the current clip region.

**Beginner-Friendly Explanation.** `drawLine` draws a straight line between two points.

---

### Purposes

- To draw straight lines, axes, borders, and connectors.
- To draw grids and guides.
- To draw simple diagrams and charts.
- To draw underlines and separators.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawLine(int x1, int y1, int x2, int y2);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x1`, `y1` | The start point of the line. |
| `x2`, `y2` | The end point of the line. |

#### Syntax Rules

- Uses the current colour.
- In `Graphics2D`, uses the current stroke.
- The line is clipped to the current clip region.

#### Constraints and Limitations

- `Graphics.drawLine` draws a 1-pixel line; use `Graphics2D.setStroke` for thicker lines.
- Coordinates are integers; use `Graphics2D.draw(Line2D)` for float precision.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Drawing a Simple Line

```java
import javax.swing.*;
import java.awt.*;

public class DrawLinePanel extends JPanel {
    public DrawLinePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.BLUE);
        g.drawLine(20, 20, 280, 180);                              // 1. Diagonal
        g.setColor(Color.RED);
        g.drawLine(20, 180, 280, 20);                              // 2. Opposite
    }
}
```

**Expected Output.** A white panel with a blue diagonal line from top-left to bottom-right and a red diagonal from bottom-left to top-right, forming an X.

---

#### Example 2: Thick Line with Graphics2D

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    g2.setStroke(new BasicStroke(4f, BasicStroke.CAP_ROUND,
                                 BasicStroke.JOIN_ROUND));
    g2.setColor(Color.DARK_GRAY);
    g2.drawLine(20, 100, 280, 100);
} finally {
    g2.dispose();
}
```

**Expected Output.** A 4-pixel-wide dark-gray horizontal line with rounded end caps.

---

### Real-World Cases

- **Chart axes** and grid lines.
- **Connectors** in diagram editors.
- **Separators** in custom components.
- **Underlines** under custom text.

---

## 4. drawRect()

### Definitions

**Core Definition.** `drawRect(int x, int y, int width, int height)` draws the outline of a rectangle with its upper-left corner at `(x, y)` and the given width and height.

**Technical Definition.** `public void drawRect(int x, int y, int width, int height)`. The rectangle is drawn using the current colour. In `Graphics`, the outline is 1 pixel wide. The rectangle spans from `x` to `x + width` and from `y` to `y + height` inclusive, so the actual width is `width + 1` pixels.

**Beginner-Friendly Explanation.** `drawRect` draws a rectangle outline.

---

### Purposes

- To draw boxes, frames, and borders.
- To draw cell outlines and table grids.
- To outline regions of interest.
- To draw simple diagrams.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawRect(int x, int y, int width, int height);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x`, `y` | Upper-left corner. |
| `width` | Width in pixels (inclusive). |
| `height` | Height in pixels (inclusive). |

#### Syntax Rules

- Uses the current colour.
- The rectangle is clipped to the current clip region.
- In `Graphics2D`, use `draw(new Rectangle2D.Double(...))` for float precision.

#### Constraints and Limitations

- `drawRect` is 1 pixel wide in `Graphics`; use `Graphics2D.setStroke` for thicker outlines.
- `width` and `height` are inclusive, so a `width` of 100 yields 101 pixels across.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Rectangle Outline

```java
import javax.swing.*;
import java.awt.*;

public class DrawRectPanel extends JPanel {
    public DrawRectPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.BLUE);
        g.drawRect(20, 20, 260, 160);                              // 1. Outline
    }
}
```

**Expected Output.** A white panel with a blue rectangular outline inset from the edges.

---

#### Example 2: Nested Rectangles

```java
g.setColor(Color.DARK_GRAY);
for (int i = 0; i < 5; i++) {
    g.drawRect(20 + i * 10, 20 + i * 10, 260 - i * 20, 160 - i * 20);
}
```

**Expected Output.** Five nested dark-gray rectangles forming a border-like pattern.

---

### Real-World Cases

- **Component borders** and frames.
- **Grid cells** in custom tables.
- **Selection rectangles** in editors.

---

## 5. drawOval()

### Definitions

**Core Definition.** `drawOval(int x, int y, int width, int height)` draws the outline of an oval that fits within the rectangle defined by `(x, y, width, height)`.

**Technical Definition.** `public void drawOval(int x, int y, int width, int height)`. The oval is drawn using the current colour. The oval is not necessarily circular; it is an ellipse inscribed in the bounding rectangle.

**Beginner-Friendly Explanation.** `drawOval` draws an ellipse inside a rectangle.

---

### Purposes

- To draw circles and ellipses.
- To draw nodes in diagrams.
- To draw indicators and icons.
- To draw decorative shapes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawOval(int x, int y, int width, int height);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x`, `y` | Upper-left corner of the bounding rectangle. |
| `width`, `height` | Size of the bounding rectangle. |

#### Syntax Rules

- The oval is inscribed in the bounding rectangle.
- A circle is drawn when `width == height`.
- Uses the current colour.

#### Constraints and Limitations

- `drawOval` is 1 pixel wide in `Graphics`; use `Graphics2D` for thicker outlines.
- For high-quality anti-aliased circles, use `Graphics2D` with `Ellipse2D` and anti-aliasing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Circle Outline

```java
import javax.swing.*;
import java.awt.*;

public class DrawOvalPanel extends JPanel {
    public DrawOvalPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.RED);
        g.drawOval(50, 50, 200, 200);                              // 1. Circle
        g.setColor(Color.BLUE);
        g.drawOval(80, 80, 140, 80);                               // 2. Ellipse
    }
}
```

**Expected Output.** A white panel with a red circle outline and a blue ellipse outline inside it.

---

### Real-World Cases

- **Diagram nodes** (circles with labels).
- **Status indicators** (dots).
- **Custom icons** and decorative shapes.

---

## 6. drawRoundRect()

### Definitions

**Core Definition.** `drawRoundRect(int x, int y, int width, int height, int arcWidth, int arcHeight)` draws the outline of a rounded rectangle.

**Technical Definition.** `public abstract void drawRoundRect(int x, int y, int width, int height, int arcWidth, int arcHeight)`. The `arcWidth` and `arcHeight` parameters specify the horizontal and vertical diameters of the arcs at the corners. The rectangle’s corners are rounded using these arcs.

**Beginner-Friendly Explanation.** `drawRoundRect` draws a rectangle with rounded corners.

---

### Purposes

- To draw rounded frames and borders.
- To draw buttons and panels with rounded corners.
- To create card-like UI elements.
- To draw decorative boxes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawRoundRect(int x, int y, int width, int height, int arcWidth, int arcHeight);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x`, `y` | Upper-left corner. |
| `width`, `height` | Size of the rectangle. |
| `arcWidth` | Horizontal diameter of the corner arcs. |
| `arcHeight` | Vertical diameter of the corner arcs. |

#### Syntax Rules

- `arcWidth` and `arcHeight` control the corner radius: larger values produce more rounded corners.
- Uses the current colour.

#### Constraints and Limitations

- The maximum arc dimensions are the width and height of the rectangle.
- For anti-aliased rounded rectangles, use `Graphics2D` with `RoundRectangle2D` and `RenderingHints`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Rounded Frame

```java
import javax.swing.*;
import java.awt.*;

public class DrawRoundRectPanel extends JPanel {
    public DrawRoundRectPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.DARK_GRAY);
        g.drawRoundRect(20, 20, 260, 160, 30, 30);                 // 1. Rounded
    }
}
```

**Expected Output.** A white panel with a dark-gray rounded rectangular outline.

---

### Real-World Cases

- **Card-style panels** in dashboards.
- **Custom buttons** with rounded corners.
- **Dialogs** and popups with rounded frames.

---

## 7. drawArc()

### Definitions

**Core Definition.** `drawArc(int x, int y, int width, int height, int startAngle, int arcAngle)` draws the outline of an arc that is part of an oval inscribed in the bounding rectangle.

**Technical Definition.** `public abstract void drawArc(int x, int y, int width, int height, int startAngle, int arcAngle)`. Angles are measured in **degrees**, with 0° at 3 o’clock (the positive x-axis). Positive angles go **counterclockwise**; negative angles go clockwise. The arc spans `arcAngle` degrees starting from `startAngle`.

**Beginner-Friendly Explanation.** `drawArc` draws a curved segment of an ellipse. Angles start at 3 o’clock and go counterclockwise.

---

### Purposes

- To draw pie charts and gauges.
- To draw circular progress indicators.
- To draw curved arrows and decorative arcs.
- To draw portions of circles and ellipses.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawArc(int x, int y, int width, int height, int startAngle, int arcAngle);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x`, `y` | Upper-left corner of the bounding rectangle. |
| `width`, `height` | Size of the bounding rectangle. |
| `startAngle` | Start angle in degrees (0° = 3 o’clock). |
| `arcAngle` | Sweep angle in degrees (positive = counterclockwise). |

#### Syntax Rules

- Angles are in degrees, not radians.
- Positive `arcAngle` sweeps counterclockwise; negative sweeps clockwise.
- An `arcAngle` of 360° or -360° draws a full oval.

#### Constraints and Limitations

- The arc is inscribed in the bounding rectangle; the oval may not be circular.
- The outline is 1 pixel wide in `Graphics`; use `Graphics2D` for thicker arcs.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Quarter-Circle Arc

```java
import javax.swing.*;
import java.awt.*;

public class DrawArcPanel extends JPanel {
    public DrawArcPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.BLUE);
        g.drawArc(50, 50, 200, 200, 0, 90);                        // 1. 0° to 90°
        g.setColor(Color.RED);
        g.drawArc(50, 50, 200, 200, 90, 180);                      // 2. 90° to 270°
    }
}
```

**Expected Output.** A white panel with a blue quarter-circle arc from 3 o’clock to 12 o’clock (counterclockwise) and a red half-circle arc from 12 o’clock to 9 o’clock.

---

### Real-World Cases

- **Pie charts** and donut charts.
- **Gauge indicators** and progress rings.
- **Curved arrows** in diagrams.
- **Clock faces** and dials.

---

## 8. drawPolygon()

### Definitions

**Core Definition.** `drawPolygon(int[] xPoints, int[] yPoints, int nPoints)` draws the outline of a closed polygon defined by the given points. `drawPolygon(Polygon p)` is an equivalent overload.

**Technical Definition.** `public abstract void drawPolygon(int[] xPoints, int[] yPoints, int nPoints)`. The polygon is closed automatically; the last point is connected back to the first. The polygon is drawn using the current colour. `Graphics2D` also allows `draw(Shape)` with a `Polygon` or `Path2D`.

**Beginner-Friendly Explanation.** `drawPolygon` draws a closed shape defined by a series of points.

---

### Purposes

- To draw closed polygons (triangles, stars, arbitrary shapes).
- To draw outlines of complex figures.
- To draw charts with polygonal shapes.
- To draw icons and symbols.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawPolygon(int[] xs, int[] ys, int nPoints);
g.drawPolygon(Polygon p);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `xPoints` | Array of x coordinates. |
| `yPoints` | Array of y coordinates. |
| `nPoints` | Number of points to use. |
| `Polygon p` | A pre-built polygon object. |

#### Syntax Rules

- The polygon is closed automatically.
- The arrays must have at least `nPoints` elements.
- Uses the current colour.

#### Constraints and Limitations

- `Graphics` polygons use integer coordinates; use `Path2D` for float precision.
- Self-intersecting polygons may produce unexpected fill results.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Triangle Outline

```java
import javax.swing.*;
import java.awt.*;

public class DrawPolygonPanel extends JPanel {
    public DrawPolygonPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        int[] xs = {150, 50, 250};
        int[] ys = {50, 250, 250};
        g.setColor(Color.BLUE);
        g.drawPolygon(xs, ys, 3);                                  // 1. Triangle
    }
}
```

**Expected Output.** A white panel with a blue triangle outline pointing upward.

---

### Real-World Cases

- **Diagram shapes** and arrows.
- **Chart outlines** and data polygons.
- **Icons** and symbols.

---

## 9. drawPolyline()

### Definitions

**Core Definition.** `drawPolyline(int[] xPoints, int[] yPoints, int nPoints)` draws a sequence of connected line segments through the given points. Unlike `drawPolygon`, the polyline is **not** closed automatically.

**Technical Definition.** `public abstract void drawPolyline(int[] xPoints, int[] yPoints, int nPoints)`. The polyline is drawn using the current colour. The last point is not connected back to the first. `Graphics2D` inherits this method and also allows `draw(Path2D)` for float-precision polylines.

**Beginner-Friendly Explanation.** `drawPolyline` draws an open path through a series of points.

---

### Purposes

- To draw line graphs and data plots.
- To draw open shapes and paths.
- To draw curved paths approximated by straight segments.
- To draw chart lines.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.drawPolyline(int[] xs, int[] ys, int nPoints);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `xPoints` | Array of x coordinates. |
| `yPoints` | Array of y coordinates. |
| `nPoints` | Number of points to use. |

#### Syntax Rules

- The polyline is not closed.
- Uses the current colour.

#### Constraints and Limitations

- `Graphics` uses integer coordinates; use `Path2D` for float precision.
- For smooth curves, approximate with many segments or use `Path2D` curves.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Zigzag Polyline

```java
import javax.swing.*;
import java.awt.*;

public class DrawPolylinePanel extends JPanel {
    public DrawPolylinePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        int[] xs = {20, 70, 120, 170, 220, 270};
        int[] ys = {180, 40, 180, 40, 180, 40};
        g.setColor(Color.BLUE);
        g.drawPolyline(xs, ys, xs.length);                         // 1. Zigzag
    }
}
```

**Expected Output.** A white panel with a blue zigzag polyline.

---

### Real-World Cases

- **Line charts** and data plots.
- **Graphs** and network diagrams.
- **Open paths** in drawing tools.

---

## 10. fillRect()

### Definitions

**Core Definition.** `fillRect(int x, int y, int width, int height)` fills a rectangle with the current colour.

**Technical Definition.** `public abstract void fillRect(int x, int y, int width, int height)`. The rectangle is filled using the current colour (or, in `Graphics2D`, the current `Paint`). The filled rectangle spans `x` to `x + width - 1` and `y` to `y + height - 1`.

**Beginner-Friendly Explanation.** `fillRect` draws a filled rectangle.

---

### Purposes

- To draw filled backgrounds and panels.
- To draw bars in bar charts.
- To draw filled boxes and buttons.
- To draw highlighted regions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
g.fillRect(int x, int y, int width, int height);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x`, `y` | Upper-left corner. |
| `width`, `height` | Size in pixels. |

#### Syntax Rules

- Uses the current colour (or paint in `Graphics2D`).
- `width` and `height` must be non-negative.

#### Constraints and Limitations

- For gradients, use `Graphics2D` with `setPaint` and `fill(Rectangle2D)`.
- For rounded filled rectangles, use `fillRoundRect` or `RoundRectangle2D`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Filled Bar

```java
import javax.swing.*;
import java.awt.*;

public class FillRectPanel extends JPanel {
    public FillRectPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(new Color(0, 120, 215));
        g.fillRect(50, 50, 200, 100);                              // 1. Bar
    }
}
```

**Expected Output.** A white panel with a blue filled rectangle.

---

### Real-World Cases

- **Bar charts** and histograms.
- **Filled backgrounds** for custom panels.
- **Highlight regions** and selections.

---

## 11. fillOval()

### Definitions

**Core Definition.** `fillOval(int x, int y, int width, int height)` fills an oval inscribed in the given bounding rectangle.

**Technical Definition.** `public abstract void fillOval(int x, int y, int width, int height)`. The oval is filled using the current colour (or `Paint` in `Graphics2D`).

**Beginner-Friendly Explanation.** `fillOval` draws a filled ellipse or circle.

---

### Purposes

- To draw filled circles and ellipses.
- To draw dots, indicators, and nodes.
- To draw decorative shapes.

---

### Syntax Rules and Structure

```java
g.fillOval(int x, int y, int width, int height);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x`, `y` | Upper-left corner of bounding rectangle. |
| `width`, `height` | Size of bounding rectangle. |

#### Constraints and Limitations

- For anti-aliased circles, use `Graphics2D` with `Ellipse2D` and anti-aliasing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Filled Circle

```java
import javax.swing.*;
import java.awt.*;

public class FillOvalPanel extends JPanel {
    public FillOvalPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.RED);
        g.fillOval(50, 50, 200, 200);                              // 1. Circle
    }
}
```

**Expected Output.** A white panel with a red filled circle.

---

### Real-World Cases

- **Nodes** in diagrams.
- **Status dots** and indicators.
- **Custom icons**.

---

## 12. fillRoundRect()

### Definitions

**Core Definition.** `fillRoundRect(int x, int y, int width, int height, int arcWidth, int arcHeight)` fills a rounded rectangle with the current colour.

**Technical Definition.** `public abstract void fillRoundRect(int x, int y, int width, int height, int arcWidth, int arcHeight)`. The rounded rectangle is filled using the current colour (or `Paint`).

**Beginner-Friendly Explanation.** `fillRoundRect` draws a filled rectangle with rounded corners.

---

### Purposes

- To draw filled card-like panels.
- To draw rounded buttons.
- To draw decorative filled shapes.

---

### Syntax Rules and Structure

```java
g.fillRoundRect(int x, int y, int w, int h, int arcW, int arcH);
```

#### Constraints and Limitations

- For anti-aliased rounded rectangles, use `Graphics2D` with `RoundRectangle2D` and anti-aliasing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Filled Rounded Box

```java
import javax.swing.*;
import java.awt.*;

public class FillRoundRectPanel extends JPanel {
    public FillRoundRectPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(new Color(0, 160, 0));
        g.fillRoundRect(50, 50, 200, 100, 30, 30);                 // 1. Fill
    }
}
```

**Expected Output.** A white panel with a green filled rounded rectangle.

---

### Real-World Cases

- **Card panels** and tiles.
- **Filled buttons** with rounded corners.
- **Dialog boxes**.

---

## 13. fillArc()

### Definitions

**Core Definition.** `fillArc(int x, int y, int width, int height, int startAngle, int arcAngle)` fills a pie-slice-shaped region corresponding to the arc.

**Technical Definition.** `public abstract void fillArc(int x, int y, int width, int height, int startAngle, int arcAngle)`. The filled region is bounded by the arc and two radii from the centre of the bounding rectangle. Angles are in degrees, with 0° at 3 o’clock and positive angles counterclockwise.

**Beginner-Friendly Explanation.** `fillArc` draws a filled pie slice.

---

### Purposes

- To draw pie charts.
- To draw gauge segments and progress arcs.
- To draw decorative segments.

---

### Syntax Rules and Structure

```java
g.fillArc(int x, int y, int w, int h, int startAngle, int arcAngle);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `startAngle` | Start angle in degrees. |
| `arcAngle` | Sweep angle in degrees (positive = counterclockwise). |

#### Constraints and Limitations

- The filled region includes the radii to the centre of the bounding rectangle.
- For anti-aliased pie charts, use `Graphics2D` with `Arc2D.PIE` and anti-aliasing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Pie Slice

```java
import javax.swing.*;
import java.awt.*;

public class FillArcPanel extends JPanel {
    public FillArcPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.ORANGE);
        g.fillArc(50, 50, 200, 200, 0, 90);                        // 1. Quarter
        g.setColor(Color.BLUE);
        g.fillArc(50, 50, 200, 200, 90, 180);                      // 2. Half
    }
}
```

**Expected Output.** A white panel with an orange quarter-circle pie slice in the first quadrant and a blue half-circle pie slice in the second and third quadrants.

---

### Real-World Cases

- **Pie charts** and donut charts.
- **Gauges** and progress indicators.
- **Segmented displays**.

---

## 14. fillPolygon()

### Definitions

**Core Definition.** `fillPolygon(int[] xPoints, int[] yPoints, int nPoints)` fills a closed polygon defined by the given points. `fillPolygon(Polygon p)` is an equivalent overload.

**Technical Definition.** `public abstract void fillPolygon(int[] xPoints, int[] yPoints, int nPoints)`. The polygon is filled using the current colour (or `Paint`). The polygon is closed automatically; the interior is filled according to the even-odd or non-zero winding rule (implementation-dependent in `Graphics`, well-defined in `Graphics2D` via `Path2D`).

**Beginner-Friendly Explanation.** `fillPolygon` fills a shape defined by a series of points.

---

### Purposes

- To draw filled triangles, stars, and complex shapes.
- To draw filled map regions and diagram shapes.
- To draw decorative filled polygons.

---

### Syntax Rules and Structure

```java
g.fillPolygon(int[] xs, int[] ys, int nPoints);
g.fillPolygon(Polygon p);
```

#### Constraints and Limitations

- Self-intersecting polygons may fill only some regions.
- Use `Path2D` in `Graphics2D` for precise control over the fill rule.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Filled Star

```java
import javax.swing.*;
import java.awt.*;

public class FillPolygonPanel extends JPanel {
    public FillPolygonPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        int cx = 150, cy = 150, r = 100;
        int[] xs = new int[10];
        int[] ys = new int[10];
        for (int i = 0; i < 10; i++) {
            double angle = -Math.PI / 2 + i * Math.PI / 5;
            int radius = (i % 2 == 0) ? r : r / 2;
            xs[i] = (int) (cx + radius * Math.cos(angle));
            ys[i] = (int) (cy + radius * Math.sin(angle));
        }
        g.setColor(Color.YELLOW);
        g.fillPolygon(xs, ys, 10);                                 // 1. Star
    }
}
```

**Expected Output.** A white panel with a yellow filled star.

---

### Real-World Cases

- **Star ratings** and decorative icons.
- **Map regions** and diagram shapes.
- **Custom chart polygons**.

---

## 15. drawString()

### Definitions

**Core Definition.** `drawString(String str, int x, int y)` draws the given text using the current font and colour, with the baseline of the first character at `(x, y)`.

**Technical Definition.** `public abstract void drawString(String str, int x, int y)`. The text is rendered using the current `Font` and `Color` (or `Paint` in `Graphics2D`). The `y` coordinate refers to the **baseline**, not the top of the text. `Graphics2D` provides `drawString(String, float, float)` and `drawString(AttributedCharacterIterator, float, float)` for float precision and attributed text.

**Beginner-Friendly Explanation.** `drawString` draws text. The `y` coordinate is the baseline, which is where the bottom of most letters sits.

---

### Purposes

- To draw labels, titles, and annotations.
- To display dynamic text values.
- To render chart titles and axis labels.
- To display debug information.

---

### Syntax Rules and Structure

```java
g.drawString(String str, int x, int y);
g2.drawString(String str, float x, float y);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `str` | The text to draw. |
| `x`, `y` | The baseline position of the first character. |

#### Syntax Rules

- Uses the current font and colour.
- The `y` coordinate is the baseline, not the top.
- Use `getFontMetrics(font)` to measure text height and width.

#### Constraints and Limitations

- `drawString` does not wrap text; draw multiple lines manually.
- For multi-line text, calculate line heights using `FontMetrics`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Centered Label

```java
import javax.swing.*;
import java.awt.*;

public class DrawStringPanel extends JPanel {
    public DrawStringPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setFont(new Font("SansSerif", Font.BOLD, 24));
        g.setColor(Color.DARK_GRAY);
        FontMetrics fm = g.getFontMetrics();
        String text = "Hello, World!";
        int x = (getWidth() - fm.stringWidth(text)) / 2;           // 1. Centre
        int y = (getHeight() + fm.getAscent() - fm.getDescent()) / 2;
        g.drawString(text, x, y);                                   // 2. Draw
    }
}
```

**Expected Output.** The text “Hello, World!” centered both horizontally and vertically in the panel.

**Why the code produces this result.** `FontMetrics.stringWidth` gives the pixel width of the text; `getAscent` and `getDescent` give the vertical extents needed to compute the baseline.

---

### Real-World Cases

- **Chart labels** and axis titles.
- **Status messages** and annotations.
- **Custom components** displaying dynamic text.

---

## 16. drawImage()

### Definitions

**Core Definition.** `drawImage(Image img, int x, int y, ImageObserver observer)` draws the given image with its upper-left corner at `(x, y)`. Overloads allow scaling the image to a specified width and height, and specifying background colour or additional clipping.

**Technical Definition.** `public abstract boolean drawImage(Image img, int x, int y, ImageObserver observer)`. Returns `true` if the image is fully loaded and drawn, `false` if the image is still loading or was not found. The `ImageObserver` receives updates as the image loads; passing the component is conventional. Overloads with `width`, `height`, `Color bgcolor`, and `ImageObserver` allow scaling and background filling. `Graphics2D` also allows `drawImage(BufferedImage, ...)` and rendering images with transforms.

**Beginner-Friendly Explanation.** `drawImage` draws a picture onto the component.

---

### Purposes

- To display images (icons, photos, sprites) in custom components.
- To draw background images.
- To draw scaled thumbnails.
- To draw game sprites.

---

### Syntax Rules and Structure

```java
g.drawImage(Image img, int x, int y, ImageObserver observer);
g.drawImage(Image img, int x, int y, int width, int height, ImageObserver observer);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `img` | The image to draw. |
| `x`, `y` | The upper-left corner of the drawn image. |
| `width`, `height` | The scaled dimensions (optional). |
| `observer` | The `ImageObserver` (usually the component). |

#### Syntax Rules

- Use `new ImageIcon("file.png").getImage()` or `ImageIO.read(...)` to load images.
- Pass the component (`this`) or `null` as the observer; `this` is conventional.
- Cache images in fields to avoid reloading.

#### Constraints and Limitations

- `drawImage` may return before the image is fully loaded; the `ImageObserver` handles the updates.
- Loading images in `paintComponent` is a performance anti-pattern; load them once and cache.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Drawing a Scaled Image

```java
import javax.swing.*;
import java.awt.*;

public class DrawImagePanel extends JPanel {
    private final Image image;

    public DrawImagePanel() {
        image = new ImageIcon("photo.png").getImage();             // 1. Load once
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_INTERPOLATION,
                                RenderingHints.VALUE_INTERPOLATION_BILINEAR);
            g2.drawImage(image, 50, 50, 300, 200, this);           // 2. Scale
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A white panel with the image scaled to 300×200 at position (50, 50).

**Why the code produces this result.** `drawImage` with `width` and `height` scales the image. The interpolation rendering hint improves quality. The image is loaded once in the constructor and cached in a field.

---

### Real-World Cases

- **Image viewers** and galleries.
- **Game sprites** and tiles.
- **Background images** in custom panels.
- **Icons** in custom buttons.

---

## 17. setColor()

### Definitions

**Core Definition.** `setColor(Color c)` sets the current colour for subsequent drawing operations.

**Technical Definition.** `public abstract void setColor(Color c)`. The colour is used by `drawLine`, `drawRect`, `drawOval`, `drawArc`, `drawPolygon`, `drawPolyline`, `drawString` (if no paint is set in `Graphics2D`), and all `fillXxx` methods. In `Graphics2D`, `setColor` also updates the `Paint` to the given colour.

**Beginner-Friendly Explanation.** `setColor` changes the paintbrush colour.

---

### Purposes

- To set the drawing colour for shapes and text.
- To switch colours between drawing operations.
- To support theming and state-driven colour changes.
- To draw with alpha transparency (colours with alpha).

---

### Syntax Rules and Structure

```java
g.setColor(Color c);
g.setColor(new Color(r, g, b));
g.setColor(new Color(r, g, b, a));
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `new Color(r, g, b)` | RGB colour with full opacity. |
| `new Color(r, g, b, a)` | RGBA colour with alpha. |
| `Color.RED`, `Color.BLUE`, etc. | Standard colours. |

#### Syntax Rules

- The colour applies to subsequent drawing operations.
- `setColor` affects both outlines and fills.
- Alpha values range from 0 (transparent) to 255 (opaque).

#### Constraints and Limitations

- `setColor` affects only the current graphics context; it does not affect other painting.
- Alpha transparency in `Graphics` may not work as expected unless the surface supports it; `Graphics2D` with `setComposite` is more robust.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Coloured Shapes

```java
import javax.swing.*;
import java.awt.*;

public class SetColorPanel extends JPanel {
    public SetColorPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.RED);
        g.fillRect(20, 20, 80, 80);                                // 1. Red
        g.setColor(Color.GREEN);
        g.fillRect(110, 20, 80, 80);                               // 2. Green
        g.setColor(Color.BLUE);
        g.fillRect(200, 20, 80, 80);                               // 3. Blue
    }
}
```

**Expected Output.** A white panel with three coloured squares: red, green, and blue.

---

### Real-World Cases

- **Themed components** that change colour based on state.
- **Charts** with coloured series.
- **Highlighting** selected regions.

---

## 18. setFont()

### Definitions

**Core Definition.** `setFont(Font font)` sets the current font used by `drawString`.

**Technical Definition.** `public abstract void setFont(Font font)`. The font is used for subsequent `drawString` calls. The font’s family, style (`PLAIN`, `BOLD`, `ITALIC`, `BOLD | ITALIC`), and size are all part of the `Font` object.

**Beginner-Friendly Explanation.** `setFont` changes the font used for text.

---

### Purposes

- To draw text with a specific family, style, and size.
- To emphasise titles and labels.
- To match the application’s typography.
- To support internationalisation (fonts that cover multiple scripts).

---

### Syntax Rules and Structure

```java
g.setFont(new Font("SansSerif", Font.BOLD, 16));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `name` | Font family name (e.g., "SansSerif", "Serif", "Monospaced"). |
| `style` | `Font.PLAIN`, `Font.BOLD`, `Font.ITALIC`, or a combination. |
| `size` | Point size. |

#### Syntax Rules

- The font applies to subsequent `drawString` calls.
- Use logical font names (`"SansSerif"`, `"Serif"`, `"Monospaced"`) for portability.

#### Constraints and Limitations

- Not all fonts support all characters; for international text, use fonts that cover the required scripts.
- Very large fonts may slow painting.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Bold Title

```java
import javax.swing.*;
import java.awt.*;

public class SetFontPanel extends JPanel {
    public SetFontPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setFont(new Font("SansSerif", Font.BOLD, 24));           // 1. Title
        g.setColor(Color.DARK_GRAY);
        g.drawString("Title", 100, 60);
        g.setFont(new Font("SansSerif", Font.PLAIN, 14));          // 2. Body
        g.setColor(Color.GRAY);
        g.drawString("Body text", 100, 100);
    }
}
```

**Expected Output.** A white panel with a bold “Title” and a plain “Body text” in different sizes and colours.

---

### Real-World Cases

- **Chart titles** and axis labels.
- **Status messages** with emphasis.
- **Custom typography** in branded applications.

---

## 19. setStroke()

### Definitions

**Core Definition.** `setStroke(Stroke s)` sets the current stroke used for drawing lines and shape outlines. This method is **only** available on `Graphics2D`, not on `Graphics`.

**Technical Definition.** `public abstract void setStroke(Stroke s)` is declared in `Graphics2D`. The `Stroke` interface defines how to turn a shape into its outline. `BasicStroke` implements it with configurable width, cap style, join style, miter limit, and dash pattern.

**Beginner-Friendly Explanation.** `setStroke` changes the thickness and style of outlines. Use it on `Graphics2D` after casting.

---

### Purposes

- To draw thick lines and shape outlines.
- To draw dashed and dotted lines.
- To configure line caps (rounded, square, butt) and joins (miter, round, bevel).
- To match the visual style of charts and diagrams.

---

### Syntax Rules and Structure

```java
Graphics2D g2 = (Graphics2D) g;
g2.setStroke(new BasicStroke(3f));
g2.setStroke(new BasicStroke(2f, BasicStroke.CAP_ROUND,
                             BasicStroke.JOIN_ROUND));
g2.setStroke(new BasicStroke(1f, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER,
                             10f, new float[]{10f, 5f}, 0f));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `width` | Line width in pixels. |
| `cap` | `CAP_BUTT`, `CAP_ROUND`, `CAP_SQUARE`. |
| `join` | `JOIN_MITER`, `JOIN_ROUND`, `JOIN_BEVEL`. |
| `miterlimit` | Limit for miter joins. |
| `dash` | Dash pattern array. |
| `dashPhase` | Offset into the dash pattern. |

#### Syntax Rules

- `setStroke` is available only on `Graphics2D`.
- The stroke applies to `drawLine`, `draw(Shape)`, and shape outlines, but not to fills.

#### Constraints and Limitations

- Thick strokes are clipped at the component’s bounds.
- Dashed strokes are supported only by `BasicStroke`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dashed Border

```java
import javax.swing.*;
import java.awt.*;

public class DashedStrokePanel extends JPanel {
    public DashedStrokePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(Color.BLUE);
            g2.setStroke(new BasicStroke(3f, BasicStroke.CAP_ROUND,
                BasicStroke.JOIN_ROUND, 10f, new float[]{10f, 5f}, 0f));
            g2.drawRect(20, 20, 260, 160);
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A white panel with a thick blue dashed rectangle.

---

### Real-World Cases

- **Selection indicators** (dashed borders around selected objects).
- **Chart axes** with thick lines.
- **Custom components** with styled outlines.

---

## 20. setPaint()

### Definitions

**Core Definition.** `setPaint(Paint p)` sets the current fill source for subsequent drawing and filling operations. This method is **only** available on `Graphics2D`.

**Technical Definition.** `public abstract void setPaint(Paint p)` is declared in `Graphics2D`. `Paint` is the interface implemented by `Color`, `GradientPaint`, `LinearGradientPaint`, `RadialGradientPaint`, and `TexturePaint`. `setPaint` supersedes `setColor` for fill operations in `Graphics2D`, though `setColor` sets both the colour and the paint to the colour.

**Beginner-Friendly Explanation.** `setPaint` sets the fill style: solid colours, gradients, or textures.

---

### Purposes

- To fill shapes with gradients.
- To fill shapes with textures or images.
- To create visually rich components.
- To support theming.

---

### Syntax Rules and Structure

```java
g2.setPaint(new GradientPaint(x1, y1, Color.BLUE, x2, y2, Color.CYAN));
g2.setPaint(new RadialGradientPaint(center, radius, fractions, colors));
g2.setPaint(new TexturePaint(textureImage, anchorRect));
```

#### Constraints and Limitations

- `setPaint` applies to fills and strokes; for pure outlines, `setColor` is often sufficient.
- Gradients require distinct coordinates.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Radial Gradient Fill

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Ellipse2D;

public class RadialGradientPanel extends JPanel {
    public RadialGradientPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setPaint(new RadialGradientPaint(
                new Point(150, 150), 100,
                new float[]{0f, 1f},
                new Color[]{Color.WHITE, Color.BLUE}));
            g2.fill(new Ellipse2D.Double(50, 50, 200, 200));
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A white panel with a circle filled with a white-to-blue radial gradient.

---

### Real-World Cases

- **Gradient backgrounds** in custom panels.
- **Charts** with gradient-filled bars.
- **Themed buttons** and controls.

---

## 21. setRenderingHint()

### Definitions

**Core Definition.** `setRenderingHint(RenderingHints.Key key, Object value)` sets a rendering hint that controls how `Graphics2D` renders shapes and text. This method is **only** available on `Graphics2D`.

**Technical Definition.** `public abstract void setRenderingHint(RenderingHints.Key hintKey, Object hintValue)` is declared in `Graphics2D`. The `RenderingHints` class provides standard keys and values, including `KEY_ANTIALIASING` (`VALUE_ANTIALIAS_ON`/`OFF`), `KEY_TEXT_ANTIALIASING`, `KEY_INTERPOLATION`, `KEY_RENDERING`, `KEY_ALPHA_INTERPOLATION`, and `KEY_COLOR_RENDERING`.

**Beginner-Friendly Explanation.** `setRenderingHint` tells `Graphics2D` how much effort to spend on quality. Anti-aliasing smooths shapes; interpolation smooths scaled images; text anti-aliasing smooths text.

---

### Purposes

- To enable anti-aliasing for smooth shapes.
- To enable high-quality text rendering.
- To enable high-quality image scaling.
- To trade performance for quality or vice versa.

---

### Syntax Rules and Structure

```java
g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                    RenderingHints.VALUE_ANTIALIAS_ON);
g2.setRenderingHint(RenderingHints.KEY_INTERPOLATION,
                    RenderingHints.VALUE_INTERPOLATION_BILINEAR);
g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                    RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
```

#### Constraints and Limitations

- Anti-aliasing has performance costs; it may be undesirable for frequent repaints of large areas.
- Text anti-aliasing may not be supported on all platforms.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Anti-Aliased Circle vs. Non-Aliased

```java
import javax.swing.*;
import java.awt.*;

public class HintPanel extends JPanel {
    public HintPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.RED);
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_OFF);
            g2.drawOval(20, 50, 150, 100);                         // 1. Jagged
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.drawOval(220, 50, 150, 100);                        // 2. Smooth
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A white panel with two red ovals side by side: one with jagged edges (anti-aliasing off) and one with smooth edges (anti-aliasing on).

---

### Real-World Cases

- **Anti-aliased charts** and diagrams.
- **High-quality image scaling** for image viewers.
- **Smooth text** in custom components.

---

## 22. Graphics Best Practices

### Definitions

**Core Definition.** Graphics best practices are the conventions and patterns that ensure painting is correct, performant, thread-safe, and maintainable.

**Technical Definition.** Best practices include: working on a `Graphics2D` copy and disposing it; calling `super.paintComponent(g)`; avoiding allocations, I/O, and state mutations inside paint methods; caching fonts, strokes, paints, and images as fields; using `repaint()` to trigger painting and `revalidate()` for layout changes; using `RenderingHints` selectively; using `Shape`-based drawing for precision; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices are the “rules of thumb” that prevent slow painting, visual glitches, and lost drawings. They include things like “use a copy,” “dispose it,” “don’t allocate in paint,” and “don’t do heavy work in paint.”

---

### Purposes

- To keep painting fast and responsive.
- To avoid side effects and state leakage between painting calls.
- To ensure correct appearance across platforms.
- To support high-quality and high-DPI rendering.
- To make painting code maintainable and testable.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);                                       // 1. Super
    Graphics2D g2 = (Graphics2D) g.create();                       // 2. Copy
    try {
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                            RenderingHints.VALUE_ANTIALIAS_ON);    // 3. Hints
        g2.setColor(BAR_COLOR);                                    // 4. Cache
        g2.fillRect(0, 0, 100, 100);
    } finally {
        g2.dispose();                                              // 5. Dispose
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Copy and dispose | `Graphics2D g2 = (Graphics2D) g.create(); try { ... } finally { g2.dispose(); }` |
| Call `super.paintComponent` | Clear background and honour UI delegate |
| Cache resources | Fonts, strokes, paints, images as `static final` or instance fields |
| No allocation in paint | Avoid `new` in `paintComponent` |
| No I/O in paint | Load files and images outside paint |
| No state mutation in paint | Paint methods must be read-only |
| Rendering hints | Set selectively for quality-sensitive drawing |
| Shape-based drawing | Use `Graphics2D.draw(Shape)` and `fill(Shape)` |
| Repaint on state change | `repaint()` after mutating fields |
| EDT | Mutate state and call `repaint()` on the EDT |

#### Syntax Rules

- **Always use a `Graphics2D` copy** when modifying state, and dispose it in a `finally` block.
- **Always call `super.paintComponent(g)`** at the start unless you replace UI painting.
- **Never allocate fonts, strokes, paints, or images inside `paintComponent`**; cache them as fields.
- **Never perform I/O or blocking calls inside `paintComponent`**.
- **Never mutate application state inside `paintComponent`**; painting must be side-effect free.
- **Always use `repaint()`** to trigger painting and `revalidate()` for layout changes.
- **Always use `Shape`-based drawing** (`Rectangle2D`, `Ellipse2D`, `Path2D`) for precision and flexibility.

#### Constraints and Limitations

- Anti-aliasing and alpha compositing have performance costs.
- Very large image scaling without interpolation can produce poor results.
- Clipping must be respected; painting outside the clip is undefined.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Component

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.RoundRectangle2D;

public class BestPracticeGraphics extends JPanel {
    private static final Color BAR_COLOR = new Color(0, 120, 215);
    private static final Font LABEL_FONT = new Font("SansSerif", Font.BOLD, 14);
    private static final Stroke BORDER_STROKE = new BasicStroke(2f);

    public BestPracticeGraphics() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 250));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                                   // 1. Super
        Graphics2D g2 = (Graphics2D) g.create();                   // 2. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setPaint(new GradientPaint(50, 50, BAR_COLOR,
                                          350, 200, Color.CYAN));  // 3. Paint
            g2.fill(new RoundRectangle2D.Double(50, 50, 300, 150, 30, 30));
            g2.setStroke(BORDER_STROKE);
            g2.setColor(Color.DARK_GRAY);
            g2.draw(new RoundRectangle2D.Double(50, 50, 300, 150, 30, 30));
            g2.setFont(LABEL_FONT);                                // 4. Cached
            g2.setColor(Color.WHITE);
            g2.drawString("Best Practice", 120, 130);
        } finally {
            g2.dispose();                                          // 5. Dispose
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Graphics");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticeGraphics());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white panel with an anti-aliased rounded rectangle filled with a blue-to-cyan gradient, a dark-gray border, and the white label “Best Practice” centered inside.

**Why the code produces this result.** Fonts, colours, and strokes are cached as static fields; the graphics copy is disposed; the paint method is side-effect free; anti-aliasing is enabled selectively.

---

#### Example 2: Common Mistake — Allocating a Font Per Line

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    for (int i = 0; i < 100; i++) {
        g.setFont(new Font("SansSerif", Font.PLAIN, 12));         // WRONG
        g.drawString("Line " + i, 10, i * 15);
    }
}
```

**Expected Output.** Slow painting because a new `Font` is allocated one hundred times per paint.

**Why this is wrong.** Allocating objects in `paintComponent` is a performance anti-pattern. The font should be created once and stored as a field.

---

#### Example 3: Common Mistake — Storing the Graphics Object

```java
private Graphics savedGraphics;                                    // WRONG

@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    savedGraphics = g;                                              // 1. Store
}
```

**Expected Output.** Subsequent drawing with `savedGraphics` may not appear or may corrupt the display because the `Graphics` object is only valid during the current paint call.

**Why this is wrong.** `Graphics` objects are transient. Storing and reusing them across paint cycles causes undefined behaviour.

---

### Real-World Cases

- **Production Swing applications** cache resources, dispose copies, and avoid allocation in paint methods.
- **Custom charts and dashboards** enable anti-aliasing selectively and use `Shape`-based drawing.
- **Game canvases** repaint dirty regions and avoid I/O in paint methods.
- **Educational examples** use the best-practice skeleton to demonstrate correct painting from the start.

---

## References

Oracle. *Class Graphics*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics.html

Oracle. *Class Graphics2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html

Oracle. *Class Color*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Color.html

Oracle. *Class Font*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Font.html

Oracle. *Class FontMetrics*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/FontMetrics.html

Oracle. *Class BasicStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/BasicStroke.html

Oracle. *Class GradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/GradientPaint.html

Oracle. *Class RadialGradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/RadialGradientPaint.html

Oracle. *Class RenderingHints*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/RenderingHints.html

Oracle. *Class Polygon*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Polygon.html

Oracle. *Class Image*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Image.html

Oracle. *Performing Custom Painting*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/painting/index.html

Oracle. *Solving Common Painting Problems*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/painting/problems.html

Oracle. *Painting in AWT and Swing*. https://www.oracle.com/java/technologies/painting.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading