# Font — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `Font` is the Java platform's representation of a specific typographic face, characterized by a **name**, a **style**, and a **size**. It is the object you install on a `Graphics` context or a `JComponent` to control how text is rendered — its typeface, weight, slant, and point size — throughout Java 2D and Swing.

**Technical Definition.** `public class Font extends Object implements Serializable`. A `Font` encapsulates a **logical name** (the name passed to the constructor), a **style** (a bitmask of `PLAIN`, `BOLD`, and `ITALIC`), and a **point size** (stored both as a rounded `int` and as a `float` for precision). The Java platform defines five **logical font families** — `Dialog`, `DialogInput`, `Serif`, `SansSerif`, and `Monospaced` — that map to physical fonts available on the host system. The `Font` class provides methods for deriving new fonts (`deriveFont`), measuring text (`getStringBounds`, `getLineMetrics`), checking glyph coverage (`canDisplay`, `canDisplayUpTo`), and obtaining metrics (`getFontMetrics`). Fonts are **immutable**: all derivation methods return new `Font` instances.

**Beginner-Friendly Explanation.** A `Font` is the **"handwriting style"** of text on screen. Every piece of text you draw in Swing — labels, buttons, titles, chart annotations — uses a font. A font is defined by three things: its **face** (the typeface family, like "SansSerif" or "Serif"), its **style** (plain, bold, italic, or bold-italic), and its **size** (measured in points, where 1 point is 1/72 of an inch). You create a `Font` with `new Font(name, style, size)`, install it on a `Graphics` or `JComponent` with `setFont`, and from then on all text drawn uses that font. Think of a `Font` as a **rubber stamp** with a specific lettering style: you press it onto the page once, and every character you print afterwards carries that style until you switch stamps.

---

### Key Characteristics

1. **Three core properties.** Every `Font` has a **name** (logical or physical), a **style** (bitmask of `PLAIN`, `BOLD`, `ITALIC`), and a **point size** (int and float accessors).
2. **Immutable.** `Font` instances cannot be mutated. To change a property, you **derive** a new font with `deriveFont(...)`.
3. **Logical vs. physical names.** Logical names (`Dialog`, `DialogInput`, `Serif`, `SansSerif`, `Monospaced`) are guaranteed to exist on every JVM; physical names (e.g., `Arial`, `Times New Roman`) are platform-specific.
4. **Platform-independent API.** `Font` provides a uniform abstraction over the host's fonts, but the actual appearance depends on the installed physical fonts.
5. **Metric measurement.** `FontMetrics` and `Font.getStringBounds` allow precise measurement of text width, height, ascent, descent, and leading.
6. **Glyph coverage.** `canDisplay(int codePoint)` and `canDisplayUpTo(String)` let you check whether a font can render a given character — crucial for internationalised text.
7. **Anti-aliasing control.** Text rendering quality is controlled through `RenderingHints.KEY_TEXT_ANTIALIASING`, not through `Font` itself.
8. **Fallback behaviour.** If a requested physical font cannot be found, the font system maps the `Font` to `"Dialog"`.
9. **Derived fonts.** `deriveFont(int style)`, `deriveFont(float size)`, `deriveFont(int style, float size)`, and `deriveFont(AffineTransform)` produce new fonts from an existing one.
10. **Loaded fonts.** `Font.createFont(...)` loads a TrueType or Type 1 font from a file or stream; the returned `Font` must be registered in the `GraphicsEnvironment` to be usable by `Font` constructors.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and anonymous classes.
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and the `create()`/`dispose()` idiom.
- **The `Graphics.setFont` / `JComponent.setFont` APIs.**
- **Coordinate system and user space:** points as units in the Java 2D coordinate system.
- **Basic typographic concepts:** baseline, ascent, descent, leading, and point size.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`java.awt.FontMetrics`** — measures text width, height, ascent, descent, and leading.
- **`java.awt.Graphics2D`** — holds the current `Font` attribute and renders text with `drawString`.
- **`javax.swing.JComponent.setFont`** — sets the font for a Swing component and all its descendants.
- **`java.awt.GraphicsEnvironment`** — lists available font families and registers loaded fonts.
- **`java.awt.font.FontRenderContext`** — captures the rendering hints, transform, and anti-aliasing state used for text layout.
- **`java.awt.font.LineMetrics`** — per-line typographic metrics (ascent, descent, leading).
- **`java.awt.font.TextLayout`** — advanced text layout with attributed strings.
- **`RenderingHints.KEY_TEXT_ANTIALIASING`** — controls text anti-aliasing quality.
- **`Font.createFont` and `GraphicsEnvironment.registerFont`** — load and register custom fonts.

---

## Core Concepts and Features

---

## 1. Font — The Class

### Definitions

**Core Definition.** `Font` is the class that represents a specific typeface, style, and size, and that is used to render text in Java 2D and Swing.

**Technical Definition.** `public class Font extends Object implements Serializable`. Constructor: `Font(String name, int style, int size)`. The `name` can be a logical name, a family name, or a face name. The `style` is a bitwise union of `PLAIN`, `BOLD`, and `ITALIC`. The `size` is the point size (1/72 inch in user coordinates). `Font` instances are immutable.

**Beginner-Friendly Explanation.** `Font` is the class you use to describe how text should look. You create one with a name, a style, and a size, and install it on a component or graphics context.

---

### Purposes

- To specify the typeface, weight, slant, and size of rendered text.
- To provide a platform-independent abstraction over physical fonts.
- To supply metrics for measuring and laying out text.
- To check glyph coverage for internationalisation.
- To serve as the basis for derived fonts (styles and sizes) and loaded fonts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Font f1 = new Font("SansSerif", Font.PLAIN, 14);
Font f2 = new Font("Serif", Font.BOLD, 18);
Font f3 = new Font("Monospaced", Font.ITALIC | Font.BOLD, 12);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String name` | Logical name (`Dialog`, `SansSerif`, etc.) or physical name (`Arial`, `Times New Roman`). |
| `int style` | `Font.PLAIN` (0), `Font.BOLD` (1), `Font.ITALIC` (2), or a bitwise union. |
| `int size` | Point size (≥ 1). |

#### Syntax Rules

- Style is a **bitmask**: `BOLD | ITALIC` produces a bold-italic font.
- The name lookup is **case-insensitive** using US locale rules.
- If a physical name cannot be mapped, the font system falls back to `"Dialog"`.
- `Font` is immutable; use `deriveFont` to create variants.

#### Constraints and Limitations

- A logical name is guaranteed to exist; a physical name may not be available on all platforms.
- The size must be `≥ 1`; `deriveFont` allows fractional sizes.
- Fonts cannot represent every Unicode code point; use `canDisplay` to check.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating and Using a Font

```java
import javax.swing.*;
import java.awt.*;

public class FontDemoPanel extends JPanel {
    public FontDemoPanel() {
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
            Font plain = new Font("SansSerif", Font.PLAIN, 18);     // 1. Create
            Font bold = new Font("SansSerif", Font.BOLD, 18);       // 2. Bold
            Font italic = new Font("Serif", Font.ITALIC, 18);       // 3. Serif italic

            g2.setFont(plain);
            g2.setColor(Color.BLACK);
            g2.drawString("Plain SansSerif", 20, 50);

            g2.setFont(bold);
            g2.drawString("Bold SansSerif", 20, 90);

            g2.setFont(italic);
            g2.drawString("Italic Serif", 20, 130);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Font Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new FontDemoPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three lines of 18-point text: plain SansSerif, bold SansSerif, and italic Serif.

**Why the code produces this result.** Each `Font` object encapsulates a name, style, and size. `setFont` installs the font on the `Graphics2D`; subsequent `drawString` calls use it.

---

### Real-World Cases

- **UI consistency:** using a single base font family with different sizes and styles across labels, buttons, and titles.
- **Charts and data visualisation:** configuring axis labels, titles, and annotations with specific fonts.
- **Internationalisation:** selecting fonts that cover the required scripts (CJK, Arabic, etc.).
- **Custom renderers:** applying fonts to list, table, and tree cells.

---

## 2. Font Properties

### 2.1 Name

**Definitions.** **Core:** The font's name, either a logical name, a family name, or a face name. **Technical:** `getName()`, `getFamily()`, `getFontName()` return different interpretations of the name. **Beginner:** The typeface name, such as "Arial" or "SansSerif".

**Purposes.** To identify the typeface; to select the correct font from those available; to support fallback logic.

**Syntax Rules.** `new Font("Arial", ...)` uses a physical name; `new Font("SansSerif", ...)` uses a logical name.

**Constraints.** Physical names are platform-dependent; logical names are guaranteed.

**Example.**

```java
Font f = new Font("Arial", Font.PLAIN, 14);
System.out.println(f.getName());       // "Arial" (logical/face name)
System.out.println(f.getFamily());     // "Arial" (family)
```

**Real-World.** Choosing a font family for an entire application.

---

### 2.2 Style

**Definitions.** **Core:** The font's weight and slant: plain, bold, italic, or bold-italic. **Technical:** `getStyle()` returns the bitmask; `isPlain()`, `isBold()`, `isItalic()` are convenience checks. **Beginner:** Whether the text is normal, bold, slanted, or both.

**Purposes.** To visually distinguish headings, emphasis, and code; to support semantic styling.

**Syntax Rules.** `Font.BOLD | Font.ITALIC` combines styles. The style is **merged** with any face style (e.g., a bold face plus bold style does not double-embolden).

**Constraints.** Algorithmic styling may be applied if the face lacks the requested style.

**Example.**

```java
Font bold = new Font("Serif", Font.BOLD, 16);
System.out.println(bold.isBold());    // true
System.out.println(bold.isItalic());  // false
```

**Real-World.** Emphasising headings and labels.

---

### 2.3 Size

**Definitions.** **Core:** The point size of the font, measured in typographic points (1/72 inch). **Technical:** `getSize()` returns the rounded int; `getSize2D()` returns the float. **Beginner:** How large the text appears.

**Purposes.** To control text size relative to UI density and reading distance.

**Syntax Rules.** `deriveFont(float size)` creates a font with a fractional size.

**Constraints.** Very large sizes increase rendering cost and may exceed layout bounds.

**Example.**

```java
Font f = new Font("SansSerif", Font.PLAIN, 24);
float exact = f.getSize2D();          // 24.0f
```

**Real-World.** Scaling UI text for high-DPI displays.

---

## 3. Font Methods

### Definitions

**Core Definition.** The `Font` methods provide access to properties, derivation of new fonts, glyph coverage checks, and text measurement.

**Technical Definition.** Key methods include `deriveFont`, `getFontMetrics`, `getStringBounds`, `getLineMetrics`, `canDisplay`, `canDisplayUpTo`, `getNumGlyphs`, `getMissingGlyphCode`, `getFamily`, `getFontName`, `getName`, `getStyle`, `getSize`, `getSize2D`, `isPlain`, `isBold`, `isItalic`, `isTransformed`, `getTransform`, `hasLayoutAttributes`.

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `deriveFont(int style)` | New font with the given style. |
| `deriveFont(float size)` | New font with the given size. |
| `deriveFont(int style, float size)` | New font with both style and size. |
| `getFontMetrics(Font)` | Returns a `FontMetrics` for measuring text. |
| `getStringBounds(String, FontRenderContext)` | Precise bounds of a string. |
| `getLineMetrics(...)` | Per-line typographic metrics. |
| `canDisplay(int)` / `canDisplay(char)` | Whether a glyph exists. |
| `canDisplayUpTo(String)` | Index of the first unrenderable char, or `-1`. |
| `getNumGlyphs()` | Number of glyphs in the font. |
| `getFamily()` / `getFontName()` / `getName()` | Different name forms. |
| `getStyle()` / `getSize()` / `getSize2D()` | Style and size accessors. |
| `isPlain()` / `isBold()` / `isItalic()` | Style checks. |
| `isTransformed()` / `getTransform()` | Transform state. |
| `hasLayoutAttributes()` | Whether advanced layout is required. |

#### Syntax Rules

- `deriveFont` returns a new `Font`; the original is unchanged.
- `getFontMetrics` requires a `FontRenderContext` (obtainable from `Graphics2D`).
- `canDisplayUpTo` returns `-1` if the entire string is displayable.

#### Constraints and Limitations

- `getFontMetrics` uses the current rendering context's hints; results may differ for different anti-aliasing settings.
- `canDisplay` does not guarantee visual quality; it only checks glyph availability.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Deriving Fonts

```java
import javax.swing.*;
import java.awt.*;

public class DerivedFontPanel extends JPanel {
    public DerivedFontPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 250));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            Font base = new Font("SansSerif", Font.PLAIN, 14);      // 1. Base
            Font bold = base.deriveFont(Font.BOLD);                  // 2. Style
            Font large = base.deriveFont(24f);                       // 3. Size
            Font largeBoldItalic = base.deriveFont(
                Font.BOLD | Font.ITALIC, 20f);                       // 4. Both

            g2.setFont(base);
            g2.drawString("Base 14pt", 20, 40);
            g2.setFont(bold);
            g2.drawString("Bold 14pt", 20, 80);
            g2.setFont(large);
            g2.drawString("Large 24pt", 20, 130);
            g2.setFont(largeBoldItalic);
            g2.drawString("Bold Italic 20pt", 20, 180);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Derived Fonts");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new DerivedFontPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Four lines of text derived from a single base font: base, bold, larger, and bold-italic larger.

**Why the code produces this result.** `deriveFont` creates new `Font` instances based on the original, preserving the family while changing style and/or size.

---

#### Example 2: Checking Glyph Coverage

```java
Font f = new Font("SansSerif", Font.PLAIN, 14);
boolean hasEuro = f.canDisplay('\u20AC');          // €
int badIndex = f.canDisplayUpTo("Hello 世界");
System.out.println("Euro supported: " + hasEuro);
System.out.println("First unsupported index: " + badIndex);
```

**Expected Output.** `true` or `false` for the Euro sign; `badIndex` indicates where the first unsupported character appears (or `-1` if all are supported).

---

#### Example 3: Measuring Text with FontMetrics

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    Font font = new Font("SansSerif", Font.BOLD, 18);
    g2.setFont(font);
    FontMetrics fm = g2.getFontMetrics();
    int width = fm.stringWidth("Hello");
    int ascent = fm.getAscent();
    int descent = fm.getDescent();
    System.out.println("Width: " + width + ", Ascent: " + ascent
        + ", Descent: " + descent);
} finally {
    g2.dispose();
}
```

**Expected Output.** Pixel width of “Hello”, the ascent (height above baseline), and the descent (height below baseline).

---

### Real-World Cases

- **UI scaling:** deriving larger fonts for high-DPI displays.
- **Emphasis:** deriving bold and italic variants for headings and alerts.
- **Measurement:** centring text precisely using `FontMetrics.stringWidth`.
- **Internationalisation:** checking `canDisplayUpTo` before rendering multilingual text.

---

## 4. Logical vs. Physical Fonts

### Definitions

**Core Definition.** **Logical fonts** are the five font families guaranteed by the Java platform (`Dialog`, `DialogInput`, `Serif`, `SansSerif`, `Monospaced`). **Physical fonts** are the actual font files installed on the host system (e.g., Arial, Times New Roman, Noto Sans).

**Technical Definition.** Logical fonts are **mapped** by the JVM to one or more physical fonts on the host platform. `getName()` returns the logical name; `getFamily()` returns the family name; `getFontName()` returns the face name. The mapping is platform-dependent, so the same logical font may render differently on Windows, macOS, and Linux.

**Beginner-Friendly Explanation.** Logical fonts are **guaranteed to exist** on every Java runtime — you can safely use "SansSerif" anywhere. Physical fonts are the **actual fonts** installed on the machine, which may vary.

---

### Purposes

- To ensure portability by using logical names.
- To access the specific look of platform fonts by using physical names.
- To support font fallback for missing glyphs.
- To enumerate available font families for font choosers.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Font logical = new Font("SansSerif", Font.PLAIN, 14);    // guaranteed
Font physical = new Font("Arial", Font.PLAIN, 14);       // platform-specific

GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();
String[] families = ge.getAvailableFontFamilyNames();     // list physical families
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| Logical names | `Dialog`, `DialogInput`, `Serif`, `SansSerif`, `Monospaced`. |
| Physical names | Any family or face name installed on the system. |
| `getAvailableFontFamilyNames()` | Returns the list of available family names. |

#### Syntax Rules

- Logical names are always available.
- Physical names may fall back to `"Dialog"` if unavailable.
- Use `GraphicsEnvironment.getAvailableFontFamilyNames()` to discover physical families.

#### Constraints and Limitations

- A physical font may not be available on all platforms.
- Logical fonts may map to different physical fonts, producing different appearances.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Listing Available Font Families

```java
import java.awt.*;

public class FontListDemo {
    public static void main(String[] args) {
        GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();
        String[] families = ge.getAvailableFontFamilyNames();     // 1. Enumerate
        System.out.println("Available font families:");
        for (String family : families) {
            System.out.println("  " + family);
        }
        System.out.println("Total: " + families.length);
    }
}
```

**Expected Output.** A list of all font family names installed on the system, followed by the total count.

---

#### Example 2: Falling Back to Logical Font

```java
Font f = new Font("NoSuchFont", Font.PLAIN, 14);               // 1. Unknown
System.out.println(f.getFamily());                             // "Dialog"
```

**Expected Output.** The family is `"Dialog"` because the requested physical font could not be mapped.

---

### Real-World Cases

- **Font choosers:** populate a combo box with `getAvailableFontFamilyNames()`.
- **Portable UI:** use logical fonts to ensure consistent availability.
- **Branding:** use a specific physical font if it is bundled or guaranteed.

---

## 5. Text Anti-Aliasing

### Definitions

**Core Definition.** Text anti-aliasing is the technique of smoothing the jagged edges of rendered text by blending edge pixels with the background. In Java 2D, it is controlled by the `RenderingHints.KEY_TEXT_ANTIALIASING` rendering hint.

**Technical Definition.** `Graphics2D.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON)` enables text anti-aliasing. Other values include `VALUE_TEXT_ANTIALIAS_OFF`, `VALUE_TEXT_ANTIALIAS_GASP`, and LCD subpixel variants (`VALUE_TEXT_ANTIALIAS_LCD_HRGB`, etc.). The hint is **advisory**; platform support varies. For Swing components, text anti-aliasing is often enabled globally via the `awt.useSystemAAFontSettings` system property.

**Beginner-Friendly Explanation.** Text anti-aliasing is what makes text look **smooth** instead of **jagged**. You enable it with a rendering hint, and the platform blends edge pixels so the letters look clean.

---

### Purposes

- To render text with smooth edges on screen.
- To improve readability, especially at small sizes.
- To match the visual quality of modern user interfaces.
- To support subpixel rendering on LCD displays.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Graphics2D g2 = (Graphics2D) g.create();
try {
    g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                        RenderingHints.VALUE_TEXT_ANTIALIAS_ON);    // 1. On
    g2.setFont(new Font("SansSerif", Font.PLAIN, 16));
    g2.drawString("Anti-Aliased Text", 20, 50);
} finally {
    g2.dispose();
}
```

#### Component Breakdown

| Hint | Meaning |
|------|---------|
| `KEY_TEXT_ANTIALIASING` | Controls text anti-aliasing. |
| `VALUE_TEXT_ANTIALIAS_ON` | Enables standard anti-aliasing. |
| `VALUE_TEXT_ANTIALIAS_OFF` | Disables anti-aliasing. |
| `VALUE_TEXT_ANTIALIAS_GASP` | Uses the font's embedded hinting. |
| `VALUE_TEXT_ANTIALIAS_LCD_HRGB` | LCD subpixel rendering (HRGB layout). |

#### Syntax Rules

- Set the hint **before** `drawString` or `setFont`+`drawString`.
- Text anti-aliasing is **separate** from shape anti-aliasing (`KEY_ANTIALIASING`).
- The hint is advisory; implementations may ignore it.

#### Constraints and Limitations

- Subpixel LCD rendering depends on the physical display; it may look wrong on non-LCD displays or when scaled.
- Anti-aliasing has a small performance cost.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Anti-Aliased and Non-Aliased Text

```java
import javax.swing.*;
import java.awt.*;

public class TextAaPanel extends JPanel {
    public TextAaPanel() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(500, 200));
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setFont(new Font("SansSerif", Font.PLAIN, 24));
            g2.setColor(Color.BLACK);

            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_OFF); // 1. Off
            g2.drawString("Non-Aliased", 20, 60);

            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);  // 2. On
            g2.drawString("Anti-Aliased", 20, 120);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Text Anti-Aliasing");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new TextAaPanel());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two lines of text: the top has jagged edges; the bottom has smooth edges.

---

#### Example 2: Global Anti-Aliasing for Swing

```java
public class GlobalAaDemo {
    public static void main(String[] args) {
        System.setProperty("awt.useSystemAAFontSettings", "on");   // 1. Global
        SwingUtilities.invokeLater(() -> {
            // Swing components use anti-aliased text
        });
    }
}
```

**Expected Output.** Swing components render text with anti-aliasing enabled globally.

---

### Real-World Cases

- **Production Swing applications** enable text anti-aliasing globally for consistent visual quality.
- **Custom-painted components** set the hint inside `paintComponent`.
- **Charts and dashboards** benefit from smooth axis labels and titles.

---

## 6. Font Best Practices

### Definitions

**Core Definition.** Font best practices are the conventions and patterns that ensure text rendering is portable, readable, performant, and maintainable.

**Technical Definition.** Best practices include: preferring logical font names for portability; caching derived fonts and `FontMetrics`; enabling text anti-aliasing; using `deriveFont` rather than constructing new fonts from scratch; checking `canDisplayUpTo` for international text; using `JComponent.setFont` rather than hard-coding sizes in layouts; and avoiding font loading in paint methods.

**Beginner-Friendly Explanation.** Best practices for fonts are the "rules of thumb" that keep text readable and rendering fast. They include things like "use logical fonts," "cache your fonts," "enable anti-aliasing," and "measure text properly."

---

### Purposes

- To ensure text is available and legible across platforms.
- To avoid performance problems from repeated font construction and measurement.
- To support internationalisation through glyph coverage checks.
- To maintain consistent typography across the application.
- To integrate fonts cleanly with layout managers and component sizing.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyPanel extends JPanel {
    private static final Font BASE = new Font("SansSerif", Font.PLAIN, 14);
    private static final Font HEADING = BASE.deriveFont(Font.BOLD, 18f);

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            g2.setFont(HEADING);
            g2.drawString("Heading", 20, 40);
            g2.setFont(BASE);
            g2.drawString("Body", 20, 80);
        } finally {
            g2.dispose();
        }
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Prefer logical fonts | `"SansSerif"`, `"Serif"`, `"Monospaced"` |
| Cache fonts | `static final Font` fields |
| Derive variants | `base.deriveFont(Font.BOLD, 18f)` |
| Enable anti-aliasing | `KEY_TEXT_ANTIALIASING` |
| Check glyph coverage | `canDisplayUpTo(text) == -1` |
| Use `JComponent.setFont` | Rather than manipulating `Graphics` inside Swing components |
| Avoid font loading in paint | Load fonts once and cache |

#### Syntax Rules

- **Prefer logical font names** for portability.
- **Cache derived fonts and `FontMetrics`** to avoid repeated computation.
- **Always enable text anti-aliasing** for smooth rendering.
- **Use `JComponent.setFont`** for Swing components; the component computes its preferred size from the font.
- **Never construct fonts inside `paintComponent`**; allocate once and cache.

#### Constraints and Limitations

- Logical fonts may not cover all scripts; use `canDisplayUpTo` to check.
- `FontMetrics` depend on the rendering context and anti-aliasing hints; measure with the correct context.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Swing Component with Cached Fonts

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeFont extends JPanel {
    private static final Font BASE = new Font("SansSerif", Font.PLAIN, 14);
    private static final Font HEADING = BASE.deriveFont(Font.BOLD, 20f);

    public BestPracticeFont() {
        setOpaque(true);
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 200));
        setLayout(new BorderLayout(10, 10));

        JLabel title = new JLabel("Heading");                      // 1. Label
        title.setFont(HEADING);                                     // 2. Set font
        title.setHorizontalAlignment(SwingConstants.CENTER);

        JLabel body = new JLabel("Body text in base font");         // 3. Body
        body.setFont(BASE);
        body.setHorizontalAlignment(SwingConstants.CENTER);

        add(title, BorderLayout.NORTH);
        add(body, BorderLayout.CENTER);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Font");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new BestPracticeFont());
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a bold 20-point heading centered at the top and 14-point body text centered below.

**Why the code produces this result.** The fonts are cached as static fields; `setFont` on `JLabel` allows the label to compute its preferred size from the font; no font allocation occurs in paint methods.

---

#### Example 2: Common Mistake — Creating Fonts in Paint

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    for (int i = 0; i < 50; i++) {
        g.setFont(new Font("SansSerif", Font.PLAIN, 12));         // WRONG
        g.drawString("Line " + i, 10, i * 15);
    }
}
```

**Expected Output.** Slow painting because 50 `Font` objects are allocated per repaint.

**Why this is wrong.** Fonts should be created once and cached as fields. Creating them in `paintComponent` is a performance anti-pattern.

---

#### Example 3: Common Mistake — Hard-Coding Physical Font Names

```java
Font f = new Font("Arial", Font.PLAIN, 14);                    // WRONG: not portable
```

**Expected Output.** On systems without Arial, the font falls back to `"Dialog"`, potentially producing an unexpected appearance.

**Why this is wrong.** Physical font names are platform-dependent. Logical names (`SansSerif`, `Serif`, `Monospaced`) are guaranteed and should be used for portable code.

---

### Real-World Cases

- **Production Swing applications** use logical fonts and cache derived variants for consistent typography.
- **Charts and dashboards** measure text with `FontMetrics` to position labels precisely.
- **Internationalised applications** check `canDisplayUpTo` before rendering multilingual text.
- **Educational examples** demonstrate cached fonts and anti-aliasing from the start.

---

## References

Oracle. *Class Font*. Java Platform, Standard Edition 6 API Specification. https://docs.oracle.com/cd/E17802_01/j2se/javase/6/jcp/mr2/apidiffs/java/awt/Font.html

Oracle. *Class FontMetrics*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/FontMetrics.html

Oracle. *Class GraphicsEnvironment*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/GraphicsEnvironment.html

Oracle. *Displaying Antialiased Text by Using Rendering Hints*. Oracle Documentation. https://docs.oracle.com/en/java/javase/...

Oracle. *Font Concepts — Logical and Physical Fonts*. Oracle Documentation. https://docs.oracle.com/en/java/javase/...

Oracle. *Class Font — deriveFont, canDisplay, getStringBounds*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Font.html

Oracle. *Class JComponent — setFont*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *Class Font — createFont*. Java Platform, Standard Edition 6 API Specification. https://docs.oracle.com/cd/E17802_01/j2se/javase/6/jcp/mr2/apidiffs/java/awt/Font.html

Stack Overflow. *Ugly Fonts in Java Swing — How to set antialiasing?* https://stackoverflow.com/posts/78056129/timeline

Stack Overflow. *Java Swing setFont best practice*. https://stackoverflow.com/...