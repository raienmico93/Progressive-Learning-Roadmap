# BorderFactory — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `BorderFactory` is a utility class in the `javax.swing` package that serves as the **primary entry point for creating standard Swing `Border` objects**. It provides a collection of static factory methods that produce instances of the built-in border types without requiring direct construction of concrete border classes. Wherever possible, the factory returns **shared, immutable instances**, reducing memory overhead and encouraging consistency across the UI.

**Technical Definition.** `public class BorderFactory extends Object`. It is a **factory class for vending standard `Border` objects**. The class consists entirely of static methods that return `Border` implementations for the eight built-in border types: `LineBorder`, `EtchedBorder`, `BevelBorder`, `SoftBevelBorder`, `EmptyBorder`, `MatteBorder`, `TitledBorder`, and `CompoundBorder`. The factory does **not** replace the `Border` interface; it simply provides convenient creation methods. The only Swing border **not** supported by `BorderFactory` is `SoftBevelBorder` (which must be constructed directly). All factory methods return objects that implement the `Border` interface, and many of these objects also implement `javax.swing.plaf.UIResource`, signalling that the look and feel may replace them when the L&F changes.

**Beginner-Friendly Explanation.** `BorderFactory` is the **border-building convenience class** for Swing. Instead of writing `new LineBorder(Color.GRAY, 2)` and remembering the constructor signatures for every border type, you call `BorderFactory.createLineBorder(Color.GRAY, 2)`. The factory handles the details, reuses border instances where possible, and returns objects that work correctly with the current look and feel. Think of it as the **standard parts counter** for borders: you ask for the border you need by name, and the factory hands you a ready-to-use instance.

---

### Key Characteristics

1. **Static factory pattern.** All methods are `static`; the class is never instantiated. You call methods such as `BorderFactory.createLineBorder(...)`, `BorderFactory.createEmptyBorder(...)`, and `BorderFactory.createTitledBorder(...)` directly.
2. **Shared instances.** The factory returns **references to shared `Border` instances** wherever possible. For example, repeated calls to `createLineBorder(Color.GRAY)` may return the same instance, reducing memory usage and object churn.
3. **`UIResource` compatibility.** Most borders returned by `BorderFactory` implement `UIResource` (either directly or through their class hierarchy), allowing the current look and feel to replace them when the L&F changes. This ensures that borders created by the application can integrate cleanly with the active theme.
4. **Eight border types, one entry point.** The factory covers all standard border types **except `SoftBevelBorder`**, which must be constructed directly if needed.
5. **Parameterised creation.** Many factory methods offer overloads that accept colors, thicknesses, titles, alignments, fonts, and icon tiles, giving fine-grained control over the created border.
6. **No custom borders.** `BorderFactory` produces only the built-in border types. For unique decorations, you must implement `Border` (typically by extending `AbstractBorder`).
7. **Immutable output.** Borders returned by the factory are effectively immutable; they are designed to be shared and reused across multiple components.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, and static method invocation.
- Basic Swing: `JComponent`, `JPanel`, `setBorder`, and `SwingUtilities.invokeLater`.
- Understanding of the `Border` interface and its role in the painting pipeline.
- Familiarity with `Color`, `Font`, `Insets`, and `Icon` (for `MatteBorder`).
- A JDK (version 8 or later); `BorderFactory` has been available since Java 1.2.

---

### Related Programming Areas

- **`javax.swing.border.Border`** — the interface implemented by all borders returned by the factory.
- **`javax.swing.border.AbstractBorder`** — the convenience base class for custom borders.
- **`javax.swing.JComponent.setBorder`** — the method used to install a border on a component.
- **`javax.swing.BorderFactory`** — the class itself.
- **`javax.swing.plaf.BorderUIResource`** — the `UIResource` wrapper for borders.
- **`javax.swing.border.TitledBorder`** — supports justification and position constants used with the factory.
- **`javax.swing.border.BevelBorder`** and **`javax.swing.border.EtchedBorder`** — provide constants (`RAISED`, `LOWERED`) used as factory parameters.

---

## Core Concepts and Features

---

## 1. BorderFactory — The Factory Class

### Definitions

**Core Definition.** `BorderFactory` is the standard utility for creating Swing border objects without direct construction of concrete border classes.

**Technical Definition.** `public class BorderFactory extends Object`. It provides static methods that return `Border` instances. The class is documented as a "factory class for vending standard `Border` objects" that "wherever possible, will hand out references to shared `Border` instances".

**Beginner-Friendly Explanation.** `BorderFactory` is the **border supermarket**: you ask for the border you need, and the factory gives you a ready-to-use instance.

---

### Purposes

- To provide a single, convenient entry point for creating all standard Swing borders.
- To return shared instances where possible, reducing memory overhead.
- To ensure created borders are `UIResource`-compatible when appropriate.
- To encapsulate the correct constructor parameters and default values for each border type.
- To encourage consistent border styling across an application.

---

### Syntax Rules and Structure

#### Complete General Syntax (Calling the Factory)

```java
Border border = BorderFactory.createXxxBorder(parameters);
component.setBorder(border);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `BorderFactory` | The factory class (never instantiated). |
| `createXxxBorder(...)` | A static factory method for a specific border type. |
| `Border border` | The returned border; stored in a variable for reuse. |
| `setBorder(border)` | Installs the border on a component. |

#### Syntax Rules

- All methods are static; call them on the class name, not an instance.
- Store the returned `Border` in a variable if you need to reuse it across components.
- Borders should be set on `JPanel`, `JLabel`, or custom `JComponent` subclasses; avoid setting borders on look-and-feel-managed components such as `JButton` or `JTextField`.

#### Constraints and Limitations

- `BorderFactory` does not create `SoftBevelBorder`; use `new SoftBevelBorder(...)` directly if needed.
- For custom borders, you must implement `Border` or extend `AbstractBorder`; the factory does not cover custom decorations.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Border Creation

```java
import javax.swing.*;
import java.awt.*;

public class BorderFactoryBasics {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel panel = new JPanel();
            Border border = BorderFactory.createLineBorder(Color.GRAY); // 1. Create
            panel.setBorder(border);                                     // 2. Install
            panel.add(new JLabel("Border from BorderFactory"));
            JFrame frame = new JFrame("BorderFactory Basics");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel surrounded by a 1-pixel gray line border.

**Why the code produces this result.** `BorderFactory.createLineBorder(Color.GRAY)` returns a shared `LineBorder` instance; `setBorder` installs it; the panel's `paintBorder` draws the outline.

---

### Real-World Cases

- **Consistent styling:** defining a small set of border constants (e.g., `PANEL_BORDER`, `SECTION_BORDER`) and reusing them throughout the application.
- **Rapid prototyping:** creating borders quickly during development without memorizing constructor signatures.
- **Theming:** using factory-created `UIResource` borders that the look and feel can replace when the theme changes.

---

## 2. Border Factory Methods — Complete Overview

### Definitions

**Core Definition.** Border factory methods are the static methods of `BorderFactory` that create specific border types.

**Technical Definition.** The factory provides methods for the following border types: `LineBorder`, `EtchedBorder`, `BevelBorder`, `EmptyBorder`, `MatteBorder`, `TitledBorder`, and `CompoundBorder`. ( `SoftBevelBorder` is not directly covered by the factory.) Each method offers overloads for common parameter combinations.

| Factory Method | Returns | Border Type |
|----------------|---------|-------------|
| `createLineBorder(Color)` | `Border` | `LineBorder` (1-pixel) |
| `createLineBorder(Color, int)` | `Border` | `LineBorder` (custom thickness) |
| `createLineBorder(Color, int, boolean)` | `Border` | `LineBorder` (rounded corners) |
| `createEtchedBorder()` | `Border` | `EtchedBorder` (LOWERED) |
| `createEtchedBorder(int)` | `Border` | `EtchedBorder` (RAISED/LOWERED) |
| `createEtchedBorder(Color, Color)` | `Border` | `EtchedBorder` (custom colors) |
| `createEtchedBorder(int, Color, Color)` | `Border` | `EtchedBorder` (type + colors) |
| `createBevelBorder(int)` | `Border` | `BevelBorder` (RAISED/LOWERED) |
| `createBevelBorder(int, Color, Color)` | `Border` | `BevelBorder` (highlight/shadow) |
| `createBevelBorder(int, Color, Color, Color, Color)` | `Border` | `BevelBorder` (outer/inner colors) |
| `createLoweredBevelBorder()` | `Border` | `BevelBorder` (LOWERED) |
| `createRaisedBevelBorder()` | `Border` | `BevelBorder` (RAISED) |
| `createEmptyBorder()` | `Border` | `EmptyBorder` (zero) |
| `createEmptyBorder(int, int, int, int)` | `Border` | `EmptyBorder` (top/left/bottom/right) |
| `createMatteBorder(int, int, int, int, Color)` | `Border` | `MatteBorder` (color) |
| `createMatteBorder(int, int, int, int, Icon)` | `Border` | `MatteBorder` (icon tile) |
| `createTitledBorder(String)` | `Border` | `TitledBorder` (default border) |
| `createTitledBorder(Border, String)` | `Border` | `TitledBorder` (specified border) |
| `createTitledBorder(Border, String, int, int)` | `Border` | `TitledBorder` (justification + position) |
| `createTitledBorder(Border, String, int, int, Font)` | `Border` | `TitledBorder` (font) |
| `createTitledBorder(Border, String, int, int, Font, Color)` | `Border` | `TitledBorder` (font + color) |
| `createCompoundBorder(Border, Border)` | `Border` | `CompoundBorder` (outer + inner) |

#### Component Breakdown

| Method | Key Parameters |
|--------|----------------|
| `createLineBorder` | `Color`, `int thickness`, `boolean roundedCorners` |
| `createEtchedBorder` | `int type` (`RAISED`/`LOWERED`), `Color highlight`, `Color shadow` |
| `createBevelBorder` | `int type`, `Color highlight`, `Color shadow`, and four-color variants |
| `createEmptyBorder` | `int top, left, bottom, right` |
| `createMatteBorder` | `int top, left, bottom, right`, `Color` or `Icon` |
| `createTitledBorder` | `Border`, `String title`, `int justification`, `int position`, `Font`, `Color` |
| `createCompoundBorder` | `Border outside`, `Border inside` |

#### Syntax Rules

- Parameter order for `createCompoundBorder` is **outside border first, inside border second**.
- `TitledBorder` justification constants include `TitledBorder.LEFT`, `CENTER`, `RIGHT`, `LEADING`, `TRAILING`, and `DEFAULT_JUSTIFICATION`.
- `TitledBorder` position constants include `ABOVE_TOP`, `TOP`, `BELOW_TOP`, `ABOVE_BOTTOM`, `BOTTOM`, `BELOW_BOTTOM`, and `DEFAULT_POSITION`.
- `MatteBorder` requires four thickness integers and either a `Color` or an `Icon`.

#### Constraints and Limitations

- The factory does not create `SoftBevelBorder`; construct it directly.
- Some overloads may return shared instances; do not mutate the returned borders.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using Multiple Factory Methods

```java
import javax.swing.*;
import java.awt.*;

public class FactoryMethodsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel panel = new JPanel(new GridLayout(2, 2, 10, 10));
            panel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

            JPanel p1 = new JPanel();
            p1.setBorder(BorderFactory.createLineBorder(Color.GRAY, 2)); // 1. Line
            p1.add(new JLabel("Line"));

            JPanel p2 = new JPanel();
            p2.setBorder(BorderFactory.createEtchedBorder()); // 2. Etched
            p2.add(new JLabel("Etched"));

            JPanel p3 = new JPanel();
            p3.setBorder(BorderFactory.createTitledBorder("Section")); // 3. Titled
            p3.add(new JLabel("Titled"));

            JPanel p4 = new JPanel();
            p4.setBorder(BorderFactory.createCompoundBorder( // 4. Compound
                BorderFactory.createLineBorder(Color.BLUE),
                BorderFactory.createEmptyBorder(8, 8, 8, 8)));
            p4.add(new JLabel("Compound"));

            panel.add(p1);
            panel.add(p2);
            panel.add(p3);
            panel.add(p4);

            JFrame frame = new JFrame("Factory Methods");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 2×2 grid of panels, each with a different border type: a 2-pixel gray line, an etched groove, a titled section, and a blue line with internal padding.

---

### Real-World Cases

- **Form sections:** combining `createTitledBorder` with `createEmptyBorder` via `createCompoundBorder` to create labeled groups with padding.
- **Emphasis panels:** using `createLineBorder` or `createMatteBorder` to draw attention to important information.
- **Status indicators:** using `createEtchedBorder` or `createBevelBorder` to create subtle 3D effects.

---

## 3. Border Factory Best Practices

### Definitions

**Core Definition.** Border factory best practices are the conventions that ensure borders are created, used, and maintained correctly, efficiently, and consistently with the look and feel.

**Technical Definition.** Best practices include: using `BorderFactory` rather than direct construction; storing and reusing border instances; combining decoration and padding via `createCompoundBorder`; setting borders on `JPanel` rather than look-and-feel components; understanding that factory borders are `UIResource`-compatible; and handling `SoftBevelBorder` separately.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that keep border code clean and safe: use the factory, reuse what you create, put borders on panels, and don't fight the look and feel.

---

### Purposes

- To reduce object creation overhead by reusing borders.
- To maintain `UIResource` compatibility so the look and feel can adapt borders to the current theme.
- To avoid breaking the appearance of look-and-feel-managed components.
- To keep border code concise, readable, and consistent.
- To combine decoration and padding cleanly with `CompoundBorder`.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyPanel extends JPanel {
    private static final Border SECTION_BORDER =
        BorderFactory.createCompoundBorder(
            BorderFactory.createTitledBorder("Section"),
            BorderFactory.createEmptyBorder(10, 10, 10, 10));  // 1. Reusable

    public MyPanel() {
        setBorder(SECTION_BORDER);                              // 2. Reuse
        // ...
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use the factory | `BorderFactory.create...(...)` |
| Reuse instances | Store borders in `static final` fields |
| Combine decoration + padding | `BorderFactory.createCompoundBorder(...)` |
| Set borders on `JPanel` | Not on `JButton`, `JTextField`, etc. |
| Understand `UIResource` | Factory borders may be replaced by L&F changes. |
| Handle `SoftBevelBorder` | Construct directly if needed. |
| EDT | Create and install borders on the EDT. |

#### Syntax Rules

- **Always use `BorderFactory`** rather than constructing borders directly, unless you need a non-`UIResource` border.
- **Always reuse border instances** by storing them in fields; borders are immutable and safe to share.
- **Always combine decoration and padding** with `createCompoundBorder` rather than nesting panels.
- **Always set borders on `JPanel`** rather than on look-and-feel-managed components.
- **Always set the outside border first** in `createCompoundBorder(outer, inner)`.

#### Constraints and Limitations

- `BorderFactory` does not create `SoftBevelBorder`.
- Factory-returned borders may be shared; do not attempt to mutate them.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Reusable Border

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeBorderFactory {
    private static final Border PANEL_BORDER =
        BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(Color.GRAY, 1),
            BorderFactory.createEmptyBorder(10, 15, 10, 15));   // 1. Reusable

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JPanel form = new JPanel(new GridLayout(2, 2, 5, 5));
            form.setBorder(PANEL_BORDER);                           // 2. Reuse
            form.add(new JLabel("Name:"));
            form.add(new JTextField(15));
            form.add(new JLabel("Email:"));
            form.add(new JTextField(15));

            JFrame frame = new JFrame("Best Practice BorderFactory");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(form);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A form panel with a 1-pixel gray frame and 10/15-pixel internal padding, created once and reused.

---

#### Example 2: Common Mistake — Setting Borders on Look-and-Feel Components

```java
JButton button = new JButton("Click");
button.setBorder(BorderFactory.createLineBorder(Color.RED)); // WRONG
```

**Expected Output.** The button loses its standard look-and-feel border and may appear broken; pressed and focus states may not paint correctly.

**Why this is wrong.** `JButton` uses its border for its standard appearance. Setting a custom border breaks the look-and-feel's conventions. Put the button in a `JPanel` and set the border on the panel instead.

---

#### Example 3: Common Mistake — Misusing CompoundBorder Parameter Order

```java
Border bad = BorderFactory.createCompoundBorder(
    BorderFactory.createEmptyBorder(10, 10, 10, 10),
    BorderFactory.createLineBorder(Color.GRAY));              // WRONG ORDER
```

**Expected Output.** The empty border appears **outside** the line border, which may produce unexpected spacing (padding outside the frame rather than inside).

**Why this is wrong.** `createCompoundBorder` expects the **outside** border first and the **inside** border second. To place padding inside a line frame, the line border must come first.

---

## References

Oracle. *Class BorderFactory*. Java Platform, Standard Edition 11 API Specification. https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/swing/BorderFactory.html

Oracle. *Class BorderFactory*. Java Development Kit Version 24 API Specification. https://docs.oracle.com/en/java/javase/24/docs/api/java.desktop/javax/swing/BorderFactory.html

Oracle. *How to Use Borders*. The Java Tutorials. http://dev.cs.ovgu.de/tutorials/Sun_Java_Tutorials/uiswing/components/border.html

Cleverence. *Java Swing Border (Java SE 8) — Complete Guide with Examples*. https://www.cleverence.com/articles/oracle-documentation/border-java-platform-se-8-4827/

Cleverence. *Java Swing Borders: Complete Guide to Cleaner, Clearer GUIs*. https://www.cleverence.com/articles/oracle-documentation/border-java-platform-se-8-4827/

Oracle. *Class Border*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/Border.html

Oracle. *Class TitledBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/TitledBorder.html

Oracle. *Class CompoundBorder*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/border/CompoundBorder.html

Stack Overflow. *BorderFactory best practices — shared instances and static factory methods*. https://stackoverflow.com/