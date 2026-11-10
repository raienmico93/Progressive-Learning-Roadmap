# Paints (Java 2D) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **Paint** in Java 2D is an object that defines **how color patterns are generated** for `Graphics2D` drawing operations. It determines the colors used by `fill(Shape)` and `draw(Shape)`—and even by text rendering—across the shape's interior or along its outline. The `Paint` interface is the root of a small hierarchy of implementations: `Color` (solid fill), `GradientPaint` and `LinearGradientPaint` (linear gradients), `RadialGradientPaint` (radial gradients), and `TexturePaint` (image-based tiled patterns).

**Technical Definition.** `public interface Paint extends Transparency`. It declares a single method: `PaintContext createContext(ColorModel cm, Rectangle deviceBounds, Rectangle2D userBounds, AffineTransform xform, RenderingHints hints)`. The `PaintContext` produces a raster of colors for the pixels covered by a shape. `Graphics2D` holds a `Paint` attribute, set via `setPaint(Paint)` and retrieved via `getPaint()`. **Instances of classes implementing `Paint` must be read-only** because `Graphics2D` does not clone these objects when they are set as an attribute or when the `Graphics2D` itself is cloned. The standard implementations include `Color` (sRGB or arbitrary color space), `GradientPaint` (two-color linear gradient), `LinearGradientPaint` (multi-stop linear gradient), `RadialGradientPaint` (radial gradient), and `TexturePaint` (tiled `BufferedImage` texture). `setColor(c)` is equivalent to `setPaint(c)` on `Graphics2D`.

**Beginner-Friendly Explanation.** A **Paint** is the “fill style” for drawing. When you fill a shape, the `Paint` decides what colors appear inside it. A `Color` gives a solid color. A `GradientPaint` blends two colors smoothly across the shape. A `TexturePaint` repeats a small image across the shape like wallpaper. You set the paint once on the `Graphics2D` context, and every subsequent `fill(Shape)` or `draw(Shape)` uses that paint. Paints are **immutable**—once created, they cannot change—so you can safely cache and share them. Think of a `Paint` as a **bucket of paint** with a specific recipe: solid, blended, or patterned. You dip your brush (`Graphics2D`) into the bucket once, and every stroke uses that recipe until you dip into a different bucket.

---

### Key Characteristics

1. **Interface with a single method.** `Paint.createContext` produces a `PaintContext`, which generates pixel colors for a given shape.
2. **Read-only requirement.** `Paint` implementations must be immutable; `Graphics2D` does not clone paints when they are set or when the context is cloned.
3. **Set once, applies to all fills and strokes.** `setPaint` affects `fill(Shape)`, `draw(Shape)`, and text rendering until changed.
4. **`setColor` is a shorthand for `setPaint`.** `setColor(c)` is equivalent to `setPaint(c)` on `Graphics2D`.
5. **Five standard implementations.** `Color`, `GradientPaint`, `LinearGradientPaint`, `RadialGradientPaint`, `TexturePaint`.
6. **User-space and device-space geometry.** Paints are defined in user space and mapped to device space via the current `AffineTransform` and clipping information passed to `createContext`.
7. **Alpha and transparency.** All standard paints support alpha; `Color` has explicit alpha constructors, and `GradientPaint` supports translucent colors.
8. **Immutable and cacheable.** Paints can be cached as `static final` fields and shared across components and paint cycles.
9. **Thread rules still apply.** Paints are read-only and can be shared across threads; rendering still happens on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and anonymous classes.
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and the `create()`/`dispose()` idiom.
- **Shape basics:** at least `Rectangle2D`, `Ellipse2D`, and `Path2D`.
- **Coordinate system:** origin at top-left, `x` rightward, `y` downward.
- **Color fundamentals:** RGB, RGBA, and alpha transparency.
- **`java.awt.image.BufferedImage`** for `TexturePaint`.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`java.awt.Graphics2D`** — the rendering context that holds a `Paint` attribute.
- **`java.awt.Color`** — the simplest `Paint` implementation (solid color).
- **`java.awt.GradientPaint`** — two-color linear gradient.
- **`java.awt.LinearGradientPaint`** — multi-stop linear gradient.
- **`java.awt.RadialGradientPaint`** — radial gradient.
- **`java.awt.TexturePaint`** — tiled image pattern.
- **`java.awt.MultipleGradientPaint`** — superclass of linear and radial multi-stop gradients.
- **`java.awt.PaintContext`** — the raster generator produced by `createContext`.
- **`java.awt.image.BufferedImage`** — the texture source for `TexturePaint`.
- **`java.awt.RenderingHints`** — quality settings that interact with paint rendering.

