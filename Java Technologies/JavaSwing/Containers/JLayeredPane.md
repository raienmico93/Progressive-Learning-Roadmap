# JLayeredPane — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JLayeredPane` is a Swing container that adds a third dimension—depth, also known as Z-order—to component positioning. When components overlap, the component at a higher depth is drawn on top of components at a lower depth, allowing developers to create layered interfaces with overlapping elements.

**Technical Definition.** `public class JLayeredPane extends JComponent implements Accessible`. The class manages its children like a normal `Container`, but divides the depth range into several distinct integer layers. Each component's layer is specified by an `Integer` object passed during the `add` call or set later via `setLayer`. Higher-numbered layers display above lower-numbered layers. `JLayeredPane` is the class used by the root panes of all Swing top-level containers (`JFrame`, `JDialog`, `JWindow`, `JApplet`, `JInternalFrame`). `JDesktopPane` is a specialized subclass designed for holding internal frames. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JLayeredPane` is like a stack of transparent sheets. On each sheet, you can draw or place a component. Sheets higher in the stack cover sheets lower in the stack. You can move components between sheets or change their order within a sheet. This is useful when you need one component to float above another—such as a tooltip above a dialog, a drag-and-drop ghost image above everything else, or a floating palette above the main content. Unlike a regular panel where components are arranged side by side, a layered pane lets components overlap in a controlled, predictable way.

---

### Key Characteristics

1. **Depth-based ordering.** Each component is assigned an integer layer. Higher layers are drawn on top of lower layers.
2. **Position within layer.** Within the same layer, components have a position. Position 0 is the topmost position within that layer, and `-1` indicates the bottommost position.
3. **Predefined layer constants.** `JLayeredPane` provides convenience constants: `DEFAULT_LAYER`, `PALETTE_LAYER`, `MODAL_LAYER`, `POPUP_LAYER`, and `DRAG_LAYER`, each with a specific semantic purpose.
4. **No layout manager by default.** A `JLayeredPane` has no default layout manager, meaning components must be positioned manually using `setBounds` or a custom layout manager must be assigned.
5. **Root pane integration.** Every Swing top-level container automatically has a layered pane in its root pane, though most programs do not interact with it directly.
6. **Dynamic reordering.** Components can be moved to the front or back within their layer using `moveToFront` and `moveToBack`, or repositioned precisely with `setPosition`.
7. **Single-thread rule.** Like all Swing components, `JLayeredPane` is not thread-safe and must be accessed on the EDT.
8. **Serialization caveat.** Serialized `JLayeredPane` objects are not compatible with future Swing releases; serialization is appropriate only for short-term storage or RMI between applications running the same Swing version.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, and the `public static void main(String[] args)` entry point.
- Familiarity with Swing containers such as `JPanel`, `JFrame`, and the `JRootPane` architecture.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of absolute positioning (`setBounds`) since `JLayeredPane` has no default layout manager.
- A JDK (version 8 or later; `JLayeredPane` is available in all modern JDKs).

---

### Related Programming Areas

- **`JRootPane`:** Every top-level Swing container's root pane contains a `JLayeredPane` that manages the menu bar, content pane, and glass pane.
- **`JDesktopPane`:** A `JLayeredPane` subclass specialized for managing internal frames (MDI applications).
- **`JViewport` and `JScrollPane`:** Layered panes are sometimes used inside scroll panes for complex canvas-like interfaces.
- **`LayoutManager`:** While layered panes do not use a layout manager by default, one can be assigned to arrange all components as if they were on a single layer.
- **`Container` Z-order API:** The AWT `Container` class provides basic Z-order manipulation, but `JLayeredPane` extends this with named layers and predefined semantics.

---

## Core Concepts and Features

---

## 1. JLayeredPane — The Class

### Definitions

**Core Definition.** `JLayeredPane` is a Swing container that provides a depth dimension for positioning components, allowing them to overlap according to layer and position assignments.

**Technical Definition.** `public class JLayeredPane extends JComponent implements Accessible`. The class was introduced in JDK 1.2. It provides a single no-argument constructor: `JLayeredPane()`. Components are added with overloaded `add` methods that accept a layer constraint (an `Integer`) and optionally a position within that layer. The class maintains an internal mapping of components to their layer and position values.

**Beginner-Friendly Explanation.** `JLayeredPane` is the Swing container that lets you stack components on top of each other. You create it with `new JLayeredPane()`, add components with a layer number, and the pane takes care of drawing them in the correct order.

---

### Purposes

- To provide a container where components can overlap in a controlled, depth-based order.
- To support the layering infrastructure required by Swing's root panes (menu bars, content panes, glass panes).
- To enable floating palettes, drag-and-drop feedback, tooltips, and modal overlays that must appear above other components.
- To serve as the foundation for `JDesktopPane` and MDI (multiple document interface) applications.
- To allow dynamic reordering of components at runtime without removing and re-adding them.

---

### Syntax Rules and Structure

#### Complete General Syntax (Construction)

```java
JLayeredPane layeredPane = new JLayeredPane();
layeredPane.setPreferredSize(new Dimension(400, 300));
layeredPane.setBorder(BorderFactory.createTitledBorder("Layered Pane"));
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JLayeredPane()` | Creates a new layered pane with no layout manager. |

#### Syntax Rules

- `JLayeredPane` has only a no-argument constructor.
- It has no default layout manager; components must be positioned with `setBounds` or a layout manager must be explicitly set.
- The layered pane must be added to a container (e.g., a `JFrame`'s content pane) to be displayed.
- Construction must occur on the EDT.

#### Constraints and Limitations

- Without a layout manager, components added to the layered pane will have zero size unless `setBounds` is called on each component.
- The layered pane does not automatically handle component positioning; the programmer is responsible for layout.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JLayeredPane with Two Overlapping Labels

```java
import javax.swing.*;
import java.awt.*;

public class MinimalLayeredPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JLayeredPane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();         // 1. Create
            layeredPane.setPreferredSize(new Dimension(400, 300));

            JLabel background = new JLabel("Background", SwingConstants.CENTER);
            background.setOpaque(true);
            background.setBackground(Color.LIGHT_GRAY);
            background.setBounds(50, 50, 200, 100);                // 2. Position

            JLabel foreground = new JLabel("Foreground", SwingConstants.CENTER);
            foreground.setOpaque(true);
            foreground.setBackground(Color.CYAN);
            foreground.setBounds(100, 80, 200, 100);               // 3. Overlap

            layeredPane.add(background, JLayeredPane.DEFAULT_LAYER); // 4. Lower layer
            layeredPane.add(foreground, JLayeredPane.PALETTE_LAYER); // 5. Higher layer

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing two overlapping labels. The cyan “Foreground” label is drawn on top of the light-gray “Background” label because it is in the `PALETTE_LAYER`, which is above the `DEFAULT_LAYER`. Both labels are positioned manually using `setBounds`.

**Why the code produces this result.** `JLayeredPane` has no layout manager, so `setBounds` explicitly positions each label. The `add` call with `JLayeredPane.DEFAULT_LAYER` places the background label in the bottommost predefined layer, while `PALETTE_LAYER` places the foreground label above it. The layered pane's painting order respects the layer numbers: higher layers are drawn on top.

---

#### Example 2: JLayeredPane with a Custom Layer Number

```java
import javax.swing.*;
import java.awt.*;

public class CustomLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom Layer");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(350, 250));

            JLabel label1 = new JLabel("Layer 5", SwingConstants.CENTER);
            label1.setOpaque(true);
            label1.setBackground(Color.ORANGE);
            label1.setBounds(20, 20, 150, 100);

            JLabel label2 = new JLabel("Layer 10", SwingConstants.CENTER);
            label2.setOpaque(true);
            label2.setBackground(Color.PINK);
            label2.setBounds(80, 60, 150, 100);

            layeredPane.add(label1, Integer.valueOf(5));           // 1. Custom layer 5
            layeredPane.add(label2, Integer.valueOf(10));          // 2. Custom layer 10

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two overlapping labels: “Layer 5” (orange) and “Layer 10” (pink). The pink label is drawn on top because layer 10 is higher than layer 5.

**Why the code produces this result.** The `add` method accepts an `Integer` object that specifies the layer. Higher integers correspond to higher layers. Using custom layer numbers allows fine-grained control over the Z-order beyond the predefined constants.

---

### Real-World Cases

- **Root pane infrastructure:** every `JFrame`, `JDialog`, and `JApplet` uses a `JLayeredPane` internally to manage the menu bar, content pane, and glass pane.
- **Drag-and-drop interfaces:** the dragged component is temporarily moved to the `DRAG_LAYER` so it appears above all other components.
- **Tooltips and popups:** `JToolTip` and popup menus are placed in the `POPUP_LAYER` to ensure they appear above the components that triggered them.
- **Floating palettes:** tool palettes in graphics editors can be placed in the `PALETTE_LAYER` so they float above the main canvas.

---

## 2. Layer Creation

### Definitions

**Core Definition.** Layer creation is the process of instantiating a `JLayeredPane` and preparing it for receiving components in different layers.

**Technical Definition.** `JLayeredPane` is created with `new JLayeredPane()`. After construction, the pane has no components and no layout manager. Components are added with an `add` overload that accepts a layer constraint (an `Integer`) and optionally a position within the layer. The layer constraint is stored as a client property under the key `LAYER_PROPERTY`.

**Beginner-Friendly Explanation.** Creating a layered pane means making the empty container first. You then add components to it, specifying which layer each component belongs to. The layer number determines the stacking order.

---

### Purposes

- To instantiate the depth-aware container with an initial size and border.
- To prepare the pane for component addition with layer specifications.
- To establish the pane's role within a larger GUI hierarchy.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JLayeredPane layeredPane = new JLayeredPane();
layeredPane.setPreferredSize(new Dimension(400, 300));
// Components are added later with layer constraints
```

#### Component Breakdown

| Operation | Meaning |
|-----------|---------|
| `new JLayeredPane()` | Creates the layered pane. |
| `setPreferredSize(Dimension)` | Gives the pane a preferred size for layout managers. |
| `add(Component, Integer)` | Adds a component to a specific layer. |

#### Syntax Rules

- The pane has no layout manager by default; `setBounds` must be called on each child.
- The preferred size should be set explicitly for the pane to appear correctly inside layout-managed containers.
- Creation must occur on the EDT.

#### Constraints and Limitations

- If no preferred size is set and no layout manager is assigned, the pane may size itself to zero and become invisible.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Layered Pane with Preferred Size and Border

```java
import javax.swing.*;
import javax.swing.border.*;
import java.awt.*;

public class LayerCreationDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Layer Creation");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();         // 1. Create
            layeredPane.setPreferredSize(new Dimension(400, 300)); // 2. Preferred size
            layeredPane.setBorder(BorderFactory.createTitledBorder( // 3. Border
                BorderFactory.createLineBorder(Color.GRAY), "Layers"));

            JLabel label = new JLabel("Content", SwingConstants.CENTER);
            label.setOpaque(true);
            label.setBackground(Color.YELLOW);
            label.setBounds(100, 100, 200, 100);                   // 4. Position

            layeredPane.add(label, JLayeredPane.DEFAULT_LAYER);    // 5. Add

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a gray titled border labeled “Layers” and a yellow label positioned at (100, 100) with size 200×100.

**Why the code produces this result.** The preferred size ensures the pane is 400×300. The border adds visual separation. `setBounds` positions the label, and `add` places it in the `DEFAULT_LAYER`.

---

### Real-World Cases

- **Creating a canvas** for a drawing application where shapes need to overlap in a controlled order.
- **Building a game board** where game pieces can be stacked and moved between layers.

---

## 3. Layer Methods — add()

### Definitions

**Core Definition.** The `add` method adds a component to the layered pane with a specified layer constraint and, optionally, a position within that layer.

**Technical Definition.** `JLayeredPane` overrides `Container.add` with overloads that accept an `Integer` layer constraint. The primary overloads are:
- `add(Component comp, Integer layer)`
- `add(Component comp, Integer layer, int position)`

The layer constraint is stored as a client property. The position argument determines the component's order within its layer. Valid positions range from `0` (topmost) to `n - 1` (bottommost), with `-1` indicating the bottommost position.

**Beginner-Friendly Explanation.** `add` is how you put a component into the layered pane. You tell it which component to add, which layer it belongs to, and (optionally) where it should sit within that layer. Position 0 means “on top of everything else in this layer,” and position -1 means “at the bottom of this layer.”

---

### Purposes

- To add a component to a specific layer in the layered pane.
- To control the component's position within its layer.
- To establish the component's depth relationship with other components.
- To dynamically place components at runtime based on application logic.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.add(component, JLayeredPane.DEFAULT_LAYER);
layeredPane.add(component, Integer.valueOf(10));
layeredPane.add(component, JLayeredPane.PALETTE_LAYER, 0);
```

#### Component Breakdown

| Overload | Parameters | Meaning |
|----------|------------|---------|
| `add(Component, Integer)` | Component, layer | Adds the component to the specified layer at the bottom position. |
| `add(Component, Integer, int)` | Component, layer, position | Adds the component to the specified layer at the specified position. |

#### Syntax Rules

- The layer argument must be an `Integer` object, not a primitive `int`. Using a primitive `int` will call the `add(Component, Object)` overload inherited from `Container`, which does not set a layer properly and can cause unexpected behavior.
- Position 0 is the topmost position within the layer; `-1` is the bottommost.
- If no position is specified, the component is placed at the bottom of its layer.

#### Constraints and Limitations

- Using a primitive `int` instead of `Integer` for the layer argument is a common mistake that bypasses the layer-setting logic.
- The component will not have a size or position unless `setBounds` is called on it after addition.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Adding Components with Layer and Position

```java
import javax.swing.*;
import java.awt.*;

public class AddDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Add Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(400, 300));

            // Three components in the same layer (PALETTE_LAYER)
            JLabel a = createLabel("A", Color.RED, 50, 50);
            JLabel b = createLabel("B", Color.GREEN, 100, 80);
            JLabel c = createLabel("C", Color.BLUE, 150, 110);

            layeredPane.add(a, JLayeredPane.PALETTE_LAYER, 0);      // 1. Topmost
            layeredPane.add(b, JLayeredPane.PALETTE_LAYER, 1);      // 2. Middle
            layeredPane.add(c, JLayeredPane.PALETTE_LAYER, 2);      // 3. Bottom

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static JLabel createLabel(String text, Color bg, int x, int y) {
        JLabel label = new JLabel(text, SwingConstants.CENTER);
        label.setOpaque(true);
        label.setBackground(bg);
        label.setBounds(x, y, 150, 100);
        return label;
    }
}
```

**Expected Output.** Three overlapping colored labels: red “A” on top, green “B” in the middle, and blue “C” at the bottom. All three are in the same `PALETTE_LAYER`, but their positions (0, 1, 2) determine their stacking order within that layer.

**Why the code produces this result.** Position 0 places “A” at the top of the layer, position 1 places “B” in the middle, and position 2 places “C” at the bottom. Even though the components overlap, the layered pane draws them in position order.

---

#### Example 2: Common Mistake — Using Primitive `int` Instead of `Integer`

```java
import javax.swing.*;

public class BadAddDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Add");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new java.awt.Dimension(300, 200));

            JLabel label = new JLabel("Wrong");
            label.setBounds(50, 50, 100, 50);

            // WRONG: primitive int calls add(Component, Object)
            layeredPane.add(label, 5);                              // 1. Primitive int

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label may not be placed in the intended layer, and its behavior may be inconsistent. The compiler does not warn about this mistake.

**Why this is wrong.** `add(Component, int)` does not exist as a layer-aware overload. The `int` is autoboxed to `Integer`, but it is passed as the `Object` constraint to `Container.add`, which may be interpreted as a layout constraint rather than a layer specification. The correct approach is to use `Integer.valueOf(5)` or a predefined layer constant.

---

### Real-World Cases

- **Adding components to the `DRAG_LAYER`** during a drag operation, then moving them back to their original layer when the drag ends.
- **Placing popups and tooltips in the `POPUP_LAYER`** so they appear above dialogs.
- **Building a drawing application** where each shape is added to a specific layer based on its z-order in the scene.

---

## 4. Layer Methods — setLayer()

### Definitions

**Core Definition.** `setLayer(Component c, int layer)` changes the layer of an existing component without removing it from the container.

**Technical Definition.** There are two overloads:
- `setLayer(Component c, int layer)` — changes the layer, placing the component at the bottom of the new layer.
- `setLayer(Component c, int layer, int position)` — changes both the layer and the position within that layer.

The method updates the component's `LAYER_PROPERTY` client property and triggers a revalidation of the container.

**Beginner-Friendly Explanation.** `setLayer` lets you move a component from one layer to another after it has already been added. This is useful for dynamically changing the stacking order—for example, bringing a dragged component to the top layer while it is being dragged, then returning it to its normal layer afterward.

---

### Purposes

- To change a component's layer dynamically without removing and re-adding it.
- To temporarily promote a component to a higher layer (e.g., the `DRAG_LAYER`) and later restore it.
- To reorganize the layering of components in response to application state changes.
- To move a component to a different semantic layer when its role changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.setLayer(component, JLayeredPane.DRAG_LAYER);
layeredPane.setLayer(component, Integer.valueOf(20), 0);
```

#### Component Breakdown

| Overload | Parameters | Meaning |
|----------|------------|---------|
| `setLayer(Component, int)` | Component, new layer | Changes the layer; places component at bottom of new layer. |
| `setLayer(Component, int, int)` | Component, new layer, position | Changes layer and position. |

#### Syntax Rules

- The layer argument is a primitive `int` in `setLayer`, unlike `add` which requires an `Integer`.
- The component must already be a child of the layered pane.
- The change is reflected immediately, and the container is revalidated.

#### Constraints and Limitations

- Calling `setLayer` on a component that is not a child of the layered pane has no effect.
- The component's position within the new layer defaults to the bottom if not specified.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Promoting a Component to the Drag Layer

```java
import javax.swing.*;
import java.awt.*;

public class SetLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetLayer Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(400, 300));

            JLabel normal = new JLabel("Normal", SwingConstants.CENTER);
            normal.setOpaque(true);
            normal.setBackground(Color.LIGHT_GRAY);
            normal.setBounds(50, 50, 150, 100);

            JLabel draggable = new JLabel("Draggable", SwingConstants.CENTER);
            draggable.setOpaque(true);
            draggable.setBackground(Color.CYAN);
            draggable.setBounds(100, 80, 150, 100);

            layeredPane.add(normal, JLayeredPane.DEFAULT_LAYER);
            layeredPane.add(draggable, JLayeredPane.PALETTE_LAYER);

            JButton promote = new JButton("Promote to Drag Layer");
            promote.addActionListener(e ->
                layeredPane.setLayer(draggable, JLayeredPane.DRAG_LAYER)); // 1. Promote

            JButton demote = new JButton("Demote to Palette");
            demote.addActionListener(e ->
                layeredPane.setLayer(draggable, JLayeredPane.PALETTE_LAYER)); // 2. Demote

            JPanel controls = new JPanel();
            controls.add(promote);
            controls.add(demote);

            frame.add(layeredPane, BorderLayout.CENTER);
            frame.add(controls, BorderLayout.SOUTH);
            frame.setSize(450, 400);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two overlapping labels. Clicking “Promote to Drag Layer” moves the cyan “Draggable” label to the `DRAG_LAYER`, placing it above everything else. Clicking “Demote to Palette” returns it to the `PALETTE_LAYER`.

**Why the code produces this result.** `setLayer` changes the component's layer in the layered pane's internal mapping. The `DRAG_LAYER` is above all other predefined layers, so the component is drawn on top. The change is visible immediately without removing and re-adding the component.

---

### Real-World Cases

- **Drag-and-drop:** during a drag operation, the dragged component is temporarily moved to the `DRAG_LAYER` and then restored to its original layer.
- **Modal overlays:** a panel can be moved to the `MODAL_LAYER` when a modal operation begins.
- **Dynamic dashboards:** widgets can be promoted or demoted based on user focus or alerts.

---

## 5. Layer Methods — setPosition()

### Definitions

**Core Definition.** `setPosition(Component c, int position)` moves a component to a specific position within its current layer, without changing its layer.

**Technical Definition.** `public void setPosition(Component c, int position)`. The position argument ranges from `0` (topmost within the layer) to `n - 1` (bottommost), with `-1` indicating the bottommost position. The method updates the component's order within its layer and triggers revalidation.

**Beginner-Friendly Explanation.** `setPosition` lets you reorder components that are already in the same layer. You can bring a component to the top of its layer (position 0) or send it to the bottom (position -1) without changing which layer it belongs to.

---

### Purposes

- To reorder components within the same layer.
- To bring a component to the front or send it to the back of its layer.
- To implement “bring to front” and “send to back” functionality within a layer.
- To adjust the stacking order of overlapping components that belong to the same semantic group.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.setPosition(component, 0);  // top of its layer
layeredPane.setPosition(component, -1); // bottom of its layer
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setPosition(Component, int)` | Moves the component to the given position within its current layer. |

#### Syntax Rules

- The position must be between `-1` and `n - 1`, where `n` is the number of components in the layer.
- Position `0` is the topmost; `-1` is the bottommost.
- The component's layer is not changed.

#### Constraints and Limitations

- If the position is out of range, the behavior may be undefined or an exception may be thrown.
- The component must already be a child of the layered pane.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reordering Within a Layer

```java
import javax.swing.*;
import java.awt.*;

public class SetPositionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetPosition Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(350, 250));

            JLabel a = makeLabel("A", Color.RED, 30, 30);
            JLabel b = makeLabel("B", Color.GREEN, 80, 60);
            JLabel c = makeLabel("C", Color.BLUE, 130, 90);

            layeredPane.add(a, JLayeredPane.DEFAULT_LAYER);
            layeredPane.add(b, JLayeredPane.DEFAULT_LAYER);
            layeredPane.add(c, JLayeredPane.DEFAULT_LAYER);

            JButton toFront = new JButton("Bring A to Front");
            toFront.addActionListener(e ->
                layeredPane.setPosition(a, 0));                      // 1. Top of layer

            JButton toBack = new JButton("Send A to Back");
            toBack.addActionListener(e ->
                layeredPane.setPosition(a, -1));                     // 2. Bottom of layer

            JPanel controls = new JPanel();
            controls.add(toFront);
            controls.add(toBack);

            frame.add(layeredPane, BorderLayout.CENTER);
            frame.add(controls, BorderLayout.SOUTH);
            frame.setSize(400, 350);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static JLabel makeLabel(String text, Color bg, int x, int y) {
        JLabel label = new JLabel(text, SwingConstants.CENTER);
        label.setOpaque(true);
        label.setBackground(bg);
        label.setBounds(x, y, 150, 100);
        return label;
    }
}
```

**Expected Output.** Three overlapping labels. Initially, they are stacked in the order they were added. Clicking “Bring A to Front” moves the red “A” label to the top of the `DEFAULT_LAYER`. Clicking “Send A to Back” moves it to the bottom.

**Why the code produces this result.** `setPosition(a, 0)` moves “A” to position 0, making it the topmost component in the `DEFAULT_LAYER`. `setPosition(a, -1)` moves it to the bottommost position. The layer itself does not change.

---

### Real-World Cases

- **“Bring to front” / “Send to back”** commands in drawing applications.
- **Reordering tabs or cards** within a custom container.
- **Managing overlapping windows** in a desktop pane.

---

## 6. Layer Methods — getLayer()

### Definitions

**Core Definition.** `getLayer(Component c)` returns the layer number of the specified component, or `0` (the `DEFAULT_LAYER`) if the component has no layer property set.

**Technical Definition.** `public static int getLayer(JComponent c)` is a static method that retrieves the `LAYER_PROPERTY` client property from the component. If the property is not set, it returns `DEFAULT_LAYER`. There is also an instance method `getLayer(Component c)` that behaves similarly.

**Beginner-Friendly Explanation.** `getLayer` tells you which layer a component is currently in. You can use it to check a component's layer before moving it or to log its position for debugging.

---

### Purposes

- To determine the current layer of a component.
- To verify that a component has been added to the intended layer.
- To make decisions about reordering or promoting components based on their current layer.
- To debug layering issues by inspecting the layer values.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int layer = JLayeredPane.getLayer(component);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getLayer(Component)` | Returns the layer of the component. |

#### Syntax Rules

- The method is static and can be called on the `JLayeredPane` class.
- If the component has no layer property, the default layer (`0`) is returned.

#### Constraints and Limitations

- The method does not indicate the component's position within the layer; use `getPosition` for that.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading a Component's Layer

```java
import javax.swing.*;
import java.awt.*;

public class GetLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("GetLayer Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(300, 200));

            JLabel label = new JLabel("Test", SwingConstants.CENTER);
            label.setBounds(50, 50, 150, 100);
            layeredPane.add(label, JLayeredPane.PALETTE_LAYER);

            JButton checkButton = new JButton("Check Layer");
            checkButton.addActionListener(e -> {
                int layer = JLayeredPane.getLayer(label);             // 1. Read layer
                System.out.println("Layer: " + layer);                // 2. Print
            });

            frame.add(layeredPane, BorderLayout.CENTER);
            frame.add(checkButton, BorderLayout.SOUTH);
            frame.setSize(350, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Check Layer” prints “Layer: 200” (the numeric value of `PALETTE_LAYER`) to the console.

**Why the code produces this result.** `getLayer` retrieves the `LAYER_PROPERTY` from the label, which was set when the label was added with `PALETTE_LAYER`. The numeric value of `PALETTE_LAYER` is 200.

---

### Real-World Cases

- **Debugging** layering issues by printing component layers.
- **Conditional logic** that behaves differently based on which layer a component occupies.
- **Validation** to ensure components have been placed in the correct layer before performing operations.

---

## 7. Layer Constants — DEFAULT_LAYER

### Definitions

**Core Definition.** `DEFAULT_LAYER` is the standard layer where most components are placed. It is the bottommost predefined layer.

**Technical Definition.** `public static final Integer DEFAULT_LAYER` has the value `0`. It is the layer used by default when a component is added without an explicit layer constraint.

**Beginner-Friendly Explanation.** `DEFAULT_LAYER` is the “ground floor” of the layered pane. Most components go here. Everything else sits above it.

---

### Purposes

- To provide a standard layer for ordinary components.
- To serve as the baseline above which higher layers are stacked.
- To allow components to be added without specifying a layer (the default behavior).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.add(component, JLayeredPane.DEFAULT_LAYER);
```

#### Component Breakdown

| Constant | Value | Meaning |
|----------|-------|---------|
| `DEFAULT_LAYER` | 0 | Bottommost predefined layer. |

#### Syntax Rules

- This is the default layer for components added without an explicit layer.
- It is the lowest layer in the predefined hierarchy.

#### Constraints and Limitations

- Components in `DEFAULT_LAYER` will be obscured by components in any higher layer.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Placing Components in DEFAULT_LAYER

```java
import javax.swing.*;
import java.awt.*;

public class DefaultLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DEFAULT_LAYER");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(300, 200));

            JLabel base = new JLabel("Base", SwingConstants.CENTER);
            base.setOpaque(true);
            base.setBackground(Color.LIGHT_GRAY);
            base.setBounds(50, 50, 150, 100);

            JLabel overlay = new JLabel("Overlay", SwingConstants.CENTER);
            overlay.setOpaque(true);
            overlay.setBackground(Color.CYAN);
            overlay.setBounds(100, 80, 150, 100);

            layeredPane.add(base, JLayeredPane.DEFAULT_LAYER);       // 1. Default
            layeredPane.add(overlay, JLayeredPane.PALETTE_LAYER);    // 2. Above

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The cyan “Overlay” label is drawn on top of the light-gray “Base” label because `PALETTE_LAYER` is above `DEFAULT_LAYER`.

**Why the code produces this result.** `DEFAULT_LAYER` is the bottommost layer, so any component in a higher layer will appear above it.

---

### Real-World Cases

- **Standard content** in a layered pane that should be beneath floating palettes and dialogs.
- **Background elements** that should not obscure interactive controls.

---

## 8. Layer Constants — PALETTE_LAYER

### Definitions

**Core Definition.** `PALETTE_LAYER` is the layer above `DEFAULT_LAYER`, intended for floating toolbars and palettes that should sit above standard components.

**Technical Definition.** `public static final Integer PALETTE_LAYER` has the value `100`. It is useful for components that should be positioned above other components but below modal dialogs and popups.

**Beginner-Friendly Explanation.** `PALETTE_LAYER` is the “first floor” above the ground floor. It is where you put toolbars, palettes, and other floating controls that should not be obscured by ordinary content.

---

### Purposes

- To position floating toolbars and palettes above standard components.
- To keep auxiliary controls visible above the main content.
- To provide a layer for components that should not be hidden by ordinary panels.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.add(toolbar, JLayeredPane.PALETTE_LAYER);
```

#### Component Breakdown

| Constant | Value | Meaning |
|----------|-------|---------|
| `PALETTE_LAYER` | 100 | Above `DEFAULT_LAYER`, for floating palettes and toolbars. |

#### Syntax Rules

- It is above `DEFAULT_LAYER` but below `MODAL_LAYER` and `POPUP_LAYER`.

#### Constraints and Limitations

- Components in `PALETTE_LAYER` can still be obscured by modal dialogs and popups.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Floating Toolbar in PALETTE_LAYER

```java
import javax.swing.*;
import java.awt.*;

public class PaletteLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("PALETTE_LAYER");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(400, 300));

            JPanel content = new JPanel();
            content.setBackground(Color.WHITE);
            content.setBounds(0, 0, 400, 300);                       // 1. Full area
            layeredPane.add(content, JLayeredPane.DEFAULT_LAYER);

            JToolBar toolbar = new JToolBar();                        // 2. Floating
            toolbar.add(new JButton("Bold"));
            toolbar.add(new JButton("Italic"));
            toolbar.setBounds(10, 10, 150, 30);
            layeredPane.add(toolbar, JLayeredPane.PALETTE_LAYER);    // 3. Palette

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white content area with a floating toolbar in the top-left corner. The toolbar appears above the white content because it is in `PALETTE_LAYER`.

**Why the code produces this result.** The toolbar is added to `PALETTE_LAYER`, which is above `DEFAULT_LAYER`. The content fills the entire pane in `DEFAULT_LAYER`, and the toolbar floats above it.

---

### Real-World Cases

- **Floating tool palettes** in graphics editors and CAD applications.
- **Detached toolbars** that can be moved freely over the main content.
- **Auxiliary control panels** that should remain accessible while the user works on the main canvas.

---

## 9. Layer Constants — MODAL_LAYER

### Definitions

**Core Definition.** `MODAL_LAYER` is the layer used for modal dialogs, ensuring they appear above toolbars, palettes, and standard components.

**Technical Definition.** `public static final Integer MODAL_LAYER` has the value `200`. Components in this layer will appear on top of any toolbars, palettes, or standard components in the container.

**Beginner-Friendly Explanation.** `MODAL_LAYER` is where modal dialogs live. They need to block interaction with everything below them, so they appear above all ordinary content and floating palettes.

---

### Purposes

- To ensure modal dialogs appear above all other non-popup components.
- To visually and interactively block access to underlying content during modal operations.
- To provide a consistent layer for dialog components that must not be obscured.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.add(dialogPanel, JLayeredPane.MODAL_LAYER);
```

#### Component Breakdown

| Constant | Value | Meaning |
|----------|-------|---------|
| `MODAL_LAYER` | 200 | Above `PALETTE_LAYER`, for modal dialogs. |

#### Syntax Rules

- It is above `DEFAULT_LAYER` and `PALETTE_LAYER` but below `POPUP_LAYER` and `DRAG_LAYER`.

#### Constraints and Limitations

- Components in `MODAL_LAYER` can be obscured by popups and drag feedback, which is usually desirable.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Modal Overlay in MODAL_LAYER

```java
import javax.swing.*;
import java.awt.*;

public class ModalLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("MODAL_LAYER");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(400, 300));

            JPanel content = new JPanel();
            content.setBackground(Color.WHITE);
            content.setBounds(0, 0, 400, 300);
            layeredPane.add(content, JLayeredPane.DEFAULT_LAYER);

            JPanel modal = new JPanel();
            modal.setBackground(new Color(255, 0, 0, 100));           // 1. Semi-transparent
            modal.setBounds(100, 75, 200, 150);
            modal.add(new JLabel("Modal Content"));
            layeredPane.add(modal, JLayeredPane.MODAL_LAYER);        // 2. Modal layer

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A white background with a semi-transparent red overlay containing “Modal Content” in the center. The overlay appears above the white content.

**Why the code produces this result.** The modal panel is in `MODAL_LAYER`, which is above `DEFAULT_LAYER`. The semi-transparent background demonstrates that it overlays the content beneath.

---

### Real-World Cases

- **Modal dialogs** that must block interaction with the underlying application.
- **Progress overlays** that prevent the user from interacting while a task completes.
- **Permission prompts** that require immediate attention.

---

## 10. Layer Constants — POPUP_LAYER

### Definitions

**Core Definition.** `POPUP_LAYER` is the layer that displays above dialogs, ensuring that popup windows associated with combo boxes, tooltips, and help text appear above the component that generated them.

**Technical Definition.** `public static final Integer POPUP_LAYER` has the value `300`. It is above `MODAL_LAYER`, so popups associated with modal components still appear correctly.

**Beginner-Friendly Explanation.** `POPUP_LAYER` is where popup menus, tooltips, and combo box drop-downs live. They need to appear on top of whatever component triggered them—even if that component is inside a modal dialog.

---

### Purposes

- To ensure popups (combo boxes, tooltips, context menus) appear above the components that generated them.
- To allow popups associated with modal dialogs to be displayed correctly.
- To provide a layer for transient help text and other popup-style components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.add(popupComponent, JLayeredPane.POPUP_LAYER);
```

#### Component Breakdown

| Constant | Value | Meaning |
|----------|-------|---------|
| `POPUP_LAYER` | 300 | Above `MODAL_LAYER`, for popups and tooltips. |

#### Syntax Rules

- It is above all other predefined layers except `DRAG_LAYER`.

#### Constraints and Limitations

- Components in `POPUP_LAYER` can be obscured by components in `DRAG_LAYER` during a drag operation.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Tooltip-Style Popup in POPUP_LAYER

```java
import javax.swing.*;
import java.awt.*;

public class PopupLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("POPUP_LAYER");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(400, 300));

            JPanel background = new JPanel();
            background.setBackground(Color.LIGHT_GRAY);
            background.setBounds(0, 0, 400, 300);
            layeredPane.add(background, JLayeredPane.DEFAULT_LAYER);

            JLabel tooltip = new JLabel(" This is a tooltip ");
            tooltip.setOpaque(true);
            tooltip.setBackground(Color.YELLOW);
            tooltip.setBorder(BorderFactory.createLineBorder(Color.BLACK));
            tooltip.setBounds(150, 120, 150, 30);
            layeredPane.add(tooltip, JLayeredPane.POPUP_LAYER);       // 1. Popup layer

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A light-gray background with a yellow tooltip-style label appearing above it. The tooltip is clearly visible because it is in `POPUP_LAYER`.

**Why the code produces this result.** `POPUP_LAYER` is above `DEFAULT_LAYER`, so the tooltip is drawn on top of the background.

---

### Real-World Cases

- **Tooltips** and help text that appear above the components they describe.
- **Combo box drop-downs** and context menus that must appear above dialog content.
- **Popup notifications** that should not be obscured by any other component.

---

## 11. Layer Constants — DRAG_LAYER

### Definitions

**Core Definition.** `DRAG_LAYER` is the topmost predefined layer, used to ensure that a component being dragged is positioned over every other component in the container.

**Technical Definition.** `public static final Integer DRAG_LAYER` has the value `400`. When dragging a component, reassigning it to the drag layer ensures that it is visually on top of all other components. After dragging, it can be reassigned to its normal layer.

**Beginner-Friendly Explanation.** `DRAG_LAYER` is the “top of the stack.” Anything in this layer is drawn above everything else. It is used during drag operations so the dragged item is never hidden behind other components.

---

### Purposes

- To place a dragged component above all other components during a drag operation.
- To provide a temporary topmost layer for visual feedback.
- To ensure that drag-and-drop interactions are always visible to the user.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
layeredPane.add(draggedComponent, JLayeredPane.DRAG_LAYER);
// or later:
layeredPane.setLayer(draggedComponent, JLayeredPane.DRAG_LAYER);
```

#### Component Breakdown

| Constant | Value | Meaning |
|----------|-------|---------|
| `DRAG_LAYER` | 400 | Topmost predefined layer. |

#### Syntax Rules

- It is above all other predefined layers.
- Components should be returned to their normal layer after the drag operation completes.

#### Constraints and Limitations

- The drag layer is typically used only temporarily; leaving components there permanently may obscure other content.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dragging a Component in DRAG_LAYER

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class DragLayerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DRAG_LAYER");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new Dimension(400, 300));

            JLabel draggable = new JLabel("Drag Me", SwingConstants.CENTER);
            draggable.setOpaque(true);
            draggable.setBackground(Color.CYAN);
            draggable.setBounds(100, 100, 120, 60);
            layeredPane.add(draggable, JLayeredPane.DEFAULT_LAYER);   // 1. Start

            draggable.addMouseListener(new MouseAdapter() {
                @Override
                public void mousePressed(MouseEvent e) {
                    layeredPane.setLayer(draggable, JLayeredPane.DRAG_LAYER); // 2. Promote
                }
                @Override
                public void mouseReleased(MouseEvent e) {
                    layeredPane.setLayer(draggable, JLayeredPane.DEFAULT_LAYER); // 3. Restore
                }
            });

            draggable.addMouseMotionListener(new MouseMotionAdapter() {
                @Override
                public void mouseDragged(MouseEvent e) {
                    draggable.setLocation(
                        draggable.getX() + e.getX() - 60,
                        draggable.getY() + e.getY() - 30);
                }
            });

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A cyan “Drag Me” label. When the user presses the mouse on the label, it is promoted to `DRAG_LAYER` and becomes the topmost component. While dragging, it moves with the mouse and stays above all other components. When the mouse is released, it is returned to `DEFAULT_LAYER`.

**Why the code produces this result.** The `mousePressed` listener moves the label to `DRAG_LAYER`, ensuring it is painted above everything else. The `mouseReleased` listener restores it to `DEFAULT_LAYER`. The `mouseDragged` listener updates the label's position.

---

### Real-World Cases

- **Drag-and-drop file managers** where the dragged file icon must appear above all other files.
- **Canvas editors** where shapes are dragged and must remain visible during the operation.
- **Dashboard widgets** that can be repositioned by dragging and should stay on top while moving.

---

## 12. JLayeredPane Best Practices

### Definitions

**Core Definition.** `JLayeredPane` best practices are the conventions and patterns that ensure layered panes are used effectively, efficiently, and maintainably in Swing applications.

**Technical Definition.** Best practices include: always using `Integer` (not primitive `int`) for layer arguments in `add`; setting explicit bounds on child components since there is no default layout manager; using the predefined layer constants for their intended purposes; promoting components to `DRAG_LAYER` only during drag operations; respecting the EDT; and considering whether a layered pane is truly necessary versus simpler alternatives.

**Beginner-Friendly Explanation.** Best practices for layered panes are the “rules of thumb” that prevent common mistakes: using the wrong type for layer arguments, forgetting to position components, or leaving components in the drag layer after the drag is over. They help you use layering effectively without creating confusing or broken interfaces.

---

### Purposes

- To prevent the common `Integer` vs. `int` mistake that silently breaks layer assignment.
- To ensure components are properly sized and positioned within the layered pane.
- To use the predefined layers appropriately for their intended semantics.
- To maintain the correct layering during and after drag operations.
- To keep the GUI thread-safe by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
JLayeredPane layeredPane = new JLayeredPane();
layeredPane.setPreferredSize(new Dimension(400, 300));          // 1. Size

JLabel background = new JLabel("Background");
background.setOpaque(true);
background.setBackground(Color.LIGHT_GRAY);
background.setBounds(0, 0, 400, 300);                          // 2. Bounds
layeredPane.add(background, JLayeredPane.DEFAULT_LAYER);       // 3. Integer

JLabel overlay = new JLabel("Overlay");
overlay.setOpaque(true);
overlay.setBackground(Color.CYAN);
overlay.setBounds(100, 100, 150, 100);
layeredPane.add(overlay, JLayeredPane.PALETTE_LAYER);          // 4. Higher layer

// For drag operations:
// layeredPane.setLayer(overlay, JLayeredPane.DRAG_LAYER);
// ... drag ...
// layeredPane.setLayer(overlay, JLayeredPane.PALETTE_LAYER);
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `Integer` for `add` layers | `add(comp, JLayeredPane.DEFAULT_LAYER)` or `add(comp, Integer.valueOf(10))` |
| Set bounds on children | `comp.setBounds(x, y, w, h)` before or after adding |
| Set preferred size on the pane | `layeredPane.setPreferredSize(...)` |
| Use predefined constants semantically | `PALETTE_LAYER` for toolbars, `MODAL_LAYER` for dialogs, etc. |
| Restore components after drag | `setLayer(comp, originalLayer)` after `mouseReleased` |
| Respect the EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Never use a primitive `int` for the layer argument in `add`**; use `Integer.valueOf(...)` or a predefined constant. Primitive `int` will call `add(Component, Object)` and not set the layer.
- **Always set bounds on child components** because `JLayeredPane` has no default layout manager.
- **Always set a preferred size** on the layered pane itself so it is visible inside layout-managed containers.
- **Use `setLayer` with primitive `int`** (not `Integer`) since `setLayer` expects a primitive `int`.
- **Promote to `DRAG_LAYER` only during drag operations** and restore the original layer afterward.

#### Constraints and Limitations

- Without bounds, components will have zero size and be invisible.
- Without a preferred size, the layered pane may collapse to zero size in layout-managed containers.
- Leaving components in `DRAG_LAYER` permanently will cause them to obscure all other content.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Layered Pane with Drag Support

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class BestPracticeLayeredPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeLayeredPane::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice JLayeredPane");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JLayeredPane layeredPane = new JLayeredPane();
        layeredPane.setPreferredSize(new Dimension(500, 400));      // 1. Size

        // Background layer
        JPanel background = new JPanel();
        background.setBackground(Color.WHITE);
        background.setBounds(0, 0, 500, 400);                      // 2. Bounds
        layeredPane.add(background, JLayeredPane.DEFAULT_LAYER);   // 3. Integer

        // Floating palette layer
        JPanel palette = new JPanel();
        palette.setBackground(Color.LIGHT_GRAY);
        palette.setBounds(20, 20, 150, 100);
        palette.add(new JLabel("Tool Palette"));
        layeredPane.add(palette, JLayeredPane.PALETTE_LAYER);      // 4. Higher layer

        // Draggable component
        JLabel draggable = new JLabel("Drag Me", SwingConstants.CENTER);
        draggable.setOpaque(true);
        draggable.setBackground(Color.CYAN);
        draggable.setBounds(200, 150, 120, 60);
        layeredPane.add(draggable, JLayeredPane.DEFAULT_LAYER);

        int originalLayer = JLayeredPane.getLayer(draggable);      // 5. Save layer

        draggable.addMouseListener(new MouseAdapter() {
            @Override
            public void mousePressed(MouseEvent e) {
                layeredPane.setLayer(draggable, JLayeredPane.DRAG_LAYER); // 6. Promote
            }
            @Override
            public void mouseReleased(MouseEvent e) {
                layeredPane.setLayer(draggable, originalLayer);    // 7. Restore
            }
        });

        draggable.addMouseMotionListener(new MouseMotionAdapter() {
            @Override
            public void mouseDragged(MouseEvent e) {
                draggable.setLocation(
                    draggable.getX() + e.getX() - 60,
                    draggable.getY() + e.getY() - 30);
            }
        });

        frame.add(layeredPane);
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A white background with a light-gray tool palette floating above it, and a cyan “Drag Me” label. When the user drags the label, it moves to the `DRAG_LAYER` and appears above the tool palette. When released, it returns to its original layer.

**Why the code produces this result.** The layered pane has a preferred size and the children have bounds set. The `add` calls use `Integer` constants for layers. During the drag, the label is promoted to `DRAG_LAYER`, and after the drag it is restored to its original layer using the saved value. The tool palette remains in `PALETTE_LAYER`, which is above `DEFAULT_LAYER` but below `DRAG_LAYER`.

---

#### Example 2: Common Mistake — Using Primitive `int` in `add()`

```java
import javax.swing.*;

public class BadLayeredPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Layered Pane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLayeredPane layeredPane = new JLayeredPane();
            layeredPane.setPreferredSize(new java.awt.Dimension(300, 200));

            JLabel label = new JLabel("Wrong");
            label.setBounds(50, 50, 100, 50);

            // WRONG: primitive int in add()
            layeredPane.add(label, 5);                              // 1. int, not Integer

            frame.add(layeredPane);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label may not be placed in the intended layer, and its layering behavior may be inconsistent. The compiler does not warn about this mistake.

**Why this is wrong.** As the Java Tutorials explicitly warn: “When adding a component to a layered pane you specify the layer with an `Integer`. When using `setLayer` to change a component's layer, you use an `int`. You might think that if you use an `int` instead of an `Integer` with the `add` method, the compiler would complain or your program would throw an illegal argument exception. But the compiler says nothing, which results in a common layered pane problem”. The correct approach is to use `Integer.valueOf(5)` or a predefined layer constant.

---

### Real-World Cases

- **Production Swing applications** use layered panes for drag-and-drop, floating palettes, and modal overlays, following the best practices to avoid silent bugs.
- **IDE and editor plugins** follow these patterns when embedding layered panes in host applications.
- **Educational examples** use the best-practice skeleton to demonstrate correct `JLayeredPane` usage from the start.

---

## References

Oracle. *Class JLayeredPane*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JLayeredPane.html.

Oracle. *How to Use Layered Panes*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/layeredpane.html.

Oracle. *Class JLayeredPane*. Java Platform, Standard Edition 17 API Specification. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/javax/swing/JLayeredPane.html.

OpenJDK. *JLayeredPane (Java SE 10 & JDK 10)*. https://docs.oracle.com/javase/10/docs/api/javax/swing/JLayeredPane.html.

OpenJDK. *JLayeredPane (Java Platform SE 9 [ad-hoc build])*. https://cr.openjdk.org/~jg/8163190/api.00/javax/swing/JLayeredPane.html.