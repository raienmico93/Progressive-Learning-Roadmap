# Custom Models — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **custom model** in Swing is a user-written class that implements one of Swing's model interfaces — `ListModel`, `TableModel`, `TreeModel`, `ComboBoxModel`, `SpinnerModel`, `BoundedRangeModel`, `ButtonModel`, `Document`, or `SingleSelectionModel` — to supply data to a component from a source that is not the standard default implementation. Custom models are the recommended approach when the data lives in a structure that cannot — or should not — be forced into the default model's storage.

**Technical Definition.** A custom model is a class that fully or partially implements a Swing model interface. Swing recommends implementing custom models by **extending the abstract base class** for the interface (`AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractDocument`, etc.) because these base classes provide the listener bookkeeping and event-firing machinery while leaving only the data-access methods abstract. The model must notify registered listeners by invoking the appropriate `fireXxx` methods after every mutation; failure to do so leaves the view stale. Custom models often wrap domain objects (a `List<Person>`, a `ResultSet`, a `File` tree, a domain-specific sequence) and provide adapters between the domain representation and the model interface. All model mutations should occur on the Event Dispatch Thread (EDT) when the model is attached to a live Swing component.

**Beginner-Friendly Explanation.** A **custom model** is the "data brain" you write yourself when the ready-made models (`DefaultListModel`, `DefaultTableModel`, `DefaultTreeModel`, `DefaultComboBoxModel`, `PlainDocument`, etc.) do not fit your data. Suppose your application already has a `List<Person>` loaded from a database, or a `File` hierarchy, or a domain-specific sequence of enum values. Converting all of that into the "standard" model structures would waste memory and time. A custom model lets you **expose your own data directly** to a Swing component. Swing gives you a generous head start: instead of implementing the whole model interface from scratch, you extend an **abstract base class** that already handles the plumbing (listener registration, event firing). You provide only the data-access methods, and you call the `fireXxx` methods whenever your data changes. Think of it as the difference between buying a fully furnished apartment (`DefaultListModel`) and having an architect design one for you (`AbstractListModel`): the latter is more work, but you get exactly what your data needs.

---

### Key Characteristics

1. **Interface-driven extensibility.** Every Swing component declares a model interface; any class that implements it can drive the component.
2. **Abstract base classes reduce boilerplate.** Swing provides `AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractDocument`, and similar classes that implement the listener plumbing.
3. **Event-firing responsibility.** Unlike the default models, custom models must fire the correct `XxxEvent` after every mutation; failing to do so leaves the view stale.
4. **Adapter pattern.** Custom models frequently adapt domain objects (`Person`, `Order`, `File`, enums) to the model interface without copying data.
5. **Read-only vs. mutable.** Custom models can be read-only (e.g., a virtual list of computed values) or mutable (with add/remove operations).
6. **Headless-testable.** Custom models are pure Java objects; they can be unit-tested without a GUI.
7. **EDT discipline.** Mutations to a custom model attached to a live component must occur on the EDT; events fired off the EDT are delivered on the mutating thread and can corrupt the view.
8. **Lazy and virtual data.** Custom models are the standard way to expose large or on-demand datasets (database cursors, computed values) to Swing components.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, generics, and lambda expressions.
- Familiarity with the Swing model interfaces: `ListModel`, `TableModel`, `TreeModel`, `ComboBoxModel`, `SpinnerModel`, `BoundedRangeModel`, `Document`, `ButtonModel`.
- Understanding of the corresponding abstract base classes: `AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractDocument`.
- Familiarity with the corresponding listener and event types: `ListDataListener`/`ListDataEvent`, `TableModelListener`/`TableModelEvent`, `TreeModelListener`/`TreeModelEvent`, `ChangeListener`/`ChangeEvent`, `DocumentListener`/`DocumentEvent`.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with the Java collections framework (`List`, `ArrayList`, `Map`).
- A JDK (version 8 or later; all model interfaces and base classes are in `javax.swing` and `javax.swing.text`).

---

### Related Programming Areas

- **Swing MVC architecture:** The separable model architecture that makes custom models possible.
- **`AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractDocument`:** The abstract base classes for custom models.
- **`ListDataEvent`, `TableModelEvent`, `TreeModelEvent`, `ChangeEvent`, `DocumentEvent`:** The event types that custom models must fire.
- **`RowSorter` and `TableRowSorter`:** Sorting and filtering for custom `TableModel`s without modifying the model.
- **`DefaultListModel`, `DefaultTableModel`, `DefaultTreeModel`, `DefaultComboBoxModel`:** The ready-made implementations to compare against.
- **`SwingWorker`:** For loading large datasets into custom models in the background.
- **`DocumentFilter`:** The companion to custom `Document` implementations for input transformation.
- **EDT and thread safety:** The threading rules that apply to all model mutations.

---

## Core Concepts and Features

---

## 1. Custom Models — The Concept

### Definitions

**Core Definition.** A custom model is any class that implements a Swing model interface and supplies data from a non-default source, typically by extending the interface's abstract base class.

**Technical Definition.** A custom model implements the abstract data-access methods of its model interface and (for the abstract base class route) inherits the listener bookkeeping. It must fire the appropriate events when its data changes. Custom models often use the **adapter pattern** to translate domain data into the model's accessor methods (`getSize`/`getElementAt`, `getRowCount`/`getColumnCount`/`getValueAt`, `getRoot`/`getChild`, etc.).

**Beginner-Friendly Explanation.** A custom model is the "model you write yourself" when the standard models do not fit your data.

---

### Purposes

- To expose domain data (lists of objects, database results, file trees) directly to Swing components without copying.
- To support very large or on-demand data that cannot be stored in a `Vector`.
- To implement read-only or virtual data sources.
- To add specialised validation, transformation, or notification logic.
- To centralise data logic in a testable, reusable class.
- To support shared data across multiple views.

---

### Syntax Rules and Structure

#### Complete General Syntax (Choosing a Base Class)

```java
// Custom list model:
public class MyListModel extends AbstractListModel<E> { /* ... */ }

// Custom table model:
public class MyTableModel extends AbstractTableModel { /* ... */ }

// Custom tree model (no abstract base class; implement directly):
public class MyTreeModel implements TreeModel { /* ... */ }

// Custom spinner model:
public class MySpinnerModel extends AbstractSpinnerModel { /* ... */ }

// Custom document:
public class MyDocument extends PlainDocument { /* ... */ }
```

#### Component Breakdown

| Base Class | For Custom Model Of |
|------------|---------------------|
| `AbstractListModel` | `ListModel` (also `ComboBoxModel` with more work) |
| `AbstractTableModel` | `TableModel` |
| `AbstractSpinnerModel` | `SpinnerModel` |
| `PlainDocument` / `AbstractDocument` | `Document` |
| `DefaultTreeModel` (or implement `TreeModel` directly) | `TreeModel` |
| `DefaultBoundedRangeModel` (or implement `BoundedRangeModel` directly) | `BoundedRangeModel` |
| `DefaultButtonModel` (or implement `ButtonModel` directly) | `ButtonModel` |

#### Syntax Rules

- Extend the abstract base class whenever one is available.
- Implement the abstract data-access methods only.
- Fire the appropriate event method after every mutation.

#### Constraints and Limitations

- `TreeModel` and `ComboBoxModel` have no dedicated abstract base class in `javax.swing` (though `DefaultTreeModel` and `DefaultComboBoxModel` can serve as starting points); they must be implemented directly or by subclassing the defaults.
- `ButtonModel` is rarely worth implementing from scratch; `DefaultButtonModel` is almost always sufficient.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Overview — Choosing the Right Base

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;

// 1. A custom list model extending AbstractListModel
class FruitListModel extends AbstractListModel<String> {
    private final String[] fruits = {"Apple", "Banana", "Cherry"};
    public int getSize() { return fruits.length; }
    public String getElementAt(int i) { return fruits[i]; }
}

// 2. A custom table model extending AbstractTableModel
class FruitTableModel extends AbstractTableModel {
    private final String[] fruits = {"Apple", "Banana", "Cherry"};
    public int getRowCount() { return fruits.length; }
    public int getColumnCount() { return 1; }
    public Object getValueAt(int r, int c) { return fruits[r]; }
    public String getColumnName(int c) { return "Fruit"; }
}

public class CustomModelOverview {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom Models Overview");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JList<String> list = new JList<>(new FruitListModel());
            JTable table = new JTable(new FruitTableModel());

            frame.setLayout(new java.awt.GridLayout(1, 2));
            frame.add(new JScrollPane(list));
            frame.add(new JScrollPane(table));
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two views of the same underlying fruits array: a `JList` on the left and a `JTable` on the right. Both display Apple, Banana, Cherry.

**Why the code produces this result.** Each custom model wraps the same array in a different model interface. The `JList` and `JTable` both query their respective models for display data. No event firing is needed because the data never changes.

---

### Real-World Cases

- **Domain-object lists:** A `List<Person>` exposed via a custom `AbstractListModel`.
- **Database tables:** A custom `AbstractTableModel` backed by a `ResultSet`.
- **File trees:** A custom `TreeModel` wrapping `java.io.File`.
- **Domain-specific spinners:** A custom `AbstractSpinnerModel` for enums or non-numeric sequences.

---

## 2. Model Implementation — Choosing the Base Class

### Definitions

**Core Definition.** Model implementation is the process of writing the class that supplies data to a Swing component, starting from the most appropriate base class and implementing only the required methods.

**Technical Definition.** The choice of base class determines how much plumbing you inherit. Swing's abstract base classes (`AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractDocument`) implement the listener registration methods and provide protected `fireXxx` methods, but leave data-access methods abstract. Direct interface implementation (`TreeModel`, `ComboBoxModel`) requires you to manage listeners manually. Subclassing the default implementations (`DefaultTreeModel`, `DefaultComboBoxModel`, `DefaultListModel`) gives you a working model you can extend.

**Beginner-Friendly Explanation.** Choosing the base class is the first decision when writing a custom model. Pick the base class that gives you the most help while still letting you own the data.

---

### Purposes

- To minimize boilerplate by inheriting listener bookkeeping from the abstract base class.
- To match the base class's built-in assumptions (data structure, event firing) with the actual data.
- To enable extension of default models when they are almost — but not quite — sufficient.
- To guide the choice between abstract base classes, default implementations, and direct interface implementation.

---

### Syntax Rules and Structure

#### Complete General Syntax (Decision Matrix)

```java
// Choose based on the scenario:

// Abstract base class — custom data, most boilerplate removed
class M1 extends AbstractListModel<E> { ... }
class M2 extends AbstractTableModel { ... }
class M3 extends AbstractSpinnerModel { ... }

// Direct interface implementation — full control, no boilerplate removed
class M4 implements TreeModel { ... }
class M5 implements ComboBoxModel<E> { ... }

// Default model subclass — extend the standard implementation
class M6 extends DefaultListModel<E> { ... }
class M7 extends DefaultTableModel { ... }
```

#### Component Breakdown

| Situation | Recommended Base |
|-----------|------------------|
| Custom data, no wrapper needed | Abstract base class |
| Data already in `TreeNode` structure | `DefaultTreeModel` |
| Data already in `Vector` structure | `DefaultListModel` / `DefaultTableModel` |
| Non-`TreeNode` hierarchy | `TreeModel` (direct) |
| Non-`Vector` combo data | `ComboBoxModel` (direct) |
| Just need a few convenience methods | Subclass the default |

#### Syntax Rules

- Prefer the abstract base class over direct interface implementation.
- Prefer direct implementation over subclassing the default when the default's data structure is wrong.
- Prefer subclassing the default when its data structure is close to what you need.

#### Constraints and Limitations

- Subclassing a default model couples your model to its internal storage.
- Direct interface implementation means you own listener management.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Three Implementations of the Same Data

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;

public class ThreeImplementations {
    // 1. Abstract base class: custom backing store
    static class AbstractModel extends AbstractTableModel {
        private final String[] data = {"A", "B", "C"};
        public int getRowCount() { return data.length; }
        public int getColumnCount() { return 1; }
        public Object getValueAt(int r, int c) { return data[r]; }
    }

    // 2. Subclass of DefaultTableModel: reuse its Vector
    static class SubclassModel extends javax.swing.table.DefaultTableModel {
        public SubclassModel() {
            super(new String[][]{{"A"}, {"B"}, {"C"}}, new String[]{"Item"});
        }
    }

    // 3. Direct interface implementation: full control
    static class DirectModel implements javax.swing.table.TableModel {
        private final String[] data = {"A", "B", "C"};
        private final java.util.List<javax.swing.event.TableModelListener> ls =
            new java.util.ArrayList<>();
        public int getRowCount() { return data.length; }
        public int getColumnCount() { return 1; }
        public Object getValueAt(int r, int c) { return data[r]; }
        public String getColumnName(int c) { return "Item"; }
        public Class<?> getColumnClass(int c) { return String.class; }
        public boolean isCellEditable(int r, int c) { return false; }
        public void setValueAt(Object v, int r, int c) {}
        public void addTableModelListener(javax.swing.event.TableModelListener l) { ls.add(l); }
        public void removeTableModelListener(javax.swing.event.TableModelListener l) { ls.remove(l); }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Three Implementations");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new java.awt.GridLayout(1, 3));
            frame.add(new JScrollPane(new JTable(new AbstractModel())));
            frame.add(new JScrollPane(new JTable(new SubclassModel())));
            frame.add(new JScrollPane(new JTable(new DirectModel())));
            frame.setSize(600, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three tables side by side, all showing A, B, C. The abstract-base-class table and the direct-interface table behave identically; the subclass table uses `DefaultTableModel`'s `Vector` storage.

**Why the code produces this result.** All three implementations satisfy the `TableModel` contract. The abstract base class provides the most balance of convenience and control; the direct implementation requires manual listener management; the subclass inherits the `Vector`-based storage.

---

### Real-World Cases

- **Domain-object tables** use `AbstractTableModel`.
- **Sortable tables** use `TableRowSorter` with `AbstractTableModel`.
- **Prototypes** use `DefaultTableModel`.
- **Large tables** use direct `TableModel` with virtual rows.

---

## 3. Custom ListModel Implementation

### Definitions

**Core Definition.** A custom `ListModel` is a class that implements `ListModel` (typically by extending `AbstractListModel`) and supplies the elements of a `JList` from a custom data source.

**Technical Definition.** `public abstract class AbstractListModel<E>` implements `addListDataListener`, `removeListDataListener`, and provides `fireContentsChanged`, `fireIntervalAdded`, and `fireIntervalRemoved`. Subclasses implement `getSize()` and `getElementAt(int)`. When the backing data changes, the subclass must call the appropriate fire method.

**Beginner-Friendly Explanation.** A custom `ListModel` is the "list data brain" you write when you want a `JList` to display your own data structure.

---

### Purposes

- To expose a custom data structure (`List`, `Set`, domain object collection) to a `JList`.
- To support large or virtual lists that cannot fit in memory.
- To enable read-only views of computed data.
- To share a list model across multiple `JList` instances or with a `JComboBox`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyListModel extends AbstractListModel<MyObject> {
    private final List<MyObject> data;

    public MyListModel(List<MyObject> data) { this.data = data; }

    public int getSize() { return data.size(); }
    public MyObject getElementAt(int index) { return data.get(index); }

    public void add(MyObject item) {
        int index = data.size();
        data.add(item);
        fireIntervalAdded(this, index, index);
    }

    public void remove(int index) {
        data.remove(index);
        fireIntervalRemoved(this, index, index);
    }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSize()` | Returns the number of elements. |
| `getElementAt(int)` | Returns the element at the index. |
| `fireIntervalAdded(src, int, int)` | Notifies listeners that elements were added. |
| `fireIntervalRemoved(src, int, int)` | Notifies listeners that elements were removed. |
| `fireContentsChanged(src, int, int)` | Notifies listeners that elements changed in place. |

#### Syntax Rules

- Always fire the appropriate event after mutating the backing data.
- Use `fireContentsChanged` for in-place edits; use `fireIntervalAdded` / `fireIntervalRemoved` for structural changes.
- Mutate on the EDT.

#### Constraints and Limitations

- The model must fire events for every mutation; otherwise the view stays stale.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom ListModel Backed by a List of Domain Objects

```java
import javax.swing.*;
import javax.swing.AbstractListModel;
import java.util.ArrayList;
import java.util.List;

public class DomainListModel extends AbstractListModel<Person> {
    public static class Person {
        String name; int age;
        Person(String n, int a) { name = n; age = a; }
        public String toString() { return name + " (" + age + ")"; }    // 1. Display
    }

    private final List<Person> people = new ArrayList<>();

    @Override public int getSize() { return people.size(); }            // 2. Size
    @Override public Person getElementAt(int i) { return people.get(i); } // 3. Access

    public void add(Person p) {                                          // 4. Add
        int i = people.size();
        people.add(p);
        fireIntervalAdded(this, i, i);                                   // 5. Fire
    }

    public void remove(int i) {                                          // 6. Remove
        people.remove(i);
        fireIntervalRemoved(this, i, i);                                 // 7. Fire
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DomainListModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DomainListModel model = new DomainListModel();
            model.add(new Person("Alice", 30));
            model.add(new Person("Bob", 25));

            JList<Person> list = new JList<>(model);

            JButton add = new JButton("Add");
            add.addActionListener(e ->
                model.add(new Person("New", 0)));

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(list), java.awt.BorderLayout.CENTER);
            panel.add(add, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(350, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A list showing “Alice (30)” and “Bob (25)”. Clicking “Add” appends “New (0)” and the list updates automatically.

**Why the code produces this result.** The model extends `AbstractListModel`, implements the two abstract methods, and fires `fireIntervalAdded` after each addition. The `JList` uses the model's `toString` of each `Person` for display.

---

### Real-World Cases

- **Contact lists** backed by a `List<Contact>`.
- **File lists** backed by `List<File>`.
- **Live feeds** that append new entries as events arrive.

---

## 4. Custom TableModel Implementation

### Definitions

**Core Definition.** A custom `TableModel` is a class that implements `TableModel` (typically by extending `AbstractTableModel`) and supplies row/column data for a `JTable` from a custom data source.

**Technical Definition.** `public abstract class AbstractTableModel` implements the listener registration methods and provides `fireTableDataChanged`, `fireTableRowsInserted`, `fireTableRowsUpdated`, `fireTableRowsDeleted`, `fireTableCellUpdated`, and `fireTableStructureChanged`. Subclasses implement `getRowCount()`, `getColumnCount()`, and `getValueAt(int, int)`. They may override `isCellEditable`, `setValueAt`, `getColumnName`, and `getColumnClass` for editing and rendering behaviour.

**Beginner-Friendly Explanation.** A custom `TableModel` is the "table data brain" you write when your data lives in domain objects, a database cursor, or any structure other than a `Vector` of `Vector`s.

---

### Purposes

- To expose a `List<DomainObject>` or a database `ResultSet` to a `JTable`.
- To support editable tables with domain-validating `setValueAt`.
- To enable virtual tables with computed cells.
- To control column names and types for correct rendering and editing.
- To integrate with `TableRowSorter` for sorting and filtering.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyTableModel extends AbstractTableModel {
    private final List<Row> rows = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return cols.length; }
    public String getColumnName(int c) { return cols[c]; }
    public Class<?> getColumnClass(int c) { return c == 0 ? String.class : Integer.class; }
    public Object getValueAt(int r, int c) { /* ... */ }
    public boolean isCellEditable(int r, int c) { return c == 1; }
    public void setValueAt(Object v, int r, int c) { /* ... fireTableCellUpdated */ }

    public void addRow(Row row) {
        int i = rows.size();
        rows.add(row);
        fireTableRowsInserted(i, i);
    }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getRowCount()` / `getColumnCount()` | Abstract; must be implemented. |
| `getValueAt(int, int)` | Abstract; returns the cell value. |
| `getColumnName(int)` | Default returns "A", "B", …; override for real names. |
| `getColumnClass(int)` | Default returns `Object.class`; override for renderer/editor selection. |
| `isCellEditable(int, int)` | Default returns `false`; override to enable editing. |
| `setValueAt(Object, int, int)` | Default no-op; override to implement editing. |
| `fireXxx` methods | Protected; call after mutations. |

#### Syntax Rules

- Implement the three abstract methods; override the optional ones as needed.
- Fire the narrowest appropriate event (`fireTableCellUpdated`, `fireTableRowsUpdated`, etc.).
- Implement `getColumnClass` to enable correct default renderers and editors.
- Always mutate on the EDT.

#### Constraints and Limitations

- `getValueAt` is called many times during painting; it must be fast.
- `fireTableStructureChanged` resets the column model; use it only for structural changes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom TableModel Backed by a List of Domain Objects

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class DomainTableModel extends AbstractTableModel {
    public static class Person {
        String name; int age;
        Person(String n, int a) { name = n; age = a; }
    }

    private final List<Person> people = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    @Override public int getRowCount() { return people.size(); }          // 1. Size
    @Override public int getColumnCount() { return cols.length; }         // 2. Cols
    @Override public String getColumnName(int c) { return cols[c]; }      // 3. Name
    @Override public Class<?> getColumnClass(int c) {                     // 4. Class
        return c == 0 ? String.class : Integer.class;
    }

    @Override
    public Object getValueAt(int r, int c) {                               // 5. Get
        Person p = people.get(r);
        return c == 0 ? p.name : p.age;
    }

    @Override public boolean isCellEditable(int r, int c) {               // 6. Editable
        return c == 1;
    }

    @Override
    public void setValueAt(Object v, int r, int c) {                       // 7. Set
        if (c == 1) {
            people.get(r).age = ((Number) v).intValue();
            fireTableCellUpdated(r, c);                                    // 8. Fire
        }
    }

    public void addPerson(Person p) {                                       // 9. Insert
        int i = people.size();
        people.add(p);
        fireTableRowsInserted(i, i);                                       // 10. Fire
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DomainTableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DomainTableModel model = new DomainTableModel();
            model.addPerson(new Person("Alice", 30));
            model.addPerson(new Person("Bob", 25));

            JTable table = new JTable(model);
            table.setAutoCreateRowSorter(true);

            JButton add = new JButton("Add Person");
            add.addActionListener(e ->
                model.addPerson(new Person("New", 0)));

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            panel.add(add, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A sortable table with “Name” and “Age” columns, showing Alice/30 and Bob/25. The Age column is editable; adding a person via the button appends a row.

**Why the code produces this result.** The model extends `AbstractTableModel`, implements `getValueAt` to translate domain objects into cell values, overrides `getColumnClass` for correct renderers, and fires the appropriate events after mutations.

---

### Real-World Cases

- **Database-backed tables** that cache `ResultSet` rows.
- **Sortable tables** with `TableRowSorter`.
- **Editable configuration tables** with domain-level validation in `setValueAt`.

---

## 5. Custom TreeModel Implementation

### Definitions

**Core Definition.** A custom `TreeModel` is a class that implements `TreeModel` (either directly or by extending `DefaultTreeModel`) and supplies a hierarchical data source to a `JTree`.

**Technical Definition.** `TreeModel` has **no abstract base class** in `javax.swing` (though `DefaultTreeModel` is a concrete implementation that can be subclassed). A custom `TreeModel` implements `getRoot`, `getChild`, `getChildCount`, `isLeaf`, `getIndexOfChild`, `valueForPathChanged`, `addTreeModelListener`, and `removeTreeModelListener`. If implementing directly, the class must manage its own listener list and fire `TreeModelEvent`s.

**Beginner-Friendly Explanation.** A custom `TreeModel` is the "tree data brain" you write when your data is a natural hierarchy (a file system, a database tree, a domain object graph) that would be awkward to wrap in `TreeNode` objects.

---

### Purposes

- To expose a natural hierarchy (e.g., `File`, domain objects) directly to a `JTree`.
- To support lazy-loading trees that query children on demand.
- To implement read-only virtual trees with computed children.
- To avoid the memory overhead of `DefaultMutableTreeNode` wrappers.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyTreeModel implements TreeModel {
    private final Node root;
    private final List<TreeModelListener> listeners = new ArrayList<>();

    public Object getRoot() { return root; }
    public Object getChild(Object p, int i) { /* ... */ }
    public int getChildCount(Object p) { /* ... */ }
    public boolean isLeaf(Object n) { /* ... */ }
    public int getIndexOfChild(Object p, Object c) { /* ... */ }
    public void valueForPathChanged(TreePath path, Object v) { /* ... */ }

    public void addTreeModelListener(TreeModelListener l) { listeners.add(l); }
    public void removeTreeModelListener(TreeModelListener l) { listeners.remove(l); }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getRoot()` | The root node. |
| `getChild(Object, int)` | The child at the index. |
| `getChildCount(Object)` | The number of children. |
| `isLeaf(Object)` | Whether the node is a leaf. |
| `getIndexOfChild(Object, Object)` | The index of a child, or `-1`. |
| `valueForPathChanged(TreePath, Object)` | Called when the user edits a node. |
| `addTreeModelListener` / `removeTreeModelListener` | Listener management. |

#### Syntax Rules

- Fire `TreeModelEvent`s (via `treeNodesChanged`, `treeNodesInserted`, `treeNodesRemoved`, `treeStructureChanged`) after mutations.
- Node objects must have consistent `equals` and `hashCode` for `TreePath` stability.
- Each child must be distinct by `equals` from its siblings.

#### Constraints and Limitations

- There is no `AbstractTreeModel`; you must implement the listener plumbing yourself (or extend `DefaultTreeModel`).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom TreeModel for a File System

```java
import javax.swing.*;
import javax.swing.tree.*;
import javax.swing.event.TreeModelListener;
import java.io.File;
import java.awt.*;

public class FileTreeModel implements TreeModel {
    private final File root;                                             // 1. Root
    private final java.util.List<TreeModelListener> listeners =
        new java.util.ArrayList<>();

    public FileTreeModel(File root) { this.root = root; }

    @Override public Object getRoot() { return root; }                   // 2. Root

    @Override
    public Object getChild(Object parent, int i) {                       // 3. Child
        File[] c = ((File) parent).listFiles();
        return (c != null) ? c[i] : null;
    }

    @Override
    public int getChildCount(Object parent) {                            // 4. Count
        File[] c = ((File) parent).listFiles();
        return (c != null) ? c.length : 0;
    }

    @Override public boolean isLeaf(Object node) {                       // 5. Leaf
        return ((File) node).isFile();
    }

    @Override
    public int getIndexOfChild(Object parent, Object child) {            // 6. Index
        File[] c = ((File) parent).listFiles();
        if (c == null) return -1;
        for (int i = 0; i < c.length; i++) if (c[i].equals(child)) return i;
        return -1;
    }

    @Override
    public void valueForPathChanged(TreePath path, Object newValue) {}   // 7. Edit

    @Override
    public void addTreeModelListener(TreeModelListener l) { listeners.add(l); } // 8. Listen
    @Override
    public void removeTreeModelListener(TreeModelListener l) { listeners.remove(l); }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("FileTreeModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JTree tree = new JTree(new FileTreeModel(new File(".")));
            frame.add(new JScrollPane(tree));
            frame.setSize(400, 350);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A tree showing the current directory and its subdirectories. Directories expand to show files and subdirectories; files are leaves.

**Why the code produces this result.** The custom `TreeModel` wraps `File` objects directly. `getChild` and `getChildCount` delegate to `File.listFiles()`. `isLeaf` returns `true` for files. `getIndexOfChild` searches the child array.

---

### Real-World Cases

- **File browsers** that wrap `File` objects.
- **Lazy-loading trees** that query children on expansion.
- **Domain-specific hierarchies** (organization charts, category trees).

---

## 6. Custom ComboBoxModel Implementation

### Definitions

**Core Definition.** A custom `ComboBoxModel` is a class that implements `ComboBoxModel` (or `MutableComboBoxModel`) and supplies both the item list and the selected item for a `JComboBox`.

**Technical Definition.** `ComboBoxModel<E>` extends `ListModel<E>` and adds `getSelectedItem()` and `setSelectedItem(Object)`. `MutableComboBoxModel<E>` adds `addElement`, `removeElement`, `insertElementAt`, and `removeElementAt`. There is no abstract base class for `ComboBoxModel`; implementations typically extend `AbstractListModel` and add the selection logic, or subclass `DefaultComboBoxModel`.

**Beginner-Friendly Explanation.** A custom `ComboBoxModel` is the "combo box data brain" you write when the item list and selected item come from a custom data source.

---

### Purposes

- To expose a custom list of items to a `JComboBox`.
- To control how the selected item is represented and validated.
- To support lazy or virtual item lists.
- To share the underlying data between a combo box and a list.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyComboBoxModel extends AbstractListModel<E>
        implements ComboBoxModel<E>, MutableComboBoxModel<E> {
    private final List<E> items = new ArrayList<>();
    private Object selected;

    public int getSize() { return items.size(); }
    public E getElementAt(int i) { return items.get(i); }
    public Object getSelectedItem() { return selected; }
    public void setSelectedItem(Object o) { selected = o; fireContentsChanged(this, -1, -1); }

    public void addElement(E e) { int i = items.size(); items.add(e); fireIntervalAdded(this, i, i); }
    public void removeElement(Object o) { int i = items.indexOf(o); if (i >= 0) removeElementAt(i); }
    public void insertElementAt(E e, int i) { items.add(i, e); fireIntervalAdded(this, i, i); }
    public void removeElementAt(int i) { items.remove(i); fireIntervalRemoved(this, i, i); }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSize` / `getElementAt` | Inherited from `ListModel`. |
| `getSelectedItem` / `setSelectedItem` | Track and change the selection. |
| `addElement` / `insertElementAt` | Add items to the list. |
| `removeElement` / `removeElementAt` | Remove items. |
| `fireContentsChanged(this, -1, -1)` | Standard way to signal a selection change. |

#### Syntax Rules

- Always fire an event when the selected item or the item list changes.
- `fireContentsChanged(this, -1, -1)` is the conventional signal for a selection change.
- Mutable models must implement the full `MutableComboBoxModel` interface.

#### Constraints and Limitations

- If the selected item is not in the list, `getSelectedItem` may still return it (a valid use case for editable combo boxes).
- The `items` list may be shared with other views.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom ComboBoxModel with Domain Objects

```java
import javax.swing.*;
import java.util.ArrayList;
import java.util.List;

public class DomainComboBoxModel extends AbstractListModel<Person>
        implements MutableComboBoxModel<Person> {
    public static class Person {
        String name; int age;
        Person(String n, int a) { name = n; age = a; }
        public String toString() { return name; }
    }

    private final List<Person> people = new ArrayList<>();
    private Object selected;

    @Override public int getSize() { return people.size(); }
    @Override public Person getElementAt(int i) { return people.get(i); }
    @Override public Object getSelectedItem() { return selected; }

    @Override
    public void setSelectedItem(Object o) {                          // 1. Select
        selected = o;
        fireContentsChanged(this, -1, -1);                            // 2. Fire
    }

    @Override
    public void addElement(Person p) {                                 // 3. Add
        int i = people.size();
        people.add(p);
        fireIntervalAdded(this, i, i);
    }
    @Override public void removeElement(Object o) { int i = people.indexOf(o); if (i >= 0) removeElementAt(i); }
    @Override public void insertElementAt(Person p, int i) { people.add(i, p); fireIntervalAdded(this, i, i); }
    @Override public void removeElementAt(int i) { people.remove(i); fireIntervalRemoved(this, i, i); }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DomainComboBoxModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DomainComboBoxModel model = new DomainComboBoxModel();
            model.addElement(new Person("Alice", 30));
            model.addElement(new Person("Bob", 25));

            JComboBox<Person> combo = new JComboBox<>(model);
            JLabel status = new JLabel("Selected: none");

            combo.addActionListener(e ->
                status.setText("Selected: " + combo.getSelectedItem()));

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(combo, java.awt.BorderLayout.NORTH);
            panel.add(status, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A combo box containing “Alice” and “Bob”. Selecting a person updates the status label.

**Why the code produces this result.** The model extends `AbstractListModel` and implements `MutableComboBoxModel`. The `setSelectedItem` method fires a `fireContentsChanged(this, -1, -1)` event, which the `JComboBox` observes.

---

### Real-World Cases

- **Domain-object combo boxes** (people, categories, products).
- **Lazy or virtual combo boxes** that fetch items on demand.
- **Shared data views** (a combo box and a list showing the same items).

---

## 7. Custom SpinnerModel Implementation

### Definitions

**Core Definition.** A custom `SpinnerModel` is a class that implements `SpinnerModel` (typically by extending `AbstractSpinnerModel`) and supplies a sequence of values to a `JSpinner`.

**Technical Definition.** `AbstractSpinnerModel` implements the change-listener registration methods and provides `fireStateChanged()`. Subclasses implement `getValue()`, `setValue(Object)`, `getNextValue()`, and `getPreviousValue()`. A custom `SpinnerModel` may return `null` from `getNextValue` or `getPreviousValue` to indicate the sequence's boundaries.

**Beginner-Friendly Explanation.** A custom `SpinnerModel` is the "spinner sequence brain" you write when the sequence is neither numeric, fixed-list, nor date-based (e.g., a sequence of enums, domain objects, or computed values).

---

### Purposes

- To provide a spinner sequence of enums or domain objects.
- To implement non-linear sequences (logarithmic scales, custom increments).
- To load sequence values on demand.
- To validate values in `setValue` with domain-specific rules.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MySpinnerModel extends AbstractSpinnerModel {
    private Object value;

    public Object getValue() { return value; }
    public void setValue(Object v) {
        if (!isValid(v)) throw new IllegalArgumentException("Invalid");
        value = v;
        fireStateChanged();
    }
    public Object getNextValue() { /* compute next */ }
    public Object getPreviousValue() { /* compute previous */ }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getValue` | The current value. |
| `setValue` | Validates and sets the value; fires `fireStateChanged`. |
| `getNextValue` / `getPreviousValue` | Compute the next/previous values, or `null` at boundaries. |
| `fireStateChanged` | Inherited from `AbstractSpinnerModel`. |

#### Syntax Rules

- `setValue` must validate the value and throw `IllegalArgumentException` for invalid input.
- `fireStateChanged` must be called when the value changes.
- `getNextValue` and `getPreviousValue` must not mutate the current value.

#### Constraints and Limitations

- The spinner's editor must be able to parse the model's values (custom editors may be required for non-`String`/`Number` types).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom SpinnerModel for a Day-of-Week Sequence

```java
import javax.swing.*;
import java.util.Arrays;
import java.util.List;

public class DaySpinnerModel extends AbstractSpinnerModel {
    private static final List<String> DAYS = Arrays.asList(
        "Monday", "Tuesday", "Wednesday", "Thursday",
        "Friday", "Saturday", "Sunday");
    private int index = 0;

    @Override public Object getValue() { return DAYS.get(index); }
    @Override
    public void setValue(Object v) {                                  // 1. Validate
        if (!DAYS.contains(v)) throw new IllegalArgumentException("Not a day: " + v);
        index = DAYS.indexOf(v);
        fireStateChanged();                                            // 2. Fire
    }
    @Override public Object getNextValue() {                          // 3. Next
        return index < DAYS.size() - 1 ? DAYS.get(index + 1) : null;
    }
    @Override public Object getPreviousValue() {                      // 4. Previous
        return index > 0 ? DAYS.get(index - 1) : null;
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DaySpinnerModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JSpinner spinner = new JSpinner(new DaySpinnerModel());
            spinner.addChangeListener(e -> System.out.println("Day: " + spinner.getValue()));
            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner shows “Monday” initially. The up arrow cycles through Tuesday … Sunday; the down arrow reverses. Each change prints the current day.

**Why the code produces this result.** The custom model implements all four abstract methods and fires `fireStateChanged` in `setValue`. `getNextValue` and `getPreviousValue` compute the neighbouring days and return `null` at the boundaries.

---

### Real-World Cases

- **Enum spinners** (priority, status, category).
- **Custom ranges** (logarithmic scales, non-linear steps).
- **Domain-specific sequences** (months, quarters).

---

## 8. Custom BoundedRangeModel Implementation

### Definitions

**Core Definition.** A custom `BoundedRangeModel` is a class that implements `BoundedRangeModel` and supplies the range state (`minimum`, `maximum`, `value`, `extent`) to a `JSlider`, `JProgressBar`, or `JScrollBar`. In practice, `DefaultBoundedRangeModel` is almost always sufficient, and custom implementations are rare.

**Technical Definition.** `BoundedRangeModel` has no abstract base class. A custom implementation must implement all four property getters and setters, `getValueIsAdjusting`/`setValueIsAdjusting`, `setRangeProperties`, and the change-listener methods. The invariant `minimum ≤ value ≤ value + extent ≤ maximum` must be enforced.

**Beginner-Friendly Explanation.** A custom `BoundedRangeModel` is rarely needed; `DefaultBoundedRangeModel` covers nearly every use case. But if you need domain-specific constraints (e.g., a value that must remain a multiple of 5), you can subclass `DefaultBoundedRangeModel` and override the setters.

---

### Purposes

- To enforce domain-specific constraints on a slider's or progress bar's value.
- To support non-standard range logic (e.g., non-linear mapping).
- To intercept changes for validation or side effects.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class SnapModel extends DefaultBoundedRangeModel {
    @Override
    public void setValue(int newValue) {
        int snapped = (newValue / 5) * 5;                              // 1. Snap
        super.setValue(snapped);                                       // 2. Delegate
    }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setValue(int)` | Override to transform or validate the value. |
| `setExtent(int)` | Override to constrain the extent. |
| `setMinimum(int)` / `setMaximum(int)` | Override to adjust the range. |
| `super.setValue(...)` | Delegate to the standard implementation after transformation. |

#### Syntax Rules

- Subclass `DefaultBoundedRangeModel` rather than implementing `BoundedRangeModel` from scratch.
- Call `super` methods to preserve the invariant and event firing.

#### Constraints and Limitations

- The invariant must be maintained; transformation must produce a value in range.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Snap-to-Multiple-of-5 Model

```java
import javax.swing.*;
import java.awt.*;

public class SnapModel extends DefaultBoundedRangeModel {
    public SnapModel(int value, int extent, int min, int max) {
        super(value, extent, min, max);
    }

    @Override
    public void setValue(int newValue) {                              // 1. Override
        int snapped = ((newValue + 2) / 5) * 5;                       // 2. Snap
        super.setValue(snapped);                                      // 3. Delegate
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Snap Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            SnapModel model = new SnapModel(50, 0, 0, 100);
            JSlider slider = new JSlider(model);
            slider.setPaintTicks(true);
            slider.setMajorTickSpacing(25);
            slider.setMinorTickSpacing(5);

            slider.addChangeListener(e ->
                System.out.println("Value: " + model.getValue()));

            frame.add(slider);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A slider from 0 to 100 that always snaps to multiples of 5. Dragging the knob prints the snapped value.

**Why the code produces this result.** The `setValue` override snaps the requested value to the nearest multiple of 5 before delegating to the standard implementation. The invariant and event firing are preserved by `super`.

---

### Real-World Cases

- **Discrete-value sliders** (e.g., a slider that picks only 0, 25, 50, 75, 100).
- **Domain-constrained progress bars** (e.g., a progress bar that only advances in increments of 10%).

---

## 9. Custom Document Implementation

### Definitions

**Core Definition.** A custom `Document` is a class that extends `PlainDocument` or `AbstractDocument` (or implements `Document` directly) and adds specialised structure, filtering, or behaviour to a text component.

**Technical Definition.** `AbstractDocument` is the standard base class for custom documents. Subclasses can override `insertUpdate` and `removeUpdate` to control event firing, install a `DocumentFilter` for input transformation, or provide a custom `Element` structure. For most use cases, extending `PlainDocument` and installing a `DocumentFilter` in the constructor is sufficient.

**Beginner-Friendly Explanation.** A custom `Document` is the "text data brain" you write when you need to control how text is stored, filtered, or structured.

---

### Purposes

- To enforce input validation or transformation via `DocumentFilter`.
- To provide a custom `Element` hierarchy for specialised rendering.
- To support undo/redo (via `UndoableEditListener`, not the document itself).
- To log or monitor changes for auditing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyDocument extends PlainDocument {
    public MyDocument() {
        setDocumentFilter(new DocumentFilter() {
            @Override
            public void insertString(FilterBypass fb, int off, String s, AttributeSet a)
                    throws BadLocationException {
                super.insertString(fb, off, transform(s), a);
            }
        });
    }
    private String transform(String s) { return s.toUpperCase(); }
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setDocumentFilter(DocumentFilter)` | Installs the filter. |
| `FilterBypass` | The mechanism for passing the (possibly transformed) operation to the document. |
| `insertString` / `replace` / `remove` | Filter methods that intercept mutations. |

#### Syntax Rules

- Install the filter in the document's constructor or via `setDocumentFilter`.
- Call `super` methods with the transformed data to apply the change.
- Never mutate the document from a `DocumentListener`.

#### Constraints and Limitations

- `DocumentFilter` can only intercept `insertString`, `replace`, and `remove`; other document methods bypass it.
- Overriding `insertUpdate`/`removeUpdate` requires calling `super` to preserve event firing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Document with Uppercase Filter

```java
import javax.swing.*;
import javax.swing.text.*;

public class UppercaseDocument extends PlainDocument {                // 1. Extend
    public UppercaseDocument() {
        setDocumentFilter(new DocumentFilter() {                      // 2. Filter
            @Override
            public void insertString(FilterBypass fb, int offset,
                String text, AttributeSet attrs) throws BadLocationException {
                super.insertString(fb, offset, text.toUpperCase(), attrs);
            }
            @Override
            public void replace(FilterBypass fb, int offset, int length,
                String text, AttributeSet attrs) throws BadLocationException {
                super.replace(fb, offset, length, text.toUpperCase(), attrs);
            }
        });
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("UppercaseDocument");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JTextField field = new JTextField(new UppercaseDocument(), "", 20);
            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing lowercase letters results in uppercase characters in the field.

**Why the code produces this result.** The `DocumentFilter` intercepts insertions and replacements, uppercasing the text before passing it to `super`. The `PlainDocument` subclass installs the filter in its constructor.

---

### Real-World Cases

- **Input masking** (phone numbers, dates, currency).
- **Uppercase fields** for codes or identifiers.
- **Numeric-only fields** with custom validation.

---

## 10. Model Implementation — Event Firing Discipline

### Definitions

**Core Definition.** Event-firing discipline is the practice of calling the correct `fireXxx` method after every mutation of a custom model, so that the view and other listeners stay synchronized.

**Technical Definition.** Each model interface provides its own event methods: `fireIntervalAdded`/`fireIntervalRemoved`/`fireContentsChanged` for `ListModel`, `fireTableRowsInserted`/`fireTableRowsUpdated`/`fireTableRowsDeleted`/`fireTableCellUpdated`/`fireTableDataChanged`/`fireTableStructureChanged` for `TableModel`, `treeNodesInserted`/`treeNodesRemoved`/`treeNodesChanged`/`treeStructureChanged` for `TreeModel`, `fireStateChanged` for `SpinnerModel`, and `fireInsertUpdate`/`fireRemoveUpdate`/`fireChangedUpdate` for `Document`. The narrowest applicable event should be fired to avoid unnecessary repaints.

**Beginner-Friendly Explanation.** Event firing is the "telling the view" part of a custom model. After every change, you call the appropriate `fireXxx` method so the view knows to repaint.

---

### Purposes

- To keep views synchronized with the model.
- To avoid unnecessary repaints by firing the narrowest appropriate event.
- To enable dependent listeners to react to changes.
- To support shared models across multiple views.

---

### Syntax Rules and Structure

#### Complete General Syntax (Event Firing by Model)

| Model | Add | Remove | Update | Structure |
|-------|-----|--------|--------|-----------|
| `ListModel` | `fireIntervalAdded` | `fireIntervalRemoved` | `fireContentsChanged` | — |
| `TableModel` | `fireTableRowsInserted` | `fireTableRowsDeleted` | `fireTableRowsUpdated` / `fireTableCellUpdated` | `fireTableStructureChanged` |
| `TreeModel` | `treeNodesInserted` | `treeNodesRemoved` | `treeNodesChanged` | `treeStructureChanged` |
| `SpinnerModel` | — | — | `fireStateChanged` | — |
| `Document` | `fireInsertUpdate` | `fireRemoveUpdate` | `fireChangedUpdate` | — |

#### Component Breakdown

| Event | Meaning |
|-------|---------|
| `fireIntervalAdded` | List elements were inserted. |
| `fireIntervalRemoved` | List elements were removed. |
| `fireContentsChanged` | List elements changed in place. |
| `fireTableCellUpdated` | A single table cell changed. |
| `fireTableRowsInserted` | Table rows were inserted. |
| `fireTableStructureChanged` | Table columns changed. |

#### Syntax Rules

- Fire **after** the data has been mutated.
- Fire the **narrowest** appropriate event.
- Use `fireTableStructureChanged` only when the column structure changed.

#### Constraints and Limitations

- Firing too broad an event (e.g., `fireTableDataChanged` for a single cell) is correct but inefficient.
- Firing the wrong event (e.g., `fireTableCellUpdated` for an insertion) causes the view to miss the change.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct Event Firing for a Custom TableModel

```java
public class EventFiringDemo extends AbstractTableModel {
    private final java.util.List<String[]> rows = new java.util.ArrayList<>();

    @Override public int getRowCount() { return rows.size(); }
    @Override public int getColumnCount() { return 2; }
    @Override public Object getValueAt(int r, int c) { return rows.get(r)[c]; }

    public void addRow(String a, String b) {                          // 1. Add
        int i = rows.size();
        rows.add(new String[]{a, b});
        fireTableRowsInserted(i, i);                                  // 2. Fire
    }

    public void updateCell(int r, int c, String v) {                  // 3. Update
        rows.get(r)[c] = v;
        fireTableCellUpdated(r, c);                                   // 4. Fire
    }

    public void removeRow(int r) {                                    // 5. Remove
        rows.remove(r);
        fireTableRowsDeleted(r, r);                                   // 6. Fire
    }
}
```

**Expected Output.** Each mutation fires the correct event, so the `JTable` repaints only the affected rows/cells.

**Why the code produces this result.** `fireTableRowsInserted`, `fireTableCellUpdated`, and `fireTableRowsDeleted` each notify the `JTable` of the specific change, allowing it to repaint only the affected range.

---

### Real-World Cases

- **Performance-critical models** fire the narrowest events to minimize repaints.
- **Shared models** fire events to notify all views.
- **Testing** verifies that events are fired correctly.

---

## 11. Model Implementation — Thread Safety

### Definitions

**Core Definition.** Model thread safety is the discipline of mutating custom models on the EDT and ensuring that listeners registered on models mutated off the EDT are themselves thread-safe.

**Technical Definition.** Swing components, their models, and their listeners are not thread-safe. Mutations to a model attached to a live component must occur on the EDT. If a custom model is mutated off the EDT (e.g., from a background thread), it must notify its listeners on the mutating thread, and those listeners must be thread-safe. In practice, the safest approach is to mutate models via `SwingUtilities.invokeLater` or `SwingWorker`.

**Beginner-Friendly Explanation.** Thread safety means mutating the model on the right thread. If the model is attached to a live component, mutate on the EDT.

---

### Purposes

- To prevent race conditions, `ConcurrentModificationException`, and display corruption.
- To comply with Swing's threading policy.
- To keep the GUI responsive by moving heavy computation off the EDT.

---

### Syntax Rules and Structure

#### Complete General Syntax (Safe Mutation from a Background Thread)

```java
new Thread(() -> {
    // heavy computation off the EDT
    List<Row> newRows = loadRows();
    SwingUtilities.invokeLater(() -> {                               // 1. Post
        for (Row r : newRows) {
            model.addRow(r);                                          // 2. Mutate
        }
    });
}).start();
```

#### Component Breakdown

| Pattern | Meaning |
|---------|---------|
| `SwingUtilities.invokeLater(...)` | Schedule the mutation on the EDT. |
| `SwingWorker` | Perform heavy work off the EDT and post results. |
| `SwingUtilities.isEventDispatchThread()` | Check whether the current thread is the EDT. |

#### Syntax Rules

- Mutate the model only on the EDT.
- For large datasets, use `SwingWorker` and post chunks.
- Never block the EDT inside a `fireXxx` listener.

#### Constraints and Limitations

- `AbstractDocument` supports off-EDT mutation but notifies listeners on the mutating thread; listeners must be thread-safe.
- Some models may be safely mutated off the EDT if no live view is attached, but this is fragile.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Background Loading into a Custom TableModel

```java
import javax.swing.*;
import java.util.ArrayList;
import java.util.List;

public class BackgroundLoadDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Background Load");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            javax.swing.table.DefaultTableModel model =
                new javax.swing.table.DefaultTableModel(
                    new Object[]{"Name"}, 0);
            JTable table = new JTable(model);

            new SwingWorker<List<String>, Void>() {                    // 1. Worker
                @Override
                protected List<String> doInBackground() {
                    List<String> data = new ArrayList<>();
                    for (int i = 0; i < 100; i++) {
                        try { Thread.sleep(10); } catch (InterruptedException e) {}
                        data.add("Item " + i);
                    }
                    return data;
                }
                @Override
                protected void done() {
                    try {
                        List<String> data = get();
                        for (String s : data) {
                            model.addRow(new Object[]{s});              // 2. EDT
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }.execute();

            frame.add(new JScrollPane(table));
            frame.setSize(300, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The table starts empty and populates with 100 items after a brief delay, without freezing the UI.

**Why the code produces this result.** `SwingWorker.doInBackground` runs off the EDT and returns the loaded data; `done()` runs on the EDT and adds the rows to the model. This preserves both responsiveness and thread safety.

---

### Real-World Cases

- **Database loading** into custom `TableModel`s via `SwingWorker`.
- **File system scans** populating custom `TreeModel`s lazily.
- **Network feeds** appending items to custom `ListModel`s.

---

## 12. Custom Models Best Practices

### Definitions

**Core Definition.** Custom model best practices are the conventions and patterns that ensure custom models are correct, thread-safe, performant, and maintainable.

**Technical Definition.** Best practices include: extending the appropriate abstract base class; implementing only the required methods; firing the correct event after every mutation; mutating on the EDT; implementing `getColumnClass` and `isCellEditable` for table models; using `fireContentsChanged(this, -1, -1)` for `ComboBoxModel` selection changes; using `getValueIsAdjusting` guards for `BoundedRangeModel`-driven components; using `DocumentFilter` for document input transformation; and unit-testing the model without a GUI.

**Beginner-Friendly Explanation.** Best practices for custom models are the "rules of thumb" that prevent stale views, frozen GUIs, and unmaintainable code. They include things like "extend `AbstractXxxModel`," "fire the right event," "mutate on the EDT," and "test the model headlessly."

---

### Purposes

- To minimize boilerplate by choosing the right base class.
- To keep views synchronized by firing the correct events.
- To maintain thread safety by respecting the EDT.
- To enable correct rendering and editing through `getColumnClass` and `isCellEditable`.
- To make models testable without a GUI.
- To support large or on-demand data.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class BestPracticeModel extends AbstractTableModel {           // 1. Abstract
    private final List<Row> rows = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return cols.length; }
    public String getColumnName(int c) { return cols[c]; }
    public Class<?> getColumnClass(int c) {                           // 2. Class
        return c == 0 ? String.class : Integer.class;
    }
    public Object getValueAt(int r, int c) { /* fast lookup */ }      // 3. Fast
    public boolean isCellEditable(int r, int c) { return c == 1; }    // 4. Editable
    public void setValueAt(Object v, int r, int c) {
        rows.get(r).age = ((Number) v).intValue();
        fireTableCellUpdated(r, c);                                   // 5. Fire
    }
    public void add(Row row) {
        int i = rows.size();
        rows.add(row);
        fireTableRowsInserted(i, i);                                  // 6. Fire
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend abstract base | `AbstractListModel`, `AbstractTableModel`, `AbstractSpinnerModel`, `AbstractDocument` |
| Fire correct event | `fireIntervalAdded`, `fireTableRowsInserted`, `fireStateChanged`, etc. |
| Mutate on EDT | `SwingUtilities.invokeLater(...)` for mutations from non-EDT threads |
| Implement `getColumnClass` | Return specific class for renderer/editor |
| Implement `isCellEditable` | Control editing |
| Fast `getValueAt` | Cache data; no I/O or expensive computation |
| Use `DocumentFilter` | For document input transformation |
| Headless testing | Test model logic without a GUI |
| `SwingWorker` for large loads | Load in background, add on EDT |

#### Syntax Rules

- **Always extend the abstract base class** when one is available.
- **Always fire the correct event** after every mutation.
- **Always mutate on the EDT** when the model is attached to a live component.
- **Always implement `getColumnClass`** in `TableModel`s for correct rendering and editing.
- **Always use `DocumentFilter`** (not `DocumentListener`) for document input transformation.
- **Always keep `getValueAt` fast**; cache or precompute expensive values.
- **Always test custom models headlessly** before wiring them to a live GUI.

#### Constraints and Limitations

- `AbstractTreeModel` does not exist; implement `TreeModel` directly or subclass `DefaultTreeModel`.
- Firing too broad an event (e.g., `fireTableDataChanged` for a single cell) is correct but inefficient.
- Off-EDT mutations can cause race conditions and display corruption.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom TableModel with Background Loading

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class BestPracticeCustomModel extends AbstractTableModel {
    public static class Row {
        String name; int age;
        Row(String n, int a) { name = n; age = a; }
    }

    private final List<Row> rows = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    @Override public int getRowCount() { return rows.size(); }
    @Override public int getColumnCount() { return cols.length; }
    @Override public String getColumnName(int c) { return cols[c]; }
    @Override public Class<?> getColumnClass(int c) {
        return c == 0 ? String.class : Integer.class;
    }
    @Override public Object getValueAt(int r, int c) {
        Row row = rows.get(r);
        return c == 0 ? row.name : row.age;
    }
    @Override public boolean isCellEditable(int r, int c) { return c == 1; }
    @Override public void setValueAt(Object v, int r, int c) {
        if (c == 1) {
            rows.get(r).age = ((Number) v).intValue();
            fireTableCellUpdated(r, c);
        }
    }

    public void add(Row row) {
        int i = rows.size();
        rows.add(row);
        fireTableRowsInserted(i, i);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeCustomModel::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Custom Model");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        BestPracticeCustomModel model = new BestPracticeCustomModel();
        JTable table = new JTable(model);
        table.setAutoCreateRowSorter(true);

        new SwingWorker<List<Row>, Void>() {
            @Override
            protected List<Row> doInBackground() {
                List<Row> result = new ArrayList<>();
                for (int i = 0; i < 50; i++) {
                    try { Thread.sleep(10); } catch (InterruptedException e) {}
                    result.add(new Row("User " + i, 20 + i));
                }
                return result;
            }
            @Override
            protected void done() {
                try {
                    for (Row r : get()) model.add(r);
                } catch (Exception e) { e.printStackTrace(); }
            }
        }.execute();

        frame.add(new JScrollPane(table));
        frame.setSize(450, 300);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A sortable table that populates with 50 rows asynchronously. The “Name” column is read-only; the “Age” column is editable; sorting works via `TableRowSorter`.

**Why the code produces this result.** The model extends `AbstractTableModel`, implements `getColumnClass` and `isCellEditable`, fires the correct events after mutations, and is populated from a `SwingWorker` on the EDT. All best practices are applied.

---

#### Example 2: Common Mistake — Firing Too Broad an Event

```java
public class BadEventModel extends AbstractTableModel {
    private final List<String> rows = new ArrayList<>();

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return 1; }
    public Object getValueAt(int r, int c) { return rows.get(r); }

    public void updateRow(int r, String s) {
        rows.set(r, s);
        fireTableDataChanged();                                       // WRONG
    }
}
```

**Expected Output.** The table repaints entirely after each single-cell update, which is inefficient for large tables.

**Why this is wrong.** `fireTableDataChanged()` is correct but inefficient. The correct event for a single-row update is `fireTableRowsUpdated(r, r)` or `fireTableCellUpdated(r, c)`. Firing the narrowest event keeps performance optimal.

---

#### Example 3: Common Mistake — Mutating Off the EDT

```java
new Thread(() -> model.add(new Row("X", 0))).start();             // WRONG
```

**Expected Output.** The model may produce inconsistent state or corrupt the display because Swing models are not thread-safe.

**Why this is wrong.** Mutations must occur on the EDT. The correct approach is `SwingUtilities.invokeLater(() -> model.add(new Row("X", 0)))`.

---

### Real-World Cases

- **Production Swing applications** use custom models extending abstract base classes, with correct event firing and EDT discipline.
- **Database-backed tables** use `SwingWorker` to load data and fire events on the EDT.
- **Large or virtual data** uses custom models with fast `getValueAt` and lazy loading.
- **Shared views** share models to keep multiple components synchronized.
- **Educational examples** use the best-practice skeleton to demonstrate correct custom model design from the start.

---

## References

Oracle. *A Swing Architecture Overview*. The Java Tutorials. https://www.oracle.com/technetwork/java/architecture-142923.html

Oracle. *Class AbstractListModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractListModel.html

Oracle. *Class AbstractTableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/AbstractTableModel.html

Oracle. *Class AbstractSpinnerModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractSpinnerModel.html

Oracle. *Class AbstractDocument*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/AbstractDocument.html

Oracle. *Class DefaultTreeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/DefaultTreeModel.html

Oracle. *Class DefaultComboBoxModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/DefaultComboBoxModel.html

Oracle. *Class DefaultBoundedRangeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/DefaultBoundedRangeModel.html

Oracle. *Interface ListModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ListModel.html

Oracle. *Interface TableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/TableModel.html

Oracle. *Interface TreeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/TreeModel.html

Oracle. *Interface ComboBoxModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ComboBoxModel.html

Oracle. *Interface SpinnerModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SpinnerModel.html

Oracle. *Interface BoundedRangeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/BoundedRangeModel.html

Oracle. *Interface Document*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/Document.html

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *Implementing a Document Filter*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/generaltext.html#filter

Stack Overflow. *AbstractTableModel vs DefaultTableModel*. https://stackoverflow.com/questions/2454413/jtable-abstracttablemodel-vs-defaulttablemodel

Stack Overflow. *Custom TreeModel with lazy loading*. https://stackoverflow.com/questions/

Cleverence. *JTable in Java SE 8: Complete Guide to Models, Rendering, Sorting & Editing*. https://www.cleverence.com/articles/oracle-documentation/jtable-java-platform-se-8-4827/

Cleverence. *Java TreeNode (SE 8) Explained: JTree, TreeModel, and Practical Examples*. https://www.cleverence.com/articles/oracle-documentation/treenode-java-platform-se-8-4827/