---

## Core Concepts and Features

---

## 1. Paint — The Interface

### Definitions

**Core Definition.** `Paint` is the interface that defines how color patterns are generated for `Graphics2D` operations. It is the parent of all fill styles.

**Technical Definition.** `public interface Paint extends Transparency`. It declares `PaintContext createContext(ColorModel cm, Rectangle deviceBounds, Rectangle2D userBounds, AffineTransform xform, RenderingHints hints)`. The `PaintContext` is the engine that produces pixel colors for the shape being rendered. `Graphics2D` provides `setPaint(Paint)` and `getPaint()`. All `Paint` implementations must be read-only.

**Beginner-Friendly Explanation.** `Paint` is the **contract** that says “I know how to produce colors for any shape.” You rarely implement it yourself; the standard implementations cover almost every need.

---

### Purposes

- To provide a uniform, pluggable mechanism for fill styles.
- To decouple the “what” (shape geometry) from the “how” (colors, gradients, patterns).
- To enable rich visual effects (gradients, textures) without changing shape code.
- To allow paints to be cached and shared across drawing operations.
- To serve as the parent interface for all Java 2D fill implementations.

---

### Syntax Rules and Structure

#### Complete General Syntax (Using a Paint)

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    Paint paint = new GradientPaint(0, 0, Color.BLUE, 200, 200, Color.CYAN);
    g2.setPaint(paint);                                         // 1. Install
    g2.fill(new Rectangle2D.Double(50, 50, 200, 150));           // 2. Fill
} finally {
    g2.dispose();
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `Paint` | The interface. |
| `setPaint(Paint)` | Installs the paint on the `Graphics2D`. |
| `getPaint()` | Returns the current paint. |
| `createContext(...)` | Generates the raster for a shape. |

#### Syntax Rules

- `setPaint` replaces the current paint; all subsequent fills and strokes use it.
- `setColor(c)` is a shortcut for `setPaint(c)`.
- Paints must be immutable; do not mutate a paint after constructing it.
- Paints are independent of the `Graphics2D` transform until rendering; the transform is passed to `createContext`.

#### Constraints and Limitations

- Implementing `Paint` from scratch is advanced and rarely necessary.
- Not all rendering surfaces (e.g., some printer contexts) support every paint feature equally.
- Very complex paints (e.g., large textures) can be slow to render.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using a Paint Polymorphically

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Rectangle2D;
import java.awt.geom.RoundRectangle2D;

public class PaintPolymorphism extends JPanel {
    private final Paint[] paints = {
        Color.RED,
        new GradientPaint(0, 0, Color.BLUE, 200, 0, Color.CYAN),
        new RadialGradientPaint(new Point(100, 100), 100,
            new float[]{0f, 1f}, new Color[]{Color.WHITE, Color.GREEN}),
        new TexturePaint(makeTexture(), new Rectangle2D.Double(0, 0, 16, 16))
    };

    private static java.awt.image.BufferedImage makeTexture() {
        java.awt.image.BufferedImage img = new java.awt.image.BufferedImage(
            16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB);
        Graphics2D g = img.createGraphics();
        g.setColor(Color.ORANGE);
        g.fillRect(0, 0, 16, 16);
        g.setColor(Color.DARK_GRAY);
        g.drawLine(0, 0, 16, 16);
        g.dispose();
        return img;
    }

    public PaintPolymorphism() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            int x = 20;
            for (Paint p : paints) {                              // 1. Polymorphic
                g2.setPaint(p);
                g2.fill(new RoundRectangle2D.Double(x, 40, 100, 120, 20, 20));
                x += 120;
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Paint Polymorphism");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new PaintPolymorphism());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Four rounded rectangles, each filled with a different paint: red solid, blue-cyan gradient, white-green radial gradient, and orange texture.

**Why the code produces this result.** A `Paint[]` holds different implementations. The rendering loop calls `setPaint(p)` polymorphically; each implementation produces its own color pattern.

---

### Real-World Cases

- **Charts and dashboards** with gradient-filled bars.
- **Custom UI panels** with textured backgrounds.
- **Data visualisation** with color-coded regions.
- **Icons and graphics** with gradient shading.
- **Theming systems** that swap paints dynamically.

---

## 2. Color — The Solid Paint

### Definitions

**Core Definition.** `Color` is a concrete implementation of `Paint` representing a solid color in the sRGB color space (or an arbitrary `ColorSpace`). It is the most common paint.

**Technical Definition.** `public class Color extends Object implements Paint, Serializable`. Constructors include `Color(int r, int g, int b)`, `Color(int r, int g, int b, int a)`, `Color(float r, float g, float b)`, and `Color(float r, float g, float b, float a)`. Static constants include `Color.RED`, `Color.BLUE`, `Color.WHITE`, `Color.BLACK`, and many others. `Graphics2D.setColor(c)` is equivalent to `setPaint(c)`.

**Beginner-Friendly Explanation.** `Color` is the **solid paint**. You specify red, green, blue, and optionally alpha, and the shape is filled with that single color.

---

### Purposes

- To fill shapes with a solid color.
- To draw outlines and text in a single color.
- To provide alpha transparency when needed.
- To serve as the default paint for drawing operations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Color rgb = new Color(0, 120, 215);
Color rgba = new Color(0, 120, 215, 200);         // alpha 200/255
Color fromFloat = new Color(0.2f, 0.5f, 0.8f, 1f);

g2.setPaint(rgb);
g2.fill(shape);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `Color(int r, int g, int b)` | RGB, opaque. |
| `Color(int r, int g, int b, int a)` | RGBA, with alpha 0–255. |
| `Color(float r, float g, float b, float a)` | RGBA, with alpha 0.0–1.0. |
| `Color(int rgb)` | Bit-packed RGB. |

#### Syntax Rules

- RGB components range 0–255; alpha ranges 0–255 (int) or 0.0–1.0 (float).
- `setColor(c)` is equivalent to `setPaint(c)`.
- `Color` is immutable.

#### Constraints and Limitations

- `Color` cannot represent gradients or patterns.
- Alpha blending is affected by the `Composite` as well as the color's alpha.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Solid Fills with Alpha

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Ellipse2D;

public class ColorPanel extends JPanel {
    public ColorPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(new Color(0, 120, 215));                   // 1. Opaque
            g2.fill(new Ellipse2D.Double(50, 50, 150, 100));
            g2.setColor(new Color(255, 0, 0, 128));                 // 2. Alpha
            g2.fill(new Ellipse2D.Double(150, 50, 150, 100));
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Color Fills");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new ColorPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white panel with an opaque blue ellipse and a semi-transparent red ellipse that overlaps the blue one, producing a purple-ish blend in the overlap.

---

### Real-World Cases

- **Standard UI fills** and text.
- **Highlight overlays** with alpha.
- **Themed color** application-wide.
- **Data points** in charts.

---

## 3. GradientPaint — Two-Color Linear Gradient

### Definitions

**Core Definition.** `GradientPaint` fills a shape with a **linear color gradient** that blends between two colors along a line segment (from a start point to an end point).

**Technical Definition.** `public class GradientPaint extends Object implements Paint`. Constructors:
- `GradientPaint(float x1, float y1, Color c1, float x2, float y2, Color c2)`
- `GradientPaint(Point2D p1, Color c1, Point2D p2, Color c2)`
- Cyclic variants with a `boolean cyclic` parameter that repeat the pattern beyond the segment.

The gradient is defined in **user space**; the `createContext` method maps it to the shape's covered pixels.

**Beginner-Friendly Explanation.** `GradientPaint` is a **smooth blend between two colors** along a straight line. You give it a start point, a start color, an end point, and an end color, and the shape is filled with a gradient from one color to the other.

---

### Purposes

- To fill shapes with two-color linear gradients.
- To create shaded buttons, panels, and charts.
- To add visual depth and dimension.
- To simulate metallic or glass-like surfaces.
- To produce smooth color transitions in infographics.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
GradientPaint gp = new GradientPaint(
    x1, y1, Color.BLUE,             // start point, start color
    x2, y2, Color.CYAN              // end point, end color
);
GradientPaint cyclic = new GradientPaint(
    x1, y1, Color.BLUE, x2, y2, Color.CYAN, true  // repeating
);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `x1, y1` | Start point of the gradient axis. |
| `c1` | Color at the start. |
| `x2, y2` | End point of the gradient axis. |
| `c2` | Color at the end. |
| `cyclic` | Whether the gradient repeats beyond the segment. |

#### Syntax Rules

- The gradient is a smooth blend from `c1` at `(x1, y1)` to `c2` at `(x2, y2)`.
- `cyclic = true` repeats the gradient, which can produce a banded effect beyond the segment.
- The start and end points must be distinct.

#### Constraints and Limitations

- Only two colors; for multi-stop gradients, use `LinearGradientPaint`.
- `GradientPaint` can be performance-heavy if used on very large or frequently painted areas.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Vertical Gradient Bar

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Rectangle2D;

public class GradientBarPanel extends JPanel {
    public GradientBarPanel() {
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
            GradientPaint gp = new GradientPaint(                      // 1. Gradient
                0, 50, Color.BLUE, 0, 250, Color.CYAN);
            g2.setPaint(gp);
            g2.fill(new Rectangle2D.Double(50, 50, 200, 200));        // 2. Fill
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Gradient Bar");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new GradientBarPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A vertical rectangle filled with a smooth blue-to-cyan gradient from top to bottom.

---

### Real-World Cases

- **Buttons** with gradient shading.
- **Chart bars** with a gradient fill.
- **Backgrounds** with a subtle color wash.
- **Custom UI** with metallic or glass effects.

---

## 4. LinearGradientPaint — Multi-Stop Linear Gradient

### Definitions

**Core Definition.** `LinearGradientPaint` fills a shape with a **linear gradient that blends between multiple colors** at specified fractions along an axis.

**Technical Definition.** `public final class LinearGradientPaint extends MultipleGradientPaint`. Constructors include `LinearGradientPaint(float startX, float startY, float endX, float endY, float[] fractions, Color[] colors)` and `LinearGradientPaint(Point2D start, Point2D end, float[] fractions, Color[] colors, CycleMethod cycleMethod, ColorSpaceType colorSpace)`. The `fractions` array specifies the positions (0.0 to 1.0) of each color along the gradient; the `colors` array provides the corresponding colors.

**Beginner-Friendly Explanation.** `LinearGradientPaint` is the **multi-color linear gradient**. You can specify several colors and where each sits along the gradient, creating rich, multi-step blends.

---

### Purposes

- To fill shapes with multi-color linear gradients.
- To create rich, realistic shading and color transitions.
- To support cyclic (repeating) gradients.
- To enable sophisticated visual effects in charts and UI.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
LinearGradientPaint lgp = new LinearGradientPaint(
    startX, startY, endX, endY,
    new float[]{0.0f, 0.5f, 1.0f},                 // positions
    new Color[]{Color.RED, Color.YELLOW, Color.GREEN}
);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `startX, startY` | Start of the gradient axis. |
| `endX, endY` | End of the gradient axis. |
| `float[] fractions` | Positions (0.0–1.0) of each color, in increasing order. |
| `Color[] colors` | Colors corresponding to the fractions. |

#### Syntax Rules

- The `fractions` array must be strictly increasing, with values in `[0.0, 1.0]`.
- The first fraction must be 0.0 and the last must be 1.0 for a full-span gradient.
- The arrays must have equal length.

#### Constraints and Limitations

- More colors mean more rendering work; use judiciously.
- The gradient is defined in user space; transforms affect its visual orientation.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Three-Color Gradient

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Rectangle2D;

public class MultiGradientPanel extends JPanel {
    public MultiGradientPanel() {
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
            LinearGradientPaint lgp = new LinearGradientPaint(        // 1. Multi
                50, 50, 250, 250,
                new float[]{0.0f, 0.5f, 1.0f},
                new Color[]{Color.RED, Color.YELLOW, Color.GREEN});
            g2.setPaint(lgp);
            g2.fill(new Rectangle2D.Double(50, 50, 200, 200));        // 2. Fill
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Multi-Stop Gradient");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new MultiGradientPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A rectangle filled with a diagonal gradient that transitions from red through yellow to green.

---

### Real-World Cases

- **Heat maps** and **weather maps** with multi-stop color scales.
- **Charts** with smooth multi-color fills.
- **UI backgrounds** with sophisticated color transitions.
- **Data visualisation** that maps values to color ranges.

---

## 5. RadialGradientPaint — Radial Gradient

### Definitions

**Core Definition.** `RadialGradientPaint` fills a shape with a **circular radial gradient** that blends between colors radiating outward from a center point.

**Technical Definition.** `public final class RadialGradientPaint extends MultipleGradientPaint`. Constructors include `RadialGradientPaint(float centerX, float centerY, float radius, float[] fractions, Color[] colors)` and `RadialGradientPaint(Point2D center, float radius, Point2D focus, float[] fractions, Color[] colors, CycleMethod cycleMethod)`. The gradient radiates from `center` outward to `radius`; the `focus` point (optional) shifts the center of the innermost color.

**Beginner-Friendly Explanation.** `RadialGradientPaint` is a **circular gradient** that radiates from a center point outward. It is perfect for spotlights, glow effects, and 3D-looking spheres.

---

### Purposes

- To fill shapes with radial gradients (spotlights, glows).
- To create 3D shading for circular objects.
- To simulate lighting effects.
- To produce rich backgrounds and decorative elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
RadialGradientPaint rgp = new RadialGradientPaint(
    centerX, centerY, radius,
    new float[]{0.0f, 1.0f},
    new Color[]{Color.WHITE, Color.BLUE}
);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `centerX, centerY` | Center of the gradient. |
| `radius` | Radius of the gradient. |
| `fractions` | Positions of each color. |
| `colors` | Colors to blend. |

#### Syntax Rules

- The gradient extends from the center to the radius.
- For a focus point, use the constructor that accepts a `Point2D focus`.

#### Constraints and Limitations

- Rendering radial gradients can be more expensive than linear gradients on large surfaces.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Glowing Sphere

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Ellipse2D;

public class GlowPanel extends JPanel {
    public GlowPanel() {
        setOpaque(true);
        setBackground(Color.BLACK);
        setPreferredSize(new Dimension(300, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            RadialGradientPaint rgp = new RadialGradientPaint(        // 1. Radial
                150, 150, 100,
                new float[]{0.0f, 1.0f},
                new Color[]{Color.WHITE, new Color(0, 120, 215)});
            g2.setPaint(rgp);
            g2.fill(new Ellipse2D.Double(50, 50, 200, 200));          // 2. Fill
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Glow");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new GlowPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A circle that appears as a glowing blue sphere with a bright white center fading to blue at the edges.

---

### Real-World Cases

- **Glow effects** in games and UI.
- **3D-looking spheres** and circular gauges.
- **Spotlights** and lighting effects.
- **Radial heat maps**.

---

## 6. TexturePaint — Image-Based Pattern

### Definitions

**Core Definition.** `TexturePaint` fills a shape with a **tiled image pattern**. The image is replicated infinitely across the user space, anchored to a specified `Rectangle2D` that determines the size and position of each tile.

**Technical Definition.** `public class TexturePaint extends Object implements Paint, Transparency`. Constructor: `TexturePaint(BufferedImage txtr, Rectangle2D anchor)`. The anchor rectangle is specified in **user space**; the texture is computed for device-space locations by conceptually replicating the `Rectangle2D` infinitely in all directions in user space and mapping the `BufferedImage` to each replicated rectangle. The image should be small because its data is copied by the `TexturePaint` object.

**Beginner-Friendly Explanation.** `TexturePaint` is the **wallpaper paint**. You give it a small image and a rectangle that defines the tile size; Java 2D repeats that image across the shape like patterned wallpaper. It is perfect for textures, patterns, and image fills.

---

### Purposes

- To fill shapes with tiled image patterns.
- To apply textures (wood, metal, fabric, paper) to UI elements.
- To fill text or shapes with small images.
- To create decorative backgrounds.
- To support repeating patterns without manual tiling loops.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
TexturePaint tp = new TexturePaint(
    bufferedImage,                       // tile image
    new Rectangle2D.Double(0, 0, 32, 32) // anchor in user space
);
g2.setPaint(tp);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `BufferedImage txtr` | The tile image; should be small. |
| `Rectangle2D anchor` | The tile size and position in user space. |

#### Syntax Rules

- The image should be small; its data is copied by `TexturePaint`.
- The anchor rectangle defines the tile size and position.
- The pattern repeats infinitely in all directions.

#### Constraints and Limitations

- Large textures consume memory and slow rendering.
- The tile image's size should match the anchor rectangle for a seamless pattern.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Tiled Pattern Fill

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Rectangle2D;
import java.awt.image.BufferedImage;

public class TexturePanel extends JPanel {
    private final TexturePaint tile;

    public TexturePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 300));

        BufferedImage img = new BufferedImage(16, 16, BufferedImage.TYPE_INT_RGB);
        Graphics2D ig = img.createGraphics();
        ig.setColor(new Color(255, 200, 100));
        ig.fillRect(0, 0, 16, 16);
        ig.setColor(Color.WHITE);
        ig.fillOval(2, 2, 12, 12);
        ig.dispose();

        tile = new TexturePaint(img, new Rectangle2D.Double(0, 0, 16, 16)); // 1. Tile
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setPaint(tile);                                          // 2. Paint
            g2.fill(new Rectangle2D.Double(50, 50, 200, 200));          // 3. Fill
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Texture Fill");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new TexturePanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A rectangle filled with a repeating pattern of orange squares containing white circles, like a tiled wallpaper.

---

### Real-World Cases

- **Textured backgrounds** for UI panels.
- **Pattern fills** in chart areas.
- **Image-filled text**.
- **Decorative borders** and frames.

---

## 7. Paint Best Practices

### Definitions

**Core Definition.** Paint best practices are the conventions and patterns that ensure paint usage is correct, performant, and maintainable.

**Technical Definition.** Best practices include: caching paints as `static final` fields (paints are immutable); using anti-aliasing for gradients and textures; preferring `LinearGradientPaint` over `GradientPaint` for multi-color needs; using small `BufferedImage` tiles for `TexturePaint`; considering performance implications of complex paints on large or frequently painted areas; using `setPaint` consistently with `setColor` shorthand; and disposing `Graphics2D` copies.

**Beginner-Friendly Explanation.** Best practices for paints are the “rules of thumb” that keep gradients and textures looking good and rendering fast. They include things like “cache your paints,” “use small textures,” and “watch out for performance on large fills.”

---

### Purposes

- To avoid reallocating paint objects in hot paint code.
- To ensure gradients and textures render smoothly.
- To prevent memory and performance issues from large textures.
- To maintain consistent visual appearance across components.
- To keep paint code readable and maintainable.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyPanel extends JPanel {
    private static final Paint GRADIENT = new GradientPaint(
        0, 0, Color.BLUE, 0, 200, Color.CYAN);
    private static final Paint TEXTURE = new TexturePaint(
        makeTile(), new Rectangle2D.Double(0, 0, 16, 16));

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setPaint(GRADIENT);                                   // 1. Cached
            g2.fill(shape1);
            g2.setPaint(TEXTURE);                                    // 2. Cached
            g2.fill(shape2);
        } finally {
            g2.dispose();
        }
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Cache paints | `static final Paint` fields |
| Use anti-aliasing | `setRenderingHint(KEY_ANTIALIASING, VALUE_ANTIALIAS_ON)` |
| Prefer multi-stop for multi-color | `LinearGradientPaint` over `GradientPaint` |
| Use small textures | `BufferedImage` 16×16 or 32×32 |
| Consider performance | Avoid complex paints on very large or high-frequency surfaces |
| Use `setColor` shorthand | `setColor(c)` = `setPaint(c)` |
| Dispose copies | `try/finally { g2.dispose(); }` |

#### Syntax Rules

- **Always cache paints** as `static final` fields; they are immutable and safe to share.
- **Always enable anti-aliasing** when using gradients and textures.
- **Always use small tile images** for `TexturePaint`; large textures consume memory and slow rendering.
- **Always consider performance**; complex gradients on large or frequently painted surfaces can degrade responsiveness.
- **Always dispose `Graphics2D` copies.**

#### Constraints and Limitations

- `GradientPaint` can be expensive on large surfaces; prefer simple paints where possible.
- `TexturePaint` copies the image data; small images are essential.
- Some paints may not render identically on all output devices (print vs. screen).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Cached Paints

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Ellipse2D;
import java.awt.geom.Rectangle2D;
import java.awt.image.BufferedImage;

public class BestPracticePaint extends JPanel {
    private static final Paint SOLID = new Color(0, 120, 215);
    private static final Paint GRADIENT = new GradientPaint(
        0, 0, Color.BLUE, 0, 200, Color.CYAN);
    private static final Paint RADIAL = new RadialGradientPaint(
        new Point(100, 100), 100,
        new float[]{0f, 1f},
        new Color[]{Color.WHITE, new Color(0, 160, 0)});
    private static final Paint TEXTURE = new TexturePaint(
        makeTile(), new Rectangle2D.Double(0, 0, 16, 16));

    private static BufferedImage makeTile() {
        BufferedImage img = new BufferedImage(16, 16, BufferedImage.TYPE_INT_RGB);
        Graphics2D g = img.createGraphics();
        g.setColor(new Color(255, 200, 100));
        g.fillRect(0, 0, 16, 16);
        g.setColor(Color.WHITE);
        g.drawLine(0, 0, 16, 16);
        g.dispose();
        return img;
    }

    public BestPracticePaint() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            int x = 20;
            for (Paint p : new Paint[]{SOLID, GRADIENT, RADIAL, TEXTURE}) {
                g2.setPaint(p);
                g2.fill(new Ellipse2D.Double(x, 40, 100, 100));
                x += 120;
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Paint");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticePaint());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Four circles, each filled with a different paint: solid blue, blue-cyan gradient, white-green radial gradient, and orange texture. The paints are cached and reused.

---

#### Example 2: Common Mistake — Allocating Paints in Paint

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        for (int i = 0; i < 100; i++) {
            g2.setPaint(new GradientPaint(0, 0, Color.BLUE,
                                          200, 200, Color.CYAN));  // WRONG
            g2.fill(new Ellipse2D.Double(i * 5, 50, 40, 40));
        }
    } finally {
        g2.dispose();
    }
}
```

**Expected Output.** Slow painting because 100 `GradientPaint` objects are allocated per repaint (potentially dozens of times per second).

**Why this is wrong.** Paints should be cached as fields; allocating them in `paintComponent` is a performance anti-pattern.

---

#### Example 3: Common Mistake — Using a Huge Texture Image

```java
BufferedImage huge = new BufferedImage(1024, 1024, BufferedImage.TYPE_INT_RGB);
TexturePaint tp = new TexturePaint(huge, new Rectangle2D.Double(0, 0, 1024, 1024)); // WRONG
```

**Expected Output.** High memory consumption and slow rendering, because `TexturePaint` copies the image data.

**Why this is wrong.** The `BufferedImage` should be small (e.g., 16×16 or 32×32) because its data is copied by `TexturePaint`. The correct approach is to use a small tile and let the paint repeat it.

---

### Real-World Cases

- **Production Swing applications** cache paints as `static final` fields and use small textures.
- **Charts and dashboards** use `LinearGradientPaint` for multi-color gradients and cache them.
- **Games** use `TexturePaint` with small tile images for patterned surfaces.
- **Educational examples** demonstrate cached paints and anti-aliasing from the start.

---

## References

Oracle. *Interface Paint*. Java Platform, Standard Edition 16 API Specification. https://docs.oracle.com/en/java/javase/16/docs/api/java.desktop/java/awt/Paint.html

Oracle. *Class Color*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Color.html

Oracle. *Class GradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/GradientPaint.html

Oracle. *Class LinearGradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/LinearGradientPaint.html

Oracle. *Class RadialGradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/RadialGradientPaint.html

Oracle. *Class TexturePaint*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/TexturePaint.html

Oracle. *Class MultipleGradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/MultipleGradientPaint.html

Oracle. *Interface PaintContext*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/PaintContext.html

Oracle. *Uses of Interface Paint*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/class-use/Paint.html

Cleverence. *Mastering Java Paint: AWT/Swing paint(), repaint() and Rendering in Java SE 8*. https://www.cleverence.com/articles/oracle-documentation/paint-java-se-8-awt-swing-rendering-4827/

Oracle. *Java 2D API: Paint*. https://www.oracle.com/java/technologies/2d/overview.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading