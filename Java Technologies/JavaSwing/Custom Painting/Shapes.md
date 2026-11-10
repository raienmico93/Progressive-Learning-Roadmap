# Java 2D Shapes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **Shape** in Java is an object that represents a geometric figure — a line, a rectangle, an ellipse, a rounded rectangle, an arc, a polygon, or an arbitrary path — and knows how to describe its geometry to the Java 2D rendering pipeline. `Shape` is the central interface of the `java.awt.geom` package; every concrete geometry in `java.awt` and `java.awt.geom` implements it.

**Technical Definition.** `public interface Shape extends Cloneable`. It declares `getBounds()`, `getBounds2D()`, `contains(double x, double y)`, `contains(Point2D p)`, `contains(Rectangle2D r)`, `intersects(Rectangle2D r)`, `intersects(double x, double y, double w, double h)`, `getPathIterator(AffineTransform at)`, `getPathIterator(AffineTransform at, double flatness)`. The interface provides **rectangular bounds**, **containment testing**, **intersection testing**, and **path iteration**. Shapes can be rendered by `Graphics2D` via `draw(Shape)` (outline) and `fill(Shape)` (interior), and converted to `Area` for boolean operations. The abstract geometric subclasses (`Line2D`, `Rectangle2D`, `Ellipse2D`, `RoundRectangle2D`, `Arc2D`, `Path2D`, `QuadCurve2D`, `CubicCurve2D`) each provide `Float` and `Double` inner classes for precision-specific storage; legacy classes (`Rectangle`, `Polygon`) also implement `Shape` but use integer coordinates.

**Beginner-Friendly Explanation.** A **Shape** is Java's way of representing a geometric figure as an **object** rather than as a sequence of drawing calls. Instead of writing `g.drawRect(10, 10, 100, 50)` and hoping you never need to move that rectangle, you create a `Rectangle2D` object, hand it to `Graphics2D`, and can then **reuse it, transform it, hit-test against it, intersect it with other shapes, and combine it with `Area`**. Shapes form a rich hierarchy: `Line2D` is a line, `Rectangle2D` is a rectangle, `Ellipse2D` is an ellipse, `RoundRectangle2D` has rounded corners, `Arc2D` is an arc or pie-slice, `Polygon` is a closed polygon, and `Path2D` is an arbitrary sequence of move/line/curve operations. Think of `Shape` as the language Java uses to describe "what a thing looks like," and `Graphics2D` as the tool that paints it.

---

### Key Characteristics

1. **Interface-based geometry.** Every shape is an object implementing the `Shape` interface, giving a uniform API for rendering, hit-testing, bounds computation, and path iteration.
2. **Float/double precision.** The abstract shape classes (`Line2D`, `Rectangle2D`, `Ellipse2D`, `RoundRectangle2D`, `Arc2D`, `Path2D`, curves) provide `Float` and `Double` subclasses. Precision is a per-shape choice.
3. **Immutable or mutable.** Most shapes are mutable (setters exist for coordinates and dimensions); `Area` is effectively immutable once constructed but exposes boolean operations that return new shapes.
4. **Boolean geometry.** `Area` provides set operations (`add`, `subtract`, `intersect`, `exclusiveOr`) for combining, subtracting, or intersecting shapes, and can be transformed with an `AffineTransform`.
5. **Rendering pipeline.** `Graphics2D.draw(Shape)` uses the current `Stroke` to compute the outline; `Graphics2D.fill(Shape)` fills the interior according to the shape's winding rule.
6. **Containment and intersection.** `contains` methods allow hit-testing (e.g., did the user click on this shape?), and `intersects` allows collision detection.
7. **Path iteration and winding rules.** `PathIterator` exposes the shape's geometry as a sequence of segments; the winding rule (`WIND_EVEN_ODD` or `WIND_NON_ZERO`) determines what "inside" means for self-intersecting paths.
8. **Transformation.** Shapes can be transformed via `AffineTransform.createTransformedShape(...)`, or by transforming the `Graphics2D` context before drawing.
9. **Screen vs. geometric coordinates.** Shapes exist in a geometric coordinate system; `getBounds()` returns integer bounds, while `getBounds2D()` returns `Rectangle2D` with the exact float/double extents.
10. **Thread and lifetime rules.** Shape objects are simple data holders; they can be shared safely across threads if not mutated. But painting must still occur on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, anonymous inner classes, and lambda expressions.
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and the `create()`/`dispose()` idiom.
- **Coordinate system:** origin at top-left, `x` increases rightward, `y` downward.
- **Basic geometry:** points, line segments, rectangles, ellipses, arcs, polygons, and curved paths.
- **`java.awt.geom` package:** the package in which the shapes live.
- **`AffineTransform`:** needed for transforming and rotating shapes.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`java.awt.Graphics2D`** — renders shapes via `draw` and `fill`.
- **`java.awt.geom.AffineTransform`** — transformation matrices.
- **`java.awt.geom.Area`** — set-based boolean operations on shapes.
- **`java.awt.geom.Path2D` and `PathIterator`** — arbitrary paths and iteration.
- **`java.awt.Polygon`** — legacy int-based polygon.
- **`java.awt.Rectangle`, `java.awt.geom.RectangularShape`** — bounds-only geometry.
- **`java.awt.geom.RectangularShape`** — abstract superclass of `Rectangle2D`, `Ellipse2D`, `RoundRectangle2D`, `Arc2D`.
- **`java.awt.Stroke`** — determines the outline of a shape.
- **`java.awt.BasicStroke`** — the standard stroke implementation.
- **Hit testing and picking** — `Shape.contains(Point2D)` for interactive editors.

---

## Core Concepts and Features

---

## 1. Shape — The Interface

### Definitions

**Core Definition.** `Shape` is the interface that defines the geometric contract for all Java 2D shapes: bounds, containment, intersection, and path iteration.

**Technical Definition.** `public interface Shape extends Cloneable`. It declares methods for bounding boxes (`getBounds`, `getBounds2D`), containment (`contains` overloads), intersection (`intersects` overloads), and path iteration (`getPathIterator`). Every geometric shape in `java.awt` and `java.awt.geom` implements `Shape`.

**Beginner-Friendly Explanation.** `Shape` is the **“what does this figure look like”** contract. Everything a `Graphics2D` can draw or fill, everything `Area` can combine, and everything you can hit-test with `contains` implements `Shape`.

---

### Purposes

- To provide a uniform interface for all geometric figures in Java 2D.
- To enable rendering, hit-testing, bounds computation, and path iteration through a single contract.
- To support boolean operations (`Area`) and transformation (`AffineTransform`) on arbitrary shapes.
- To allow precise float/double geometry alongside legacy integer shapes.
- To enable animation, collision detection, and interaction on geometric primitives.

---

### Syntax Rules and Structure

#### Complete General Syntax (Using Shape)

```java
Shape s = new Rectangle2D.Double(10, 10, 100, 50);

Rectangle bounds = s.getBounds();                 // integer bounds
Rectangle2D bounds2D = s.getBounds2D();           // exact bounds

boolean inside = s.contains(50, 30);              // hit test
boolean overlaps = s.intersects(0, 0, 20, 20);    // intersection test

PathIterator it = s.getPathIterator(null);        // iterate geometry

g2.draw(s);                                       // outline
g2.fill(s);                                       // interior
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getBounds()` | Returns the smallest integer `Rectangle` containing the shape. |
| `getBounds2D()` | Returns the smallest `Rectangle2D` containing the shape. |
| `contains(double, double)` | Whether the given point is inside the shape. |
| `contains(Point2D)` | Whether the given point is inside the shape. |
| `contains(Rectangle2D)` | Whether the entire rectangle is inside the shape. |
| `intersects(double, double, double, double)` | Whether the shape intersects the rectangle. |
| `intersects(Rectangle2D)` | Whether the shape intersects the rectangle. |
| `getPathIterator(AffineTransform)` | Iterates the shape's path. |

#### Syntax Rules

- `getBounds()` returns integer bounds; use `getBounds2D()` for precision.
- `contains` respects the winding rule for complex shapes.
- `getPathIterator` may be called with `null` to indicate no transform.

#### Constraints and Limitations

- `Shape` implementations in legacy classes (`Rectangle`, `Polygon`) use integer coordinates; floating-point shapes are needed for precision.
- `contains` can be slow for very complex shapes; `Area` optimises some cases.
- Path iteration returns the shape's segments, which may involve curve approximations.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using Shape Polymorphically

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.*;
import java.util.ArrayList;
import java.util.List;

public class ShapePolymorphismPanel extends JPanel {
    private final List<Shape> shapes = new ArrayList<>();

    public ShapePolymorphismPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 400));

        shapes.add(new Rectangle2D.Double(30, 30, 120, 80));      // 1. Rect
        shapes.add(new Ellipse2D.Double(180, 30, 120, 80));        // 2. Ellipse
        shapes.add(new RoundRectangle2D.Double(330, 30, 120, 80, 30, 30));
        shapes.add(new Line2D.Double(30, 140, 450, 140));
        shapes.add(new Arc2D.Double(30, 160, 100, 100, 0, 90, Arc2D.PIE));
        shapes.add(new Path2D.Double() {{
            moveTo(160, 160); lineTo(260, 260); lineTo(160, 260); closePath();
        }});                                                        // 3. Path
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setStroke(new BasicStroke(2f));
            g2.setColor(Color.BLUE);
            for (Shape s : shapes) {                              // 4. Iterate
                g2.draw(s);
            }

            Point click = new Point(100, 60);
            g2.setColor(Color.RED);
            for (Shape s : shapes) {
                if (s.contains(click)) {                          // 5. Hit test
                    g2.fill(s);
                }
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shape Polymorphism");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new ShapePolymorphismPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white panel with six outlined shapes of different types. The rectangle containing the point (100, 60) is filled red, demonstrating that the same `Shape` API works for every geometric figure.

**Why the code produces this result.** A `List<Shape>` holds rectangles, ellipses, rounded rectangles, lines, arcs, and paths. The rendering loop iterates over the list and calls `g2.draw(s)` on each. The hit-test loop calls `s.contains(click)` — the same method on every shape.

---

### Real-World Cases

- **Diagram editors** storing heterogeneous shapes in a `List<Shape>`.
- **Collision detection** in games using `contains` and `intersects`.
- **Charts and infographics** rendering compound shapes.
- **Icon systems** using `Path2D` for arbitrary glyphs.
- **Custom UI components** hit-testing for interactive regions.

---

## 2. Line2D

### Definitions

**Core Definition.** `Line2D` is an abstract class representing a line segment between two points. Its concrete subclasses are `Line2D.Float` and `Line2D.Double`.

**Technical Definition.** `public abstract class Line2D extends Object implements Shape, Cloneable`. Subclasses: `Line2D.Float` (stores coordinates as `float`) and `Line2D.Double` (stores coordinates as `double`). Methods include `getP1()`, `getP2()`, `setLine(double x1, double y1, double x2, double y2)`, `setLine(Point2D p1, Point2D p2)`, `ptLineDist(...)`, `ptSegDist(...)`, and the inherited `Shape` methods. A line has zero area; `fill` on a line does nothing visible.

**Beginner-Friendly Explanation.** `Line2D` is a **line segment**. You give it two endpoints; it can be drawn, tested for proximity, and used as a building block for diagrams. A line has no interior, so `fill` has no visible effect on it.

---

### Purposes

- To represent a straight line segment as a reusable geometric object.
- To compute the distance from a point to a line or segment.
- To test intersection with other shapes (a line can intersect other shapes).
- To serve as the geometric backbone of arrows, connectors, and chart axes.
- To enable precise collision detection in games and diagrams.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Line2D line = new Line2D.Double(x1, y1, x2, y2);
Line2D line2 = new Line2D.Float(x1f, y1f, x2f, y2f);

double dist = line.ptSegDist(px, py);          // point-to-segment distance
double dline = line.ptLineDist(px, py);         // point-to-line distance

g2.draw(line);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `Line2D.Double(x1, y1, x2, y2)` | Creates a line with `double` precision. |
| `Line2D.Float(x1f, y1f, x2f, y2f)` | Creates a line with `float` precision. |

| Method | Meaning |
|--------|---------|
| `getP1()` / `getP2()` | The two endpoints as `Point2D`. |
| `setLine(...)` | Updates the endpoints. |
| `ptSegDist(px, py)` | Distance from the point to the segment. |
| `ptLineDist(px, py)` | Distance from the point to the infinite line. |

#### Syntax Rules

- A line has no interior; `fill` has no visible effect.
- `contains` returns `false` for essentially all points (a line has zero area).
- `ptSegDist` respects the segment endpoints; `ptLineDist` extends the line infinitely.

#### Constraints and Limitations

- Lines are not usefully fillable; use `BasicStroke` and `draw`.
- For thick lines, the outline depends on the current `Stroke`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Line Diagram with Distance-Based Highlight

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.Line2D;

public class LinePanel extends JPanel {
    private final Line2D line = new Line2D.Double(50, 50, 400, 250);
    private Point mouse = new Point(-1, -1);

    public LinePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(450, 300));
        addMouseMotionListener(new MouseMotionAdapter() {
            @Override public void mouseMoved(MouseEvent e) {
                mouse = e.getPoint();
                repaint();
            }
        });
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            double d = line.ptSegDist(mouse);                        // 1. Distance
            g2.setStroke(new BasicStroke(d < 10 ? 5f : 2f));          // 2. Thicken
            g2.setColor(d < 10 ? Color.RED : Color.BLUE);             // 3. Highlight
            g2.draw(line);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Line Proximity");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new LinePanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A blue line segment from (50, 50) to (400, 250). When the mouse is within 10 pixels of the line, it becomes a thick red line.

**Why the code produces this result.** `ptSegDist` computes the shortest distance from the mouse to the line segment; when the distance is below the threshold, the stroke widens and the colour changes.

---

### Real-World Cases

- **Diagram connectors** between nodes.
- **Chart axes** and grid lines.
- **Vector-style drawing tools** where users sketch with thick lines.
- **Game physics** (raycasting, segment intersections).
- **Proximity highlighting** in interactive canvases.

---

## 3. Rectangle2D

### Definitions

**Core Definition.** `Rectangle2D` is an abstract class representing a rectangle specified by an upper-left corner and a width/height. Its concrete subclasses are `Rectangle2D.Float` and `Rectangle2D.Double`. The legacy `java.awt.Rectangle` also implements `Shape` but uses integer coordinates.

**Technical Definition.** `public abstract class Rectangle2D extends RectangularShape`. Methods include `setRect(x, y, w, h)`, `intersectsLine(Line2D)`, `outcode(double, double)`, `createIntersection(...)`, `createUnion(...)`, and the inherited `Shape` methods. `Rectangle` is a subclass of `Rectangle2D` with integer precision.

**Beginner-Friendly Explanation.** `Rectangle2D` is a **rectangle** with floating-point precision. It is the most common shape for hit-testing, laying out, and clipping.

---

### Purposes

- To represent axis-aligned rectangles with float precision.
- To serve as clipping regions for `Graphics2D`.
- To perform hit-testing and intersection tests.
- To serve as the bounds of other shapes.
- To be the standard shape for panels, buttons, and layout metrics.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Rectangle2D rect = new Rectangle2D.Double(x, y, w, h);
Rectangle2D rect2 = new Rectangle2D.Float(x, y, w, h);

boolean hit = rect.contains(px, py);
boolean overlap = rect.intersects(x, y, w, h);
Rectangle2D inter = Rectangle2D.intersect(rect, other);       // static helper
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `Rectangle2D.Double(x, y, w, h)` | Double-precision rectangle. |
| `Rectangle2D.Float(x, y, w, h)` | Float-precision rectangle. |
| `Rectangle2D.Float()` | Empty rectangle at (0, 0). |

| Method | Meaning |
|--------|---------|
| `setRect(x, y, w, h)` | Sets the rectangle. |
| `intersectsLine(Line2D l)` | Whether this rectangle intersects a line. |
| `createIntersection(Rectangle2D)` | The intersection rectangle. |
| `createUnion(Rectangle2D)` | The union rectangle. |

#### Syntax Rules

- The rectangle spans `(x, y)` to `(x + w, y + h)`. Coordinates are inclusive of the origin; the extent is `w` × `h` (unlike `Graphics.fillRect`, which includes the last pixel).

#### Constraints and Limitations

- Coordinates must be non-negative for width and height; negative dimensions may produce unexpected results.
- For rotated rectangles, use `AffineTransform.createTransformedShape`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Rectangles with Hit-Testing

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.Rectangle2D;

public class RectanglesPanel extends JPanel {
    private final Rectangle2D[] rects = {
        new Rectangle2D.Double(50, 50, 120, 80),
        new Rectangle2D.Double(200, 50, 120, 80),
        new Rectangle2D.Double(350, 50, 120, 80)
    };
    private Point mouse = new Point(-1, -1);

    public RectanglesPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(520, 200));
        addMouseMotionListener(new MouseMotionAdapter() {
            @Override public void mouseMoved(MouseEvent e) {
                mouse = e.getPoint();
                repaint();
            }
        });
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            for (Rectangle2D r : rects) {
                if (r.contains(mouse)) {                             // 1. Hit test
                    g2.setColor(new Color(0, 120, 215, 80));
                    g2.fill(r);
                }
                g2.setColor(Color.DARK_GRAY);
                g2.setStroke(new BasicStroke(2f));
                g2.draw(r);
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Rectangles");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new RectanglesPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three dark-gray outlined rectangles. When the mouse hovers over a rectangle, it fills with a translucent blue.

**Why the code produces this result.** `contains(Point)` returns `true` if the mouse is inside the rectangle; the filled blue is drawn behind the outline.

---

### Real-World Cases

- **Hit-testing** for buttons, panels, and canvases.
- **Clipping regions** via `g2.setClip(rect)`.
- **Layout metrics** — every `Component`'s bounds is a rectangle.
- **Chart plot areas**.
- **Collision detection** for axis-aligned bounding boxes.

---

## 4. Ellipse2D

### Definitions

**Core Definition.** `Ellipse2D` is an abstract class representing an ellipse inscribed in a bounding rectangle. Its concrete subclasses are `Ellipse2D.Float` and `Ellipse2D.Double`.

**Technical Definition.** `public abstract class Ellipse2D extends RectangularShape`. Methods include `getX()`, `getY()`, `getWidth()`, `getHeight()`, `setFrame(...)`, and the inherited `Shape` methods. A circle is an ellipse where width equals height.

**Beginner-Friendly Explanation.** `Ellipse2D` is an **ellipse** or **circle**. The ellipse is inscribed in the bounding rectangle: its major and minor axes touch the midpoints of the rectangle's sides.

---

### Purposes

- To represent ellipses and circles as precise geometry.
- To serve as nodes in graph diagrams.
- To draw gauges, dials, and circular indicators.
- To be combined with `Area` for boolean operations and masks.
- To fill circular panels and decorations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Ellipse2D e = new Ellipse2D.Double(x, y, w, h);
Ellipse2D circle = new Ellipse2D.Double(cx - r, cy - r, 2 * r, 2 * r);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `Ellipse2D.Double(x, y, w, h)` | Ellipse inscribed in the bounding rectangle. |
| `Ellipse2D.Float(...)` | Float-precision version. |

#### Syntax Rules

- The ellipse is inscribed in `(x, y, w, h)`; `w == h` produces a circle.
- `contains(Point2D)` respects the ellipse's actual curved boundary, not just its bounding box.

#### Constraints and Limitations

- Ellipse2D cannot represent rotated ellipses directly; apply an `AffineTransform`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Circular Progress Ring

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Arc2D;
import java.awt.geom.Ellipse2D;

public class CircularProgressPanel extends JPanel {
    private double progress = 0.0;

    public CircularProgressPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));
        new Timer(50, e -> {
            progress = (progress + 0.01) % 1.0;
            repaint();
        }).start();
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            int cx = 150, cy = 150, r = 100;
            g2.setColor(Color.LIGHT_GRAY);
            g2.fill(new Ellipse2D.Double(cx - r, cy - r, 2 * r, 2 * r));   // 1. Background

            g2.setColor(new Color(0, 160, 0));
            g2.fill(new Arc2D.Double(cx - r, cy - r, 2 * r, 2 * r,
                                     90, -360 * progress, Arc2D.PIE));     // 2. Arc
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Circular Progress");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new CircularProgressPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A light-gray circle with a green pie-slice arc that rotates as `progress` cycles from 0 to 1 and back.

**Why the code produces this result.** `Ellipse2D` draws the grey background circle. `Arc2D` with `Arc2D.PIE` draws a pie slice from the top going clockwise (negative angle) for `progress` of the full circle.

---

### Real-World Cases

- **Graph nodes** (circles with labels).
- **Gauges and dials** (concentric ellipses).
- **Indicators** and status dots.
- **Image masks** (elliptical vignettes).
- **Custom chart markers** (circular data points).

---

## 5. RoundRectangle2D

### Definitions

**Core Definition.** `RoundRectangle2D` is an abstract class representing a rectangle with rounded corners. Its concrete subclasses are `RoundRectangle2D.Float` and `RoundRectangle2D.Double`.

**Technical Definition.** `public abstract class RoundRectangle2D extends RectangularShape`. The rounded corners are defined by `arcWidth` (horizontal diameter of the corner arcs) and `arcHeight` (vertical diameter). Methods include `setRoundRect(x, y, w, h, arcw, arch)`, `getArcWidth()`, `getArcHeight()`, and the inherited `Shape` methods.

**Beginner-Friendly Explanation.** `RoundRectangle2D` is a **rounded rectangle** — a rectangle with softened corners. Larger `arcWidth` and `arcHeight` values mean more rounded corners.

---

### Purposes

- To draw modern UI elements with rounded corners.
- To create card-style and pill-shaped controls.
- To render buttons and panels with a softened, branded look.
- To be used as a clipping shape for custom drawing.
- To construct rich, anti-aliased UI visuals.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
RoundRectangle2D rr = new RoundRectangle2D.Double(x, y, w, h, arcW, arcH);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x, y` | Upper-left corner. |
| `w, h` | Size of the rectangle. |
| `arcW, arcH` | Diameters of the corner arcs; larger values round more. |

#### Syntax Rules

- `arcW` and `arcH` are diameters, not radii: a corner with `arcW = 30` has a 15-pixel horizontal radius.
- `arcW` and `arcH` are clamped to the rectangle's dimensions.

#### Constraints and Limitations

- Extreme `arcW` / `arcH` produce elliptical or capsule shapes.
- Anti-aliasing is highly recommended for smooth corners.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Card with Gradient and Rounded Corners

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.RoundRectangle2D;

public class RoundedCardPanel extends JPanel {
    public RoundedCardPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 250));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            RoundRectangle2D card =
                new RoundRectangle2D.Double(50, 50, 300, 150, 30, 30);   // 1. Card
            g2.setPaint(new GradientPaint(50, 50, new Color(0, 120, 215),
                                          350, 200, new Color(0, 200, 100)));
            g2.fill(card);
            g2.setColor(Color.DARK_GRAY);
            g2.setStroke(new BasicStroke(2f));
            g2.draw(card);
            g2.setColor(Color.WHITE);
            g2.setFont(new Font("SansSerif", Font.BOLD, 20));
            g2.drawString("Welcome", 140, 135);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Rounded Card");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new RoundedCardPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white panel with a blue-to-green gradient rounded card, a dark-gray border, and white "Welcome" text.

---

### Real-World Cases

- **Card-style UIs** (dashboards, profile cards).
- **Rounded buttons** with gradient fills.
- **Notifications and toasts**.
- **Tooltips** with rounded corners.

---

## 6. Arc2D

### Definitions

**Core Definition.** `Arc2D` is an abstract class representing a segment of an ellipse inscribed in a bounding rectangle. Its concrete subclasses are `Arc2D.Float` and `Arc2D.Double`. Three arc types are supported: `OPEN`, `CHORD`, and `PIE`.

**Technical Definition.** `public abstract class Arc2D extends RectangularShape`. Constructor: `Arc2D.Double(x, y, w, h, start, extent, type)`. `start` is the start angle in **degrees**; `extent` is the sweep angle in degrees (positive counterclockwise); `type` is `OPEN`, `CHORD`, or `PIE`. `PIE` and `CHORD` are fillable; `OPEN` is not fillable (it has no interior).

**Beginner-Friendly Explanation.** `Arc2D` is a **segment of an ellipse**. It has three shapes: **OPEN** (just the curved outline), **CHORD** (the curved outline plus a straight chord closing it), and **PIE** (the curved outline plus two radii to the centre). Only `CHORD` and `PIE` are fillable.

---

### Purposes

- To draw arcs, pie slices, and semi-circles.
- To implement gauges, dials, and progress rings.
- To draw pie charts and donut charts.
- To draw decorative curved shapes.
- To model curved portions of composite designs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Arc2D arc = new Arc2D.Double(x, y, w, h, startDegrees, extentDegrees, Arc2D.PIE);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x, y, w, h` | Bounding rectangle. |
| `startDegrees` | Start angle, 0° at 3 o'clock, positive counterclockwise. |
| `extentDegrees` | Sweep angle; positive counterclockwise, negative clockwise. |
| `type` | `OPEN`, `CHORD`, or `PIE`. |

#### Syntax Rules

- Angles are in **degrees**.
- `PIE` includes the radii; `CHORD` includes the chord; `OPEN` includes neither.
- Full circle: `extent = 360`.

#### Constraints and Limitations

- `OPEN` arcs have no fillable interior.
- Angles are measured from the positive x-axis in standard math orientation (counterclockwise), not screen-clockwise.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Pie Chart with Three Slices

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Arc2D;

public class PieChartPanel extends JPanel {
    private final double[] values = {40, 30, 20, 10};
    private final Color[] colors = {Color.RED, Color.GREEN, Color.BLUE, Color.ORANGE};

    public PieChartPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 400));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            double total = 0;
            for (double v : values) total += v;
            double start = 90;                                       // 1. Start top
            for (int i = 0; i < values.length; i++) {
                double extent = -360 * values[i] / total;            // 2. Slice
                g2.setColor(colors[i]);
                g2.fill(new Arc2D.Double(50, 50, 300, 300,
                                         start, extent, Arc2D.PIE));
                start += extent;
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Pie Chart");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new PieChartPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A pie chart with four slices (red, green, blue, orange) filling a circle from the top clockwise.

**Why the code produces this result.** Each `Arc2D.PIE` slice starts at the running `start` angle and sweeps `extent` degrees clockwise (negative). Slices accumulate to fill the full circle.

---

### Real-World Cases

- **Pie and donut charts**.
- **Gauge indicators** (progress rings).
- **Dials and clocks**.
- **Decorative arcs** in custom UI.

---

## 7. Polygon

### Definitions

**Core Definition.** `Polygon` is a legacy class representing a closed polygon with integer coordinates. It is not part of `java.awt.geom`, but it implements `Shape`.

**Technical Definition.** `public class Polygon extends Object implements Shape, Cloneable`. It stores an array of `x` coordinates and an array of `y` coordinates. Methods include `addPoint(int x, int y)`, `contains(int x, int y)`, `contains(Point p)`, `getBounds()`, `getBoundingBox()`, and the inherited `Shape` methods. A `Polygon` is closed automatically.

**Beginner-Friendly Explanation.** `Polygon` is a **closed polygon** built from integer points. It is the legacy alternative to `Path2D`; new code should prefer `Path2D.Double` for float precision and richer curve support.

---

### Purposes

- To represent closed polygons with integer precision.
- To draw polygons, stars, and custom closed shapes.
- To fill arbitrary polygonal regions.
- To support hit-testing (`contains`) inside polygons.
- To be drawn by `Graphics2D.draw` and `Graphics2D.fill`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Polygon p = new Polygon();
p.addPoint(x1, y1);
p.addPoint(x2, y2);
p.addPoint(x3, y3);

Polygon p2 = new Polygon(int[] xs, int[] ys, int n);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addPoint(int, int)` | Appends a vertex. |
| `Polygon(int[], int[], int)` | Creates from coordinate arrays. |
| `contains(int, int)` | Hit test. |

#### Syntax Rules

- The polygon closes automatically; the last point connects to the first.
- Coordinates are `int` only; use `Path2D.Double` for float precision.

#### Constraints and Limitations

- Self-intersecting polygons use the even-odd fill rule by default; use `Path2D` with explicit winding for finer control.
- Integer coordinates cause precision loss under transforms.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Filled Star

```java
import javax.swing.*;
import java.awt.*;

public class StarPolygonPanel extends JPanel {
    public StarPolygonPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 400));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            int cx = 200, cy = 200, R = 150, r = 60;
            Polygon star = new Polygon();                            // 1. Polygon
            for (int i = 0; i < 10; i++) {
                double a = -Math.PI / 2 + i * Math.PI / 5;
                int radius = (i % 2 == 0) ? R : r;
                star.addPoint(
                    (int) (cx + radius * Math.cos(a)),
                    (int) (cy + radius * Math.sin(a)));
            }
            g2.setColor(Color.YELLOW);
            g2.fill(star);                                            // 2. Fill
            g2.setColor(Color.ORANGE);
            g2.setStroke(new BasicStroke(2f));
            g2.draw(star);                                            // 3. Outline
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Star Polygon");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new StarPolygonPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A yellow star with an orange outline, centred in the panel.

