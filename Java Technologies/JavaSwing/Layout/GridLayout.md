# GridLayout — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `GridLayout` is a layout manager that arranges a container's components into a rectangular grid of equally-sized cells. Every cell has exactly the same width and height, and every component is stretched to completely fill its cell. The grid is defined by a number of rows and columns, and components are placed into the cells in left‑to‑right, top‑to‑bottom order.

**Technical Definition.** `public class GridLayout extends Object implements LayoutManager, Serializable`. `GridLayout` makes a bunch of components equal in size and displays them in the requested number of rows and columns . The number of rows and columns is specified at construction; if the container has more components than cells, the grid expands automatically to accommodate them (usually by adding columns). Horizontal and vertical gaps between cells are configurable via `hgap` and `vgap`. `GridLayout` is not orientation‑aware; it does not respect right‑to‑left component orientation for the order of components (though the look and feel may mirror painting). Swing layout construction should occur on the Event Dispatch Thread (EDT).

**Beginner‑Friendly Explanation.** `GridLayout` is the "spreadsheet" layout manager. You tell it how many rows and columns you want, and it carves the container into a grid of identical rectangular cells. Each component you add fills the next empty cell, and every component is stretched to fill its cell completely. Think of a calculator keypad: every button is the same size, arranged in a neat grid. GridLayout is perfect when you want uniform‑sized components arranged in a rectangle, and it is unsuitable when components need different sizes or when you want to align things precisely without stretching. Every cell is the same size—no exceptions—so if you want a tall button next to a short one, GridLayout cannot do it.

---

### Key Characteristics

1. **Uniform cell sizes.** Every cell in the grid has exactly the same width and height. The container's available space is divided equally among all rows and columns.
2. **Components fill their cells.** Each component is stretched to occupy its entire cell. There is no concept of preferred size influencing the final layout (the preferred size is used only to compute the container's preferred size, not to constrain the component).
3. **Row/column specification.** You specify the number of rows and columns; if you specify zero for one dimension, the other dimension determines the cell count. `GridLayout(0, 3)` means "any number of rows, but exactly 3 columns."
4. **Automatic cell growth.** If more components are added than there are cells, the grid grows (typically by adding rows or columns depending on which dimension was fixed).
5. **Configurable gaps.** `hgap` and `vgap` control spacing between cells; the default is zero (cells touch).
6. **No spanning.** Unlike `GridBagLayout`, `GridLayout` does not support components spanning multiple cells.
7. **Not ideal for forms.** Because every component is stretched to fill its cell, labels stretch alongside text fields, producing an unpleasant appearance. Use `GridBagLayout` or nested `BorderLayout` for forms.
8. **Single‑thread rule and simple implementation.** `GridLayout` is simple to use but requires discipline on the EDT for construction and modification.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, and lambda expressions.
- Familiarity with `Container`, `Component`, `JPanel`, and `JFrame`.
- Understanding of `getPreferredSize`, `getMinimumSize`, and `getMaximumSize`.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `BorderFactory` for outer padding (gaps only affect internal spacing).
- A JDK (version 8 or later; `GridLayout` is in `java.awt`).

---

### Related Programming Areas

- **`LayoutManager`:** The parent interface that `GridLayout` implements.
- **`GridBagLayout`:** The flexible, constraint‑based alternative when components need different sizes or spanning.
- **`BorderLayout`, `FlowLayout`, `BoxLayout`:** Common companions for nesting.
- **`Component.getPreferredSize`:** Used for computing the container's preferred size, not for constraining components.
- **`BorderFactory`:** For padding around the entire grid.
- **`JPanel` and `JFrame`:** Typical containers that use `GridLayout`.
- **GUI builders:** Generate `GridLayout` for uniform button rows or keypads.

---

## Core Concepts and Features

---

## 1. GridLayout — The Class

### Definitions

**Core Definition.** `GridLayout` is a layout manager that arranges components into a rectangular grid of equal‑sized cells.

**Technical Definition.** `public class GridLayout extends Object implements LayoutManager, Serializable`. It provides constructors that accept rows and columns, and optionally horizontal and vertical gaps. It implements the five `LayoutManager` methods: `addLayoutComponent`, `removeLayoutComponent`, `preferredLayoutSize`, `minimumLayoutSize`, and `layoutContainer`.

**Beginner‑Friendly Explanation.** `GridLayout` is the class you use when you want a uniform grid of equally‑sized components.

---

### Purposes

- To arrange components in a uniform grid of equal‑sized cells.
- To build calculator keypads, key panels, and uniform button arrays.
- To provide an even distribution of space among components.
- To serve as an inner layout for sections of a larger `BorderLayout`‑based GUI.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
GridLayout grid = new GridLayout();                                  // 1 row, 0 cols (dynamic)
GridLayout grid = new GridLayout(int rows, int cols);                // specified grid
GridLayout grid = new GridLayout(int rows, int cols, int hgap, int vgap); // with gaps
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `GridLayout()` | Creates a grid layout with a default of one row per component and one column. |
| `GridLayout(int rows, int cols)` | Creates a grid layout with the specified number of rows and columns. |
| `GridLayout(int rows, int cols, int hgap, int vgap)` | Creates a grid layout with the specified rows, columns, and gaps. |

#### Syntax Rules

- `rows` and `cols` may be zero, but not both. A zero dimension is filled dynamically as components are added.
- `hgap` and `vgap` may be zero or positive.
- Construction and modification should occur on the EDT.

#### Constraints and Limitations

- All cells are the same size; components are stretched to fill their cells.
- No component spanning; each component occupies exactly one cell.
- The grid size may change after construction if more components are added than cells available.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal GridLayout with Six Buttons

```java
import javax.swing.*;
import java.awt.*;

public class MinimalGridLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal GridLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(2, 3, 5, 5));  // 1. Grid
            for (int i = 1; i <= 6; i++) {
                panel.add(new JButton("Button " + i));              // 2. Add
            }

            frame.add(panel);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with six equal‑sized buttons arranged in a 2×3 grid with 5‑pixel horizontal and vertical gaps.

**Why the code produces this result.** The `GridLayout(2, 3, 5, 5)` constructor carves the panel into two rows and three columns with 5‑pixel gaps. Each button is stretched to fill its cell.

---

### Real-World Cases

- **Calculator keypads** with uniform button sizes.
- **Telephone keypads** with 3×4 grids.
- **Keyboard‑like button panels** for games or kiosks.

---

## 2. Grid Layout Creation

### Definitions

**Core Definition.** Grid layout creation is the process of instantiating a `GridLayout` with the desired rows, columns, and gaps, and installing it on a container.

**Technical Definition.** `GridLayout` provides three constructors: no‑argument (one row per component), rows/columns, and rows/columns/gaps. The layout can be installed on a container at construction or via `setLayout`.

**Beginner‑Friendly Explanation.** Creating a `GridLayout` means deciding the shape of the grid and the spacing between cells.

---

### Purposes

- To instantiate the layout manager with the desired grid shape.
- To configure gaps between cells.
- To install the manager on a panel or content pane.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JPanel panel = new JPanel(new GridLayout(3, 4, 10, 10));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int rows` | Number of rows; may be zero for dynamic. |
| `int cols` | Number of columns; may be zero for dynamic. |
| `int hgap` | Horizontal gap between cells. |
| `int vgap` | Vertical gap between cells. |

#### Syntax Rules

- At least one of `rows` or `cols` must be positive.
- Gaps may be zero or positive.

#### Constraints and Limitations

- The grid shape may change if more components are added than cells.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating with Dynamic Rows

```java
import javax.swing.*;
import java.awt.*;

public class DynamicRows {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Dynamic Rows");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(0, 3, 5, 5));  // 1. Fixed cols
            for (int i = 1; i <= 9; i++) {
                panel.add(new JButton("B" + i));                    // 2. Add
            }

            frame.add(panel);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Nine buttons arranged in three columns and three rows.

**Why the code produces this result.** `GridLayout(0, 3, 5, 5)` means "any number of rows, exactly 3 columns," producing 3 rows for 9 components.

---

### Real-World Cases

- **Dynamic grids** where the number of rows depends on the number of items.
- **Fixed‑column layouts** for uniform panels.

---

## 3. Grid Layout Properties — Rows

### Definitions

**Core Definition.** The `rows` property specifies the number of rows in the grid.

**Technical Definition.** `rows` is set at construction and read via `getRows()` and `setRows(int rows)`. A value of zero means the number of rows is determined dynamically from the columns and the number of components.

**Beginner‑Friendly Explanation.** `rows` is how many horizontal layers of cells the grid has.

---

### Purposes

- To define the vertical dimension of the grid.
- To allow dynamic row growth when set to zero.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
grid.setRows(4);
int current = grid.getRows();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setRows(int)` | Sets the row count. |
| `getRows()` | Returns the row count. |

#### Syntax Rules

- Zero means dynamic rows.
- Changing rows may re‑compute the layout.

#### Constraints and Limitations

- Setting both rows and columns to zero throws `IllegalArgumentException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Changing Row Count

```java
import javax.swing.*;
import java.awt.*;

public class RowsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Rows Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            GridLayout grid = new GridLayout(1, 6, 5, 5);
            JPanel panel = new JPanel(grid);
            for (int i = 1; i <= 6; i++) panel.add(new JButton("" + i));

            JButton change = new JButton("Use 3 Rows");
            change.addActionListener(e -> {
                grid.setRows(3);                                    // 1. New rows
                grid.setColumns(2);                                 // 2. New cols
                panel.revalidate();
            });

            frame.add(panel, BorderLayout.CENTER);
            frame.add(change, BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Initially a single row of six buttons; clicking the button re-arranges them into a 3×2 grid.

**Why the code produces this result.** `setRows` and `setColumns` update the grid dimensions; `revalidate()` triggers the new layout.

---

### Real-World Cases

- **Adaptive grids** that change shape based on user preference.

---

## 4. Grid Layout Properties — Columns

### Definitions

**Core Definition.** The `cols` property specifies the number of columns in the grid.

**Technical Definition.** `cols` is set at construction and read via `getColumns()` and `setColumns(int cols)`. A value of zero means the number of columns is determined dynamically.

**Beginner‑Friendly Explanation.** `cols` is how many vertical layers of cells the grid has.

---

### Purposes

- To define the horizontal dimension of the grid.
- To allow dynamic column growth when set to zero.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
grid.setColumns(5);
int current = grid.getColumns();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setColumns(int)` | Sets the column count. |
| `getColumns()` | Returns the column count. |

#### Syntax Rules

- Zero means dynamic columns.

#### Constraints and Limitations

- Setting both to zero throws `IllegalArgumentException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Fixed Columns with Dynamic Rows

```java
import javax.swing.*;
import java.awt.*;

public class ColumnsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Columns Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(0, 4, 5, 5));  // 1. Fixed cols
            for (int i = 1; i <= 12; i++) panel.add(new JButton("" + i));

            frame.add(panel);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Twelve buttons arranged in four columns and three rows.

**Why the code produces this result.** `GridLayout(0, 4, 5, 5)` fixes four columns and lets rows grow to fit all components.

---

### Real-World Cases

- **Grid‑based forms** where the number of columns is fixed.
- **Icon grids** where columns are aligned with a grid of thumbnails.

---

## 5. Grid Layout Properties — Horizontal Gap

### Definitions

**Core Definition.** The `hgap` property specifies the horizontal distance in pixels between adjacent columns.

**Technical Definition.** `hgap` is set at construction and read via `getHgap()` and `setHgap(int hgap)`. The default is zero.

**Beginner‑Friendly Explanation.** `hgap` controls the space between columns.

---

### Purposes

- To control horizontal spacing between cells.
- To create tighter or looser grids.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
grid.setHgap(10);
int current = grid.getHgap();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setHgap(int)` | Sets the horizontal gap. |
| `getHgap()` | Returns the horizontal gap. |

#### Syntax Rules

- Must be zero or positive.

#### Constraints and Limitations

- `hgap` applies between cells and to the container's left and right edges.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Wide Horizontal Gaps

```java
import javax.swing.*;
import java.awt.*;

public class HGapDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("hgap Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(2, 3, 20, 5)); // 1. hgap=20
            for (int i = 1; i <= 6; i++) panel.add(new JButton("" + i));

            frame.add(panel);
            frame.setSize(500, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Six buttons with 20 pixels of horizontal spacing between columns.

**Why the code produces this result.** The constructor sets `hgap = 20`, spacing cells and container edges horizontally.

---

### Real-World Cases

- **Spacious button grids** for touch UIs.
- **Compact grids** with zero gaps.

---

## 6. Grid Layout Properties — Vertical Gap

### Definitions

**Core Definition.** The `vgap` property specifies the vertical distance in pixels between adjacent rows.

**Technical Definition.** `vgap` is set at construction and read via `getVgap()` and `setVgap(int vgap)`. The default is zero.

**Beginner‑Friendly Explanation.** `vgap` controls the space between rows.

---

### Purposes

- To control vertical spacing between cells.
- To create tighter or looser grids.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
grid.setVgap(10);
int current = grid.getVgap();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setVgap(int)` | Sets the vertical gap. |
| `getVgap()` | Returns the vertical gap. |

#### Syntax Rules

- Must be zero or positive.

#### Constraints and Limitations

- `vgap` applies between cells and to the container's top and bottom edges.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Wide Vertical Gaps

```java
import javax.swing.*;
import java.awt.*;

public class VGapDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("vgap Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(3, 2, 5, 20)); // 1. vgap=20
            for (int i = 1; i <= 6; i++) panel.add(new JButton("" + i));

            frame.add(panel);
            frame.setSize(300, 350);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Six buttons with 20 pixels of vertical spacing between rows.

**Why the code produces this result.** The constructor sets `vgap = 20`, spacing rows and top/bottom edges vertically.

---

### Real-World Cases

- **Tall keypads** needing comfortable row spacing.
- **Compact grids** with zero vertical gaps.

---

## 7. GridLayout Best Practices

### Definitions

**Core Definition.** `GridLayout` best practices are the conventions and patterns that ensure grid layouts are used effectively, accessibly, and appropriately in Swing applications.

**Technical Definition.** Best practices include: using `GridLayout` for uniform, equal‑sized components; not using it for forms with different component sizes; nesting `GridLayout` inside `BorderLayout` or `BoxLayout` for structured GUIs; using non‑zero gaps for readability; using `BorderFactory.createEmptyBorder` for outer padding; and choosing `GridBagLayout` when components need different sizes or spanning.

**Beginner‑Friendly Explanation.** Best practices for `GridLayout` are the "rules of thumb" that prevent stretched labels, awkward forms, and stretched buttons that look unnatural. They include things like "use `GridLayout` for uniform grids," "don't use it for forms," and "add gaps for readability."

---

### Purposes

- To ensure grids are used where all cells should be the same size.
- To avoid stretched labels in forms by using `GridBagLayout`.
- To provide readable spacing through gaps and borders.
- To nest `GridLayout` inside larger layouts for structured GUIs.

---

### Syntax Rules and Structure

#### Complete General Best‑Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JPanel grid = new JPanel(new GridLayout(3, 3, 5, 5));         // 1. Grid
    grid.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10)); // 2. Padding
    for (int i = 0; i < 9; i++) {
        grid.add(new JButton("" + (i + 1)));
    }
    main.add(grid, BorderLayout.CENTER);                          // 3. Nest
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use for uniform components | `new GridLayout(rows, cols)` |
| Add gaps for readability | `new GridLayout(rows, cols, 5, 5)` |
| Outer padding via borders | `BorderFactory.createEmptyBorder(...)` |
| Avoid for forms | Use `GridBagLayout` or `GroupLayout`. |
| Nest inside structured layouts | `main.add(grid, BorderLayout.CENTER)` |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Use `GridLayout` for uniform grids** (keypads, button arrays, icon grids), not for forms.
- **Always add gaps** (`hgap`, `vgap`) for readability unless a fully touching grid is desired.
- **Always add outer padding** with `BorderFactory.createEmptyBorder`; gaps do not pad the container's outer edges by themselves.
- **Never use `GridLayout` for forms with labels and fields**, as both will be stretched equally; use `GridBagLayout` or nested `BorderLayout`.
- **Always nest `GridLayout` inside larger layouts** when building a structured GUI.

#### Constraints and Limitations

- All cells are forced to the same size; components cannot span multiple cells.
- Does not respect preferred component sizes for final placement—only for computing the container's preferred size.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best‑Practice Calculator Keypad

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeGrid {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeGrid::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice GridLayout");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JPanel keypad = new JPanel(new GridLayout(4, 4, 5, 5));      // 1. Grid
        keypad.setBorder(
            BorderFactory.createEmptyBorder(10, 10, 10, 10));        // 2. Padding

        String[] keys = {
            "7", "8", "9", "/",
            "4", "5", "6", "*",
            "1", "2", "3", "-",
            "0", ".", "=", "+"
        };
        for (String key : keys) {
            keypad.add(new JButton(key));                            // 3. Fill
        }

        JPanel main = new JPanel(new BorderLayout());
        main.add(new JTextField(), BorderLayout.NORTH);              // 4. Display
        main.add(keypad, BorderLayout.CENTER);                       // 5. Nest

        frame.add(main);
        frame.setSize(300, 400);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A calculator keypad with 16 equal‑sized buttons in a 4×4 grid, 5‑pixel gaps, and 10‑pixel outer padding, with a text field above.

**Why the code produces this result.** `GridLayout(4, 4, 5, 5)` produces a uniform grid. The empty border pads the whole keypad, and the `BorderLayout` nests the display above the keypad.

---

#### Example 2: Common Mistake — Using GridLayout for a Form

```java
import javax.swing.*;
import java.awt.*;

public class BadFormLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Form");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel form = new JPanel(new GridLayout(3, 2, 5, 5));    // WRONG
            form.add(new JLabel("Name:"));
            form.add(new JTextField(15));
            form.add(new JLabel("Email:"));
            form.add(new JTextField(15));
            form.add(new JLabel("Phone:"));
            form.add(new JTextField(15));

            frame.add(form);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Labels and text fields arranged in a grid, but with each label stretched to fill its cell—the labels look unnaturally wide, and the fields are compressed to match the labels' cell width.

**Why this is wrong.** `GridLayout` forces every cell to be the same size, so a short label gets stretched to match the width of a text field. This produces visually unbalanced forms. Use `GridBagLayout` or nested `BorderLayout`/`BoxLayout` panels for forms instead.

---

### Real-World Cases

- **Production Swing applications** use `GridLayout` only for uniform grids (keypads, icon grids, button arrays) and nest it inside `BorderLayout` or `BoxLayout` for structured GUIs.
- **Calculator and keypad interfaces** are the canonical `GridLayout` use case.
- **Educational examples** use the best‑practice skeleton to demonstrate correct `GridLayout` usage from the start.

---

## References

Oracle. *How to Use GridLayout*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/grid.html

Oracle. *Class GridLayout*. Java® Platform, Standard Edition & Java Development Kit Version 24 API Specification. https://docs.oracle.com/en/java/javase/24/docs/api/java.desktop/java/awt/GridLayout.html

Oracle. *A Visual Guide to Layout Managers*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/visual.html

Oracle. *Using Layout Managers*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/using.html

Oracle. *Class GridLayout*. Java Platform, Standard Edition 17 API Specification. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/java/awt/GridLayout.html

GNU Classpath. *Class GridLayout*. https://developer.classpath.org/sinjdoc/java/awt/GridLayout.html

Stack Overflow. *GridLayout for forms — why GridBagLayout is preferred*. https://browse.library.kiwix.org/ (community consensus on layout choices)

Stack Overflow. *GridLayout rows and columns with zero*. https://stackoverflow.com/questions/ (discussion of `GridLayout(0, n)` semantics)