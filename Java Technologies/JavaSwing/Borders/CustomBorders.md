# Custom Borders — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **Custom Border** is a user-implemented `javax.swing.border.Border` (almost always by extending `AbstractBorder`) that paints a unique visual decoration around a component and reports the space that decoration occupies as insets. It is the standard extension point for adding frames, edges, focus rings, patterned margins, and compound decorations that Swing's built-in borders do not provide.

**Technical Definition.** `public interface Border` declares `void paintBorder(Component c, Graphics g, int x, int y, int width, int height)`, `Insets getBorderInsets(Component c)`, `boolean isBorderOpaque()`, and — since Java 9 — the default method `Insets getBorderInsets(Component c, Insets insets)`. `public abstract class AbstractBorder implements Border, Serializable` supplies default implementations and convenience helpers (`getInteriorRectangle(...)`, `getBaseline(...)`, `getBaselineResizeBehavior(...)`). A custom border overrides `paintBorder` to draw its decoration inside the rectangle `(x, y, width, height)`, overrides `getBorderInsets` to report the space it consumes, and optionally overrides `isBorderOpaque` to signal whether it fills its entire painted area. A well-behaved border is **immutable**, **stateless** (or thread‑safe with no mutable state used across paint calls), **cheap to paint**, **paints only within its own bounds**, and reports insets that exactly match its painted area. Borders may implement `javax.swing.plaf.UIResource` (or be wrapped by `BorderUIResource`) to indicate that the look and feel may replace them when the L&F changes; custom borders that should persist across L&F changes must **not** implement `UIResource`. All border painting and installation should occur on the Event Dispatch Thread (EDT).

**Beginner‑Friendly Explanation.** A **Custom Border** is a **hand‑painted picture frame** that you write yourself. The built‑in borders (LineBorder, TitledBorder, CompoundBorder, etc.) cover most needs, but sometimes you want something they can't do: a frame with a rounded double stroke, a bar with an animated gradient, edges decorated with a repeating tile, or a dotted focus ring that only paints the top and bottom. To get that, you write a small class that extends `AbstractBorder`, override one method that draws the frame, and override another method that tells Swing how much space the frame occupies. Swing takes care of the rest: it calls your drawing method at the right time, adds your insets to the component's padding, and repaints when needed. Think of it as designing your own border “stamp”: you carve it once, and Swing presses it around every component you attach it to.

---

### Key Characteristics

1. **Interface‑driven extensibility.** Any class implementing `Border` can be installed with `JComponent.setBorder(...)`; custom borders are first‑class citizens of Swing's painting pipeline.
2. **Painted inside the border's bounds.** The rectangle `(x, y, width, height)` passed to `paintBorder` is the *only* area the border is allowed to paint; painting outside it is clipped or undefined.
3. **Insets must match painting.** Whatever `paintBorder` draws must be accounted for by `getBorderInsets`, or content will overlap with the border or be misaligned by layout managers.
4. **Immutability and statelessness.** A border should not hold mutable state that changes per component or per paint; if it needs per‑component data, read it from the `Component` argument.
5. **Cheap painting.** `paintBorder` runs on every repaint; avoid allocations, I/O, font loading, and expensive computations inside it.
6. **Opacity as an optimisation signal.** `isBorderOpaque()` should be `true` only if the border paints every pixel in its area; otherwise, keep it `false` so the parent repaints correctly.
7. **`UIResource` semantics.** Implementing `UIResource` (or wrapping in `BorderUIResource`) signals that the L&F may replace the border. Custom borders intended to survive L&F changes should omit `UIResource`.
8. **EDT discipline.** Painting, installation, and any state consultation happen on the EDT.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, anonymous classes.
- Basic Swing: `JComponent`, `JPanel`, `setBorder`, `SwingUtilities.invokeLater`.
- The painting pipeline: `paintComponent` → `paintBorder` → `paintChildren`.
- Java 2D basics: `Graphics`, `Graphics2D`, `Color`, `Font`, `Insets`, `Rectangle`, `Shape`, `Stroke`, `Paint`.
- Familiarity with `BorderFactory` and the standard built‑in border types.
- A JDK (version 9 or later recommended; `getBorderInsets(Component, Insets)` and `getBaseline` helpers are more modern).

---

### Related Programming Areas

- **`javax.swing.border.Border`** — the interface every border must implement.
- **`javax.swing.border.AbstractBorder`** — the abstract base class that makes custom borders easy.
- **`javax.swing.plaf.BorderUIResource`** — the `UIResource` wrapper for custom and standard borders.
- **`javax.swing.JComponent.setBorder` / `getBorder`** — installation methods.
- **`javax.swing.JComponent.paintBorder`** — the method that calls the installed border's `paintBorder`.
- **`javax.swing.BorderFactory`** — for mixing built‑in borders with custom ones.
- **`javax.swing.border.TitledBorder`** and **`CompoundBorder`** — built‑in borders that wrap other borders, including custom ones.
- **`java.awt.Insets` and `getInteriorRectangle`** — for computing the content area after borders.
- **`java.awt.geom.RoundRectangle2D`, `Area`, `Path2D`** — typical building blocks inside `paintBorder`.

---

## 1. Custom Borders — The Concept

### Definitions

**Core Definition.** A custom border is a `Border` implementation written by the developer to draw a decoration not provided by the built‑in border types.

**Technical Definition.** A custom border typically extends `AbstractBorder` and overrides three methods: `paintBorder`, `getBorderInsets`, and (optionally) `isBorderOpaque`. The border is installed on a `JComponent` via `setBorder` and painted by the component's `paintBorder` call. All calls are made on the EDT during the component's paint cycle.

**Beginner‑Friendly Explanation.** A custom border is a **frame that you draw yourself**. You write a class that knows how to paint the frame and how much space it takes.

---

### Purposes

- To draw visual decorations that the built‑in borders do not support (gradients, icons, rounded double strokes, patterned edges).
- To create brand‑specific frames and edges that match a custom UI.
- To implement focus rings, highlight indicators, or animated edges.
- To combine drawing and layout behaviour into a single reusable object.
- To encapsulate complex border logic in a testable, shareable class.
- To integrate with `CompoundBorder` and `TitledBorder` alongside built‑in borders.

---

### Syntax Rules and Structure

#### Complete General Syntax (Skeleton of a Custom Border)

```java
public class MyBorder extends AbstractBorder {
    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int width, int height) {
        // paint inside the given rectangle
    }

    @Override
    public Insets getBorderInsets(Component c) {
        return new Insets(top, left, bottom, right);
    }

    @Override
    public boolean isBorderOpaque() {
        return false;
    }
}
```

#### Component Breakdown

| Element | Responsibility |
|---------|----------------|
| `extends AbstractBorder` | Inherits `Border` defaults, `getInteriorRectangle`, baseline helpers. |
| `paintBorder(...)` | Draws the decoration inside `(x, y, width, height)`. |
| `getBorderInsets(...)` | Reports the space consumed by the decoration. |
| `isBorderOpaque()` | Indicates whether the border paints all its pixels. |

#### Syntax Rules

- `paintBorder` must paint only within the rectangle `(x, y, width, height)`.
- Insets must be non‑negative integers that exactly match the painted area.
- The class should be immutable: constructor‑set values only, no mutable per‑paint state.

#### Constraints and Limitations

- Borders cannot extend the component's visible bounds; they paint inside them.
- Borders on top‑level containers (`JFrame`, `JDialog`) are not supported via `setBorder`.
- Setting custom borders on look‑and‑feel‑managed components (`JButton`, `JTextField`) can break their appearance.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Custom Border

```java
import javax.swing.*;
import javax.swing.border.AbstractBorder;
import java.awt.*;

public class SimpleBorder extends AbstractBorder {                   // 1. Abstract
    private final Color color;

    public SimpleBorder(Color color) {
        this.color = color;
    }

    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int width, int height) {   // 2. Paint
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(color);
            g2.drawRect(x, y, width - 1, height - 1);
        } finally {
            g2.dispose();
        }
    }

    @Override
    public Insets getBorderInsets(Component c) {                     // 3. Insets
        return new Insets(1, 1, 1, 1);
    }

    @Override
    public boolean isBorderOpaque() { return false; }                 // 4. Opaque

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel panel = new JPanel();
            panel.setBorder(new SimpleBorder(Color.DARK_GRAY));       // 5. Install
            panel.add(new JLabel("Custom border demo"));
            JFrame frame = new JFrame("Simple Custom Border");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel surrounded by a 1‑pixel dark‑gray rectangle; the label sits just inside the frame.

**Why the code produces this result.** `paintBorder` draws the outline inside the rectangle, insets reserve 1 pixel on all sides, and `isBorderOpaque()` returns `false` because the border does not fill the whole area.

---

### Real-World Cases

- **Custom UI libraries** that need unique framing to match a design system.
- **Games and kiosk applications** that require themed frames.
- **Focus rings** and **selection indicators** drawn around custom components.
- **Brand frames** that combine logos, colors, or textures in the border itself.

---

## 2. AbstractBorder — The Base Class

### Definitions

**Core Definition.** `AbstractBorder` is the abstract base class for border implementations, providing default implementations of `Border` methods and helper methods for interior rectangles and baselines.

**Technical Definition.** `public abstract class AbstractBorder implements Border, Serializable`. It provides default `getBorderInsets(Component)` returning `(0,0,0,0)`, default `isBorderOpaque()` returning `false`, the static convenience `getInteriorRectangle(Component, int, int, int, int)` and its instance counterpart `getInteriorRectangle(Component, int, int, int, int)`, and baseline helpers (`getBaseline(JComponent, int, int)` and `getBaselineResizeBehavior(JComponent)`). The interface default `getBorderInsets(Component, Insets)` (Java 9+) can be overridden for allocation‑free insets queries.

**Beginner‑Friendly Explanation.** `AbstractBorder` is the **helper base class** for borders. It gives you working defaults so you only override what matters, and it provides utilities like “what is the rectangle inside the border”.

---

### Purposes

- To reduce boilerplate when writing custom borders.
- To provide working defaults for `getBorderInsets` and `isBorderOpaque`.
- To supply helpers that compute the border's interior rectangle.
- To support baseline alignment for components with text.
- To serve as the parent class for both built‑in and custom borders.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyBorder extends AbstractBorder {
    @Override public void paintBorder(Component c, Graphics g,
                                      int x, int y, int width, int height) { ... }
    @Override public Insets getBorderInsets(Component c) { ... }
    @Override public boolean isBorderOpaque() { ... }
    // Optional overrides:
    @Override public Insets getBorderInsets(Component c, Insets insets) { ... }
    @Override public Rectangle getInteriorRectangle(Component c,
                                                    int x, int y, int width, int height) { ... }
    @Override public int getBaseline(JComponent c, int width, int height) { ... }
    @Override public Component.BaselineResizeBehavior
            getBaselineResizeBehavior(JComponent c) { ... }
}
```

#### Component Breakdown

| Method | Default Behaviour |
|--------|-------------------|
| `getBorderInsets(Component)` | Returns `new Insets(0, 0, 0, 0)`. |
| `isBorderOpaque()` | Returns `false`. |
| `getBorderInsets(Component, Insets)` | Fills the supplied `Insets` with `getBorderInsets(c)` and returns it. |
| `getInteriorRectangle(...)` (static) | Subtracts insets from the rectangle. |
| `getBaseline(...)` | Returns `-1` (no baseline). |
| `getBaselineResizeBehavior(...)` | Returns `OTHER`. |

#### Syntax Rules

- Override `getBorderInsets(Component, Insets)` when insets are queried frequently, to avoid allocating new `Insets` per query.
- Override baseline helpers for borders that affect text alignment.

#### Constraints and Limitations

- The default `getBorderInsets` returns zero, so a border that paints a decoration **must** override it.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Allocation‑Free Insets

```java
import javax.swing.*;
import javax.swing.border.AbstractBorder;
import java.awt.*;

public class EfficientBorder extends AbstractBorder {
    private final Insets myInsets = new Insets(4, 6, 4, 6);           // 1. Cached

    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int w, int h) {
        g.setColor(Color.GRAY);
        g.drawRect(x, y, w - 1, h - 1);
    }

    @Override
    public Insets getBorderInsets(Component c) {
        return myInsets;                                               // 2. Reuse
    }

    @Override
    public Insets getBorderInsets(Component c, Insets insets) {        // 3. No alloc
        insets.set(4, 6, 4, 6);
        return insets;
    }

    @Override public boolean isBorderOpaque() { return false; }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel panel = new JPanel();
            panel.setBorder(new EfficientBorder());
            panel.add(new JLabel("Allocation-free insets"));
            JFrame frame = new JFrame("Efficient Border");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a gray rectangle and 4/6‑pixel insets, no new `Insets` allocated per query.

---

#### Example 2: Interior Rectangle Helper

```java
Rectangle bounds = new Rectangle(0, 0, c.getWidth(), c.getHeight());
Rectangle interior = getInteriorRectangle(c, 0, 0,
                                          c.getWidth(), c.getHeight()); // 1. Interior
```

**Expected Output.** `interior` is the content area minus the border's insets.

---

### Real-World Cases

- **Custom borders** that use `getInteriorRectangle` to align child components.
- **Text‑containing components** that rely on baselines.
- **Performance‑sensitive borders** that override the allocation‑free insets variant.

---

## 3. Custom Border Implementation — paintBorder

### Definitions

**Core Definition.** `paintBorder` is the method where the border draws its visual decoration.

**Technical Definition.** `public void paintBorder(Component c, Graphics g, int x, int y, int width, int height)` is invoked by the component's paint pipeline. The arguments describe the rectangle in which the border may paint. The method receives a shared `Graphics` object; safe practice is to derive a `Graphics2D` copy, set the state, paint, and dispose.

**Beginner‑Friendly Explanation.** `paintBorder` is where you draw the frame. Everything must fit inside the rectangle `(x, y, width, height)`.

---

### Purposes

- To draw the border's visual decoration.
- To adapt drawing to the component's size and state.
- To use the component's properties (foreground, background, font) when the border should follow the component.
- To compose multi‑stage decorations (background, foreground, focus).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void paintBorder(Component c, Graphics g,
                        int x, int y, int width, int height) {
    Graphics2D g2 = (Graphics2D) g.create();       // 1. Private copy
    try {
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                            RenderingHints.VALUE_ANTIALIAS_ON);
        g2.setColor(color);                        // 2. Paint state
        g2.drawRect(x, y, width - 1, height - 1);  // 3. Draw inside
    } finally {
        g2.dispose();                              // 4. Dispose
    }
}
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Component c` | The component being decorated. Use for properties and state. |
| `Graphics g` | The shared paint context; do not retain it. |
| `int x, y` | Origin of the border rectangle. |
| `int width, height` | Size of the border rectangle. |

#### Syntax Rules

- Always work on a `Graphics2D` copy when modifying state; dispose it.
- Paint only inside `(x, y, width, height)`.
- Do not mutate the component inside `paintBorder`.
- Do not allocate large resources per paint call; cache them.

#### Constraints and Limitations

- Painting outside the rectangle is undefined and may be clipped.
- The `Graphics` object is shared with the rest of the paint cycle; modifying it without a copy leaks state.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Rounded Double‑Stroke Border

```java
import javax.swing.*;
import javax.swing.border.AbstractBorder;
import java.awt.*;
import java.awt.geom.RoundRectangle2D;

public class RoundedDoubleBorder extends AbstractBorder {
    private final Color outer, inner;
    private final int radius, gap;

    public RoundedDoubleBorder(Color outer, Color inner, int radius, int gap) {
        this.outer = outer;
        this.inner = inner;
        this.radius = radius;
        this.gap = gap;
    }

    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int width, int height) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setStroke(new BasicStroke(2f));

            g2.setColor(outer);
            g2.draw(new RoundRectangle2D.Double(x + 1, y + 1,
                width - 2, height - 2, radius, radius));

            g2.setColor(inner);
            g2.draw(new RoundRectangle2D.Double(
                x + 1 + gap, y + 1 + gap,
                width - 2 - 2 * gap, height - 2 - 2 * gap,
                radius, radius));
        } finally {
            g2.dispose();
        }
    }

    @Override
    public Insets getBorderInsets(Component c) {
        return new Insets(1 + gap + 1, 1 + gap + 1, 1 + gap + 1, 1 + gap + 1);
    }

    @Override public boolean isBorderOpaque() { return false; }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel panel = new JPanel();
            panel.setBorder(new RoundedDoubleBorder(
                new Color(0, 90, 160), new Color(150, 200, 240), 16, 4));
            panel.add(new JLabel("Rounded double stroke"));
            JFrame frame = new JFrame("Rounded Double Border");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a blue rounded outer stroke and a lighter rounded inner stroke, 16‑pixel corner radius, 4‑pixel spacing between strokes.

**Why the code produces this result.** Two `RoundRectangle2D` outlines are drawn on a copy of the graphics, with insets that reserve exactly the space they consume.

---

#### Example 2: Gradient Bar Border

```java
import javax.swing.*;
import javax.swing.border.AbstractBorder;
import java.awt.*;

public class GradientBarBorder extends AbstractBorder {
    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int width, int height) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setPaint(new GradientPaint(
                x, y, Color.BLUE, x + width, y, Color.CYAN));
            g2.fillRect(x, y, width, 4);                           // top bar
            g2.fillRect(x, y + height - 4, width, 4);              // bottom bar
        } finally {
            g2.dispose();
        }
    }

    @Override public Insets getBorderInsets(Component c) {
        return new Insets(4, 0, 4, 0);
    }
    @Override public boolean isBorderOpaque() { return false; }
}
```

**Expected Output.** A panel with a blue‑to‑cyan gradient bar on the top and bottom, no side edges.

---

### Real-World Cases

- **Focus rings** drawn around interactive custom components.
- **Highlight frames** around selected items in custom lists.
- **Decorative cards** with branded edges.
- **Status panels** with animated or colored bars.

---

## 4. Custom Border Implementation — getBorderInsets

### Definitions

**Core Definition.** `getBorderInsets` reports the space consumed by the border on each side.

**Technical Definition.** `public Insets getBorderInsets(Component c)` must return non‑negative top, left, bottom, and right values that match the pixels painted by `paintBorder`. The Java 9 default `getBorderInsets(Component c, Insets insets)` fills a supplied `Insets` and returns it, avoiding allocation.

**Beginner‑Friendly Explanation.** `getBorderInsets` tells Swing how much room the frame needs so the content can be positioned correctly.

---

### Purposes

- To let layout managers reserve space for the border.
- To ensure content does not overlap the border decoration.
- To enable allocation‑free queries via the Java 9 overload.
- To allow dynamic insets based on the component's state.

---

### Syntax Rules and Structure

```java
@Override
public Insets getBorderInsets(Component c) {
    return new Insets(top, left, bottom, right);
}

@Override
public Insets getBorderInsets(Component c, Insets insets) {
    insets.set(top, left, bottom, right);
    return insets;
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `top, left, bottom, right` | Pixel amounts; must match the painted decoration. |

#### Syntax Rules

- Insets must be non‑negative.
- Insets must match painting; otherwise content overlaps or leaves gaps.
- Prefer the `Insets` overload for allocation‑free queries.

#### Constraints and Limitations

- Insets are always four numbers; asymmetric painting (e.g., only bottom bar) must reflect that in insets.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Asymmetric Insets

```java
@Override public Insets getBorderInsets(Component c) {
    return new Insets(0, 8, 0, 8);                                // sides only
}
```

**Expected Output.** Eight pixels reserved on the left and right, none on top and bottom.

---

## 5. Custom Border Implementation — isBorderOpaque

### Definitions

**Core Definition.** `isBorderOpaque` signals whether the border paints every pixel in its area.

**Technical Definition.** `public boolean isBorderOpaque()` should return `true` if `paintBorder` fills the entire rectangle it is given, and `false` otherwise. The default implementation in `AbstractBorder` returns `false`.

**Beginner‑Friendly Explanation.** `isBorderOpaque` tells Swing whether the border completely covers its area or leaves the background visible.

---

### Purposes

- To inform Swing whether repainting optimisations are safe.
- To prevent stale background showing through a transparent border.
- To match the border's painting behaviour consistently.

---

### Syntax Rules and Structure

```java
@Override
public boolean isBorderOpaque() {
    return false;   // when the border does not fill every pixel
}
```

#### Syntax Rules

- Return `true` only if `paintBorder` guarantees every pixel in the rectangle is painted.
- Return `false` if the border paints only edges, uses transparency, or has holes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Opaque Fill Border

```java
public class SolidFillBorder extends AbstractBorder {
    private final Color color;
    public SolidFillBorder(Color color) { this.color = color; }

    @Override public void paintBorder(Component c, Graphics g,
                                      int x, int y, int w, int h) {
        g.setColor(color);
        g.fillRect(x, y, w, h);                                   // fills all
    }
    @Override public Insets getBorderInsets(Component c) {
        return new Insets(4, 4, 4, 4);
    }
    @Override public boolean isBorderOpaque() { return true; }    // opaque
}
```

**Expected Output.** A border that fills the component area with a solid color inside the 4‑pixel insets.

---

## 6. Additional Overrides — getInteriorRectangle, Baselines, and Insets Overload

### Definitions

**Core Definition.** `AbstractBorder` provides optional helper methods for computing the content area and for aligning text baselines.

**Technical Definition.** `getInteriorRectangle(Component c, int x, int y, int width, int height)` returns the rectangle inside the border's insets. `getBaseline(JComponent c, int width, int height)` returns the border's baseline, and `getBaselineResizeBehavior` describes how the baseline responds to resizing.

**Beginner‑Friendly Explanation.** These helpers let a border cooperate with layout and text alignment.

---

### Purposes

- To compute content positions without manual insets subtraction.
- To align a component's text with surrounding elements.
- To give layout managers predictable baseline behaviour.

---

### Syntax Rules and Structure

```java
@Override public int getBaseline(JComponent c, int width, int height) {
    Insets i = getBorderInsets(c);
    return i.top;   // baseline aligned to top inset
}
@Override public Component.BaselineResizeBehavior
        getBaselineResizeBehavior(JComponent c) {
    return Component.BaselineResizeBehavior.CONSTANT_ASCENT;
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getInteriorRectangle` | Returns the rectangle inside the border. |
| `getBaseline` | Returns the border's baseline (or `-1`). |
| `getBaselineResizeBehavior` | Enumerates how the baseline changes with resize. |

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Baseline for Text Borders

```java
@Override public int getBaseline(JComponent c, int width, int height) {
    return getBorderInsets(c).top;                               // 1. Baseline
}
@Override public Component.BaselineResizeBehavior
        getBaselineResizeBehavior(JComponent c) {
    return Component.BaselineResizeBehavior.CONSTANT_ASCENT;    // 2. Constant
}
```

**Expected Output.** A border whose baseline is the top inset, with constant ascent behaviour.

---

## 7. UIResource and BorderUIResource

### Definitions

**Core Definition.** `UIResource` is a marker interface that indicates the value is owned by the look and feel and may be replaced when the L&F changes. `BorderUIResource` is a wrapper that implements `UIResource` for any border.

**Technical Definition.** `public interface UIResource` has no methods. `BorderUIResource` extends `AbstractBorder` and delegates to an inner `Border`. Custom borders intended to be replaceable by L&F changes implement `UIResource`; custom borders intended to persist across L&F changes do **not**.

**Beginner‑Friendly Explanation.** `UIResource` is a **tag**: “the theme owns this value.” If your border should be replaceable by the L&F, tag it; if your application owns the border and it should survive theme changes, don't tag it.

---

### Purposes

- To signal that a border is part of the L&F and may be replaced.
- To preserve application‑owned custom borders across L&F changes.
- To wrap a custom border for cases where L&F replacement is desired.

---

### Syntax Rules and Structure

```java
// Replaceable by L&F changes:
public class MyLafBorder extends AbstractBorder implements UIResource { ... }

// Persistent across L&F changes:
public class MyAppBorder extends AbstractBorder { ... }

// Wrap an existing border as a UIResource:
Border wrapped = new BorderUIResource(myCustomBorder);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `implements UIResource` | Marks the border as L&F‑owned. |
| `new BorderUIResource(border)` | Wraps a border as L&F‑owned. |

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: UIResource‑Wrapped Custom Border

```java
Border original = new RoundedDoubleBorder(Color.BLUE, Color.CYAN, 12, 2);
Border uires = new BorderUIResource(original);                   // 1. Wrap
panel.setBorder(uires);
```

**Expected Output.** The border behaves identically to the original but will be replaced if the L&F changes.

---

## 8. Custom Borders — Real‑World Cases

- **Design‑system frames** with rounded corners and brand colors.
- **Focus rings** for custom text fields and canvas widgets.
- **Status bars** with gradient or animated edges.
- **Image‑tiled frames** produced by painting a tile across a strip.
- **Table and card frames** that combine a title, padding, and accent stroke.
- **Custom titled borders** that integrate with `CompoundBorder` and the standard L&F.
- **Error highlighting** around invalid inputs with a red ribbon.

---

## 9. Custom Borders Best Practices

### Definitions

**Core Definition.** Custom border best practices are the conventions that ensure custom borders are correct, performant, predictable, and compatible with layout managers and look and feels.

**Technical Definition.** Best practices include: extending `AbstractBorder`; matching insets to painted area; painting only inside the given rectangle; using a private `Graphics2D` copy and disposing it; caching strokes, paints, fonts; being immutable and stateless; using `BorderUIResource` for replaceable borders; avoiding heavy work in `paintBorder`; overriding the Java 9 `getBorderInsets(Component, Insets)` for allocation‑free queries; and testing the border with `CompoundBorder`, `TitledBorder`, and other borders.

**Beginner‑Friendly Explanation.** Best practices are the rules of thumb that keep custom borders working smoothly: paint inside the box, report correct insets, cache what can be cached, don't mutate the component, and keep the border stateless.

---

### Purposes

- To ensure layout managers position content correctly.
- To avoid paint clipping and visual artifacts.
- To keep repaint performance smooth.
- To make the border reusable and thread‑safe.
- To integrate cleanly with built‑in borders and L&F changes.

---

### Syntax Rules and Structure

#### Complete General Best‑Practice Skeleton

```java
public class BestPracticeBorder extends AbstractBorder {           // 1. Abstract
    private final Color color;
    private final Insets insets = new Insets(4, 4, 4, 4);

    public BestPracticeBorder(Color color) { this.color = color; }

    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int w, int h) {          // 2. Paint
        Graphics2D g2 = (Graphics2D) g.create();                   // 3. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(color);
            g2.drawRect(x, y, w - 1, h - 1);
        } finally {
            g2.dispose();                                          // 4. Dispose
        }
    }

    @Override public Insets getBorderInsets(Component c) {          // 5. Match
        return insets;
    }
    @Override public Insets getBorderInsets(Component c, Insets ins) {
        ins.set(insets);
        return ins;
    }
    @Override public boolean isBorderOpaque() { return false; }     // 6. Opaque
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend `AbstractBorder` | `extends AbstractBorder` |
| Match insets | `getBorderInsets` returns the exact painted thickness |
| Paint inside rectangle | Use `(x, y, w, h)` |
| Use `Graphics2D` copy | `(Graphics2D) g.create()` in `try`/`finally` |
| Cache resources | `final` fields for color, stroke, insets |
| Be stateless | No per‑paint mutations |
| Correct opacity | `isBorderOpaque()` returns `true` only if fully filled |
| Allocation‑free insets | Override `getBorderInsets(Component, Insets)` |
| `UIResource` when replaceable | Implement `UIResource` or wrap with `BorderUIResource` |
| EDT | Paint only on the EDT |

#### Syntax Rules

- **Always extend `AbstractBorder`** unless there is a specific reason not to.
- **Always match insets to painted area** — no more, no less.
- **Always paint only inside the supplied rectangle**; painting outside is clipped or undefined.
- **Always work on a `Graphics2D` copy** for state changes and dispose it.
- **Always cache fonts, strokes, paints, and insets**; do not allocate in `paintBorder`.
- **Never mutate the component** inside `paintBorder`.
- **Return `isBorderOpaque() == true`** only if every pixel is painted.

#### Constraints and Limitations

- Custom borders on look‑and‑feel‑managed components may break appearance; prefer panels.
- Very complex painters can hurt repaint performance; pre‑render to an image when necessary.
- The border is painted many times; keep it fast.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best‑Practice Custom Border

```java
import javax.swing.*;
import javax.swing.border.AbstractBorder;
import java.awt.*;

public class BestPracticeCustomBorder extends AbstractBorder {
    private final Color color;
    private final Insets insets;
    private static final BasicStroke STROKE = new BasicStroke(2f);

    public BestPracticeCustomBorder(Color color, int inset) {
        this.color = color;
        this.insets = new Insets(inset, inset, inset, inset);
    }

    @Override
    public void paintBorder(Component c, Graphics g,
                            int x, int y, int width, int height) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(color);
            g2.setStroke(STROKE);
            int t = insets.top;
            g2.drawRoundRect(x + t / 2, y + t / 2,
                             width - t, height - t, 12, 12);
        } finally {
            g2.dispose();
        }
    }

    @Override public Insets getBorderInsets(Component c) { return insets; }
    @Override public Insets getBorderInsets(Component c, Insets i) {
        i.set(insets);
        return i;
    }
    @Override public boolean isBorderOpaque() { return false; }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel panel = new JPanel();
            panel.setBorder(new BestPracticeCustomBorder(
                new Color(0, 120, 215), 6));
            panel.add(new JLabel("Best practice custom border"));
            JFrame frame = new JFrame("Best Practice Border");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a 2‑pixel rounded blue frame and 6‑pixel insets on all sides.

**Why the code produces this result.** The border caches its stroke and insets, paints a rounded rectangle inside the given bounds, and reports exact insets for layout.

---

#### Example 2: Common Mistake — Painting Outside the Border Bounds

```java
public class BadBorder1 extends AbstractBorder {
    @Override public void paintBorder(Component c, Graphics g,
                                      int x, int y, int w, int h) {
        g.setColor(Color.RED);
        g.drawRect(x - 3, y - 3, w + 6, h + 6);                   // WRONG
    }
    @Override public Insets getBorderInsets(Component c) {
        return new Insets(0, 0, 0, 0);
    }
}
```

**Expected Output.** The outer parts of the drawn rectangle are clipped; content overlaps with the visible stroke.

**Why this is wrong.** Painting outside the border rectangle is undefined, and the reported zero insets do not match the painted area.

---

#### Example 3: Common Mistake — Mismatched Insets

```java
public class BadBorder2 extends AbstractBorder {
    @Override public void paintBorder(Component c, Graphics g,
                                      int x, int y, int w, int h) {
        g.setColor(Color.GRAY);
        g.fillRect(x, y, w, 4);                                    // top bar
    }
    @Override public Insets getBorderInsets(Component c) {
        return new Insets(0, 0, 0, 0);                             // WRONG
    }
}
```

**Expected Output.** The top bar overlaps the component content because no top inset was reserved.

**Why this is wrong.** Insets must reflect the exact space consumed by the border; the top bar needs a top inset of 4.

---

#### Example 4: Common Mistake — Storing State in paintBorder

```java
public class BadBorder3 extends AbstractBorder {
    private int paintCount;                                       // WRONG
    @Override public void paintBorder(Component c, Graphics g,
                                      int x, int y, int w, int h) {
        paintCount++;
        g.setColor(paintCount % 2 == 0 ? Color.RED : Color.BLUE);
        g.drawRect(x, y, w - 1, h - 1);
    }
    @Override public Insets getBorderInsets(Component c) {
        return new Insets(1, 1, 1, 1);
    }
}
```

**Expected Output.** The border flickers between colors depending on repaint count and is not thread‑safe.

**Why this is wrong.** Borders should be stateless across paint calls; per‑paint state causes visual glitches and thread‑safety problems.

---

#### Example 5: Common Mistake — Not Disposing the Graphics Copy

```java
@Override public void paintBorder(Component c, Graphics g,
                                  int x, int y, int w, int h) {
    Graphics2D g2 = (Graphics2D) g.create();                       // WRONG
    g2.setColor(Color.RED);
    g2.drawRect(x, y, w - 1, h - 1);
    // missing g2.dispose()
}
```

**Expected Output.** Painting is correct, but native resources associated with the copy are not released, potentially causing performance degradation.

**Why this is wrong.** Every `Graphics2D` copy must be disposed in a `finally` block.

---

### Real-World Cases

- **Production Swing applications** use custom borders for focus rings, brand frames, and status panels.
- **Design systems** package custom borders as reusable classes with strict immutability.
- **Games and kiosks** use custom borders to render themed frames around dialogs and HUD panels.
- **Educational examples** demonstrate the correct insets, graphics disposal, and immutability from the start.

---

## References

Oracle. *Interface Border*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/Border.html

Oracle. *Class AbstractBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/AbstractBorder.html

Oracle. *Class BorderUIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/BorderUIResource.html

Oracle. *Interface UIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/UIResource.html

Oracle. *How to Use Borders*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/border.html

Oracle. *Class JComponent — setBorder, getBorder, paintBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *Class Graphics2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html

Oracle. *Class BasicStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/BasicStroke.html

Oracle. *Class Insets*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Insets.html

Oracle. *Class Rectangle*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Rectangle.html

Oracle. *Class BorderFactory*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/BorderFactory.html

Oracle. *Class CompoundBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/CompoundBorder.html

Oracle. *Class TitledBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/TitledBorder.html

Oracle. *Painting in AWT and Swing*. https://www.oracle.com/java/technologies/painting.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Cleverence. *Java Swing Border (Java SE 8): Complete Guide with Examples*. https://www.cleverence.com/articles/oracle-documentation/border-java-platform-se-8-4827/