---

### Real-World Cases

- **Star ratings** and decorative icons.
- **Map regions** and territory shapes.
- **Custom chart polygons**.
- **Hit-testing game objects** with arbitrary polygonal shapes.

---

## 8. Area

### Definitions

**Core Definition.** `Area` is a class that provides **set-based boolean operations** on shapes — combine (`add`), subtract (`subtract`), intersect (`intersect`), and exclusive-or (`exclusiveOr`) — and can be transformed by an `AffineTransform`. `Area` internally represents its geometry as sequences of non-intersecting closed sub-paths.

**Technical Definition.** `public class Area extends Object implements Shape, Cloneable`. Constructor: `Area(Shape s)`. Methods include `add(Area a)`, `subtract(Area a)`, `intersect(Area a)`, `exclusiveOr(Area a)`, `isPolygonal()`, `isRectangular()`, `isSingular()`, `isEmpty()`, `transform(AffineTransform)`, `createTransformedArea(AffineTransform)`, and the inherited `Shape` methods. The geometry stored in an `Area` is guaranteed to be non-self-intersecting, which makes `contains` and boolean operations efficient and correct.

**Beginner-Friendly Explanation.** `Area` is a **geometry calculator** for shapes. Give it two shapes, and it produces a new shape representing their union, difference, intersection, or XOR. It's the way to build "letter-shaped" cutouts, crop an image to a shape, or draw complex compound icons.

---

### Purposes

- To combine multiple shapes into a single compound shape.
- To subtract one shape from another (e.g., a hole in a panel).
- To intersect shapes (e.g., the overlap region of two circles).
- To compute XOR regions for creative geometry.
- To create complex shapes from simple ones.
- To crop rendering to irregular regions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Area a = new Area(shape1);
a.add(new Area(shape2));                 // union
a.subtract(new Area(shape3));            // difference
a.intersect(new Area(shape4));           // intersection
a.exclusiveOr(new Area(shape5));         // XOR

boolean empty = a.isEmpty();
boolean polygonal = a.isPolygonal();

Area transformed = a.createTransformedArea(new AffineTransform(...));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `add(Area)` | Union of the areas. |
| `subtract(Area)` | Difference (this minus other). |
| `intersect(Area)` | Intersection. |
| `exclusiveOr(Area)` | Symmetric difference. |
| `isEmpty()` | Whether the area is empty. |
| `createTransformedArea(AffineTransform)` | Returns a new transformed `Area`. |

#### Syntax Rules

- `Area` operations mutate the `Area` and return `void`.
- `contains` and other `Shape` methods work on the resulting geometry.
- The result of `Area` operations is guaranteed non-self-intersecting.

#### Constraints and Limitations

- `Area` operations can be computationally expensive for very complex shapes; cache results when reusing.
- The geometry inside `Area` may not be representable by a simpler shape; drawing uses `Area`'s own path iteration.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Ring by Subtracting Circles

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Area;
import java.awt.geom.Ellipse2D;

public class RingPanel extends JPanel {
    public RingPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 400));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            Area ring = new Area(new Ellipse2D.Double(50, 50, 300, 300));  // 1. Outer
            ring.subtract(new Area(
                new Ellipse2D.Double(120, 120, 160, 160)));                // 2. Hole

            g2.setColor(new Color(0, 120, 215));
            g2.fill(ring);                                                  // 3. Fill
            g2.setColor(Color.DARK_GRAY);
            g2.setStroke(new BasicStroke(2f));
            g2.draw(ring);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Ring");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new RingPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A blue ring with a dark-gray outline: a large circle with a smaller circle subtracted from its centre.

**Why the code produces this result.** `subtract` removes the smaller ellipse from the larger one, producing a ring geometry. `fill` paints the interior of the ring only.

---

#### Example 2: Letter-Shaped Cutout

```java
Area letterA = new Area(new Polygon(new int[]{100, 200, 150}, new int[]{250, 250, 100}, 3));
letterA.subtract(new Area(new Rectangle2D.Double(130, 200, 20, 50)));
letterA.subtract(new Area(new Rectangle2D.Double(150, 200, 20, 50)));
g2.fill(letterA);
```

**Expected Output.** A triangular shape with two rectangular holes, resembling the letter A.

---

### Real-World Cases

- **Ring and donut visualisations**.
- **Masks and vignettes** for images.
- **Logo construction** from primitives.
- **Cutout UI** (panels with shaped holes).
- **Hit-testing complex regions**.

---

## 9. GeneralPath

### Definitions

**Core Definition.** `GeneralPath` is the **legacy** path class (since Java 1.2) representing an arbitrary geometric path built from move, line, and curve segments. It is **deprecated in favour of `Path2D`** (Java 6+). New code should use `Path2D.Double` or `Path2D.Float`.

**Technical Definition.** `public final class GeneralPath extends Path2D.Float`. It is a concrete subclass of `Path2D.Float` that exists for backward compatibility. It adds no new features. The `Path2D` hierarchy (`Path2D.Float` and `Path2D.Double`) provides the modern API.

**Beginner-Friendly Explanation.** `GeneralPath` is the **old name** for what is now `Path2D.Float`. New code should use `Path2D.Double` (or `Path2D.Float`) instead; `GeneralPath` still works but is legacy.

---

### Purposes

- **Legacy compatibility:** To support code written before Java 6.
- **Migration:** To help recognise old code that should be replaced with `Path2D`.
- **Same semantics as `Path2D.Float`:** It offers the same move/line/curve/close operations.

---

### Syntax Rules and Structure

#### Complete General Syntax (Legacy vs. Modern)

