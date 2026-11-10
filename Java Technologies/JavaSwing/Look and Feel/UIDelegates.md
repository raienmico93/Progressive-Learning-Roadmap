# UI Delegates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **UI Delegate** is a Swing class that implements the `ComponentUI` (or one of its specialized subclasses like `ButtonUI`, `LabelUI`, `TableUI`) interface and is responsible for the **painting, sizing, and event handling** of a specific `JComponent`. It is the "V" and part of the "C" in Swing's MVC-inspired architecture.

**Technical Definition.** `public abstract class ComponentUI extends Object`. `ComponentUI` is the abstract base class for all UI delegates. It defines the contract that every delegate must fulfill: `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `update(Graphics, JComponent)`, `getPreferredSize(JComponent)`, `getMinimumSize(JComponent)`, `getMaximumSize(JComponent)`, `contains(JComponent, int, int)`, and `createUI(JComponent)` (the static factory used by components to obtain their delegate). Specialized delegates extend `ComponentUI` and add component-specific methods: `ButtonUI`, `LabelUI`, `PanelUI`, `TableUI`, `TreeUI`, `ListUI`, `ScrollBarUI`, `SliderUI`, `SpinnerUI`, `TabbedPaneUI`, `ToolBarUI`, `RootPaneUI`, etc. The concrete delegates in `javax.swing.plaf.basic` (e.g., `BasicButtonUI`, `BasicLabelUI`, `BasicPanelUI`) provide the foundational, L&F-agnostic implementations that other L&Fs extend or replace.

**Beginner-Friendly Explanation.** A **UI Delegate** is the **"painter and behavior manager"** for a Swing component. When you create a `JButton`, the button itself doesn't know how to draw its border, its text, or its pressed state—it asks its UI delegate to do that. The button is a thin shell; the UI delegate does the actual drawing and handles mouse and keyboard events. Different look and feels provide different delegates: Metal's button delegate looks different from Nimbus's or Windows's. `ComponentUI` is the **base contract** that every delegate must implement. `BasicButtonUI` is the **standard button painter** that most L&Fs build on. `BasicLabelUI` is the **standard label painter**. Think of it as the difference between a picture frame (the `JComponent`) and the painting inside it (the `ComponentUI`): the frame is always the same shape, but the painting can be swapped for a completely different style.

---

### Key Characteristics

1. **Separation of concerns.** The `JComponent` holds state and API; the `ComponentUI` holds painting and input-handling logic.
2. **Pluggable architecture.** Every L&F provides its own set of `ComponentUI` implementations. Changing the L&F changes the delegates.
3. **One delegate per component.** Every `JComponent` has exactly one installed `ComponentUI`, accessible via `getUI()` and replaceable via `setUI()`.
4. **Install/uninstall lifecycle.** `installUI` is called when the delegate is installed; `uninstallUI` is called when it is removed. This is where listeners, layout managers, and defaults are wired up.
5. **Painting delegation.** The component's `paintComponent` calls the delegate's `paint` method; the delegate draws the component.
6. **Sizing delegation.** `getPreferredSize`, `getMinimumSize`, and `getMaximumSize` are delegated to the UI delegate.
7. **Hit testing.** `contains` determines whether a point is inside the component, which the delegate may override for custom shapes.
8. **Static factory pattern.** `createUI(JComponent)` returns a new delegate for the component; this is how `UIManager` creates delegates.
9. **Install/uninstall symmetric.** Every resource acquired in `installUI` should be released in `uninstallUI`.
10. **EDT discipline.** All delegate installation and painting must occur on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and anonymous classes.
- **Basic Swing:** `JComponent`, `JButton`, `JLabel`, and `SwingUtilities.invokeLater`.
- **Understanding of the `UIManager` and the L&F architecture.**
- **Knowledge of the `UIResource` convention.**
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and rendering hints.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`javax.swing.plaf.ComponentUI`** — the base delegate class.
- **`javax.swing.plaf.basic.BasicButtonUI`** — the standard button delegate.
- **`javax.swing.plaf.basic.BasicLabelUI`** — the standard label delegate.
- **`javax.swing.plaf.basic.BasicPanelUI`** — the standard panel delegate.
- **`javax.swing.plaf.ButtonUI`, `LabelUI`, `PanelUI`, `TableUI`, etc.** — component-specific delegate interfaces.
- **`UIManager`** — installs delegates via `getUI` and `setUI`.
- **`LookAndFeel`** — supplies the delegate class mappings via `UIDefaults`.
- **`SwingUtilities.updateComponentTreeUI`** — reinstalls delegates after an L&F change.

---

## 1. UI Delegates — The Concept

### Definitions

**Core Definition.** A UI delegate is a `ComponentUI` implementation that performs painting, sizing, and event handling for a `JComponent`.

**Technical Definition.** The UI delegate is retrieved by `JComponent.getUI()` and installed by `JComponent.setUI(ComponentUI)`. The component delegates its `paintComponent`, `getPreferredSize`, `getMinimumSize`, `getMaximumSize`, and `contains` methods to the delegate. The delegate is created by the static `createUI(JComponent)` factory method and configured through `installUI` and `uninstallUI`.

**Beginner-Friendly Explanation.** A UI delegate is the **component's painter and behavior handler**. It knows how to draw the component and how to respond to input.

---

### Purposes

- To separate the component's API from its visual representation and behavior.
- To enable pluggable L&Fs without changing component code.
- To allow multiple L&Fs to produce different appearances for the same component type.
- To allow custom delegates to be installed on individual components for specialised rendering.
- To centralise sizing, painting, and input handling in a single testable class.

---

### Syntax Rules and Structure

#### Complete General Syntax (Accessing and Replacing a Delegate)

```java
JButton button = new JButton("Click");
ComponentUI delegate = button.getUI();                            // 1. Get
System.out.println(delegate.getClass().getName());                 // 2. Inspect

// Replace the delegate:
button.setUI(new MyButtonUI());                                    // 3. Set
button.updateUI();                                                  // 4. Reset
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getUI()` | Returns the installed delegate. |
| `setUI(ComponentUI)` | Replaces the delegate. |
| `updateUI()` | Resets the delegate to the current L&F's delegate. |

#### Syntax Rules

- Every `JComponent` has a delegate after construction.
- `setUI` is normally followed by `revalidate()` and `repaint()`.
- `updateUI` is called automatically after an L&F change.
- All delegate access should occur on the EDT.

#### Constraints and Limitations

- Replacing a delegate without understanding the L&F's expectations can break behaviour.
- Custom delegates must implement the full `ComponentUI` contract.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting the UI Delegate

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;

public class InspectDelegate {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Inspect");                // 1. Create
            ComponentUI ui = button.getUI();                        // 2. Get
            System.out.println("Delegate class: " + ui.getClass().getName()); // 3. Inspect

            JFrame frame = new JFrame("Inspect Delegate");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints the fully qualified class name of the current `ButtonUI` delegate (e.g., `javax.swing.plaf.metal.MetalButtonUI`).

**Why the code produces this result.** `JButton.getUI()` returns the delegate installed by the current L&F.

---

### Real-World Cases

- **Debugging:** inspecting which delegate is active.
- **Custom components:** providing a custom delegate for specialised rendering.
- **L&F development:** implementing delegates for a new L&F.

---

## 2. ComponentUI — The Base Class

### Definitions

**Core Definition.** `ComponentUI` is the abstract base class for all UI delegates. It defines the methods that every delegate must implement.

**Technical Definition.** `public abstract class ComponentUI extends Object`. It declares `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `update(Graphics, JComponent)`, `getPreferredSize(JComponent)`, `getMinimumSize(JComponent)`, `getMaximumSize(JComponent)`, `contains(JComponent, int, int)`, and the static `createUI(JComponent)`. It also inherits `getAccessibleChildrenCount`, `getAccessibleChild`, and `getBaseline` methods (with default implementations).

**Beginner-Friendly Explanation.** `ComponentUI` is the **base contract** for every delegate. If you want to write your own delegate, you extend `ComponentUI` (or a subclass like `ButtonUI`) and implement its methods.

---

### Purposes

- To define the universal delegate contract.
- To provide default implementations for optional methods.
- To ensure every delegate can be installed, uninstalled, painted, and sized.
- To provide hit testing through `contains`.
- To serve as the parent of component-specific delegate interfaces.

---

### Syntax Rules and Structure

#### Complete General Syntax (The `ComponentUI` Contract)

```java
public class MyUI extends ComponentUI {
    public static ComponentUI createUI(JComponent c) { return new MyUI(); }
    public void installUI(JComponent c) { /* install listeners, layout */ }
    public void uninstallUI(JComponent c) { /* remove listeners */ }
    public void paint(Graphics g, JComponent c) { /* paint component */ }
    public void update(Graphics g, JComponent c) { paint(g, c); }
    public Dimension getPreferredSize(JComponent c) { /* ... */ }
    public Dimension getMinimumSize(JComponent c) { /* ... */ }
    public Dimension getMaximumSize(JComponent c) { /* ... */ }
    public boolean contains(JComponent c, int x, int y) {
        return x >= 0 && x < c.getWidth() && y >= 0 && y < c.getHeight();
    }
}
```

#### Component Breakdown

| Method | Responsibility |
|--------|----------------|
| `createUI(JComponent)` | Static factory; returns a new delegate instance. |
| `installUI(JComponent)` | Called when the delegate is installed; sets up state. |
| `uninstallUI(JComponent)` | Called when the delegate is removed; tears down state. |
| `paint(Graphics, JComponent)` | Paints the component. |
| `update(Graphics, JComponent)` | Clears and repaints; default calls `paint`. |
| `getPreferredSize(JComponent)` | Preferred size. |
| `getMinimumSize(JComponent)` | Minimum size. |
| `getMaximumSize(JComponent)` | Maximum size. |
| `contains(JComponent, int, int)` | Hit test. |

#### Syntax Rules

- Every delegate must provide `installUI`, `uninstallUI`, `paint`, and the size methods.
- `installUI` and `uninstallUI` must be symmetric.
- `createUI` should return a new instance each time (or a shared singleton for stateless delegates).

#### Constraints and Limitations

- `ComponentUI` does not enforce implementation of all methods, but subclasses must override the ones they need.
- Default implementations of size methods return `null`, so subclasses must override them.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Custom `ComponentUI`

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;
import java.awt.*;

public class DiagonalFillUI extends ComponentUI {                    // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new DiagonalFillUI();
    }

    @Override
    public void installUI(JComponent c) {                              // 3. Install
        c.setOpaque(true);
        c.setBackground(Color.LIGHT_GRAY);
    }

    @Override
    public void uninstallUI(JComponent c) {                            // 4. Uninstall
        // nothing to release
    }

    @Override
    public void paint(Graphics g, JComponent c) {                     // 5. Paint
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            int w = c.getWidth(), h = c.getHeight();
            g2.setColor(c.getBackground());
            g2.fillRect(0, 0, w, h);
            g2.setColor(Color.DARK_GRAY);
            for (int x = -h; x < w; x += 10) {
                g2.drawLine(x, 0, x + h, h);
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Custom Painter");
            button.setUI(new DiagonalFillUI());                       // 6. Install
            JFrame frame = new JFrame("Custom UI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button rendered with a light-gray background crossed by diagonal dark-gray lines.

**Why the code produces this result.** `setUI` replaces the button's delegate with `DiagonalFillUI`, whose `paint` draws the diagonal pattern. The button's text is not painted because the delegate does not call `super.paint` or paint text.

---

### Real-World Cases

- **Custom components** that need specialised rendering.
- **Themed UI** elements that go beyond what L&F defaults provide.
- **Prototyping** new UI styles before packaging them as full L&Fs.

---

## 3. BasicButtonUI — The Standard Button Delegate

### Definitions

**Core Definition.** `BasicButtonUI` is the foundational `ButtonUI` implementation that provides standard painting and behaviour for all button subclasses (`JButton`, `JToggleButton`, `JCheckBox`, `JRadioButton`).

**Technical Definition.** `public class BasicButtonUI extends ButtonUI`. It provides `createUI(JComponent)`, `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `paintText(Graphics, AbstractButton, Rectangle, String)`, `paintIcon(Graphics, AbstractButton, Rectangle)`, `paintFocus(Graphics, AbstractButton, Rectangle, Rectangle)`, `paintButtonPressed(Graphics, AbstractButton)`, `getPreferredSize(JComponent)`, `getMinimumSize(JComponent)`, and `getMaximumSize(JComponent)`. It reads defaults from `UIDefaults` (text, font, icon, margins, etc.) and delegates to overridable paint methods.

**Beginner-Friendly Explanation.** `BasicButtonUI` is the **standard button painter**. It knows how to draw the text, icon, focus rectangle, and pressed state of any button. Most L&Fs extend `BasicButtonUI` (or one of its subclasses) and override the paint methods to add their own style.

---

### Purposes

- To provide a complete, working button delegate.
- To serve as the base class for L&F-specific button delegates.
- To define overridable paint stages (text, icon, focus, pressed).
- To read button properties from `UIDefaults` for L&F customization.
- To provide standard sizing and hit-testing.

---

### Syntax Rules and Structure

#### Complete General Syntax (Extending BasicButtonUI)

```java
public class MyButtonUI extends BasicButtonUI {
    public static ComponentUI createUI(JComponent c) {
        return new MyButtonUI();
    }

    @Override
    protected void paintButtonPressed(Graphics g, AbstractButton b) {
        // custom pressed state painting
    }

    @Override
    protected void paintText(Graphics g, AbstractButton b, Rectangle textRect, String text) {
        // custom text painting
    }

    @Override
    protected void paintFocus(Graphics g, AbstractButton b,
                              Rectangle viewRect, Rectangle textRect,
                              Rectangle iconRect) {
        // custom focus painting
    }
}
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `createUI(JComponent)` | Returns a new `BasicButtonUI` (or subclass). |
| `installUI(JComponent)` | Sets up listeners, defaults. |
| `uninstallUI(JComponent)` | Tears down listeners, defaults. |
| `paint(Graphics, JComponent)` | Orchestrates the painting stages. |
| `paintText(...)` | Paints button text. |
| `paintIcon(...)` | Paints button icon. |
| `paintFocus(...)` | Paints focus rectangle. |
| `paintButtonPressed(...)` | Paints pressed state. |
| `getPreferredSize(JComponent)` | Sizing based on text, icon, margins. |

#### Syntax Rules

- `BasicButtonUI` is abstract; `createUI` is the recommended factory.
- Subclasses should call `super` methods to preserve default behaviour.
- The `paint` method calls `paintButtonPressed` if the button is pressed, then `paintText`, then `paintFocus`.

#### Constraints and Limitations

- `BasicButtonUI` does not paint the button background by default; backgrounds are painted by the component or L&F-specific delegate.
- Custom delegates must handle the button's `ButtonModel` state (pressed, armed, rollover, selected).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: BasicButtonUI with Custom Text Painting

```java
import javax.swing.*;
import javax.swing.plaf.basic.BasicButtonUI;
import java.awt.*;

public class CustomButtonUI extends BasicButtonUI {                   // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new CustomButtonUI();
    }

    @Override
    protected void paintText(Graphics g, AbstractButton b,            // 3. Override
                             Rectangle textRect, String text) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            g2.setColor(b.isEnabled() ? Color.WHITE : Color.GRAY);
            g2.setFont(b.getFont());
            g2.drawString(text, textRect.x, textRect.y + g2.getFontMetrics().getAscent());
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Custom Text");
            button.setBackground(new Color(0, 120, 215));             // 4. Custom
            button.setForeground(Color.WHITE);
            button.setUI(new CustomButtonUI());                        // 5. Install
            JFrame frame = new JFrame("BasicButtonUI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button with a blue background and white anti-aliased text "Custom Text".

**Why the code produces this result.** The custom delegate overrides `paintText` to draw the text with anti-aliasing and a custom color. The button's background is painted by the component (since `BasicButtonUI` doesn't paint backgrounds for non-opaque buttons, but `setBackground` combined with `setOpaque(true)` may be needed depending on the L&F).

---

### Real-World Cases

- **Themed buttons** that need custom text rendering.
- **Icon buttons** that need custom icon positioning.
- **Branded UI** with custom pressed states.

---

## 4. BasicLabelUI — The Standard Label Delegate

### Definitions

**Core Definition.** `BasicLabelUI` is the foundational `LabelUI` implementation that provides standard painting and sizing for `JLabel`.

**Technical Definition.** `public class BasicLabelUI extends LabelUI implements PropertyChangeListener`. It provides `createUI(JComponent)`, `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `layout(LabelUI, JLabel, FontMetrics, String, Icon, Rectangle, Rectangle, Rectangle)`, `paintEnabledText(JLabel, Graphics, String, int, int)`, `paintDisabledText(JLabel, Graphics, String, int, int)`, and sizing methods. It respects the label's text, icon, horizontal alignment, vertical alignment, and text position.

**Beginner-Friendly Explanation.** `BasicLabelUI` is the **standard label painter**. It knows how to lay out the label's text and icon and paint them in the correct positions.

---

### Purposes

- To provide a complete, working label delegate.
- To serve as the base class for L&F-specific label delegates.
- To lay out text and icon according to alignment and position properties.
- To paint enabled and disabled text with different colors.
- To read label properties from `UIDefaults` for L&F customization.

---

### Syntax Rules and Structure

#### Complete General Syntax (Extending BasicLabelUI)

```java
public class MyLabelUI extends BasicLabelUI {
    public static ComponentUI createUI(JComponent c) {
        return new MyLabelUI();
    }

    @Override
    protected void paintEnabledText(JLabel l, Graphics g, String s,
                                    int textX, int textY) {
        // custom enabled text painting
    }

    @Override
    protected void paintDisabledText(JLabel l, Graphics g, String s,
                                     int textX, int textY) {
        // custom disabled text painting
    }

    @Override
    public void paint(Graphics g, JComponent c) {
        super.paint(g, c);
        // extra decorations
    }
}
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `createUI(JComponent)` | Returns a new `BasicLabelUI` (or subclass). |
| `installUI(JComponent)` | Sets up listeners. |
| `paint(Graphics, JComponent)` | Lays out and paints the text and icon. |
| `layout(...)` | Computes positions for text, icon, and focus rect. |
| `paintEnabledText(...)` | Paints text for enabled labels. |
| `paintDisabledText(...)` | Paints text for disabled labels. |
| `getPreferredSize(JComponent)` | Sizing based on text, icon, insets. |

#### Syntax Rules

- `BasicLabelUI` calls `layout` to compute positions, then paints the text and icon.
- Subclasses should call `super` to preserve default painting unless replacing it entirely.

#### Constraints and Limitations

- `BasicLabelUI` does not draw a border; the label's border is painted by the component's `paintBorder`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: BasicLabelUI with Custom Disabled Text

```java
import javax.swing.*;
import javax.swing.plaf.basic.BasicLabelUI;
import java.awt.*;

public class CustomLabelUI extends BasicLabelUI {                     // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new CustomLabelUI();
    }

    @Override
    protected void paintDisabledText(JLabel l, Graphics g, String s,  // 3. Override
                                     int textX, int textY) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.LIGHT_GRAY);
            g2.drawString(s, textX, textY);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JLabel label = new JLabel("Disabled Label");
            label.setEnabled(false);                                    // 4. Disabled
            label.setUI(new CustomLabelUI());                           // 5. Install
            JFrame frame = new JFrame("BasicLabelUI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(label);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A disabled label with light-gray text instead of the default gray.

**Why the code produces this result.** The custom delegate overrides `paintDisabledText` to paint the text in light gray. All other behavior (layout, sizing) is inherited from `BasicLabelUI`.

---

#### Example 2: BasicLabelUI with Additional Decoration

```java
public class DecoratedLabelUI extends BasicLabelUI {
    public static ComponentUI createUI(JComponent c) {
        return new DecoratedLabelUI();
    }

    @Override
    public void paint(Graphics g, JComponent c) {
        super.paint(g, c);                                            // 1. Default
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.RED);                                    // 2. Decor
            g2.drawRect(0, 0, c.getWidth() - 1, c.getHeight() - 1);
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A label with a red rectangle drawn around it after the standard text painting.

---

### Real-World Cases

- **Themed labels** with custom text colors.
- **Branded UI** with decorative effects.
- **Accessibility** with high-contrast disabled text.

---

## 5. UI Delegate Best Practices

### Definitions

**Core Definition.** UI delegate best practices are the conventions and patterns that ensure custom and L&F delegates are correct, maintainable, and compatible with Swing's architecture.

**Technical Definition.** Best practices include: extending `BasicXxxUI` rather than implementing `ComponentUI` directly; using the `createUI` static factory pattern; making `installUI` and `uninstallUI` symmetric; calling `super` methods to preserve default behaviour; using `UIDefaults` for configurable values; respecting `UIResource` conventions; avoiding storing state in the delegate (the component is the state holder); and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that keep custom UI delegates working smoothly. They include things like "extend `BasicButtonUI`," "use `createUI`," "call `super`," and "don't store component state in the delegate."

---

### Purposes

- To minimize the code needed for custom UI delegates.
- To maintain compatibility with the L&F architecture.
- To avoid resource leaks from mismatched install/uninstall.
- To keep delegates stateless and reusable.
- To support runtime L&F switching and component updates.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyButtonUI extends BasicButtonUI {                      // 1. Extend
    public static ComponentUI createUI(JComponent c) {               // 2. Factory
        return new MyButtonUI();
    }

    @Override
    public void installUI(JComponent c) {
        super.installUI(c);                                          // 3. Super
        // additional setup
    }

    @Override
    public void uninstallUI(JComponent c) {
        // additional cleanup
        super.uninstallUI(c);                                        // 4. Super
    }

    @Override
    protected void paintText(Graphics g, AbstractButton b,
                             Rectangle textRect, String text) {
        super.paintText(g, b, textRect, text);                       // 5. Super
        // additional decoration
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend `BasicXxxUI` | `extends BasicButtonUI` |
| Use `createUI` factory | `public static ComponentUI createUI(...)` |
| Symmetric install/uninstall | `installUI` sets up; `uninstallUI` tears down |
| Call `super` methods | Preserve default behaviour |
| Use `UIDefaults` for values | `UIManager.getDefaults()` |
| Stateless delegates | The component holds state |
| Respect `UIResource` | Wrap defaults in `ColorUIResource`, etc. |
| EDT | Install and paint on the EDT |

#### Syntax Rules

- **Always extend `BasicXxxUI`** rather than implementing `ComponentUI` directly.
- **Always provide a `createUI` factory** so `UIManager` can instantiate the delegate.
- **Always make `installUI` and `uninstallUI` symmetric** to avoid leaks.
- **Always call `super`** to preserve default painting, sizing, and setup.
- **Never store state in the delegate**; the component is the state holder.
- **Always use `UIResource`** for values that the L&F may override.

#### Constraints and Limitations

- Custom delegates must be compatible with the component's model; otherwise, painting and input handling break.
- Replacing a delegate may break L&F-specific behaviours (e.g., custom borders).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Button UI

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;
import javax.swing.plaf.basic.BasicButtonUI;
import java.awt.*;

public class BestPracticeButtonUI extends BasicButtonUI {             // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new BestPracticeButtonUI();
    }

    @Override
    public void installUI(JComponent c) {
        super.installUI(c);                                          // 3. Super
        c.setOpaque(false);
    }

    @Override
    public void uninstallUI(JComponent c) {
        super.uninstallUI(c);                                        // 4. Super
    }

    @Override
    public void paint(Graphics g, JComponent c) {
        AbstractButton b = (AbstractButton) c;
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(b.isEnabled() ? new Color(0, 120, 215) : Color.GRAY);
            g2.fillRoundRect(0, 0, c.getWidth(), c.getHeight(), 12, 12); // 5. Background
        } finally {
            g2.dispose();
        }
        super.paint(g, c);                                            // 6. Default text
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Best Practice");
            button.setForeground(Color.WHITE);
            button.setUI(new BestPracticeButtonUI());
            JFrame frame = new JFrame("Best Practice UI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button with a rounded blue background and white text, drawn with anti-aliasing.

**Why the code produces this result.** The delegate overrides `paint` to draw a rounded background, then calls `super.paint` to paint the text and icon. `installUI` and `uninstallUI` call `super` for symmetric setup.

---

#### Example 2: Common Mistake — Implementing `ComponentUI` Directly

```java
public class BadUI implements ComponentUI {                            // WRONG
    // No createUI, no size methods, no install/uninstall
}
```

**Expected Output.** Components may fail to initialise, throw `NullPointerException` when sized, or exhibit incomplete behavior.

**Why this is wrong.** `ComponentUI` is an abstract class, not an interface; implementing it directly means missing the essential sizing and lifecycle methods. Extending `BasicButtonUI` (or `BasicLabelUI`) provides those implementations.

---

#### Example 3: Common Mistake — Storing State in the Delegate

```java
public class BadStateUI extends BasicButtonUI {
    private boolean pressed;                                          // WRONG
    @Override
    protected void paintButtonPressed(Graphics g, AbstractButton b) {
        pressed = true;                                              // Bad
        super.paintButtonPressed(g, b);
    }
}
```

**Expected Output.** Unpredictable behavior when the delegate is reused across components, and state leaks between paint cycles.

**Why this is wrong.** UI delegates should be stateless; the component (and its model) hold the state. Storing state in the delegate breaks sharing and reuse.

---

## References

Oracle. *Class ComponentUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/ComponentUI.html

Oracle. *Class BasicButtonUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/basic/BasicButtonUI.html

Oracle. *Class BasicLabelUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/basic/BasicLabelUI.html

Oracle. *Class ButtonUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/ButtonUI.html

Oracle. *Class LabelUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/LabelUI.html

Oracle. *Class JComponent — getUI and setUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *A Swing Architecture Overview*. https://www.oracle.com/technetwork/java/architecture-142923.html

Oracle. *How to Set the Look and Feel*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/lookandfeel/plaf.html

Oracle. *Class BasicLookAndFeel*. Java Platform, Standard Edition 27 API Specification. https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/plaf/basic/BasicLookAndFeel.html

Swing Chapter 21 — Pluggable Look & Feel. http://dev.cs.ovgu.de/java/Books/SwingBook/Chapter21html/index.html

CSDN. *Java外观皮肤包定制与UI美化实战*. https://blog.csdn.net/weixin_42355400/article/details/152283020

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/# UI Delegates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **UI Delegate** is the Swing object responsible for painting a component, responding to user interaction, and computing the component's minimum, preferred, and maximum sizes. Every Swing component—`JButton`, `JLabel`, `JTable`, `JTree`, and so on—delegates its visual appearance and much of its behaviour to a separate object called its **UI delegate** (formally, a subclass of `ComponentUI`). This separation is what enables Swing's pluggable look and feel.

**Technical Definition.** `public abstract class ComponentUI extends Object`. `ComponentUI` is the abstract base class for all UI delegates. It defines the contract that every delegate must implement: `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `update(Graphics, JComponent)`, `getPreferredSize(JComponent)`, `getMinimumSize(JComponent)`, `getMaximumSize(JComponent)`, `contains(JComponent, int, int)`, and the static `createUI(JComponent)` factory. Specialized delegate base classes extend `ComponentUI` for each component type (`ButtonUI`, `LabelUI`, `PanelUI`, `TableUI`, `TreeUI`, `ListUI`, `ScrollBarUI`, `SliderUI`, `SpinnerUI`, `TabbedPaneUI`, `ToolBarUI`, `RootPaneUI`, etc.). The `javax.swing.plaf.basic` package provides foundational implementations (`BasicButtonUI`, `BasicLabelUI`, `BasicPanelUI`, `BasicTableUI`, etc.) that other look and feels extend or replace. All UI delegate installation and painting must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** A **UI Delegate** is the "painter and behaviour manager" of a Swing component. The `JButton` object does not know how to draw its border, its text, or its pressed state; it asks its UI delegate to do that. The button is a thin shell that holds state and events; the delegate holds the drawing code and input handling. Different look and feels provide different delegates: a Metal button delegate looks different from a Nimbus or Windows one. `ComponentUI` is the base contract every delegate must satisfy. `BasicButtonUI` is the standard button painter. `BasicLabelUI` is the standard label painter. Think of it as the difference between a picture frame (the `JComponent`) and the painting inside it (the `ComponentUI`): the frame is always the same shape, but the painting can be swapped for a completely different style.

---

### Key Characteristics

1. **Separation of component and rendering.** The component holds state and API; the delegate holds painting and input-handling logic.
2. **Pluggable architecture.** Every L&F provides its own set of `ComponentUI` implementations; changing the L&F changes the delegates.
3. **One delegate per component.** Every `JComponent` has exactly one installed `ComponentUI`, accessed via `getUI()` and replaced via `setUI()`.
4. **Install/uninstall lifecycle.** `installUI` is called when the delegate is installed; `uninstallUI` is called when it is removed. This is where listeners, layout managers, and defaults are wired up and torn down.
5. **Painting delegation.** The component's `paintComponent` method calls the delegate's `paint` method; the delegate draws the component.
6. **Sizing delegation.** `getPreferredSize`, `getMinimumSize`, and `getMaximumSize` are delegated to the UI delegate.
7. **Hit-testing.** `contains` determines whether a point is inside the component; the delegate may override it for custom shapes.
8. **Static factory.** `createUI(JComponent)` returns a new delegate; this is how `UIManager` and components obtain delegates.
9. **Symmetry.** Every resource acquired in `installUI` must be released in `uninstallUI`.
10. **EDT discipline.** All delegate installation and painting must occur on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, anonymous classes.
- **Basic Swing:** `JComponent`, `JButton`, `JLabel`, and `SwingUtilities.invokeLater`.
- **Understanding of the `UIManager` and the pluggable L&F architecture.**
- **Knowledge of the `UIResource` convention.**
- **Basic Java 2D:** `Graphics2D`, `paintComponent`, and rendering hints.
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`javax.swing.plaf.ComponentUI`** — the base delegate class.
- **`javax.swing.plaf.basic.BasicButtonUI`** — the standard button delegate.
- **`javax.swing.plaf.basic.BasicLabelUI`** — the standard label delegate.
- **`javax.swing.plaf.basic.BasicPanelUI`** — the standard panel delegate.
- **`javax.swing.plaf.ButtonUI`, `LabelUI`, `PanelUI`, `TableUI`, etc.** — component-specific delegate interfaces.
- **`UIManager`** — installs delegates via `createUI` and maps `uiClassID` keys.
- **`LookAndFeel`** — supplies the delegate class mappings via `UIDefaults`.
- **`SwingUtilities.updateComponentTreeUI`** — reinstalls delegates after an L&F change.

---

## 1. UI Delegates — The Concept

### Definitions

**Core Definition.** A UI delegate is a `ComponentUI` implementation that performs painting, sizing, and event handling for a `JComponent`.

**Technical Definition.** The UI delegate is retrieved by `JComponent.getUI()` and installed by `JComponent.setUI(ComponentUI)`. The component delegates its `paintComponent`, `getPreferredSize`, `getMinimumSize`, `getMaximumSize`, and `contains` methods to the delegate. The delegate is created by the static `createUI(JComponent)` factory method and configured through `installUI` and `uninstallUI`.

**Beginner-Friendly Explanation.** A UI delegate is the component's painter and behavior handler. It knows how to draw the component and how to respond to input.

---

### Purposes

- To separate the component's API from its visual representation and behaviour.
- To enable pluggable L&Fs without changing component code.
- To allow multiple L&Fs to produce different appearances for the same component type.
- To allow custom delegates to be installed on individual components for specialised rendering.
- To centralise sizing, painting, and input handling in a single testable class.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JButton button = new JButton("Click");
ComponentUI delegate = button.getUI();                            // 1. Get
System.out.println(delegate.getClass().getName());                 // 2. Inspect

button.setUI(new MyButtonUI());                                    // 3. Replace
button.updateUI();                                                  // 4. Reset to L&F
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getUI()` | Returns the installed delegate. |
| `setUI(ComponentUI)` | Replaces the delegate. |
| `updateUI()` | Resets the delegate to the current L&F's delegate. |

#### Syntax Rules

- Every `JComponent` has a delegate after construction.
- `setUI` is normally followed by `revalidate()` and `repaint()`.
- `updateUI` is called automatically after an L&F change.
- All delegate access should occur on the EDT.

#### Constraints and Limitations

- Replacing a delegate without understanding the L&F's expectations can break behaviour.
- Custom delegates must implement the full `ComponentUI` contract.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting the UI Delegate

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;

public class InspectDelegate {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Inspect");                // 1. Create
            ComponentUI ui = button.getUI();                        // 2. Get
            System.out.println("Delegate class: " + ui.getClass().getName()); // 3. Inspect

            JFrame frame = new JFrame("Inspect Delegate");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints the fully qualified class name of the current `ButtonUI` delegate (e.g., `javax.swing.plaf.metal.MetalButtonUI`).

**Why the code produces this result.** `JButton.getUI()` returns the delegate installed by the current L&F.

---

### Real-World Cases

- **Debugging:** inspecting which delegate is active.
- **Custom components:** providing a custom delegate for specialised rendering.
- **L&F development:** implementing delegates for a new L&F.

---

## 2. ComponentUI — The Base Class

### Definitions

**Core Definition.** `ComponentUI` is the abstract base class for all UI delegates. It defines the methods that every delegate must implement.

**Technical Definition.** `public abstract class ComponentUI extends Object`. It declares `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `update(Graphics, JComponent)`, `getPreferredSize(JComponent)`, `getMinimumSize(JComponent)`, `getMaximumSize(JComponent)`, `contains(JComponent, int, int)`, and the static `createUI(JComponent)`. It also inherits `getAccessibleChildrenCount`, `getAccessibleChild`, and `getBaseline` methods (with default implementations).

**Beginner-Friendly Explanation.** `ComponentUI` is the base contract for every delegate. If you want to write your own delegate, you extend `ComponentUI` (or a subclass like `ButtonUI`) and implement its methods.

---

### Purposes

- To define the universal delegate contract.
- To provide default implementations for optional methods.
- To ensure every delegate can be installed, uninstalled, painted, and sized.
- To provide hit-testing through `contains`.
- To serve as the parent of component-specific delegate interfaces.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyUI extends ComponentUI {
    public static ComponentUI createUI(JComponent c) { return new MyUI(); }
    public void installUI(JComponent c)   { /* install listeners, layout */ }
    public void uninstallUI(JComponent c) { /* remove listeners */ }
    public void paint(Graphics g, JComponent c) { /* paint component */ }
    public void update(Graphics g, JComponent c) { paint(g, c); }
    public Dimension getPreferredSize(JComponent c) { /* ... */ }
    public Dimension getMinimumSize(JComponent c)   { /* ... */ }
    public Dimension getMaximumSize(JComponent c)   { /* ... */ }
    public boolean contains(JComponent c, int x, int y) {
        return x >= 0 && x < c.getWidth() && y >= 0 && y < c.getHeight();
    }
}
```

#### Component Breakdown

| Method | Responsibility |
|--------|----------------|
| `createUI(JComponent)` | Static factory; returns a new delegate instance. |
| `installUI(JComponent)` | Called when the delegate is installed; sets up state. |
| `uninstallUI(JComponent)` | Called when the delegate is removed; tears down state. |
| `paint(Graphics, JComponent)` | Paints the component. |
| `update(Graphics, JComponent)` | Clears and repaints; default calls `paint`. |
| `getPreferredSize(JComponent)` | Preferred size. |
| `getMinimumSize(JComponent)` | Minimum size. |
| `getMaximumSize(JComponent)` | Maximum size. |
| `contains(JComponent, int, int)` | Hit test. |

#### Syntax Rules

- Every delegate must provide `installUI`, `uninstallUI`, `paint`, and the size methods.
- `installUI` and `uninstallUI` must be symmetric.
- `createUI` should return a new instance each time (or a shared singleton for stateless delegates).

#### Constraints and Limitations

- `ComponentUI` does not enforce implementation of all methods, but subclasses must override the ones they need.
- Default implementations of size methods return `null`, so subclasses must override them.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Custom `ComponentUI`

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;
import java.awt.*;

public class DiagonalFillUI extends ComponentUI {                    // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new DiagonalFillUI();
    }

    @Override
    public void installUI(JComponent c) {                              // 3. Install
        c.setOpaque(true);
        c.setBackground(Color.LIGHT_GRAY);
    }

    @Override
    public void uninstallUI(JComponent c) {                            // 4. Uninstall
        // nothing to release
    }

    @Override
    public void paint(Graphics g, JComponent c) {                     // 5. Paint
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            int w = c.getWidth(), h = c.getHeight();
            g2.setColor(c.getBackground());
            g2.fillRect(0, 0, w, h);
            g2.setColor(Color.DARK_GRAY);
            for (int x = -h; x < w; x += 10) {
                g2.drawLine(x, 0, x + h, h);
            }
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Custom Painter");
            button.setUI(new DiagonalFillUI());                       // 6. Install
            JFrame frame = new JFrame("Custom UI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button rendered with a light-gray background crossed by diagonal dark-gray lines.

**Why the code produces this result.** `setUI` replaces the button's delegate with `DiagonalFillUI`, whose `paint` draws the diagonal pattern. The button's text is not painted because the delegate does not call `super.paint` or paint text.

---

### Real-World Cases

- **Custom components** that need specialised rendering.
- **Themed UI** elements that go beyond what L&F defaults provide.
- **Prototyping** new UI styles before packaging them as full L&Fs.

---

## 3. BasicButtonUI — The Standard Button Delegate

### Definitions

**Core Definition.** `BasicButtonUI` is the foundational `ButtonUI` implementation that provides standard painting and behaviour for all button subclasses (`JButton`, `JToggleButton`, `JCheckBox`, `JRadioButton`).

**Technical Definition.** `public class BasicButtonUI extends ButtonUI`. It provides `createUI(JComponent)`, `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `paintText(Graphics, AbstractButton, Rectangle, String)`, `paintIcon(Graphics, AbstractButton, Rectangle)`, `paintFocus(Graphics, AbstractButton, Rectangle, Rectangle)`, `paintButtonPressed(Graphics, AbstractButton)`, `getPreferredSize(JComponent)`, `getMinimumSize(JComponent)`, and `getMaximumSize(JComponent)`. It reads defaults from `UIDefaults` (text, font, icon, margins, etc.) and delegates to overridable paint methods.

**Beginner-Friendly Explanation.** `BasicButtonUI` is the standard button painter. It knows how to draw the text, icon, focus rectangle, and pressed state of any button. Most L&Fs extend `BasicButtonUI` (or one of its subclasses) and override the paint methods to add their own style.

---

### Purposes

- To provide a complete, working button delegate.
- To serve as the base class for L&F-specific button delegates.
- To define overridable paint stages (text, icon, focus, pressed).
- To read button properties from `UIDefaults` for L&F customisation.
- To provide standard sizing and hit-testing.

---

### Syntax Rules and Structure

#### Complete General Syntax (Extending `BasicButtonUI`)

```java
public class MyButtonUI extends BasicButtonUI {
    public static ComponentUI createUI(JComponent c) {
        return new MyButtonUI();
    }

    @Override
    protected void paintButtonPressed(Graphics g, AbstractButton b) {
        // custom pressed state painting
    }

    @Override
    protected void paintText(Graphics g, AbstractButton b,
                             Rectangle textRect, String text) {
        // custom text painting
    }

    @Override
    protected void paintFocus(Graphics g, AbstractButton b,
                              Rectangle viewRect, Rectangle textRect,
                              Rectangle iconRect) {
        // custom focus painting
    }
}
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `createUI(JComponent)` | Returns a new `BasicButtonUI` (or subclass). |
| `installUI(JComponent)` | Sets up listeners, defaults. |
| `uninstallUI(JComponent)` | Tears down listeners, defaults. |
| `paint(Graphics, JComponent)` | Orchestrates the painting stages. |
| `paintText(...)` | Paints button text. |
| `paintIcon(...)` | Paints button icon. |
| `paintFocus(...)` | Paints focus rectangle. |
| `paintButtonPressed(...)` | Paints pressed state. |
| `getPreferredSize(JComponent)` | Sizing based on text, icon, margins. |

#### Syntax Rules

- `BasicButtonUI` is abstract; `createUI` is the recommended factory.
- Subclasses should call `super` methods to preserve default behaviour.
- The `paint` method calls `paintButtonPressed` if the button is pressed, then `paintText`, then `paintFocus`.

#### Constraints and Limitations

- `BasicButtonUI` does not paint the button background by default; backgrounds are painted by the component or L&F-specific delegate.
- Custom delegates must handle the button's `ButtonModel` state (pressed, armed, rollover, selected).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: `BasicButtonUI` with Custom Text Painting

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;
import javax.swing.plaf.basic.BasicButtonUI;
import java.awt.*;

public class CustomButtonUI extends BasicButtonUI {                   // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new CustomButtonUI();
    }

    @Override
    protected void paintText(Graphics g, AbstractButton b,            // 3. Override
                             Rectangle textRect, String text) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING,
                                RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            g2.setColor(b.isEnabled() ? Color.WHITE : Color.GRAY);
            g2.setFont(b.getFont());
            g2.drawString(text, textRect.x,
                textRect.y + g2.getFontMetrics().getAscent());
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Custom Text");
            button.setBackground(new Color(0, 120, 215));             // 4. Custom
            button.setForeground(Color.WHITE);
            button.setUI(new CustomButtonUI());                        // 5. Install
            JFrame frame = new JFrame("BasicButtonUI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button with a blue background and white anti-aliased text "Custom Text".

**Why the code produces this result.** The custom delegate overrides `paintText` to draw the text with anti-aliasing and a custom colour. The button's background is painted by the component (since `BasicButtonUI` doesn't paint backgrounds for non-opaque buttons, but `setBackground` combined with `setOpaque(true)` may be needed depending on the L&F).

---

### Real-World Cases

- **Themed buttons** that need custom text rendering.
- **Icon buttons** that need custom icon positioning.
- **Branded UI** with custom pressed states.

---

## 4. BasicLabelUI — The Standard Label Delegate

### Definitions

**Core Definition.** `BasicLabelUI` is the foundational `LabelUI` implementation that provides standard painting and sizing for `JLabel`.

**Technical Definition.** `public class BasicLabelUI extends LabelUI implements PropertyChangeListener`. It provides `createUI(JComponent)`, `installUI(JComponent)`, `uninstallUI(JComponent)`, `paint(Graphics, JComponent)`, `layout(LabelUI, JLabel, FontMetrics, String, Icon, Rectangle, Rectangle, Rectangle)`, `paintEnabledText(JLabel, Graphics, String, int, int)`, `paintDisabledText(JLabel, Graphics, String, int, int)`, and sizing methods. It respects the label's text, icon, horizontal alignment, vertical alignment, and text position.

**Beginner-Friendly Explanation.** `BasicLabelUI` is the standard label painter. It knows how to lay out the label's text and icon and paint them in the correct positions.

---

### Purposes

- To provide a complete, working label delegate.
- To serve as the base class for L&F-specific label delegates.
- To lay out text and icon according to alignment and position properties.
- To paint enabled and disabled text with different colours.
- To read label properties from `UIDefaults` for L&F customisation.

---

### Syntax Rules and Structure

#### Complete General Syntax (Extending `BasicLabelUI`)

```java
public class MyLabelUI extends BasicLabelUI {
    public static ComponentUI createUI(JComponent c) {
        return new MyLabelUI();
    }

    @Override
    protected void paintEnabledText(JLabel l, Graphics g, String s,
                                    int textX, int textY) {
        // custom enabled text painting
    }

    @Override
    protected void paintDisabledText(JLabel l, Graphics g, String s,
                                     int textX, int textY) {
        // custom disabled text painting
    }

    @Override
    public void paint(Graphics g, JComponent c) {
        super.paint(g, c);
        // extra decorations
    }
}
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `createUI(JComponent)` | Returns a new `BasicLabelUI` (or subclass). |
| `installUI(JComponent)` | Sets up listeners. |
| `paint(Graphics, JComponent)` | Lays out and paints the text and icon. |
| `layout(...)` | Computes positions for text, icon, and focus rect. |
| `paintEnabledText(...)` | Paints text for enabled labels. |
| `paintDisabledText(...)` | Paints text for disabled labels. |
| `getPreferredSize(JComponent)` | Sizing based on text, icon, insets. |

#### Syntax Rules

- `BasicLabelUI` calls `layout` to compute positions, then paints the text and icon.
- Subclasses should call `super` to preserve default painting unless replacing it entirely.

#### Constraints and Limitations

- `BasicLabelUI` does not draw a border; the label's border is painted by the component's `paintBorder`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: `BasicLabelUI` with Custom Disabled Text

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;
import javax.swing.plaf.basic.BasicLabelUI;
import java.awt.*;

public class CustomLabelUI extends BasicLabelUI {                     // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new CustomLabelUI();
    }

    @Override
    protected void paintDisabledText(JLabel l, Graphics g, String s,  // 3. Override
                                     int textX, int textY) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.LIGHT_GRAY);
            g2.drawString(s, textX, textY);
        } finally {
            g2.dispose();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JLabel label = new JLabel("Disabled Label");
            label.setEnabled(false);                                    // 4. Disabled
            label.setUI(new CustomLabelUI());                           // 5. Install
            JFrame frame = new JFrame("BasicLabelUI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(label);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A disabled label with light-gray text instead of the default gray.

**Why the code produces this result.** The custom delegate overrides `paintDisabledText` to paint the text in light gray. All other behaviour (layout, sizing) is inherited from `BasicLabelUI`.

---

#### Example 2: `BasicLabelUI` with Additional Decoration

```java
public class DecoratedLabelUI extends BasicLabelUI {
    public static ComponentUI createUI(JComponent c) {
        return new DecoratedLabelUI();
    }

    @Override
    public void paint(Graphics g, JComponent c) {
        super.paint(g, c);                                            // 1. Default
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setColor(Color.RED);                                    // 2. Decor
            g2.drawRect(0, 0, c.getWidth() - 1, c.getHeight() - 1);
        } finally {
            g2.dispose();
        }
    }
}
```

**Expected Output.** A label with a red rectangle drawn around it after the standard text painting.

---

### Real-World Cases

- **Themed labels** with custom text colours.
- **Branded UI** with decorative effects.
- **Accessibility** with high-contrast disabled text.

---

## 5. UI Delegate Best Practices

### Definitions

**Core Definition.** UI delegate best practices are the conventions and patterns that ensure custom and L&F delegates are correct, maintainable, and compatible with Swing's architecture.

**Technical Definition.** Best practices include: extending `BasicXxxUI` rather than implementing `ComponentUI` directly; using the `createUI` static factory pattern; making `installUI` and `uninstallUI` symmetric; calling `super` methods to preserve default behaviour; using `UIDefaults` for configurable values; respecting `UIResource` conventions; avoiding storing state in the delegate (the component is the state holder); and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices are the rules of thumb that keep custom UI delegates working smoothly. They include things like "extend `BasicButtonUI`," "use `createUI`," "call `super`," and "don't store component state in the delegate."

---

### Purposes

- To minimise the code needed for custom UI delegates.
- To maintain compatibility with the L&F architecture.
- To avoid resource leaks from mismatched install/uninstall.
- To keep delegates stateless and reusable.
- To support runtime L&F switching and component updates.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyButtonUI extends BasicButtonUI {                      // 1. Extend
    public static ComponentUI createUI(JComponent c) {               // 2. Factory
        return new MyButtonUI();
    }

    @Override
    public void installUI(JComponent c) {
        super.installUI(c);                                          // 3. Super
        // additional setup
    }

    @Override
    public void uninstallUI(JComponent c) {
        // additional cleanup
        super.uninstallUI(c);                                        // 4. Super
    }

    @Override
    protected void paintText(Graphics g, AbstractButton b,
                             Rectangle textRect, String text) {
        super.paintText(g, b, textRect, text);                       // 5. Super
        // additional decoration
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend `BasicXxxUI` | `extends BasicButtonUI` |
| Use `createUI` factory | `public static ComponentUI createUI(...)` |
| Symmetric install/uninstall | `installUI` sets up; `uninstallUI` tears down |
| Call `super` methods | Preserve default behaviour |
| Use `UIDefaults` for values | `UIManager.getDefaults()` |
| Stateless delegates | The component holds state |
| Respect `UIResource` | Wrap defaults in `ColorUIResource`, etc. |
| EDT | Install and paint on the EDT |

#### Syntax Rules

- **Always extend `BasicXxxUI`** rather than implementing `ComponentUI` directly.
- **Always provide a `createUI` factory** so `UIManager` can instantiate the delegate.
- **Always make `installUI` and `uninstallUI` symmetric** to avoid leaks.
- **Always call `super`** to preserve default painting, sizing, and setup.
- **Never store state in the delegate**; the component is the state holder.
- **Always use `UIResource`** for values that the L&F may override.

#### Constraints and Limitations

- Custom delegates must be compatible with the component's model; otherwise, painting and input handling break.
- Replacing a delegate may break L&F-specific behaviours (e.g., custom borders).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Button UI

```java
import javax.swing.*;
import javax.swing.plaf.ComponentUI;
import javax.swing.plaf.basic.BasicButtonUI;
import java.awt.*;

public class BestPracticeButtonUI extends BasicButtonUI {             // 1. Extend
    public static ComponentUI createUI(JComponent c) {                // 2. Factory
        return new BestPracticeButtonUI();
    }

    @Override
    public void installUI(JComponent c) {
        super.installUI(c);                                          // 3. Super
        c.setOpaque(false);
    }

    @Override
    public void uninstallUI(JComponent c) {
        super.uninstallUI(c);                                        // 4. Super
    }

    @Override
    public void paint(Graphics g, JComponent c) {
        AbstractButton b = (AbstractButton) c;
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(b.isEnabled() ? new Color(0, 120, 215) : Color.GRAY);
            g2.fillRoundRect(0, 0, c.getWidth(), c.getHeight(), 12, 12); // 5. Background
        } finally {
            g2.dispose();
        }
        super.paint(g, c);                                            // 6. Default text
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Best Practice");
            button.setForeground(Color.WHITE);
            button.setUI(new BestPracticeButtonUI());
            JFrame frame = new JFrame("Best Practice UI");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button with a rounded blue background and white text, drawn with anti-aliasing.

**Why the code produces this result.** The delegate overrides `paint` to draw a rounded background, then calls `super.paint` to paint the text and icon. `installUI` and `uninstallUI` call `super` for symmetric setup.

---

#### Example 2: Common Mistake — Implementing `ComponentUI` Directly

```java
public class BadUI extends ComponentUI {                              // WRONG
    // No createUI, no size methods, no install/uninstall
}
```

**Expected Output.** Components may fail to initialise, throw `NullPointerException` when sized, or exhibit incomplete behaviour.

**Why this is wrong.** `ComponentUI` is an abstract class, not an interface; implementing it directly means missing the essential sizing and lifecycle methods. Extending `BasicButtonUI` (or `BasicLabelUI`) provides those implementations.

---

#### Example 3: Common Mistake — Storing State in the Delegate

```java
public class BadStateUI extends BasicButtonUI {
    private boolean pressed;                                          // WRONG
    @Override
    protected void paintButtonPressed(Graphics g, AbstractButton b) {
        pressed = true;                                              // Bad
        super.paintButtonPressed(g, b);
    }
}
```

**Expected Output.** Unpredictable behaviour when the delegate is reused across components, and state leaks between paint cycles.

**Why this is wrong.** UI delegates should be stateless; the component (and its model) hold the state. Storing state in the delegate breaks sharing and reuse.

---

## References

Oracle. *Class ComponentUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/ComponentUI.html

Oracle. *Class BasicButtonUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/basic/BasicButtonUI.html

Oracle. *Class BasicLabelUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/basic/BasicLabelUI.html

Oracle. *Class ButtonUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/ButtonUI.html

Oracle. *Class LabelUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/LabelUI.html

Oracle. *Class BasicPanelUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/basic/BasicPanelUI.html

Oracle. *Class JComponent — getUI and setUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *A Swing Architecture Overview*. https://www.oracle.com/technetwork/java/architecture-142923.html

Oracle. *How to Set the Look and Feel*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/lookandfeel/plaf.html

Oracle. *Class BasicLookAndFeel*. Java Platform, Standard Edition 27 API Specification. https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/plaf/basic/BasicLookAndFeel.html

Swing Chapter 21 — Pluggable Look & Feel. http://dev.cs.ovgu.de/java/Books/SwingBook/Chapter21html/index.html

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/