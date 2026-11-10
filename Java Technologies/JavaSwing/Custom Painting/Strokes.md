# Strokes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **Stroke** is an object that defines how a `Graphics2D` draws the **outline** of a `Shape`. It converts an abstract geometric shape (which has zero width) into a rendered outline with a specific visual style — line thickness, dash pattern, how segment ends are capped, how segment corners are joined, and how sharp a corner's miter can be before it is clipped. The `Stroke` interface is implemented primarily by `BasicStroke`, the standard implementation that covers virtually every drawing style Swing applications need.

**Technical Definition.** `public interface Stroke`. It declares a single method: `Shape createStrokedShape(Shape p)`. Given an input shape, the stroke returns a **new shape** representing the filled region that, when painted, produces the stroked outline. `Graphics2D.draw(Shape)` internally calls `getStroke().createStrokedShape(shape)` and then fills the resulting outline shape with the current paint. `public class BasicStroke extends Object implements Stroke`. It is immutable, with properties: **line width** (a `float`), **end cap** (`CAP_BUTT`, `CAP_ROUND`, `CAP_SQUARE`), **line join** (`JOIN_MITER`, `JOIN_ROUND`, `JOIN_BEVEL`), **miter limit** (a `float` that limits miter extension), and **dash attributes** (`float[] dashArray` and `float dashPhase`). Strokes may be set only on `Graphics2D` via `setStroke(Stroke s)`; the basic `Graphics` class does not support them. Strokes are affected by the current `AffineTransform` of the `Graphics2D` — the stroke operates in user space, so transforms scale the rendered outline accordingly.

**Beginner-Friendly Explanation.** A **stroke** is the **brush style** for drawing outlines. Without a stroke, `draw(shape)` would produce a 1-pixel-wide line. With a `BasicStroke`, you can draw thick lines, dashed lines, rounded line ends, rounded corners, and more. Think of a stroke as the **pen** in a drawing program: it has a nib (the width), an end style (cap), a corner style (join), and it can be solid or dashed. You set the stroke once, then all subsequent `draw(Shape)` calls use it. Fills are **not** affected by strokes — filling uses the current `Paint` directly, not the stroke.

---

### Key Characteristics

1. **Interface with one method.** `Stroke.createStrokedShape(Shape)` converts a geometric shape into its stroked outline as a fillable shape.
2. **Standard implementation.** `BasicStroke` implements `Stroke` with five configurable properties: width, cap, join, miter limit, dash array and dash phase.
3. **Immutable.** `BasicStroke` instances are immutable; you create a new one to change a property.
4. **Applied only by `Graphics2D`.** The base `Graphics` class cannot use strokes; you must cast to `Graphics2D` and use `setStroke`.
5. **Affects outlines, not fills.** `draw(Shape)` respects the stroke; `fill(Shape)` ignores it and uses only the paint and clip.
6. **Affected by transforms.** The stroke is defined in **user space**, so applying a transform scales the stroked outline visually. Rotating, scaling, or shearing the `Graphics2D` affects the stroke's appearance.
7. **Stroke affects path rendering, not stroke of stroke.** Strokes do not compose recursively; a stroked shape is itself a shape (its outline can be stroked, but `Graphics2D` will not do this automatically).
8. **Line width is a float.** Sub-pixel widths are allowed and supported by anti-aliased rendering.
9. **Dashes are configured by pattern and phase.** The dash array alternates "on" and "off" segments in user-space units; the dash phase offsets the start of the pattern.
10. **Thread and lifetime rules.** `BasicStroke` is immutable and safe to cache and share. Paint operations still run on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and anonymous classes.
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and the copy/dispose idiom.
- **Shape basics:** at least `Line2D`, `Rectangle2D`, `Ellipse2D`, and `Path2D`.
- **Coordinate system:** origin at top-left, `x` rightward, `y` downward.
- **Float precision:** widths, miter limit, and dash patterns are floats.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`java.awt.Graphics2D`** — the rendering context that applies the stroke to `draw(Shape)`.
- **`java.awt.BasicStroke`** — the standard `Stroke` implementation.
- **`java.awt.Shape`** — the geometry that the stroke converts into an outline.
- **`java.awt.geom.Path2D`, `Line2D`, `Rectangle2D`, `Ellipse2D`, `Area`** — common stroke targets.
- **`java.awt.RenderingHints`** — anti-aliasing improves the visual quality of thick/dashed strokes.
- **`java.awt.Paint`** — determines the fill of the stroked outline.
- **`java.awt.geom.AffineTransform`** — affects how strokes appear when rendered.
- **Custom `Stroke` implementations** — for specialised visual effects (e.g., textured outlines).