```java
// Legacy (avoid in new code):
GeneralPath gp = new GeneralPath();
gp.moveTo(10, 10);
gp.lineTo(100, 100);
gp.closePath();

// Modern equivalent:
Path2D.Double gp = new Path2D.Double();
gp.moveTo(10, 10);
gp.lineTo(100, 100);
gp.closePath();
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `GeneralPath()` | Creates an empty `Path2D.Float` path. |
| `moveTo`, `lineTo`, `closePath` | Same as `Path2D`. |

#### Syntax Rules

- `GeneralPath` uses `float` precision (it extends `Path2D.Float`).
- The API is identical to `Path2D.Float`'s API.

#### Constraints and Limitations

- Marked deprecated; use `Path2D.Double` for modern code.
- Precision is `float`-only; use `Path2D.Double` for `double` precision.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Triangle via GeneralPath

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.GeneralPath;

public class GeneralPathPanel extends JPanel {
    public GeneralPathPanel() {
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
            GeneralPath gp = new GeneralPath();                       // 1. Legacy
            gp.moveTo(150, 50);
            gp.lineTo(250, 250);
            gp.lineTo(50, 250);
            gp.closePath();
            g2.setColor(Color.BLUE);
            g2.fill(gp);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("GeneralPath");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new GeneralPathPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A blue triangle. (The example works, but the modern equivalent would use `Path2D.Double`.)

---

### Real-World Cases

- **Legacy applications** that need maintenance but not migration.
- **Understanding old tutorials and code samples**.

---

## 10. Path2D

### Definitions

**Core Definition.** `Path2D` is the **modern** abstract class for arbitrary geometric paths. It supports move, line, quadratic, and cubic Bézier operations, plus append and close operations. Concrete subclasses: `Path2D.Float` and `Path2D.Double`.

**Technical Definition.** `public abstract class Path2D extends Object implements Shape, Cloneable`. Methods include `moveTo(double x, double y)`, `lineTo(double x, double y)`, `quadTo(double x1, double y1, double x2, double y2)`, `curveTo(double x1, double y1, double x2, double y2, double x3, double y3)`, `closePath()`, `append(Shape s, boolean connect)`, `transform(AffineTransform)`, `getCurrentPoint()`, `getWindingRule()`, `setWindingRule(int)`, `reset()`, and the inherited `Shape` methods. It also provides `Path2D.Double(Shape s, AffineTransform at)` for construction from a shape and a transform.

**Beginner-Friendly Explanation.** `Path2D` is the **modern drawing pen**. It records a sequence of "move here," "draw a straight line to there," "draw a curve," and "close the path." It can produce arbitrary shapes: triangles, stars, arrows, icons, and complex illustrations.

---

### Purposes

- To construct arbitrary geometric paths from simple commands.
- To create polygons, stars, arrows, and custom icons.
- To append existing shapes into a path.
- To support Bézier curves for smooth geometry.
- To control fill behaviour via the winding rule (`WIND_EVEN_ODD` or `WIND_NON_ZERO`).
- To serve as the modern replacement for `GeneralPath`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Path2D path = new Path2D.Double();
path.moveTo(x, y);
path.lineTo(x2, y2);
path.quadTo(cx, cy, x3, y3);            // quadratic curve
path.curveTo(cx1, cy1, cx2, cy2, x4, y4); // cubic curve
path.closePath();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `moveTo(x, y)` | Starts a new sub-path at `(x, y)`. |
| `lineTo(x, y)` | Adds a line segment to `(x, y)`. |
| `quadTo(cx, cy, x, y)` | Adds a quadratic Bézier curve. |
| `curveTo(cx1, cy1, cx2, cy2, x, y)` | Adds a cubic Bézier curve. |
| `closePath()` | Connects the last point to the start of the current sub-path. |
| `append(Shape, boolean)` | Appends another shape's geometry; `connect` joins with a line. |
| `setWindingRule(int)` | `WIND_EVEN_ODD` or `WIND_NON_ZERO`. |

#### Syntax Rules

- A path must begin with `moveTo`.
- Subsequent segments continue from the current point.
- The path can have multiple sub-paths; `moveTo` begins a new one.
- The winding rule affects how self-intersecting paths are filled.

#### Constraints and Limitations

- Paths are mutable; mutating a path used as a key or in a set breaks invariants.
- Very complex paths can be slow to render; simplify or cache as a `BufferedImage` if necessary.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Curved Arrow with Path2D

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Path2D;

public class ArrowPanel extends JPanel {
    public ArrowPanel() {
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
            Path2D arrow = new Path2D.Double();                       // 1. Path
            arrow.moveTo(50, 200);
            arrow.lineTo(300, 100);                                    // 2. Shaft
            arrow.lineTo(320, 120);
            arrow.lineTo(300, 100);
            arrow.lineTo(320, 80);
            arrow.closePath();
            g2.setColor(Color.DARK_GRAY);
            g2.fill(arrow);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Arrow");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new ArrowPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A dark-gray arrow pointing up-right.

---

#### Example 2: Heart Shape with Cubic Curves

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Path2D;

public class HeartPanel extends JPanel {
    public HeartPanel() {
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
            Path2D heart = new Path2D.Double();
            heart.moveTo(150, 250);
            heart.curveTo(80, 180, 40, 120, 80, 80);                   // 1. Left
            heart.curveTo(120, 40, 150, 80, 150, 120);                 // 2. Top
            heart.curveTo(150, 80, 180, 40, 220, 80);                  // 3. Right
            heart.curveTo(260, 120, 220, 180, 150, 250);               // 4. Bottom
            heart.closePath();
            g2.setColor(new Color(220, 20, 60));
            g2.fill(heart);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Heart");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new HeartPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A crimson heart shape drawn with four cubic Bézier curves.

**Why the code produces this result.** Each `curveTo` call defines a cubic Bézier curve from the current point to the specified endpoint via two control points. Four curves form the heart's lobes and bottom.

---

### Real-World Cases

- **Custom icons and illustrations**.
- **Diagram shapes** (arrows, callouts, symbols).
- **Interactive drawing tools** that record paths from mouse gestures.
- **Animations** that morph between paths.
- **Maps** and **route lines**.

---

## 11. Shape Best Practices

### Definitions

**Core Definition.** Shape best practices are the conventions and patterns that ensure geometric code is accurate, efficient, and maintainable.

**Technical Definition.** Best practices include: using `Path2D.Double` instead of `GeneralPath`; using `Area` for boolean operations; using `getBounds2D()` for precision; using `contains` for hit-testing and `intersects` for collision; using `AffineTransform.createTransformedShape` instead of manually transforming coordinates; caching shapes; and preferring the `Double` subclasses for mathematical accuracy.

**Beginner-Friendly Explanation.** Best practices for shapes are the "rules of thumb" that keep geometric code accurate and fast. They include things like "use `Path2D.Double` instead of `GeneralPath`," "use `Area` for combinations," "use `getBounds2D` for precision," and "cache complex shapes."

---

### Purposes

- To use the modern, precise API (`Path2D`, `Rectangle2D.Double`) rather than legacy classes.
- To compute accurate bounds with `getBounds2D`.
- To perform efficient hit-testing and collision detection.
- To reuse complex geometry rather than recomputing it.
- To transform shapes correctly with `AffineTransform`.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
// Use Path2D for arbitrary paths:
Path2D.Double path = new Path2D.Double();
path.moveTo(x, y);
path.lineTo(x2, y2);
path.closePath();

// Use Area for boolean combinations:
Area ring = new Area(outer);
ring.subtract(new Area(inner));

// Use getBounds2D for precision:
Rectangle2D b = shape.getBounds2D();

// Use contains for hit-testing:
if (shape.contains(mousePoint)) { /* ... */ }

// Use createTransformedShape for transforms:
Shape rotated = new Ellipse2D.Double(0, 0, 100, 50);
Shape rotatedShape = AffineTransform.getRotateInstance(
    Math.toRadians(30), 50, 25).createTransformedShape(rotated);
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use Path2D, not GeneralPath | `new Path2D.Double()` |
| Use Area for combinations | `new Area(shape1).subtract(new Area(shape2))` |
| Use getBounds2D | `shape.getBounds2D()` |
| Use contains/intersects | `shape.contains(p)` |
| Use createTransformedShape | `at.createTransformedShape(shape)` |
| Use Double subclasses | `Rectangle2D.Double`, `Path2D.Double` |
| Cache complex shapes | Store as `final` fields |
| Close paths | `path.closePath()` |
| Set winding rule for self-intersecting paths | `path.setWindingRule(Path2D.WIND_EVEN_ODD)` |

#### Syntax Rules

- **Use `Path2D.Double`** for new arbitrary-path geometry; avoid `GeneralPath`.
- **Use `Area`** for boolean geometry; do not attempt set operations on shapes manually.
- **Use `getBounds2D()`** for floating-point precision.
- **Use `contains` and `intersects`** for hit-testing and collision rather than manual geometry math.
- **Use `AffineTransform.createTransformedShape`** rather than transforming individual coordinates.
- **Cache complex shapes** as fields; recomputation in a paint method is a performance anti-pattern.
- **Close paths** with `closePath()` so that the outline is closed correctly.

#### Constraints and Limitations

- `Area` operations are expensive for very complex shapes; cache results.
- `GeneralPath` uses `float` precision and is deprecated; migrate to `Path2D.Double`.
- `Polygon` uses `int` precision; for sub-pixel accuracy, use `Path2D.Double`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Icon with Path2D and Area

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.*;

public class BestPracticeShape extends JPanel {
    private final Shape house;

    public BestPracticeShape() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 400));

        // 1. Body (rectangle)
        Area body = new Area(new Rectangle2D.Double(100, 180, 200, 120));

        // 2. Roof (triangle via Path2D.Double)
        Path2D roofPath = new Path2D.Double();
        roofPath.moveTo(80, 180);
        roofPath.lineTo(200, 90);
        roofPath.lineTo(320, 180);
        roofPath.closePath();
        body.add(new Area(roofPath));

        // 3. Door (rectangle subtracted from body)
        body.subtract(new Area(new Rectangle2D.Double(180, 240, 40, 60)));

        // 4. Windows (two rectangles subtracted from body)
        body.subtract(new Area(new Rectangle2D.Double(120, 200, 40, 40)));
        body.subtract(new Area(new Rectangle2D.Double(240, 200, 40, 40)));

        house = body;                                                 // 5. Cache
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(new Color(120, 100, 60));
            g2.fill(house);                                            // 6. Fill
            g2.setColor(Color.DARK_GRAY);
            g2.setStroke(new BasicStroke(2f));
            g2.draw(house);                                            // 7. Outline
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("House Icon");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticeShape());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A brown house shape with a triangular roof, a door, and two windows cut out.

**Why the code produces this result.** The shape is composed with `Area` additions and subtractions, cached in a field, and reused on every paint. `Path2D.Double` builds the roof triangle. Anti-aliasing smooths the edges.

---

#### Example 2: Common Mistake — Rebuilding Complex Shapes in Paint

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        Area complex = new Area(new Ellipse2D.Double(50, 50, 300, 300)); // WRONG
        for (int i = 0; i < 50; i++) {
            complex.subtract(new Area(
                new Ellipse2D.Double(50 + i * 4, 50 + i * 4, 20, 20)));
        }
        g2.fill(complex);
    } finally {
        g2.dispose();
    }
}
```

