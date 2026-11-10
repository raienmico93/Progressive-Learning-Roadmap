# Built-in Borders — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Built-in Borders** are the concrete `Border` implementations supplied by the Java platform in the `javax.swing.border` package (and produced through the `BorderFactory` factory). Each class provides a specific visual decoration: a solid line (`LineBorder`), an etched groove (`EtchedBorder`), a 3D bevel (`BevelBorder` / `SoftBevelBorder`), invisible padding (`EmptyBorder`), colored bands or tiled images on the edges (`MatteBorder`), a labeled frame (`TitledBorder`), or a nested combination of two borders (`CompoundBorder`).

**Technical Definition.** All built-in borders extend `javax.swing.border.AbstractBorder`, which implements the `Border` interface and provides convenience methods. `LineBorder`, `EtchedBorder`, `BevelBorder`, `SoftBevelBorder`, `EmptyBorder`, `MatteBorder`, `TitledBorder`, and `CompoundBorder` each implement `javax.swing.plaf.UIResource` (directly or through their factory-provided instances), allowing the look and feel to replace them when the L&F changes. `BorderFactory` returns **shared instances** for many common configurations, particularly for `createLineBorder(Color)`, `createEmptyBorder()`, `createEtchedBorder()`, and `createBevelBorder(int)`. Every built-in border reports its painted area through `getBorderInsets(Component)`, which layout managers use to reserve space. All borders are effectively immutable; shared instances must not be mutated.

**Beginner-Friendly Explanation.** Built-in borders are the **ready-made picture frames** that Java ships with. Instead of writing custom painting code, you pick a frame type and attach it to a panel with `setBorder(...)`. Java gives you eight standard frames:

- A **solid line** (`LineBorder`)
- A **carved groove** (`EtchedBorder`)
- A **3D bevel** that looks raised or pressed (`BevelBorder`)
- A **softer 3D bevel** (`SoftBevelBorder`)
- An **invisible spacer** (`EmptyBorder`)
- **Colored bands or image strips** on the edges (`MatteBorder`)
- A **labeled group frame** (`TitledBorder`)
- Two frames **nested together** (`CompoundBorder`)

The easiest way to create any of them is through `BorderFactory`, which returns shared instances for efficiency.

---

### Key Characteristics

1. **`UIResource` compatibility.** Factory-built borders implement `UIResource`, so the current L&F may replace them at runtime. Borders created with the class constructors directly are not `UIResource` unless explicitly wrapped with `BorderUIResource`.
2. **Immutable and shareable.** Built-in borders are effectively immutable; the factory reuses shared instances for common configurations.
3. **Insets-aware.** Each border reports its painted thickness through `getBorderInsets`, which the component adds to its own insets.
4. **Composable.** `CompoundBorder` combines any two borders (outer + inner), enabling decoration-plus-padding patterns.
5. **Title support.** `TitledBorder` can wrap any other border with a text label and control its font, color, justification, and position.
6. **Color customisation.** Line, etched, bevel, matte, and titled borders accept explicit colors; defaults are derived from the component's background.
7. **Painted by the pipeline.** `paintBorder` runs after `paintComponent` and before `paintChildren`, so borders paint over the background but under children.
8. **Layout integration.** Insets correctly reserved keep layout managers from overlapping content with the border decoration.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces.
- Basic Swing: `JComponent`, `JPanel`, `setBorder`, `SwingUtilities.invokeLater`.
- The painting pipeline: `paintComponent` → `paintBorder` → `paintChildren`.
- Basic Java 2D: `Graphics`, `Color`, `Font`, `Insets`, `Graphics2D`.
- A JDK (version 8 or later); `getBorderInsets(Component, Insets)` is a default method since Java 9.

---

### Related Programming Areas

- **`javax.swing.border.Border`** — the parent interface.
- **`javax.swing.border.AbstractBorder`** — the shared base class.
- **`javax.swing.BorderFactory`** — the preferred factory.
- **`javax.swing.plaf.BorderUIResource`** — the `UIResource` wrapper.
- **`javax.swing.JComponent.setBorder` / `getBorder`** — installation methods.
- **`javax.swing.border.TitledBorder`** — the only built-in border that wraps another border.
- **`javax.swing.border.CompoundBorder`** — the general-purpose combination border.
- **`javax.swing.border.MatteBorder`** — the only built-in border with per-edge thickness support.

---

## 1. LineBorder

### Definitions

**Core Definition.** `LineBorder` is a built-in border that draws a solid line of a specified color and thickness around the component, optionally with rounded corners.

**Technical Definition.** `public class LineBorder extends AbstractBorder implements UIResource`. Constructors: `LineBorder(Color color)`, `LineBorder(Color color, int thickness)`, `LineBorder(Color color, int thickness, boolean roundedCorners)`. `BorderFactory.createLineBorder(Color)` returns a **shared instance** for thickness 1; `createLineBorder(Color, int)` and `createLineBorder(Color, int, boolean)` return new instances.

**Beginner-Friendly Explanation.** `LineBorder` is a **simple colored frame** around the component.

---

### Purposes

- To provide a simple visual frame around a component or panel.
- To emphasise important components with a colored outline.
- To create thin visual separators between sections.
- To draw a focus or selection indicator.
- To serve as the outer border in a `CompoundBorder` combined with padding.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Border thin  = BorderFactory.createLineBorder(Color.GRAY);
Border thick = BorderFactory.createLineBorder(Color.DARK_GRAY, 2);
Border round = BorderFactory.createLineBorder(Color.BLUE, 2, true);
panel.setBorder(thick);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `Color color` | The line color. |
| `int thickness` | The line thickness in pixels; default 1. |
| `boolean roundedCorners` | Whether to round the corners; default `false`. |

#### Syntax Rules

- Thickness is drawn **inside** the component's bounds; insets equal the thickness.
- Thickness 1 returns a shared instance from the factory.
- Rounded corners on thick lines produce smoother appearance with anti-aliased rendering.

#### Constraints and Limitations

- Very thick lines consume significant space from the content area.
- The line is painted at integer coordinates; at fractional scales the line may appear thicker or thinner.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic LineBorder

```java
import javax.swing.*;
import java.awt.*;

public class LineBorderPanel extends JPanel {
    public LineBorderPanel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createLineBorder(Color.DARK_GRAY, 2));
        add(new JLabel("Content inside 2px line border"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("LineBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new LineBorderPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel surrounded by a 2-pixel dark-gray line, with the label centered inside.

**Why the code produces this result.** `BorderFactory.createLineBorder(Color.DARK_GRAY, 2)` creates a 2-pixel-thick line border; `setBorder` installs it; the panel's `paintBorder` draws the outline.

---

#### Example 2: Rounded LineBorder

```java
JPanel panel = new JPanel();
panel.setBorder(BorderFactory.createLineBorder(
    new Color(0, 120, 215), 3, true));                              // rounded
```

**Expected Output.** A panel with a 3-pixel blue rounded rectangle frame.

---

### Real-World Cases

- **Selection highlights** around the currently focused component.
- **Form section frames** inside a larger layout.
- **Error indicators** using red line borders around invalid inputs.
- **Emphasis frames** around important metrics in dashboards.

---

## 2. EtchedBorder

### Definitions

**Core Definition.** `EtchedBorder` is a built-in border that draws a two-color etched effect, producing a subtle carved groove (lowered) or raised ridge.

**Technical Definition.** `public class EtchedBorder extends AbstractBorder implements UIResource`. Constants: `RAISED` and `LOWERED`. Constructors: `EtchedBorder()`, `EtchedBorder(int etchType)`, `EtchedBorder(Color highlight, Color shadow)`, `EtchedBorder(int etchType, Color highlight, Color shadow)`. `BorderFactory.createEtchedBorder()` returns a shared lowered instance; `createEtchedBorder(int)`, `createEtchedBorder(Color, Color)`, and `createEtchedBorder(int, Color, Color)` return new instances.

**Beginner-Friendly Explanation.** `EtchedBorder` draws a **thin engraved line** with a light and a dark side, producing a subtle 3D effect.

---

### Purposes

- To group related components with a subtle visual frame.
- To reproduce classic UI conventions (grid lines, frames).
- To draw lightweight dividers without the visual weight of a solid line.
- To serve as an inner border inside a `TitledBorder`.
- To create a raised or lowered appearance without the hardness of a bevel.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Border lowered = BorderFactory.createEtchedBorder(EtchedBorder.LOWERED);
Border raised  = BorderFactory.createEtchedBorder(EtchedBorder.RAISED);
Border custom  = BorderFactory.createEtchedBorder(
    EtchedBorder.LOWERED, new Color(255,255,255), new Color(150,150,150));
panel.setBorder(lowered);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int etchType` | `EtchedBorder.RAISED` or `EtchedBorder.LOWERED`. |
| `Color highlight` | The light edge of the etch. |
| `Color shadow` | The dark edge of the etch. |

#### Syntax Rules

- Default colors are derived from the component's background.
- Explicit colors override the derived defaults.

#### Constraints and Limitations

- Etched effects are subtle; at small sizes they may not be visible on low-contrast displays.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Etched Section

```java
import javax.swing.*;
import javax.swing.border.EtchedBorder;
import java.awt.*;

public class EtchedPanel extends JPanel {
    public EtchedPanel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createEtchedBorder(EtchedBorder.LOWERED));
        add(new JLabel("Etched grouped section"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("EtchedBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new EtchedPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a subtle lowered groove around its edge, looking slightly carved in.

---

#### Example 2: Custom Etched Colors

```java
JPanel panel = new JPanel();
panel.setBorder(BorderFactory.createEtchedBorder(
    EtchedBorder.RAISED,
    Color.WHITE,               // highlight
    new Color(100, 100, 100))); // shadow
```

**Expected Output.** A panel with a custom highlighted ridge.

---

### Real-World Cases

- **Legacy-style group frames**.
- **Table dividers** rendered with subtle etched lines.
- **Settings sections** combined with a titled border.

---

## 3. BevelBorder

### Definitions

**Core Definition.** `BevelBorder` is a built-in border that draws a raised or lowered 3D bevel with light and dark edges on the top/left and bottom/right sides.

**Technical Definition.** `public class BevelBorder extends AbstractBorder implements UIResource`. Constants: `RAISED` and `LOWERED`. Constructors: `BevelBorder(int bevelType)`, `BevelBorder(int bevelType, Color highlight, Color shadow)`, `BevelBorder(int bevelType, Color highlightOuter, Color highlightInner, Color shadowOuter, Color shadowInner)`. `BorderFactory.createBevelBorder(int)` returns a shared instance; other overloads return new instances.

**Beginner-Friendly Explanation.** `BevelBorder` draws a **3D-looking edge**: a raised block appears to pop out, and a lowered block appears pushed in. The effect is created by painting a light line on two sides and a dark line on the other two.

---

### Purposes

- To give panels a physical, 3D appearance.
- To mimic classic toolbar and button frames.
- To emphasise raised or recessed containers.
- To build a retro UI appearance when combined with other bevel elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Border raised  = BorderFactory.createBevelBorder(BevelBorder.RAISED);
Border lowered = BorderFactory.createBevelBorder(BevelBorder.LOWERED);
Border custom  = BorderFactory.createBevelBorder(
    BevelBorder.RAISED, Color.WHITE, Color.GRAY);
panel.setBorder(raised);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int bevelType` | `BevelBorder.RAISED` or `BevelBorder.LOWERED`. |
| `Color highlight` | Light side color. |
| `Color shadow` | Dark side color. |

#### Syntax Rules

- Two colors or four colors can be supplied; four-color versions use separate outer/inner colors for finer control.

#### Constraints and Limitations

- The visual effect depends on the background color; mismatched colors can produce an unpleasant look.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Raised Panel

```java
import javax.swing.*;
import javax.swing.border.BevelBorder;
import java.awt.*;

public class RaisedBevel extends JPanel {
    public RaisedBevel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createBevelBorder(BevelBorder.RAISED));
        add(new JLabel("Raised 3D panel"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("BevelBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new RaisedBevel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a raised 3D bevel: light on the top/left, dark on the bottom/right.

---

#### Example 2: Lowered Bevel

```java
JPanel panel = new JPanel();
panel.setBorder(BorderFactory.createBevelBorder(BevelBorder.LOWERED));
```

**Expected Output.** A panel that appears recessed, with light on the bottom/right and dark on the top/left.

---

### Real-World Cases

- **Classic toolbar separators**.
- **Status bars** with a raised or lowered effect.
- **Retro-styled UI panels**.

---

## 4. SoftBevelBorder

### Definitions

**Core Definition.** `SoftBevelBorder` is a variant of `BevelBorder` with **rounded corners** on the bevel, producing a softer appearance.

**Technical Definition.** `public class SoftBevelBorder extends BevelBorder`. Constants `RAISED` and `LOWERED` inherited from `BevelBorder`. Constructors: `SoftBevelBorder(int bevelType)`, `SoftBevelBorder(int bevelType, Color highlight, Color shadow)`, `SoftBevelBorder(int bevelType, Color highlightOuter, Color highlightInner, Color shadowOuter, Color shadowInner)`. `BorderFactory.createSoftBevelBorder(int)` creates one.

**Beginner-Friendly Explanation.** `SoftBevelBorder` is like a `BevelBorder` but with **rounded corners**, giving a smoother, softer 3D look.

---

### Purposes

- To provide a softer 3D edge than `BevelBorder`.
- To match rounded visual styles in modern or semi-modern UIs.
- To create a subtle raised or recessed container.
- To complement `TitledBorder` in a rounded group frame.

---

### Syntax Rules and Structure

```java
Border raised = BorderFactory.createSoftBevelBorder(BevelBorder.RAISED);
Border lowered = BorderFactory.createSoftBevelBorder(BevelBorder.LOWERED);
panel.setBorder(raised);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int bevelType` | `BevelBorder.RAISED` or `BevelBorder.LOWERED`. |
| `Color highlight` | Light side color. |
| `Color shadow` | Dark side color. |

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Soft Raised Panel

```java
import javax.swing.*;
import javax.swing.border.BevelBorder;
import java.awt.*;

public class SoftBevelPanel extends JPanel {
    public SoftBevelPanel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createSoftBevelBorder(BevelBorder.RAISED));
        add(new JLabel("Soft raised panel"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SoftBevelBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new SoftBevelPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a softened 3D raised edge (rounded corners).

---

### Real-World Cases

- **Rounded UI cards**.
- **Soft-styled status panels**.
- **Modern-leaning legacy UIs**.

---

## 5. EmptyBorder

### Definitions

**Core Definition.** `EmptyBorder` is a built-in border that paints nothing and only reserves space as padding.

**Technical Definition.** `public class EmptyBorder extends AbstractBorder implements UIResource`. Constructor: `EmptyBorder(int top, int left, int bottom, int right)`. `BorderFactory.createEmptyBorder(top, left, bottom, right)` returns a shared instance; a no-arg version is also available.

**Beginner-Friendly Explanation.** `EmptyBorder` is **invisible padding**. It adds whitespace around the component's content without drawing any frame.

---

### Purposes

- To add consistent padding around a component's content.
- To serve as the inner border of a `CompoundBorder` so padding surrounds content inside a visible frame.
- To provide a single side with extra space (e.g., extra top padding).
- To visually align multiple panels by adding uniform insets.
- To simulate margins in layouts that lack their own spacing mechanisms.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Border all  = BorderFactory.createEmptyBorder(10, 10, 10, 10);
Border top  = BorderFactory.createEmptyBorder(20, 0, 0, 0);
Border side = BorderFactory.createEmptyBorder(0, 15, 0, 15);
panel.setBorder(all);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int top` | Top padding in pixels. |
| `int left` | Left padding in pixels. |
| `int bottom` | Bottom padding in pixels. |
| `int right` | Right padding in pixels. |

#### Syntax Rules

- No visual painting; only insets.
- Any side may be 0.

#### Constraints and Limitations

- Because the border is invisible, misaligned insets are hard to spot and diagnose.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Uniform Padding

```java
import javax.swing.*;
import java.awt.*;

public class EmptyBorderPanel extends JPanel {
    public EmptyBorderPanel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createEmptyBorder(20, 20, 20, 20));
        add(new JLabel("Content with 20px padding"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("EmptyBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new EmptyBorderPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with 20 pixels of invisible padding around the label.

---

#### Example 2: Combing Padding with Decoration

```java
panel.setBorder(BorderFactory.createCompoundBorder(
    BorderFactory.createLineBorder(Color.GRAY),
    BorderFactory.createEmptyBorder(10, 15, 10, 15)));
```

**Expected Output.** A gray line border around 10/15-pixel interior padding.

---

### Real-World Cases

- **Panel spacing** in grid layouts.
- **Inner padding** inside titled groups.
- **Cushioning side content** inside a compound border.

---

## 6. MatteBorder

### Definitions

**Core Definition.** `MatteBorder` is a built-in border that draws solid color bands or tiled images on the edges of the component, with **independent thickness** for each side.

**Technical Definition.** `public class MatteBorder extends EmptyBorder`. Constructors: `MatteBorder(int top, int left, int bottom, int right, Color matteColor)` and `MatteBorder(int top, int left, int bottom, int right, Icon tileIcon)`. `BorderFactory.createMatteBorder(...)` provides both variants.

**Beginner-Friendly Explanation.** `MatteBorder` draws **edge bands of different thicknesses**, either in a solid color or as a tiled image.

---

### Purposes

- To create asymmetric borders (e.g., a thick bottom bar and thin sides).
- To fill edges with an image tile or pattern.
- To simulate "underline" or "header" effects on panels.
- To combine per-edge customization that other borders cannot provide.
- To visually mimic a table cell or card with varied border weights.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Border color  = BorderFactory.createMatteBorder(0, 0, 4, 0, Color.BLUE);
Border icon   = BorderFactory.createMatteBorder(4, 4, 4, 4, someIcon);
Border mixed  = BorderFactory.createMatteBorder(2, 0, 2, 0, Color.GRAY);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int top, left, bottom, right` | Per-side thickness in pixels. |
| `Color matteColor` | The solid color used for the painted bands. |
| `Icon tileIcon` | Icon used as a tile pattern for the edges. |

#### Syntax Rules

- Thickness 0 on a side means no band on that side.
- The image variant repeats the icon across the border strips.

#### Constraints and Limitations

- The image-based variant can be expensive if the icon is large.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Bottom Bar

```java
import javax.swing.*;
import java.awt.*;

public class MatteBorderPanel extends JPanel {
    public MatteBorderPanel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createMatteBorder(
            0, 0, 4, 0, new Color(0, 120, 215)));                    // bottom bar
        add(new JLabel("Panel with 4px bottom bar"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("MatteBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new MatteBorderPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a 4-pixel blue bar on its bottom edge only.

---

#### Example 2: Image-Tiled Matte

```java
ImageIcon pattern = new ImageIcon("pattern.png");
panel.setBorder(BorderFactory.createMatteBorder(6, 6, 6, 6, pattern));
```

**Expected Output.** A panel whose edges are tiled with the given pattern.

---

### Real-World Cases

- **Header or footer stripes** on panels.
- **Tab-like underlines** in navigation.
- **Framed thumbnails** with image-tiled edges.

---

## 7. TitledBorder

### Definitions

**Core Definition.** `TitledBorder` is a built-in border that draws a text title on top of another border, producing a labeled group frame.

**Technical Definition.** `public class TitledBorder extends AbstractBorder implements UIResource`. Constants: justification (`LEFT`, `CENTER`, `RIGHT`, `LEADING`, `TRAILING`, `DEFAULT_JUSTIFICATION`) and position (`ABOVE_TOP`, `TOP`, `BELOW_TOP`, `ABOVE_BOTTOM`, `BOTTOM`, `BELOW_BOTTOM`, `DEFAULT_POSITION`). Constructors: `TitledBorder(String title)`, `TitledBorder(Border border)`, `TitledBorder(Border border, String title)`, `TitledBorder(Border border, String title, int titleJustification, int titlePosition)`, `TitledBorder(Border border, String title, int titleJustification, int titlePosition, Font titleFont)`, `TitledBorder(Border border, String title, int titleJustification, int titlePosition, Font titleFont, Color titleColor)`. `BorderFactory.createTitledBorder(...)` provides matching overloads.

**Beginner-Friendly Explanation.** `TitledBorder` is a **frame with a label**. It wraps another border (or uses a default line border) and displays a title on the top edge.

---

### Purposes

- To group related components under a labeled section.
- To provide accessible, visible structure to a form.
- To display a title above a bordered area without adding a separate `JLabel`.
- To match the look-and-feel's title styling by default.
- To visually organise content in settings and preference dialogs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
TitledBorder titled = BorderFactory.createTitledBorder("Section Title");
panel.setBorder(titled);

// With an explicit underlying border and styling:
TitledBorder custom = BorderFactory.createTitledBorder(
    BorderFactory.createLineBorder(Color.GRAY),
    "Section Title",
    TitledBorder.LEFT,
    TitledBorder.ABOVE_TOP,
    new Font("SansSerif", Font.BOLD, 14),
    Color.DARK_GRAY);
panel.setBorder(custom);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Border border` | The underlying border; defaults to a line border. |
| `String title` | The text shown on the border. |
| `int justification` | `LEFT`, `CENTER`, `RIGHT`, `LEADING`, `TRAILING`, `DEFAULT_JUSTIFICATION`. |
| `int position` | `ABOVE_TOP`, `TOP`, `BELOW_TOP`, `ABOVE_BOTTOM`, `BOTTOM`, `BELOW_BOTTOM`, `DEFAULT_POSITION`. |
| `Font titleFont` | Optional title font. |
| `Color titleColor` | Optional title color. |

#### Syntax Rules

- Passing only the title uses a default line border and default styling.
- The underlying border's insets are expanded to accommodate the title.

#### Constraints and Limitations

- The title is drawn on the border, so it must fit; a very long title widens the border.
- Title styling overrides the L&F's defaults when explicitly supplied.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple TitledBorder

```java
import javax.swing.*;
import java.awt.*;

public class TitledBorderPanel extends JPanel {
    public TitledBorderPanel() {
        setLayout(new GridLayout(2, 1, 5, 5));
        setBorder(BorderFactory.createTitledBorder("Preferences"));  // 1. Title
        add(new JCheckBox("Enable auto-save"));
        add(new JCheckBox("Show tooltips"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("TitledBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new TitledBorderPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a titled border reading "Preferences" and two checkboxes stacked inside.

---

#### Example 2: Fully Customised TitledBorder

```java
TitledBorder border = BorderFactory.createTitledBorder(
    BorderFactory.createLineBorder(Color.GRAY),
    "Advanced Settings",
    TitledBorder.RIGHT,
    TitledBorder.TOP,
    new Font("SansSerif", Font.BOLD, 16),
    new Color(0, 90, 160));
panel.setBorder(border);
```

**Expected Output.** A titled border with a right-justified bold title in blue above the border.

---

### Real-World Cases

- **Preference dialogs** grouping related settings.
- **Registration forms** with labelled sections.
- **Data-entry cards** grouping fields.

---

## 8. CompoundBorder

### Definitions

**Core Definition.** `CompoundBorder` is a built-in border that wraps an **outer** border and an **inner** border, painting both around the same component.

**Technical Definition.** `public class CompoundBorder extends AbstractBorder implements UIResource`. Constructors: `CompoundBorder()` (allows nulls), `CompoundBorder(Border outsideBorder, Border insideBorder)`. `BorderFactory.createCompoundBorder(outer, inner)` creates one; `outer` may be `null`.

**Beginner-Friendly Explanation.** `CompoundBorder` is a **border of two borders**. It nests one inside the other, so you can combine decoration with padding.

---

### Purposes

- To combine visual decoration and padding (`LineBorder` + `EmptyBorder`).
- To layer multiple decorations (e.g., a titled border with a bevel and an empty border).
- To reuse existing borders without rebuilding the appearance.
- To fine-tune the spacing between a border and the content it surrounds.
- To compose consistent section frames for forms and dialogs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Border compound = BorderFactory.createCompoundBorder(
    BorderFactory.createLineBorder(Color.GRAY),
    BorderFactory.createEmptyBorder(10, 10, 10, 10));
panel.setBorder(compound);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Border outsideBorder` | Painted outside; may be `null`. |
| `Border insideBorder` | Painted inside; may be `null`. |

#### Syntax Rules

- Insets are the sum of both borders' insets.
- Both borders can be `null`, producing an empty border.

#### Constraints and Limitations

- Deep nesting of compound borders adds cumulative insets; excessive nesting reduces usable space.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Decoration Plus Padding

```java
import javax.swing.*;
import java.awt.*;

public class CompoundPanel extends JPanel {
    public CompoundPanel() {
        setLayout(new FlowLayout());
        setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(Color.GRAY, 1),
            BorderFactory.createEmptyBorder(10, 15, 10, 15)));
        add(new JLabel("Content inside a framed pad"));
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("CompoundBorder Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new CompoundPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with a 1-pixel gray frame, 10 pixels of padding above and below the content, and 15 pixels on the sides.

---

#### Example 2: Three-Level Nesting

```java
Border inner = BorderFactory.createEmptyBorder(8, 8, 8, 8);
Border middle = BorderFactory.createEtchedBorder(EtchedBorder.LOWERED);
Border outer = BorderFactory.createTitledBorder(middle, "Group");
panel.setBorder(BorderFactory.createCompoundBorder(outer, inner));
```

**Expected Output.** A titled etched border around 8 pixels of padding.

---

### Real-World Cases

- **Form sections** with a titled frame plus internal padding.
- **Card panels** with a colored frame and interior breathing room.
- **Consistent spacing** around bordered components.

---

## 9. Built-in Border Best Practices

### Definitions

**Core Definition.** Built-in border best practices are the conventions and patterns that ensure built-in borders are used correctly, efficiently, and consistently with layout managers and look and feels.

**Technical Definition.** Best practices include: using `BorderFactory` for border construction to leverage shared instances and `UIResource` compatibility; combining decoration with padding via `CompoundBorder`; applying borders on `JPanel`s rather than on look-and-feel-managed components; avoiding deep nesting that eats content space; ensuring custom styling of `TitledBorder` keeps the title legible on the underlying border; and setting borders on the EDT.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that keep built-in borders looking good and behaving correctly. They include things like "use `BorderFactory`," "combine with `EmptyBorder` for padding," "don't put borders on buttons," and "keep the title readable."

---

### Purposes

- To reduce memory usage through shared factory instances.
- To retain `UIResource` compatibility so L&F changes work correctly.
- To avoid breaking look-and-feel-managed components (buttons, text fields).
- To keep layouts predictable by using correct insets.
- To maintain visual consistency across the application.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
JPanel form = new JPanel(new GridLayout(2, 2, 5, 5));
form.setBorder(BorderFactory.createCompoundBorder(                 // 1. Compose
    BorderFactory.createTitledBorder(                              // 2. Titled
        BorderFactory.createLineBorder(Color.GRAY), "User Info"),
    BorderFactory.createEmptyBorder(10, 10, 10, 10)));             // 3. Padding
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `BorderFactory` | `BorderFactory.createLineBorder(...)` etc. |
| Combine decoration and padding | `CompoundBorder(decoration, EmptyBorder(...))` |
| Set borders on `JPanel` | Not on `JFrame`, `JDialog`, or look-and-feel components. |
| Prefer `UIResource` borders | Factory-built borders |
| Avoid deep nesting | Limit compound nesting to 2–3 levels. |
| Ensure title legibility | Choose a contrasting title color and font. |
| EDT | `SwingUtilities.invokeLater` for border installation. |

#### Syntax Rules

- **Always use `BorderFactory`** rather than constructing borders directly, unless you specifically need a non-`UIResource` border.
- **Always combine decoration and padding** with `CompoundBorder` when both are needed.
- **Always set borders on `JPanel` or `JComponent` subclasses**, not on look-and-feel-managed components.
- **Never paint custom borders outside their insets** and **never report insets that disagree with the painted area**.

#### Constraints and Limitations

- Setting borders on `JButton`, `JTextField`, or other L&F-managed components may disrupt their appearance.
- Excessive nesting of `CompoundBorder` insets can shrink content drastically.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Grouped Form

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeBorder {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Borders");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel form = new JPanel(new GridLayout(2, 2, 5, 5));
            form.setBorder(BorderFactory.createCompoundBorder(
                BorderFactory.createTitledBorder(
                    BorderFactory.createLineBorder(Color.GRAY),
                    "User Information"),
                BorderFactory.createEmptyBorder(10, 10, 10, 10)));
            form.add(new JLabel("Name:"));
            form.add(new JTextField(15));
            form.add(new JLabel("Email:"));
            form.add(new JTextField(15));

            frame.add(form);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a titled gray frame "User Information" enclosing a 2×2 grid of labels and text fields, with 10 pixels of padding.

---

#### Example 2: Common Mistake — Setting Borders on Buttons

```java
JButton button = new JButton("Click");
button.setBorder(BorderFactory.createLineBorder(Color.RED));      // WRONG
```

**Expected Output.** The button loses its standard L&F border and may look broken; pressed and focus states may not paint correctly.

**Why this is wrong.** `JButton` uses its border for the standard appearance. Replacing it breaks the L&F's conventions.

---

#### Example 3: Common Mistake — Mismatched Insets in Custom Borders

```java
public class BadBorder extends AbstractBorder {
    @Override public void paintBorder(Component c, Graphics g,
                                      int x, int y, int w, int h) {
        g.setColor(Color.RED);
        g.drawRect(x, y, w - 1, h - 1);                            // 1. paint
    }
    @Override public Insets getBorderInsets(Component c) {
        return new Insets(0, 0, 0, 0);                             // WRONG: 0
    }
}
```

**Expected Output.** The border paints a rectangle but reports zero insets, so it overlaps the component content and may be clipped.

**Why this is wrong.** Insets must match the painted area; otherwise layout managers and content painters will overlap with the border.

---

### Real-World Cases

- **Production Swing applications** use factory borders combined with `EmptyBorder` padding.
- **Settings dialogs** use `TitledBorder` and `CompoundBorder` for grouped sections.
- **Dashboards** use `LineBorder` or `MatteBorder` for emphasis.
- **Legacy maintenance** uses `EtchedBorder` and `BevelBorder` for retro appearances.

---

## References

Oracle. *Class LineBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/LineBorder.html

Oracle. *Class EtchedBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/EtchedBorder.html

Oracle. *Class BevelBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/BevelBorder.html

Oracle. *Class SoftBevelBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/SoftBevelBorder.html

Oracle. *Class EmptyBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/EmptyBorder.html

Oracle. *Class MatteBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/MatteBorder.html

Oracle. *Class TitledBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/TitledBorder.html

Oracle. *Class CompoundBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/CompoundBorder.html

Oracle. *Class BorderFactory*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/BorderFactory.html

Oracle. *Class AbstractBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/AbstractBorder.html

Oracle. *Class BorderUIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/BorderUIResource.html

Oracle. *How to Use Borders*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/border.html

Cleverence. *Java Swing Border (Java SE 8): Complete Guide with Examples*. https://www.cleverence.com/articles/oracle-documentation/border-java-platform-se-8-4827/