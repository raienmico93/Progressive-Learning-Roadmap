# BorderLayout — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `BorderLayout` is a layout manager that arranges a container's components into five distinct regions: **north**, **south**, **east**, **west**, and **center**. Each region can contain at most one component. When the container is resized, the center region absorbs all extra space, while the other regions expand only as much as necessary to accommodate their components.

**Technical Definition.** `public class BorderLayout extends Object implements LayoutManager2, Serializable`. It is the default layout manager for every content pane. Components are positioned using the constants `NORTH`, `SOUTH`, `EAST`, `WEST`, and `CENTER`. The `NORTH` and `SOUTH` components may be stretched horizontally; `EAST` and `WEST` components may be stretched vertically; the `CENTER` component stretches in both directions to fill any remaining space. `BorderLayout` also supports orientation‑relative constants (`PAGE_START`, `PAGE_END`, `LINE_START`, `LINE_END`) that map to the absolute regions based on the container's `ComponentOrientation`.

**Beginner‑Friendly Explanation.** `BorderLayout` is like a picture frame with five slots: a banner at the top (north), a footer at the bottom (south), a sidebar on the right (east), a sidebar on the left (west), and a large canvas in the middle (center). You can place one component in each slot. When you enlarge the window, the canvas in the middle grows to fill the extra space, while the banner, footer, and sidebars stay at their preferred sizes. This makes `BorderLayout` ideal for the main window of an application: a toolbar at the top, a status bar at the bottom, navigation on the left, details on the right, and the primary content in the center.

---

### Key Characteristics

1. **Five regions, one component each.** `BorderLayout` can hold at most five components—one per region. Adding a second component to a region replaces the first one.
2. **Centre gets the extra space.** When the container grows, the center region expands to fill all available space; the other regions keep their preferred sizes.
3. **Default for content panes.** Every `JFrame`, `JDialog`, and `JApplet` content pane is initialised with `BorderLayout`.
4. **Orientation‑aware constants.** Modern code should use `PAGE_START`, `PAGE_END`, `LINE_START`, and `LINE_END` instead of `NORTH`, `SOUTH`, `WEST`, and `EAST` to support right‑to‑left languages.
5. **Gaps control spacing.** The constructor `BorderLayout(int hgap, int vgap)` and the methods `setHgap`/`setVgap` add horizontal and vertical spacing between regions.
6. **Toolbar docking requires `BorderLayout`.** For `JToolBar` drag‑and‑drop docking to work, the tool bar must be placed in a `BorderLayout` container, and the other edge positions should be empty.
7. **Not for vertical writing modes.** `BorderLayout` does not currently support vertical orientations; the `isVertical` property of `ComponentOrientation` is not respected.
8. **Single‑thread rule.** Layout construction and modification should occur on the Event Dispatch Thread (EDT).

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, and lambda expressions.
- Familiarity with `Container`, `Component`, `JPanel`, and `JFrame`.
- Understanding of `getPreferredSize` and component size hints.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `ComponentOrientation` for internationalisation.
- A JDK (version 8 or later; `BorderLayout` is in `java.awt`).

---

### Related Programming Areas

- **`LayoutManager2`:** The interface `BorderLayout` implements; adds constraint‑aware methods such as `addLayoutComponent(Component, Object)`.
- **`FlowLayout`, `GridLayout`, `BoxLayout`:** Alternative managers commonly nested inside `BorderLayout` regions.
- **`JToolBar`:** Requires `BorderLayout` for drag‑out docking.
- **`ComponentOrientation`:** Resolves `LINE_START`/`LINE_END` constants.
- **`BorderFactory`:** Used to add padding inside `BorderLayout` regions.
- **`pack()` and `revalidate()`:** Essential for triggering layout after building or modifying the hierarchy.

---

## Core Concepts and Features

---

## 1. BorderLayout — The Class

### Definitions

**Core Definition.** `BorderLayout` is a concrete `LayoutManager2` that arranges components into five fixed regions.

**Technical Definition.** `public class BorderLayout extends Object implements LayoutManager2, Serializable`. It provides constructors accepting horizontal and vertical gaps. The five region constants are `NORTH`, `SOUTH`, `EAST`, `WEST`, and `CENTER`; the orientation‑relative constants `PAGE_START`, `PAGE_END`, `LINE_START`, and `LINE_END` are preferred for internationalisation.

**Beginner‑Friendly Explanation.** `BorderLayout` is the class you use to divide a window into five major areas.

---

### Purposes

- To structure a window into five major regions (top, bottom, left, right, centre).
- To provide a default layout for content panes that is immediately usable.
- To allow a central content area to absorb resizing while edges remain fixed.
- To support tool bar docking and status‑bar placement.
- To serve as the outer skeleton into which other layout managers are nested.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
BorderLayout layout = new BorderLayout();                          // no gaps
BorderLayout layout = new BorderLayout(int hgap, int vgap);        // with gaps
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `BorderLayout()` | Creates a border layout with no gaps between components. |
| `BorderLayout(int hgap, int vgap)` | Creates a border layout with the specified horizontal and vertical gaps. |

#### Syntax Rules

- Components are added with `add(component, BorderLayout.NORTH)` etc.
- The `CENTER` region is the default if no constraint is supplied.
- Only one component per region; adding another replaces the previous one.
- Construction and modification should occur on the EDT.

#### Constraints and Limitations

- `BorderLayout` cannot hold more than five components directly; nest a `JPanel` to add more.
- `BorderLayout` does not support vertical writing modes.
- The relative constants (`PAGE_START`, etc.) take precedence if both absolute and relative constants are used.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal BorderLayout with Five Buttons

```java
import javax.swing.*;
import java.awt.*;

public class MinimalBorderLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal BorderLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            frame.setLayout(new BorderLayout());                    // 1. Layout
            frame.add(new JButton("North"), BorderLayout.NORTH);    // 2. Regions
            frame.add(new JButton("South"), BorderLayout.SOUTH);
            frame.add(new JButton("East"),  BorderLayout.EAST);
            frame.add(new JButton("West"),  BorderLayout.WEST);
            frame.add(new JButton("Center"), BorderLayout.CENTER);

            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with five buttons placed in the north, south, east, west, and centre regions. The centre button expands to fill the remaining space when the window is resized.

**Why the code produces this result.** The `BorderLayout` places each button in its assigned region. The centre button receives all extra space, while the edge buttons keep their preferred sizes.

---

### Real-World Cases

- **Main application windows** with a tool bar at the top, a status bar at the bottom, navigation on the left, and content in the centre.
- **Dialogs** with a form in the centre and OK/Cancel buttons at the bottom.

---

## 2. Border Layout Creation

### Definitions

**Core Definition.** Border layout creation is the process of instantiating a `BorderLayout` with optional gaps and installing it on a container.

**Technical Definition.** `BorderLayout` provides two constructors: a no‑argument constructor with zero gaps and a constructor accepting `hgap` and `vgap`. The layout can be installed at container construction or later via `setLayout`.

**Beginner‑Friendly Explanation.** Creating a `BorderLayout` means deciding whether there should be spacing between the five regions.

---

### Purposes

- To instantiate the layout manager with appropriate gap settings.
- To install the manager on a content pane or panel.
- To prepare the container for components to be added to the five regions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JPanel panel = new JPanel(new BorderLayout(10, 10));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int hgap` | Horizontal gap between regions. |
| `int vgap` | Vertical gap between regions. |

#### Syntax Rules

- Gaps may be zero or positive.
- The layout manager is installed on the container via the constructor or `setLayout`.

#### Constraints and Limitations

- Default gaps are zero; for padding around the container, use `BorderFactory.createEmptyBorder`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating BorderLayout with Gaps

```java
import javax.swing.*;
import java.awt.*;

public class GappedBorderLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Gapped BorderLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new BorderLayout(15, 10));    // 1. Gaps
            panel.add(new JButton("North"), BorderLayout.NORTH);
            panel.add(new JButton("Center"), BorderLayout.CENTER);
            panel.add(new JButton("South"), BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A panel with 15‑pixel horizontal gaps and 10‑pixel vertical gaps between the north, centre, and south buttons.

**Why the code produces this result.** The `BorderLayout` constructor sets the gaps, which are applied between the regions.

---

### Real-World Cases

- **Spacious dashboards** with comfortable spacing between panels.
- **Compact tool windows** with zero gaps.

---

## 3. Border Layout Regions — NORTH

### Definitions

**Core Definition.** `BorderLayout.NORTH` places a component at the top of the container, stretched horizontally to the full width.

**Technical Definition.** `public static final String NORTH = "North"`. The component in this region is stretched horizontally to the container's width; its height is its preferred height.

**Beginner‑Friendly Explanation.** `NORTH` is the top slot—perfect for tool bars, headers, or menu‑like panels.

---

### Purposes

- To place a tool bar or header at the top of the window.
- To provide a banner or title area.
- To stretch a component across the full width.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
container.add(component, BorderLayout.NORTH);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `NORTH` | Top region; stretched horizontally. |

#### Syntax Rules

- Only one component can occupy the `NORTH` region.
- The component's preferred height is respected.

#### Constraints and Limitations

- If no component is in `NORTH`, the `EAST`, `CENTER`, and `WEST` regions expand vertically to fill the space.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Tool Bar in NORTH

```java
import javax.swing.*;
import java.awt.*;

public class NorthRegion {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("NORTH Region");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new BorderLayout());

            JToolBar toolBar = new JToolBar();                        // 1. Tool bar
            toolBar.add(new JButton("New"));
            toolBar.add(new JButton("Open"));
            toolBar.add(new JButton("Save"));

            frame.add(toolBar, BorderLayout.NORTH);                  // 2. NORTH
            frame.add(new JTextArea(), BorderLayout.CENTER);

            frame.setSize(500, 350);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A tool bar at the top of the window, with a text area filling the rest.

**Why the code produces this result.** `NORTH` stretches the tool bar across the full width; the centre text area absorbs the remaining space.

---

### Real-World Cases

- **Tool bars** and **menu bars** in application windows.
- **Header panels** with titles or logos.

---

## 4. Border Layout Regions — SOUTH

### Definitions

**Core Definition.** `BorderLayout.SOUTH` places a component at the bottom of the container, stretched horizontally to the full width.

**Technical Definition.** `public static final String SOUTH = "South"`. The component is stretched horizontally; its height is its preferred height.

**Beginner‑Friendly Explanation.** `SOUTH` is the bottom slot—ideal for status bars or button rows.

---

### Purposes

- To place a status bar or footer at the bottom.
- To provide a button row (OK/Cancel) at the bottom of a dialog.
- To stretch a component across the full width.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
container.add(component, BorderLayout.SOUTH);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `SOUTH` | Bottom region; stretched horizontally. |

#### Syntax Rules

- Only one component can occupy the `SOUTH` region.

#### Constraints and Limitations

- If no component is in `SOUTH`, the centre region expands vertically.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Status Bar in SOUTH

```java
import javax.swing.*;
import java.awt.*;

public class SouthRegion {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SOUTH Region");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new BorderLayout());

            JLabel status = new JLabel("Ready", SwingConstants.CENTER);
            status.setBorder(BorderFactory.createEtchedBorder());

            frame.add(status, BorderLayout.SOUTH);                   // 1. SOUTH
            frame.add(new JTextArea(), BorderLayout.CENTER);

            frame.setSize(500, 350);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A status bar at the bottom of the window, with a text area filling the rest.

**Why the code produces this result.** `SOUTH` anchors the status label to the bottom edge and stretches it horizontally.

---

### Real-World Cases

- **Status bars** showing application state.
- **Dialog button bars** at the bottom.

---

## 5. Border Layout Regions — EAST

### Definitions

**Core Definition.** `BorderLayout.EAST` places a component on the right side of the container, stretched vertically to the full height.

**Technical Definition.** `public static final String EAST = "East"`. The component is stretched vertically; its width is its preferred width.

**Beginner‑Friendly Explanation.** `EAST` is the right sidebar—useful for property panels or navigation on the right.

---

### Purposes

- To place a property inspector or details panel on the right.
- To provide a right‑hand navigation or tool palette.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
container.add(component, BorderLayout.EAST);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `EAST` | Right region; stretched vertically. |

#### Syntax Rules

- Only one component can occupy the `EAST` region.

#### Constraints and Limitations

- The `EAST` component is stretched vertically but not horizontally.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Property Panel in EAST

```java
import javax.swing.*;
import java.awt.*;

public class EastRegion {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("EAST Region");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new BorderLayout());

            JPanel properties = new JPanel();                        // 1. Right panel
            properties.setBorder(BorderFactory.createTitledBorder("Properties"));
            properties.setPreferredSize(new Dimension(200, 0));
            properties.add(new JLabel("Width:"));
            properties.add(new JTextField(8));

            frame.add(properties, BorderLayout.EAST);                // 2. EAST
            frame.add(new JTextArea(), BorderLayout.CENTER);

            frame.setSize(600, 400);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A property panel on the right side of the window, with a text area filling the centre.

**Why the code produces this result.** `EAST` anchors the property panel to the right edge and stretches it vertically.

---

### Real-World Cases

- **Property inspectors** in IDEs and design tools.
- **Right‑hand navigation panels**.

---

## 6. Border Layout Regions — WEST

### Definitions

**Core Definition.** `BorderLayout.WEST` places a component on the left side of the container, stretched vertically to the full height.

**Technical Definition.** `public static final String WEST = "West"`. The component is stretched vertically; its width is its preferred width.

**Beginner‑Friendly Explanation.** `WEST` is the left sidebar—ideal for navigation trees, lists, or tool palettes.

---

### Purposes

- To place a navigation tree or list on the left.
- To provide a left‑hand tool palette.
- To separate navigation from content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
container.add(component, BorderLayout.WEST);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `WEST` | Left region; stretched vertically. |

#### Syntax Rules

- Only one component can occupy the `WEST` region.

#### Constraints and Limitations

- The `WEST` component is stretched vertically but not horizontally.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Navigation Tree in WEST

```java
import javax.swing.*;
import java.awt.*;

public class WestRegion {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("WEST Region");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new BorderLayout());

            JTree tree = new JTree();                                // 1. Navigation
            JScrollPane treeScroll = new JScrollPane(tree);
            treeScroll.setPreferredSize(new Dimension(200, 0));

            frame.add(treeScroll, BorderLayout.WEST);                // 2. WEST
            frame.add(new JTextArea(), BorderLayout.CENTER);

            frame.setSize(600, 400);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A navigation tree on the left side, with a text area filling the centre.

**Why the code produces this result.** `WEST` anchors the tree scroll pane to the left edge and stretches it vertically.

---

### Real-World Cases

- **File explorers** with a directory tree on the left.
- **Mail clients** with a folder list on the left.

---

## 7. Border Layout Regions — CENTER

### Definitions

**Core Definition.** `BorderLayout.CENTER` places a component in the middle of the container. This component receives all extra space when the container is resized.

**Technical Definition.** `public static final String CENTER = "Center"`. The component in this region stretches both horizontally and vertically to fill any space left over after the other regions are laid out.

**Beginner‑Friendly Explanation.** `CENTER` is the main content area—it grows to fill whatever space is left after the top, bottom, left, and right regions have taken their preferred sizes.

---

### Purposes

- To provide the primary content area of the window.
- To absorb all resizing behaviour.
- To be the default region when no constraint is specified.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
container.add(component, BorderLayout.CENTER);
// or simply:
container.add(component); // defaults to CENTER
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `CENTER` | Middle region; stretches in both directions. |

#### Syntax Rules

- Only one component can occupy the `CENTER` region.
- If no constraint is provided, the component is placed in `CENTER`.

#### Constraints and Limitations

- The centre component receives all extra space; if it has a large minimum size, it may force the container to grow.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Main Content in CENTER

```java
import javax.swing.*;
import java.awt.*;

public class CenterRegion {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("CENTER Region");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new BorderLayout());

            JTextArea content = new JTextArea("Main content area");
            JScrollPane scroll = new JScrollPane(content);

            frame.add(new JLabel("Header"), BorderLayout.NORTH);
            frame.add(scroll, BorderLayout.CENTER);                  // 1. CENTER

            frame.setSize(500, 350);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A header at the top and a scrollable text area filling the rest of the window.

**Why the code produces this result.** `CENTER` stretches the scroll pane to fill all remaining space after the `NORTH` region takes its preferred height.

---

### Real-World Cases

- **Document editors** with the editing area in the centre.
- **Dashboards** with the main chart or table in the centre.

---

## 8. Border Layout Gaps

### Definitions

**Core Definition.** Gaps are the horizontal and vertical spacing between the five regions of a `BorderLayout`.

**Technical Definition.** Gaps are specified via the constructor `BorderLayout(int hgap, int vgap)` or the methods `setHgap(int)` and `setVgap(int)`. The horizontal gap is inserted between the centre and the west/east regions, and between the north/south regions and the centre; the vertical gap is inserted between the north/south regions and the centre, and between the centre and the west/east regions.

**Beginner‑Friendly Explanation.** Gaps control how much empty space separates the five regions—useful for making the layout feel less cramped.

---

### Purposes

- To add visual breathing room between regions.
- To create consistent spacing without nesting extra panels.
- To improve readability and aesthetics.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
new BorderLayout(int hgap, int vgap);
layout.setHgap(10);
layout.setVgap(10);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `BorderLayout(int hgap, int vgap)` | Sets both gaps at construction. |
| `setHgap(int)` | Sets the horizontal gap. |
| `setVgap(int)` | Sets the vertical gap. |

#### Syntax Rules

- Gaps are specified in pixels.
- The default is zero.

#### Constraints and Limitations

- Gaps apply between regions but not around the outer edges of the container; use `BorderFactory.createEmptyBorder` for outer padding.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Gaps Between All Regions

```java
import javax.swing.*;
import java.awt.*;

public class GapsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Gaps Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new BorderLayout(10, 10));    // 1. Gaps
            panel.add(new JButton("North"), BorderLayout.NORTH);
            panel.add(new JButton("South"), BorderLayout.SOUTH);
            panel.add(new JButton("East"),  BorderLayout.EAST);
            panel.add(new JButton("West"),  BorderLayout.WEST);
            panel.add(new JButton("Center"), BorderLayout.CENTER);

            frame.add(panel);
            frame.setSize(500, 400);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Five buttons with 10‑pixel horizontal and vertical gaps between all regions.

**Why the code produces this result.** The `BorderLayout(10, 10)` constructor sets both gaps, which are applied between the regions.

---

### Real-World Cases

- **Spacious dashboards** with comfortable spacing.
- **Tool windows** with zero gaps for a compact look.

---

## 9. BorderLayout Best Practices

### Definitions

**Core Definition.** `BorderLayout` best practices are the conventions and patterns that ensure border‑layout‑based GUIs are portable, maintainable, and visually correct.

**Technical Definition.** Best practices include: using the orientation‑relative constants (`PAGE_START`, `PAGE_END`, `LINE_START`, `LINE_END`) instead of the absolute constants for internationalisation; using `BorderFactory.createEmptyBorder` for outer padding rather than relying on gaps; nesting panels with other layout managers inside `BorderLayout` regions; and ensuring that only one component is added per region.

**Beginner‑Friendly Explanation.** Best practices for `BorderLayout` are the “rules of thumb” that prevent misaligned regions, locale bugs, and cramped layouts. They include things like “use `LINE_START`/`LINE_END` instead of `WEST`/`EAST`,” “add padding with borders,” and “nest other layouts inside the five regions.”

---

### Purposes

- To support internationalisation through orientation‑aware constants.
- To provide consistent outer padding via borders.
- To enable complex layouts through nesting.
- To avoid replacing components accidentally by adding multiple components to the same region.

---

### Syntax Rules and Structure

#### Complete General Best‑Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JPanel main = new JPanel(new BorderLayout(5, 5));              // 1. Layout
    main.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10)); // 2. Padding

    JPanel form = new JPanel(new GridBagLayout());                 // 3. Nested
    // ... add form fields ...
    main.add(form, BorderLayout.CENTER);

    JPanel buttons = new JPanel(new FlowLayout(FlowLayout.TRAILING)); // 4. Nested
    buttons.add(new JButton("OK"));
    buttons.add(new JButton("Cancel"));
    main.add(buttons, BorderLayout.PAGE_END);                      // 5. Relative
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Relative constants | `PAGE_START`, `PAGE_END`, `LINE_START`, `LINE_END` |
| Outer padding | `BorderFactory.createEmptyBorder(...)` |
| Nesting | Add `JPanel`s with other managers to regions |
| One component per region | Use a container to group multiple components |
| Gaps for spacing between regions | `new BorderLayout(hgap, vgap)` |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always use the relative constants** (`PAGE_START`, `PAGE_END`, `LINE_START`, `LINE_END`) instead of the absolute compass constants for internationalisation.
- **Always add outer padding** with `BorderFactory.createEmptyBorder` rather than relying on gaps, which only affect spacing between regions.
- **Always nest other layout managers** inside `BorderLayout` regions to build complex interfaces.
- **Never add more than one component** to the same region; the last one added replaces the previous one.

#### Constraints and Limitations

- `BorderLayout` does not support vertical writing modes.
- Mixing absolute and relative constants can lead to unpredictable results; the relative constants take precedence.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best‑Practice Application Window

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeBorderLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeBorderLayout::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice BorderLayout");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JPanel main = new JPanel(new BorderLayout(5, 5));            // 1. Layout
        main.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10)); // 2. Padding

        JToolBar toolBar = new JToolBar();                            // 3. NORTH
        toolBar.add(new JButton("New"));
        toolBar.add(new JButton("Open"));
        main.add(toolBar, BorderLayout.PAGE_START);                   // 4. Relative

        JTree tree = new JTree();                                     // 5. WEST
        JScrollPane treeScroll = new JScrollPane(tree);
        treeScroll.setPreferredSize(new Dimension(200, 0));
        main.add(treeScroll, BorderLayout.LINE_START);                // 6. Relative

        JTextArea content = new JTextArea();                          // 7. CENTER
        main.add(new JScrollPane(content), BorderLayout.CENTER);

        JLabel status = new JLabel("Ready");                          // 8. SOUTH
        main.add(status, BorderLayout.PAGE_END);                     // 9. Relative

        frame.add(main);
        frame.setSize(800, 600);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A well‑structured application window with a tool bar at the top, a navigation tree on the left, a text area in the centre, a status bar at the bottom, and 10‑pixel outer padding. The layout uses relative constants, so it adapts correctly to right‑to‑left locales.

**Why the code produces this result.** The `BorderLayout` provides the five‑region skeleton, the relative constants ensure internationalisation, the empty border provides outer padding, and the nested components (tool bar, tree, text area, status bar) occupy the appropriate regions.

---

#### Example 2: Common Mistake — Using Absolute Constants for Internationalisation

```java
import javax.swing.*;
import java.awt.*;

public class BadConstants {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Constants");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new BorderLayout());

            frame.add(new JButton("Navigation"), BorderLayout.WEST); // WRONG: absolute
            frame.add(new JButton("Details"), BorderLayout.EAST);   // WRONG: absolute
            frame.add(new JButton("Content"), BorderLayout.CENTER);

            frame.setSize(500, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The layout works in left‑to‑right locales but places navigation on the left and details on the right regardless of the locale. In right‑to‑left locales, the navigation should appear on the right, but it remains on the left because absolute constants are used.

**Why this is wrong.** The absolute constants (`WEST`, `EAST`) do not respect component orientation. The relative constants (`LINE_START`, `LINE_END`) map to the correct edges based on orientation, ensuring proper behaviour in all locales. The correct approach is to use the relative constants for any layout that may be internationalised.

---

### Real-World Cases

- **Production Swing applications** use `BorderLayout` as the outer skeleton, nest other managers inside it, and use relative constants for internationalisation.
- **Cross‑platform applications** rely on `BorderLayout` to handle resizing predictably across different screen sizes and font metrics.
- **Educational examples** use the best‑practice skeleton to demonstrate correct `BorderLayout` usage from the start.

---

## References

Oracle. *How to Use BorderLayout*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/border.html

Oracle. *Class BorderLayout*. Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/BorderLayout.html

Oracle. *A Visual Guide to Layout Managers*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/visual.html

Oracle. *Using Layout Managers*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/using.html

Stack Overflow. *BorderLayout gaps and best practices*. https://stackoverflow.com/questions/ (discussion of `hgap`/`vgap` and layout nesting)

Oracle. *Class ComponentOrientation*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/ComponentOrientation.html