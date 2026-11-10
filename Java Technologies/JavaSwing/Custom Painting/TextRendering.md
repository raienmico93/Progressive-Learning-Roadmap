# Text Rendering — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Text rendering** in Java 2D is the process of converting a sequence of characters into visible glyphs on a graphics surface, with precise control over typographic layout, measurement, and visual quality. The `java.awt.font` package provides the low‑level classes that make this possible: **`TextLayout`** (the immutable graphical representation of styled text), **`FontRenderContext`** (the measurement and rendering configuration), and **`GlyphVector`** (the collection of shaped glyphs that make up a line of text).

**Technical Definition.** `TextLayout` is described as "an immutable graphical representation of styled character data" that provides metric information such as ascent, descent, advance, and leading, and is rendered using its `draw` method. `FontRenderContext` "stores the information necessary to precisely measure the size of text" and is used by a number of `Font` methods and by `TextLayout` and `LineBreakMeasurer`; it is most commonly obtained by calling `getFontRenderContext()` on a `Graphics2D` object. `GlyphVector` is a collection of glyphs with metrics and positions, obtained from `Font.createGlyphVector(FontRenderContext, String)` or `Font.layoutGlyphVector(...)`; it provides `getGlyphOutline(int)`, `getGlyphPosition(int)`, and `getGlyphMetrics(int)` for per‑glyph shape, position, and metric access. All text rendering and measurement must occur on the Event Dispatch Thread (EDT) for Swing components.

**Beginner-Friendly Explanation.** **Text rendering** is how Java turns a `String` into pixels on the screen with precise control. **`FontRenderContext`** is the “ruler and settings” — it records the transform, anti‑aliasing state, and fractional‑metrics setting used for measuring and drawing text. **`TextLayout`** is a **pre‑measured, ready‑to‑draw line of text**: it knows its ascent, descent, width, and baseline, and it can draw itself onto a `Graphics2D` at a specified position. **`GlyphVector`** is the **detailed breakdown** of the text into individual glyphs, each with its own outline, position, and metrics — useful when you need to manipulate or inspect characters individually. Think of `FontRenderContext` as the **workspace settings**, `TextLayout` as a **finished typeset line**, and `GlyphVector` as the **individual letter stamps** used to compose that line.

---

### Key Characteristics

1. **Three complementary layers.** `FontRenderContext` configures measurement; `TextLayout` provides a measured, drawable line; `GlyphVector` exposes individual glyph geometry.
2. **Precision measurement.** `TextLayout` provides `getAscent()`, `getDescent()`, `getLeading()`, and `getAdvance()` for precise line positioning.
3. **Immutable text representation.** `TextLayout` is immutable and safe to cache and reuse.
4. **Glyph‑level access.** `GlyphVector` allows per‑glyph outline extraction, position querying, and metrics inspection.
5. **Internationalisation support.** `TextLayout` and `GlyphVector` handle bidirectional text, complex scripts, and font fallback.
6. **Rendering‑context‑sensitive.** Measurement results depend on the `FontRenderContext` (transform, anti‑aliasing, fractional metrics).
7. **Caching‑friendly.** `TextLayout` and `GlyphVector` are immutable and can be cached for performance.
8. **EDT discipline.** All text rendering and measurement for Swing must occur on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and anonymous classes.
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and the `create()`/`dispose()` idiom.
- **`Font` basics:** `Font`, `FontMetrics`, and `deriveFont`.
- **Coordinate system:** baseline‑relative coordinates for text metrics.
- **Basic typographic concepts:** ascent, descent, leading, advance, and baseline.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`java.awt.font` package** — contains `TextLayout`, `FontRenderContext`, `GlyphVector`, `LineMetrics`, `LineBreakMeasurer`, `TextAttribute`, and related classes.
- **`java.awt.Font`** — creates `GlyphVector` instances and provides `getStringBounds`.
- **`java.awt.Graphics2D`** — provides `getFontRenderContext()` and draws text.
- **`java.awt.font.LineBreakMeasurer`** — breaks styled text into lines for paragraph layout.
- **`java.awt.font.TextAttribute`** — defines attributes for styled text (font, size, weight, etc.).
- **`java.text.AttributedCharacterIterator`** — iterates styled text runs.
- **`RenderingHints.KEY_TEXT_ANTIALIASING`** — controls text anti‑aliasing quality.
- **`java.awt.geom.AffineTransform`** — used within `FontRenderContext`.

---

## Core Concepts and Features

---

## 1. Text Rendering — The Concept

### Definitions

**Core Definition.** Text rendering is the process of laying out and drawing styled character data as glyphs on a graphics surface, with precise control over measurement and visual quality.

**Technical Definition.** Text rendering in Java 2D involves three cooperating components: a `Font` that provides the glyph shapes; a `FontRenderContext` that configures measurement and rendering; and a rendering target (`Graphics2D`). `TextLayout` encapsulates a fully measured line of text, while `GlyphVector` exposes the individual glyphs that compose it.

**Beginner-Friendly Explanation.** Text rendering is how Java draws text with precision. You configure the workspace with `FontRenderContext`, measure and draw with `TextLayout`, and inspect individual characters with `GlyphVector`.

---

### Purposes

- To draw text with precise control over position, alignment, and line spacing.
- To measure text accurately for layout and centring.
- To handle complex scripts, bidirectional text, and font fallback.
- To enable advanced typographic effects (glyph manipulation, path‑based text).
- To support high‑quality anti‑aliased text rendering.
- To provide a foundation for paragraph layout via `LineBreakMeasurer`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    FontRenderContext frc = g2.getFontRenderContext();           // 1. Context
    Font font = new Font("SansSerif", Font.PLAIN, 16);
    TextLayout layout = new TextLayout("Hello", font, frc);      // 2. Layout
    float ascent = layout.getAscent();                            // 3. Measure
    float descent = layout.getDescent();
    float advance = layout.getAdvance();
    layout.draw(g2, x, y + ascent);                               // 4. Draw
} finally {
    g2.dispose();
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `FontRenderContext` | Configures measurement (transform, AA, fractional metrics). |
| `TextLayout` | A measured, drawable line of text. |
| `getAscent()` | Height above the baseline. |
| `getDescent()` | Depth below the baseline. |
| `getAdvance()` | Width of the line. |
| `draw(g2, x, y)` | Draws the layout at the baseline position. |

#### Syntax Rules

- Obtain the `FontRenderContext` from `Graphics2D.getFontRenderContext()`.
- `TextLayout` is immutable and safe to cache.
- `draw` uses the baseline as the y‑coordinate, not the top of the text.
- All rendering must occur on the EDT.

#### Constraints and Limitations

- `TextLayout` does not wrap text automatically; use `LineBreakMeasurer` for paragraphs.
- Measurement results depend on the `FontRenderContext` used.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal TextLayout Rendering

```java
import javax.swing.*;
import java.awt.*;
import java.awt.font.*;

public class TextLayoutPanel extends JPanel {
    public TextLayoutPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);

            FontRenderContext frc = g2.getFontRenderContext();         // 1. FRC
            Font font = new Font("SansSerif", Font.BOLD, 24);
            TextLayout layout = new TextLayout("Hello, TextLayout!",
                                               font, frc);              // 2. Layout

            float x = 20, y = 100;
            g2.setColor(Color.DARK_GRAY);
            layout.draw(g2, x, y);                                      // 3. Draw
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("TextLayout Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new TextLayoutPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text "Hello, TextLayout!" drawn in bold 24‑point SansSerif, with the baseline at y = 100.

**Why the code produces this result.** `TextLayout` is constructed with the text, the font, and the `FontRenderContext`. `draw` positions the text so that its baseline aligns with the given y‑coordinate.

---

### Real-World Cases

- **Custom‑painted components** that display text with precise positioning.
- **Charts** that need measured axis labels and titles.
- **Text editors** with custom rendering.
- **Internationalised applications** requiring complex script support.

---

## 2. TextLayout — The Measured Text Representation

### Definitions

**Core Definition.** `TextLayout` is an immutable graphical representation of styled character data. It provides metric information and can draw itself onto a `Graphics2D`.

**Technical Definition.** `public final class TextLayout extends Object implements Cloneable`. It provides `getAscent()`, `getDescent()`, `getLeading()`, `getAdvance()`, `getBounds()`, `getPixelBounds()`, `draw(Graphics2D, float, float)`, `getCaretShapes(int)`, `getCursorShapes()`, and `getJustifiedLayout(float)` methods. It is immutable and may be cached.

**Beginner-Friendly Explanation.** `TextLayout` is a **pre‑measured line of text** that knows its own dimensions and can draw itself. You create it once, measure it with `getAscent`, `getDescent`, `getAdvance`, and reuse it.

---

### Purposes

- To provide a measured, drawable line of text.
- To supply exact typographic metrics (ascent, descent, leading, advance).
- To enable precise positioning and alignment.
- To support justified text.
- To expose caret and cursor shapes for text editing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
FontRenderContext frc = g2.getFontRenderContext();
TextLayout layout = new TextLayout(text, font, frc);

float ascent = layout.getAscent();
float descent = layout.getDescent();
float leading = layout.getLeading();
float advance = layout.getAdvance();

layout.draw(g2, x, y + ascent);          // baseline at y + ascent
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getAscent()` | Distance from top to baseline. |
| `getDescent()` | Distance from baseline to bottom. |
| `getLeading()` | Extra space between lines. |
| `getAdvance()` | Width of the line. |
| `getBounds()` | Logical bounds in baseline‑relative coordinates. |
| `draw(g2, x, y)` | Draws at the baseline position. |

#### Syntax Rules

- Construct with `new TextLayout(String, Font, FontRenderContext)`.
- For styled text, use `new TextLayout(AttributedCharacterIterator, FontRenderContext)`.
- `draw` expects the baseline y‑coordinate; for top‑aligned drawing, add `getAscent()`.

#### Constraints and Limitations

- `TextLayout` does not wrap automatically.
- Re‑measurement is needed if the `FontRenderContext` changes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Measured Text with Ascent and Descent

```java
import javax.swing.*;
import java.awt.*;
import java.awt.font.*;

public class MeasuredTextPanel extends JPanel {
    public MeasuredTextPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            Font font = new Font("Serif", Font.PLAIN, 20);
            TextLayout layout = new TextLayout("Measured text",
                                               font, g2.getFontRenderContext()); // 1. Layout

            float ascent = layout.getAscent();                          // 2. Ascent
            float descent = layout.getDescent();                        // 3. Descent
            float advance = layout.getAdvance();                        // 4. Advance

            float x = 50, baseline = 100;
            g2.setColor(Color.LIGHT_GRAY);
            g2.drawLine((int) x, (int) (baseline - ascent),
                        (int) (x + advance), (int) (baseline - ascent));  // 5. Top
            g2.drawLine((int) x, (int) baseline,
                        (int) (x + advance), (int) baseline);            // 6. Baseline
            g2.drawLine((int) x, (int) (baseline + descent),
                        (int) (x + advance), (int) (baseline + descent)); // 7. Bottom

            g2.setColor(Color.BLACK);
            layout.draw(g2, x, baseline);                                // 8. Draw
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Measured Text");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new MeasuredTextPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text "Measured text" with three horizontal guide lines: one at the ascent (top), one at the baseline, and one at the descent (bottom).

**Why the code produces this result.** `getAscent`, `getDescent`, and `getAdvance` provide the exact typographic metrics. The guide lines visualise these boundaries relative to the baseline.

---

### Real-World Cases

- **Centred text:** `x = (width − layout.getAdvance()) / 2`.
- **Top‑aligned text:** draw at `y + layout.getAscent()`.
- **Line spacing:** advance the y‑position by `ascent + descent + leading`.

---

## 3. FontRenderContext — The Measurement Configuration

### Definitions

**Core Definition.** `FontRenderContext` stores the information necessary to precisely measure the size of text — the transform, anti‑aliasing state, and fractional‑metrics setting.

**Technical Definition.** `public class FontRenderContext extends Object`. It provides `isAntiAliased()`, `usesFractionalMetrics()`, and `getTransform()`. It is used by `Font.getStringBounds`, `Font.getLineMetrics`, `TextLayout`, and `LineBreakMeasurer`. It is obtained from `Graphics2D.getFontRenderContext()`.

**Beginner-Friendly Explanation.** `FontRenderContext` is the **workspace settings** for text measurement. It records whether text is anti‑aliased, whether fractional metrics are used, and what transform is applied.

---

### Purposes

- To configure measurement precision (fractional vs. integer metrics).
- To configure anti‑aliasing state for measurement.
- To provide a consistent measurement context for `TextLayout` and `Font.getStringBounds`.
- To support transformation‑aware measurement.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
FontRenderContext frc = g2.getFontRenderContext();                 // 1. From G2D
boolean aa = frc.isAntiAliased();                                   // 2. AA state
boolean fm = frc.usesFractionalMetrics();                           // 3. Fractional
AffineTransform tx = frc.getTransform();                            // 4. Transform
Rectangle2D bounds = font.getStringBounds(text, frc);              // 5. Measure
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `isAntiAliased()` | Whether anti‑aliasing is enabled. |
| `usesFractionalMetrics()` | Whether fractional metrics are used. |
| `getTransform()` | The transform applied to measurement. |
| `font.getStringBounds(text, frc)` | Measures the logical bounds of the string. |

#### Syntax Rules

- Obtain from `Graphics2D.getFontRenderContext()`; the constructor is rarely used directly.
- Different `FontRenderContext` instances produce different measurement results.

#### Constraints and Limitations

- `getStringBounds` returns logical bounds, which may not enclose all glyphs (e.g., accents).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Measuring Text with FontRenderContext

```java
import javax.swing.*;
import java.awt.*;
import java.awt.font.FontRenderContext;
import java.awt.geom.Rectangle2D;

public class MeasurePanel extends JPanel {
    public MeasurePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 150));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            Font font = new Font("SansSerif", Font.PLAIN, 18);
            FontRenderContext frc = g2.getFontRenderContext();        // 1. FRC
            String text = "Measure me!";
            Rectangle2D bounds = font.getStringBounds(text, frc);      // 2. Measure

            System.out.println("Width: " + bounds.getWidth());
            System.out.println("Height: " + bounds.getHeight());

            g2.setColor(Color.BLUE);
            g2.drawString(text, 20, 80);                               // 3. Draw
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Measure");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new MeasurePanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints the width and height of "Measure me!"; the text is drawn in blue.