**Expected Output.** Slow painting, because the complex `Area` is recomputed from scratch on every repaint (potentially dozens of times per second).

**Why this is wrong.** Building complex shapes in `paintComponent` wastes CPU. The shape should be constructed once and cached as a field. The correct approach is to build in the constructor or on first use, cache in a field, and reuse.

---

#### Example 3: Common Mistake — Using Legacy `GeneralPath`

```java
GeneralPath gp = new GeneralPath();                                // WRONG
gp.moveTo(10, 10);
gp.lineTo(100, 100);
gp.closePath();
```

**Expected Output.** The path is created, but the code uses the deprecated `GeneralPath` class rather than the modern `Path2D.Double`, so it lacks the modern API and precision.

**Why this is wrong.** `GeneralPath` is deprecated in favour of `Path2D`. New code should use `Path2D.Double` (or `Path2D.Float` for performance-sensitive contexts). The code should be migrated.

---

#### Example 4: Common Mistake — Ignoring the Winding Rule

```java
Path2D path = new Path2D.Double();                                 // WRONG
path.setWindingRule(Path2D.WIND_EVEN_ODD);
path.moveTo(0, 0);
path.lineTo(100, 0);
path.lineTo(100, 100);
path.lineTo(0, 100);
path.lineTo(0, 0);
path.moveTo(25, 25);                                                // Inner loop
path.lineTo(75, 25);
path.lineTo(75, 75);
path.lineTo(25, 75);
path.lineTo(25, 25);
g2.fill(path);                                                       // Hole
```

**Expected Output.** With `WIND_EVEN_ODD`, the inner loop creates a **hole**. With `WIND_NON_ZERO` (the default), the inner loop is filled. This is not a bug, but a common source of surprise: winding rules determine what counts as "inside" a self-intersecting or multi-loop path.

**Why this matters.** Understanding winding rules lets you produce holes and cutouts with a single path. The correct approach depends on the desired visual effect: use `WIND_EVEN_ODD` when you want alternating fill, and `WIND_NON_ZERO` when you want overlapping loops to remain filled.

---

### Real-World Cases

- **Production Swing applications** cache complex shapes and use `Path2D.Double` and `Area` for geometric composition.
- **Icon libraries** construct compound icons with `Area` and render them as `BufferedImage` for performance.
- **Interactive editors** use `contains` for hit-testing and `AffineTransform` for rotation/scaling.
- **Data visualisations** build chart markers and custom shapes from `Path2D`.
- **Educational examples** demonstrate the modern API (`Path2D.Double`) and winding rules from the start.

---

## References

Oracle. *Interface Shape*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Shape.html

Oracle. *Class Line2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Line2D.html

Oracle. *Class Rectangle2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Rectangle2D.html

Oracle. *Class Ellipse2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Ellipse2D.html

Oracle. *Class RoundRectangle2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/RoundRectangle2D.html

Oracle. *Class Arc2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Arc2D.html

Oracle. *Class Polygon*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Polygon.html

Oracle. *Class Area*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Area.html

Oracle. *Class GeneralPath*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/GeneralPath.html

Oracle. *Class Path2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Path2D.html

Oracle. *Class Path2D.Double*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Path2D.Double.html

Oracle. *Class Path2D.Float*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Path2D.Float.html

Oracle. *Interface PathIterator*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/PathIterator.html

Oracle. *Class AffineTransform*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/AffineTransform.html

Oracle. *Class RectangularShape*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/RectangularShape.html

Oracle. *Class QuadCurve2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/QuadCurve2D.html

Oracle. *Class CubicCurve2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/CubicCurve2D.html

Oracle. *Java 2D API Overview*. https://www.oracle.com/java/technologies/2d/overview.html

Oracle. *Working with Geometry — Java 2D*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/2d/geometry/index.html

Oracle. *Stroking and Filling Graphics Primitives*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/2d/geometry/strokeandfill.html

Oracle. *Manipulating Complex Shapes with Area*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/2d/geometry/area.html