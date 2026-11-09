# JList — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JList` is a Swing component that displays a list of objects and allows the user to select one or more items. A separate model, `ListModel`, maintains the contents of the list.

**Technical Definition.** `public class JList<E> extends JComponent implements Scrollable, Accessible`. The selection state of a `JList` is managed by a separate `ListSelectionModel`. `JList` provides convenience methods such as `setSelectedIndex` and `getSelectedValue` that interact with the selection model. By default, the selection model is configured for `MULTIPLE_INTERVAL_SELECTION`, but this can be changed via `setSelectionMode`. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JList` is the scrollable list of items you see in many applications—a file chooser’s list of files, a settings dialog’s list of options, or a chat window’s list of contacts. It shows a column (or grid) of items, and the user can click to select one or more of them. Unlike a `JComboBox`, which shows only the selected item until opened, a `JList` displays multiple items at once, making it ideal when you want the user to see several options simultaneously. Think of it as a physical list on a clipboard: all items are visible, and you can highlight one, several, or a continuous range depending on how the list is configured.

---

### Key Characteristics

1. **Model-View-Controller architecture.** `JList` separates its data (`ListModel`), its selection state (`ListSelectionModel`), and its visual rendering. This allows the same data to be shared across multiple lists or updated dynamically without recreating the component.
2. **Flexible selection modes.** Three selection modes are supported: `SINGLE_SELECTION` (one item at a time), `SINGLE_INTERVAL_SELECTION` (one contiguous range), and `MULTIPLE_INTERVAL_SELECTION` (any combination, the default).
3. **No built-in scrolling.** `JList` does not provide scroll bars on its own. It implements the `Scrollable` interface, so it should typically be placed inside a `JScrollPane` when the list is longer than the visible area.
4. **Customizable rendering.** A `ListCellRenderer` controls how each item is painted, enabling icons, custom fonts, colors, and even complex components in list cells.
5. **Dynamic updates through the model.** To add or remove items, you modify the `ListModel` (e.g., a `DefaultListModel`), not the `JList` directly. The model then notifies the list of changes.
6. **`ListSelectionListener` for selection events.** Selection changes are observed via `ListSelectionListener`, whose `valueChanged` method is invoked when the selection changes. The listener should check `getValueIsAdjusting()` to avoid reacting prematurely during drag-selection.
7. **Single-thread rule and serialization caveat.** `JList` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: generics, classes, interfaces, and lambda expressions.
- Familiarity with Swing containers (`JFrame`, `JPanel`) and the `JScrollPane`.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ListSelectionListener` and the `javax.swing.event` package.
- A JDK (version 8 or later; `JList` is available in all modern JDKs).

---

### Related Programming Areas

- **`ListModel` and `DefaultListModel`:** The data model interfaces and the default mutable implementation.
- **`ListSelectionModel`:** The selection state model, with its three selection mode constants.
- **`ListCellRenderer`:** Controls the visual appearance of each list item.
- **`JScrollPane`:** The standard container for providing scrolling to a `JList`.
- **`JComboBox`:** Alternative for one-of-many selection when the list is large or the user should see only the selected item.
- **`JTable`:** Alternative when items have multiple columns of data.

---

## Core Concepts and Features

---

## 1. JList — The Class

### Definitions

**Core Definition.** `JList` is the Swing class for creating a component that displays a list of objects from which the user can select one or more items.

**Technical Definition.** `public class JList<E> extends JComponent implements Scrollable, Accessible`. It provides constructors that accept an array, a `Vector`, or a `ListModel`. The class maintains a `dataModel` (the `ListModel`) and a `selectionModel` (`ListSelectionModel`). The `ListUI` delegate handles rendering and user interaction, keeping the visual representation synchronized with the model.

**Beginner-Friendly Explanation.** `JList` is the class you use to create a list. You give it an array of items (strings, objects) or a model, and it displays them as a scrollable list. The user clicks to select items, and you attach a listener to react to those selections.

---

### Purposes

- To display a list of items (strings, objects, or custom data) for user selection.
- To support single or multiple selection based on the configured selection mode.
- To serve as a compact, scrollable alternative to radio buttons or check boxes when there are many options.
- To provide a model-driven architecture that supports dynamic updates and shared data.
- To integrate with `JScrollPane` and custom renderers for a complete list-based UI.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JList<String> list = new JList<>();
JList<String> list = new JList<>(String[] data);
JList<String> list = new JList<>(Vector<String> data);
JList<String> list = new JList<>(ListModel<String> model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JList()` | Creates an empty list with a default `DefaultListModel`. |
| `JList(E[] data)` | Creates a list with a read-only `ListModel` wrapping the array. |
| `JList(Vector<E> data)` | Creates a list with a read-only `ListModel` wrapping the vector. |
| `JList(ListModel<E> model)` | Creates a list using the given model. |

#### Syntax Rules

- The list must be added to a container (typically a `JScrollPane`) to be displayed.
- The array and `Vector` constructors create a read-only model; for dynamic updates, use a `DefaultListModel`.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JList` does not provide built-in scrolling; wrap it in a `JScrollPane` when the content overflows.
- The default model from the array/`Vector` constructors is read-only; `addElement` cannot be used on it.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JList with Array and Scroll Pane

```java
import javax.swing.*;
import java.awt.*;

public class MinimalJList {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JList");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"Apple", "Banana", "Cherry", "Date", "Elderberry"};
            JList<String> list = new JList<>(items);                  // 1. Create list
            JScrollPane scroll = new JScrollPane(list);               // 2. Wrap in scroll

            list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION); // 3. Single
            list.addListSelectionListener(e -> {                      // 4. Listener
                if (!e.getValueIsAdjusting()) {
                    String selected = list.getSelectedValue();
                    System.out.println("Selected: " + selected);
                }
            });

            frame.add(scroll, BorderLayout.CENTER);                    // 5. Add
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a scrollable list of five fruits. Clicking an item prints “Selected: [fruit]” to the console. Only one item can be selected at a time.

**Why the code produces this result.** The array constructor creates a read-only `ListModel`. `setSelectionMode` restricts selection to a single item. The `ListSelectionListener` checks `getValueIsAdjusting()` to avoid firing on intermediate selection states during a drag.

---

### Real-World Cases

- **File choosers:** A `JList` of files and folders in the current directory.
- **Settings dialogs:** A list of categories or options from which the user selects one.
- **Contact lists:** A list of contacts that the user can click to view details.
- **Multi-select filters:** A list of tags or categories where multiple selections are allowed.

---

## 2. List Creation

### Definitions

**Core Definition.** List creation is the process of instantiating a `JList` with an initial set of items, either from an array, a `Vector`, or a `ListModel`.

**Technical Definition.** `JList` provides constructors that accept an array, a `Vector`, or a `ListModel`. The array and `Vector` constructors create a read-only `ListModel` internally. The `ListModel` constructor uses the supplied model directly, enabling dynamic updates if the model is mutable.

**Beginner-Friendly Explanation.** Creating a list means deciding what items it should show. You can pass an array of strings (the simplest case), a vector, or a custom model for dynamic data.

---

### Purposes

- To instantiate the list component with the appropriate initial items.
- To choose between the convenience of array-based creation and the flexibility of a custom model.
- To prepare the list for selection mode configuration, listener attachment, and scroll pane wrapping.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JList<String> list = new JList<>(new String[]{"A", "B", "C"});
JList<String> list = new JList<>(new Vector<>(List.of("A", "B")));
JList<String> list = new JList<>(customModel);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `E[] data` | Array of items to display. |
| `Vector<E> data` | Vector of items to display. |
| `ListModel<E> model` | Custom model managing the data. |

#### Syntax Rules

- The array and `Vector` constructors create a read-only model.
- A `ListModel` can be changed later with `setModel`.

#### Constraints and Limitations

- The read-only model from array/`Vector` constructors does not support `addElement` or `removeElement`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a JList with a Custom Model

```java
import javax.swing.*;
import java.awt.*;

public class ModelCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultListModel<String> model = new DefaultListModel<>(); // 1. Model
            model.addElement("Red");
            model.addElement("Green");
            model.addElement("Blue");

            JList<String> list = new JList<>(model);                   // 2. Use model
            list.setVisibleRowCount(3);                                 // 3. Height hint
            JScrollPane scroll = new JScrollPane(list);                 // 4. Scroll

            frame.add(scroll);
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A list displaying “Red,” “Green,” and “Blue” within a scroll pane sized to show three rows.

**Why the code produces this result.** The `DefaultListModel` is populated before being passed to the constructor. `setVisibleRowCount(3)` gives the scroll pane a preferred height of three rows.

---

### Real-World Cases

- **Dynamic data:** Loading items from a database into a `DefaultListModel`.
- **Shared models:** Using the same model across multiple `JList` instances.
- **Read-only data:** Using the array constructor for static lists that never change.

---

## 3. List Model — DefaultListModel

### Definitions

**Core Definition.** `DefaultListModel<E>` is the default mutable implementation of the `ListModel` interface. It provides a `java.util.Vector`-like API for managing list elements.

**Technical Definition.** `public class DefaultListModel<E> extends AbstractListModel<E>`. It implements the `ListModel` interface and also provides methods such as `addElement`, `removeElement`, `insertElementAt`, and `elementAt` that resemble the `Vector` API. When the model changes, it notifies registered `ListDataListener`s, allowing the `JList` to update its display automatically.

**Beginner-Friendly Explanation.** `DefaultListModel` is the “container” that holds the items for a `JList`. Unlike the read-only model created from an array, `DefaultListModel` lets you add, remove, and change items at runtime, and the list updates automatically.

---

### Purposes

- To provide a mutable model for dynamic list content.
- To manage list elements with a familiar `Vector`-like API.
- To notify the `JList` of changes so the display stays synchronized.
- To serve as the standard model for lists that need runtime modification.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultListModel<String> model = new DefaultListModel<>();
model.addElement("Item 1");
model.removeElement("Item 1");
model.insertElementAt("Item 0", 0);
JList<String> list = new JList<>(model);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addElement(E element)` | Appends an element to the end. |
| `removeElement(Object obj)` | Removes the first occurrence of the element. |
| `insertElementAt(E element, int index)` | Inserts an element at the given index. |
| `elementAt(int index)` | Returns the element at the given index. |

#### Syntax Rules

- Changes to the model automatically notify the `JList` and trigger a repaint.
- The model can be shared across multiple `JList` instances.

#### Constraints and Limitations

- The model is not thread-safe; modifications must occur on the EDT.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Model Updates

```java
import javax.swing.*;
import java.awt.*;

public class DynamicModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Dynamic Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultListModel<String> model = new DefaultListModel<>(); // 1. Model
            JList<String> list = new JList<>(model);
            JTextField input = new JTextField(10);
            JButton addButton = new JButton("Add");
            JButton removeButton = new JButton("Remove Selected");

            addButton.addActionListener(e -> {
                String text = input.getText().trim();
                if (!text.isEmpty()) {
                    model.addElement(text);                            // 2. Add
                    input.setText("");
                }
            });

            removeButton.addActionListener(e -> {
                int index = list.getSelectedIndex();
                if (index != -1) {
                    model.remove(index);                               // 3. Remove
                }
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(list), BorderLayout.CENTER);
            JPanel controls = new JPanel();
            controls.add(input);
            controls.add(addButton);
            controls.add(removeButton);
            panel.add(controls, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(350, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** An empty list initially. Typing an item and clicking “Add” appends it to the list. Selecting an item and clicking “Remove Selected” removes it.

**Why the code produces this result.** The `DefaultListModel` manages the data, and `addElement`/`remove` trigger model change notifications that update the `JList` automatically.

---

### Real-World Cases

- **Task managers:** Adding and removing tasks dynamically.
- **Playlist editors:** Building a list of songs.
- **Chat contact lists:** Adding new contacts as they come online.

---

## 4. List Methods — setModel()

### Definitions

**Core Definition.** `setModel(ListModel<E> model)` replaces the list’s data model with the specified model.

**Technical Definition.** `public void setModel(ListModel<E> model)`. The method sets the `model` property, notifies property change listeners, and clears the list’s selection. The new model must not be `null`.

**Beginner-Friendly Explanation.** `setModel` swaps the entire dataset of the list. The list then displays the contents of the new model and clears any previous selection.

---

### Purposes

- To replace the list’s data with a new model.
- To switch between different datasets at runtime.
- To initialize the model after the list has been created.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.setModel(newModel);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setModel(ListModel<E>)` | Replaces the data model; clears selection. |

#### Syntax Rules

- The model must not be `null`; passing `null` throws `IllegalArgumentException`.
- The selection is cleared when the model changes.

#### Constraints and Limitations

- The old model is not disposed; references to it remain valid.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Models

```java
import javax.swing.*;
import java.awt.*;

public class SetModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setModel Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultListModel<String> modelA = new DefaultListModel<>();
            modelA.addElement("Alpha");
            modelA.addElement("Beta");

            DefaultListModel<String> modelB = new DefaultListModel<>();
            modelB.addElement("Gamma");
            modelB.addElement("Delta");

            JList<String> list = new JList<>(modelA);
            JButton switchButton = new JButton("Switch Model");
            switchButton.addActionListener(e ->
                list.setModel(modelB));                                // 1. Switch

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(list), BorderLayout.CENTER);
            panel.add(switchButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The list initially shows “Alpha” and “Beta.” Clicking “Switch Model” replaces the content with “Gamma” and “Delta.”

**Why the code produces this result.** `setModel` swaps the data source, and the `JList` repaints with the new items.

---

### Real-World Cases

- **Switching datasets:** Changing the list of items based on a category selection.
- **Refreshing data:** Reloading a list from a database or file.

---

## 5. List Methods — getSelectedValue() / getSelectedValues()

### Definitions

**Core Definition.** `getSelectedValue()` returns the first selected item, or `null` if nothing is selected. `getSelectedValues()` returns all selected items as an array of `Object`, in ascending order by index. `getSelectedValuesList()` (the modern equivalent) returns a `List<E>`.

**Technical Definition.** `public E getSelectedValue()`, `public Object[] getSelectedValues()`, `public List<E> getSelectedValuesList()`. These methods delegate to the selection model and the data model to retrieve the selected objects.

**Beginner-Friendly Explanation.** `getSelectedValue()` gives you the one item the user selected (or the first of several). `getSelectedValues()` gives you all selected items as an array. Use the modern `getSelectedValuesList()` for a typed `List`.

---

### Purposes

- To read the user’s selection for processing.
- To retrieve single or multiple selected items depending on the selection mode.
- To compare selections against expected values or display them elsewhere.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Object single = list.getSelectedValue();
Object[] array = list.getSelectedValues();       // legacy
List<E> typed = list.getSelectedValuesList();    // modern
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSelectedValue()` | Returns the first selected item, or `null`. |
| `getSelectedValues()` | Returns all selected items as an array. |
| `getSelectedValuesList()` | Returns all selected items as a `List<E>`. |

#### Syntax Rules

- `getSelectedValue` returns `null` if no item is selected.
- `getSelectedValues` returns an empty array if nothing is selected.
- `getSelectedValuesList` returns an empty `List` if nothing is selected.

#### Constraints and Limitations

- `getSelectedValues()` is a legacy method; prefer `getSelectedValuesList()` for type safety.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading Selected Values

```java
import javax.swing.*;
import java.awt.*;
import java.util.List;

public class GetSelectedValuesDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Selected Values");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"A", "B", "C", "D"};
            JList<String> list = new JList<>(items);
            list.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);

            JButton showButton = new JButton("Show Selected");
            showButton.addActionListener(e -> {
                List<String> selected = list.getSelectedValuesList(); // 1. Get all
                System.out.println("Selected: " + selected);
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(list), BorderLayout.CENTER);
            panel.add(showButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting multiple items (e.g., “A” and “C”) and clicking “Show Selected” prints “Selected: [A, C]” to the console.

**Why the code produces this result.** `getSelectedValuesList()` returns all selected items in index order as a typed `List`.

---

### Real-World Cases

- **Multi-select forms:** Reading all chosen options.
- **Bulk operations:** Applying an action to all selected items.

---

## 6. List Methods — getSelectedIndex() / getSelectedIndices()

### Definitions

**Core Definition.** `getSelectedIndex()` returns the index of the first selected item, or `-1` if nothing is selected. `getSelectedIndices()` returns all selected indices in ascending order as an `int[]`.

**Technical Definition.** `public int getSelectedIndex()`, `public int[] getSelectedIndices()`. These methods delegate to the `ListSelectionModel` and are convenience wrappers.

**Beginner-Friendly Explanation.** `getSelectedIndex()` tells you the position of the selected item. `getSelectedIndices()` gives you all selected positions.

---

### Purposes

- To read the position of the selected item for array or list access.
- To determine which items are selected for multi-selection operations.
- To synchronize the list selection with other components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int index = list.getSelectedIndex();
int[] indices = list.getSelectedIndices();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSelectedIndex()` | Returns the first selected index, or `-1`. |
| `getSelectedIndices()` | Returns all selected indices as an `int[]`. |

#### Syntax Rules

- `getSelectedIndex` returns `-1` if no item is selected.
- `getSelectedIndices` returns an empty array if nothing is selected.

#### Constraints and Limitations

- None; the methods are safe to call at any time.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using Selected Indices

```java
import javax.swing.*;

public class GetSelectedIndicesDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Selected Indices");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(
                new String[]{"One", "Two", "Three", "Four"});
            list.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);

            JButton showButton = new JButton("Show Indices");
            showButton.addActionListener(e -> {
                int[] indices = list.getSelectedIndices();             // 1. Get indices
                System.out.print("Indices: ");
                for (int i : indices) {
                    System.out.print(i + " ");
                }
                System.out.println();
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(list), BorderLayout.CENTER);
            panel.add(showButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting items and clicking “Show Indices” prints the selected indices (e.g., “Indices: 0 2”).

**Why the code produces this result.** `getSelectedIndices` returns all selected positions in ascending order.

---

### Real-World Cases

- **Index-based operations:** Accessing parallel arrays or data structures.
- **Selection tracking:** Logging which positions are selected.

---

## 7. List Methods — setSelectionMode()

### Definitions

**Core Definition.** `setSelectionMode(int selectionMode)` determines whether single-item or multiple-item selections are allowed.

**Technical Definition.** `public void setSelectionMode(int selectionMode)`. The following values are allowed: `ListSelectionModel.SINGLE_SELECTION` (only one list index can be selected at a time), `ListSelectionModel.SINGLE_INTERVAL_SELECTION` (one contiguous index interval can be selected at a time), and `ListSelectionModel.MULTIPLE_INTERVAL_SELECTION` (no restriction on what can be selected; this is the default).

**Beginner-Friendly Explanation.** `setSelectionMode` controls how many items the user can select: one, one contiguous block, or any combination.

---

### Purposes

- To restrict the user to a single selection.
- To allow a single contiguous range of selections.
- To allow any combination of selections (the default).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
list.setSelectionMode(ListSelectionModel.SINGLE_INTERVAL_SELECTION);
list.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `SINGLE_SELECTION` | Only one item can be selected. |
| `SINGLE_INTERVAL_SELECTION` | One contiguous range can be selected. |
| `MULTIPLE_INTERVAL_SELECTION` | Any combination can be selected (default). |

#### Syntax Rules

- The default is `MULTIPLE_INTERVAL_SELECTION`.
- The selection mode can also be changed on the `ListSelectionModel` directly.

#### Constraints and Limitations

- Changing the mode may deselect items that violate the new mode.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Selection Modes

```java
import javax.swing.*;

public class SelectionModeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Selection Mode");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(
                new String[]{"A", "B", "C", "D", "E"});

            JComboBox<String> modeCombo = new JComboBox<>(
                new String[]{"Single", "Single Interval", "Multiple Interval"});

            modeCombo.addActionListener(e -> {
                int index = modeCombo.getSelectedIndex();
                switch (index) {
                    case 0: list.setSelectionMode(
                        ListSelectionModel.SINGLE_SELECTION); break; // 1. Single
                    case 1: list.setSelectionMode(
                        ListSelectionModel.SINGLE_INTERVAL_SELECTION); break; // 2. Interval
                    case 2: list.setSelectionMode(
                        ListSelectionModel.MULTIPLE_INTERVAL_SELECTION); break; // 3. Multiple
                }
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(list), BorderLayout.CENTER);
            panel.add(modeCombo, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A list whose selection behavior changes based on the combo box. “Single” allows only one selection; “Single Interval” allows a contiguous range; “Multiple Interval” allows any combination.

**Why the code produces this result.** `setSelectionMode` delegates to the `ListSelectionModel`, which enforces the selected mode.

---

### Real-World Cases

- **Single-selection lists:** Choosing a category or file.
- **Interval-selection lists:** Selecting a range of dates or rows.
- **Multi-selection lists:** Tag selection or bulk operations.

---

## 8. List Methods — setVisibleRowCount()

### Definitions

**Core Definition.** `setVisibleRowCount(int visibleRowCount)` sets the preferred number of rows to display without requiring scrolling (for vertical orientation).

**Technical Definition.** `public void setVisibleRowCount(int visibleRowCount)`. For a `VERTICAL` layout orientation, this sets the preferred number of rows to display without scrolling. For other orientations, it has different meanings. A value of `-1` makes the list display the maximum number of items possible in the available space.

**Beginner-Friendly Explanation.** `setVisibleRowCount` tells the `JScrollPane` how tall the list should be—how many rows are visible before scrolling.

---

### Purposes

- To control the preferred height of the list within a scroll pane.
- To ensure the list shows an appropriate number of items initially.
- To let the list size itself automatically with `-1`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.setVisibleRowCount(5);
list.setVisibleRowCount(-1); // maximum possible
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setVisibleRowCount(int)` | Sets the preferred number of visible rows. |

#### Syntax Rules

- The value is a hint; the actual height depends on the layout manager and font.
- `-1` makes the list fill the available space.

#### Constraints and Limitations

- Does not affect the actual number of items; only the visible area.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Controlling Visible Rows

```java
import javax.swing.*;

public class VisibleRowCountDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Visible Row Count");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = new String[20];
            for (int i = 0; i < items.length; i++) {
                items[i] = "Item " + (i + 1);
            }

            JList<String> list = new JList<>(items);
            list.setVisibleRowCount(5);                                // 1. 5 rows

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A scroll pane tall enough to show approximately five rows, with scroll bars for the remaining items.

**Why the code produces this result.** `setVisibleRowCount(5)` informs the scroll pane of the preferred height.

---

### Real-World Cases

- **Compact lists:** Showing 5–10 rows in a settings dialog.
- **Adaptive lists:** Using `-1` to let the list fill the available space.

---

## 9. Selection Modes — SINGLE_SELECTION

### Definitions

**Core Definition.** `SINGLE_SELECTION` is a selection mode in which only one list index can be selected at a time.

**Technical Definition.** `ListSelectionModel.SINGLE_SELECTION` (value 0). In this mode, `setSelectionInterval` and `addSelectionInterval` are equivalent, and only the second index argument is used.

**Beginner-Friendly Explanation.** Single selection means the user can pick exactly one item. Selecting a new item deselects the previous one.

---

### Purposes

- To enforce exactly one selection.
- To simplify logic when only one choice is valid.
- To mimic radio-button behavior in a list format.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `SINGLE_SELECTION` | Only one item can be selected. |

#### Syntax Rules

- The previously selected item is automatically deselected when a new one is selected.

#### Constraints and Limitations

- The user cannot select multiple items.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Single Selection Mode

```java
import javax.swing.*;

public class SingleSelectionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Single Selection");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(
                new String[]{"Red", "Green", "Blue"});
            list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION); // 1. Single

            list.addListSelectionListener(e -> {
                if (!e.getValueIsAdjusting()) {
                    System.out.println("Selected: " + list.getSelectedValue());
                }
            });

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking any item selects it and deselects the previous one. Only one item is ever selected.

**Why the code produces this result.** The selection model enforces mutual exclusivity.

---

### Real-World Cases

- **Category selection:** Choosing one category from a list.
- **File selection:** Opening a single file.

---

## 10. Selection Modes — SINGLE_INTERVAL_SELECTION

### Definitions

**Core Definition.** `SINGLE_INTERVAL_SELECTION` is a selection mode in which one contiguous index interval can be selected at a time.

**Technical Definition.** `ListSelectionModel.SINGLE_INTERVAL_SELECTION` (value 1). In this mode, `setSelectionInterval` and `addSelectionInterval` are equivalent.

**Beginner-Friendly Explanation.** The user can select a continuous block of items (e.g., items 2 through 5), but cannot select two separate blocks.

---

### Purposes

- To allow range selection without allowing disjoint selections.
- To support operations on contiguous blocks of data.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.setSelectionMode(ListSelectionModel.SINGLE_INTERVAL_SELECTION);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `SINGLE_INTERVAL_SELECTION` | One contiguous range can be selected. |

#### Syntax Rules

- Selecting a new range replaces the previous range.

#### Constraints and Limitations

- Disjoint selections are not allowed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Interval Selection Mode

```java
import javax.swing.*;

public class IntervalSelectionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Interval Selection");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(
                new String[]{"A", "B", "C", "D", "E", "F"});
            list.setSelectionMode(
                ListSelectionModel.SINGLE_INTERVAL_SELECTION);        // 1. Interval

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The user can drag to select a contiguous range (e.g., B through E) but cannot select two separate ranges.

**Why the code produces this result.** The selection model enforces a single contiguous interval.

---

### Real-World Cases

- **Date range selection:** Selecting a block of consecutive dates.
- **Row range operations:** Applying an action to a contiguous block of rows.

---

## 11. Selection Modes — MULTIPLE_INTERVAL_SELECTION

### Definitions

**Core Definition.** `MULTIPLE_INTERVAL_SELECTION` is the default selection mode, in which there is no restriction on what can be selected.

**Technical Definition.** `ListSelectionModel.MULTIPLE_INTERVAL_SELECTION` (value 2). Any combination of items can be selected, including disjoint ranges.

**Beginner-Friendly Explanation.** The user can select any number of items in any combination—individual items, ranges, or a mix.

---

### Purposes

- To allow unrestricted multi-selection.
- To support bulk operations on arbitrary sets of items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `MULTIPLE_INTERVAL_SELECTION` | No restriction on selections. |

#### Syntax Rules

- This is the default mode.

#### Constraints and Limitations

- The user must use Ctrl (or Shift) to select multiple disjoint items, depending on the look and feel.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Interval Selection

```java
import javax.swing.*;

public class MultipleSelectionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Multiple Selection");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(
                new String[]{"One", "Two", "Three", "Four", "Five"});
            list.setSelectionMode(
                ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);      // 1. Multiple

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The user can select any combination of items (e.g., “One,” “Three,” and “Five”) using Ctrl-click.

**Why the code produces this result.** The selection model places no restriction on selections.

---

### Real-World Cases

- **Tag selection:** Choosing multiple tags from a list.
- **Bulk file operations:** Selecting multiple files for deletion or copying.

---

## 12. List Events — ListSelectionListener

### Definitions

**Core Definition.** `ListSelectionListener` is an interface for receiving `ListSelectionEvent`s that fire when the selection in a list is changing or has just changed.

**Technical Definition.** `public interface ListSelectionListener extends EventListener`. The `valueChanged(ListSelectionEvent e)` method is invoked when the selection changes. The event indicates the range of indices whose selection may have changed. The listener should check `getValueIsAdjusting()` to avoid reacting to intermediate selection states during a drag operation.

**Beginner-Friendly Explanation.** A `ListSelectionListener` runs when the user selects or deselects an item. You use it to react to the new selection—for example, to display details about the selected item.

---

### Purposes

- To react to selection changes in the list.
- To update other UI components based on the selected item.
- To perform actions when the selection is finalized (after `getValueIsAdjusting()` returns `false`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
list.addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        // selection finalized
    }
});
```

#### Component Breakdown

| Element | Meaning |
|--------|---------|
| `addListSelectionListener(ListSelectionListener)` | Registers the listener. |
| `valueChanged(ListSelectionEvent e)` | Called on selection change. |
| `e.getValueIsAdjusting()` | `true` if the selection is still being adjusted. |

#### Syntax Rules

- The listener is invoked on the EDT.
- `getValueIsAdjusting()` should be checked to avoid premature reactions.
- The event does not contain the new selection; query the list via `getSelectedValue` or `getSelectedIndices`.

#### Constraints and Limitations

- The listener fires multiple times during a drag-selection; use `getValueIsAdjusting()` to filter.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reacting to Selection Changes

```java
import javax.swing.*;
import java.awt.*;

public class ListSelectionListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ListSelectionListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(
                new String[]{"Apple", "Banana", "Cherry", "Date"});
            JLabel status = new JLabel("Select an item.");

            list.addListSelectionListener(e -> {                      // 1. Listener
                if (!e.getValueIsAdjusting()) {                        // 2. Finalized
                    String selected = list.getSelectedValue();
                    status.setText(selected != null
                        ? "Selected: " + selected
                        : "Nothing selected.");
                }
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(list), BorderLayout.CENTER);
            panel.add(status, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting an item updates the status label to “Selected: [item].” Deselecting all items (if the mode allows) shows “Nothing selected.”

**Why the code produces this result.** The listener checks `getValueIsAdjusting()` and updates the label only when the selection is finalized.

---

### Real-World Cases

- **Detail views:** Showing details of the selected item in another panel.
- **Form enabling:** Enabling buttons based on whether an item is selected.
- **Multi-list synchronization:** Updating a second list based on the selection in the first.

---

## 13. JList Best Practices

### Definitions

**Core Definition.** `JList` best practices are the conventions and patterns that ensure lists are used effectively, efficiently, and maintainably in Swing applications.

**Technical Definition.** Best practices include: always wrapping `JList` in a `JScrollPane`; using `setVisibleRowCount` and `setPrototypeCellValue` to control sizing instead of `setPreferredSize`; using `DefaultListModel` for dynamic content; avoiding `null` layouts; checking `getValueIsAdjusting()` in selection listeners; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for lists are the “rules of thumb” that prevent common problems: forgetting the scroll pane, setting rigid sizes, or reacting to intermediate selection states. They include things like “always use a scroll pane,” “use the model to change data, not the list,” and “check `getValueIsAdjusting()` before reacting.”

---

### Purposes

- To ensure the list scrolls correctly when content overflows.
- To avoid layout rigidity caused by `setPreferredSize` or `setBounds`.
- To keep the list synchronized with its data model.
- To handle selection events correctly without premature reactions.
- To maintain thread safety and responsive GUIs.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    DefaultListModel<String> model = new DefaultListModel<>();     // 1. Mutable model
    JList<String> list = new JList<>(model);
    list.setVisibleRowCount(8);                                    // 2. Row hint
    list.setPrototypeCellValue("A very long prototype value");    // 3. Width hint
    JScrollPane scroll = new JScrollPane(list);                    // 4. Scroll pane
    list.addListSelectionListener(e -> {                           // 5. Listener
        if (!e.getValueIsAdjusting()) {
            // react to finalized selection
        }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use a scroll pane | `new JScrollPane(list)` |
| Use `DefaultListModel` for dynamic lists | `model.addElement` / `model.remove` |
| Size with `setVisibleRowCount` | `list.setVisibleRowCount(8)` |
| Size width with `setPrototypeCellValue` | `list.setPrototypeCellValue("Long item")` |
| Avoid `setPreferredSize` | Let the list compute its preferred size. |
| Check `getValueIsAdjusting()` | Filter intermediate selection events. |
| Avoid `null` layouts | Use layout managers. |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always wrap `JList` in a `JScrollPane`** for scrolling.
- **Always use `DefaultListModel`** (or another mutable model) for lists that change at runtime.
- **Always set `setVisibleRowCount` and `setPrototypeCellValue`** to control sizing instead of `setPreferredSize`.
- **Always check `getValueIsAdjusting()`** in `ListSelectionListener` to avoid reacting during drag-selection.
- **Never use `null` layouts or `setBounds`** on the list or its scroll pane.

#### Constraints and Limitations

- `JList` does not provide scrolling on its own; a `JScrollPane` is mandatory for overflow.
- `setPreferredSize` can prevent the scroll pane from computing the correct size and may hide scroll bars.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice JList

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeJList {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeJList::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice JList");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        DefaultListModel<String> model = new DefaultListModel<>();  // 1. Mutable model
        model.addElement("Short");
        model.addElement("Medium");
        model.addElement("A very long list item");

        JList<String> list = new JList<>(model);
        list.setVisibleRowCount(5);                                  // 2. Row hint
        list.setPrototypeCellValue("A very long list item");        // 3. Width hint
        list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION); // 4. Mode

        JLabel status = new JLabel("Select an item.");
        list.addListSelectionListener(e -> {                         // 5. Listener
            if (!e.getValueIsAdjusting()) {
                String selected = list.getSelectedValue();
                status.setText(selected != null ? "Selected: " + selected : "None");
            }
        });

        JPanel panel = new JPanel(new BorderLayout(5, 5));           // 6. Layout
        panel.add(new JScrollPane(list), BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(350, 250);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A list sized to show five rows, wide enough to display “A very long list item” without truncation. Selecting an item updates the status label. Only one item can be selected.

**Why the code produces this result.** The mutable model supports dynamic updates, `setVisibleRowCount` and `setPrototypeCellValue` control sizing, and the listener reacts only to finalized selections.

---

#### Example 2: Common Mistake — Setting Preferred Size on JList

```java
import javax.swing.*;

public class BadJList {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad JList");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = new String[50];
            for (int i = 0; i < items.length; i++) {
                items[i] = "Item " + (i + 1);
            }

            JList<String> list = new JList<>(items);
            list.setPreferredSize(new java.awt.Dimension(200, 100)); // WRONG: rigid

            frame.add(new JScrollPane(list));
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The list may not display scroll bars because `setPreferredSize` forces a fixed size that prevents the scroll pane from computing the correct preferred size. The list may appear truncated without scrolling.

**Why this is wrong.** Setting the preferred size on a `JList` prevents it from computing its own preferred size based on its content and the scroll pane’s available space. As the community guidance states: “Do not set the preferred size of the JList, this will prevent it from calculating the size of the list based on its content, which means it will never exceed the preferred size you provide, which means it will never show any scrollbars”. Instead, use `setVisibleRowCount` and `setPrototypeCellValue`.

---

### Real-World Cases

- **Production Swing applications** use `DefaultListModel` for dynamic data, `JScrollPane` for scrolling, and `setVisibleRowCount`/`setPrototypeCellValue` for sizing.
- **IDE and editor plugins** follow these patterns when embedding lists in host applications.
- **Educational examples** use the best-practice skeleton to demonstrate correct `JList` usage from the start.

---

## References

Oracle. *Class JList<E>*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JList.html

Oracle. *Class JList<E>*. Java Platform, Standard Edition 25 API Specification. https://docs.oracle.com/en/java/javase/25/docs/api/java.desktop/javax/swing/JList.html

Oracle. *How to Use Lists*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/list.html

Oracle. *How to Write a List Selection Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/listselectionlistener.html

Oracle. *Class DefaultListModel<E>*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/DefaultListModel.html

Oracle. *Interface ListSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ListSelectionModel.html

Stack Overflow. *JList best practices — avoid setPreferredSize, use setVisibleRowCount and setPrototypeCellValue*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/...