---

### Real-World Cases

- **Centring text:** use `getStringBounds` to compute x‑position.
- **Layout:** measure strings to size components.
- **Consistent measurement:** cache the `FontRenderContext` for repeated measurements.

---

## 4. GlyphVector — The Glyph Collection

### Definitions

**Core Definition.** `GlyphVector` is a collection of glyphs, each with its own outline, position, and metrics, that together represent a run of text.

**Technical Definition.** `public abstract class GlyphVector extends Object implements Cloneable`. It provides `getGlyphOutline(int)`, `getGlyphPosition(int)`, `getGlyphMetrics(int)`, `getGlyphCharIndex(int)`, `getNumGlyphs()`, `getFont()`, `getFontRenderContext()`, and `getPixelBounds(FontRenderContext, float, float)`. It is obtained from `Font.createGlyphVector(FontRenderContext, String)` or `Font.layoutGlyphVector(...)`.

**Beginner-Friendly Explanation.** `GlyphVector` is the **individual letter stamps** that make up a line of text. You can get the outline shape of each glyph, its position, and its metrics — useful for per‑character manipulation.

---

### Purposes

- To access individual glyph outlines for custom rendering.
- To obtain per‑glyph positions and metrics.
- To support path‑based text, text effects, and character‑level animation.
- To enable glyph‑level measurement for advanced typography.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
FontRenderContext frc = g2.getFontRenderContext();
Font font = new Font("Serif", Font.PLAIN, 24);
GlyphVector gv = font.createGlyphVector(frc, "Hello");              // 1. Create