---

## Core Concepts and Features

---

## 1. Stroke — The Interface

### Definitions

**Core Definition.** `Stroke` is the interface that converts a geometric shape into a fillable outline shape.

**Technical Definition.** `public interface Stroke`. It declares `Shape createStrokedShape(Shape p)`. `Graphics2D.draw(Shape)` calls this method and fills the result with the current paint. Custom `Stroke` implementations can produce arbitrary outline shapes — textures, brushes, variable widths, etc.

**Beginner-Friendly Explanation.** `Stroke` is the **contract** that turns "a path" into "an outline of the path." Standard `BasicStroke` is the tool for thickness, caps, joins, and dashes.

---

### Purposes

- To define the visual style of drawn outlines.
- To allow applications to control line thickness, dash patterns, and corner/end shapes.
- To support custom visual effects by implementing the interface.
- To allow strokes to be shared (since implementations are typically immutable).
- To be applied uniformly across all `draw(Shape)` calls in a rendering context.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    g2.setStroke(new BasicStroke(2f));   // Replace with a custom Stroke for effects
    g2.draw(someShape);
} finally {
    g2.dispose();
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `Stroke` | The interface. |
| `createStrokedShape(Shape)` | The single method to implement. |
| `BasicStroke` | The standard implementation. |
| `Graphics2D.setStroke(Stroke)` | Installs the stroke on the context. |

#### Syntax Rules

- Only `Graphics2D` supports strokes; the base `Graphics` does not.
- The stroke affects `draw(Shape)` and `drawLine`.
- The stroke does **not** affect `fill(Shape)` or `drawString`.
- Strokes may be shared across contexts; `BasicStroke` is immutable.

#### Constraints and Limitations

- A custom `Stroke` must return a valid `Shape`; returning `null` produces undefined behaviour.
- Some custom strokes are expensive to compute; cache their results if reused.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Stroke Replacement

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Line2D;

public class StrokeBasicPanel extends JPanel {
    public StrokeBasicPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();                       // 1. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);    // 2. AA
            g2.setColor(Color.BLUE);
            g2.setStroke(new BasicStroke(8f));                         // 3. Stroke
            g2.draw(new Line2D.Double(50, 100, 350, 100));             // 4. Draw
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Stroke Basic");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new StrokeBasicPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A thick, blue, 8-pixel horizontal line across the panel.

**Why the code produces this result.** The `BasicStroke(8f)` creates an 8-pixel stroke. `draw(Line2D)` uses the current stroke to produce the outline, then fills that outline with the current colour. Anti-aliasing smooths the edges.

---

### Real-World Cases

- **Charts and graphs** with styled axes and data lines.
- **Diagram editors** with styled connectors.
- **Custom UI** with bold separators or dividers.
- **Maps** with dashed route lines.
- **Custom brushes** implemented as `Stroke`.

---

## 2. BasicStroke — The Standard Implementation

### Definitions

**Core Definition.** `BasicStroke` is the standard immutable `Stroke` implementation, configurable with width, end cap, line join, miter limit, and dash pattern.

**Technical Definition.** `public class BasicStroke extends Object implements Stroke`. Constructors:
- `BasicStroke(float width)`
- `BasicStroke(float width, int cap, int join)`
- `BasicStroke(float width, int cap, int join, float miterlimit)`
- `BasicStroke(float width, int cap, int join, float miterlimit, float[] dash, float dashPhase)`

`BasicStroke` implements `createStrokedShape(Shape)` and provides getters for width, cap, join, miter limit, and dash attributes. `BasicStroke` is immutable and safe to share.

**Beginner-Friendly Explanation.** `BasicStroke` is the **standard pen**. It has five settings: thickness, how ends are capped, how corners are joined, how sharp a corner can get before being trimmed, and whether it is solid or dashed.

---

### Purposes

- To configure line thickness for `draw(Shape)`.
- To configure how line ends are drawn (butt, round, square).
- To configure how corners are drawn (miter, round, bevel).
- To configure dashed patterns.
- To be cached and shared across the application.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
BasicStroke simple = new BasicStroke(2f);
BasicStroke capped = new BasicStroke(4f, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND);
BasicStroke mitered = new BasicStroke(4f, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER, 10f);
BasicStroke dashed = new BasicStroke(2f, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER,
                                     10f, new float[]{10f, 5f}, 0f);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `width` | Line width in user-space units (float). |
| `cap` | `CAP_BUTT`, `CAP_ROUND`, or `CAP_SQUARE`. |
| `join` | `JOIN_MITER`, `JOIN_ROUND`, or `JOIN_BEVEL`. |
| `miterlimit` | Maximum miter extension ratio. |
| `dash` | Array of alternating on/off dash lengths. |
| `dashPhase` | Offset into the dash pattern. |

#### Syntax Rules

- Width must be `>= 0`.
- The miter limit must be `>= 1`.
- Dash entries must be `>= 0` and at least one nonzero; otherwise, an `IllegalArgumentException` is thrown.
- `BasicStroke` is immutable.

#### Constraints and Limitations

- A width of 0 draws a 1-pixel cosmetic line (implementation-defined).
- A miter limit below the geometric need causes miter joins to fall back to bevel joins automatically.
- Dash arrays require anti-aliasing for smooth appearance on curves.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Family of Strokes

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Line2D;

public class StrokeFamilyPanel extends JPanel {
    public StrokeFamilyPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 400));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            int y = 40;
            String[] labels = {"Thin", "Thick", "Round", "Dashed"};
            BasicStroke[] strokes = {
                new BasicStroke(1f),
                new BasicStroke(6f),
                new BasicStroke(6f, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND),
                new BasicStroke(3f, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER,
                                10f, new float[]{12f, 6f}, 0f)
            };
            g2.setColor(Color.DARK_GRAY);
            g2.setFont(new Font("SansSerif", Font.PLAIN, 14));

            for (int i = 0; i < strokes.length; i++) {
                g2.drawString(labels[i], 20, y - 10);
                g2.setColor(Color.BLUE);
                g2.setStroke(strokes[i]);
                g2.draw(new Line2D.Double(100, y, 450, y));
                g2.setColor(Color.DARK_GRAY);
                y += 80;
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Stroke Family");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new StrokeFamilyPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Four labelled blue horizontal lines, each using a different stroke: thin, thick, round-capped, and dashed.

---

### Real-World Cases

- **Chart lines** with configurable widths.
- **Dashed borders** on selection rectangles.
- **Rounded edges** on custom UI elements.
- **Stylised diagrams** and maps.

---

## 3. Stroke Properties

### 3.1 Width

**Definitions.** **Core:** The thickness of the stroked outline in user-space units. **Technical:** `public float getLineWidth()`. Set at construction. A value of 0 produces a 1-pixel-wide cosmetic stroke (implementation-dependent). **Beginner:** How thick the line is.

**Purposes.** To visually distinguish lines by importance; to match design specifications; to represent data lines in charts.

**Syntax Rules.** `new BasicStroke(float width)`. Width must be `>= 0`.

**Constraints.** Very large widths can exceed the surface's clipping and produce unexpected visual overlap.

**Example.**

```java
BasicStroke thin = new BasicStroke(1f);
BasicStroke thick = new BasicStroke(6f);
```

**Real-World.** Chart series with different visual weights.

---

### 3.2 Cap

**Definitions.** **Core:** Determines how the **ends** of a stroked open path are drawn. **Technical:** `CAP_BUTT` (default, ends exactly at endpoint), `CAP_ROUND` (rounded semicircle at each end), `CAP_SQUARE` (extends the line by half the width). **Beginner:** Whether the ends of your line are flat, rounded, or squared.

**Purposes.** To visually finish the ends of lines consistently; to make data lines look rounded or crisp.

**Syntax Rules.** `new BasicStroke(width, BasicStroke.CAP_ROUND, ...)`. **Constraints.** Cap style has no effect on closed paths (e.g., rectangles, ellipses) because they have no ends.

**Example.**

```java
BasicStroke rounded = new BasicStroke(4f,
    BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND);
```

---

### 3.3 Join

**Definitions.** **Core:** Determines how the **corners** where two segments meet are drawn. **Technical:** `JOIN_MITER` (sharp corner, extends by miter limit), `JOIN_ROUND` (rounded corner), `JOIN_BEVEL` (flat cut-off corner). **Beginner:** Whether corners are sharp, rounded, or cut off.

**Purposes.** To match the visual style of chart corners and UI decorations.

**Syntax Rules.** `new BasicStroke(width, cap, BasicStroke.JOIN_ROUND)`. **Constraints.** For very sharp angles, miter joins are clipped to a bevel if they exceed the miter limit.

**Example.**

```java
BasicStroke sharp = new BasicStroke(3f,
    BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER, 10f);
```

---

### 3.4 Miter Limit

**Definitions.** **Core:** The maximum length a miter join may extend from the endpoint before the join is converted to a bevel. **Technical:** `public float getMiterLimit()`. The miter limit is expressed as a **ratio** of the miter length to the line width (must be `>= 1`). Default is 10. **Beginner:** How pointy a corner can get before it is cut off.

**Purposes.** To prevent very sharp miter spikes from producing visual artifacts.

**Syntax Rules.** Only relevant when `join == JOIN_MITER`. A low value cuts off sharp corners sooner; a high value lets them extend further.

**Example.**

```java
BasicStroke sharpish = new BasicStroke(3f, BasicStroke.CAP_BUTT,
    BasicStroke.JOIN_MITER, 4f); // cut off sharper than 4:1
```

**Constraints.** Below 1 is illegal.

---

### 3.5 Dash

**Definitions.** **Core:** A pattern of alternating "on" and "off" segment lengths and a starting phase that produces dashed or dotted lines. **Technical:** `float[] dash` (alternating on/off lengths) and `float dashPhase` (initial offset). **Beginner:** The dashed-line pattern — how long each "dash" and "gap" is.

**Purposes.** To draw dashed borders, dotted selections, dashed routes, and decorative lines.

**Syntax Rules.** `new BasicStroke(width, cap, join, miterlimit, dash, dashPhase)`. Dash values must be non-negative, with at least one nonzero.

**Example.**

```java
BasicStroke dashed = new BasicStroke(2f, BasicStroke.CAP_BUTT,
    BasicStroke.JOIN_MITER, 10f, new float[]{10f, 6f}, 0f);
```

**Constraints.** Dashes look best with anti-aliasing; without it, curves show visible discontinuities.

---

### Real-World Cases

- **Dash:** Selection marquees, align guides, "no access" lines in maps.
- **Cap:** Rounded ends in gauge needles, data line endpoints.
- **Join:** Rounded corners in custom buttons and cards.
- **Width:** Chart series with varying emphasis.

---

## 4. Stroke Best Practices

### Definitions

**Core Definition.** Stroke best practices are the conventions and patterns that ensure stroked rendering is visually consistent, performant, thread-safe, and maintainable.

**Technical Definition.** Best practices include: caching `BasicStroke` instances as `static final` fields; enabling anti-aliasing for thick or dashed strokes; using `CAP_ROUND` and `JOIN_ROUND` consistently; using the miter limit to clip sharp miter artifacts; using dashes with `CAP_BUTT` for crisp dash ends; considering `AffineTransform` interactions (width scales with transform); disposing the `Graphics2D` copy after use; and using custom `Stroke` implementations sparingly, caching their results.

**Beginner-Friendly Explanation.** Best practices for strokes are the "rules of thumb" that keep outlines looking right and rendering fast. They include "cache your strokes," "enable anti-aliasing," "keep caps and joins consistent," and "watch out for transforms."

---

### Purposes

- To avoid reallocating `BasicStroke` objects in hot paint code.
- To make thick and dashed lines look smooth.
- To prevent visual artifacts from sharp miter corners.
- To handle interactions between strokes and transforms correctly.
- To keep custom strokes performant by caching.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyComponent extends JPanel {
    private static final BasicStroke THIN = new BasicStroke(1f);
    private static final BasicStroke THICK = new BasicStroke(4f,
        BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND);
    private static final BasicStroke DASHED = new BasicStroke(2f,
        BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER, 10f,
        new float[]{10f, 6f}, 0f);

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setStroke(THIN);
            g2.draw(...);
            g2.setStroke(DASHED);
            g2.draw(...);
        } finally {
            g2.dispose();
        }
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Cache strokes | `static final BasicStroke` fields |
| Enable anti-aliasing | `setRenderingHint(KEY_ANTIALIASING, VALUE_ANTIALIAS_ON)` |
| Use consistent caps and joins | Prefer `CAP_ROUND`/`JOIN_ROUND` for organic UI |
| Clip miter spikes | Set miter limit ≈ 4–10 |
| Crisp dashes | Use `CAP_BUTT` for dash ends |
| Transform correctly | Remember that stroke width scales with the transform |
| Dispose copies | `try/finally { g2.dispose(); }` |
| Cache custom strokes | Reuse custom `Stroke` instances |

#### Syntax Rules

- **Always cache `BasicStroke` objects**, especially in `paintComponent`.
- **Always enable anti-aliasing** when using thick or dashed strokes.
- **Prefer `CAP_ROUND` and `JOIN_ROUND`** for organic, modern UI; use `CAP_BUTT` and `JOIN_MITER` for crisp technical diagrams.
- **Use the miter limit** to prevent unsightly spikes on acute angles.
- **Remember that transforms scale the stroke width**; a 2-pixel stroke drawn under a 2× scale appears 4 pixels wide.
- **Always dispose `Graphics2D` copies.**

#### Constraints and Limitations

- `BasicStroke` is immutable; do not attempt to modify its state.
- Custom strokes may be costly to compute; cache the resulting shapes when reused.
- Very thick strokes may overlap the shape's interior when drawn.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Dashboard with Cached Strokes

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.Line2D;
import java.awt.geom.Rectangle2D;

public class BestPracticeStroke extends JPanel {
    private static final BasicStroke AXIS = new BasicStroke(2f);
    private static final BasicStroke GRID = new BasicStroke(1f,
        BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER, 10f,
        new float[]{4f, 4f}, 0f);
    private static final BasicStroke DATA = new BasicStroke(3f,
        BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND);

    public BestPracticeStroke() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();                     // 1. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);  // 2. AA

            // Grid lines (dashed)
            g2.setColor(new Color(220, 220, 220));
            g2.setStroke(GRID);                                       // 3. Cached
            for (int x = 50; x <= 450; x += 40) {
                g2.draw(new Line2D.Double(x, 30, x, 270));
            }

            // Axes (solid, 2 px)
            g2.setColor(Color.DARK_GRAY);
            g2.setStroke(AXIS);                                       // 4. Axis
            g2.draw(new Line2D.Double(50, 270, 450, 270));
            g2.draw(new Line2D.Double(50, 30, 50, 270));

            // Data line (thick, rounded)
            g2.setColor(new Color(0, 120, 215));
            g2.setStroke(DATA);                                       // 5. Data
            g2.draw(new Rectangle2D.Double(90, 200, 40, 70));
            g2.draw(new Rectangle2D.Double(170, 150, 40, 120));
            g2.draw(new Rectangle2D.Double(250, 100, 40, 170));
            g2.draw(new Rectangle2D.Double(330, 170, 40, 100));
        } finally {
            g2.dispose();                                            // 6. Dispose
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Stroke");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticeStroke());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white dashboard with dashed grid lines, solid axes, and four blue rounded rectangles (as data bars).

**Why the code produces this result.** The strokes are cached as static finals and reused. Anti-aliasing is enabled. Grid lines use a dashed stroke with `CAP_BUTT` for crisp dashes; axes use a 2-pixel solid stroke; data bars use a 3-pixel stroke with round caps and joins. The `Graphics2D` copy is disposed.

---

#### Example 2: Common Mistake — Allocating Strokes in Paint

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        for (int i = 0; i < 100; i++) {
            g2.setStroke(new BasicStroke(i % 5 + 1));                // WRONG
            g2.drawLine(10, i * 3, 400, i * 3);
        }
    } finally {
        g2.dispose();
    }
}
```

**Expected Output.** Slow painting because 100 `BasicStroke` objects are allocated per repaint (potentially dozens of times per second).

**Why this is wrong.** `BasicStroke` allocation in paint methods is a performance anti-pattern. Cache the strokes as fields (or use `switch` on precomputed strokes). The correct approach is to precompute a small set of strokes and reuse them.

---

#### Example 3: Common Mistake — Ignoring Transform Scaling

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        g2.setStroke(new BasicStroke(2f));                            // WRONG: 2 px assumed
        g2.scale(4, 4);
        g2.draw(new Rectangle2D.Double(10, 10, 100, 100));
    } finally {
        g2.dispose();
    }
}
```

**Expected Output.** The rectangle is drawn at 4× scale, and the stroke is scaled accordingly, producing a visually 8-pixel-wide outline — not 2 pixels as the code suggests.

**Why this is wrong.** Strokes are defined in **user space** and are affected by the current transform. To preserve a visually consistent stroke width across scales, either set the transform before setting the stroke (with the stroke width scaled by the inverse transform) or draw with `create()` and apply the transform only to the geometry. The correct approach depends on the desired effect: for a constant visual width, use `(Graphics2D) g.create()` with `setTransform(null)` before drawing, or scale the stroke width inversely.

---

#### Example 4: Common Mistake — Using Miter Joins on Very Sharp Angles

```java
g2.setStroke(new BasicStroke(6f, BasicStroke.CAP_BUTT,
                             BasicStroke.JOIN_MITER));                // WRONG
Path2D sharp = new Path2D.Double();
sharp.moveTo(10, 100);
sharp.lineTo(200, 10);
sharp.lineTo(10, 20);
g2.draw(sharp);
```

**Expected Output.** The sharp angle at (200, 10) produces a long, ugly miter spike extending far beyond the vertex.

**Why this is wrong.** With `JOIN_MITER` and a large width, acute angles produce extreme miter spikes. The solution is either to reduce the miter limit (e.g., `new BasicStroke(6f, CAP_BUTT, JOIN_MITER, 4f)`) to clip the spike, or to use `JOIN_ROUND` or `JOIN_BEVEL` to avoid spikes entirely.

---

### Real-World Cases

- **Production dashboards** cache strokes as `static final` fields.
- **Vector editors** use dashes for selection marquees and alignment guides.
- **Games** use strokes with anti-aliasing for custom graphics.
- **Maps** use dashed strokes for routes and boundaries.
- **Educational examples** demonstrate cached strokes, miter limits, and transform interactions from the start.

---

## References

Oracle. *Interface Stroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Stroke.html

Oracle. *Class BasicStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/BasicStroke.html

Oracle. *Class Graphics2D — setStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html

Oracle. *Stroking and Filling Graphics Primitives*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/2d/geometry/strokeandfill.html

Oracle. *Class Shape*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Shape.html

Oracle. *Class Path2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Path2D.html

Oracle. *Class Line2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Line2D.html

Oracle. *Class Rectangle2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/Rectangle2D.html

Oracle. *Class RenderingHints*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/RenderingHints.html

Oracle. *Class AffineTransform*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/geom/AffineTransform.html

Oracle. *Java 2D API — Strokes*. https://www.oracle.com/java/technologies/2d/overview.html

Oracle. *Painting in AWT and Swing*. https://www.oracle.com/java/technologies/painting.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading