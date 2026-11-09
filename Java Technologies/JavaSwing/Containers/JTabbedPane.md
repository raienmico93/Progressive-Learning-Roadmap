# JTabbedPane — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JTabbedPane` is a Swing component that lets the user switch between a group of components by clicking on a tab with a given title and/or icon. It implements the familiar “tabbed folder” metaphor, where multiple components share the same display space and only the component associated with the selected tab is visible.

**Technical Definition.** `public class JTabbedPane extends JComponent implements Serializable, Accessible, SwingConstants`. Tabs and components are added to a `JTabbedPane` object using the `addTab` and `insertTab` methods. Each tab is represented by an index corresponding to its position, where the first tab has an index of 0 and the last tab has an index equal to the tab count minus 1. The `JTabbedPane` uses a `SingleSelectionModel` to represent the set of tab indices and the currently selected index. If the tab count is greater than 0, there is always a selected index, initialized to the first tab by default. If the tab count is 0, the selected index is -1. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JTabbedPane` is like a physical file folder with labeled tabs. Imagine a folder with tabs for “Home,” “Work,” and “Personal.” When you pull the “Home” tab, you see the home documents; when you pull the “Work” tab, the content changes to work documents—all within the same folder. In a GUI, a `JTabbedPane` lets you place multiple panels in the same window area and gives the user a set of clickable tabs to switch between them. It is the standard way to organize related but distinct views in a compact space, such as a settings dialog with “General,” “Appearance,” and “Advanced” tabs.

---

### Key Characteristics

1. **Single-selection model.** `JTabbedPane` uses a `SingleSelectionModel`. Exactly one tab is selected at a time when the tab count is greater than zero. The selected index is initialized to the first tab by default.
2. **Index-based identification.** Tabs are identified by integer indices, starting at 0. The order of indices corresponds to the order in which tabs were added, although `insertTab` allows insertion at a specific index.
3. **Tab placement.** Tabs can be placed at the top, bottom, left, or right of the content area using `setTabPlacement(int)`. The default is `TOP`.
4. **Tab layout policy.** When there are more tabs than can fit in a single run, the layout policy determines whether tabs wrap to multiple rows (`WRAP_TAB_LAYOUT`) or are scrolled (`SCROLL_TAB_LAYOUT`). The default is `WRAP_TAB_LAYOUT`.
5. **Custom tab components.** Instead of the look-and-feel-rendered text and icon, a custom `Component` can be set as the tab’s renderer via `setTabComponentAt`. This allows complex tab headers with close buttons, animations, or other widgets.
6. **Change events.** A `ChangeListener` can be registered with `addChangeListener` to be notified whenever the selected tab changes.
7. **Single-thread rule.** `JTabbedPane` is not thread-safe. All access must occur on the EDT.
8. **Serialization caveat.** Serialized `JTabbedPane` objects are not compatible with future Swing releases; serialization is appropriate only for short-term storage or RMI between applications running the same Swing version.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, anonymous inner classes, and lambda expressions.
- Familiarity with `JPanel`, `JFrame`, and the Swing container hierarchy.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ChangeListener` and the `javax.swing.event` package.
- A JDK (version 8 or later; `JTabbedPane` is available in all modern JDKs).

---

### Related Programming Areas

- **`CardLayout`:** Provides similar functionality (switching between components in the same space) without the tab interface. The Java Tutorials note that `CardLayout` can be used as an alternative when the tab metaphor is not desired.
- **`JPanel`:** The most common content type for tabs.
- **`JScrollPane`:** Often used inside tabs to handle overflow.
- **`ChangeListener` and `ChangeEvent`:** The event model for tab selection changes.
- **`SingleSelectionModel`:** The underlying model for tab selection.
- **Look and feel:** The appearance of tabs, including the one-touch expandable arrows and scroll arrows, is determined by the current look and feel.

---

## Core Concepts and Features

---

## 1. JTabbedPane — The Class

### Definitions

**Core Definition.** `JTabbedPane` is a Swing container that organizes multiple components into a tabbed interface, allowing the user to switch between them by selecting tabs.

**Technical Definition.** `public class JTabbedPane extends JComponent implements Serializable, Accessible, SwingConstants`. The class provides constructors that accept tab placement and tab layout policy. Components are added with `addTab` or `insertTab`. Each tab is associated with an index, a title, an optional icon, a tooltip, and a mnemonic. The `SingleSelectionModel` manages the selected index. A `ChangeListener` can be registered to observe changes.

**Beginner-Friendly Explanation.** `JTabbedPane` is the Swing “tabbed notebook.” You create the tabbed pane, create the panels you want to show, and add them with labels. The user clicks the tabs to switch between the panels. It is the standard way to fit multiple views into a single window area.

---

### Purposes

- To organize multiple components into a single, compact display area where only one component is visible at a time.
- To provide a familiar, look-and-feel-consistent tabbed interface for switching between related views.
- To reduce window clutter by consolidating multiple panels into one container.
- To support dynamic addition and removal of tabs at runtime.
- To serve as the structural backbone for settings dialogs, document interfaces, and multi-view applications.

---

### Syntax Rules and Structure

#### Complete General Syntax (Construction)

```java
// Default: tabs on TOP, WRAP_TAB_LAYOUT
JTabbedPane tabs = new JTabbedPane();

// Specify tab placement
JTabbedPane tabs = new JTabbedPane(JTabbedPane.TOP);

// Specify tab placement and layout policy
JTabbedPane tabs = new JTabbedPane(
    JTabbedPane.TOP,
    JTabbedPane.SCROLL_TAB_LAYOUT);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JTabbedPane()` | Creates an empty tabbed pane with tabs at `TOP` and `WRAP_TAB_LAYOUT`. |
| `JTabbedPane(int tabPlacement)` | Creates an empty tabbed pane with the specified placement (`TOP`, `BOTTOM`, `LEFT`, `RIGHT`). |
| `JTabbedPane(int tabPlacement, int tabLayoutPolicy)` | Creates an empty tabbed pane with the specified placement and layout policy (`WRAP_TAB_LAYOUT` or `SCROLL_TAB_LAYOUT`). |

#### Syntax Rules

- Tabs are added with `addTab` or `insertTab`; components are never added directly with `add`.
- The selected index is automatically set to 0 when the first tab is added.
- The tab placement and layout policy can be changed after construction with `setTabPlacement` and `setTabLayoutPolicy`.
- `JTabbedPane` is not thread-safe; construct and manipulate it on the EDT.

#### Constraints and Limitations

- Only one tab is selected at a time. Multi-selection is not supported.
- The tab count can be 0, in which case the selected index is -1.
- Serialized `JTabbedPane` objects are not compatible with future Swing releases.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Tabbed Pane

```java
import javax.swing.*;
import java.awt.*;

public class MinimalTabbedPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JTabbedPane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();                     // 1. Create

            JPanel panel1 = new JPanel();
            panel1.add(new JLabel("Content of Tab 1"));
            panel1.setBackground(Color.LIGHT_GRAY);

            JPanel panel2 = new JPanel();
            panel2.add(new JLabel("Content of Tab 2"));
            panel2.setBackground(Color.CYAN);

            tabs.addTab("Tab 1", panel1);                             // 2. Add tabs
            tabs.addTab("Tab 2", panel2);

            frame.add(tabs);                                           // 3. Add to frame
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with two tabs labeled “Tab 1” and “Tab 2.” Tab 1 shows a light-gray panel with the label “Content of Tab 1.” Clicking “Tab 2” switches to a cyan panel with “Content of Tab 2.” The first tab is selected by default.

**Why the code produces this result.** The `JTabbedPane` is created with the default `TOP` placement. `addTab` adds each panel with its title. The `SingleSelectionModel` automatically selects the first tab when it is added, so Tab 1 is visible initially.

---

#### Example 2: Tabbed Pane with Scroll Layout

```java
import javax.swing.*;
import java.awt.*;

public class ScrollTabLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Scroll Tab Layout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(
                JTabbedPane.TOP,
                JTabbedPane.SCROLL_TAB_LAYOUT);                       // 1. Scroll policy

            for (int i = 1; i <= 10; i++) {
                JPanel panel = new JPanel();
                panel.add(new JLabel("Panel " + i));
                tabs.addTab("Tab " + i, panel);                       // 2. Add many tabs
            }

            frame.add(tabs);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with ten tabs. Because the tabs do not fit in a single row at 300 pixels wide, scroll arrows appear at the edges of the tab area. Clicking the arrows reveals hidden tabs without changing the selection.

**Why the code produces this result.** `SCROLL_TAB_LAYOUT` instructs the look and feel to display scroll arrows when tabs overflow. This is in contrast to `WRAP_TAB_LAYOUT`, which would create multiple rows of tabs.

---

### Real-World Cases

- **Settings dialogs** with tabs for “General,” “Appearance,” “Advanced,” etc.
- **Document interfaces** where each tab represents an open file or document.
- **Dashboards** with tabs for different data views (charts, tables, logs).
- **IDEs** where tabs represent open source files.

---

## 2. Tab Creation

### Definitions

**Core Definition.** Tab creation is the process of instantiating a `JTabbedPane` and preparing it for receiving tabs.

**Technical Definition.** A `JTabbedPane` is created via one of its three constructors. After construction, the tabbed pane is empty (`getTabCount()` returns 0, and `getSelectedIndex()` returns -1). Tabs are then added using `addTab` or `insertTab`.

**Beginner-Friendly Explanation.** Creating a tabbed pane means making the container first. You can choose where the tabs go (top, bottom, left, right) and how they behave when there are too many (wrap or scroll). Then you add your panels as tabs.

---

### Purposes

- To instantiate the tabbed container with the desired tab placement and layout policy.
- To prepare an empty container for subsequent tab additions.
- To establish the tabbed pane’s configuration before adding content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane();
JTabbedPane tabs = new JTabbedPane(JTabbedPane.LEFT);
JTabbedPane tabs = new JTabbedPane(
    JTabbedPane.BOTTOM,
    JTabbedPane.SCROLL_TAB_LAYOUT);
```

#### Component Breakdown

| Constructor Parameter | Meaning |
|-----------------------|---------|
| `int tabPlacement` | `TOP`, `BOTTOM`, `LEFT`, or `RIGHT`. |
| `int tabLayoutPolicy` | `WRAP_TAB_LAYOUT` or `SCROLL_TAB_LAYOUT`. |

#### Syntax Rules

- The default placement is `TOP`; the default layout policy is `WRAP_TAB_LAYOUT`.
- The tabbed pane must be added to a container (e.g., a `JFrame`) to be displayed.
- Creation must occur on the EDT.

#### Constraints and Limitations

- An empty tabbed pane has a selected index of -1 and displays nothing.
- Passing an invalid placement value throws `IllegalArgumentException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Constructing with Left Placement

```java
import javax.swing.*;
import java.awt.*;

public class LeftPlacement {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Left Placement");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(JTabbedPane.LEFT); // 1. Left tabs

            tabs.addTab("Alpha", new JLabel("Alpha content", SwingConstants.CENTER));
            tabs.addTab("Beta",  new JLabel("Beta content",  SwingConstants.CENTER));
            tabs.addTab("Gamma", new JLabel("Gamma content", SwingConstants.CENTER));

            frame.add(tabs);
            frame.setSize(450, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Tabs appear vertically along the left side of the content area.

**Why the code produces this result.** `JTabbedPane.LEFT` places the tabs on the left. The content area occupies the remaining space.

---

### Real-World Cases

- **Left-placed tabs** in preference dialogs where vertical space is more abundant.
- **Bottom-placed tabs** in wizards where the main content should dominate the top of the window.

---

## 3. Tab Addition — addTab()

### Definitions

**Core Definition.** `addTab(String title, Component component)` appends a new tab to the end of the tabbed pane with the specified title and component.

**Technical Definition.** `public void addTab(String title, Component component)`. Overloads allow specifying an icon, tooltip, and mnemonic. The tab is appended at the end of the tab list, and its index is `getTabCount() - 1` at the time of addition. If the tabbed pane was empty, the new tab becomes the selected tab.

**Beginner-Friendly Explanation.** `addTab` is the primary way to put a new tab at the end of the tabbed pane. You give it a title (the text on the tab) and the component you want to show when the tab is selected.

---

### Purposes

- To add a new tab to the end of the tabbed pane’s tab list.
- To associate a component with a tab title.
- To build the tabbed interface incrementally.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.addTab("Title", component);
tabs.addTab("Title", icon, component);
tabs.addTab("Title", icon, component, "Tooltip");
tabs.addTab("Title", icon, component, "Tooltip", mnemonic);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String title` | Text displayed on the tab. |
| `Icon icon` | Optional icon displayed on the tab. |
| `Component component` | The component shown when the tab is selected. |
| `String tip` | Tooltip text shown when hovering over the tab. |
| `int mnemonic` | Keyboard mnemonic for selecting the tab. |

#### Syntax Rules

- The tab is appended to the end of the list.
- The component is not added to the tabbed pane’s container hierarchy until the tab is selected (the `JTabbedPane` uses a single child container internally).
- Adding a tab to an empty tabbed pane automatically selects it.

#### Constraints and Limitations

- The component must not already have a parent; otherwise, it may be removed from its previous container.
- The title can be `null` if a custom tab component will be used.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Adding Tabs with Icons and Tooltips

```java
import javax.swing.*;
import java.awt.*;

public class AddTabDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("AddTab Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();

            ImageIcon icon = new ImageIcon(new java.awt.image.BufferedImage(
                16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB));

            JPanel p1 = new JPanel();
            p1.add(new JLabel("First panel"));
            tabs.addTab("First", icon, p1, "This is the first tab"); // 1. Full addTab

            JPanel p2 = new JPanel();
            p2.add(new JLabel("Second panel"));
            tabs.addTab("Second", p2);                                // 2. Simple addTab

            frame.add(tabs);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two tabs labeled “First” and “Second.” The “First” tab has a small icon and shows the tooltip “This is the first tab” when hovered. The first tab is selected by default.

**Why the code produces this result.** The full `addTab` overload adds the title, icon, component, and tooltip. The simple `addTab` adds just the title and component. The first tab is selected automatically.

---

### Real-World Cases

- **Adding document tabs** as files are opened.
- **Adding settings sections** to a preferences dialog.
- **Building dashboards** by appending new data views.

---

## 4. Tab Addition — insertTab()

### Definitions

**Core Definition.** `insertTab(String title, Icon icon, Component component, String tip, int index)` inserts a new tab at the specified index, shifting existing tabs to the right.

**Technical Definition.** `public void insertTab(String title, Icon icon, Component component, String tip, int index)`. The `index` parameter must be between 0 and `getTabCount()` inclusive. The new tab occupies the specified index, and all tabs at or after that index are shifted one position to the right.

**Beginner-Friendly Explanation.** `insertTab` is like `addTab`, but you choose where the new tab goes. If you want a tab to appear first, third, or anywhere else, `insertTab` lets you place it precisely.

---

### Purposes

- To insert a tab at a specific position rather than at the end.
- To maintain a logical ordering of tabs when new tabs are added dynamically.
- To place a new tab before or after an existing tab.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.insertTab("Title", icon, component, "Tooltip", index);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int index` | The position where the tab should be inserted (0 to tab count). |

#### Syntax Rules

- `index` must be between 0 and `getTabCount()` inclusive; otherwise, `IndexOutOfBoundsException` is thrown.
- Existing tabs at and after `index` are shifted right.
- If the tabbed pane was empty and `index` is 0, the new tab becomes selected.

#### Constraints and Limitations

- The component must not already have a parent.
- Inserting a tab does not change the selected tab unless the tabbed pane was empty.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inserting a Tab at the Beginning

```java
import javax.swing.*;

public class InsertTabDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("InsertTab Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Second", new JLabel("Second content"));
            tabs.addTab("Third",  new JLabel("Third content"));

            tabs.insertTab("First", null,
                new JLabel("First content"), null, 0);              // 1. Insert at 0

            frame.add(tabs);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three tabs appear in the order “First,” “Second,” “Third.” The “First” tab is at index 0, and the previously existing tabs have been shifted right. The “Second” tab (which was originally at index 0) is still selected.

**Why the code produces this result.** `insertTab` with `index = 0` places the new tab at the beginning. The existing tabs shift to indices 1 and 2. The selected index remains unchanged (it was 0 for “Second”), so “Second” is still selected.

---

### Real-World Cases

- **Dynamic document ordering** where a new tab should appear in alphabetical or logical order.
- **Inserting a “Home” tab** before other tabs.
- **Reordering tabs** by removing and re-inserting them.

---

## 5. Tab Methods — setSelectedIndex()

### Definitions

**Core Definition.** `setSelectedIndex(int index)` selects the tab at the specified index, making its component visible.

**Technical Definition.** `public void setSelectedIndex(int index)`. The method updates the `SingleSelectionModel` and fires a `ChangeEvent` to registered `ChangeListener` objects. The index must be between 0 and `getTabCount() - 1`, or -1 to clear the selection (though a selected index of -1 is only valid when the tab count is 0).

**Beginner-Friendly Explanation.** `setSelectedIndex` is how you programmatically switch to a specific tab. You give it the index number (0 for the first tab, 1 for the second, etc.), and that tab becomes active.

---

### Purposes

- To programmatically select a tab by its index.
- To restore a previously selected tab from saved state.
- To synchronize tab selection with other UI controls.
- To implement wizard-style navigation by advancing the selected tab.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.setSelectedIndex(2);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setSelectedIndex(int)` | Selects the tab at the given index. |

#### Syntax Rules

- The index must be valid (0 to tab count - 1) or -1 if the tab count is 0.
- A `ChangeEvent` is fired after the selection changes.
- The method should be called on the EDT.

#### Constraints and Limitations

- Selecting an invalid index throws `IndexOutOfBoundsException`.
- The method does not verify that the tab is enabled; selecting a disabled tab may have no visible effect or may throw an exception depending on the look and feel.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Programmatic Tab Selection

```java
import javax.swing.*;
import java.awt.*;

public class SetSelectedIndexDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetSelectedIndex");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("One",   new JLabel("First"));
            tabs.addTab("Two",   new JLabel("Second"));
            tabs.addTab("Three", new JLabel("Third"));

            JButton selectThird = new JButton("Select Tab 3");
            selectThird.addActionListener(e ->
                tabs.setSelectedIndex(2));                             // 1. Select index 2

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(selectThird, BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “One” tab is selected by default. Clicking “Select Tab 3” switches the selection to the third tab (“Three”).

**Why the code produces this result.** `setSelectedIndex(2)` updates the `SingleSelectionModel` to index 2, making the third tab’s component visible. A `ChangeEvent` is fired to any registered listeners.

---

### Real-World Cases

- **Restoring the last selected tab** when reopening a document.
- **Wizard navigation** where “Next” selects the next tab.
- **Synchronizing tabs** with a navigation menu.

---

## 6. Tab Methods — getSelectedIndex()

### Definitions

**Core Definition.** `getSelectedIndex()` returns the index of the currently selected tab, or -1 if no tab is selected.

**Technical Definition.** `public int getSelectedIndex()`. The method queries the `SingleSelectionModel` for the selected index. If the tab count is 0, the selected index is -1.

**Beginner-Friendly Explanation.** `getSelectedIndex` tells you which tab is currently active. It returns a number: 0 for the first tab, 1 for the second, and so on.

---

### Purposes

- To determine which tab is currently selected.
- To branch logic based on the active tab.
- To save the selected tab for later restoration.
- To synchronize other UI elements with the current tab.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int index = tabs.getSelectedIndex();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSelectedIndex()` | Returns the selected index or -1. |

#### Syntax Rules

- Returns -1 if the tab count is 0.
- The method is safe to call at any time.

#### Constraints and Limitations

- The returned index does not indicate whether the tab is enabled or disabled.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading the Selected Index

```java
import javax.swing.*;
import java.awt.*;

public class GetSelectedIndexDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("GetSelectedIndex");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("A", new JLabel("A"));
            tabs.addTab("B", new JLabel("B"));
            tabs.addTab("C", new JLabel("C"));

            JLabel status = new JLabel("Selected: 0");
            tabs.addChangeListener(e ->
                status.setText("Selected: " + tabs.getSelectedIndex())); // 1. Read index

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The status label displays “Selected: 0” initially. When the user clicks tab B, the label updates to “Selected: 1”; clicking tab C shows “Selected: 2.”

**Why the code produces this result.** The `ChangeListener` reads `getSelectedIndex()` whenever the selection changes and updates the label accordingly.

---

### Real-World Cases

- **Analytics** tracking which tab users view most.
- **Conditional logic** that changes behavior based on the active tab.
- **Saving the selected tab** in user preferences.

---

## 7. Tab Methods — setTitleAt()

### Definitions

**Core Definition.** `setTitleAt(int index, String title)` changes the title of the tab at the specified index.

**Technical Definition.** `public void setTitleAt(int index, String title)`. The title is the text rendered by the look and feel on the tab. The index must be valid.

**Beginner-Friendly Explanation.** `setTitleAt` lets you rename a tab after it has been added. You can update the title dynamically—for example, to show a file name or a status indicator.

---

### Purposes

- To change a tab’s title dynamically.
- To reflect state changes (e.g., adding an asterisk for unsaved changes).
- To support internationalization by updating titles based on locale.
- To correct a title that was set incorrectly at creation time.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.setTitleAt(0, "New Title");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setTitleAt(int index, String title)` | Sets the title of the tab at `index`. |

#### Syntax Rules

- The index must be between 0 and `getTabCount() - 1`.
- The title can be `null`; the tab will render no text.
- The change is reflected immediately.

#### Constraints and Limitations

- The look and feel may truncate long titles.
- If a custom tab component is set at the index, the title is ignored (the custom component is responsible for rendering).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Title Update

```java
import javax.swing.*;

public class SetTitleAtDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetTitleAt");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Untitled", new JLabel("Document content"));

            JButton saveButton = new JButton("Save");
            saveButton.addActionListener(e ->
                tabs.setTitleAt(0, "document.txt"));                   // 1. Update title

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(saveButton, BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The tab initially shows “Untitled.” Clicking “Save” changes the title to “document.txt.”

**Why the code produces this result.** `setTitleAt(0, "document.txt")` updates the title of the first tab, and the look and feel repaints the tab header.

---

### Real-World Cases

- **Document editors** showing file names and unsaved-change markers.
- **Chat applications** showing the contact name or unread message count.
- **Settings dialogs** reflecting the current configuration name.

---

## 8. Tab Methods — setIconAt()

### Definitions

**Core Definition.** `setIconAt(int index, Icon icon)` sets or changes the icon displayed on the tab at the specified index.

**Technical Definition.** `public void setIconAt(int index, Icon icon)`. The icon is rendered alongside the title (or in place of it, depending on the look and feel and the custom tab component). Passing `null` removes the icon.

**Beginner-Friendly Explanation.** `setIconAt` lets you put a small picture on a tab. It can be used to indicate the tab’s type (e.g., a document icon, a settings gear, a chat bubble).

---

### Purposes

- To add a visual indicator to a tab.
- To change the icon based on state (e.g., a warning icon for an error).
- To distinguish tabs by category.
- To support branding or theming.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.setIconAt(0, icon);
tabs.setIconAt(1, null); // remove icon
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setIconAt(int index, Icon icon)` | Sets the icon for the tab at `index`. |

#### Syntax Rules

- The index must be valid.
- The icon can be `null` to remove it.
- The icon is rendered by the look and feel (or by the custom tab component if one is set).

#### Constraints and Limitations

- Icon size and placement are look-and-feel-dependent.
- If a custom tab component is set, the icon may be ignored.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Changing a Tab Icon

```java
import javax.swing.*;
import java.awt.*;

public class SetIconAtDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetIconAt");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Tab", new JLabel("Content"));

            ImageIcon icon = new ImageIcon(new java.awt.image.BufferedImage(
                16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB));

            JButton setIconButton = new JButton("Set Icon");
            setIconButton.addActionListener(e ->
                tabs.setIconAt(0, icon));                               // 1. Set icon

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(setIconButton, BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The tab initially has no icon. Clicking “Set Icon” adds a small 16×16 icon to the tab header.

**Why the code produces this result.** `setIconAt(0, icon)` associates the icon with the first tab. The look and feel renders it alongside the title.

---

### Real-World Cases

- **File-type icons** in document tabs.
- **Status icons** (e.g., red dot for unsaved changes).
- **Category icons** in settings dialogs.

---

## 9. Tab Methods — setToolTipTextAt()

### Definitions

**Core Definition.** `setToolTipTextAt(int index, String toolTipText)` sets the tooltip text displayed when the user hovers over the tab at the specified index.

**Technical Definition.** `public void setToolTipTextAt(int index, String toolTipText)`. The tooltip is shown by the look and feel’s tooltip manager. Passing `null` removes the tooltip.

**Beginner-Friendly Explanation.** `setToolTipTextAt` adds a small help message that appears when the user hovers over a tab. It can explain what the tab contains or provide a shortcut hint.

---

### Purposes

- To provide additional information about a tab without cluttering the interface.
- To display keyboard shortcuts or mnemonics.
- To show the full title when the tab title is truncated.
- To guide users in unfamiliar interfaces.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.setToolTipTextAt(0, "This tab shows general settings");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setToolTipTextAt(int index, String text)` | Sets the tooltip for the tab at `index`. |

#### Syntax Rules

- The index must be valid.
- The tooltip text can be `null` to remove it.
- The tooltip is displayed after the look and feel’s initial delay.

#### Constraints and Limitations

- Tooltips are not shown if the tab is disabled (depending on the look and feel).
- The tooltip does not appear when a custom tab component is used unless the custom component provides its own tooltip.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Adding a Tooltip

```java
import javax.swing.*;

public class SetToolTipDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetToolTipTextAt");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("General", new JLabel("General settings"));
            tabs.setToolTipTextAt(0, "Configure general options");    // 1. Set tooltip

            tabs.addTab("Advanced", new JLabel("Advanced settings"));
            tabs.setToolTipTextAt(1, "Configure advanced options");

            frame.add(tabs);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Hovering over the “General” tab shows the tooltip “Configure general options.” Hovering over “Advanced” shows “Configure advanced options.”

**Why the code produces this result.** `setToolTipTextAt` registers the tooltip text with the tab’s metadata, and the look and feel’s tooltip manager displays it on hover.

---

### Real-World Cases

- **IDE tabs** showing the full file path in the tooltip.
- **Settings dialogs** explaining what each tab controls.
- **Wizard tabs** describing the current step.

---

## 10. Tab Methods — setEnabledAt()

### Definitions

**Core Definition.** `setEnabledAt(int index, boolean enabled)` enables or disables the tab at the specified index.

**Technical Definition.** `public void setEnabledAt(int index, boolean enabled)`. When a tab is disabled, the user cannot select it by clicking, and it is typically rendered in a grayed-out state (depending on the look and feel).

**Beginner-Friendly Explanation.** `setEnabledAt` greys out a tab or makes it clickable again. Disabled tabs cannot be selected by the user, which is useful when a tab’s content is not yet available.

---

### Purposes

- To prevent the user from selecting a tab that is not ready.
- To indicate that a tab’s content is unavailable or locked.
- To implement wizard flows where later steps are disabled until earlier steps are completed.
- To conditionally enable tabs based on application state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.setEnabledAt(1, false); // disable second tab
tabs.setEnabledAt(1, true);  // re-enable
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setEnabledAt(int index, boolean enabled)` | Enables or disables the tab. |

#### Syntax Rules

- The index must be valid.
- A disabled tab cannot be selected by user interaction, but `setSelectedIndex` may still select it programmatically (behavior is look-and-feel-dependent).
- Should be called on the EDT.

#### Constraints and Limitations

- The appearance of disabled tabs varies by look and feel.
- Some look-and-feel implementations may still allow keyboard selection of disabled tabs.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling a Tab

```java
import javax.swing.*;
import java.awt.*;

public class SetEnabledAtDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetEnabledAt");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Step 1", new JLabel("Complete step 1 first"));
            tabs.addTab("Step 2", new JLabel("Step 2 content"));
            tabs.setEnabledAt(1, false);                               // 1. Disable tab 2

            JButton enableButton = new JButton("Enable Step 2");
            enableButton.addActionListener(e ->
                tabs.setEnabledAt(1, true));                            // 2. Enable tab 2

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(enableButton, BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “Step 2” tab is initially grayed out and cannot be selected by clicking. Clicking “Enable Step 2” makes it selectable.

**Why the code produces this result.** `setEnabledAt(1, false)` disables the second tab, and the look and feel renders it as disabled. The button re-enables it.

---

### Real-World Cases

- **Wizard dialogs** where later steps are disabled until prerequisites are met.
- **Tabbed editors** where a tab is disabled while a file is loading.
- **Permission-based UIs** where certain tabs are disabled for users without access.

---

## 11. Tab Methods — remove()

### Definitions

**Core Definition.** `remove(int index)` removes the tab and its associated component from the tabbed pane at the specified index.

**Technical Definition.** `public void remove(int index)` is inherited from `Container` but overridden by `JTabbedPane` to remove the tab metadata as well as the component. The tabs after the removed index shift left. If the removed tab was selected, the selection moves to another tab (usually the next one, or the previous one if the removed tab was last).

**Beginner-Friendly Explanation.** `remove` deletes a tab and its content from the tabbed pane. The other tabs close the gap.

---

### Purposes

- To remove a tab and its component at runtime.
- To close documents or views in a tabbed interface.
- To clean up tabs that are no longer needed.
- To implement a “close tab” feature.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.remove(2);           // remove by index
tabs.remove(component);    // remove by component reference
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `remove(int index)` | Removes the tab at `index`. |
| `remove(Component)` | Removes the tab associated with the component. |

#### Syntax Rules

- The index must be valid.
- If the removed tab was selected, the selection changes to another tab (or to -1 if no tabs remain).
- A `ChangeEvent` is fired if the selected tab changes.

#### Constraints and Limitations

- The removed component is not disposed; it can be reused.
- Removing the last tab sets the selected index to -1.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Removing a Tab

```java
import javax.swing.*;
import java.awt.*;

public class RemoveTabDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Remove Tab");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("A", new JLabel("A"));
            tabs.addTab("B", new JLabel("B"));
            tabs.addTab("C", new JLabel("C"));

            JButton removeButton = new JButton("Remove Tab B");
            removeButton.addActionListener(e -> {
                int index = tabs.indexOfTab("B");
                if (index != -1) {
                    tabs.remove(index);                                // 1. Remove
                }
            });

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(removeButton, BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three tabs (“A,” “B,” “C”) appear. Clicking “Remove Tab B” removes the “B” tab. The remaining tabs are “A” and “C,” and the selection moves to “C” (since “B” was selected at index 1).

**Why the code produces this result.** `remove(index)` removes the tab at the found index. The `SingleSelectionModel` adjusts the selected index accordingly.

---

### Real-World Cases

- **Closing documents** in an editor.
- **Removing completed tasks** from a task manager.
- **Dismissing temporary views** in a dashboard.

---

## 12. Tab Methods — removeAll()

### Definitions

**Core Definition.** `removeAll()` removes all tabs and their components from the tabbed pane.

**Technical Definition.** `public void removeAll()` is inherited from `Container` and overridden by `JTabbedPane` to remove all tab metadata and components. After the call, `getTabCount()` returns 0 and `getSelectedIndex()` returns -1.

**Beginner-Friendly Explanation.** `removeAll` clears the tabbed pane completely. All tabs disappear, and the tabbed pane becomes empty.

---

### Purposes

- To clear all tabs at once.
- To reset the tabbed pane to an empty state.
- To rebuild the tabbed interface with a new set of tabs.
- To clean up resources when the tabbed pane is no longer needed.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.removeAll();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `removeAll()` | Removes all tabs and components. |

#### Syntax Rules

- After `removeAll()`, the tabbed pane is empty; adding a tab will select it automatically.
- A `ChangeEvent` may be fired as the selected index changes to -1.

#### Constraints and Limitations

- The removed components are not disposed; they may still be referenced elsewhere.
- The method should be called on the EDT.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Clearing and Rebuilding

```java
import javax.swing.*;
import java.awt.*;

public class RemoveAllDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Remove All");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Old 1", new JLabel("Old 1"));
            tabs.addTab("Old 2", new JLabel("Old 2"));

            JButton resetButton = new JButton("Reset Tabs");
            resetButton.addActionListener(e -> {
                tabs.removeAll();                                      // 1. Clear all
                tabs.addTab("New 1", new JLabel("New 1"));             // 2. Add new
                tabs.addTab("New 2", new JLabel("New 2"));
            });

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(resetButton, BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The tabbed pane initially shows “Old 1” and “Old 2.” Clicking “Reset Tabs” clears them and replaces them with “New 1” and “New 2.”

**Why the code produces this result.** `removeAll()` removes every tab and component. The subsequent `addTab` calls rebuild the tabbed pane from scratch.

---

### Real-World Cases

- **Refreshing the entire tabbed interface** when data changes.
- **Resetting a wizard** to its initial state.
- **Switching between different tab sets** based on user role or context.

---

## 13. Tab Methods — getTabCount()

### Definitions

**Core Definition.** `getTabCount()` returns the number of tabs in the tabbed pane.

**Technical Definition.** `public int getTabCount()`. Returns 0 for an empty tabbed pane.

**Beginner-Friendly Explanation.** `getTabCount` tells you how many tabs there are. It is useful for loops and for checking whether the tabbed pane is empty.

---

### Purposes

- To determine the number of tabs.
- To iterate over tabs.
- To check whether the tabbed pane is empty.
- To validate indices before accessing tabs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int count = tabs.getTabCount();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getTabCount()` | Returns the number of tabs. |

#### Syntax Rules

- Returns 0 if no tabs have been added.
- Valid indices are 0 to `getTabCount() - 1`.

#### Constraints and Limitations

- None; the method is safe to call at any time.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Counting Tabs

```java
import javax.swing.*;

public class GetTabCountDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("GetTabCount");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("One", new JLabel("One"));
            tabs.addTab("Two", new JLabel("Two"));

            JButton countButton = new JButton("Count Tabs");
            countButton.addActionListener(e ->
                System.out.println("Tab count: " + tabs.getTabCount())); // 1. Count

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(countButton, BorderLayout.SOUTH);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Count Tabs” prints “Tab count: 2” to the console.

**Why the code produces this result.** `getTabCount()` returns 2 because two tabs have been added.

---

### Real-World Cases

- **Iterating over tabs** to save state or update titles.
- **Checking if the tabbed pane is empty** before adding a tab.
- **Validating indices** before calling `setSelectedIndex` or `remove`.

---

## 14. Tab Placement — TOP

### Definitions

**Core Definition.** `JTabbedPane.TOP` places the tabs above the content area. This is the default placement for `JTabbedPane`.

**Technical Definition.** The constant `JTabbedPane.TOP` has the value `SwingConstants.TOP`. It is used in the constructor or with `setTabPlacement(int)` to position tabs at the top of the content area.

**Beginner-Friendly Explanation.** TOP placement is the most common tab layout: the tabs are in a horizontal row above the content, like a typical web browser or settings dialog.

---

### Purposes

- To use the default, most familiar tab layout.
- To maximize horizontal space for the content area.
- To align tabs with the top edge of the content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane(JTabbedPane.TOP);
// or:
tabs.setTabPlacement(JTabbedPane.TOP);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `JTabbedPane.TOP` | Tabs above the content. |

#### Syntax Rules

- This is the default placement if none is specified.
- The tab area is horizontal.

#### Constraints and Limitations

- With many tabs, `WRAP_TAB_LAYOUT` creates multiple rows, reducing content space.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Explicit TOP Placement

```java
import javax.swing.*;

public class TopPlacementDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("TOP Placement");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(JTabbedPane.TOP);      // 1. TOP
            tabs.addTab("Home",    new JLabel("Home",    SwingConstants.CENTER));
            tabs.addTab("Profile", new JLabel("Profile", SwingConstants.CENTER));

            frame.add(tabs);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Tabs appear horizontally above the content area.

**Why the code produces this result.** `JTabbedPane.TOP` specifies that tabs should be placed at the top.

---

### Real-World Cases

- **Web browsers** with tabs above the page content.
- **Settings dialogs** with tabs at the top.
- **Document editors** with file tabs above the editor area.

---

## 15. Tab Placement — BOTTOM

### Definitions

**Core Definition.** `JTabbedPane.BOTTOM` places the tabs below the content area.

**Technical Definition.** The constant `JTabbedPane.BOTTOM` is used with the constructor or `setTabPlacement` to position tabs at the bottom.

**Beginner-Friendly Explanation.** BOTTOM placement puts the tabs below the content, like a status bar with tabs.

---

### Purposes

- To place tabs below the content for visual variety.
- To keep the top of the content area unobstructed.
- To support layouts where the primary action or content is at the top.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane(JTabbedPane.BOTTOM);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `JTabbedPane.BOTTOM` | Tabs below the content. |

#### Syntax Rules

- The tab area is horizontal.
- The content area occupies the space above the tabs.

#### Constraints and Limitations

- Bottom placement may feel less conventional to some users.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Bottom Placement

```java
import javax.swing.*;

public class BottomPlacementDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("BOTTOM Placement");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(JTabbedPane.BOTTOM);   // 1. BOTTOM
            tabs.addTab("Log",     new JLabel("Log content",     SwingConstants.CENTER));
            tabs.addTab("Console", new JLabel("Console content", SwingConstants.CENTER));

            frame.add(tabs);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Tabs appear horizontally below the content area.

**Why the code produces this result.** `JTabbedPane.BOTTOM` specifies that tabs should be placed at the bottom.

---

### Real-World Cases

- **Debug panels** with log and console tabs at the bottom.
- **Terminal emulators** with tabs below the main display.

---

## 16. Tab Placement — LEFT

### Definitions

**Core Definition.** `JTabbedPane.LEFT` places the tabs vertically along the left side of the content area.

**Technical Definition.** The constant `JTabbedPane.LEFT` is used with the constructor or `setTabPlacement` to position tabs on the left.

**Beginner-Friendly Explanation.** LEFT placement stacks the tabs vertically on the left, like a sidebar navigation menu.

---

### Purposes

- To provide vertical tab navigation for settings or navigation-heavy interfaces.
- To maximize horizontal space for wide content.
- To create a sidebar-style layout.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane(JTabbedPane.LEFT);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `JTabbedPane.LEFT` | Tabs on the left side, stacked vertically. |

#### Syntax Rules

- Tabs are stacked vertically.
- The content area occupies the space to the right of the tabs.

#### Constraints and Limitations

- With many tabs, the vertical space may be exhausted; `WRAP_TAB_LAYOUT` may create multiple columns.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Left Placement

```java
import javax.swing.*;

public class LeftPlacementDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("LEFT Placement");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(JTabbedPane.LEFT);     // 1. LEFT
            tabs.addTab("Inbox",   new JLabel("Inbox",   SwingConstants.CENTER));
            tabs.addTab("Sent",    new JLabel("Sent",    SwingConstants.CENTER));
            tabs.addTab("Archive", new JLabel("Archive", SwingConstants.CENTER));

            frame.add(tabs);
            frame.setSize(450, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Tabs appear vertically along the left side of the content area.

**Why the code produces this result.** `JTabbedPane.LEFT` specifies left-side placement.

---

### Real-World Cases

- **Email clients** with folder navigation on the left.
- **Settings dialogs** with vertical category tabs.
- **IDEs** with tool windows organized as left-side tabs.

---

## 17. Tab Placement — RIGHT

### Definitions

**Core Definition.** `JTabbedPane.RIGHT` places the tabs vertically along the right side of the content area.

**Technical Definition.** The constant `JTabbedPane.RIGHT` is used with the constructor or `setTabPlacement` to position tabs on the right.

**Beginner-Friendly Explanation.** RIGHT placement stacks the tabs vertically on the right side, like a right-hand sidebar.

---

### Purposes

- To provide vertical tab navigation on the right side.
- To mirror left-placed layouts for right-to-left languages or aesthetic reasons.
- To keep the primary content area anchored to the left.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane(JTabbedPane.RIGHT);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `JTabbedPane.RIGHT` | Tabs on the right side, stacked vertically. |

#### Syntax Rules

- Tabs are stacked vertically.
- The content area occupies the space to the left of the tabs.

#### Constraints and Limitations

- With many tabs, vertical space may be exhausted.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Right Placement

```java
import javax.swing.*;

public class RightPlacementDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("RIGHT Placement");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(JTabbedPane.RIGHT);    // 1. RIGHT
            tabs.addTab("Properties", new JLabel("Properties", SwingConstants.CENTER));
            tabs.addTab("History",    new JLabel("History",    SwingConstants.CENTER));

            frame.add(tabs);
            frame.setSize(450, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Tabs appear vertically along the right side of the content area.

**Why the code produces this result.** `JTabbedPane.RIGHT` specifies right-side placement.

---

### Real-World Cases

- **Property inspectors** on the right side of an editor.
- **Right-to-left language interfaces** that mirror left-placed layouts.

---

## 18. Tab Layout — WRAP_TAB_LAYOUT

### Definitions

**Core Definition.** `WRAP_TAB_LAYOUT` is a tab layout policy that causes tabs to wrap to multiple rows (or columns, depending on placement) when they do not fit in a single run.

**Technical Definition.** The constant `JTabbedPane.WRAP_TAB_LAYOUT` has the value 0. It is the default tab layout policy. When tabs exceed the available space in the tab area, the look and feel creates additional rows of tabs.

**Beginner-Friendly Explanation.** WRAP means the tabs form multiple rows, like text wrapping in a paragraph. If there are too many tabs for one row, they continue on a second row below.

---

### Purposes

- To display all tabs without scrolling.
- To provide a familiar, compact multi-row tab layout.
- To avoid the need for scroll arrows.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane(
    JTabbedPane.TOP,
    JTabbedPane.WRAP_TAB_LAYOUT); // default
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `WRAP_TAB_LAYOUT` | Tabs wrap to multiple rows/columns. |

#### Syntax Rules

- This is the default layout policy.
- Wrapping occurs automatically when tabs overflow.

#### Constraints and Limitations

- Multiple rows reduce the available space for the content area.
- The wrapping behavior varies by look and feel.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: WRAP_TAB_LAYOUT with Many Tabs

```java
import javax.swing.*;

public class WrapLayoutDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("WRAP_TAB_LAYOUT");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(
                JTabbedPane.TOP,
                JTabbedPane.WRAP_TAB_LAYOUT);                         // 1. Wrap

            for (int i = 1; i <= 12; i++) {
                tabs.addTab("Tab " + i, new JLabel("Content " + i));
            }

            frame.add(tabs);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Twelve tabs arranged in multiple rows at the top of the tabbed pane. All tabs are visible without scrolling.

**Why the code produces this result.** `WRAP_TAB_LAYOUT` causes the look and feel to create multiple rows when the tabs do not fit in one row.

---

### Real-World Cases

- **Settings dialogs** with many tabs that should all be visible.
- **Tabbed interfaces** where scrolling would be inconvenient.

---

## 19. Tab Layout — SCROLL_TAB_LAYOUT

### Definitions

**Core Definition.** `SCROLL_TAB_LAYOUT` is a tab layout policy that keeps tabs in a single run and provides scroll arrows when the tabs do not fit.

**Technical Definition.** The constant `JTabbedPane.SCROLL_TAB_LAYOUT` has the value 1. When tabs exceed the available space, scroll arrows appear at the edges of the tab area, allowing the user to scroll through the tabs without changing the selected tab.

**Beginner-Friendly Explanation.** SCROLL means the tabs stay in one row, and small arrows appear so you can scroll left or right to see the hidden tabs.

---

### Purposes

- To keep tabs in a single row for a cleaner appearance.
- To allow many tabs without consuming vertical space for multiple rows.
- To provide a scrolling mechanism for tab overflow.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTabbedPane tabs = new JTabbedPane(
    JTabbedPane.TOP,
    JTabbedPane.SCROLL_TAB_LAYOUT);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `SCROLL_TAB_LAYOUT` | Tabs scroll; arrows appear when overflowing. |

#### Syntax Rules

- Scroll arrows appear only when tabs overflow.
- Clicking an arrow scrolls the tab strip; it does not select a tab.

#### Constraints and Limitations

- Not all look-and-feel implementations fully support scroll arrows.
- Scrolling can make some tabs less discoverable.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SCROLL_TAB_LAYOUT with Many Tabs

```java
import javax.swing.*;

public class ScrollLayoutDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SCROLL_TAB_LAYOUT");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane(
                JTabbedPane.TOP,
                JTabbedPane.SCROLL_TAB_LAYOUT);                        // 1. Scroll

            for (int i = 1; i <= 12; i++) {
                tabs.addTab("Tab " + i, new JLabel("Content " + i));
            }

            frame.add(tabs);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Twelve tabs in a single row. Because 300 pixels is insufficient to show all tabs, scroll arrows appear. Clicking the arrows reveals hidden tabs without changing the selection.

**Why the code produces this result.** `SCROLL_TAB_LAYOUT` keeps tabs in one row and adds scroll arrows when needed.

---

### Real-World Cases

- **IDEs** with many open files where vertical space is precious.
- **Browsers** with many tabs and a scrollable tab strip.

---

## 20. Tab Events — ChangeListener

### Definitions

**Core Definition.** `ChangeListener` is an interface for receiving `ChangeEvent` notifications when the selected tab changes in a `JTabbedPane`.

**Technical Definition.** `public interface ChangeListener extends EventListener`. The `JTabbedPane` class provides `addChangeListener(ChangeListener l)` and `removeChangeListener(ChangeListener l)`. The `ChangeListener`’s `stateChanged(ChangeEvent e)` method is invoked whenever the selected index or selected tab changes. The event source is the `JTabbedPane` itself.

**Beginner-Friendly Explanation.** A `ChangeListener` is like a notification service for tab changes. You tell the tabbed pane “let me know when the user switches tabs,” and it calls your `stateChanged` method each time the selection changes.

---

### Purposes

- To react to tab selection changes.
- To update other UI components based on the selected tab.
- To save the selected tab state when it changes.
- To lazy-load the content of a tab when it is first selected.
- To enforce validation or permission checks before switching tabs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tabs.addChangeListener(new ChangeListener() {
    @Override
    public void stateChanged(ChangeEvent e) {
        int index = tabs.getSelectedIndex();
        // react to the change
    }
});

// Or with a lambda:
tabs.addChangeListener(e -> {
    int index = tabs.getSelectedIndex();
    // react
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addChangeListener(ChangeListener)` | Registers a listener. |
| `stateChanged(ChangeEvent e)` | Called when the selection changes. |
| `getSelectedIndex()` | Returns the new selected index. |

#### Syntax Rules

- The listener is invoked on the EDT.
- The `ChangeEvent` source is the `JTabbedPane` instance.
- `stateChanged` is called after the selection has changed.
- Multiple listeners can be registered.

#### Constraints and Limitations

- The listener is called for both user-initiated and programmatic selection changes.
- The `ChangeEvent` does not contain the old index; the listener must track it if needed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Updating a Status Label on Tab Change

```java
import javax.swing.*;
import javax.swing.event.*;
import java.awt.*;

public class ChangeListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ChangeListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Home",    new JLabel("Home"));
            tabs.addTab("Profile", new JLabel("Profile"));
            tabs.addTab("Settings",new JLabel("Settings"));

            JLabel status = new JLabel("Selected: Home");

            tabs.addChangeListener(new ChangeListener() {              // 1. Register
                @Override
                public void stateChanged(ChangeEvent e) {
                    int index = tabs.getSelectedIndex();                // 2. Read index
                    String title = tabs.getTitleAt(index);
                    status.setText("Selected: " + title);
                }
            });

            frame.add(tabs, BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The status label shows “Selected: Home” initially. When the user clicks the “Profile” tab, it updates to “Selected: Profile”; clicking “Settings” updates it to “Selected: Settings.”

**Why the code produces this result.** The `ChangeListener` is notified each time the selection changes. It reads the new index and the corresponding title and updates the label.

---

### Real-World Cases

- **Lazy loading** tab content when first selected.
- **Saving the last selected tab** in user preferences.
- **Synchronizing a navigation menu** with the selected tab.
- **Logging tab usage** for analytics.

---

## 21. JTabbedPane Best Practices

### Definitions

**Core Definition.** `JTabbedPane` best practices are the conventions and patterns that ensure tabbed panes are used effectively, efficiently, and maintainably in Swing applications.

**Technical Definition.** Best practices include: constructing and manipulating the tabbed pane on the EDT; using `setSelectedIndex` or `setSelectedComponent` rather than `setVisible` on tab components; choosing the appropriate tab placement and layout policy for the content; using `ChangeListener` for selection changes; lazy-loading tab content to improve startup time; and disposing of resources when tabs are removed.

**Beginner-Friendly Explanation.** Best practices are the “rules of thumb” for using tabbed panes well. They include things like “build the tabbed pane on the special GUI thread,” “use `setSelectedIndex` to switch tabs instead of trying to make components visible directly,” and “load heavy tab content only when the user first opens that tab.”

---

### Purposes

- To prevent thread-related bugs by ensuring all GUI work occurs on the EDT.
- To avoid incorrect tab visibility by using the proper selection methods.
- To improve performance through lazy loading and proper resource management.
- To create intuitive, accessible tabbed interfaces.
- To maintain code that is easy to understand and extend.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JTabbedPane tabs = new JTabbedPane(
        JTabbedPane.TOP,
        JTabbedPane.SCROLL_TAB_LAYOUT);                                // 1. EDT + policy

    tabs.addTab("Tab 1", createPanel1());                              // 2. Lazy load
    tabs.addTab("Tab 2", createPanel2());

    tabs.addChangeListener(e -> {
        int index = tabs.getSelectedIndex();                            // 3. Listener
        loadTabContentIfNeeded(index);
    });

    frame.add(tabs);                                                    // 4. Add
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT | `SwingUtilities.invokeLater(...)` |
| Selection method | Use `setSelectedIndex` or `setSelectedComponent`, not `setVisible` |
| Layout policy | Choose `SCROLL_TAB_LAYOUT` for many tabs, `WRAP_TAB_LAYOUT` for few. |
| Change listener | Use `addChangeListener` to react to selection changes. |
| Lazy loading | Load heavy content in the `ChangeListener` when a tab is first selected. |
| Resource cleanup | Remove and dispose components when tabs are removed. |

#### Syntax Rules

- **Never use `setVisible` directly on a tab component** to make it visible. Use `setSelectedComponent` or `setSelectedIndex` instead.
- **Always construct and manipulate `JTabbedPane` on the EDT**.
- **Choose the layout policy based on tab count**: `SCROLL_TAB_LAYOUT` for many tabs, `WRAP_TAB_LAYOUT` for few.
- **Use `ChangeListener`** to react to tab changes rather than polling.
- **Lazy-load tab content** to improve startup performance.

#### Constraints and Limitations

- `setVisible` on tab components does not affect tab selection and can cause inconsistent state.
- `SCROLL_TAB_LAYOUT` may not be fully supported by all look-and-feel implementations.
- Disabled tabs may still be selectable programmatically.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Tabbed Pane with Lazy Loading

```java
import javax.swing.*;
import javax.swing.event.*;
import java.awt.*;

public class BestPracticeTabbedPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeTabbedPane::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Tabs");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JTabbedPane tabs = new JTabbedPane(
            JTabbedPane.TOP,
            JTabbedPane.SCROLL_TAB_LAYOUT);                                // 1. Policy

        tabs.addTab("Summary", new JLabel("Summary content", SwingConstants.CENTER));
        tabs.addTab("Details", new JPanel());                              // 2. Placeholder

        tabs.addChangeListener(new ChangeListener() {                      // 3. Listener
            private boolean detailsLoaded = false;

            @Override
            public void stateChanged(ChangeEvent e) {
                int index = tabs.getSelectedIndex();
                if (index == 1 && !detailsLoaded) {                        // 4. Lazy load
                    JPanel detailsPanel = new JPanel(new BorderLayout());
                    detailsPanel.add(new JLabel("Details loaded lazily",
                        SwingConstants.CENTER));
                    tabs.setComponentAt(1, detailsPanel);                  // 5. Replace
                    detailsLoaded = true;
                }
            }
        });

        frame.add(tabs);
        frame.setSize(500, 350);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** The tabbed pane starts with a “Summary” tab and an empty “Details” tab. When the user clicks “Details” for the first time, its content is created and displayed. Subsequent selections of “Details” reuse the already-loaded content.

**Why the code produces this result.** The `ChangeListener` checks whether the selected tab is the “Details” tab and whether it has already been loaded. If not, it creates the content and replaces the placeholder with `setComponentAt`. This lazy-loading approach keeps startup fast.

---

#### Example 2: Common Mistake — Using `setVisible` on Tab Components

```java
import javax.swing.*;

public class BadTabVisibility {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Tab Visibility");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel1 = new JPanel();
            panel1.add(new JLabel("Panel 1"));
            JPanel panel2 = new JPanel();
            panel2.add(new JLabel("Panel 2"));

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("One", panel1);
            tabs.addTab("Two", panel2);

            // WRONG: trying to show a tab by setting component visibility
            panel2.setVisible(true); // Does not select the tab

            frame.add(tabs);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “One” tab remains selected regardless of `panel2.setVisible(true)`. The visibility flag on the component does not affect tab selection.

**Why this is wrong.** The `JTabbedPane` documentation explicitly states: “Do not use `setVisible` directly on a tab component to make it visible, use `setSelectedComponent` or `setSelectedIndex` methods instead”. Tab visibility is controlled by the selection model, not by the component’s visible flag.

---

### Real-World Cases

- **Production Swing applications** use lazy loading and proper selection methods to create responsive, maintainable tabbed interfaces.
- **IDE and editor plugins** follow these practices when embedding tabbed panes in host applications.
- **Educational examples** use the best-practice skeleton to demonstrate correct `JTabbedPane` usage from the start.

---

## References

Oracle. *How to Use Tabbed Panes*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/tabbedpane.html.

Oracle. *Class JTabbedPane (Java SE 25)*. Java Platform, Standard Edition 25 API Specification. https://docs.oracle.com/en/java/javase/25/docs/api/java.desktop/javax/swing/JTabbedPane.html.

Oracle. *JTabbedPane: Scrollable Tabs*. https://docs.oracle.com/javase/tutorial/uiswing/components/tabbedpane.html (scrollable tabs section).

OpenJDK. *JTabbedPane (Java SE 10)*. https://cr.openjdk.org/jeps/... (API documentation).

GNU Classpath. *Class JTabbedPane*. https://developer.classpath.org/sinjdoc/javax/swing/JTabbedPane.html.

Stack Overflow. *JTabbedPane ChangeListener example*. https://stackoverflow.com/questions/... (ChangeListener usage).

Stack Overflow. *Horizontal Scrolling in Swing*. https://stackoverflow.com/questions/... (SCROLL_TAB_LAYOUT vs WRAP_TAB_LAYOUT).