int num = gv.getNumGlyphs();
for (int i = 0; i < num; i++) {
    Shape glyph = gv.getGlyphOutline(i);                             // 2. Outline
    Point2D pos = gv.getGlyphPosition(i);                            // 3. Position
    GlyphMetrics m = gv.getGlyphMetrics(i);                          // 4. Metrics
    // Use glyph, pos, m
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getNumGlyphs()` | Number of glyphs. |
| `getGlyphOutline(i)` | Shape of glyph `i` centred on its origin. |
| `getGlyphPosition(i)` | Position of glyph `i`. |
| `getGlyphMetrics(i)` | Metrics of glyph `i`. |
| `getGlyphCharIndex(i)` | Character index corresponding to glyph `i`. |

#### Syntax Rules

- Create via `Font.createGlyphVector(FontRenderContext, String)`.
- Glyph outlines are centred on the glyph's origin; translate by position to place correctly.
- `GlyphVector` is immutable.

#### Constraints and Limitations

- A `GlyphVector` is tied to a specific `Font` and `FontRenderContext`.
- Complex scripts may produce glyphs that do not map one‑to‑one with characters.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Drawing Individual Glyph Outlines

```java
import javax.swing.*;
import java.awt.*;
import java.awt.font.*;
import java.awt.geom.*;

public class GlyphVectorPanel extends JPanel {
    public GlyphVectorPanel() {
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
            Font font = new Font("Serif", Font.BOLD, 40);
            FontRenderContext frc = g2.getFontRenderContext();
            GlyphVector gv = font.createGlyphVector(frc, "Hello");   // 1. GlyphVector

            float x = 20, y = 100;
            g2.setColor(Color.BLUE);
            for (int i = 0; i < gv.getNumGlyphs(); i++) {
                Shape glyph = gv.getGlyphOutline(i);                 // 2. Outline
                Point2D pos = gv.getGlyphPosition(i);                // 3. Position
                AffineTransform tx = AffineTransform.getTranslateInstance(
                    x + pos.getX(), y + pos.getY());                 // 4. Translate
                Shape placed = tx.createTransformedShape(glyph);
                g2.fill(placed);
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("GlyphVector");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new GlyphVectorPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The word "Hello" drawn as individually filled glyph outlines.

**Why the code produces this result.** Each glyph's outline is extracted with `getGlyphOutline`, translated by its position, and filled.

---

### Real-World Cases

- **Path‑based text** that follows a curve.
- **Text effects** (gradient‑filled, stroked, or distorted text).
- **Glyph‑level animation** and manipulation.
- **Custom text rendering** for games and visualisations.

---

## 5. Text Rendering Best Practices

### Definitions

**Core Definition.** Text rendering best practices are the conventions and patterns that ensure text is measured correctly, rendered smoothly, cached efficiently, and integrated cleanly with Swing components.

**Technical Definition.** Best practices include: using `TextLayout` for measured lines; caching `TextLayout` and `GlyphVector` instances; obtaining the `FontRenderContext` from the `Graphics2D` used for rendering; enabling `KEY_TEXT_ANTIALIASING`; using `getStringBounds` for logical bounds and `TextLayout.getBounds` for visual bounds; preferring `LineBreakMeasurer` for paragraphs; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for text rendering are the “rules of thumb” that keep text looking good and rendering fast. They include things like “cache your layouts,” “use the right context,” “enable anti‑aliasing,” and “use `LineBreakMeasurer` for paragraphs.”

---

### Purposes

- To ensure accurate measurement and positioning.
- To avoid unnecessary re‑layout of text.
- To improve rendering quality with anti‑aliasing.
- To handle paragraphs and styled text correctly.
- To keep the UI responsive.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyPanel extends JPanel {
    private static final Font FONT = new Font("SansSerif", Font.PLAIN, 16);

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            FontRenderContext frc = g2.getFontRenderContext();       // 1. FRC
            TextLayout layout = new TextLayout("Cached text", FONT, frc); // 2. Cache
            float x = (getWidth() - layout.getAdvance()) / 2;       // 3. Centre
            float y = (getHeight() + layout.getAscent()
                       - layout.getDescent()) / 2;                   // 4. Vertical
            layout.draw(g2, x, y);                                   // 5. Draw
        } finally {
            g2.dispose();
        }
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `TextLayout` | For measured, drawable lines. |
| Cache layouts | Store as fields when text is static. |
| Obtain `FontRenderContext` from `Graphics2D` | `g2.getFontRenderContext()`. |
| Enable anti‑aliasing | `KEY_TEXT_ANTIALIASING` |
| Use `getStringBounds` for logical bounds | `font.getStringBounds(text, frc)` |
| Use `LineBreakMeasurer` for paragraphs | `new LineBreakMeasurer(iterator, frc)` |
| Dispose `Graphics2D` copies | `try/finally { g2.dispose(); }` |
| EDT | All rendering on the EDT. |

#### Syntax Rules

- **Always obtain `FontRenderContext` from the `Graphics2D`** used for rendering.
- **Always cache `TextLayout` and `GlyphVector`** for static text.
- **Always enable text anti‑aliasing** for smooth rendering.
- **Always use `LineBreakMeasurer`** for multi‑line paragraph layout.
- **Always dispose `Graphics2D` copies.**

#### Constraints and Limitations

- `FontRenderContext` changes invalidate cached measurements.
- Complex scripts may require `TextLayout` for correct shaping.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Cached Centred Text with TextLayout

```java
import javax.swing.*;
import java.awt.*;
import java.awt.font.*;

public class BestPracticeText extends JPanel {
    private TextLayout layout;

    public BestPracticeText() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            if (layout == null) {                                     // 1. Cache
                Font font = new Font("SansSerif", Font.BOLD, 24);
                layout = new TextLayout("Centred Text", font,
                                        g2.getFontRenderContext());
            }
            float x = (getWidth() - layout.getAdvance()) / 2f;       // 2. Centre X
            float y = (getHeight() + layout.getAscent()
                       - layout.getDescent()) / 2f;                   // 3. Centre Y
            g2.setColor(Color.DARK_GRAY);
            layout.draw(g2, x, y);                                   // 4. Draw
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Text");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticeText());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text "Centred Text" perfectly centred both horizontally and vertically in the panel.

**Why the code produces this result.** `layout.getAdvance()` gives the width; `layout.getAscent()` and `getDescent()` give the vertical extents for centring. The layout is cached and reused.

---

#### Example 2: Common Mistake — Creating TextLayout in Every Paint

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        for (int i = 0; i < 100; i++) {
            TextLayout layout = new TextLayout("Line " + i,
                new Font("SansSerif", Font.PLAIN, 12),
                g2.getFontRenderContext());                          // WRONG
            layout.draw(g2, 10, i * 15);
        }
    } finally {
        g2.dispose();
    }
}
```

**Expected Output.** Slow painting because 100 `TextLayout` and `Font` objects are allocated per repaint.

**Why this is wrong.** `TextLayout` construction is expensive. Cache layouts for static text.

---

#### Example 3: Common Mistake — Using the Wrong FontRenderContext

```java
FontRenderContext frc = new FontRenderContext(null, false, false); // WRONG
TextLayout layout = new TextLayout("Text", font, frc);
```

**Expected Output.** Measurement may differ from the actual rendering context, causing misalignment.

**Why this is wrong.** The `FontRenderContext` should be obtained from the `Graphics2D` used for rendering so that measurement matches rendering.

---

### Real-World Cases

- **Custom chart labels** centred with `TextLayout`.
- **Paragraph layout** with `LineBreakMeasurer`.
- **Text effects** with `GlyphVector`.
- **Internationalised text** with complex script support.

---

## References

Oracle. *Class TextLayout*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/TextLayout.html

Oracle. *Class FontRenderContext*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/FontRenderContext.html

Oracle. *Class GlyphVector*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/GlyphVector.html

Oracle. *Class GlyphMetrics*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/GlyphMetrics.html

Oracle. *Class LineMetrics*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/LineMetrics.html

Oracle. *Class LineBreakMeasurer*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/LineBreakMeasurer.html

Oracle. *Class TextAttribute*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/font/TextAttribute.html

Oracle. *Class Font*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Font.html

Oracle. *Class Graphics2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html

Oracle. *Class RenderingHints*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/RenderingHints.html

Oracle. *The java.awt.font Package*. O'Reilly Java Foundation Classes. https://cs.ait.ac.th/~on/O/oreilly/java-ent/jfc/ch15_01.htm

Oracle. *Uses of Class java.awt.font.FontRenderContext*. OpenJDK Documentation. https://cr.openjdk.org/~pminborg/stable-values/api/java.desktop/java/awt/font/class-use/FontRenderContext.html

Oracle. *TextLayout Java API — getAdvance, getAscent, getDescent, getLeading*. https://docs.oracle.com/javase/8/docs/api/java/awt/font/TextLayout.html

Oracle. *GlyphVector — getGlyphOutline, getGlyphPosition, getGlyphMetrics*. https://docs.oracle.com/javase/8/docs/api/java/awt/font/GlyphVector.html

Stack Overflow. *Drawing justified Arabic text with TextLayout and LineBreakMeasurer*. https://stackoverflow.com/posts/50374846/edit#1

Stack Overflow. *Java text rendering performance — caching TextLayout*. https://mail.openjdk.org/pipermail/openjfx-dev/

Stack Overflow. *TextLayout and FontRenderContext for anti-aliased text*. https://stackoverflow.com/revisions/9994727b-365f-4957-a4e2-0444172cb57a/view-source