# JPopupMenu — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JPopupMenu` is a Swing component that implements a popup menu — a small window that pops up and displays a series of choices. It is the menu that appears when the user selects an item on the menu bar, the "pull-right" menu that appears when a menu item is activated, and the context menu that appears when the user right-clicks in a specified area.

**Technical Definition.** `public class JPopupMenu extends JComponent implements Accessible, MenuElement`. It is a container for `JMenuItem`, `JCheckBoxMenuItem`, `JRadioButtonMenuItem`, and `JPopupMenu.Separator` objects. The component is not added to a normal container hierarchy; instead, it is displayed on top of all other components using the `show(Component invoker, int x, int y)` method, which positions the popup at the specified coordinates relative to the invoker component. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`. Serialized `JPopupMenu` objects are not compatible with future Swing releases.

**Beginner-Friendly Explanation.** `JPopupMenu` is the floating menu that appears when you right-click something — the one with options like "Cut," "Copy," "Paste," or "Delete." Unlike a regular menu bar that is always visible at the top of a window, a popup menu stays hidden until the user triggers it. It then appears at a specific location (typically where the mouse was clicked) and disappears once the user makes a selection or clicks elsewhere. Think of it as a contextual "command palette" that shows only the actions relevant to whatever the user just clicked on. It is also the mechanism behind the drop-down menus in a menu bar and the submenus that appear when you hover over an item like "Export As >" .

---

### Key Characteristics

1. **Context-sensitive display.** `JPopupMenu` is not permanently visible. It appears on demand, typically in response to a right-click (context menu) or as part of a menu bar's drop-down behavior.
2. **Invoker-relative positioning.** The `show(Component invoker, int x, int y)` method positions the popup at `(x, y)` in the coordinate space of the specified invoker component. The invoker is the component that "owns" the popup for positioning purposes.
3. **Lightweight by default.** `JPopupMenu` is a lightweight container by default, meaning it is rendered entirely in Java without a native peer window. The `defaultLightWeightPopupEnabled` property is `true` by default. This can be disabled with `setLightWeightPopupEnabled(false)` if heavyweight components (like AWT `Canvas` or `Panel`) are present in the invoker's hierarchy and the popup needs to appear above them.
4. **Item container.** It holds `JMenuItem` objects (and subclasses), which fire `ActionEvent`s when selected. `JCheckBoxMenuItem` and `JRadioButtonMenuItem` provide toggle and radio-button semantics, respectively.
5. **Separator grouping.** `addSeparator()` appends a `JPopupMenu.Separator` to visually divide menu items into logical groups.
6. **PopupMenuListener events.** `PopupMenuListener` provides callbacks for the three key lifecycle moments: `popupMenuWillBecomeVisible`, `popupMenuWillBecomeInvisible`, and `popupMenuCanceled`.
7. **Single-thread rule and serialization caveat.** All construction and manipulation must occur on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with `JFrame`, `JPanel`, and the Swing container hierarchy.
- Understanding of `Action` and `ActionListener`.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `MouseListener` and `MouseEvent` for triggering context menus.
- A JDK (version 8 or later; `JPopupMenu` is available in all modern JDKs).

---

### Related Programming Areas

- **`JMenuItem`, `JCheckBoxMenuItem`, `JRadioButtonMenuItem`:** The items that populate a popup menu.
- **`JMenu` and `JMenuBar`:** `JPopupMenu` is the underlying mechanism for menu bar drop-downs and submenus.
- **`PopupMenuListener` and `PopupMenuEvent`:** Observe the popup's lifecycle.
- **`MouseListener` and `MouseEvent`:** Trigger popup menus in response to right-clicks or other mouse gestures.
- **`JComponent.setComponentPopupMenu()`:** A convenience method that automatically shows a popup menu on right-click without manual mouse listeners.
- **`PopupFactory`:** The underlying factory that creates the popup window, with light/medium/heavyweight strategies.

---

## Core Concepts and Features

---

## 1. JPopupMenu — The Class

### Definitions

**Core Definition.** `JPopupMenu` is the Swing class for creating a floating menu of choices that appears on demand.

**Technical Definition.** `public class JPopupMenu extends JComponent implements Accessible, MenuElement`. It provides two constructors: `JPopupMenu()` (without an invoker) and `JPopupMenu(String label)` (with a title). Items are added with `add(JMenuItem)`, `add(Action)`, or `add(String)`. The popup is displayed with `show(Component invoker, int x, int y)`. The class supports a pluggable look-and-feel via `PopupMenuUI`.

**Beginner-Friendly Explanation.** `JPopupMenu` is the class you use to create a popup menu. You create it, add menu items to it, and then call `show()` when you want it to appear.

---

### Purposes

- To provide a context-sensitive menu that appears on demand (e.g., right-click menus).
- To serve as the drop-down mechanism for `JMenu` in a `JMenuBar` and for submenus.
- To display a compact set of choices without permanently occupying screen space.
- To support customisable, invoker-relative positioning.
- To integrate with `PopupMenuListener` for lifecycle-aware behavior.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JPopupMenu popup = new JPopupMenu();
JPopupMenu popup = new JPopupMenu("Title");
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JPopupMenu()` | Creates a popup menu without an invoker. |
| `JPopupMenu(String label)` | Creates a popup menu with the specified title. |

#### Syntax Rules

- The popup menu is not added to a container; it is shown with `show()`.
- Items are added with `add()`, `insert()`, or `addSeparator()`.
- Construction must occur on the EDT.

#### Constraints and Limitations

- The popup menu is not displayed until `show()` is called.
- A `JPopupMenu` cannot be added to multiple containers; it is a single-instance floating component.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JPopupMenu with Manual Show

```java
import javax.swing.*;
import java.awt.event.*;

public class MinimalPopupMenu {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JPopupMenu");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();                     // 1. Create popup
            JMenuItem cut = new JMenuItem("Cut");                    // 2. Create items
            JMenuItem copy = new JMenuItem("Copy");
            JMenuItem paste = new JMenuItem("Paste");
            popup.add(cut);                                          // 3. Add items
            popup.add(copy);
            popup.add(paste);

            JPanel panel = new JPanel();
            panel.setPreferredSize(new java.awt.Dimension(300, 200));
            panel.addMouseListener(new MouseAdapter() {              // 4. Trigger
                @Override
                public void mousePressed(MouseEvent e) {
                    if (e.isPopupTrigger()) {
                        popup.show(e.getComponent(), e.getX(), e.getY()); // 5. Show
                    }
                }
                @Override
                public void mouseReleased(MouseEvent e) {
                    if (e.isPopupTrigger()) {
                        popup.show(e.getComponent(), e.getX(), e.getY());
                    }
                }
            });

            frame.add(panel);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Right-clicking on the panel shows a popup menu with "Cut," "Copy," and "Paste" at the click position.

**Why the code produces this result.** The `MouseAdapter` listens for the platform-specific popup trigger (`isPopupTrigger()`). When triggered, `popup.show()` displays the menu at the mouse coordinates relative to the invoker (the panel).

---

### Real-World Cases

- **Text editors** with right-click cut/copy/paste menus.
- **File managers** with context menus for rename, delete, and properties.
- **IDEs** with context menus for refactoring, navigation, and debugging.
- **Drawing applications** with context menus for color, stroke, and layer options.

---

## 2. Popup Menu Creation

### Definitions

**Core Definition.** Popup menu creation is the process of instantiating a `JPopupMenu` and populating it with menu items and separators.

**Technical Definition.** `JPopupMenu` provides two constructors: a no-argument constructor and a constructor accepting a title string. After creation, items are added with the overloaded `add` methods, and separators with `addSeparator()`.

**Beginner-Friendly Explanation.** Creating a popup menu means making the container and then filling it with the commands you want to offer.

---

### Purposes

- To instantiate the popup container with an optional title.
- To populate the popup with the appropriate menu items.
- To prepare the popup for display via `show()` or `setComponentPopupMenu()`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JPopupMenu popup = new JPopupMenu("Edit");
popup.add(new JMenuItem("Cut"));
popup.add(new JMenuItem("Copy"));
popup.addSeparator();
popup.add(new JMenuItem("Paste"));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String label` | An optional title displayed at the top of the popup (rarely used in context menus). |

#### Syntax Rules

- The title is only visible if the popup is displayed; it is not a replacement for a `JMenu` label.
- Items are added in the order they should appear.

#### Constraints and Limitations

- The popup is not displayed until `show()` or `setComponentPopupMenu()` is used.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Popup with a Title

```java
import javax.swing.*;

public class TitledPopupCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Titled Popup");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu("Options");            // 1. Titled
            popup.add(new JMenuItem("Option 1"));
            popup.add(new JMenuItem("Option 2"));

            JButton button = new JButton("Right-click me");
            button.setComponentPopupMenu(popup);                     // 2. Attach

            frame.add(button);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Right-clicking the button shows a popup menu titled "Options" with two items.

**Why the code produces this result.** The title constructor sets the label, and `setComponentPopupMenu` automatically wires the popup to the button's right-click behavior.

---

### Real-World Cases

- **Context menus** with descriptive titles (rarely used, but useful for clarity).
- **Reusable popup menus** shared across multiple components via `setComponentPopupMenu()`.

---

## 3. Popup Menu Methods — show()

### Definitions

**Core Definition.** `show(Component invoker, int x, int y)` displays the popup menu at the specified position within the coordinate space of the invoker component.

**Technical Definition.** `public void show(Component invoker, int x, int y)`. The method sets the invoker, computes the appropriate screen location, and makes the popup visible. If the popup is already visible, the call is ignored. The `invoker` is the component relative to which the coordinates are interpreted.

**Beginner-Friendly Explanation.** `show()` is how you make the popup appear. You tell it which component the popup belongs to and where (in that component's coordinates) the popup should appear.

---

### Purposes

- To display the popup menu on demand.
- To position the popup relative to the component that triggered it.
- To integrate with custom mouse listeners for non-standard trigger gestures.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
popup.show(invokerComponent, x, y);
```

#### Component Breakdown

| Parameter | Meaning |
|--------|---------|
| `Component invoker` | The component in whose coordinate space `(x, y)` is interpreted. |
| `int x` | The x-coordinate relative to the invoker. |
| `int y` | The y-coordinate relative to the invoker. |

#### Syntax Rules

- The invoker must be showing on screen; otherwise, the behavior is undefined.
- The popup is positioned so that it does not extend beyond the screen edges (look-and-feel dependent).
- Must be called on the EDT.

#### Constraints and Limitations

- The popup does not automatically hide when the user clicks elsewhere; Swing's `PopupFactory` handles this via a mouse grabber.
- Using `show()` on a component that is not visible may cause the popup to appear at an unexpected location.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Showing a Popup at the Mouse Position

```java
import javax.swing.*;
import java.awt.event.*;

public class ShowPopupDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("show() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();
            popup.add(new JMenuItem("Action 1"));
            popup.add(new JMenuItem("Action 2"));

            JTextArea area = new JTextArea();
            area.addMouseListener(new MouseAdapter() {
                @Override
                public void mousePressed(MouseEvent e) {
                    if (e.isPopupTrigger()) {
                        popup.show(e.getComponent(), e.getX(), e.getY()); // 1. Show
                    }
                }
                @Override
                public void mouseReleased(MouseEvent e) {
                    if (e.isPopupTrigger()) {
                        popup.show(e.getComponent(), e.getX(), e.getY());
                    }
                }
            });

            frame.add(new JScrollPane(area));
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Right-clicking inside the text area shows the popup at the exact click position.

**Why the code produces this result.** The mouse listener captures the click coordinates (`e.getX()`, `e.getY()`) relative to the text area and passes them to `show()`.

---

### Real-World Cases

- **Custom trigger gestures** beyond right-click (e.g., a button click that opens a menu).
- **Programmatic popup display** in response to keyboard shortcuts or application events.

---

## 4. Popup Menu Methods — add()

### Definitions

**Core Definition.** `add(JMenuItem menuItem)`, `add(Action a)`, and `add(String s)` append a menu item to the end of the popup menu.

**Technical Definition.** `public JMenuItem add(JMenuItem menuItem)`, `public JMenuItem add(Action a)`, and `public JMenuItem add(String s)`. The `add(Action)` overload creates a `JMenuItem` configured from the action's properties. The `add(String)` overload creates a `JMenuItem` with the specified text.

**Beginner-Friendly Explanation.** `add` puts a new command at the bottom of the popup menu. You can add a pre-built `JMenuItem`, an `Action`, or just a text label.

---

### Purposes

- To populate the popup with menu items.
- To append items in the order they should appear.
- To leverage `Action` objects for shared state between menus and toolbars.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
popup.add(new JMenuItem("Cut"));
popup.add(cutAction);
popup.add("Copy");
```

#### Component Breakdown

| Overload | Meaning |
|----------|---------|
| `add(JMenuItem)` | Appends the specified item. |
| `add(Action)` | Creates and appends a `JMenuItem` from the action. |
| `add(String)` | Creates and appends a `JMenuItem` with the given text. |

#### Syntax Rules

- Items are appended in the order of the calls.
- The `add(Action)` method automatically synchronizes the item's enabled state and text with the action.

#### Constraints and Limitations

- Adding the same `JMenuItem` to multiple popups removes it from the previous popup (a Swing component can only have one parent).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Adding Items via Different Overloads

```java
import javax.swing.*;
import java.awt.event.ActionEvent;

public class AddMethodsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("add() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();

            Action cutAction = new AbstractAction("Cut") {           // 1. Action
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Cut");
                }
            };
            popup.add(cutAction);                                    // 2. add(Action)

            popup.add(new JMenuItem("Copy"));                        // 3. add(JMenuItem)

            popup.add("Paste");                                      // 4. add(String)

            JButton button = new JButton("Right-click");
            button.setComponentPopupMenu(popup);

            frame.add(button);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Right-clicking the button shows a popup with "Cut," "Copy," and "Paste."

**Why the code produces this result.** The three different `add` overloads append items to the popup.

---

### Real-World Cases

- **Building context menus** dynamically based on application state.
- **Sharing commands** between a menu bar and a context menu via `Action` objects.

---

## 5. Popup Menu Methods — insert()

### Definitions

**Core Definition.** `insert(Component component, int index)` inserts a menu item at the specified position in the popup menu.

**Technical Definition.** `public void insert(Component component, int index)`. The component is inserted at the given index, and all items at or after that index shift to the right. An overload `insert(Action a, int index)` inserts an action-derived item.

**Beginner-Friendly Explanation.** `insert` places a new item at a specific position in the menu, rather than at the end.

---

### Purposes

- To insert an item at a precise position in the menu.
- To maintain a logical ordering when adding items dynamically.
- To place a new item before or after existing items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
popup.insert(new JMenuItem("New"), 1);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Component component` | The item to insert. |
| `int index` | The position where the item should be inserted (0-based). |

#### Syntax Rules

- The index must be between 0 and `getComponentCount()` inclusive.
- Items after the index shift right.

#### Constraints and Limitations

- Inserting at an invalid index throws `IllegalArgumentException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inserting at a Specific Position

```java
import javax.swing.*;

public class InsertDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("insert() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();
            popup.add("First");
            popup.add("Third");
            popup.insert(new JMenuItem("Second"), 1);               // 1. Insert

            JButton button = new JButton("Right-click");
            button.setComponentPopupMenu(popup);

            frame.add(button);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The popup shows "First," "Second," "Third" in that order.

**Why the code produces this result.** `insert(item, 1)` places "Second" at index 1, shifting "Third" to index 2.

---

### Real-World Cases

- **Dynamic menus** where items must appear in a specific order.
- **Inserting separators** at a particular position (via `insertSeparator(int)`).

---

## 6. Popup Menu Methods — remove()

### Definitions

**Core Definition.** `remove(Component component)` removes the specified component from the popup menu. `remove(int index)` removes the component at the given index.

**Technical Definition.** `public void remove(Component component)` and `public void remove(int index)` are inherited from `Container` and work as expected for `JPopupMenu`. After removal, the menu is revalidated and repainted.

**Beginner-Friendly Explanation.** `remove` takes an item out of the popup menu.

---

### Purposes

- To remove obsolete or inapplicable items from the popup.
- To dynamically rebuild the menu based on state.
- To clean up the menu before repopulating it.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
popup.remove(item);
popup.remove(0);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `remove(Component)` | Removes the specified component. |
| `remove(int)` | Removes the component at the index. |

#### Syntax Rules

- After removal, call `revalidate()` and `repaint()` if the popup is currently visible (rare).

#### Constraints and Limitations

- Removing an item that is not present has no effect.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Removing an Item

```java
import javax.swing.*;

public class RemoveDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("remove() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();
            popup.add("Keep");
            JMenuItem removeItem = new JMenuItem("Remove Me");
            popup.add(removeItem);

            removeItem.addActionListener(e -> {
                popup.remove(removeItem);                            // 1. Remove
            });

            JButton button = new JButton("Right-click");
            button.setComponentPopupMenu(popup);

            frame.add(button);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The popup initially shows "Keep" and "Remove Me." Clicking "Remove Me" removes it from the popup, leaving only "Keep."

**Why the code produces this result.** The action listener calls `popup.remove(removeItem)`, which removes the item from the container.

---

### Real-World Cases

- **Context-sensitive menus** that remove irrelevant items based on the current selection.
- **Dynamic menus** that rebuild their contents when the application state changes.

---

## 7. Popup Menu Methods — addSeparator()

### Definitions

**Core Definition.** `addSeparator()` appends a new separator (`JPopupMenu.Separator`) at the end of the popup menu.

**Technical Definition.** `public void addSeparator()`. The method adds a `JPopupMenu.Separator` instance to the menu. The separator is a visual divider that groups related menu items.

**Beginner-Friendly Explanation.** `addSeparator()` draws a thin line between groups of menu items, making the menu easier to read.

---

### Purposes

- To visually group related menu items.
- To break long menus into logical sections.
- To improve readability and scannability.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
popup.addSeparator();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addSeparator()` | Appends a `JPopupMenu.Separator`. |

#### Syntax Rules

- Separators are not selectable; they are purely visual.
- Excessive separators can fragment the menu visually.

#### Constraints and Limitations

- Separators do not fire events.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Grouping Items with Separators

```java
import javax.swing.*;

public class SeparatorDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Separator");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();
            popup.add("Cut");
            popup.add("Copy");
            popup.add("Paste");
            popup.addSeparator();                                    // 1. Separator
            popup.add("Select All");
            popup.addSeparator();                                    // 2. Another
            popup.add("Properties");

            JTextArea area = new JTextArea();
            area.setComponentPopupMenu(popup);

            frame.add(new JScrollPane(area));
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A context menu with Cut, Copy, Paste, a separator, Select All, another separator, and Properties.

**Why the code produces this result.** Each `addSeparator()` call inserts a visual divider at the end of the menu.

---

### Real-World Cases

- **Edit menus** separating clipboard operations from selection operations.
- **File menus** separating "Exit" from the save/open group.
- **Context menus** grouping related commands (e.g., view options vs. edit options).

---

## 8. Popup Menu Events — PopupMenuListener

### Definitions

**Core Definition.** `PopupMenuListener` is an interface for receiving `PopupMenuEvent`s that fire at three key lifecycle moments: before the popup becomes visible, before it becomes invisible, and when it is canceled.

**Technical Definition.** `public interface PopupMenuListener extends EventListener`. It declares three methods: `popupMenuWillBecomeVisible(PopupMenuEvent e)`, `popupMenuWillBecomeInvisible(PopupMenuEvent e)`, and `popupMenuCanceled(PopupMenuEvent e)`. The listener is registered with `addPopupMenuListener(PopupMenuListener l)`.

**Beginner-Friendly Explanation.** A `PopupMenuListener` lets you react to the popup opening and closing. You can use it to populate the menu dynamically before it appears or to clean up after it disappears.

---

### Purposes

- To populate or update the menu just before it becomes visible.
- To perform cleanup or logging when the menu becomes invisible.
- To detect when the menu is canceled (e.g., the user clicked elsewhere).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
popup.addPopupMenuListener(new PopupMenuListener() {
    public void popupMenuWillBecomeVisible(PopupMenuEvent e) { /* populate */ }
    public void popupMenuWillBecomeInvisible(PopupMenuEvent e) { /* cleanup */ }
    public void popupMenuCanceled(PopupMenuEvent e) { /* canceled */ }
});
```

#### Component Breakdown

| Method | Invoked When |
|--------|-------------|
| `popupMenuWillBecomeVisible` | Before the popup becomes visible. |
| `popupMenuWillBecomeInvisible` | Before the popup becomes invisible. |
| `popupMenuCanceled` | When the popup is canceled (clicked outside). |

#### Syntax Rules

- The listener is invoked on the EDT.
- `popupMenuWillBecomeVisible` is the ideal place to rebuild the menu's contents dynamically.

#### Constraints and Limitations

- The listener does not indicate which item was selected; use `ActionListener` on individual items for that.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Population on Visibility

```java
import javax.swing.*;
import javax.swing.event.PopupMenuEvent;
import javax.swing.event.PopupMenuListener;

public class PopupListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("PopupMenuListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPopupMenu popup = new JPopupMenu();
            popup.addPopupMenuListener(new PopupMenuListener() {     // 1. Listener
                public void popupMenuWillBecomeVisible(PopupMenuEvent e) {
                    popup.removeAll();                               // 2. Rebuild
                    popup.add("Item 1");
                    popup.add("Item 2");
                    popup.revalidate();
                    popup.repaint();
                }
                public void popupMenuWillBecomeInvisible(PopupMenuEvent e) {}
                public void popupMenuCanceled(PopupMenuEvent e) {}
            });

            JButton button = new JButton("Right-click");
            button.setComponentPopupMenu(popup);

            frame.add(button);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Each time the popup is shown, it is repopulated with "Item 1" and "Item 2."

**Why the code produces this result.** The `popupMenuWillBecomeVisible` callback fires before the popup appears, allowing the menu to be rebuilt dynamically.

---

### Real-World Cases

- **“Recent Files” menus** populated on demand.
- **Context-sensitive menus** that show different items based on the current selection.
- **Menus that reflect live data** (e.g., open windows, connected devices).

---

## 9. JPopupMenu Best Practices

### Definitions

**Core Definition.** `JPopupMenu` best practices are the conventions and patterns that ensure popup menus are used safely, consistently, and accessibly in Swing applications.

**Technical Definition.** Best practices include: constructing and showing popups on the EDT; using `setComponentPopupMenu()` for simple right-click scenarios; using `Action` objects for shared command state; handling platform-specific popup triggers (`isPopupTrigger()`); populating menus dynamically via `PopupMenuListener`; and avoiding the reuse of the same `JMenuItem` across multiple popups (since a Swing component can only have one parent).

**Beginner-Friendly Explanation.** Best practices for popup menus are the “rules of thumb” that prevent common bugs: forgetting the EDT, using the wrong trigger condition, or trying to share menu items across multiple popups. They include things like “use `setComponentPopupMenu` for simple cases,” “check `isPopupTrigger()` for right-click,” and “rebuild the menu before it appears.”

---

### Purposes

- To maintain thread safety by respecting the EDT.
- To provide a consistent right-click experience across platforms.
- To avoid common pitfalls with component parentage and menu item reuse.
- To keep menus dynamic and responsive to application state.
- To improve accessibility and keyboard navigation.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JPopupMenu popup = new JPopupMenu();                          // 1. Popup
    popup.add(popupAction);                                        // 2. Actions
    popup.addSeparator();
    popup.addPopupMenuListener(new PopupMenuListener() {           // 3. Dynamic
        public void popupMenuWillBecomeVisible(PopupMenuEvent e) {
            // rebuild menu based on current state
        }
        public void popupMenuWillBecomeInvisible(PopupMenuEvent e) {}
        public void popupMenuCanceled(PopupMenuEvent e) {}
    });

    component.setComponentPopupMenu(popup);                        // 4. Attach
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT | `SwingUtilities.invokeLater(...)` |
| Simple right-click | `component.setComponentPopupMenu(popup)` |
| Custom trigger | `MouseListener` + `e.isPopupTrigger()` |
| Shared commands | `popup.add(action)` |
| Dynamic population | `PopupMenuListener.popupMenuWillBecomeVisible` |
| No menu item reuse | Create new `JMenuItem`s or use `Action` objects. |
| Lightweight vs. heavyweight | `setLightWeightPopupEnabled(false)` if needed |

#### Syntax Rules

- **Always construct and show popups on the EDT.**
- **Always use `isPopupTrigger()`** rather than hard-coding `MouseEvent.BUTTON3` for right-click detection; the trigger is platform-dependent.
- **Always use `Action` objects** when the same command appears in multiple menus or toolbars.
- **Never add the same `JMenuItem` to multiple popups**; a Swing component can only have one parent. Use separate items or shared `Action`s.
- **Prefer `setComponentPopupMenu()`** for simple right-click context menus; use a `MouseListener` with `show()` for custom triggers.

#### Constraints and Limitations

- Lightweight popups can be occluded by heavyweight AWT components; use `setLightWeightPopupEnabled(false)` to force a heavyweight popup when necessary.
- Popup menus are not suitable for very large item lists; consider a dialog or scrollable component instead.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Context Menu with Dynamic Population

```java
import javax.swing.*;
import javax.swing.event.PopupMenuEvent;
import javax.swing.event.PopupMenuListener;
import java.awt.event.ActionEvent;

public class BestPracticePopupMenu {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticePopupMenu::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Popup");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JPopupMenu popup = new JPopupMenu();                          // 1. Popup

        Action copyAction = new AbstractAction("Copy") {              // 2. Shared Action
            public void actionPerformed(ActionEvent e) {
                System.out.println("Copying...");
            }
        };
        popup.add(copyAction);

        popup.addPopupMenuListener(new PopupMenuListener() {           // 3. Dynamic
            public void popupMenuWillBecomeVisible(PopupMenuEvent e) {
                popup.removeAll();
                popup.add(copyAction);
                popup.addSeparator();
                popup.add(new JMenuItem("Paste"));
                popup.revalidate();
                popup.repaint();
            }
            public void popupMenuWillBecomeInvisible(PopupMenuEvent e) {}
            public void popupMenuCanceled(PopupMenuEvent e) {}
        });

        JTextArea textArea = new JTextArea();
        textArea.setComponentPopupMenu(popup);                         // 4. Attach

        frame.add(new JScrollPane(textArea));
        frame.setSize(400, 300);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Right-clicking the text area shows a popup with "Copy," a separator, and "Paste." The menu is rebuilt each time it appears.

**Why the code produces this result.** The `PopupMenuListener` rebuilds the menu in `popupMenuWillBecomeVisible`, ensuring the contents are always up to date. The `Action` is reused across rebuilds.

---

#### Example 2: Common Mistake — Adding the Same MenuItem to Multiple Popups

```java
import javax.swing.*;

public class BadPopupReuse {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Popup Reuse");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuItem sharedItem = new JMenuItem("Shared");           // 1. Shared item

            JPopupMenu popup1 = new JPopupMenu();
            popup1.add(sharedItem);                                    // 2. Add to 1
            JPopupMenu popup2 = new JPopupMenu();
            popup2.add(sharedItem);                                    // 3. Add to 2

            JButton b1 = new JButton("Button 1");
            b1.setComponentPopupMenu(popup1);
            JButton b2 = new JButton("Button 2");
            b2.setComponentPopupMenu(popup2);

            JPanel panel = new JPanel();
            panel.add(b1);
            panel.add(b2);
            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The `sharedItem` appears only in the second popup; it is automatically removed from the first popup when added to the second, because a Swing component can only have one parent.

**Why this is wrong.** A `JMenuItem` (or any Swing component) cannot belong to two containers simultaneously. Adding it to the second popup removes it from the first. The correct approach is to create separate `JMenuItem` instances or share `Action` objects instead.

---

### Real-World Cases

- **Production Swing applications** use `setComponentPopupMenu` for simple context menus and `PopupMenuListener` for dynamic content.
- **Cross-platform applications** rely on `isPopupTrigger()` to handle the different mouse buttons and modifier keys used for context menus on Windows, macOS, and Linux.
- **Educational examples** use the best-practice skeleton to demonstrate correct popup menu usage from the start.

---

## References

Oracle. *Class JPopupMenu*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JPopupMenu.html

Oracle. *Class JPopupMenu*. Java Development Kit Version 23 API Specification. https://docs.oracle.com/en/java/javase/23/docs/api/java.desktop/javax/swing/JPopupMenu.html

Oracle. *How to Use Menus*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/menu.html

Oracle. *Interface PopupMenuListener*. Java Platform, Standard Edition 11 API Specification (Japanese). https://docs.oracle.com/javase/jp/11/docs/api/java.desktop/javax/swing/event/PopupMenuListener.html

OpenJDK. *JPopupMenu.show(Component, int, int) documentation*. https://bugs.openjdk.org/ (JDK bug tracker reference for `show` coordinates).

Stack Overflow. *Stopping JPopupMenu stealing the focus*. https://stackoverflow.com/questions/ (discussion of `setFocusable(false)` for popups).

Stack Overflow. *JMenuItem reuse across popups*. https://stackoverflow.com/questions/ (component parentage limitation).

Stack Overflow. *JPopupMenu and lightweight/heavyweight popups*. https://stackoverflow.com/questions/ (lightweight vs heavyweight discussion).