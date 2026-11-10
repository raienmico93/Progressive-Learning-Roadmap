# Custom Layout Managers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **custom layout manager** is a user-written class that implements `java.awt.LayoutManager` (or its subinterface `java.awt.LayoutManager2`) and is installed on a `Container` via `setLayout`. It replaces the arrangement algorithm of the standard managers with one tailored to a specific layout pattern, such as a circular arrangement, a diagonal stack, a percentage-based grid, or a domain-specific panel arrangement.

**Technical Definition.** `LayoutManager` declares five methods that every layout manager must implement: `addLayoutComponent(String, Component)`, `removeLayoutComponent(Component)`, `preferredLayoutSize(Container)`, `minimumLayoutSize(Container)`, and `layoutContainer(Container)`. `LayoutManager2` extends `LayoutManager` and adds constraint-aware methods: `addLayoutComponent(Component, Object)`, `maximumLayoutSize(Container)`, `getLayoutAlignmentX(Container)`, `getLayoutAlignmentY(Container)`, and `invalidateLayout(Container)`. A container calls these methods to compute its own preferred, minimum, and maximum sizes, and then calls `layoutContainer` to position and size each visible child. A well-written custom layout manager must respect the container's `Insets`, honour each component's size hints, and be safe under the container's tree lock.

**Beginner-Friendly Explanation.** A custom layout manager is a "write-your-own-arranger" for Swing. Instead of using the built-in managers (`BorderLayout`, `GridBagLayout`, `BoxLayout`, and so on), you write a small class that contains the exact algorithm you want. Perhaps your application needs a circular clock face, a diagonal card stack, a percentage-based dashboard, or a Treemap layout. Writing a custom manager lets you encode that arrangement once and reuse it anywhere. You implement a small number of methods: "what size would you like to be?", "what is your smallest acceptable size?", and "please lay these components out." The container calls your methods whenever it needs to lay itself out, and you do the rest.

---

### Key Characteristics

1. **Two interfaces.** `LayoutManager` is for simple layouts with no per-component constraints. `LayoutManager2` is for layouts where the programmer supplies a constraint object (or a `String` key) for each component, and where the manager needs to report a maximum size and an alignment.
2. **Five required methods (seven for `LayoutManager2`).** Every custom manager must implement `preferredLayoutSize`, `minimumLayoutSize`, and `layoutContainer`, plus the two component-registration methods; `LayoutManager2` adds `maximumLayoutSize`, `getLayoutAlignmentX`, `getLayoutAlignmentY`, and `invalidateLayout`.
3. **The container drives the process.** The layout manager is passive. The container calls `preferredLayoutSize`/`minimumLayoutSize` when it needs to know its preferred size (e.g., during `pack()`), and it calls `layoutContainer` whenever the container's bounds are set or the layout is invalidated.
4. **Tree-lock discipline.** `Container.getTreeLock()` must be held during any iteration of the container's component list. Standard layout managers synchronise on this lock; custom managers should too, to avoid `ConcurrentModificationException` or inconsistent layouts.
5. **Insets and gaps.** Custom managers are responsible for honouring the container's `Insets` (padding) and for any gaps they introduce between components.
6. **Size hints, not commands.** A custom manager reads `getPreferredSize`, `getMinimumSize`, and `getMaximumSize` from each child, but is free to ignore them. Well-behaved managers respect these hints.
7. **Alignment properties.** `getLayoutAlignmentX` and `getLayoutAlignmentY` are used when the container is itself a child of a `BoxLayout`. Most custom managers return `0.5f` (centre) for both.
8. **Testing is essential.** Custom layout managers are hard to validate visually; unit tests that check `preferredLayoutSize` and `layoutContainer` results are recommended.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, lambda expressions, and collections.
- Familiarity with `Container`, `Component`, `JPanel`, and `JFrame`.
- Understanding of `Insets`, `Dimension`, `getPreferredSize`, `getMinimumSize`, and `getMaximumSize`.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `getTreeLock()` and `synchronized` blocks.
- Familiarity with the standard layout managers (`BorderLayout`, `BoxLayout`, `GridBagLayout`, `FlowLayout`, `GridLayout`) as reference implementations.
- A JDK (version 8 or later; `LayoutManager` and `LayoutManager2` are in `java.awt`).

---

### Related Programming Areas

- **`java.awt.LayoutManager` and `java.awt.LayoutManager2`:** The interfaces custom managers implement.
- **`java.awt.Container`:** The class that calls layout-manager methods; provides `getTreeLock()`, `getInsets()`, `getComponents()`, and `add`/`remove`.
- **`java.awt.Insets` and `java.awt.Dimension`:** The sizing types a custom manager works with.
- **`javax.swing.BoxLayout`:** A good reference implementation of a non-constraint layout manager.
- **`javax.swing.GroupLayout` and `java.awt.GridBagLayout`:** Good reference implementations of constraint-based managers.
- **`javax.swing.LayoutStyle`:** Provides platform-preferred gap values that a custom manager can adopt.
- **`SwingUtilities.invokeLater`:** Ensures layout construction and modification happen on the EDT.

---

## Core Concepts and Features

---

## 1. Custom Layout Managers — The Concept

### Definitions

**Core Definition.** A custom layout manager is a class you write that implements `LayoutManager` or `LayoutManager2` and encodes an arrangement algorithm you need but that Swing does not already provide.

**Technical Definition.** Any class implementing `java.awt.LayoutManager` can be installed on a `Container` via `setLayout`. The container calls the manager's methods whenever it needs to compute its preferred/minimum/maximum size or to lay out its children. The manager must read each child's size hints, honour the container's insets, and position children using `Component.setBounds`. `LayoutManager2` should be used when per-component constraints are needed.

**Beginner-Friendly Explanation.** A custom layout manager is a small class you write to encode exactly how you want your components arranged, then hand to your panel via `setLayout`.

---

### Purposes

- To encode an arrangement algorithm that is not provided by any standard manager (e.g., circular, diagonal, percentage-based, treemap).
- To provide a cleaner API than `GridBagLayout` for a domain-specific layout.
- To reuse the same arrangement logic across multiple panels.
- To encapsulate layout logic in a testable, maintainable class.
- To teach the layout-manager contract by implementing it directly.

---

### Syntax Rules and Structure

#### Complete General Syntax (Installing a Custom Layout)

```java
JPanel panel = new JPanel();
panel.setLayout(new MyCustomLayout());
// or with constraints:
panel.add(component, myConstraints);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new MyCustomLayout()` | Creates an instance of the custom manager. |
| `panel.setLayout(layout)` | Installs the manager on the container. |
| `panel.add(component)` | Adds a component; the manager will see it during layout. |

#### Syntax Rules

- The manager is installed with `setLayout`; the container retains a reference.
- The manager's `layoutContainer` method is called when the container is validated.
- Layout construction and installation should occur on the EDT.

#### Constraints and Limitations

- A custom manager is called frequently during layout; avoid heavy work in `preferredLayoutSize`/`layoutContainer`.
- The manager must synchronise on `getTreeLock()` while iterating the container's components.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Custom Layout Manager — A Row Layout

```java
import java.awt.*;
import javax.swing.*;

// A simple custom LayoutManager that places components in one row.
public class RowLayout implements LayoutManager {                    // 1. Implement
    private final int gap;

    public RowLayout(int gap) { this.gap = gap; }

    public void addLayoutComponent(String name, Component comp) {}    // 2. Required
    public void removeLayoutComponent(Component comp) {}

    public Dimension preferredLayoutSize(Container parent) {          // 3. Preferred
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int w = 0, h = 0, n = parent.getComponentCount();
            for (int i = 0; i < n; i++) {
                Component c = parent.getComponent(i);
                if (!c.isVisible()) continue;
                Dimension d = c.getPreferredSize();
                w += d.width + (i > 0 ? gap : 0);
                h = Math.max(h, d.height);
            }
            return new Dimension(in.left + in.right + w, in.top + in.bottom + h);
        }
    }

    public Dimension minimumLayoutSize(Container parent) {            // 4. Minimum
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int w = 0, h = 0, n = parent.getComponentCount();
            for (int i = 0; i < n; i++) {
                Component c = parent.getComponent(i);
                if (!c.isVisible()) continue;
                Dimension d = c.getMinimumSize();
                w += d.width + (i > 0 ? gap : 0);
                h = Math.max(h, d.height);
            }
            return new Dimension(in.left + in.right + w, in.top + in.bottom + h);
        }
    }

    public void layoutContainer(Container parent) {                    // 5. Layout
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int x = in.left;
            int y = in.top;
            int h = parent.getHeight() - in.top - in.bottom;
            int n = parent.getComponentCount();
            for (int i = 0; i < n; i++) {
                Component c = parent.getComponent(i);
                if (!c.isVisible()) continue;
                Dimension d = c.getPreferredSize();
                c.setBounds(x, y, d.width, h);
                x += d.width + gap;
            }
        }
    }
}

// Usage:
public class RowLayoutDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom RowLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JPanel panel = new JPanel(new RowLayout(10));
            panel.add(new JButton("One"));
            panel.add(new JButton("Two"));
            panel.add(new JButton("Three"));
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with three buttons arranged in a single horizontal row, each at its preferred width and stretched to the container's full height, separated by 10-pixel gaps.

**Why the code produces this result.** `preferredLayoutSize` sums the children's preferred widths plus gaps and adds the container's insets. `layoutContainer` sets each child's bounds sequentially, honouring the insets and the vertical stretch. The tree lock prevents concurrent modification during iteration.

---

### Real-World Cases

- **Circular toolbars** in media or drawing applications.
- **Percentage-based dashboards** where each tile takes a fraction of the container.
- **Treemap layouts** for visualizing hierarchical data.
- **Custom card layouts** with specific transitions and alignments.

---

## 2. LayoutManager — The Basic Interface

### Definitions

**Core Definition.** `LayoutManager` is the interface that defines the contract for layout managers. It declares five methods that every layout manager must implement.

**Technical Definition.** `public interface LayoutManager`. The five methods are: `void addLayoutComponent(String name, Component comp)`, `void removeLayoutComponent(Component comp)`, `Dimension preferredLayoutSize(Container parent)`, `Dimension minimumLayoutSize(Container parent)`, and `void layoutContainer(Container parent)`. The string name in `addLayoutComponent` is the constraints object (often `null`).

**Beginner-Friendly Explanation.** `LayoutManager` is the basic contract. You implement five methods and your class becomes a layout manager.

---

### Purposes

- To define a simple, constraint-free layout manager contract.
- To allow custom arrangements without per-component constraint objects.
- To serve as the base interface for `LayoutManager2`.

---

### Syntax Rules and Structure

#### Complete General Syntax (Implementation)

```java
public class MyLayout implements LayoutManager {
    public void addLayoutComponent(String name, Component comp) {}
    public void removeLayoutComponent(Component comp) {}
    public Dimension preferredLayoutSize(Container parent) { ... }
    public Dimension minimumLayoutSize(Container parent) { ... }
    public void layoutContainer(Container parent) { ... }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addLayoutComponent(String, Component)` | Called when a component is added; the string is the constraints (often ignored). |
| `removeLayoutComponent(Component)` | Called when a component is removed; usually a no-op. |
| `preferredLayoutSize(Container)` | Returns the container's preferred size given its children. |
| `minimumLayoutSize(Container)` | Returns the container's minimum size. |
| `layoutContainer(Container)` | Positions and sizes the container's children. |

#### Syntax Rules

- All five methods must be implemented.
- `preferredLayoutSize` and `minimumLayoutSize` must synchronise on `parent.getTreeLock()`.
- `layoutContainer` must also synchronise on `parent.getTreeLock()`.
- The manager must honour the container's `Insets`.

#### Constraints and Limitations

- `LayoutManager` has no maximum-size or alignment support; use `LayoutManager2` when those are needed.
- The `String` constraint in `addLayoutComponent` is limited to a single key; use `LayoutManager2` for richer constraint objects.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Vertical Stack LayoutManager

```java
import java.awt.*;
import javax.swing.*;

public class VStackLayout implements LayoutManager {                   // 1. Implement
    private final int gap;
    public VStackLayout(int gap) { this.gap = gap; }

    public void addLayoutComponent(String name, Component comp) {}
    public void removeLayoutComponent(Component comp) {}

    public Dimension preferredLayoutSize(Container parent) {           // 2. Preferred
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int w = 0, h = 0, n = parent.getComponentCount();
            for (int i = 0; i < n; i++) {
                Component c = parent.getComponent(i);
                if (!c.isVisible()) continue;
                Dimension d = c.getPreferredSize();
                w = Math.max(w, d.width);
                h += d.height + (i > 0 ? gap : 0);
            }
            return new Dimension(in.left + in.right + w, in.top + in.bottom + h);
        }
    }

    public Dimension minimumLayoutSize(Container parent) {             // 3. Minimum
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int w = 0, h = 0, n = parent.getComponentCount();
            for (int i = 0; i < n; i++) {
                Component c = parent.getComponent(i);
                if (!c.isVisible()) continue;
                Dimension d = c.getMinimumSize();
                w = Math.max(w, d.width);
                h += d.height + (i > 0 ? gap : 0);
            }
            return new Dimension(in.left + in.right + w, in.top + in.bottom + h);
        }
    }

    public void layoutContainer(Container parent) {                    // 4. Layout
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int x = in.left;
            int y = in.top;
            int w = parent.getWidth() - in.left - in.right;
            int n = parent.getComponentCount();
            for (int i = 0; i < n; i++) {
                Component c = parent.getComponent(i);
                if (!c.isVisible()) continue;
                Dimension d = c.getPreferredSize();
                c.setBounds(x, y, w, d.height);
                y += d.height + gap;
            }
        }
    }
}

public class VStackDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom VStack");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JPanel panel = new JPanel(new VStackLayout(8));
            panel.add(new JLabel("Name:"));
            panel.add(new JTextField(15));
            panel.add(new JLabel("Email:"));
            panel.add(new JTextField(15));
            panel.add(new JButton("Submit"));
            frame.add(panel);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with five components stacked vertically, each stretched to the full width of the panel, separated by 8-pixel gaps.

**Why the code produces this result.** `preferredLayoutSize` computes the maximum width and sum of heights; `layoutContainer` positions each component at the current `y`, stretches it to the panel width, and advances `y` by the component's preferred height plus the gap.

---

### Real-World Cases

- **Sidebars** with a vertical list of buttons or labels.
- **Log panels** that stack message components.
- **Custom toolbars** without wrapping.

---

## 3. LayoutManager2 — The Constraint-Aware Interface

### Definitions

**Core Definition.** `LayoutManager2` is a subinterface of `LayoutManager` that adds support for constraint objects, maximum-size reporting, alignment reporting, and explicit invalidation. It is the interface used by `BorderLayout`, `CardLayout`, `GridBagLayout`, `BoxLayout`, `GroupLayout`, `SpringLayout`, and `OverlayLayout`.

**Technical Definition.** `public interface LayoutManager2 extends LayoutManager`. It adds: `void addLayoutComponent(Component comp, Object constraints)`, `Dimension maximumLayoutSize(Container target)`, `float getLayoutAlignmentX(Container target)`, `float getLayoutAlignmentY(Container target)`, and `void invalidateLayout(Container target)`. The `addLayoutComponent(Component, Object)` method replaces (or complements) the `String`-keyed overload; the constraint object is implementation-specific.

**Beginner-Friendly Explanation.** `LayoutManager2` is the "constraint-aware" contract. It lets you define a per-component constraint object (like `GridBagConstraints`) and gives the container a maximum size and alignment so it can behave correctly when nested inside other layouts.

---

### Purposes

- To support per-component constraint objects (e.g., a `PercentConstraints` class for a percentage layout).
- To report the container's maximum size to the parent layout.
- To report alignment so the container behaves correctly inside a `BoxLayout`.
- To allow the container to invalidate the layout when constraints change.

---

### Syntax Rules and Structure

#### Complete General Syntax (Implementation)

```java
public class MyLayout2 implements LayoutManager2 {
    private final Map<Component, Object> constraints = new HashMap<>();

    public void addLayoutComponent(Component comp, Object c) {
        constraints.put(comp, c);
    }
    public void addLayoutComponent(String name, Component comp) {
        constraints.put(comp, null);
    }
    public void removeLayoutComponent(Component comp) {
        constraints.remove(comp);
    }
    public Dimension preferredLayoutSize(Container parent) { ... }
    public Dimension minimumLayoutSize(Container parent) { ... }
    public Dimension maximumLayoutSize(Container target) { ... }
    public float getLayoutAlignmentX(Container target) { return 0.5f; }
    public float getLayoutAlignmentY(Container target) { return 0.5f; }
    public void invalidateLayout(Container target) {}
    public void layoutContainer(Container parent) { ... }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addLayoutComponent(Component, Object)` | Called when a component is added with a constraint object. |
| `maximumLayoutSize(Container)` | Returns the container's maximum size. |
| `getLayoutAlignmentX(Container)` | Returns the horizontal alignment (0.0–1.0). |
| `getLayoutAlignmentY(Container)` | Returns the vertical alignment (0.0–1.0). |
| `invalidateLayout(Container)` | Called when the layout is invalidated; usually a no-op or cache clear. |

#### Syntax Rules

- All methods of `LayoutManager` must still be implemented (inherited by the interface).
- The constraint object is typically stored in a `Map<Component, Object>` keyed by the component.
- `maximumLayoutSize` should return a meaningful maximum or `new Dimension(Integer.MAX_VALUE, Integer.MAX_VALUE)` if unbounded.
- Alignment values should be in `[0.0, 1.0]`; `0.5f` means centre.

#### Constraints and Limitations

- A component may be added with either the `String` or `Component, Object` overload; the manager must handle both.
- The constraint object is not automatically validated; the manager must check its type.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Percentage-Based LayoutManager2

```java
import java.awt.*;
import java.util.HashMap;
import java.util.Map;
import javax.swing.*;

public class PercentLayout implements LayoutManager2 {                 // 1. Impl
    // Public constraint class for callers.
    public static final class Percent {
        public final double x, y, w, h;
        public Percent(double x, double y, double w, double h) {
            this.x = x; this.y = y; this.w = w; this.h = h;
        }
    }

    private final Map<Component, Percent> map = new HashMap<>();

    public void addLayoutComponent(Component comp, Object c) {          // 2. Constraint
        if (!(c instanceof Percent))
            throw new IllegalArgumentException("Need Percent constraint");
        map.put(comp, (Percent) c);
    }
    public void addLayoutComponent(String name, Component comp) {       // 3. String key
        map.put(comp, new Percent(0, 0, 1, 1));
    }
    public void removeLayoutComponent(Component comp) {
        map.remove(comp);
    }
    public Dimension preferredLayoutSize(Container parent) {            // 4. Pref
        return parent.getSize();
    }
    public Dimension minimumLayoutSize(Container parent) {
        return new Dimension(0, 0);
    }
    public Dimension maximumLayoutSize(Container target) {              // 5. Max
        return new Dimension(Integer.MAX_VALUE, Integer.MAX_VALUE);
    }
    public float getLayoutAlignmentX(Container target) { return 0.5f; } // 6. Align
    public float getLayoutAlignmentY(Container target) { return 0.5f; }
    public void invalidateLayout(Container target) {}

    public void layoutContainer(Container parent) {                     // 7. Layout
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int w = parent.getWidth() - in.left - in.right;
            int h = parent.getHeight() - in.top - in.bottom;
            for (Component c : parent.getComponents()) {
                Percent p = map.get(c);
                if (p == null || !c.isVisible()) continue;
                int x = in.left + (int) Math.round(p.x * w);
                int y = in.top  + (int) Math.round(p.y * h);
                int cw = (int) Math.round(p.w * w);
                int ch = (int) Math.round(p.h * h);
                c.setBounds(x, y, cw, ch);
            }
        }
    }
}

public class PercentLayoutDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Percent Layout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JPanel panel = new JPanel(new PercentLayout());

            JButton top    = new JButton("Top");
            JButton left   = new JButton("Left");
            JButton right  = new JButton("Right");
            JButton bottom = new JButton("Bottom");

            panel.add(top,    new PercentLayout.Percent(0.0, 0.0, 1.0, 0.2));
            panel.add(left,   new PercentLayout.Percent(0.0, 0.2, 0.5, 0.6));
            panel.add(right,  new PercentLayout.Percent(0.5, 0.2, 0.5, 0.6));
            panel.add(bottom, new PercentLayout.Percent(0.0, 0.8, 1.0, 0.2));

            frame.add(panel);
            frame.setSize(500, 400);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with four buttons occupying percentage-based regions: “Top” across the full width at 20% height, “Left” and “Right” side-by-side each 50% wide and 60% tall, and “Bottom” across the full width at 20% height. The regions resize proportionally when the window is resized.

**Why the code produces this result.** `layoutContainer` scales each constraint's `(x, y, w, h)` fractions by the container's available width and height, and calls `setBounds`. The `maximumLayoutSize` returns `Integer.MAX_VALUE` so the manager imposes no upper bound, and the alignment values of `0.5f` centre it if used inside a `BoxLayout`.

---

### Real-World Cases

- **Dashboard layouts** with percentage-based tiles.
- **Responsive form layouts** where fields span variable fractions of the width.
- **Overlay layouts** where components overlap and are stacked by constraints.

---

## 4. Custom Layout Best Practices

### Definitions

**Core Definition.** Custom layout best practices are the conventions and patterns that ensure hand-written layout managers are correct, safe, and maintainable.

**Technical Definition.** Best practices include: synchronising on `Container.getTreeLock()` during iteration and size computation; honouring the container's `Insets`; respecting each child's `getPreferredSize`/`getMinimumSize`/`getMaximumSize`; using `LayoutManager2` when constraints or a maximum size are needed; returning meaningful `maximumLayoutSize` values; caching nothing that can change without `invalidateLayout`; and writing unit tests for `preferredLayoutSize` and `layoutContainer`.

**Beginner-Friendly Explanation.** Best practices for custom layout managers are the "rules of thumb" that prevent `ConcurrentModificationException`, ignored padding, clashing sizes, and unmaintainable code. They include things like "lock the tree," "respect insets," "use `LayoutManager2` for constraints," and "test your sizing logic."

---

### Purposes

- To prevent concurrent-modification errors by synchronising on the tree lock.
- To ensure padding and gaps are correctly applied by honouring insets.
- To respect the size hints of well-behaved components.
- To integrate correctly with parent layouts through meaningful maximum sizes and alignments.
- To keep the manager maintainable through unit tests and minimal, focused code.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class MyLayout implements LayoutManager2 {
    private final Map<Component, MyConstraints> constraints = new HashMap<>();

    public void addLayoutComponent(Component comp, Object c) {
        constraints.put(comp, (MyConstraints) c);
        // no need to call revalidate; Container does it
    }
    public void removeLayoutComponent(Component comp) {
        constraints.remove(comp);
    }
    public Dimension preferredLayoutSize(Container parent) {
        synchronized (parent.getTreeLock()) { ... }
    }
    public Dimension minimumLayoutSize(Container parent) {
        synchronized (parent.getTreeLock()) { ... }
    }
    public Dimension maximumLayoutSize(Container parent) {
        return new Dimension(Integer.MAX_VALUE, Integer.MAX_VALUE);
    }
    public float getLayoutAlignmentX(Container parent) { return 0.5f; }
    public float getLayoutAlignmentY(Container parent) { return 0.5f; }
    public void invalidateLayout(Container parent) { /* clear caches */ }
    public void layoutContainer(Container parent) {
        synchronized (parent.getTreeLock()) { ... }
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Tree lock | `synchronized (parent.getTreeLock())` in size and layout methods. |
| Honor insets | `Insets in = parent.getInsets();` and subtract from bounds. |
| Respect size hints | `c.getPreferredSize()`, `c.getMinimumSize()`, `c.getMaximumSize()`. |
| Use `LayoutManager2` when needed | Constraints, maximum size, alignment. |
| Meaningful maximum | `Integer.MAX_VALUE` for unbounded, or a computed max. |
| Invalidate caches | `invalidateLayout` clears any cached sizes. |
| Unit tests | Test `preferredLayoutSize` and `layoutContainer` with real components. |
| EDT | Install and modify on the EDT. |

#### Syntax Rules

- **Always synchronise on `parent.getTreeLock()`** while iterating the container's components or reading their sizes.
- **Always honour the container's `Insets`** when computing sizes and laying out children.
- **Always respect the component size hints** unless the layout's semantics require otherwise (e.g., a percentage layout intentionally overrides them).
- **Use `LayoutManager2` when constraints are needed**, and implement `maximumLayoutSize`, alignment, and invalidation.
- **Never cache results without clearing them in `invalidateLayout`**; layout managers are called after invalidation.

#### Constraints and Limitations

- Custom layout managers are harder to debug than standard ones; unit tests are essential.
- The manager is not responsible for calling `revalidate`; the container does so after `add`/`remove`.
- Serialization of custom layout managers is not guaranteed across versions.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Layout with Constraint Caching and Invalidation

```java
import java.awt.*;
import java.util.HashMap;
import java.util.Map;
import javax.swing.*;

public class BestPracticeCustomLayout implements LayoutManager2 {      // 1. Impl 2
    public static final class Constraints {
        public final int col, row, colspan, rowspan;
        public Constraints(int col, int row, int colspan, int rowspan) {
            this.col = col; this.row = row;
            this.colspan = colspan; this.rowspan = rowspan;
        }
    }

    private final Map<Component, Constraints> map = new HashMap<>();
    private Dimension cachedPreferred = null;                           // 2. Cache

    public void addLayoutComponent(Component comp, Object c) {
        if (!(c instanceof Constraints))
            throw new IllegalArgumentException("Need Constraints");
        map.put(comp, (Constraints) c);
        cachedPreferred = null;                                          // 3. Invalidate
    }
    public void addLayoutComponent(String name, Component comp) {
        map.put(comp, new Constraints(0, 0, 1, 1));
        cachedPreferred = null;
    }
    public void removeLayoutComponent(Component comp) {
        map.remove(comp);
        cachedPreferred = null;
    }

    public Dimension preferredLayoutSize(Container parent) {             // 4. Pref
        synchronized (parent.getTreeLock()) {
            if (cachedPreferred != null) return cachedPreferred;
            Insets in = parent.getInsets();
            int cols = 0, rows = 0;
            for (Constraints k : map.values()) {
                cols = Math.max(cols, k.col + k.colspan);
                rows = Math.max(rows, k.row + k.rowspan);
            }
            int cellW = 0, cellH = 0;
            for (Component c : parent.getComponents()) {
                if (!c.isVisible() || !map.containsKey(c)) continue;
                Constraints k = map.get(c);
                Dimension d = c.getPreferredSize();
                cellW = Math.max(cellW, d.width / k.colspan);
                cellH = Math.max(cellH, d.height / k.rowspan);
            }
            cachedPreferred = new Dimension(
                in.left + in.right + cols * cellW,
                in.top + in.bottom + rows * cellH);
            return cachedPreferred;
        }
    }
    public Dimension minimumLayoutSize(Container parent) {
        return preferredLayoutSize(parent);
    }
    public Dimension maximumLayoutSize(Container target) {
        return new Dimension(Integer.MAX_VALUE, Integer.MAX_VALUE);
    }
    public float getLayoutAlignmentX(Container target) { return 0.5f; }
    public float getLayoutAlignmentY(Container target) { return 0.5f; }
    public void invalidateLayout(Container target) { cachedPreferred = null; } // 5. Inval

    public void layoutContainer(Container parent) {                       // 6. Layout
        synchronized (parent.getTreeLock()) {
            Insets in = parent.getInsets();
            int cols = 0, rows = 0;
            for (Constraints k : map.values()) {
                cols = Math.max(cols, k.col + k.colspan);
                rows = Math.max(rows, k.row + k.rowspan);
            }
            if (cols == 0 || rows == 0) return;
            int w = (parent.getWidth() - in.left - in.right) / cols;
            int h = (parent.getHeight() - in.top - in.bottom) / rows;
            for (Component c : parent.getComponents()) {
                Constraints k = map.get(c);
                if (k == null || !c.isVisible()) continue;
                c.setBounds(in.left + k.col * w, in.top + k.row * h,
                    k.colspan * w, k.rowspan * h);
            }
        }
    }
}

public class BestPracticeCustomLayoutDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Custom Layout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JPanel panel = new JPanel(new BestPracticeCustomLayout());

            panel.add(new JButton("A"), new BestPracticeCustomLayout.Constraints(0, 0, 1, 1));
            panel.add(new JButton("B"), new BestPracticeCustomLayout.Constraints(1, 0, 1, 1));
            panel.add(new JButton("C"), new BestPracticeCustomLayout.Constraints(0, 1, 2, 1));

            frame.add(panel);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a 2×2 grid in which “A” and “B” occupy the first row and “C” spans both columns in the second row.

**Why the code produces this result.** The `Constraints` object defines each component's column, row, colspan, and rowspan. The `preferredLayoutSize` caches its result for performance, and `invalidateLayout` clears that cache when the constraints or children change. `layoutContainer` computes equal cell sizes and positions each component according to its span.

---

#### Example 2: Common Mistake — Not Synchronising on the Tree Lock

```java
import java.awt.*;
import javax.swing.*;

public class BadLayout implements LayoutManager {
    public void addLayoutComponent(String name, Component comp) {}
    public void removeLayoutComponent(Component comp) {}

    public Dimension preferredLayoutSize(Container parent) {            // WRONG: no lock
        int w = 0;
        for (Component c : parent.getComponents()) {                    // 1. Unsafe
            w += c.getPreferredSize().width;
        }
        return new Dimension(w, 10);
    }
    public Dimension minimumLayoutSize(Container parent) {
        return preferredLayoutSize(parent);
    }
    public void layoutContainer(Container parent) {                     // WRONG: no lock
        int x = 0;
        for (Component c : parent.getComponents()) {                    // 2. Unsafe
            c.setBounds(x, 0, c.getPreferredSize().width, parent.getHeight());
            x += c.getPreferredSize().width;
        }
    }
}
```

**Expected Output.** The layout may work in simple single-threaded cases but can throw `ConcurrentModificationException` or produce inconsistent results when components are added or removed while a layout is in progress (e.g., during a background task or animation).

**Why this is wrong.** As the Java documentation for `Container` states, iteration over a container's components must be synchronised on `getTreeLock()`. Standard layout managers (including `BoxLayout`, `GridLayout`, and `BorderLayout`) all use `synchronized (parent.getTreeLock())` in their size and layout methods. Custom managers must do the same to be thread-safe and consistent.

---

### Real-World Cases

- **Production Swing applications** that need domain-specific arrangements (circular menus, treemaps, diagonal stacks) use custom layout managers with the best-practice skeleton.
- **GUI framework developers** implement custom layout managers as part of reusable UI libraries.
- **Educational examples** use the best-practice skeleton to teach the layout-manager contract and the tree-lock discipline from the start.

---

## References

Oracle. *Interface LayoutManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/LayoutManager.html

Oracle. *Interface LayoutManager2*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/LayoutManager2.html

Oracle. *Class Container*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/Container.html

Oracle. *Laying Out Components Within a Container*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/index.html

Oracle. *How Layout Management Works*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/howLayoutWorks.html

Oracle. *Creating a Custom Layout Manager*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/custom.html

Oracle. *Class BoxLayout*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/BoxLayout.html

Oracle. *Class GridBagLayout*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/GridBagLayout.html

Oracle. *Class LayoutStyle*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/LayoutStyle.html

Stack Overflow. *How to create a custom LayoutManager*. https://stackoverflow.com/questions/ (community discussion of `LayoutManager` vs `LayoutManager2`, tree lock, and insets).

Stack Overflow. *Why synchronise on Container.getTreeLock()*. https://stackoverflow.com/questions/ (discussion of thread-safe component iteration).