# Swing Model Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **model** in Swing is an object that holds a component's **state and data**, independent of how that state is displayed. It is the "M" in Swing's adaptation of the **Model-View-Controller (MVC)** pattern. Every non-trivial Swing component — `JList`, `JTable`, `JTree`, `JComboBox`, `JSlider`, `JSpinner`, `JProgressBar`, `JTextComponent`, `JTabbedPane`, `AbstractButton` — delegates its data and state to a model object. Models are the single source of truth for the component's content; the component (the "view") queries the model to paint itself and notifies the model of user edits.

**Technical Definition.** A Swing model is any class that implements a **model interface** declared in `javax.swing` or one of its subpackages. The key model interfaces include: `ButtonModel`, `BoundedRangeModel`, `ComboBoxModel`, `ListModel`, `SpinnerModel`, `TableModel`, `TreeModel`, `TreeSelectionModel`, `ListSelectionModel`, `SingleSelectionModel`, `ColorSelectionModel`, `MutableComboBoxModel`, and `Document` (in `javax.swing.text`). Each interface declares the accessor and mutator methods that the corresponding component requires; the component communicates with the model through these methods and through the model's **event listeners**. Models typically follow the JavaBeans pattern: they expose properties, fire `PropertyChangeEvent`s (or model-specific events such as `TableModelEvent`, `TreeModelEvent`, `ListDataEvent`, `DocumentEvent`), and are independently testable without a GUI. Swing provides default implementations (`DefaultListModel`, `DefaultTableModel`, `DefaultTreeModel`, `DefaultComboBoxModel`, `DefaultBoundedRangeModel`, `DefaultButtonModel`, `DefaultSingleSelectionModel`, `DefaultColorSelectionModel`) and abstract base classes (`AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractAction`) to reduce boilerplate. All model mutations should occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** A Swing model is the **filing cabinet** that holds a component's data — the actual list of items, the actual table rows, the actual tree nodes, the actual text. The **view** (the visible `JList`, `JTable`, `JTree`, or `JTextField`) is the **window** through which you see the filing cabinet. The **controller** (listeners, key bindings, UI delegates) is the **clerk** that takes your requests ("add this row," "select that cell") and updates the filing cabinet accordingly. You can swap out the window (change the view) or the clerk (change the controller) without disturbing the filing cabinet. You can even have **multiple windows looking into the same filing cabinet** — that is why models are so powerful. When the data changes, the model notifies all interested views, and they repaint automatically. This separation is the heart of Swing's architecture.

---

### Key Characteristics

1. **Single source of truth.** The model owns the data; the component is a projection of the model. Multiple components can share the same model and stay synchronized automatically.
2. **Interface-driven.** Every component type defines a model interface that specifies the required methods; you can supply a custom model by implementing that interface.
3. **Event-based notification.** Models fire events (`ListDataEvent`, `TableModelEvent`, `TreeModelEvent`, `ChangeEvent`, `DocumentEvent`, `PropertyChangeEvent`) to inform listeners of changes.
4. **Abstract base classes.** Swing provides abstract base classes that implement the listener bookkeeping so you only implement the data-access methods.
5. **Default implementations.** Swing provides ready-made concrete implementations for the most common cases.
6. **Headless-testable.** Models are pure Java objects that do not require a GUI; they can be unit-tested without the EDT in many cases.
7. **EDT discipline.** Models attached to live Swing components must be mutated on the EDT (with specific documented exceptions such as `AbstractDocument`'s thread-safe methods).
8. **Extensibility.** Custom models allow you to display any data structure — a database result set, a file system tree, a domain object graph — without changing the component.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, generics, anonymous inner classes, and lambda expressions.
- Familiarity with at least one Swing model-backed component: `JList`, `JTable`, `JTree`, or `JComboBox`.
- Understanding of the event delegation model and the EDT.
- Familiarity with `java.beans.PropertyChangeListener` (for models that fire property changes).
- Knowledge of `SwingUtilities.invokeLater`.
- A JDK (version 8 or later; all model interfaces and default implementations are in `javax.swing`).

---

### Related Programming Areas

- **MVC architecture:** The design pattern underlying Swing's separation of model, view, and controller.
- **`ButtonModel` and `DefaultButtonModel`:** The model behind `AbstractButton`.
- **`BoundedRangeModel` and `DefaultBoundedRangeModel`:** The model behind `JSlider`, `JProgressBar`, and `JScrollBar`.
- **`ComboBoxModel` and `MutableComboBoxModel`:** The models behind `JComboBox`.
- **`ListModel` and `DefaultListModel`:** The models behind `JList`.
- **`SpinnerModel`, `SpinnerNumberModel`, `SpinnerListModel`, and `SpinnerDateModel`:** The models behind `JSpinner`.
- **`TableModel`, `AbstractTableModel`, `DefaultTableModel`, and `TableColumnModel`:** The models behind `JTable`.
- **`TreeModel`, `DefaultTreeModel`, `TreeNode`, and `TreeSelectionModel`:** The models behind `JTree`.
- **`SingleSelectionModel` and `DefaultSingleSelectionModel`:** The models behind `JTabbedPane` and `JMenuBar`.
- **`ColorSelectionModel` and `DefaultColorSelectionModel`:** The model behind `JColorChooser`.
- **`Document` and `AbstractDocument`:** The model behind all text components.
- **`ListSelectionModel` and `DefaultListSelectionModel`:** The selection state behind `JList` and `JTable`.
- **`SwingWorker` and the EDT:** The threading infrastructure that keeps model mutations safe.

---

## Core Concepts and Features

---

## 1. Models — The Concept

### Definitions

**Core Definition.** A model is the component's data and state, separated from how it is displayed and how it is manipulated.

**Technical Definition.** A Swing model is a Java object that implements a model interface (e.g., `TableModel`, `ListModel`, `TreeModel`) and fires events when its state changes. The component registers listeners on the model to receive notifications, queries the model for its current state during painting, and notifies the model of user edits by invoking mutator methods. Models may have their own internal structure (arrays, vectors, lists, trees, database cursors) and may provide caching, lazy loading, or virtual data.

**Beginner-Friendly Explanation.** The model is the “data behind” the component. It holds the actual values, and the component is just a window that shows them.

---

### Purposes

- To separate data from presentation, so each can evolve independently.
- To allow multiple views to share the same data.
- To enable headless testing of data logic.
- To support arbitrary data structures behind standard Swing components.
- To provide a single point of change notification for dependent views.

---

### Syntax Rules and Structure

#### Complete General Syntax (Model-Backed Component)

```java
// Create a model.
DefaultListModel<String> model = new DefaultListModel<>();
model.addElement("One");
model.addElement("Two");

// Create the view with the model.
JList<String> list = new JList<>(model);

// Optionally observe the model.
list.addListSelectionListener(e -> { /* selection change */ });
model.addListDataListener(e -> { /* data change */ });
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `Model` | An object implementing the component's model interface. |
| `new DefaultListModel<>()` | A ready-made model implementation. |
| `new JList<>(model)` | Creates the view bound to the model. |
| `addXxxListener(...)` | Observes the model's changes. |

#### Syntax Rules

- The model must be created and populated **before** the component queries it for preferred sizes.
- Model mutations should occur on the EDT.
- Multiple components can share the same model instance.

#### Constraints and Limitations

- Custom models must fire the correct events; failing to do so leaves the view stale.
- Different components require different model interfaces; they are not interchangeable.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Model-View Separation with JList

```java
import javax.swing.*;
import java.awt.*;

public class ModelViewDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Model-View Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultListModel<String> model = new DefaultListModel<>();   // 1. Model
            model.addElement("Alpha");
            model.addElement("Beta");
            model.addElement("Gamma");

            JList<String> list = new JList<>(model);                    // 2. View
            JTextArea log = new JTextArea(5, 20);
            log.setEditable(false);

            model.addListDataListener(e -> {                             // 3. Observe
                log.append("Model changed: size=" + model.size() + "\n");
            });

            JButton add = new JButton("Add");
            add.addActionListener(e ->
                model.addElement("Item " + (model.size() + 1)));       // 4. Mutate

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(list), BorderLayout.CENTER);
            frame.add(new JScrollPane(log), BorderLayout.SOUTH);
            frame.add(add, BorderLayout.NORTH);
            frame.setSize(350, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The list shows Alpha, Beta, and Gamma. Each click on “Add” appends a new item to the model, the list updates automatically, and the log shows the new size.

**Why the code produces this result.** The `DefaultListModel` is the single source of truth. The `JList` queries it for display and observes it via a `ListDataListener`. When the button mutates the model, the model fires an event and the list repaints.

---

#### Example 2: Sharing One Model Across Two Views

```java
import javax.swing.*;
import java.awt.*;

public class SharedModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shared Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultListModel<String> model = new DefaultListModel<>();
            model.addElement("One");
            model.addElement("Two");
            model.addElement("Three");

            JList<String> listA = new JList<>(model);                    // 1. Share
            JList<String> listB = new JList<>(model);                    // 2. Share

            JButton add = new JButton("Add Item");
            add.addActionListener(e ->
                model.addElement("New " + model.size()));

            JPanel panel = new JPanel(new GridLayout(1, 2));
            panel.add(new JScrollPane(listA));
            panel.add(new JScrollPane(listB));

            frame.setLayout(new BorderLayout());
            frame.add(panel, BorderLayout.CENTER);
            frame.add(add, BorderLayout.SOUTH);
            frame.setSize(500, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two lists display the same items. Clicking “Add Item” adds the new item to both lists simultaneously.

**Why the code produces this result.** Both `JList` instances share the same model. When the model fires an insertion event, both views update automatically—this is the power of the model-view separation.

---

### Real-World Cases

- **Master-detail views:** A list of items and a detail panel share the same model; selecting in one updates the other.
- **Multiple perspectives:** A table and a chart display the same data model.
- **Virtual data:** A model reads from a database on demand, enabling tables with millions of rows.
- **Testing:** Model logic is tested without instantiating a GUI.

---

## 2. The MVC Pattern in Swing

### Definitions

**Core Definition.** MVC is the architectural pattern that separates an application into three concerns: **Model** (data and rules), **View** (presentation), and **Controller** (input handling). Swing adapts the classical pattern into a **separable model architecture** sometimes called **Model-Delegate**.

**Technical Definition.** In Swing's **separable model architecture**, the "C" in MVC is distributed between the component's **UI delegate** (`ComponentUI`) and the **event listeners** the application registers. The component (view) paints using the UI delegate; the model holds the data; and event listeners (controller) respond to input and mutate the model. This adaptation is often called **model-delegate** or **separated model** architecture. The model is unaware of the view, the view is unaware of other views, and all communication flows through the model's event notifications.

**Beginner-Friendly Explanation.** MVC splits your GUI into three roles. The **model** is the data. The **view** is the screen. The **controller** is the wiring that turns user input into model changes. Swing's twist is that the controller is often spread between the component's look-and-feel delegate and your listeners—but the essential separation (data vs. presentation) is preserved.

---

### Purposes

- To enable multiple views of the same data.
- To decouple data logic from presentation logic.
- To allow independent testing and evolution of each concern.
- To support look-and-feel independence through the UI delegate.
- To provide a clean architecture for complex applications.

---

### Syntax Rules and Structure

#### Complete General Syntax (MVC Components)

```java
// Model
DefaultListModel<String> model = new DefaultListModel<>();

// View
JList<String> view = new JList<>(model);

// Controller (listener on the model or on other controls)
view.addListSelectionListener(e -> { /* react */ });
someButton.addActionListener(e -> model.addElement("New"));
```

#### Component Breakdown

| Layer | Role | Example |
|-------|------|---------|
| Model | Data and rules | `DefaultListModel`, `DefaultTableModel` |
| View | Presentation | `JList`, `JTable`, `JTree` |
| Controller | Input handling | `ListSelectionListener`, `ActionListener`, UI delegate |

#### Syntax Rules

- The model must not reference the view.
- The view must not contain business logic; it displays model state.
- The controller mutates the model; it does not mutate the view directly.
- Model events notify all registered views.

#### Constraints and Limitations

- Swing's "controller" is distributed (UI delegate + listeners), which can complicate strict MVC interpretations.
- Shared models must be thread-safe on the EDT; off-EDT mutations are the application's responsibility.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Explicit MVC Layers

```java
import javax.swing.*;
import java.awt.*;

public class MvcDemo {
    // Model: a simple count.
    static class CounterModel {
        private int value = 0;
        private final java.util.List<java.beans.PropertyChangeListener> listeners
            = new java.util.ArrayList<>();

        public int getValue() { return value; }

        public void increment() {
            int old = value;
            value++;
            for (var l : listeners)
                l.propertyChange(new java.beans.PropertyChangeEvent(
                    this, "value", old, value));
        }

        public void addListener(java.beans.PropertyChangeListener l) {
            listeners.add(l);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("MVC Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CounterModel model = new CounterModel();                  // 1. Model

            JLabel view = new JLabel("Count: 0", SwingConstants.CENTER); // 2. View
            model.addListener(evt ->                                    // 3. Controller
                view.setText("Count: " + evt.getNewValue()));

            JButton inc = new JButton("Increment");
            inc.addActionListener(e -> model.increment());              // 4. Controller

            frame.setLayout(new BorderLayout());
            frame.add(view, BorderLayout.CENTER);
            frame.add(inc, BorderLayout.SOUTH);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A label showing “Count: 0” and a button. Each click increments the model, the model fires a property change, and the label updates.

**Why the code produces this result.** The model owns the data and fires `PropertyChangeEvent`s. The view displays model state. The controller (listener + button action) mutates the model. Each layer is independent.

---

#### Example 2: Swing's Separable Model Architecture in Action

```java
import javax.swing.*;
import java.awt.*;

public class SeparableDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Separable Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            // Model: independent of any view.
            DefaultListModel<String> model = new DefaultListModel<>();
            model.addElement("One");
            model.addElement("Two");

            // View: reads from model.
            JList<String> list = new JList<>(model);
            list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);

            // Separate selection model.
            ListSelectionModel selectionModel = list.getSelectionModel();

            // Controller: reacts to selection and data changes.
            selectionModel.addListSelectionListener(e -> {
                if (!e.getValueIsAdjusting()) {
                    System.out.println("Selected: " + list.getSelectedValue());
                }
            });

            // UI delegate paints the list; application does not.
            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The list displays One and Two. Selecting an item prints “Selected: [item]” to the console.

**Why the code produces this result.** The data model (`DefaultListModel`), the view (`JList`), the selection model (`ListSelectionModel`), the painting delegate (`ListUI`), and the controller (the listener) are all separate objects. Each can be replaced without affecting the others.

---

### Real-World Cases

- **Multi-document editors:** Each document has its own text model; multiple views (split editors) share it.
- **Data dashboards:** A single data model feeds a table, a chart, and a detail panel.
- **Configuration editors:** The model holds settings; the view displays and edits them; validation lives in the controller.

---

## 3. Model Types

### 3.1 ButtonModel

**Definitions.** **Core:** The model behind `AbstractButton` (buttons, check boxes, radio buttons, toggle buttons). **Technical:** `public interface ButtonModel extends ItemSelectable`, with `DefaultButtonModel` as the default implementation. Tracks `pressed`, `armed`, `rollover`, `selected`, and `enabled`. **Beginner:** The on/off/pressed state of a button.

**Purposes.** To track button state; to fire `ItemEvent`s when selection changes; to support button groups; to synchronize buttons sharing the same model.

**Syntax Rules.** `button.getModel()` returns the model; `setSelected`, `setPressed`, `setArmed`, `setEnabled` modify it; `addItemListener`, `addActionListener`, `addChangeListener` observe it.

**Constraints.** Directly modifying the model bypasses the button’s own action listeners; prefer `button.doClick()` for simulated clicks.

**Example.**

```java
JCheckBox cb = new JCheckBox("Option");
cb.getModel().addItemListener(e -> System.out.println("Selected: " + cb.isSelected()));
```

**Expected Output.** Toggling the check box prints “Selected: true/false”.

**Real-World.** Check boxes, radio buttons, toggle buttons.

---

### 3.2 BoundedRangeModel

**Definitions.** **Core:** The model behind `JSlider`, `JProgressBar`, and `JScrollBar`, managing `minimum`, `maximum`, `value`, and `extent`. **Technical:** `public interface BoundedRangeModel`, with `DefaultBoundedRangeModel`. **Beginner:** The range and current value of a slider, progress bar, or scroll bar.

**Purposes.** To track the value of sliders and progress bars; to share values across components; to enforce `minimum ≤ value ≤ maximum`.

**Syntax Rules.** `slider.getModel().setValue(50);`, `model.getValueIsAdjusting()` distinguishes intermediate and final changes.

**Constraints.** `setValue` fires `ChangeEvent`s; guard against recursion.

**Example.**

```java
JSlider slider = new JSlider(0, 100, 50);
slider.getModel().addChangeListener(e -> System.out.println(slider.getValue()));
```

**Real-World.** Sliders, progress bars, scroll bars.

---

### 3.3 ComboBoxModel and MutableComboBoxModel

**Definitions.** **Core:** The models behind `JComboBox`. `ComboBoxModel` extends `ListModel` and adds `getSelectedItem`/`setSelectedItem`. `MutableComboBoxModel` adds `addElement`/`removeElement`/`insertElementAt`/`removeElementAt`. **Technical:** `DefaultComboBoxModel` is the concrete implementation. **Beginner:** The list of items in a combo box and the currently selected item.

**Purposes.** To populate and manage combo box items; to track the selected item; to enable dynamic add/remove.

**Syntax Rules.** `combo.setModel(new DefaultComboBoxModel<>(items));` or `combo.addItem(...)` for mutable models.

**Example.**

```java
DefaultComboBoxModel<String> model = new DefaultComboBoxModel<>();
model.addElement("Small");
model.addElement("Medium");
JComboBox<String> combo = new JComboBox<>(model);
combo.addItemListener(e -> System.out.println("Selected: " + combo.getSelectedItem()));
```

**Real-World.** Font selection, country selection, category filters.

---

### 3.4 ListModel

**Definitions.** **Core:** The model behind `JList`. **Technical:** `public interface ListModel<E>` with `getSize`, `getElementAt`, and `addListDataListener`/`removeListDataListener`. `DefaultListModel<E>` provides a mutable implementation with `addElement`, `insertElementAt`, `remove`, `removeAllElements`. **Beginner:** The items in a list.

**Purposes.** To hold list items; to fire `ListDataEvent`s on changes; to enable custom backing stores.

**Syntax Rules.** `model.addElement(x)` fires an insertion event; `model.remove(index)` fires a removal event. `AbstractListModel` is the base for custom models.

**Constraints.** Mutations must occur on the EDT; the model must fire the correct event type.

**Example.**

```java
DefaultListModel<String> model = new DefaultListModel<>();
model.addElement("A");
model.addElement("B");
JList<String> list = new JList<>(model);
model.addListDataListener(e -> System.out.println("Size: " + model.size()));
```

**Real-World.** Item lists, contact lists, file lists.

---

### 3.5 SpinnerModel

**Definitions.** **Core:** The model behind `JSpinner`. Subinterfaces/implementations: `SpinnerNumberModel`, `SpinnerListModel`, `SpinnerDateModel`. **Technical:** `public interface SpinnerModel` with `getValue`, `setValue`, `getNextValue`, `getPreviousValue`, `addChangeListener`, `removeChangeListener`. **Beginner:** The sequence of values a spinner cycles through.

**Purposes.** To define the sequence of values; to fire `ChangeEvent`s on value change; to support numeric, list-based, and date-based spinners.

**Syntax Rules.** `new JSpinner(new SpinnerNumberModel(10, 0, 100, 5));` creates a spinner with a defined sequence.

**Example.**

```java
JSpinner spinner = new JSpinner(new SpinnerNumberModel(10, 0, 100, 5));
spinner.addChangeListener(e -> System.out.println(spinner.getValue()));
```

**Real-World.** Quantity selectors, font size selectors, date pickers.

---

### 3.6 TableModel

**Definitions.** **Core:** The model behind `JTable`. **Technical:** `public interface TableModel` with `getRowCount`, `getColumnCount`, `getValueAt`, `setValueAt`, `isCellEditable`, `getColumnName`, `getColumnClass`, `addTableModelListener`, `removeTableModelListener`. `AbstractTableModel` and `DefaultTableModel` are the standard base and concrete implementations. `TableColumnModel` manages columns. **Beginner:** The rows and columns of a table.

**Purposes.** To hold table data; to fire `TableModelEvent`s on insertions, updates, and deletions; to support custom data sources; to enable sorting and filtering via `RowSorter`.

**Syntax Rules.** Custom models extend `AbstractTableModel` and call `fireTableDataChanged`, `fireTableRowsInserted`, `fireTableRowsDeleted`, or `fireTableCellUpdated`.

**Constraints.** `DefaultTableModel` uses a `Vector`; for custom data structures, extend `AbstractTableModel`.

**Example.**

```java
AbstractTableModel model = new AbstractTableModel() {
    String[][] data = {{"Alice", "30"}, {"Bob", "25"}};
    String[] cols = {"Name", "Age"};
    public int getRowCount() { return data.length; }
    public int getColumnCount() { return cols.length; }
    public Object getValueAt(int r, int c) { return data[r][c]; }
    public String getColumnName(int c) { return cols[c]; }
};
JTable table = new JTable(model);
```

**Real-World.** Spreadsheets, data browsers, admin consoles.

---

### 3.7 TreeModel

**Definitions.** **Core:** The model behind `JTree`. **Technical:** `public interface TreeModel` with `getRoot`, `getChild`, `getChildCount`, `isLeaf`, `getIndexOfChild`, `getPathToRoot`, `addTreeModelListener`, `removeTreeModelListener`. `DefaultTreeModel` and `DefaultMutableTreeNode` are the standard implementations. **Beginner:** The hierarchical data displayed by a tree.

**Purposes.** To hold tree structure; to fire `TreeModelEvent`s on changes; to enable custom node types; to support lazy loading.

**Syntax Rules.** `model.insertNodeInto(newChild, parent, index)` and `model.removeNodeFromParent(node)` mutate and fire events.

**Example.**

```java
DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
root.add(new DefaultMutableTreeNode("Child"));
DefaultTreeModel model = new DefaultTreeModel(root);
JTree tree = new JTree(model);
model.addTreeModelListener(e -> System.out.println("Tree changed"));
```

**Real-World.** File explorers, organizational charts, document outlines.

---

### 3.8 TreeSelectionModel and ListSelectionModel

**Definitions.** **Core:** The models that track **selection state** for trees and lists/tables. **Technical:** `TreeSelectionModel` and `ListSelectionModel` interfaces; `DefaultTreeSelectionModel` and `DefaultListSelectionModel` implementations. Both support `SINGLE_SELECTION`, `SINGLE_INTERVAL_SELECTION`, and `MULTIPLE_INTERVAL_SELECTION`. **Beginner:** Which items are currently selected.

**Purposes.** To track which items are selected; to support shared selection models across views; to fire selection events.

**Example.**

```java
JList<String> a = new JList<>(items);
JList<String> b = new JList<>(items);
b.setSelectionModel(a.getSelectionModel());                          // shared
```

**Real-World.** Linked list views, synchronized master-detail selections.

---

### 3.9 SingleSelectionModel

**Definitions.** **Core:** The model behind `JTabbedPane` and `JMenuBar`, tracking a single selected index. **Technical:** `public interface SingleSelectionModel` with `getSelectedIndex`, `setSelectedIndex`, `getSelectedModel`, `setSelectedModel`, `clearSelection`, `addChangeListener`, `removeChangeListener`. `DefaultSingleSelectionModel` is the implementation.

**Purposes.** To track the selected tab or menu; to fire `ChangeEvent`s on selection change; to support shared selection across tabs.

**Example.**

```java
JTabbedPane tabs = new JTabbedPane();
tabs.getModel().addChangeListener(e ->
    System.out.println("Tab: " + tabs.getSelectedIndex()));
```

**Real-World.** Tab selection, menu selection.

---

### 3.10 ColorSelectionModel

**Definitions.** **Core:** The model behind `JColorChooser`. **Technical:** `public interface ColorSelectionModel` with `getSelectedColor`, `setSelectedColor`, `addChangeListener`, `removeChangeListener`. `DefaultColorSelectionModel` is the implementation.

**Purposes.** To hold the selected color; to fire `ChangeEvent`s on change; to allow live preview.

**Example.**

```java
JColorChooser chooser = new JColorChooser();
chooser.getSelectionModel().addChangeListener(e ->
    System.out.println(chooser.getColor()));
```

**Real-World.** Color pickers, theme editors.

---

### 3.11 Document

**Definitions.** **Core:** The model behind all `JTextComponent` subclasses (`JTextField`, `JTextArea`, `JTextPane`, `JEditorPane`). **Technical:** `public interface Document` and `public abstract class AbstractDocument`. Subinterfaces include `StyledDocument` and `PlainDocument`. Provides text storage, editing API (`insertString`, `remove`), and `DocumentListener` notifications. **Beginner:** The text content of a text component.

**Purposes.** To store text content; to support undo/redo, filtering, and styled text; to fire `DocumentEvent`s on changes.

**Example.**

```java
JTextField field = new JTextField(20);
field.getDocument().addDocumentListener(new DocumentListener() {
    public void insertUpdate(DocumentEvent e) { updateCount(); }
    public void removeUpdate(DocumentEvent e) { updateCount(); }
    public void changedUpdate(DocumentEvent e) {}
    void updateCount() { System.out.println(field.getText().length()); }
});
```

**Real-World.** Text input validation, character counters, styled editors.

---

### Real-World Cases (Model Types)

- **Enterprise applications:** custom `TableModel` backed by database cursors for huge tables.
- **IDEs:** custom `TreeModel` for project explorers, with lazy-loading `TreeExpansionListener`s.
- **Word processors:** `StyledDocument` models with custom `DocumentFilter`s for input validation.
- **Configuration editors:** `PropertyChangeListener`-driven models for live settings.

---

## 4. Model Best Practices

### Definitions

**Core Definition.** Model best practices are the conventions and patterns that ensure Swing models are correct, thread-safe, testable, and maintainable.

**Technical Definition.** Best practices include: implementing custom models by extending the appropriate abstract base class (`AbstractTableModel`, `AbstractListModel`, `AbstractSpinnerModel`); using the model's notification methods (`fireTableChanged`, `fireListDataChanged`, `fireTreeNodesInserted`) so views update; mutating models only on the EDT; separating model code from view code for testability; using `RowSorter` for sorting rather than sorting the model itself; and avoiding `DefaultTableModel` for custom data structures.

**Beginner-Friendly Explanation.** Best practices for models are the “rules of thumb” that prevent stale views, frozen GUIs, and unmaintainable code. They include things like “extend the abstract base class,” “fire the right events,” “use the EDT,” and “keep data logic out of the view.”

---

### Purposes

- To keep views synchronized with data through proper event firing.
- To maintain thread safety by respecting the EDT.
- To enable headless testing of data logic.
- To support arbitrary data structures without rewriting the view.
- To avoid performance pitfalls from unnecessary model updates.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
// 1. Extend the abstract base class for custom models.
public class MyTableModel extends AbstractTableModel {
    private final java.util.List<Row> rows = new java.util.ArrayList<>();

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return 2; }
    public Object getValueAt(int r, int c) { /* ... */ }

    public void addRow(Row row) {
        int index = rows.size();
        rows.add(row);
        fireTableRowsInserted(index, index);                        // 2. Fire event
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend abstract base | `AbstractTableModel`, `AbstractListModel`, `AbstractSpinnerModel` |
| Fire events on change | `fireTableRowsInserted`, `fireListDataChanged`, `fireTreeNodesInserted` |
| Use the EDT | `SwingUtilities.invokeLater(...)` for mutations |
| Separate concerns | Model has no view references; view has no business logic |
| Use `RowSorter` | `table.setAutoCreateRowSorter(true)` for sorting |
| Avoid `DefaultTableModel` for custom data | Use `AbstractTableModel` with your own `List` |

#### Syntax Rules

- **Always extend the abstract base class** for custom models; it handles listener bookkeeping.
- **Always fire the appropriate event method** after mutating the model; otherwise the view stays stale.
- **Always mutate models on the EDT** when they are attached to live components.
- **Always separate model logic from view logic** for testability and reuse.
- **Never use `DefaultTableModel` for custom data structures**; extend `AbstractTableModel` instead.

#### Constraints and Limitations

- Custom models must fire the **correct** event type for the **correct** range; firing too broad an event causes inefficient repaints.
- Off-EDT model mutations can cause `ConcurrentModificationException` or display corruption.
- Some models (e.g., `AbstractDocument`) support off-EDT mutation, but the listeners are notified on the mutating thread and must be thread-safe.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom `TableModel`

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class BestPracticeTableModel extends AbstractTableModel {
    public static class Person {
        String name; int age;
        Person(String n, int a) { name = n; age = a; }
    }

    private final List<Person> people = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    public int getRowCount() { return people.size(); }
    public int getColumnCount() { return cols.length; }
    public String getColumnName(int c) { return cols[c]; }
    public Object getValueAt(int r, int c) {
        Person p = people.get(r);
        return c == 0 ? p.name : p.age;
    }

    public void addPerson(Person p) {                               // 1. Mutate
        int index = people.size();
        people.add(p);
        fireTableRowsInserted(index, index);                        // 2. Notify
    }

    public void updateAge(int row, int age) {                       // 3. Update
        people.get(row).age = age;
        fireTableCellUpdated(row, 1);                               // 4. Notify
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice TableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            BestPracticeTableModel model = new BestPracticeTableModel();
            model.addPerson(new Person("Alice", 30));
            model.addPerson(new Person("Bob", 25));

            JTable table = new JTable(model);
            JButton add = new JButton("Add Row");
            add.addActionListener(e ->
                model.addPerson(new Person("New", 0)));

            frame.setLayout(new java.awt.BorderLayout());
            frame.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            frame.add(add, java.awt.BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A table with “Alice” and “Bob.” Clicking “Add Row” appends “New, 0” to the model and the table updates automatically.

**Why the code produces this result.** The custom model extends `AbstractTableModel` and fires the correct events (`fireTableRowsInserted`, `fireTableCellUpdated`) after mutations. The view repaints only the affected rows.

---

#### Example 2: Common Mistake — Mutating the Model Without Firing Events

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class BadTableModel extends AbstractTableModel {
    private final List<String> items = new ArrayList<>();

    public int getRowCount() { return items.size(); }
    public int getColumnCount() { return 1; }
    public Object getValueAt(int r, int c) { return items.get(r); }

    public void addItem(String item) {
        items.add(item);                                            // WRONG: no event
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad TableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            BadTableModel model = new BadTableModel();
            JTable table = new JTable(model);
            JButton add = new JButton("Add");
            add.addActionListener(e -> model.addItem("X"));

            frame.setLayout(new java.awt.BorderLayout());
            frame.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            frame.add(add, java.awt.BorderLayout.SOUTH);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Add” adds items to the internal list, but the table does **not** update because no `TableModelEvent` was fired.

**Why this is wrong.** The view relies on model events to know when to repaint. Mutating the internal list without calling `fireTableRowsInserted` leaves the view stale. The correct approach is to call the appropriate `fireXxx` method after every mutation.

---

#### Example 3: Common Mistake — Using `DefaultTableModel` for Custom Data

```java
import javax.swing.table.DefaultTableModel;

public class BadCustomModel extends DefaultTableModel {             // WRONG
    // Trying to back this with a custom data structure is awkward.
}
```

**Expected Output.** The class works, but the model stores data in a `Vector` of `Vector`s, so you would have to synchronize a custom data structure with the internal vectors manually.

**Why this is wrong.** As the Java documentation advises, extend `AbstractTableModel` for custom data, and use `DefaultTableModel` only when no subclassing is required. Extending `DefaultTableModel` couples your model to its internal `Vector` representation and makes it harder to use domain objects efficiently.

---

### Real-World Cases

- **Production Swing applications** extend `AbstractTableModel`, `AbstractListModel`, or implement `TreeModel` for custom data.
- **Database-backed tables** use custom `TableModel`s that lazily fetch rows on demand.
- **Large data sets** rely on `RowSorter` for sorting rather than sorting the model itself.
- **Educational examples** use the best-practice skeleton to demonstrate correct model design from the start.

---

## References

Oracle. *Package javax.swing*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html

Oracle. *A Swing Architecture Overview*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/

Oracle. *Interface ButtonModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ButtonModel.html

Oracle. *Interface BoundedRangeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/BoundedRangeModel.html

Oracle. *Interface ComboBoxModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ComboBoxModel.html

Oracle. *Interface ListModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ListModel.html

Oracle. *Interface SpinnerModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SpinnerModel.html

Oracle. *Interface TableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/TableModel.html

Oracle. *Interface TreeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/TreeModel.html

Oracle. *Interface ListSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ListSelectionModel.html

Oracle. *Interface TreeSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/TreeSelectionModel.html

Oracle. *Interface SingleSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SingleSelectionModel.html

Oracle. *Interface ColorSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/colorchooser/ColorSelectionModel.html

Oracle. *Interface Document*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/Document.html

Oracle. *Class AbstractTableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/AbstractTableModel.html

Oracle. *Class AbstractListModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractListModel.html

Oracle. *Class DefaultListModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/DefaultListModel.html

Oracle. *Class DefaultTableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/DefaultTableModel.html

Oracle. *Class DefaultTreeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/DefaultTreeModel.html

Oracle. *Class DefaultButtonModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/DefaultButtonModel.html

Oracle. *Class Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *How to Use Tables — Creating a Table Model*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/table.html#data