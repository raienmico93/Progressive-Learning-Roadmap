# Java Swing Menu System — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** The Swing menu system is a set of cooperating components that provide a space-saving, hierarchical way to present the user with a list of actions or options. A **JMenuBar** is the container that holds the top-level menus (typically displayed horizontally across the top of a window). A **JMenu** is a clickable menu that, when activated, reveals a **JPopupMenu** containing menu items. A **JMenuItem** is the actual button-like command that the user selects to trigger an action. Specialized menu items — **JCheckBoxMenuItem** and **JRadioButtonMenuItem** — provide toggle and mutually-exclusive selection states, respectively. A **JSeparator** provides a visual dividing line that breaks the menu items into logical groups.

**Technical Definition.** `JMenuBar` extends `JComponent` and is the implementation of a menu bar. It is designed to be set on a `JFrame` (or other `RootPaneContainer`) via `setJMenuBar`; the frame's root pane then positions it along the upper edge of the window. `JMenu` extends `JMenuItem` and represents a menu that can contain other menu items and submenus. `JMenuItem` extends `AbstractButton` and is the base class for items that can be selected from a menu. `JCheckBoxMenuItem` and `JRadioButtonMenuItem` extend `JMenuItem` and provide toggle and radio-button selection semantics respectively. `JSeparator` extends `JComponent` and provides a general-purpose divider; it is most commonly used between menu items, though it can also be used elsewhere in a GUI. Swing is not thread-safe; all menu construction and manipulation must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** Think of the Swing menu system as a filing cabinet. The **JMenuBar** is the cabinet itself, mounted at the top of your window. Inside the cabinet are labeled drawers — these are the **JMenu** objects (File, Edit, View, Help). Pulling open a drawer reveals folders and documents inside — these are the **JMenuItem** objects (Open, Save, Copy, Paste). Some folders have checkboxes or radio buttons attached (**JCheckBoxMenuItem** and **JRadioButtonMenuItem**) so you can toggle settings like “Show Toolbar” or choose one font size from a mutually exclusive set. A thin divider line (**JSeparator**) groups related items. The user navigates this hierarchy to trigger the commands your application provides.

---

### Key Characteristics

1. **Hierarchical organisation.** `JMenuBar` → `JMenu` → `JMenuItem` (and optionally nested `JMenu` submenus) provides an arbitrarily deep hierarchy of commands.
2. **`JMenu` is a `JMenuItem`.** Because `JMenu` extends `JMenuItem`, a `JMenu` can be added anywhere a `JMenuItem` is accepted — this is how submenus are built.
3. **Popup-driven.** A `JMenu` does not display its contents until activated; it uses a `JPopupMenu` to show its items.
4. **Selection semantics.** `JCheckBoxMenuItem` toggles between selected and deselected states; `JRadioButtonMenuItem` is part of a mutually exclusive group (typically managed by a `ButtonGroup`).
5. **Separators group items.** `JSeparator` (usually added via `addSeparator()`) visually breaks items into logical clusters.
6. **Keyboard accelerators and mnemonics.** Menu items support accelerators (global keyboard shortcuts) and mnemonics (Alt-key navigation), while `JMenu` supports only mnemonics — `setAccelerator` is not defined for `JMenu`.
7. **Single-thread rule.** All menu components must be created and manipulated on the EDT.
8. **Serialization caveat.** Serialized menu objects are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with `JFrame`, `JPanel`, and the Swing container hierarchy.
- Understanding of `Action` and `ActionListener`.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- A JDK (version 8 or later; all menu classes are in `javax.swing`).

---

### Related Programming Areas

- **`Action`:** The preferred way to encapsulate menu item functionality; `JMenuItem` can be created from an `Action`.
- **`JPopupMenu`:** The popup container used by `JMenu` to display its items.
- **`ButtonGroup`:** Used with `JRadioButtonMenuItem` to enforce mutual exclusivity.
- **`KeyStroke` and `InputEvent`:** Used for accelerators and mnemonics.
- **`MenuListener` and `MenuEvent`:** Observe menu selection and cancellation events.
- **`JToolBar`:** Complements menus by providing quick-access buttons for common commands.

---

## Core Concepts and Features

---

## 1. JMenuBar — The Menu Bar

### Definitions

**Core Definition.** `JMenuBar` is a component that implements a menu bar — a horizontal (or platform-appropriate) container for `JMenu` objects. It is set on a `JFrame` or other `RootPaneContainer` via `setJMenuBar()`.

**Technical Definition.** `public class JMenuBar extends JComponent implements Accessible, MenuElement`. It provides constructors: `JMenuBar()`. Menus are added with `add(JMenu)` or `insert(JMenu, int)`. The menu bar is positioned by the root pane of the frame; it is never added to a `JPanel` or the content pane directly.

**Beginner-Friendly Explanation.** The `JMenuBar` is the horizontal strip at the top of the window that holds “File,” “Edit,” “View,” and similar menus.

---

### Purposes

- To provide a top-level container for an application's menus.
- To serve as the entry point for menu navigation.
- To integrate with the frame's root pane for proper positioning and platform integration.
- To support multiple menus (File, Edit, Help, etc.) in a single bar.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JMenuBar menuBar = new JMenuBar();
frame.setJMenuBar(menuBar);
JMenu fileMenu = new JMenu("File");
menuBar.add(fileMenu);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new JMenuBar()` | Creates an empty menu bar. |
| `frame.setJMenuBar(menuBar)` | Attaches the menu bar to the frame's root pane. |
| `menuBar.add(JMenu)` | Appends a menu to the bar. |
| `menuBar.insert(JMenu, int)` | Inserts a menu at the specified index. |

#### Syntax Rules

- The menu bar must be set on a `JFrame` (or `JDialog`, `JInternalFrame`) via `setJMenuBar`, **not** added to the content pane with `add()`. Adding a `JMenuBar` to a `JPanel` is a common mistake.
- The `JMenuBar` manages its own layout; do not set a layout manager on it.
- By default, `focusTraversalKeysEnabled` is `false`; call `setFocusTraversalKeysEnabled(true)` if Tab should move focus away from the menu bar.

#### Constraints and Limitations

- A `JMenuBar` can only be attached to one container at a time.
- On macOS, the menu bar may be rendered by the OS at the top of the screen rather than inside the window.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JMenuBar with One Menu

```java
import javax.swing.*;
import java.awt.*;

public class MinimalMenuBar {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal MenuBar");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar menuBar = new JMenuBar();                     // 1. Create bar
            JMenu fileMenu = new JMenu("File");                   // 2. Create menu
            menuBar.add(fileMenu);                                 // 3. Add menu to bar
            frame.setJMenuBar(menuBar);                            // 4. Attach to frame

            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a menu bar at the top containing a single “File” menu. Clicking “File” opens an empty popup menu.

**Why the code produces this result.** `setJMenuBar` attaches the bar to the frame's root pane, which positions it along the upper edge. The `add` call places the `JMenu` inside the bar.

---

### Real-World Cases

- **Standard desktop applications** (text editors, IDEs, calculators) with File, Edit, View, Help menus.
- **Multi-window applications** where the menu bar is shared across frames (with platform-specific handling).

---

## 2. JMenu — The Menu

### Definitions

**Core Definition.** `JMenu` is a clickable menu component that, when activated, displays a `JPopupMenu` containing its child menu items and submenus.

**Technical Definition.** `public class JMenu extends JMenuItem implements Accessible, MenuElement`. Because it extends `JMenuItem`, a `JMenu` can be nested inside another `JMenu` to create submenus. Items are added with `add(JMenuItem)` or `add(Action)`; separators with `addSeparator()`. `JMenu` supports mnemonics (`setMnemonic`) but **not** accelerators — `setAccelerator` is not defined for `JMenu` and throws `Error` if called.

**Beginner-Friendly Explanation.** A `JMenu` is one of the labeled drop-down menus (“File,” “Edit”) that lives inside the menu bar.

---

### Purposes

- To group related menu items under a single label.
- To support nested submenus for deeper hierarchies.
- To provide keyboard mnemonics for quick menu access.
- To serve as the primary organisational unit of the menu system.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JMenu fileMenu = new JMenu("File");
fileMenu.setMnemonic(KeyEvent.VK_F);
JMenuItem openItem = new JMenuItem("Open");
fileMenu.add(openItem);
fileMenu.addSeparator();
JMenu subMenu = new JMenu("Recent");
fileMenu.add(subMenu); // submenu
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `new JMenu(String)` | Creates a menu with the specified label. |
| `add(JMenuItem)` | Adds an item or submenu. |
| `add(Action)` | Adds a menu item backed by an `Action`. |
| `addSeparator()` | Appends a `JSeparator`. |
| `setMnemonic(int)` | Sets the Alt-key mnemonic. |

#### Syntax Rules

- Use `setMnemonic` for keyboard access; do **not** call `setAccelerator` on a `JMenu`.
- Submenus are created by adding a `JMenu` to another `JMenu`.

#### Constraints and Limitations

- Accelerators are supported only on `JMenuItem` (and its subclasses), not on `JMenu`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: JMenu with Items and a Separator

```java
import javax.swing.*;
import java.awt.event.KeyEvent;

public class MenuWithItems {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Menu with Items");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            file.setMnemonic(KeyEvent.VK_F);                       // 1. Mnemonic

            JMenuItem open = new JMenuItem("Open");                // 2. Item
            JMenuItem save = new JMenuItem("Save");
            file.add(open);
            file.add(save);
            file.addSeparator();                                    // 3. Separator
            JMenuItem exit = new JMenuItem("Exit");
            file.add(exit);

            bar.add(file);
            frame.setJMenuBar(bar);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A “File” menu with “Open,” “Save,” a separator, and “Exit.” Alt+F activates the menu.

**Why the code produces this result.** `setMnemonic` underlines the “F” for Alt+F access. `addSeparator` inserts a visual divider.

---

### Real-World Cases

- **File menus** (Open, Save, Exit), **Edit menus** (Cut, Copy, Paste), **View menus** (Zoom, Toolbars).
- **Submenus** for “Recent Files,” “Export As,” or “Font Size.”

---

## 3. JMenuItem — The Menu Item

### Definitions

**Core Definition.** `JMenuItem` is an implementation of an item in a menu — a button-like component that fires an `ActionEvent` when selected by the user.

**Technical Definition.** `public class JMenuItem extends AbstractButton implements Accessible, MenuElement`. It supports text, icons, mnemonics, and accelerators. `JMenuItem` can also be constructed from an `Action`, which centralises the item's text, icon, enabled state, and accelerator.

**Beginner-Friendly Explanation.** A `JMenuItem` is one of the actual commands inside a menu — “Open,” “Save,” “Copy,” etc.

---

### Purposes

- To represent a selectable command within a menu.
- To fire an `ActionEvent` when activated, triggering application logic.
- To display keyboard accelerators for quick access.
- To serve as the base class for specialised items (`JCheckBoxMenuItem`, `JRadioButtonMenuItem`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JMenuItem item = new JMenuItem("Open");
item.setMnemonic(KeyEvent.VK_O);
item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_O, InputEvent.CTRL_DOWN_MASK));
item.addActionListener(e -> openFile());
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `new JMenuItem(String)` | Creates an item with the specified text. |
| `setMnemonic(int)` | Sets the Alt-key mnemonic. |
| `setAccelerator(KeyStroke)` | Sets a global keyboard shortcut. |
| `addActionListener(ActionListener)` | Handles activation. |

#### Syntax Rules

- Accelerators are global shortcuts that work even when the menu is not open.
- Mnemonics work only while the menu is visible (Alt+key to open the menu, then key to select).
- Use `Action` objects (`new JMenuItem(action)`) to share state between menu items and tool bar buttons.

#### Constraints and Limitations

- Accelerators must be unique within the application window to avoid conflicts.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: JMenuItem with Accelerator

```java
import javax.swing.*;
import java.awt.event.*;

public class AcceleratorItem {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Accelerator");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            JMenuItem save = new JMenuItem("Save");                // 1. Item
            save.setAccelerator(KeyStroke.getKeyStroke(            // 2. Accelerator
                KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));
            save.addActionListener(e ->                            // 3. Action
                System.out.println("Saved!"));
            file.add(save);
            bar.add(file);
            frame.setJMenuBar(bar);

            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Ctrl+S (or selecting “Save” from the File menu) prints “Saved!” to the console.

**Why the code produces this result.** `setAccelerator` binds Ctrl+S to the item. The `ActionListener` handles activation.

---

### Real-World Cases

- **Every application command:** Open, Save, Print, Cut, Copy, Paste, Undo, Redo, Find, Replace.

---

## 4. JCheckBoxMenuItem — The Check Box Menu Item

### Definitions

**Core Definition.** `JCheckBoxMenuItem` is a menu item that can be selected or deselected. When selected, it typically appears with a checkmark next to it.

**Technical Definition.** `public class JCheckBoxMenuItem extends JMenuItem implements SwingConstants, Accessible`. It supports `isSelected()` and `setSelected(boolean)` (preferred) as well as the older `getState()`/`setState(boolean)` methods.

**Beginner-Friendly Explanation.** A `JCheckBoxMenuItem` is a menu item with a checkbox — you can toggle it on and off (e.g., “Show Toolbar” or “Word Wrap”).

---

### Purposes

- To toggle a boolean setting from within a menu.
- To provide a visual indication of whether an option is enabled.
- To complement `JCheckBox` controls in the main UI.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JCheckBoxMenuItem showToolbar = new JCheckBoxMenuItem("Show Toolbar", true);
showToolbar.addItemListener(e -> {
    boolean selected = (e.getStateChange() == ItemEvent.SELECTED);
    toolbar.setVisible(selected);
});
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `new JCheckBoxMenuItem(String, boolean)` | Creates a check box item with initial state. |
| `isSelected()` / `setSelected(boolean)` | Preferred state accessors. |

#### Syntax Rules

- Use `isSelected`/`setSelected` (inherited from `AbstractButton`), not the deprecated `getState`/`setState`.
- Register an `ItemListener` (not `ActionListener`) to react to state changes.

#### Constraints and Limitations

- Check box items are independent; multiple can be selected simultaneously.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Toggle Menu Item

```java
import javax.swing.*;
import java.awt.event.ItemEvent;

public class CheckBoxMenuItemDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("CheckBox MenuItem");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu view = new JMenu("View");

            JCheckBoxMenuItem wrap = new JCheckBoxMenuItem("Word Wrap", true); // 1. Initial
            wrap.addItemListener(e -> {                            // 2. ItemListener
                boolean on = (e.getStateChange() == ItemEvent.SELECTED);
                System.out.println("Word wrap: " + on);
            });
            view.add(wrap);
            bar.add(view);
            frame.setJMenuBar(bar);

            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “Word Wrap” item starts checked. Clicking it toggles the checkmark and prints “Word wrap: true/false.”

**Why the code produces this result.** The `ItemListener` fires on state changes, reading `e.getStateChange()`.

---

### Real-World Cases

- **View menus** with “Show Status Bar,” “Show Toolbar,” “Word Wrap.”
- **Debug menus** with “Enable Logging.”

---

## 5. JRadioButtonMenuItem — The Radio Button Menu Item

### Definitions

**Core Definition.** `JRadioButtonMenuItem` is a menu item that is part of a group in which only one item can be selected at a time.

**Technical Definition.** `public class JRadioButtonMenuItem extends JMenuItem implements Accessible`. It is typically added to a `ButtonGroup` to enforce mutual exclusivity. The selected item displays its selected state; selecting another item in the group deselects the previous one.

**Beginner-Friendly Explanation.** A `JRadioButtonMenuItem` is a menu item with a radio button — only one in the group can be selected (e.g., choosing one font size).

---

### Purposes

- To choose one option from a mutually exclusive set within a menu.
- To provide a visual indicator of the current selection.
- To manage settings like font size, zoom level, or view mode.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
ButtonGroup group = new ButtonGroup();
JRadioButtonMenuItem small = new JRadioButtonMenuItem("Small");
JRadioButtonMenuItem medium = new JRadioButtonMenuItem("Medium", true);
JRadioButtonMenuItem large = new JRadioButtonMenuItem("Large");
group.add(small);
group.add(medium);
group.add(large);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `ButtonGroup` | Enforces mutual exclusivity. |
| `JRadioButtonMenuItem(String, boolean)` | Creates an item with initial state. |

#### Syntax Rules

- All items in the group must be added to the `ButtonGroup`; otherwise, they behave like independent check boxes.
- Use `isSelected()` to read the current selection.

#### Constraints and Limitations

- Only one item per group can be selected.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Radio Button Menu Group

```java
import javax.swing.*;

public class RadioMenuItemDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Radio MenuItem");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu size = new JMenu("Font Size");
            ButtonGroup group = new ButtonGroup();                 // 1. Group

            JRadioButtonMenuItem small = new JRadioButtonMenuItem("Small");
            JRadioButtonMenuItem medium = new JRadioButtonMenuItem("Medium", true); // 2. Default
            JRadioButtonMenuItem large = new JRadioButtonMenuItem("Large");

            group.add(small);
            group.add(medium);
            group.add(large);
            size.add(small);
            size.add(medium);
            size.add(large);

            bar.add(size);
            frame.setJMenuBar(bar);

            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A “Font Size” menu with “Small,” “Medium” (selected by default), and “Large.” Only one can be selected at a time.

**Why the code produces this result.** The `ButtonGroup` enforces exclusivity.

---

### Real-World Cases

- **Font size selection** (Small, Medium, Large).
- **View modes** (Normal, Compact, Full Screen).
- **Zoom levels** (50%, 75%, 100%, 150%).

---

## 6. JSeparator — The Menu Separator

### Definitions

**Core Definition.** `JSeparator` is a general-purpose component that implements a divider line, most commonly used to break menu items into logical groupings.

**Technical Definition.** `public class JSeparator extends JComponent implements SwingConstants, Accessible`. Instead of using `JSeparator` directly, you can use `JMenu.addSeparator()` or `JPopupMenu.addSeparator()`, which create and add a separator at the end of the menu. `insertSeparator(int)` inserts at a specific position.

**Beginner-Friendly Explanation.** A `JSeparator` is the thin line you see between groups of related menu items (e.g., between “Cut/Copy/Paste” and “Select All”).

---

### Purposes

- To visually group related menu items.
- To improve menu readability.
- To break up long menus into logical sections.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
menu.addSeparator();          // append at the end
menu.insertSeparator(2);      // insert at index 2
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addSeparator()` | Appends a separator. |
| `insertSeparator(int)` | Inserts a separator at the given index. |

#### Syntax Rules

- Use `addSeparator()` on `JMenu` or `JPopupMenu` rather than creating `JSeparator` directly.
- Separators do not respond to user interaction.

#### Constraints and Limitations

- Excessive separators can fragment the menu visually.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Separator Between Groups

```java
import javax.swing.*;

public class SeparatorDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Separator");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu edit = new JMenu("Edit");
            edit.add(new JMenuItem("Cut"));
            edit.add(new JMenuItem("Copy"));
            edit.add(new JMenuItem("Paste"));
            edit.addSeparator();                                   // 1. Separator
            edit.add(new JMenuItem("Select All"));

            bar.add(edit);
            frame.setJMenuBar(bar);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** An “Edit” menu with Cut, Copy, Paste, a separator line, and Select All.

**Why the code produces this result.** `addSeparator` inserts a visual divider before “Select All.”

---

### Real-World Cases

- **Edit menus** separating clipboard operations from selection operations.
- **File menus** separating “Exit” from the save/open group.

---

## 7. Menu Methods — add() / insert() / remove()

### Definitions

**Core Definition.** `add(Component)` appends a menu item, submenu, or separator to a `JMenu` or `JMenuBar`. `insert(Component, int)` inserts at a specific position. `remove(Component)` removes the item.

**Technical Definition.** `JMenu` provides `add(JMenuItem)`, `add(String)`, `add(Action)`, and `insert(JMenuItem, int)`. `JMenuBar` provides `add(JMenu)` and `insert(JMenu, int)`. Removal is handled by `remove(Component)` or `remove(int index)`.

**Beginner-Friendly Explanation.** `add` puts a new item at the end; `insert` puts it at a specific position; `remove` takes it out.

---

### Purposes

- To build the menu hierarchy.
- To dynamically modify menus at runtime.
- To reorder items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
menu.add(item);
menu.insert(item, 0);
menu.remove(item);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `add(Component)` | Appends to the end. |
| `insert(Component, int)` | Inserts at the index. |
| `remove(Component)` | Removes the item. |

#### Syntax Rules

- After dynamic changes, the menu should be revalidated (`revalidate()`) and repainted (`repaint()`).

#### Constraints and Limitations

- Indices shift after insertions and removals.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Insertion

```java
import javax.swing.*;

public class InsertDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Insert");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            file.add(new JMenuItem("Open"));
            file.add(new JMenuItem("Exit"));
            file.insert(new JMenuItem("Save"), 1);                 // 1. Insert

            bar.add(file);
            frame.setJMenuBar(bar);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A File menu with Open, Save, and Exit (Save inserted between Open and Exit).

**Why the code produces this result.** `insert(item, 1)` places Save at index 1.

---

### Real-World Cases

- **Dynamic menus** that add “Recent Files” entries.
- **Context-sensitive menus** that add or remove items based on state.

---

## 8. Menu Methods — setMnemonic() / setAccelerator()

### Definitions

**Core Definition.** `setMnemonic(int)` sets an Alt-key shortcut that activates the menu or item when the menu is visible. `setAccelerator(KeyStroke)` sets a global keyboard shortcut that activates the item without opening the menu.

**Technical Definition.** `JMenuItem` supports both mnemonics and accelerators. `JMenu` supports only mnemonics; calling `setAccelerator` on a `JMenu` throws an `Error`. Mnemonics are typically Alt+key; accelerators are typically Ctrl+key or other modifier combinations.

**Beginner-Friendly Explanation.** A mnemonic is the underlined letter you press with Alt to open a menu. An accelerator is a global shortcut like Ctrl+S that works even when the menu is closed.

---

### Purposes

- To provide keyboard access to menus and items.
- To speed up common commands via accelerators.
- To support accessibility and power-user workflows.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
menu.setMnemonic(KeyEvent.VK_F);
item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setMnemonic(int)` | Sets the Alt-key mnemonic (for `JMenu` and `JMenuItem`). |
| `setAccelerator(KeyStroke)` | Sets a global shortcut (for `JMenuItem` only). |

#### Syntax Rules

- Mnemonics work while the menu is visible; accelerators work globally.
- Do not call `setAccelerator` on a `JMenu`; use `setMnemonic` instead.

#### Constraints and Limitations

- Accelerators must be unique to avoid conflicts.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Mnemonic and Accelerator

```java
import javax.swing.*;
import java.awt.event.*;

public class MnemonicAccelerator {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Mnemonic & Accelerator");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            file.setMnemonic(KeyEvent.VK_F);                        // 1. Mnemonic
            JMenuItem save = new JMenuItem("Save");
            save.setAccelerator(KeyStroke.getKeyStroke(             // 2. Accelerator
                KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));
            file.add(save);
            bar.add(file);
            frame.setJMenuBar(bar);

            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Alt+F opens the File menu; Ctrl+S activates Save regardless of focus.

**Why the code produces this result.** `setMnemonic` enables Alt+F; `setAccelerator` binds Ctrl+S globally.

---

### Real-World Cases

- **File menus** with Alt+F; **Save** with Ctrl+S; **Undo** with Ctrl+Z.

---

## 9. Menu Methods — setEnabled()

### Definitions

**Core Definition.** `setEnabled(boolean)` controls whether a menu item can be selected by the user.

**Technical Definition.** `setEnabled(boolean b)` is inherited from `AbstractButton`. A disabled menu item is typically rendered in a grayed-out state and does not respond to clicks or keyboard activation.

**Beginner-Friendly Explanation.** Disabling a menu item greys it out and makes it unclickable.

---

### Purposes

- To indicate that a command is currently unavailable.
- To prevent invalid actions.
- To reflect application state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
item.setEnabled(false);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setEnabled(boolean)` | `false` disables the item. |

#### Syntax Rules

- Disabled items can still be manipulated programmatically.

#### Constraints and Limitations

- The visual appearance is look-and-feel-dependent.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling a Menu Item

```java
import javax.swing.*;

public class DisableDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Disable");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            JMenuItem save = new JMenuItem("Save");
            save.setEnabled(false);                                // 1. Disable
            file.add(save);
            bar.add(file);
            frame.setJMenuBar(bar);

            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “Save” item appears greyed out and cannot be clicked.

**Why the code produces this result.** `setEnabled(false)` disables the item.

---

### Real-World Cases

- **Edit menus** where “Paste” is disabled when the clipboard is empty.
- **File menus** where “Save” is disabled when there are no changes.

---

## 10. Menu Events — ActionListener

### Definitions

**Core Definition.** `ActionListener` is the interface for receiving `ActionEvent`s fired when a `JMenuItem` (or subclass) is selected.

**Technical Definition.** `public interface ActionListener extends EventListener`. The `actionPerformed(ActionEvent e)` method is invoked when the item is activated by mouse or keyboard. `JMenuItem` does **not** support `MenuListener` directly — `MenuListener` is for `JMenu` selection events.

**Beginner-Friendly Explanation.** An `ActionListener` runs when the user clicks a menu item (or presses its accelerator).

---

### Purposes

- To execute application logic when a command is chosen.
- To handle both mouse and keyboard activation.
- To centralise command handling via `Action`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
item.addActionListener(e -> { /* command */ });
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addActionListener(ActionListener)` | Registers the listener. |
| `actionPerformed(ActionEvent)` | Called on activation. |

#### Syntax Rules

- The listener is invoked on the EDT.

#### Constraints and Limitations

- `JMenuItem` does not support `MenuListener`; use `ActionListener` for item activation.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Handling Menu Item Action

```java
import javax.swing.*;

public class ActionListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ActionListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            JMenuItem exit = new JMenuItem("Exit");
            exit.addActionListener(e -> {                          // 1. Listener
                System.out.println("Exiting...");
                System.exit(0);
            });
            file.add(exit);
            bar.add(file);
            frame.setJMenuBar(bar);

            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting “Exit” prints “Exiting...” and terminates the JVM.

**Why the code produces this result.** The `ActionListener` handles the activation.

---

### Real-World Cases

- **Every menu command** — Open, Save, Copy, Paste, Exit.

---

## 11. Menu Events — MenuListener

### Definitions

**Core Definition.** `MenuListener` is the interface for receiving `MenuEvent`s that fire when a `JMenu` is selected, deselected, or canceled.

**Technical Definition.** `public interface MenuListener extends EventListener`. It declares `menuSelected(MenuEvent e)`, `menuDeselected(MenuEvent e)`, and `menuCanceled(MenuEvent e)`. It is registered on a `JMenu` via `addMenuListener`. `JMenuItem` does not fire `MenuEvent`s.

**Beginner-Friendly Explanation.** A `MenuListener` runs when a menu is opened or closed. You can use it to populate the menu dynamically.

---

### Purposes

- To react when a menu is about to become visible (`menuSelected`).
- To react when a menu is closed without a selection (`menuCanceled`).
- To dynamically populate or update menu items before display.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
menu.addMenuListener(new MenuListener() {
    public void menuSelected(MenuEvent e) { /* populate */ }
    public void menuDeselected(MenuEvent e) { }
    public void menuCanceled(MenuEvent e) { }
});
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `menuSelected` | The menu is about to become visible. |
| `menuDeselected` | The menu is no longer selected. |
| `menuCanceled` | The menu was closed without a selection. |

#### Syntax Rules

- `MenuListener` is registered on a `JMenu`, not on a `JMenuItem`.

#### Constraints and Limitations

- `JMenu` does not fire `ActionEvent`; use `MenuListener` for menu-level events.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Populating a Menu on Open

```java
import javax.swing.*;
import javax.swing.event.MenuEvent;
import javax.swing.event.MenuListener;

public class MenuListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("MenuListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            JMenu recent = new JMenu("Recent");
            recent.addMenuListener(new MenuListener() {             // 1. Register
                public void menuSelected(MenuEvent e) {
                    recent.removeAll();                             // 2. Populate
                    recent.add(new JMenuItem("File1.txt"));
                    recent.add(new JMenuItem("File2.txt"));
                    recent.revalidate();
                    recent.repaint();
                }
                public void menuDeselected(MenuEvent e) {}
                public void menuCanceled(MenuEvent e) {}
            });
            bar.add(recent);
            frame.setJMenuBar(bar);

            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Each time the “Recent” menu is opened, it is repopulated with “File1.txt” and “File2.txt.”

**Why the code produces this result.** `menuSelected` fires before the menu becomes visible, allowing dynamic population.

---

### Real-World Cases

- **“Recent Files” menus** populated on demand.
- **Context-sensitive menus** that show different items based on application state.

---

## 12. Menu Best Practices

### Definitions

**Core Definition.** Menu best practices are the conventions and patterns that ensure menus are constructed safely, consistently, and accessibly in Swing applications.

**Technical Definition.** Best practices include: constructing and manipulating all menu components on the EDT; attaching the `JMenuBar` to the frame via `setJMenuBar()` (never adding it to a `JPanel`); using `Action` objects to share state between menu items and tool bar buttons; providing mnemonics and accelerators; using `MenuListener` for `JMenu`-level events and `ActionListener` for `JMenuItem` activation; and avoiding `setAccelerator` on `JMenu` (use `setMnemonic` instead).

**Beginner-Friendly Explanation.** Best practices are the “rules of thumb” that prevent common menu bugs: adding the menu bar to the wrong container, forgetting the EDT, mixing up `MenuListener` and `ActionListener`, or trying to set an accelerator on a `JMenu`.

---

### Purposes

- To maintain thread safety and avoid race conditions.
- To ensure the menu bar is positioned correctly by the frame's root pane.
- To provide consistent keyboard access through mnemonics and accelerators.
- To centralise command state through `Action` objects.
- To use the correct event listener for each component.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JMenuBar bar = new JMenuBar();                                // 1. Menu bar
    JMenu file = new JMenu("File");                               // 2. Menu
    file.setMnemonic(KeyEvent.VK_F);                              // 3. Mnemonic
    JMenuItem save = new JMenuItem(saveAction);                   // 4. Action
    save.setAccelerator(KeyStroke.getKeyStroke(
        KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));               // 5. Accelerator
    file.add(save);
    bar.add(file);
    frame.setJMenuBar(bar);                                       // 6. Attach
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT | `SwingUtilities.invokeLater(...)` |
| Attach to frame | `frame.setJMenuBar(bar)` — never `panel.add(bar)` |
| Use `Action` | `new JMenuItem(action)` |
| Mnemonic on `JMenu` | `setMnemonic(KeyEvent.VK_F)` |
| Accelerator on `JMenuItem` | `setAccelerator(KeyStroke)` |
| `MenuListener` for `JMenu` | `addMenuListener` |
| `ActionListener` for `JMenuItem` | `addActionListener` |
| Don't use `setAccelerator` on `JMenu` | Throws `Error` |

#### Syntax Rules

- **Always set the menu bar with `setJMenuBar`** on the frame. Do not add it to a `JPanel`.
- **Always use `Action` objects** when the same command appears in menus and tool bars.
- **Always provide mnemonics** for `JMenu` and `JMenuItem`.
- **Always use `MenuListener` for `JMenu`** and `ActionListener` for `JMenuItem`.
- **Never call `setAccelerator` on a `JMenu`**; use `setMnemonic` instead.

#### Constraints and Limitations

- `JMenu` does not support accelerators.
- Menu bars are platform-sensitive (e.g., macOS renders them in the system menu bar).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Menu with Action and Mnemonic

```java
import javax.swing.*;
import java.awt.event.*;

public class BestPracticeMenu {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeMenu::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Menu");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        Action saveAction = new AbstractAction("Save") {           // 1. Action
            public void actionPerformed(ActionEvent e) {
                System.out.println("Saved via action.");
            }
        };
        saveAction.putValue(Action.ACCELERATOR_KEY,
            KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));
        saveAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);

        JMenuBar bar = new JMenuBar();                              // 2. Bar
        JMenu file = new JMenu("File");
        file.setMnemonic(KeyEvent.VK_F);                            // 3. Mnemonic
        JMenuItem saveItem = new JMenuItem(saveAction);             // 4. From Action
        file.add(saveItem);
        bar.add(file);
        frame.setJMenuBar(bar);                                     // 5. Attach

        frame.setSize(350, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A File menu with a Save item (Ctrl+S accelerator, Alt+S mnemonic). Selecting Save prints “Saved via action.”

**Why the code produces this result.** The `Action` centralises the command, accelerator, and mnemonic. The menu bar is properly attached to the frame.

---

#### Example 2: Common Mistake — Adding Menu Bar to a JPanel

```java
import javax.swing.*;

public class BadMenuBar {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad MenuBar");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JMenuBar bar = new JMenuBar();
            bar.add(new JMenu("File"));

            JPanel panel = new JPanel();                             // WRONG
            panel.add(bar);                                          // 1. Add to panel
            frame.add(panel);

            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The menu bar may appear inside the panel but will not behave correctly as a frame menu bar (no proper positioning, no root-pane integration).

**Why this is wrong.** The `JMenuBar` must be set on the frame with `setJMenuBar(bar)`. Adding it to a `JPanel` bypasses the root pane's layout and platform integration.

---

### Real-World Cases

- **Production Swing applications** use `Action` objects for menu items and tool bar buttons, attach the menu bar via `setJMenuBar`, and provide mnemonics and accelerators for every command.
- **Cross-platform applications** account for macOS menu-bar behaviour and focus-traversal settings.
- **Educational examples** use the best-practice skeleton to demonstrate correct menu construction from the start.

---

## References

Oracle. *Class JMenuBar*. Java Platform, Standard Edition 26 API Specification. https://docs.oracle.com/en/java/javase/26/docs/api/java.desktop/javax/swing/JMenuBar.html

Oracle. *How to Use Menus*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/menu.html

Oracle. *Class JMenuItem*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JMenuItem.html

Oracle. *Class JCheckBoxMenuItem*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JCheckBoxMenuItem.html

Oracle. *Class JRadioButtonMenuItem*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JRadioButtonMenuItem.html

Oracle. *Class JSeparator*. Java Platform, Standard Edition 27 API Specification. https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/JSeparator.html

Oracle. *Interface MenuListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/MenuListener.html

Stack Overflow. *How to set an accelerator for a JMenu sub menu?*. https://stackoverflow.com/questions/12909851/how-to-set-an-accelerator-for-a-jmenu-sub-menu

Stack Overflow. *JMenuBar should not be added to a JPanel*. https://stackoverflow.com/questions/35819229/

Oracle. *Class JPopupMenu*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JPopupMenu.html

Oracle. *Class ButtonGroup*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ButtonGroup.html