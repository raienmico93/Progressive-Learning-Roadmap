# TableModel, AbstractTableModel & DefaultTableModel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `TableModel` is the **data model interface** behind every `JTable`. It defines the minimal contract that any tabular data source must fulfill: report the number of rows and columns, return the value at any cell, optionally allow editing, and notify registered listeners when data changes. `AbstractTableModel` is the **abstract base class** that implements the listener bookkeeping while leaving the three data-access methods to the subclass. `DefaultTableModel` is the **concrete, mutable implementation** that stores cell values in a `Vector` of `Vector`s of `Object`s and provides convenience methods for adding, removing, and modifying rows and columns.

**Technical Definition.** `public interface TableModel` declares seven methods: `int getRowCount()`, `int getColumnCount()`, `Object getValueAt(int rowIndex, int columnIndex)`, `void setValueAt(Object aValue, int rowIndex, int columnIndex)`, `String getColumnName(int columnIndex)`, `Class<?> getColumnClass(int columnIndex)`, `boolean isCellEditable(int rowIndex, int columnIndex)`, plus `void addTableModelListener(TableModelListener l)` and `void removeTableModelListener(TableModelListener l)`. A correct `TableModel` **must** notify registered `TableModelListener`s whenever the data changes, using `TableModelEvent` (with `INSERT`, `UPDATE`, or `DELETE` type, plus a row range and column index). `public abstract class AbstractTableModel extends Object implements TableModel, Serializable` implements listener registration and provides protected `fireXxx` methods (`fireTableDataChanged`, `fireTableRowsInserted`, `fireTableRowsUpdated`, `fireTableRowsDeleted`, `fireTableCellUpdated`, `fireTableStructureChanged`) but contains **no specific data structure internally**. `public class DefaultTableModel extends AbstractTableModel implements Serializable` implements a `Vector`-based mutable model with methods like `addRow`, `insertRow`, `removeRow`, `addColumn`, `setValueAt`, and `setDataVector`. All model mutations should occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** A `TableModel` is the **"data brain"** of a `JTable`. The visible table is the **face** — it paints rows and columns, handles scrolling, sorting, and cell editing. The model holds the actual data. Swing offers **three levels of model implementation**, each giving you progressively more control:

- **`DefaultTableModel`** — "everything is taken care of for you". Use it for simple mutable tables.
- **`AbstractTableModel`** — "you manage the data and invoke the fire methods". Use it when your data lives in a custom structure (a `List<Person>`, a database cursor, a domain object graph).
- **`TableModel`** — "you manage everything". Use it only when you need complete control over listener management.

Think of it as a spreadsheet: `DefaultTableModel` is a ready-made spreadsheet app; `AbstractTableModel` is a spreadsheet framework where you supply the data source but the calculation engine is built in; `TableModel` is raw cells and formulas — you build everything.

---

### Key Characteristics

1. **Separable model architecture.** `JTable` and `TableModel` are separate objects, following Swing's MVC-inspired design. Multiple `JTable` instances can share the same model. The `JTable` **automatically registers itself** as a `TableModelListener` on its model to keep its display synchronized.
2. **Event-driven synchronization.** A correct model notifies `TableModelListener`s whenever its data changes. The `TableModelEvent` carries the change type, row range, and column index.
3. **Three-tier implementation.** `TableModel` (interface) → `AbstractTableModel` (listener plumbing, no data) → `DefaultTableModel` (concrete `Vector`-backed implementation).
4. **`TableModelEvent` identifies changes.** `INSERT` (`1`), `UPDATE` (`0`), and `DELETE` (`-1`) types; `ALL_COLUMNS` (`-1`) indicates all columns affected; `HEADER_ROW` (`-1`) indicates the header row changed.
5. **Default implementations for common methods.** `AbstractTableModel` provides default implementations for `getColumnName` (returns “A”, “B”, “C”…), `getColumnClass` (returns `Object.class`), `isCellEditable` (returns `false`), and `setValueAt` (no-op).
6. **Cell rendering and editing depend on the model.** `getColumnClass` informs the default renderer and editor; `isCellEditable` controls whether cells can be edited.
7. **Sorting via `RowSorter`.** The recommended way to sort a `JTable` is to attach a `RowSorter` (e.g., `TableRowSorter`) rather than sorting the model itself; `setAutoCreateRowSorter(true)` is the shortcut.
8. **EDT discipline.** Model mutations should occur on the EDT. The `JTable`'s internal listener processes changes on the EDT; application listeners should also respond on the EDT.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, generics, anonymous inner classes, and lambda expressions.
- Familiarity with `JTable`, `JScrollPane`, and basic Swing containers.
- Understanding of `TableModelListener`, `TableModelEvent`, and the delegation event model.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `List`, `Vector`, or arrays (the backing stores for typical implementations).
- A JDK (version 8 or later; `TableModel`, `AbstractTableModel`, and `DefaultTableModel` are in `javax.swing.table`).

---

### Related Programming Areas

- **`JTable`:** The view component that consumes `TableModel`.
- **`TableModelListener` and `TableModelEvent`:** The event contract that models use to notify views.
- **`TableColumnModel`:** The separate model that manages columns (width, order, header renderers).
- **`ListSelectionModel`:** The separate model that tracks selection (distinct from data).
- **`TableRowSorter`:** Sorting and filtering for tables.
- **`TableCellRenderer` and `TableCellEditor`:** Control how cells are drawn and edited.
- **`AbstractTableModel`:** The recommended base class for custom models.
- **`DefaultTableModel`:** The ready-made mutable implementation.
- **`SwingUtilities.invokeLater`:** Ensures model mutations occur on the EDT.

---

## Core Concepts and Features

---

## 1. TableModel — The Interface

### Definitions

**Core Definition.** `TableModel` is the minimal interface that any data source must implement to back a `JTable`.

**Technical Definition.** `public interface TableModel`. It declares `getRowCount`, `getColumnCount`, `getValueAt`, `setValueAt`, `getColumnName`, `getColumnClass`, `isCellEditable`, `addTableModelListener`, and `removeTableModelListener`. A correct implementation **must** notify registered listeners on every content or structure change. The row/column indices are zero-based and refer to the model, not the view (which may differ when sorting or filtering).

**Beginner-Friendly Explanation.** `TableModel` is the "contract" that says: "tell me how many rows and columns you have, give me the value at any cell, tell me the column names and types, tell me whether a cell is editable, and let me register a listener so I know when things change."

---

### Purposes

- To provide a data source for `JTable` that is independent of the visual representation.
- To allow arbitrary data structures (arrays, lists, database cursors, domain objects) to back a table.
- To define the notification contract that keeps views synchronized.
- To control cell editing, rendering, and type information.
- To enable read-only virtual tables (e.g., computing cells on demand).

---

### Syntax Rules and Structure

#### Complete General Syntax (Implementing TableModel)

```java
public class MyTableModel implements TableModel {
    private final List<Row> rows = new ArrayList<>();
    private final List<TableModelListener> listeners = new ArrayList<>();

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return 2; }
    public Object getValueAt(int r, int c) { /* return cell */ }
    public void setValueAt(Object v, int r, int c) { /* update cell */ }
    public String getColumnName(int c) { return c == 0 ? "Name" : "Age"; }
    public Class<?> getColumnClass(int c) { return c == 0 ? String.class : Integer.class; }
    public boolean isCellEditable(int r, int c) { return false; }

    public void addTableModelListener(TableModelListener l) { listeners.add(l); }
    public void removeTableModelListener(TableModelListener l) { listeners.remove(l); }
    // Must also fire events when data changes.
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getRowCount()` | Returns the number of rows. |
| `getColumnCount()` | Returns the number of columns. |
| `getValueAt(int, int)` | Returns the value at the specified cell. |
| `setValueAt(Object, int, int)` | Sets the value at the specified cell. |
| `getColumnName(int)` | Returns the column header. |
| `getColumnClass(int)` | Returns the column's class (used for rendering/editing). |
| `isCellEditable(int, int)` | Returns whether the cell can be edited. |
| `addTableModelListener(...)` / `removeTableModelListener(...)` | Manage listeners. |

#### Syntax Rules

- `getValueAt` must return a valid value for every cell in `[0, rowCount) × [0, columnCount)`.
- The model **must** notify listeners when contents or structure change.
- Implementing `TableModel` directly requires manual listener management.

#### Constraints and Limitations

- Implementing `TableModel` directly is error-prone; `AbstractTableModel` is the recommended base.
- `TableModel` does not enforce mutability; read-only models are valid.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Read-Only Virtual TableModel

```java
import javax.swing.*;
import javax.swing.table.TableModel;
import javax.swing.event.TableModelListener;
import java.awt.*;

public class VirtualTableModel implements TableModel {
    @Override public int getRowCount() { return 1_000_000; }             // 1. Huge
    @Override public int getColumnCount() { return 2; }

    @Override
    public Object getValueAt(int r, int c) {                             // 2. Lazy
        return c == 0 ? "Row " + r : (r * 2);
    }

    @Override public String getColumnName(int c) { return c == 0 ? "Label" : "Value"; }
    @Override public Class<?> getColumnClass(int c) { return c == 0 ? String.class : Integer.class; }
    @Override public boolean isCellEditable(int r, int c) { return false; }
    @Override public void setValueAt(Object v, int r, int c) { /* no-op */ }
    @Override public void addTableModelListener(TableModelListener l) { /* no-op */ }
    @Override public void removeTableModelListener(TableModelListener l) { /* no-op */ }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Virtual TableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JTable table = new JTable(new VirtualTableModel());
            table.setAutoResizeMode(JTable.AUTO_RESIZE_OFF);
            frame.add(new JScrollPane(table));
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A scrollable table showing “Row 0 / 0”, “Row 1 / 2”, etc., computed on demand. No memory is allocated for all one million rows.

**Why the code produces this result.** `getValueAt` computes cell values lazily. The read-only model has no listeners to notify, so those methods are no-ops. This pattern is ideal for large or computed datasets.

---

### Real-World Cases

- **Virtual tables** with millions of rows computed on demand.
- **Read-only views** of fixed data or database cursors.
- **Custom data sources** that cannot afford the memory of a `Vector` of `Vector`s.

---

## 2. AbstractTableModel — The Abstract Base Class

### Definitions

**Core Definition.** `AbstractTableModel` is the abstract base class that implements the **listener bookkeeping** for `TableModel` while leaving the three essential data-access methods to the subclass.

**Technical Definition.** `public abstract class AbstractTableModel extends Object implements TableModel, Serializable`. It implements `addTableModelListener` and `removeTableModelListener`, and provides protected fire methods: `fireTableDataChanged()`, `fireTableRowsInserted(int, int)`, `fireTableRowsUpdated(int, int)`, `fireTableRowsDeleted(int, int)`, `fireTableCellUpdated(int, int)`, and `fireTableStructureChanged()`. It **contains no specific data structure internally**. Subclasses must implement `getRowCount()`, `getColumnCount()`, and `getValueAt(int, int)`. It provides default implementations for `getColumnName` (returns “A”, “B”, “C”…), `getColumnClass` (`Object.class`), `isCellEditable` (`false`), and `setValueAt` (no-op).

**Beginner-Friendly Explanation.** `AbstractTableModel` is the "half-built spreadsheet" — the listener plumbing is already installed, but you supply the data and call the `fire` methods when it changes.

---

### Purposes

- To reduce boilerplate by providing listener registration and event firing.
- To serve as the recommended base class for custom models backed by arbitrary data structures.
- To enable clean separation between data management and event notification.
- To support both read-only and mutable custom models.
- To provide sensible default implementations for the non-essential methods.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyModel extends AbstractTableModel {
    private final List<Person> data = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    public int getRowCount() { return data.size(); }
    public int getColumnCount() { return cols.length; }
    public Object getValueAt(int r, int c) {
        Person p = data.get(r);
        return c == 0 ? p.getName() : p.getAge();
    }
    public String getColumnName(int c) { return cols[c]; }

    public void addRow(Person p) {
        int index = data.size();
        data.add(p);
        fireTableRowsInserted(index, index);                             // 1. Fire
    }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `fireTableDataChanged()` | Notifies that all data may have changed. |
| `fireTableRowsInserted(int firstRow, int lastRow)` | Notifies that rows were inserted. |
| `fireTableRowsUpdated(int firstRow, int lastRow)` | Notifies that rows were updated. |
| `fireTableRowsDeleted(int firstRow, int lastRow)` | Notifies that rows were deleted. |
| `fireTableCellUpdated(int row, int column)` | Notifies that a single cell changed. |
| `fireTableStructureChanged()` | Notifies that the structure (columns) changed. |

#### Syntax Rules

- Subclasses must implement `getRowCount`, `getColumnCount`, and `getValueAt`.
- After mutating the backing data, call the appropriate `fireXxx` method.
- `fireTableStructureChanged` should be used when the columns change; it causes the table to rebuild its `TableColumnModel`.

#### Constraints and Limitations

- `AbstractTableModel` does not store data; the subclass must provide its own backing store.
- Failing to fire events leaves the `JTable` stale.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Model Backed by a List of Domain Objects

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class PersonTableModel extends AbstractTableModel {
    public static class Person {
        String name; int age;
        Person(String n, int a) { name = n; age = a; }
    }

    private final List<Person> people = new ArrayList<>();              // 1. Data
    private final String[] cols = {"Name", "Age"};

    @Override public int getRowCount() { return people.size(); }        // 2. Size
    @Override public int getColumnCount() { return cols.length; }
    @Override public String getColumnName(int c) { return cols[c]; }

    @Override
    public Object getValueAt(int r, int c) {                            // 3. Access
        Person p = people.get(r);
        return c == 0 ? p.name : p.age;
    }

    @Override
    public Class<?> getColumnClass(int c) {                             // 4. Type
        return c == 0 ? String.class : Integer.class;
    }

    @Override
    public boolean isCellEditable(int r, int c) {                       // 5. Editable
        return c == 1;
    }

    @Override
    public void setValueAt(Object v, int r, int c) {                    // 6. Update
        if (c == 1) {
            people.get(r).age = ((Number) v).intValue();
            fireTableCellUpdated(r, c);                                  // 7. Fire
        }
    }

    public void addPerson(Person p) {                                    // 8. Insert
        int index = people.size();
        people.add(p);
        fireTableRowsInserted(index, index);                             // 9. Fire
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Person TableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            PersonTableModel model = new PersonTableModel();
            model.addPerson(new Person("Alice", 30));
            model.addPerson(new Person("Bob", 25));

            JTable table = new JTable(model);
            frame.add(new JScrollPane(table));
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A two-column table (“Name”, “Age”) showing Alice/30 and Bob/25. The Age column is editable; double-clicking an age and changing it updates the model and fires a cell-update event.

**Why the code produces this result.** The model extends `AbstractTableModel`, implements the essential methods, and calls `fireTableCellUpdated` after edits and `fireTableRowsInserted` after additions.

---

### Real-World Cases

- **Domain-driven applications** where the table displays a `List<Person>`, `List<Order>`, or other domain objects.
- **Database-backed tables** where the model wraps a `ResultSet` or a DAO.
- **Custom sorted/filtered tables** where the model exposes a derived view.

---

## 3. DefaultTableModel — The Mutable Implementation

### Definitions

**Core Definition.** `DefaultTableModel` is the concrete, mutable implementation of `TableModel` that stores cell values in a `Vector` of `Vector`s and provides convenience methods for row and column manipulation.

**Technical Definition.** `public class DefaultTableModel extends AbstractTableModel implements Serializable`. It implements a `Vector`-based API, including `addRow`, `insertRow`, `removeRow`, `addColumn`, `setValueAt`, `setDataVector`, and `getDataVector`. Constructors accept a row/column array, a `Vector` of rows, or a column name vector. The model fires the appropriate `TableModelEvent`s automatically. It is "the easiest model for simple tables that do not require a custom backing data structure".

**Beginner-Friendly Explanation.** `DefaultTableModel` is the "ready-made spreadsheet" — you can add rows, remove rows, and modify cells with simple method calls, and the table updates automatically.

---

### Purposes

- To provide a ready-to-use mutable model for simple tables.
- To support dynamic row and column addition/removal.
- To serve as the default model for `JTable` when no custom model is provided.
- To enable quick prototyping without writing a custom model.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultTableModel model = new DefaultTableModel(
    new Object[][]{{"Alice", 30}, {"Bob", 25}},
    new Object[]{"Name", "Age"});
JTable table = new JTable(model);
model.addRow(new Object[]{"Carol", 35});
model.removeRow(0);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `DefaultTableModel()` | Empty model with zero rows and columns. |
| `DefaultTableModel(int rowCount, int columnCount)` | Creates an empty grid of the specified size. |
| `DefaultTableModel(Object[][] data, Object[] columnNames)` | Creates a model with the given data and column names. |
| `DefaultTableModel(Vector<?> columnNames, int rowCount)` | Creates a model with the specified columns and row count. |

#### Key Methods

| Method | Purpose |
|--------|---------|
| `addRow(Object[])` | Appends a row; fires `INSERT`. |
| `insertRow(int, Object[])` | Inserts a row at the index; fires `INSERT`. |
| `removeRow(int)` | Removes the row; fires `DELETE`. |
| `addColumn(Object)` | Appends a column. |
| `setValueAt(Object, int, int)` | Sets a cell value; fires `UPDATE`. |
| `setDataVector(Vector, Vector)` | Replaces the entire data and columns. |
| `getDataVector()` | Returns the raw `Vector` of `Vector`s. |

#### Syntax Rules

- All mutating methods fire the appropriate `TableModelEvent` automatically.
- The internal storage is a `Vector` of `Vector`s of `Object`s.
- Mutations should occur on the EDT.

#### Constraints and Limitations

- `DefaultTableModel` uses a `Vector`-based storage; for custom data structures, extend `AbstractTableModel` instead. The Java documentation recommends `AbstractTableModel` for subclasses and `DefaultTableModel` only when no subclassing is required.
- The model does not enforce type information unless the subclass overrides `getColumnClass`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Rows with DefaultTableModel

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class DefaultTableModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DefaultTableModel Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(                // 1. Model
                new Object[]{"Name", "Age"}, 0);
            JTable table = new JTable(model);

            JTextField nameField = new JTextField(10);
            JButton add = new JButton("Add");
            add.addActionListener(e -> {                                     // 2. Add
                if (!nameField.getText().isEmpty()) {
                    model.addRow(new Object[]{nameField.getText(), 0});
                    nameField.setText("");
                }
            });

            JButton remove = new JButton("Remove Selected");
            remove.addActionListener(e -> {                                  // 3. Remove
                int row = table.getSelectedRow();
                if (row >= 0) model.removeRow(row);
            });

            JPanel controls = new JPanel();
            controls.add(nameField);
            controls.add(add);
            controls.add(remove);

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(table), BorderLayout.CENTER);
            frame.add(controls, BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The table starts with columns Name and Age but no rows. Typing a name and clicking “Add” appends a row. Selecting a row and clicking “Remove Selected” removes it.

**Why the code produces this result.** `DefaultTableModel` handles all event firing. `addRow` fires an `INSERT` event; `removeRow` fires a `DELETE` event; the `JTable` repaints automatically.

---

### Real-World Cases

- **Simple data entry tables** where rows are added and removed at runtime.
- **Prototyping** before building a custom model.
- **Configuration editors** with dynamic key/value rows.
- **Todo lists** and task managers with add/remove row operations.

---

## 4. Table Model Methods — Complete Reference

### Definitions

**Core Definition.** The `TableModel` methods provide access to the tabular data, control editing and rendering, and notify listeners of changes.

**Technical Definition.** The table below summarises the methods across all three types.

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `getRowCount()` | `int` | Number of rows. |
| `getColumnCount()` | `int` | Number of columns. |
| `getValueAt(int, int)` | `Object` | Cell value at the given row and column. |
| `setValueAt(Object, int, int)` | `void` | Sets the cell value; fires `UPDATE`. |
| `getColumnName(int)` | `String` | Column header name. |
| `getColumnClass(int)` | `Class<?>` | Column class (for rendering/editing). |
| `isCellEditable(int, int)` | `boolean` | Whether the cell is editable. |
| `addTableModelListener(...)` | `void` | Registers a listener. |
| `removeTableModelListener(...)` | `void` | Unregisters a listener. |
| `fireTableDataChanged()` | `void` | (AbstractTableModel) All data changed. |
| `fireTableRowsInserted(int, int)` | `void` | (AbstractTableModel) Rows inserted. |
| `fireTableRowsUpdated(int, int)` | `void` | (AbstractTableModel) Rows updated. |
| `fireTableRowsDeleted(int, int)` | `void` | (AbstractTableModel) Rows deleted. |
| `fireTableCellUpdated(int, int)` | `void` | (AbstractTableModel) Cell updated. |
| `fireTableStructureChanged()` | `void` | (AbstractTableModel) Columns changed. |
| `addRow(Object[])` | `void` | (DefaultTableModel) Appends a row. |
| `insertRow(int, Object[])` | `void` | (DefaultTableModel) Inserts a row. |
| `removeRow(int)` | `void` | (DefaultTableModel) Removes a row. |
| `addColumn(Object)` | `void` | (DefaultTableModel) Appends a column. |
| `setDataVector(Vector, Vector)` | `void` | (DefaultTableModel) Replaces all data. |

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getValueAt` | The fundamental data access method; called during painting. |
| `setValueAt` | Called by the cell editor when the user commits an edit. |
| `isCellEditable` | Controls whether the default editor starts. |
| `getColumnClass` | Selects the default renderer and editor. |
| `fireTableCellUpdated` | The most efficient notification for single-cell changes. |
| `fireTableStructureChanged` | Triggers a complete rebuild of the column model. |

#### Syntax Rules

- `getValueAt` must return a value for every valid cell; returning `null` is allowed.
- `setValueAt` should only modify the model if `isCellEditable` returns `true` for that cell.
- `getColumnClass` should return a specific class (`String.class`, `Integer.class`, `Boolean.class`, etc.) to enable appropriate default renderers and editors.
- Fire the narrowest event that accurately describes the change: `fireTableCellUpdated` for one cell, `fireTableRowsUpdated` for a row range, `fireTableDataChanged` for everything.

#### Constraints and Limitations

- `getValueAt` is called many times during painting; it must be fast.
- `fireTableStructureChanged` also resets the column model's widths and order, so use it sparingly.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Editing Cells with `isCellEditable` and `setValueAt`

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.awt.*;

public class EditableTableModel extends AbstractTableModel {
    private final String[] names = {"Alice", "Bob", "Carol"};
    private final int[] ages = {30, 25, 35};
    private final String[] cols = {"Name", "Age"};

    public int getRowCount() { return names.length; }
    public int getColumnCount() { return 2; }
    public String getColumnName(int c) { return cols[c]; }
    public Class<?> getColumnClass(int c) { return c == 0 ? String.class : Integer.class; }

    public Object getValueAt(int r, int c) {
        return c == 0 ? names[r] : ages[r];
    }

    public boolean isCellEditable(int r, int c) { return c == 1; }       // 1. Editable

    public void setValueAt(Object v, int r, int c) {                     // 2. Set
        if (c == 1) {
            ages[r] = ((Number) v).intValue();
            fireTableCellUpdated(r, c);                                   // 3. Fire
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Editable TableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            EditableTableModel model = new EditableTableModel();
            JTable table = new JTable(model);

            frame.add(new JScrollPane(table));
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A table with a read-only “Name” column and an editable “Age” column. Double-clicking an age opens the default editor; after editing, the model updates and fires a cell event.

**Why the code produces this result.** `isCellEditable` returns `true` only for the Age column, so the default editor only activates there. `setValueAt` updates the backing array and fires `fireTableCellUpdated`. `getColumnClass` returns `Integer.class` for Age, which selects the default numeric editor.

---

#### Example 2: Using `DefaultTableModel` with `setValueAt`

```java
DefaultTableModel model = new DefaultTableModel(
    new Object[][]{{"Alice", 30}}, new Object[]{"Name", "Age"});
model.setValueAt(31, 0, 1);                                            // Fires UPDATE
```

**Expected Output.** The table's first row's age is updated to 31.

---

### Real-World Cases

- **Spreadsheet-style applications** with editable cells.
- **Configuration tables** with editable values.
- **Data browsers** with read-only virtual data and occasional updates.

---

## 5. Table Model Best Practices

### Definitions

**Core Definition.** Table model best practices are the conventions and patterns that ensure tabular data handling is correct, thread-safe, and maintainable.

**Technical Definition.** Best practices include: extending `AbstractTableModel` for custom data structures; using `DefaultTableModel` only for simple mutable tables; implementing `getColumnClass` to enable correct renderers and editors; firing the narrowest appropriate event after mutations; mutating models on the EDT; using `RowSorter` for sorting rather than sorting the model; and converting between view and model indices when sorting or filtering is active.

**Beginner-Friendly Explanation.** Best practices for table models are the "rules of thumb" that prevent stale views, frozen GUIs, and unmaintainable code. They include things like "extend `AbstractTableModel` for custom data," "fire the right event," "use `getColumnClass`," and "use `RowSorter` for sorting."

---

### Purposes

- To choose the right model implementation for the data source.
- To keep views synchronized by firing the correct events.
- To maintain thread safety by respecting the EDT.
- To enable correct rendering and editing through `getColumnClass` and `isCellEditable`.
- To avoid index confusion when sorting or filtering is active.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class BestPracticeTableModel extends AbstractTableModel {        // 1. Abstract
    private final List<Row> rows = new ArrayList<>();
    private final String[] cols = {"Name", "Age"};

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return cols.length; }
    public String getColumnName(int c) { return cols[c]; }
    public Class<?> getColumnClass(int c) {                             // 2. Class
        return c == 0 ? String.class : Integer.class;
    }
    public Object getValueAt(int r, int c) { /* ... */ }
    public boolean isCellEditable(int r, int c) { return c == 1; }      // 3. Editable

    public void addRow(Row row) {
        int i = rows.size();
        rows.add(row);
        fireTableRowsInserted(i, i);                                    // 4. Fire
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend `AbstractTableModel` for custom data | `class MyModel extends AbstractTableModel` |
| Use `DefaultTableModel` only for simple cases | `new DefaultTableModel(...)` |
| Implement `getColumnClass` | Return `String.class`, `Integer.class`, etc. |
| Implement `isCellEditable` appropriately | `return c == 1;` |
| Fire the narrowest event | `fireTableCellUpdated`, `fireTableRowsUpdated`, etc. |
| Mutate on EDT | `SwingUtilities.invokeLater(...)` |
| Use `RowSorter` for sorting | `table.setAutoCreateRowSorter(true)` |
| Convert view/model indices | `convertRowIndexToModel`, `convertColumnIndexToModel` |

#### Syntax Rules

- **Always extend `AbstractTableModel` for custom data structures**; it handles listener plumbing and provides sensible defaults.
- **Use `DefaultTableModel` only when subclassing is not required**, as the Java documentation recommends.
- **Always implement `getColumnClass`** to enable correct default renderers and editors; otherwise, everything is treated as `Object` and rendered as text.
- **Always implement `isCellEditable`** to control editing; the default returns `false`.
- **Always fire the appropriate `fireXxx` method** after mutations; the narrowest event that accurately describes the change is best.
- **Always use `RowSorter` for sorting** rather than sorting the model itself; `setAutoCreateRowSorter(true)` is the shortcut.
- **Always convert between view and model indices** when sorting or filtering is active.
- **Always mutate models on the EDT**.

#### Constraints and Limitations

- `DefaultTableModel` uses a `Vector` internally; for large or custom data, extend `AbstractTableModel`.
- `fireTableStructureChanged` resets the column widths and order; use it only when the column structure genuinely changes.
- View and model indices differ when sorting or filtering; incorrect conversions cause off-by-one bugs.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom Model with Sorting

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
    public Class<?> getColumnClass(int c) {
        return c == 0 ? String.class : Integer.class;
    }
    public Object getValueAt(int r, int c) {
        Person p = people.get(r);
        return c == 0 ? p.name : p.age;
    }
    public boolean isCellEditable(int r, int c) { return c == 1; }
    public void setValueAt(Object v, int r, int c) {
        if (c == 1) {
            people.get(r).age = ((Number) v).intValue();
            fireTableCellUpdated(r, c);
        }
    }
    public void addPerson(Person p) {
        int i = people.size();
        people.add(p);
        fireTableRowsInserted(i, i);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeTableModel::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice TableModel");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        BestPracticeTableModel model = new BestPracticeTableModel();
        model.addPerson(new Person("Carol", 35));
        model.addPerson(new Person("Alice", 30));
        model.addPerson(new Person("Bob", 25));

        JTable table = new JTable(model);
        table.setAutoCreateRowSorter(true);                              // 1. Sorting

        JButton add = new JButton("Add Row");
        add.addActionListener(e ->
            model.addPerson(new Person("New " + model.getRowCount(), 0)));

        JPanel panel = new JPanel(new java.awt.BorderLayout());
        panel.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
        panel.add(add, java.awt.BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(450, 300);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A table with sortable columns (clicking a header sorts by that column), an editable Age column, and an “Add Row” button. The model fires the correct events, and the sorter keeps the view in order.

**Why the code produces this result.** The model extends `AbstractTableModel`, implements `getColumnClass` and `isCellEditable`, and fires the appropriate events. `setAutoCreateRowSorter(true)` installs a `TableRowSorter` that sorts the view without changing the model.

---

#### Example 2: Common Mistake — Mutating Without Firing Events

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class BadTableModel extends AbstractTableModel {
    private final List<String> rows = new ArrayList<>();

    public int getRowCount() { return rows.size(); }
    public int getColumnCount() { return 1; }
    public Object getValueAt(int r, int c) { return rows.get(r); }

    public void addRow(String s) {
        rows.add(s);                                                     // WRONG: no fire
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad TableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            BadTableModel model = new BadTableModel();
            JTable table = new JTable(model);
            JButton add = new JButton("Add");
            add.addActionListener(e -> model.addRow("Row"));
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

**Expected Output.** Clicking “Add” adds rows to the internal list, but the table **does not update** because no `TableModelEvent` was fired.

**Why this is wrong.** The `JTable` relies on `TableModelEvent`s to know when to repaint. Mutating the internal list without calling `fireTableRowsInserted` leaves the view stale. The correct approach is to call the appropriate `fireXxx` method after every mutation.

---

#### Example 3: Common Mistake — Using `getValueAt` with Slow Computations

```java
public Object getValueAt(int r, int c) {
    // WRONG: database query on every paint
    return database.query("SELECT ... WHERE row = " + r);
}
```

**Expected Output.** The table becomes extremely slow to scroll or repaint because `getValueAt` is called for every visible cell on every repaint.

**Why this is wrong.** `getValueAt` is a performance-critical method; it is called many times during painting. Expensive computations should be cached in the model's backing data, and only `getValueAt`'s fast lookup should run on each call.

---

### Real-World Cases

- **Production Swing applications** extend `AbstractTableModel` for custom data sources, implement `getColumnClass` and `isCellEditable`, and use `RowSorter` for sorting.
- **Database-backed tables** cache query results in a `List` and refresh the cache explicitly, firing `fireTableDataChanged`.
- **Large datasets** use virtual `TableModel`s that compute rows on demand.
- **Educational examples** use the best-practice skeleton to demonstrate correct model design from the start.

---

## References

Oracle. *Interface TableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/TableModel.html

Oracle. *Class AbstractTableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/AbstractTableModel.html

Oracle. *Class DefaultTableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/DefaultTableModel.html

Oracle. *How to Use Tables*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/table.html

Oracle. *How to Use Tables — Creating a Table Model*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/table.html#data

Oracle. *Class TableModelEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TableModelEvent.html

Oracle. *Interface TableModelListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TableModelListener.html

Oracle. *Class TableRowSorter*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/TableRowSorter.html

Oracle. *Class RowSorter*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/RowSorter.html

Oracle. *Class JTable*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JTable.html

Cleverence. *JTable in Java SE 8: Complete Guide to Models, Rendering, Sorting & Editing*. https://www.cleverence.com/articles/oracle-documentation/jtable-java-platform-se-8-4827/

Stack Overflow. *JTable — AbstractTableModel vs DefaultTableModel*. https://stackoverflow.com/questions/2454413/jtable-abstracttablemodel-vs-defaulttablemodel

Stack Overflow. *How to use RowSorter with a custom TableModel*. https://stackoverflow.com/

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading