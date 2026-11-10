# Custom Painting Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Custom painting** is the process of taking direct control of how a Swing component draws itself by overriding one or more of the painting methods inherited from `JComponent`. The programmer supplies drawing instructions using a `Graphics` (usually `Graphics2D`) object, and Swing calls those methods whenever the component needs to be displayed, repainted, or printed. Custom painting is the mechanism behind custom charts, diagrams, game canvases, status indicators, rounded panels, gradient backgrounds, custom borders, and specialised renderers.

**Technical Definition.** Custom painting in Swing is built on the **painting pipeline** defined by `JComponent`. When a component needs to be painted, Swing invokes its `paint(Graphics g)` method. The default implementation of `paint` calls three protected methods in a fixed order: `paintComponent(Graphics g)`, then `paintBorder(Graphics g)`, then `paintChildren(Graphics g)`. `paintComponent` paints the component itself (the UI delegate’s background and content), `paintBorder` paints the component’s border, and `paintChildren` recursively paints each child component. Application code typically overrides `paintComponent` (and occasionally `paintBorder`, `paintChildren`, or `paint` itself). `Graphics2D` is the modern drawing API, offering anti-aliasing, strokes, paints, transforms, clipping, composites, and text layout. Painting is performed on the Event Dispatch Thread (EDT) for on-screen rendering; the `Graphics` object is transient and must not be retained. All state that painting depends on must be stored as fields on the component (or its model), and changes to that state must trigger `repaint()`.

**Beginner-Friendly Explanation.** **Custom painting** is how you draw your own graphics on a Swing component. Normally, a `JButton` draws its own text and border, a `JLabel` draws its own text, and a `JPanel` draws its own background — all using the current look and feel. But when you want to draw a bar chart, a pie chart, a game sprite, a custom-shaped panel, a gradient background, or a rounded border, you need to tell Swing how to draw it. You do this by **subclassing** a Swing component (usually `JPanel` or `JComponent`) and **overriding `paintComponent(Graphics g)`**. Inside that method, you use `Graphics` (usually cast to `Graphics2D`) to draw shapes, lines, text, and images. Swing calls your method every time the component needs to be redrawn — for example, when it is first shown, when it is resized, or when you call `repaint()`. Think of it as a **canvas**: the component is the frame, `paintComponent` is your brush, and `repaint()` is how you ask the canvas to be refreshed.

---

### Key Characteristics

1. **Paint is called by Swing, not by you.** You must **not** invoke `paintComponent` or `paint` directly. You trigger repainting via `repaint()` (or `repaint(Rectangle)`), and Swing schedules the paint on the EDT.
2. **The pipeline is fixed.** `JComponent.paint` calls `paintComponent`, then `paintBorder`, then `paintChildren`. You override the stage appropriate to your needs.
3. **`super.paintComponent(g)` is almost always required.** The default implementation delegates to the UI delegate, which clears the background (if the component is opaque) and paints any look-and-feel content. Skipping the `super` call often leaves stale pixels or a blank background.
4. **State lives in fields, not in the paint method.** The paint method should **read** state and draw it; it should not **mutate** state. Storing values in fields and calling `repaint()` when they change is the correct pattern.
5. **`Graphics` is a stateful, transient object.** Stroke, colour, font, clip, and transform are mutable; changes affect subsequent drawing. Use `Graphics2D g2 = (Graphics2D) g.create()` when you need to modify state and be sure to dispose the copy.
6. **Use `Graphics2D` for modern drawing.** Cast `Graphics` to `Graphics2D` to access anti-aliasing, strokes, gradient paints, transforms, alpha compositing, and advanced text rendering.
7. **Painting is frequent and must be fast.** `paintComponent` is called many times per second in some scenarios. Expensive work (loading images, allocating fonts, computing layouts) must be cached outside the paint method.
8. **Double buffering is enabled by default.** Swing’s `RepaintManager` buffers drawing off-screen, so flicker is eliminated in the vast majority of cases.
9. **Renderers are painting, too.** `ListCellRenderer`, `TableCellRenderer`, and `TreeCellRenderer` are components whose `paintComponent` is called by the containing component. Custom painting in renderers follows the same rules but has additional constraints (no hierarchy, careful sizing).
10. **Painting runs on the EDT.** Do not block it; do not access UI from other threads without `SwingUtilities.invokeLater`.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, anonymous inner classes, and lambda expressions.
- **Swing component basics:** `JComponent`, `JPanel`, `JFrame`, layout managers, and the EDT.
- **The concept of a graphics context:** what a `Graphics` object is and why it is transient.
- **Basic geometry:** coordinates, `Rectangle`, `Point`, and the coordinate system (`x` increases to the right, `y` increases downward).
- **`java.awt.Graphics2D`:** familiarity with `setColor`, `drawLine`, `fillRect`, `drawString`, and `drawImage`.
- **A JDK (version 8 or later).** All painting classes are in `java.awt` and `javax.swing`.

---

### Related Programming Areas

- **`java.awt.Graphics` and `java.awt.Graphics2D`:** The drawing APIs used in `paintComponent`.
- **`javax.swing.JComponent`:** The superclass whose `paint`, `paintComponent`, `paintBorder`, and `paintChildren` methods form the pipeline.
- **`javax.swing.RepaintManager`:** The manager that schedules repaints and performs double buffering.
- **`java.awt.image.BufferedImage`:** Off-screen image used for caching complex drawings.
- **`java.awt.Paint` and its implementations (`Color`, `GradientPaint`, `LinearGradientPaint`, `RadialGradientPaint`, `TexturePaint`):** The colour source for fills.
- **`java.awt.Stroke` and `java.awt.BasicStroke`:** Control the shape and width of drawn lines.
- **`java.awt.Composite` and `java.awt.AlphaComposite`:** Control transparency and blending.
- **`javax.swing.border.Border`:** The border abstraction painted by `paintBorder`.
- **`java.awt.RenderingHints`:** Control anti-aliasing, interpolation, and text rendering quality.
- **`javax.swing.Timer`:** The standard mechanism for triggering periodic `repaint()` calls for animation.
- **`ListCellRenderer`, `TableCellRenderer`, `TreeCellRenderer`:** Renderers whose `paintComponent` can be customised.

---

## Core Concepts and Features

---

## 1. Custom Painting — The Concept

### Definitions

**Core Definition.** Custom painting is the practice of overriding a component's painting methods to render graphics, text, and images directly onto the component's surface.

**Technical Definition.** Custom painting in Swing is implemented by subclassing a `JComponent` (usually `JPanel` or `JComponent` itself) and overriding one or more of `paintComponent(Graphics)`, `paintBorder(Graphics)`, `paintChildren(Graphics)`, or `paint(Graphics)`. The overridden method uses a `Graphics2D` object to draw shapes, text, and images. The component stores the state it paints in fields, and calls `repaint()` when that state changes.

**Beginner-Friendly Explanation.** Custom painting is “drawing your own component.” Instead of relying on the look and feel to draw the component, you provide the drawing code.

---

### Purposes

- To render graphics that no standard Swing component draws (charts, graphs, diagrams, game scenes).
- To create custom backgrounds, borders, and decorations for panels.
- To display images, icons, or sprites under programmatic control.
- To implement custom renderers for lists, tables, and trees.
- To visualise data or state in ways that text and standard components cannot.
- To draw overlays (glass panes) and special effects.
- To create highly branded or themed interfaces.

---

### Syntax Rules and Structure

#### Complete General Syntax (Subclassing for Custom Painting)

```java
public class MyCanvas extends JPanel {
    public MyCanvas() {
        setOpaque(true);                       // if you paint the background yourself
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300)); // if the component has no children
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);               // clear the background (opaque)
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            // drawing code here
        } finally {
            g2.dispose();
        }
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `extends JPanel` | The subclass that will host the custom painting. |
| `setOpaque(true)` | Tells Swing to fill the component’s background before painting. |
| `setPreferredSize(...)` | Gives the component a preferred size for layout managers. |
| `paintComponent(Graphics g)` | The overridden painting hook. |
| `super.paintComponent(g)` | Delegates background clearing and UI painting to the superclass. |
| `Graphics2D g2 = (Graphics2D) g.create()` | Creates a private copy of the graphics state. |
| `g2.dispose()` | Releases the copy’s resources. |

#### Syntax Rules

- Override `paintComponent`, not `paint`, for ordinary custom drawing.
- Call `super.paintComponent(g)` first if the component is opaque or has a UI delegate that paints.
- Use `Graphics2D` for modern drawing.
- Restore or dispose modified graphics state.
- Trigger repaints via `repaint()`, never by calling `paintComponent` directly.

#### Constraints and Limitations

- Painting must be fast; no blocking, no I/O, no expensive allocations.
- Painting must not mutate application state.
- Custom components with no children need a preferred size or they may have zero size in layout managers.
- Painting runs on the EDT for display; other painting contexts (e.g., printing) may run on other threads.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Custom Painting — A Red Rectangle

```java
import javax.swing.*;
import java.awt.*;

public class RedRectanglePanel extends JPanel {

    public RedRectanglePanel() {
        setOpaque(true);                                       // 1. Opaque
        setBackground(Color.WHITE);                            // 2. Background
        setPreferredSize(new Dimension(400, 300));             // 3. Size
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                               // 4. Clear background
        Graphics2D g2 = (Graphics2D) g.create();               // 5. Private copy
        try {
            g2.setColor(Color.RED);                            // 6. Red
            g2.fillRect(50, 50, 200, 150);                     // 7. Rectangle
        } finally {
            g2.dispose();                                      // 8. Dispose
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Red Rectangle");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new RedRectanglePanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 400×300 window containing a white panel with a red 200×150 rectangle positioned at (50, 50).

**Why the code produces this result.** The panel is opaque, so `super.paintComponent(g)` fills its background with white. The `Graphics2D` copy is configured with red and used to fill a rectangle at the given coordinates. The copy is disposed after drawing.

---

### Real-World Cases

- **Charts and dashboards:** A custom panel draws bar, line, or pie charts.
- **Game canvases:** A custom panel draws sprites and game scenes at high frame rates.
- **Diagram editors:** A custom panel draws nodes, connections, and selection handles.
- **Custom panels:** Rounded or gradient panels that standard look-and-feel does not provide.
- **Custom renderers:** Components that draw list cells, table cells, or tree nodes.

---

## 2. paintComponent() — The Component Painting Hook

### Definitions

**Core Definition.** `paintComponent(Graphics g)` is the protected method that paints the component itself — its background and its own visual content — before its border and children are painted.

**Technical Definition.** `protected void paintComponent(Graphics g)` is declared in `JComponent`. The default implementation delegates to the component’s `ui` (UI delegate) via `ui.update(g, this)`, which clears the background if the component is opaque and then paints the delegate’s content. Overriding `paintComponent` replaces or augments this behaviour. The method is called from `paint` and, in Swing’s double-buffered pipeline, is invoked on the EDT during display rendering.

**Beginner-Friendly Explanation.** `paintComponent` is your **primary drawing hook**. It paints the component’s own visual content — background, shapes, text, and images — before any border or child component is drawn.

---

### Purposes

- To draw the component’s own content (shapes, text, images).
- To customise the background (gradients, textures, transparency).
- To augment the look-and-feel’s default painting (e.g., draw over a button’s default appearance).
- To display dynamic content driven by application state.
- To serve as the painting hook for custom renderers.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);                    // 1. Clear background (if opaque)
    Graphics2D g2 = (Graphics2D) g.create();    // 2. Private graphics copy
    try {
        // 3. Configure graphics
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                            RenderingHints.VALUE_ANTIALIAS_ON);
        g2.setColor(Color.BLUE);
        // 4. Draw
        g2.fillOval(10, 10, 100, 100);
    } finally {
        g2.dispose();                           // 5. Dispose
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `@Override` | Confirms the method overrides the superclass declaration. |
| `super.paintComponent(g)` | Delegates to the UI delegate to clear background and paint defaults. |
| `Graphics2D g2 = (Graphics2D) g.create()` | Creates a private graphics context that can be safely modified. |
| `try/finally` | Ensures `dispose()` is called. |
| `g2.dispose()` | Releases the copy. |

#### Syntax Rules

- Call `super.paintComponent(g)` **first** unless you intend to paint over (or replace) the UI delegate’s content entirely.
- Modify graphics state freely on the private copy; do not rely on restoring it on the original.
- Never store a reference to the `Graphics` object; it is transient.
- Do not perform long-running work; painting is called frequently.

#### Constraints and Limitations

- Calling `super.paintComponent` after your drawing will paint the UI delegate’s content **over** yours.
- Skipping `super.paintComponent` on an opaque component may leave stale pixels from previous paints.
- Overriding `paintComponent` on components with complex look-and-feel rendering (e.g., `JTable`, `JTree`) may break their rendering.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Anti-Aliased Circle with Text

```java
import javax.swing.*;
import java.awt.*;

public class CirclePanel extends JPanel {
    public CirclePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                                 // 1. Clear
        Graphics2D g2 = (Graphics2D) g.create();                 // 2. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON); // 3. AA
            g2.setColor(new Color(0, 120, 215));                 // 4. Colour
            g2.fillOval(100, 80, 200, 120);                      // 5. Circle
            g2.setColor(Color.WHITE);
            g2.setFont(new Font("SansSerif", Font.BOLD, 24));
            g2.drawString("Hello", 165, 148);                    // 6. Text
        } finally {
            g2.dispose();                                        // 7. Dispose
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Circle Panel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new CirclePanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white panel with a smooth blue ellipse and the white text “Hello” centred inside it.

**Why the code produces this result.** Anti-aliasing smooths the ellipse’s edges. The text is drawn with a custom font. The graphics copy is disposed after drawing, releasing resources without affecting other painting.

---

#### Example 2: Dynamic Painting Driven by State

```java
import javax.swing.*;
import java.awt.*;

public class ProgressPanel extends JPanel {
    private int progress = 0;

    public ProgressPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 80));
    }

    public void setProgress(int p) {
        progress = Math.max(0, Math.min(100, p));
        repaint();                                              // 1. Trigger
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                                // 2. Clear
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.LIGHT_GRAY);
            g2.fillRect(10, 30, 380, 20);                       // 3. Track
            g2.setColor(new Color(0, 160, 0));
            g2.fillRect(10, 30, 380 * progress / 100, 20);      // 4. Bar
            g2.setColor(Color.BLACK);
            g2.drawString(progress + "%", 180, 25);             // 5. Label
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Progress Panel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            ProgressPanel panel = new ProgressPanel();
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            new Timer(100, e -> {
                int p = (panel.progress + 5) % 105;
                panel.setProgress(p);
            }).start();
        });
    }
}
```

**Expected Output.** A panel showing a grey track with a growing green bar and a percentage label, advancing by 5% every 100 ms and looping.

**Why the code produces this result.** The state (`progress`) is stored in a field and updated via a `Swing Timer` on the EDT. `setProgress` calls `repaint()`, which schedules a new paint. `paintComponent` reads the current state and draws accordingly.

---

### Real-World Cases

- **Dashboards and data visualisation:** Charts, gauges, heatmaps.
- **Custom buttons and controls:** Rounded buttons, toggle switches.
- **Game canvases:** Game boards, sprites, and animations.
- **Custom containers:** Panels with gradient or texture backgrounds.
- **Custom renderers:** Components that paint list or table cells.

---

## 3. paint() — The Full Painting Pipeline

### Definitions

**Core Definition.** `paint(Graphics g)` is the public method that Swing calls when a component needs to be painted. Its default implementation orchestrates `paintComponent`, `paintBorder`, and `paintChildren` in that order.

**Technical Definition.** `public void paint(Graphics g)` is declared in `JComponent`. The default implementation performs:

```java
public void paint(Graphics g) {
    paintComponent(g);
    paintBorder(g);
    paintChildren(g);
}
```

Overriding `paint` is rarely necessary; it is appropriate when the component must **coordinate** all three stages, such as a glass pane that must draw on top of everything, or a container that wants to apply a transform to its children. The `RepaintManager` calls `paint` during repaint cycles.

**Beginner-Friendly Explanation.** `paint` is the **director** of the painting show. It calls the component painter, then the border painter, then the child painter. You rarely override the director — you override the individual performers. Override `paint` only when you need to change the order, skip a stage, or draw over everything.

---

### Purposes

- To customise the entire painting sequence when `paintComponent` alone is insufficient.
- To draw overlays above children (glass-pane style).
- To apply a transform, clip, or composite to children before painting them.
- To bypass the standard pipeline in specialised components.
- To control painting order explicitly.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void paint(Graphics g) {
    super.paint(g);                    // 1. Paint component, border, children
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        // 2. Draw over everything
    } finally {
        g2.dispose();
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `super.paint(g)` | Runs the standard three-stage pipeline. |
| Drawing after `super.paint` | Draws **over** the component and all its children. |
| Drawing before `super.paint` | Is typically overwritten by the superclass pipeline (avoid). |
| Drawing between `paintComponent` and `paintChildren` | Requires overriding `paint` and calling the stages manually. |

#### Syntax Rules

- Calling `super.paint(g)` is usually required to get the standard stages.
- Drawing after `super.paint` produces an overlay above children.
- Drawing before `super.paint` is almost always a mistake.
- Overriding `paint` on components with look-and-feel rendering may interfere with their appearance.

#### Constraints and Limitations

- Overriding `paint` is rarely the right choice; prefer `paintComponent`.
- Glass-pane overlays are easier to implement as custom `JComponent` subclasses with `paintComponent` overrides.
- Custom containers that transform children should override `paintChildren` instead.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Overlay Painting Above Children

```java
import javax.swing.*;
import java.awt.*;

public class OverlayPanel extends JPanel {
    public OverlayPanel() {
        setLayout(new FlowLayout());
        add(new JButton("Click Me"));
        add(new JLabel("Label"));
        setPreferredSize(new Dimension(400, 200));
        setOpaque(true);
        setBackground(Color.WHITE);
    }

    @Override
    public void paint(Graphics g) {
        super.paint(g);                                       // 1. Standard stages
        Graphics2D g2 = (Graphics2D) g.create();              // 2. Overlay copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(new Color(255, 0, 0, 80));            // 3. Semi-transparent
            g2.fillRect(0, 0, getWidth(), getHeight());       // 4. Red overlay
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Overlay Panel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new OverlayPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel containing a button and a label, with a translucent red tint drawn over the entire panel — including the child components.

**Why the code produces this result.** `super.paint(g)` paints the panel’s background, border, and children normally. The subsequent drawing is layered on top, producing a semi-transparent red overlay that tints everything beneath it.

---

#### Example 2: Manual Pipeline Control

```java
@Override
public void paint(Graphics g) {
    paintComponent(g);                                    // 1. Component
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        g2.setComposite(AlphaComposite.getInstance(
            AlphaComposite.SRC_OVER, 0.5f));              // 2. Fade children
        Graphics gChild = g2;
        paintBorder(gChild);
        paintChildren(gChild);
    } finally {
        g2.dispose();
    }
}
```

**Expected Output.** The component itself paints normally, but its border and children are painted with 50% alpha.

**Why the code produces this result.** Overriding `paint` allows precise control of each stage with a shared graphics context.

---

### Real-World Cases

- **Glass panes** for drag-and-drop feedback, loading overlays, rubber-band selection.
- **Fade-in animations** that gradually reveal children.
- **Printing** where custom painting must respect a different graphics context.
- **Specialised containers** that apply transforms to children.

---

## 4. paintBorder() — The Border Painting Hook

### Definitions

**Core Definition.** `paintBorder(Graphics g)` is the protected method that paints the component’s border. Its default implementation delegates to the border set via `setBorder`.

**Technical Definition.** `protected void paintBorder(Graphics g)` is declared in `JComponent`. The default implementation calls `getBorder()`; if non-`null`, it invokes `border.paintBorder(this, g, 0, 0, getWidth(), getHeight())`. Overriding `paintBorder` is rare — the recommended way to customise borders is to implement the `Border` interface and set it via `setBorder`. Overriding `paintBorder` is useful when the border must depend on mutable component state that is not convenient to express in a `Border` implementation.

**Beginner-Friendly Explanation.** `paintBorder` paints the border around the component. **Normally you set a border with `setBorder(...)` and do not override this method.** Override it only if you need custom border behaviour that depends on component state.

---

### Purposes

- To paint a border around a component.
- To customise the border’s appearance beyond what `Border` implementations provide.
- To make the border depend on component state without creating a stateful `Border` object.
- To coordinate border painting with custom backgrounds.

---

### Syntax Rules and Structure

#### Complete General Syntax (Preferred: Set a Border)

```java
// Preferred:
setBorder(BorderFactory.createLineBorder(Color.GRAY, 2));

// Override only when necessary:
@Override
protected void paintBorder(Graphics g) {
    super.paintBorder(g);                          // 1. Paint default border
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        g2.setColor(Color.RED);
        g2.drawRect(0, 0, getWidth() - 1, getHeight() - 1); // 2. Extra border
    } finally {
        g2.dispose();
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `setBorder(Border)` | The recommended way to configure a border. |
| `super.paintBorder(g)` | Paints the configured border, if any. |
| Drawing after `super.paintBorder` | Adds extra border decoration. |

#### Syntax Rules

- Prefer `setBorder` with a `Border` implementation.
- If overriding `paintBorder`, call `super.paintBorder(g)` to preserve the configured border.
- Border painting must not exceed the component’s bounds (it would be clipped).

#### Constraints and Limitations

- Overriding `paintBorder` on components that rely on the look and feel’s border painting may conflict with the UI delegate.
- Custom border painting should not paint outside the component’s bounds.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Border Painted via `setBorder`

```java
import javax.swing.*;
import javax.swing.border.Border;
import java.awt.*;

public class DashedBorderPanel extends JPanel {

    public DashedBorderPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(300, 200));
        setBorder(new Border() {                             // 1. Border impl
            @Override public void paintBorder(Component c, Graphics g,
                    int x, int y, int w, int h) {
                Graphics2D g2 = (Graphics2D) g.create();
                try {
                    g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                        RenderingHints.VALUE_ANTIALIAS_ON);
                    g2.setColor(Color.BLUE);
                    g2.setStroke(new BasicStroke(2f, BasicStroke.CAP_BUTT,
                        BasicStroke.JOIN_MITER, 10f, new float[]{10f, 5f}, 0f));
                    g2.drawRect(x + 1, y + 1, w - 2, h - 2); // 2. Dashed
                } finally {
                    g2.dispose();
                }
            }
            @Override public Insets getBorderInsets(Component c) {
                return new Insets(4, 4, 4, 4);
            }
            @Override public boolean isBorderOpaque() { return false; }
        });
    }
}
```

**Expected Output.** A white panel with a blue dashed border.

---

## 5. paintChildren() — The Child Painting Hook

### Definitions

**Core Definition.** `paintChildren(Graphics g)` is the protected method that paints the component’s child components, in their z-order.

**Technical Definition.** `protected void paintChildren(Graphics g)` is declared in `JComponent`. The default implementation iterates through the component’s children (respecting visibility and z-order) and calls each child’s `paint` method. Overriding `paintChildren` is used to apply a transform, clip, composite, or other graphics state to all children at once, or to control z-order or filtering of children.

**Beginner-Friendly Explanation.** `paintChildren` paints all child components. Override it when you want to apply the same effect to all children — for example, render them at 50% opacity or under a transform.

---

### Purposes

- To paint child components with a modified graphics state (e.g., transform, alpha, clip).
- To control which children are painted.
- To customise z-order or painting order.
- To add effects (fade, scale, rotate) uniformly to a set of children.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
protected void paintChildren(Graphics g) {
    Graphics2D g2 = (Graphics2D) g.create();
    try {
        g2.setComposite(AlphaComposite.getInstance(
            AlphaComposite.SRC_OVER, 0.5f));         // 1. Effect
        super.paintChildren(g2);                      // 2. Paint children
    } finally {
        g2.dispose();
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `g2 = (Graphics2D) g.create()` | Private copy for modifying state. |
| Effect configuration | Transform, composite, clip, etc. |
| `super.paintChildren(g2)` | Paints all children using the modified graphics. |

#### Syntax Rules

- Call `super.paintChildren(g2)` to paint the children normally.
- Always dispose the copy.
- Transform applies to child bounds and painting; children’s layout is unaffected.

#### Constraints and Limitations

- Overriding `paintChildren` on look-and-feel-managed containers may interfere with optimisations.
- Children should be painted in their z-order; do not reorder them unless necessary.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Fading Children

```java
import javax.swing.*;
import java.awt.*;

public class FadingPanel extends JPanel {
    public FadingPanel() {
        setLayout(new FlowLayout());
        add(new JButton("One"));
        add(new JButton("Two"));
        add(new JLabel("Label"));
        setPreferredSize(new Dimension(400, 200));
        setOpaque(true);
        setBackground(Color.WHITE);
    }

    @Override
    protected void paintChildren(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();                 // 1. Copy
        try {
            g2.setComposite(AlphaComposite.getInstance(
                AlphaComposite.SRC_OVER, 0.4f));                 // 2. Fade
            super.paintChildren(g2);                              // 3. Children
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Fading Panel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new FadingPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The panel’s children appear at 40% opacity, giving a ghosted effect.

**Why the code produces this result.** The composite is applied to the graphics context used to paint children; children paint at 40% alpha.

---

### Real-World Cases

- **Fade transitions** when a panel is loading.
- **Disabled containers** that render their children at low opacity.
- **Custom containers** that draw children under a transform.
- **Screenshot or snapshot** rendering.

---

## 6. The Repaint Cycle — repaint(), revalidate(), and paintImmediately()

### Definitions

**Core Definition.** The repaint cycle is the mechanism by which state changes in a component lead to it being repainted. `repaint()` marks the component (or a region) as dirty and schedules painting; the `RepaintManager` batches repaint requests and paints on the EDT. `revalidate()` is separate — it invalidates layout, not painting.

**Technical Definition.** `JComponent.repaint()` and its overloads notify the `RepaintManager` that the component needs to be painted. The manager coalesces overlapping requests and later invokes the component’s `paint` method on the EDT. `repaint(int x, int y, int w, int h)` and `repaint(Rectangle)` limit the repainted area. `revalidate()` invalidates the layout and triggers a layout cycle; it does not itself paint. `paintImmediately` paints synchronously and is rarely used.

**Beginner-Friendly Explanation.** `repaint()` says “I changed; please redraw me.” `revalidate()` says “My layout changed; please re-lay-out me.” They serve different purposes and are often called together after structural changes.

---

### Purposes

- To trigger repainting when component state changes.
- To limit repainting to a dirty region for performance.
- To trigger relayout after adding, removing, or resizing children.
- To understand when and how painting happens.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
repaint();                            // Full repaint
repaint(x, y, w, h);                  // Dirty region repaint
repaint(Rectangle r);                 // Dirty region repaint
revalidate();                         // Layout invalidation
revalidate(); repaint();              // Both after structural changes
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `repaint()` | Schedules a full repaint. |
| `repaint(int, int, int, int)` | Schedules a dirty-region repaint. |
| `revalidate()` | Marks layout as invalid; triggers layout. |
| `paintImmediately(...)` | Paints synchronously (rarely used). |

#### Syntax Rules

- `repaint()` and `revalidate()` are safe to call from any thread (they post to the EDT internally).
- Call `repaint()` after state changes that affect the visuals.
- Call `revalidate()` (and usually `repaint()`) after adding/removing/resizing children.

#### Constraints and Limitations

- `repaint()` is asynchronous; it does not paint immediately.
- Calling `repaint()` from `paintComponent` risks infinite loops.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dirty-Region Repaint

```java
public class DirtyRegionPanel extends JPanel {
    private Point last = new Point(-1, -1);

    public DirtyRegionPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 400));
        addMouseMotionListener(new java.awt.event.MouseMotionAdapter() {
            @Override
            public void mouseDragged(java.awt.event.MouseEvent e) {
                if (last.x >= 0) {
                    int x = Math.min(last.x, e.getX());
                    int y = Math.min(last.y, e.getY());
                    int w = Math.abs(last.x - e.getX()) + 10;
                    int h = Math.abs(last.y - e.getY()) + 10;
                    last = e.getPoint();
                    repaint(x - 5, y - 5, w, h);                 // 1. Region
                } else {
                    last = e.getPoint();
                    repaint();
                }
            }
        });
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        if (last.x >= 0) {
            Graphics2D g2 = (Graphics2D) g.create();
            try {
                g2.setColor(Color.BLUE);
                g2.fillOval(last.x - 5, last.y - 5, 10, 10);
            } finally {
                g2.dispose();
            }
        }
    }
}
```

**Expected Output.** A window where dragging the mouse leaves a trail of blue dots, with only the affected region repainted per event.

---

## 7. Painting with Graphics2D — Rendering Hints, Strokes, and Transforms

### Definitions

**Core Definition.** `Graphics2D` extends `Graphics` and provides modern 2D drawing: anti-aliasing, strokes, paint objects, transforms, compositing, clipping, and advanced text.

**Technical Definition.** `public abstract class Graphics2D extends Graphics`. It adds `setRenderingHint`, `setPaint`, `setStroke`, `setComposite`, `setClip`, `transform`, `translate`, `rotate`, `scale`, `draw`, `fill` (for `Shape`), and text methods with `FontRenderContext` support. All Swing painting uses a `Graphics2D`; casting `Graphics` is safe.

**Beginner-Friendly Explanation.** `Graphics2D` is the “pro” drawing tool. It adds anti-aliasing for smooth edges, strokes for line styles, paints for gradients, and transforms for rotating or scaling.

---

### Purposes

- To draw smooth, anti-aliased shapes.
- To draw with gradients, textures, and custom paints.
- To draw with strokes of various widths and dash patterns.
- To apply transforms to a group of shapes.
- To use alpha compositing for transparency.
- To draw complex shapes via `Shape` and `Path2D`.
- To render high-quality text.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                        RenderingHints.VALUE_ANTIALIAS_ON);
    g2.setPaint(new GradientPaint(0, 0, Color.BLUE,
                                  getWidth(), getHeight(), Color.CYAN));
    g2.setStroke(new BasicStroke(3f, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
    g2.translate(100, 100);
    g2.rotate(Math.toRadians(30));
    g2.fill(new RoundRectangle2D.Double(0, 0, 200, 100, 20, 20));
} finally {
    g2.dispose();
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setRenderingHint` | Configures quality (anti-aliasing, interpolation). |
| `setPaint` | Sets the fill paint (Color, GradientPaint, TexturePaint). |
| `setStroke` | Configures line width, caps, joins, dashes. |
| `setComposite` | Configures alpha blending. |
| `translate`, `rotate`, `scale` | Modify the coordinate transform. |
| `fill(Shape)` | Fills the given shape. |
| `draw(Shape)` | Outlines the given shape. |

#### Syntax Rules

- Always work on a `Graphics2D` copy when modifying state.
- Restore or dispose the copy.
- Use `Shape` implementations for flexible geometry.
- Use `Path2D` for arbitrary paths.

#### Constraints and Limitations

- Anti-aliasing and alpha compositing have performance costs; use sparingly on large areas or high-frequency paints.
- `GradientPaint` requires non-identical start and end points.
- Text rendering quality is controlled by `KEY_TEXT_ANTIALIASING`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Gradient-Filled Rounded Rectangle

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
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);           // 1. AA
            GradientPaint gp = new GradientPaint(
                50, 50, Color.BLUE, 300, 250, Color.CYAN);                    // 2. Paint
            g2.setPaint(gp);
            g2.fill(new RoundRectangle2D.Double(50, 50, 250, 200, 30, 30));   // 3. Shape
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A rounded rectangle filled with a blue-to-cyan gradient.

---

## 8. Custom Painting in Renderers — JList, JTable, JTree

### Definitions

**Core Definition.** Renderers are components whose `paintComponent` is invoked by a container (a `JList`, `JTable`, or `JTree`) to draw each cell. Custom painting in a renderer follows the same rules as any component, but with additional constraints because renderers are not part of the normal container hierarchy.

**Technical Definition.** A `ListCellRenderer` returns a `Component` (usually a `JLabel` or `JPanel`) whose size the list sets before calling `paint`. The renderer should be **stateless** or reset its state each time. `isOpaque` should be set to `true` if the renderer paints its own background and to `false` if the container’s background should show through. `TableCellRenderer` and `TreeCellRenderer` follow the same pattern.

**Beginner-Friendly Explanation.** Renderers are “stamps” used to draw cells in lists and tables. They use `paintComponent`, but they must be careful: they are called for every visible cell, repeatedly, and their state must be reset each time.

---

### Purposes

- To render custom cells in lists, tables, and trees.
- To draw icons, images, or custom text layouts in cells.
- To convey state (selection, focus, warnings) through custom drawing.

---

### Syntax Rules and Structure

#### Complete General Syntax (Custom List Cell Renderer)

```java
public class MyRenderer extends JPanel implements ListCellRenderer<MyItem> {
    private final JLabel label = new JLabel();

    public MyRenderer() {
        setLayout(new BorderLayout());
        add(label, BorderLayout.CENTER);
    }

    @Override
    public Component getListCellRendererComponent(
            JList<? extends MyItem> list, MyItem value,
            int index, boolean isSelected, boolean cellHasFocus) {
        label.setText(value.toString());
        setBackground(isSelected ? list.getSelectionBackground()
                                 : list.getBackground());              // 1. Reset
        setOpaque(true);
        return this;
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.BLUE);
            g2.fillOval(4, getHeight() / 2 - 5, 10, 10);               // 2. Custom
        } finally {
            g2.dispose();
        }
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getListCellRendererComponent` | Configures the renderer for a specific cell. |
| `setOpaque(true/false)` | Controls background painting. |
| `setBackground(...)` | Resets background based on selection state. |
| `paintComponent` | Optional custom drawing. |

#### Syntax Rules

- Reset state in `getListCellRendererComponent` each time.
- Do not store `Graphics` or per-cell state across calls.
- Set the renderer’s size (the container does this for lists; custom renderers may need to respect the size).

#### Constraints and Limitations

- Renderers are called frequently; painting must be very fast.
- Renderer components are not in the container hierarchy; do not rely on `getParent()`.
- Avoid expensive calculations in `paintComponent` of renderers.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom List Cell with Indicator

```java
import javax.swing.*;
import java.awt.*;

public class IndicatorRenderer extends JPanel implements ListCellRenderer<String> {
    public IndicatorRenderer() {
        setLayout(new BorderLayout(5, 0));
        setBorder(BorderFactory.createEmptyBorder(4, 4, 4, 4));
    }

    @Override
    public Component getListCellRendererComponent(
            JList<? extends String> list, String value,
            int index, boolean isSelected, boolean cellHasFocus) {
        removeAll();
        JLabel indicator = new JLabel("●");
        indicator.setForeground(index % 2 == 0 ? Color.GREEN : Color.RED); // 1. State
        JLabel label = new JLabel(value);
        add(indicator, BorderLayout.WEST);
        add(label, BorderLayout.CENTER);
        setOpaque(true);
        setBackground(isSelected ? list.getSelectionBackground()
                                 : list.getBackground());                  // 2. Reset
        return this;
    }
}
```

**Expected Output.** A list where each row shows a green or red indicator dot followed by the item text.

---

## 9. Custom Painting Best Practices

### Definitions

**Core Definition.** Custom painting best practices are the conventions and patterns that ensure painted components are correct, fast, thread-safe, and maintainable.

**Technical Definition.** Best practices include: overriding `paintComponent` (not `paint`) for ordinary custom drawing; calling `super.paintComponent(g)` to honour the UI delegate and opacity; working on a `Graphics2D` copy and disposing it; storing state in fields and calling `repaint()` on change; setting a preferred size for leaf components; caching expensive resources; keeping paint methods free of side effects; using dirty-region repaints where beneficial; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices are the “rules of thumb” that prevent flicker, slow painting, and lost drawing. They include things like “call `super.paintComponent`,” “don’t create objects in `paintComponent`,” “don’t do work in `paintComponent`,” “set a preferred size,” and “use `repaint()` not direct painting.”

---

### Purposes

- To ensure custom components paint correctly and completely.
- To keep painting fast enough for responsive user interfaces.
- To avoid common bugs (flicker, stale pixels, lost background).
- To keep paint code testable, maintainable, and free of side effects.
- To support high-quality, high-DPI rendering.
- To integrate with renderers and other painting contexts.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class BestPracticePanel extends JPanel {
    private int value = 0;

    public BestPracticePanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));
    }

    public void setValue(int v) {
        if (value != v) {
            value = v;
            repaint();                                        // 1. Repaint
        }
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                              // 2. Super
        Graphics2D g2 = (Graphics2D) g.create();              // 3. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            // read-only drawing using `value`
        } finally {
            g2.dispose();                                     // 4. Dispose
        }
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Override `paintComponent` | Not `paint` (unless you need full control) |
| Call `super.paintComponent` | Honour the UI delegate and background |
| Use `Graphics2D` copy | `(Graphics2D) g.create()` in try/finally |
| Store state in fields | Not in the `Graphics` or the paint method |
| Trigger repaint on change | `repaint()` after state updates |
| Set preferred size | `setPreferredSize` or override `getPreferredSize` |
| Cache resources | Fonts, strokes, paints, images as fields |
| Limit repaint region | `repaint(x, y, w, h)` for small changes |
| Avoid side effects | Paint methods must not change state |
| EDT | Mutate state and repaint on the EDT |

#### Syntax Rules

- **Always call `super.paintComponent(g)`** at the start unless you intentionally replace UI painting.
- **Always set `setOpaque(true)`** if your component paints its background and should not show its parent’s background.
- **Always use a `Graphics2D` copy** when modifying state; dispose it.
- **Never store the `Graphics` object** or reuse it after the method returns.
- **Never perform expensive computations, I/O, or image loading in `paintComponent`**.
- **Always call `repaint()`** after state changes that affect painting.
- **Always set a preferred size** for components with no children.

#### Constraints and Limitations

- Anti-aliasing and alpha blending have costs; disable when unnecessary.
- Painting may occur on threads other than the EDT for printing or off-screen rendering; keep paint methods free of assumptions about threads.
- Renderers are stamped repeatedly; they must reset their state.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Chart Component

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.RoundRectangle2D;

public class BestPracticeChart extends JPanel {
    private int[] values = {50, 80, 30, 90, 60};
    private static final Color BAR_COLOR = new Color(0, 120, 215);
    private static final Font TITLE_FONT = new Font("SansSerif", Font.BOLD, 14);

    public BestPracticeChart() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 250));
    }

    public void setValues(int[] v) {
        values = v;
        repaint();                                                  // 1. Repaint
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                                    // 2. Super
        Graphics2D g2 = (Graphics2D) g.create();                    // 3. Copy
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            int w = getWidth(), h = getHeight();
            int pad = 40;
            int barWidth = (w - 2 * pad) / Math.max(1, values.length);

            g2.setFont(TITLE_FONT);
            g2.setColor(Color.DARK_GRAY);
            g2.drawString("Bar Chart", pad, 25);

            int baseline = h - pad;
            int maxHeight = h - 2 * pad;
            for (int i = 0; i < values.length; i++) {
                int bh = (int) (maxHeight * values[i] / 100.0);
                int x = pad + i * barWidth + 4;
                int y = baseline - bh;
                g2.setColor(BAR_COLOR);
                g2.fill(new RoundRectangle2D.Double(x, y, barWidth - 8, bh, 6, 6));
                g2.setColor(Color.GRAY);
                g2.drawLine(pad, baseline, w - pad, baseline);
            }
        } finally {
            g2.dispose();                                           // 4. Dispose
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Chart");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticeChart());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A custom chart panel drawing five rounded bars of varying heights, a baseline, a title, and anti-aliased edges.

**Why the code produces this result.** The component stores its values and the constants (font, colour) as fields to avoid allocation in the paint method. `paintComponent` reads the state, draws the chart, and calls `super.paintComponent` to clear the background. The graphics copy is disposed.

---

#### Example 2: Common Mistake — Skipping `super.paintComponent`

```java
public class BadPaintPanel extends JPanel {
    public BadPaintPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
    }

    @Override
    protected void paintComponent(Graphics g) {
        // WRONG: no super call
        g.setColor(Color.RED);
        g.fillOval(10, 10, 100, 100);
    }
}
```

**Expected Output.** The component may show stale pixels, a black background, or inconsistent clearing, because the UI delegate was never called to clear the background.

**Why this is wrong.** `super.paintComponent(g)` is what clears the background for an opaque component. Without it, previous drawings may persist behind your new drawing, causing visual artifacts. The correct approach is to call `super.paintComponent(g)` first.

---

#### Example 3: Common Mistake — Performing Work in `paintComponent`

```java
public class BadWorkPanel extends JPanel {
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        // WRONG: heavy work in the paint method
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        java.util.List<String> data = loadFromDatabase();             // I/O
        g.drawString(data.get(0), 10, 10);
    }
}
```

**Expected Output.** The UI freezes or stutters, because the paint method blocks the EDT on every repaint.

**Why this is wrong.** Painting happens frequently; blocking it hurts responsiveness. Data should be loaded outside the paint method (e.g., in a `SwingWorker`), stored in a field, and read in `paintComponent`.

---

#### Example 4: Common Mistake — Allocating in `paintComponent`

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    for (int i = 0; i < 1000; i++) {
        Font f = new Font("SansSerif", Font.PLAIN, 12);               // WRONG
        g.setFont(f);
        g.drawString("Line " + i, 10, i * 15);
    }
}
```

**Expected Output.** Painting becomes slow because a new `Font` is allocated one thousand times per paint.

**Why this is wrong.** Allocating objects in the paint method is a significant performance problem. The correct approach is to create the font once and store it in a field.

---

### Real-World Cases

- **Production Swing applications** follow these practices to keep painted components fast, correct, and maintainable.
- **Data visualisation tools** cache fonts, paints, and images as fields.
- **Game engines** repaint dirty regions rather than the whole component.
- **Custom renderers** reset their state on every call and avoid allocations.
- **Educational examples** use the best-practice skeleton to demonstrate correct painting from the start.

---

## References

Oracle. *Performing Custom Painting*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/painting/index.html

Oracle. *Solving Common Painting Problems*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/painting/problems.html

Oracle. *Class JComponent — paint, paintComponent, paintBorder, paintChildren*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *Class Graphics*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics.html

Oracle. *Class Graphics2D*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html

Oracle. *Class RenderingHints*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/RenderingHints.html

Oracle. *Class BasicStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/BasicStroke.html

Oracle. *Class GradientPaint*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/GradientPaint.html

Oracle. *Class AlphaComposite*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/AlphaComposite.html

Oracle. *Class BufferedImage*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/image/BufferedImage.html

Oracle. *Interface Border*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/Border.html

Oracle. *Class RepaintManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/RepaintManager.html

Oracle. *Class Timer (javax.swing.Timer)*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/Timer.html

Oracle. *Interface ListCellRenderer*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ListCellRenderer.html

Oracle. *Interface TableCellRenderer*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/TableCellRenderer.html

Oracle. *Interface TreeCellRenderer*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/TreeCellRenderer.html

Oracle. *Painting in AWT and Swing*. The Java Tutorials. https://www.oracle.com/java/technologies/painting.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading