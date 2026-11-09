# JTable — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JTable` is a Swing component used to display and edit regular two-dimensional tables of cells. It is the workhorse table component of Java Swing, designed for building desktop tools, admin consoles, and cross-platform utilities that display and edit tabular data.

**Technical Definition.** `public class JTable extends JComponent implements TableModelListener, Scrollable, TableColumnModelListener, ListSelectionListener, CellEditorListener, Accessible, RowSorterListener`. The `JTable` follows an MVC‑inspired architecture: a `TableModel` owns the data and notifies listeners when values or structure change; the `JTable` (the view) uses row/column coordinates to query the model and paint cells; and pluggable delegates handle how cells are drawn (`TableCellRenderer`) and how edits occur (`TableCellEditor`). A `JTable` does not contain or cache data—this lack of caching makes `JTable` perform well even when its data comes from a huge database. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JTable` is the Swing component that displays data in a grid—rows and columns, like a spreadsheet. It is used for showing lists of records, inventory items, user accounts, or any data that naturally fits into a table. It is important to understand that the `JTable` is just the view: it does not hold the data itself. Instead, you supply a model (such as `DefaultTableModel` or a custom `AbstractTableModel` subclass) that provides the data and manages changes. The table then displays whatever the model contains, and when the user edits a cell, the table updates the model. Think of the model as the filing cabinet and the `JTable` as the window through which you view and edit the files.

---

### Key Characteristics

1. **MVC architecture.** `JTable` separates data ( `TableModel` ), view (`JTable`), column configuration ( `TableColumnModel` ), and selection state ( `ListSelectionModel` ). Delegates handle cell rendering and editing.
2. **No data caching.** `JTable` does not cache data; it queries the model via `getValueAt(int, int)` during painting. This makes it efficient even for very large datasets.
3. **Model-driven.** The `TableModel` is the single source of truth. `DefaultTableModel` provides a convenient, mutable, `Vector`-based implementation. `AbstractTableModel` is the recommended base class when you need a custom model backed by your own data structures.
4. **Scroll pane integration.** `JTable`s are typically placed inside a `JScrollPane`. By default, the table adjusts its width so a horizontal scrollbar is unnecessary. To allow horizontal scrolling, call `setAutoResizeMode(AUTO_RESIZE_OFF)`.
5. **Selection models.** Row selection and column selection are managed by `ListSelectionModel` instances. The selection mode can be `SINGLE_SELECTION`, `SINGLE_INTERVAL_SELECTION`, or `MULTIPLE_INTERVAL_SELECTION`.
6. **Sorting and filtering.** A `RowSorter` can be attached to enable row sorting and filtering. The easiest way is `setAutoCreateRowSorter(true)`.
7. **View vs. model coordinates.** Column and row indexes returned by `JTable` methods are in terms of the view and may not match the model’s indexes if columns are rearranged or rows are sorted.
8. **Single-thread rule and serialization caveat.** `JTable` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, generics, and lambda expressions.
- Familiarity with Swing containers (`JFrame`, `JPanel`, `JScrollPane`) and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ListSelectionListener`, `TableModelListener`, and the `javax.swing.event` package.
- Basic understanding of the MVC pattern.
- A JDK (version 8 or later; `JTable` is available in all modern JDKs).

---

### Related Programming Areas

- **`TableModel` and `AbstractTableModel`:** The data model interfaces and the recommended base class for custom models.
- **`DefaultTableModel`:** A convenient, mutable, `Vector`-based model implementation.
- **`TableColumnModel` and `ListSelectionModel`:** Manage column configuration and selection state.
- **`TableCellRenderer` and `TableCellEditor`:** Control how cells are drawn and edited.
- **`RowSorter` and `TableRowSorter`:** Enable sorting and filtering.
- **`JScrollPane`:** The standard container for tables.
- **`SwingWorker`:** For loading large datasets off the EDT.

---

## Core Concepts and Features

---

## 1. JTable — The Class

### Definitions

**Core Definition.** `JTable` is the Swing class for creating and displaying two-dimensional tables of data.

**Technical Definition.** `public class JTable extends JComponent`. It provides constructors that accept row/column data arrays, a `TableModel`, or a `TableModel` and `TableColumnModel`. The table delegates data access to the model and handles painting, editing, and selection through its UI delegate and associated models.

**Beginner-Friendly Explanation.** `JTable` is the class you use to create a table. You give it a model (or arrays of data), wrap it in a `JScrollPane`, and add it to your window.

---

### Purposes

- To display tabular data in a grid of rows and columns.
- To allow users to view, select, and optionally edit cell values.
- To separate data management (model) from presentation (view) for maintainability.
- To support sorting, filtering, and custom rendering/editing of cells.
- To integrate with `JScrollPane` for scrolling through large datasets.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JTable table = new JTable();                                     // empty
JTable table = new JTable(Object[][] rowData, Object[] columnNames); // array data
JTable table = new JTable(TableModel model);                     // custom model
JTable table = new JTable(TableModel model, TableColumnModel columnModel);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JTable()` | Creates an empty table with a default model. |
| `JTable(Object[][], Object[])` | Creates a table with the given data and column names; internally creates a `DefaultTableModel`. |
| `JTable(TableModel)` | Creates a table using the supplied model. |
| `JTable(TableModel, TableColumnModel)` | Uses both the given data model and column model. |

#### Syntax Rules

- The table must be added to a container (typically a `JScrollPane`) to be displayed.
- The model must not be `null`.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JTable` does not store data; it queries the model on demand.
- Column and row indexes in `JTable` methods are view-based and may differ from model indexes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JTable with Array Data

```java
import javax.swing.*;
import java.awt.*;

public class MinimalTable {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JTable");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[][] data = {
                {"Alice", "30", "Engineer"},
                {"Bob", "25", "Designer"},
                {"Carol", "35", "Manager"}
            };
            String[] columns = {"Name", "Age", "Role"};

            JTable table = new JTable(data, columns);             // 1. Create
            JScrollPane scroll = new JScrollPane(table);           // 2. Scroll pane

            frame.add(scroll, BorderLayout.CENTER);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing a table with three columns (“Name,” “Age,” “Role”) and three rows of data. The table is scrollable.

**Why the code produces this result.** The array constructor creates an internal `DefaultTableModel`. The `JScrollPane` provides scrolling and also displays the table header.

---

### Real-World Cases

- **Admin consoles** listing users, orders, or inventory.
- **Data browsers** displaying query results from a database.
- **Configuration screens** showing application settings in tabular form.

---

## 2. Table Creation

### Definitions

**Core Definition.** Table creation is the process of instantiating a `JTable` with data, either from arrays or from a `TableModel`.

**Technical Definition.** `JTable` provides constructors that accept raw arrays, a `TableModel`, or both a `TableModel` and a `TableColumnModel`. When arrays are supplied, a `DefaultTableModel` is created internally.

**Beginner-Friendly Explanation.** Creating a table means deciding how to supply the data—via simple arrays for quick tables, or via a model for more control.

---

### Purposes

- To instantiate the table with the appropriate data source.
- To choose between the convenience of array-based creation and the flexibility of a custom model.
- To prepare the table for scroll pane wrapping and configuration.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTable table = new JTable(dataArray, columnNamesArray);
JTable table = new JTable(tableModel);
JScrollPane scroll = new JScrollPane(table);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Object[][] rowData` | The cell values, organized by row. |
| `Object[] columnNames` | The column headers. |
| `TableModel model` | The data model. |

#### Syntax Rules

- The table must be wrapped in a `JScrollPane` to display the header and support scrolling.
- The model may be replaced later via `setModel`.

#### Constraints and Limitations

- Array-based creation creates a `DefaultTableModel` that uses a `Vector` of `Vector`s internally.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Table with a Custom Model

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.awt.*;

public class CustomModelTable {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            AbstractTableModel model = new AbstractTableModel() {   // 1. Custom model
                String[] cols = {"ID", "Name"};
                Object[][] data = {{1, "Alpha"}, {2, "Beta"}};
                public int getRowCount() { return data.length; }
                public int getColumnCount() { return cols.length; }
                public Object getValueAt(int r, int c) { return data[r][c]; }
                public String getColumnName(int c) { return cols[c]; }
            };

            JTable table = new JTable(model);                       // 2. Use model
            frame.add(new JScrollPane(table));
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A table displaying two columns (“ID,” “Name”) and two rows of data.

**Why the code produces this result.** The anonymous `AbstractTableModel` subclass provides the required methods (`getRowCount`, `getColumnCount`, `getValueAt`), and the `JTable` queries it to render the cells.

---

### Real-World Cases

- **Database-backed tables** where the model wraps a `ResultSet`.
- **Custom data structures** (e.g., `List<Person>`) exposed through a `TableModel`.

---

## 3. Table Model — DefaultTableModel

### Definitions

**Core Definition.** `DefaultTableModel` is a concrete `TableModel` implementation that stores cell values in a `Vector` of `Vector`s of `Object`s. It provides convenience methods for adding, removing, and manipulating rows and columns.

**Technical Definition.** `public class DefaultTableModel extends AbstractTableModel`. It is the easiest model for simple tables that do not require a custom backing data structure. It is mutable and fires `TableModelEvent`s when data changes.

**Beginner-Friendly Explanation.** `DefaultTableModel` is the “easy” table model. You can add rows, remove rows, and change cell values with simple method calls, and the table updates automatically.

---

### Purposes

- To provide a ready-to-use mutable model for simple tables.
- To avoid writing a custom `TableModel` subclass when the data fits naturally into a `Vector`-based structure.
- To support dynamic row addition and removal.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultTableModel model = new DefaultTableModel();
model.addColumn("Name");
model.addRow(new Object[]{"Alice"});
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `addColumn(Object)` | Adds a column. |
| `addRow(Object[])` | Adds a row of data. |
| `removeRow(int)` | Removes the row at the given index. |
| `setValueAt(Object, int, int)` | Sets the value at the given row and column. |

#### Syntax Rules

- The model is mutable by default.
- Changes automatically fire `TableModelEvent`s, causing the table to repaint.

#### Constraints and Limitations

- `DefaultTableModel` does not support sorting or filtering directly; use a `RowSorter` if needed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Rows with DefaultTableModel

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class DefaultModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("DefaultTableModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(       // 1. Model
                new Object[]{"Name", "Age"}, 0);
            JTable table = new JTable(model);
            JTextField nameField = new JTextField(10);
            JButton addButton = new JButton("Add");

            addButton.addActionListener(e -> {
                model.addRow(new Object[]{nameField.getText(), "?"}); // 2. Add row
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(table), BorderLayout.CENTER);
            JPanel south = new JPanel();
            south.add(nameField);
            south.add(addButton);
            panel.add(south, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** An initially empty table with columns “Name” and “Age.” Typing a name and clicking “Add” appends a row with that name and “?” in the age column.

**Why the code produces this result.** `DefaultTableModel` is created with column names and zero rows. `addRow` appends data, and the model notifies the table of the change.

---

### Real-World Cases

- **Simple data entry forms** where rows are added dynamically.
- **Prototyping** before building a custom model.

---

## 4. Table Model — AbstractTableModel

### Definitions

**Core Definition.** `AbstractTableModel` is an abstract implementation of `TableModel` that provides the event plumbing (listener registration and firing) and leaves only three methods to be implemented: `getRowCount()`, `getColumnCount()`, and `getValueAt(int, int)`.

**Technical Definition.** `public abstract class AbstractTableModel implements TableModel, Serializable`. The Java documentation recommends using `AbstractTableModel` as the base class for creating custom models, and `DefaultTableModel` only when subclassing is not required.

**Beginner-Friendly Explanation.** `AbstractTableModel` is the recommended starting point for custom table models. You extend it, implement three methods, and your model can be used by a `JTable`. It handles the listener bookkeeping for you.

---

### Purposes

- To provide a base class for custom models backed by application-specific data structures.
- To avoid the `Vector`-based storage of `DefaultTableModel`.
- To enable efficient integration with existing domain models (e.g., `List<Person>`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class MyModel extends AbstractTableModel {
    public int getRowCount() { return data.size(); }
    public int getColumnCount() { return COLUMNS.length; }
    public Object getValueAt(int r, int c) { return ...; }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getRowCount()` | Returns the number of rows. |
| `getColumnCount()` | Returns the number of columns. |
| `getValueAt(int, int)` | Returns the value at the given row and column. |

#### Syntax Rules

- Must implement all three abstract methods.
- Use `fireTableDataChanged()`, `fireTableRowsInserted()`, etc., to notify the table of changes.

#### Constraints and Limitations

- `AbstractTableModel` does not store data; you must provide your own backing store.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Model Backed by a List

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class ListModelDemo {
    static class Person {
        String name; int age;
        Person(String n, int a) { name = n; age = a; }
    }

    static class PersonModel extends AbstractTableModel {          // 1. Extend
        private final List<Person> people = new ArrayList<>();
        private final String[] cols = {"Name", "Age"};
        public int getRowCount() { return people.size(); }
        public int getColumnCount() { return cols.length; }
        public String getColumnName(int c) { return cols[c]; }
        public Object getValueAt(int r, int c) {
            Person p = people.get(r);
            return c == 0 ? p.name : p.age;
        }
        void add(Person p) {
            people.add(p);
            fireTableRowsInserted(people.size() - 1, people.size() - 1); // 2. Fire
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("List-Backed Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            PersonModel model = new PersonModel();
            model.add(new Person("Alice", 30));                     // 3. Add data
            model.add(new Person("Bob", 25));

            JTable table = new JTable(model);
            frame.add(new JScrollPane(table));
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A table with “Name” and “Age” columns, showing “Alice 30” and “Bob 25.”

**Why the code produces this result.** The `PersonModel` extends `AbstractTableModel` and implements the three required methods. `fireTableRowsInserted` notifies the table of new rows.

---

### Real-World Cases

- **Domain-driven applications** where the table displays a `List` of business objects.
- **Large datasets** where copying data into a `DefaultTableModel` would be wasteful.

---

## 5. Table Methods — Model Access (setModel / getModel)

### Definitions

**Core Definition.** `setModel(TableModel)` replaces the table’s data model; `getModel()` returns the current model.

**Technical Definition.** `public void setModel(TableModel dataModel)` and `public TableModel getModel()`. Replacing the model triggers the table to re-query all data and update its display.

**Beginner-Friendly Explanation.** `setModel` swaps the entire dataset of the table. `getModel` gives you access to the current model.

---

### Purposes

- To change the table’s data source at runtime.
- To access the model for direct data manipulation.
- To share a model across multiple tables.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
table.setModel(newModel);
TableModel current = table.getModel();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setModel(TableModel)` | Replaces the model. |
| `getModel()` | Returns the current model. |

#### Syntax Rules

- The model must not be `null`.
- The table revalidates and repaints after the model is replaced.

#### Constraints and Limitations

- Selection is cleared when the model changes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Models

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;

public class SetModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel modelA = new DefaultTableModel(
                new Object[][]{{"A1"}, {"A2"}}, new Object[]{"Col"});
            DefaultTableModel modelB = new DefaultTableModel(
                new Object[][]{{"B1"}, {"B2"}, {"B3"}}, new Object[]{"Col"});

            JTable table = new JTable(modelA);
            JButton switchButton = new JButton("Switch Model");
            switchButton.addActionListener(e ->
                table.setModel(modelB));                            // 1. Switch

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            panel.add(switchButton, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The table initially shows two rows (“A1,” “A2”). Clicking “Switch Model” replaces them with three rows (“B1,” “B2,” “B3”).

**Why the code produces this result.** `setModel` swaps the data source, and the table repaints with the new content.

---

### Real-World Cases

- **Switching datasets** based on user selection.
- **Refreshing data** from a database.

---

## 6. Table Methods — Value Access (getValueAt / setValueAt)

### Definitions

**Core Definition.** `getValueAt(int row, int column)` returns the value at the specified cell; `setValueAt(Object aValue, int row, int column)` sets the value.

**Technical Definition.** These methods delegate to the `TableModel`. The `JTable` uses `getValueAt` during painting to retrieve the value for each cell. `setValueAt` updates the model, which fires a `TableModelEvent` to refresh the display.

**Beginner-Friendly Explanation.** `getValueAt` reads a cell; `setValueAt` writes to a cell.

---

### Purposes

- To read cell values programmatically.
- To update cell values without direct model access.
- To implement custom logic that depends on cell contents.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Object value = table.getValueAt(row, col);
table.setValueAt(newValue, row, col);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getValueAt(int, int)` | Returns the cell value. |
| `setValueAt(Object, int, int)` | Sets the cell value. |

#### Syntax Rules

- Row and column indexes are view-based; use `convertRowIndexToModel` and `convertColumnIndexToModel` if the view order differs from the model order.
- The model must support writes (i.e., `isCellEditable` must return `true` for the cell).

#### Constraints and Limitations

- `getValueAt` returns `null` for out-of-range indexes or uninitialized cells.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading and Writing Cells

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;

public class ValueAccessDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Value Access");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[][]{{"Alice", 30}}, new Object[]{"Name", "Age"});
            JTable table = new JTable(model);

            JButton read = new JButton("Read Cell");
            read.addActionListener(e ->
                System.out.println("Value: " + table.getValueAt(0, 0))); // 1. Read

            JButton write = new JButton("Write Cell");
            write.addActionListener(e ->
                table.setValueAt("Bob", 0, 0));                       // 2. Write

            JPanel panel = new JPanel();
            panel.add(read);
            panel.add(write);

            frame.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            frame.add(panel, java.awt.BorderLayout.SOUTH);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Read Cell” prints “Value: Alice.” Clicking “Write Cell” changes the first cell to “Bob.”

**Why the code produces this result.** `getValueAt` retrieves the cell value; `setValueAt` updates the model, and the table repaints.

---

### Real-World Cases

- **Data validation** after user edits.
- **Programmatic updates** from background processes.

---

## 7. Table Methods — Selection Access

### Definitions

**Core Definition.** `getSelectedRow()` returns the index of the first selected row, or `-1` if none. `getSelectedRows()` returns all selected row indices. `getSelectedColumn()` returns the index of the first selected column. `getSelectedColumns()` returns all selected column indices.

**Technical Definition.** These methods delegate to the `ListSelectionModel`. `getSelectedRows()` returns an array of integers containing the indices of all selected rows, or an empty array if no row is selected.

**Beginner-Friendly Explanation.** These methods tell you which rows and columns the user has selected.

---

### Purposes

- To read the user’s selection for processing.
- To support single-row and multi-row operations.
- To synchronize the selection with other UI components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int row = table.getSelectedRow();
int[] rows = table.getSelectedRows();
int col = table.getSelectedColumn();
int[] cols = table.getSelectedColumns();
```

#### Component Breakdown

| Method | Return |
|--------|--------|
| `getSelectedRow()` | First selected row index, or `-1`. |
| `getSelectedRows()` | Array of selected row indices (empty if none). |
| `getSelectedColumn()` | First selected column index, or `-1`. |
| `getSelectedColumns()` | Array of selected column indices (empty if none). |

#### Syntax Rules

- Always check for `-1` or empty arrays before using the results.
- Row indices are view-based; convert to model indices if sorting or filtering is active.

#### Constraints and Limitations

- Column selection is enabled only if `setColumnSelectionAllowed(true)` has been called.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading Selection

```java
import javax.swing.*;

public class SelectionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Selection");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[][] data = {{"A"}, {"B"}, {"C"}};
            JTable table = new JTable(data, new String[]{"Col"});

            JButton show = new JButton("Show Selection");
            show.addActionListener(e -> {
                int[] rows = table.getSelectedRows();                  // 1. Get rows
                System.out.print("Selected rows: ");
                for (int r : rows) System.out.print(r + " ");
                System.out.println();
            });

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            panel.add(show, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting multiple rows and clicking “Show Selection” prints their indices.

**Why the code produces this result.** `getSelectedRows()` returns an array of selected row indices.

---

### Real-World Cases

- **Bulk operations** on selected rows.
- **Detail views** that update based on the selected row.

---

## 8. Table Methods — setSelectionMode()

### Definitions

**Core Definition.** `setSelectionMode(int mode)` sets whether the table allows single or multiple row/column selection.

**Technical Definition.** `public void setSelectionMode(int selectionMode)`. The mode is passed to the `ListSelectionModel`. Valid constants are `ListSelectionModel.SINGLE_SELECTION`, `SINGLE_INTERVAL_SELECTION`, and `MULTIPLE_INTERVAL_SELECTION`.

**Beginner-Friendly Explanation.** `setSelectionMode` controls how many rows the user can select at once.

---

### Purposes

- To restrict selection to a single row.
- To allow contiguous or non-contiguous multi-row selection.
- To match the table’s behavior to the application’s requirements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `SINGLE_SELECTION` | One row at a time. |
| `SINGLE_INTERVAL_SELECTION` | One contiguous range. |
| `MULTIPLE_INTERVAL_SELECTION` | Any combination (default). |

#### Syntax Rules

- The default is `MULTIPLE_INTERVAL_SELECTION`.
- The mode affects row selection by default; column selection has its own selection model.

#### Constraints and Limitations

- Changing the mode may deselect rows that violate the new mode.

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

            JTable table = new JTable(
                new String[][]{{"A"}, {"B"}, {"C"}}, new String[]{"Col"});
            table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION); // 1. Single

            frame.add(new JScrollPane(table));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Only one row can be selected at a time.

**Why the code produces this result.** The selection model enforces single selection.

---

### Real-World Cases

- **Single-record detail views.**
- **Master-detail interfaces** where one master row is selected.

---

## 9. Table Methods — setAutoResizeMode()

### Definitions

**Core Definition.** `setAutoResizeMode(int mode)` controls whether and how the table automatically resizes its columns to fill the available width.

**Technical Definition.** `public void setAutoResizeMode(int mode)`. The default mode causes the table to adjust its width so that a horizontal scrollbar is unnecessary. To allow a horizontal scrollbar, use `AUTO_RESIZE_OFF`. Other modes include `AUTO_RESIZE_LAST_COLUMN`, `AUTO_RESIZE_SUBSEQUENT_COLUMNS`, `AUTO_RESIZE_NEXT_COLUMN`, and `AUTO_RESIZE_ALL_COLUMNS`.

**Beginner-Friendly Explanation.** This method controls whether columns stretch to fill the table or stay fixed width (allowing horizontal scrolling).

---

### Purposes

- To prevent unwanted horizontal scrollbars.
- To allow horizontal scrolling for wide tables.
- To control column resizing behavior when the table is resized.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
table.setAutoResizeMode(JTable.AUTO_RESIZE_OFF);
```

#### Component Breakdown

| Constant | Behavior |
|----------|----------|
| `AUTO_RESIZE_OFF` | No automatic resizing; horizontal scrollbar may appear. |
| `AUTO_RESIZE_LAST_COLUMN` | The last column absorbs size changes. |
| `AUTO_RESIZE_ALL_COLUMNS` | All columns share size changes. |

#### Syntax Rules

- The default mode avoids horizontal scrollbars.

#### Constraints and Limitations

- With `AUTO_RESIZE_OFF`, the table may be wider than the viewport, requiring scrolling.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Disabling Auto-Resize

```java
import javax.swing.*;

public class AutoResizeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Auto Resize");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTable table = new JTable(
                new String[][]{{"A"}, {"B"}}, new String[]{"Column"});
            table.setAutoResizeMode(JTable.AUTO_RESIZE_OFF);       // 1. Off

            frame.add(new JScrollPane(table));
            frame.setSize(200, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The table column keeps its default width, and a horizontal scrollbar appears if the column is wider than the window.

**Why the code produces this result.** `AUTO_RESIZE_OFF` prevents automatic column resizing.

---

### Real-World Cases

- **Wide tables** with many columns that require scrolling.
- **Fixed-width column layouts.**

---

## 10. Table Methods — setRowHeight() / setShowGrid() / setGridColor()

### Definitions

**Core Definition.** `setRowHeight(int)` sets the height of all rows. `setShowGrid(boolean)` toggles the display of grid lines. `setGridColor(Color)` sets the color of the grid lines.

**Technical Definition.** These methods control the visual appearance of the table. The default grid color is gray.

**Beginner-Friendly Explanation.** These methods adjust the table’s visual styling: row height, whether grid lines are shown, and what color they are.

---

### Purposes

- To control row height for readability or compactness.
- To show or hide grid lines for a cleaner look.
- To customize grid color for theming.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
table.setRowHeight(25);
table.setShowGrid(false);
table.setGridColor(Color.BLUE);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setRowHeight(int)` | Sets row height in pixels. |
| `setShowGrid(boolean)` | `false` hides grid lines. |
| `setGridColor(Color)` | Sets the grid line color. |

#### Syntax Rules

- Row height must be positive.

#### Constraints and Limitations

- Very small row heights may clip cell content.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Customizing Table Appearance

```java
import javax.swing.*;
import java.awt.*;

public class AppearanceDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Appearance");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTable table = new JTable(
                new String[][]{{"A"}, {"B"}}, new String[]{"Col"});
            table.setRowHeight(30);                                 // 1. Row height
            table.setShowGrid(false);                               // 2. No grid
            table.setGridColor(Color.RED);                          // 3. Grid color

            frame.add(new JScrollPane(table));
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A table with taller rows and no visible grid lines.

**Why the code produces this result.** `setRowHeight` increases row size, `setShowGrid(false)` hides grid lines, and `setGridColor` sets the color that would be used if grids were shown.

---

### Real-World Cases

- **Touch-friendly UIs** with larger row heights.
- **Minimalist designs** without grid lines.

---

## 11. Table Events — ListSelectionListener

### Definitions

**Core Definition.** `ListSelectionListener` is an interface for receiving `ListSelectionEvent`s that fire when the table’s row or column selection changes.

**Technical Definition.** `public interface ListSelectionListener extends EventListener`. The `valueChanged(ListSelectionEvent e)` method is invoked when the selection changes. The listener should check `getValueIsAdjusting()` to avoid reacting to intermediate states during a drag.

**Beginner-Friendly Explanation.** A `ListSelectionListener` runs when the user selects or deselects rows. You use it to react to the new selection.

---

### Purposes

- To react to row selection changes.
- To update detail views or enable buttons based on selection.
- To perform actions when a row is selected.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
table.getSelectionModel().addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        // selection finalized
    }
});
```

#### Component Breakdown

| Element | Meaning |
|--------|---------|
| `getSelectionModel()` | Returns the row selection model. |
| `addListSelectionListener(...)` | Registers the listener. |
| `getValueIsAdjusting()` | `true` during drag-selection. |

#### Syntax Rules

- The listener is invoked on the EDT.
- Use `getValueIsAdjusting()` to avoid premature reactions.

#### Constraints and Limitations

- The event does not carry the selected indices; query the table.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reacting to Row Selection

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;

public class SelectionListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ListSelectionListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTable table = new JTable(
                new String[][]{{"Alice"}, {"Bob"}}, new String[]{"Name"});
            JLabel status = new JLabel("Select a row");

            table.getSelectionModel().addListSelectionListener(e -> { // 1. Listener
                if (!e.getValueIsAdjusting()) {                      // 2. Final
                    int row = table.getSelectedRow();
                    if (row >= 0) {
                        status.setText("Selected: " + table.getValueAt(row, 0));
                    }
                }
            });

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            panel.add(status, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting a row updates the status label with the selected name.

**Why the code produces this result.** The `ListSelectionListener` checks `getValueIsAdjusting()` and reads the selected row to update the label.

---

### Real-World Cases

- **Master-detail views** that update when a row is selected.
- **Enabling action buttons** only when a row is selected.

---

## 12. Table Events — TableModelListener

### Definitions

**Core Definition.** `TableModelListener` is an interface for receiving `TableModelEvent`s that fire when the table’s data model changes (e.g., cell values change, rows are added or removed).

**Technical Definition.** `public interface TableModelListener extends EventListener`. The `tableChanged(TableModelEvent e)` method is invoked when the model changes. The `JTable` itself automatically uses a `TableModelListener` to keep its GUI synchronized with the model.

**Beginner-Friendly Explanation.** A `TableModelListener` runs when the data in the table changes. You can use it to react to cell edits, row insertions, or deletions.

---

### Purposes

- To react to data changes in the model.
- To validate or process cell edits.
- To synchronize other UI components with the table’s data.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
table.getModel().addTableModelListener(e -> {
    int row = e.getFirstRow();
    int col = e.getColumn();
    // react to change
});
```

#### Component Breakdown

| Element | Meaning |
|--------|---------|
| `getModel()` | Returns the `TableModel`. |
| `addTableModelListener(...)` | Registers the listener. |
| `getFirstRow()` / `getColumn()` | Identify the changed cell. |

#### Syntax Rules

- The listener is invoked on the EDT.
- `getColumn()` returns `TableModelEvent.ALL_COLUMNS` if the change affects all columns.

#### Constraints and Limitations

- The event does not carry the new value; query the model via `getValueAt`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting Cell Changes

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;

public class ModelListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("TableModelListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTable table = new JTable(
                new String[][]{{"Alice"}}, new String[]{"Name"});
            JLabel status = new JLabel("No changes");

            table.getModel().addTableModelListener(e -> {            // 1. Listener
                if (e.getType() == TableModelEvent.UPDATE) {
                    int row = e.getFirstRow();
                    int col = e.getColumn();
                    Object value = table.getModel().getValueAt(row, col); // 2. Read
                    status.setText("Updated: " + value);
                }
            });

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            panel.add(status, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Editing the cell updates the status label with the new value.

**Why the code produces this result.** The `TableModelListener` fires on `UPDATE` events and reads the new value from the model.

---

### Real-World Cases

- **Live validation** of cell edits.
- **Audit logging** of data changes.

---

## 13. JTable Best Practices

### Definitions

**Core Definition.** `JTable` best practices are the conventions and patterns that ensure tables are used effectively, thread-safely, and performantly in Swing applications.

**Technical Definition.** Best practices include: always wrapping `JTable` in a `JScrollPane`; using `AbstractTableModel` as the base for custom models rather than extending `DefaultTableModel`; avoiding `null` layouts and `setBounds`; offloading large data loads to a `SwingWorker`; and converting between view and model indices when sorting or filtering is active.

**Beginner-Friendly Explanation.** Best practices for tables are the “rules of thumb” that prevent frozen GUIs, data inconsistency, and poor performance. They include things like “always use a scroll pane,” “use `AbstractTableModel` for custom data,” and “don’t do heavy work on the GUI thread.”

---

### Purposes

- To ensure the table scrolls correctly and displays its header.
- To maintain a clean separation between data and presentation.
- To keep the GUI responsive during large data operations.
- To avoid index confusion when sorting or filtering is enabled.
- To create accessible and maintainable tables.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    AbstractTableModel model = new MyCustomModel(data);          // 1. Custom model
    JTable table = new JTable(model);                             // 2. Table
    table.setAutoCreateRowSorter(true);                           // 3. Sorting
    table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);  // 4. Selection
    JScrollPane scroll = new JScrollPane(table);                  // 5. Scroll pane
    frame.add(scroll, BorderLayout.CENTER);
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use a scroll pane | `new JScrollPane(table)` |
| Use `AbstractTableModel` for custom data | Extend it and implement three methods. |
| Avoid `null` layouts | Use layout managers. |
| Enable sorting safely | `setAutoCreateRowSorter(true)` |
| Convert view/model indices | `convertRowIndexToModel`, `convertColumnIndexToModel` |
| Off-EDT loading | `SwingWorker` for large datasets. |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always wrap `JTable` in a `JScrollPane`** to display the header and enable scrolling.
- **Always use `AbstractTableModel` as the base for custom models** rather than extending `DefaultTableModel`.
- **Never use `null` layouts or `setBounds`** on the table or its scroll pane.
- **Always convert between view and model indices** when sorting or filtering is active.
- **Never load large datasets on the EDT**; use `SwingWorker`.

#### Constraints and Limitations

- `DefaultTableModel` uses a `Vector` and is not suitable for custom data structures.
- Sorting and filtering change the view order, making index conversion essential.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Table with Custom Model and Sorting

```java
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import java.awt.*;

public class BestPracticeTable {
    static class MyModel extends AbstractTableModel {               // 1. Abstract
        private final String[] cols = {"Name", "Age"};
        private final Object[][] data = {{"Alice", 30}, {"Bob", 25}};
        public int getRowCount() { return data.length; }
        public int getColumnCount() { return cols.length; }
        public Object getValueAt(int r, int c) { return data[r][c]; }
        public String getColumnName(int c) { return cols[c]; }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Table");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            MyModel model = new MyModel();                           // 2. Model
            JTable table = new JTable(model);                         // 3. Table
            table.setAutoCreateRowSorter(true);                       // 4. Sorting
            table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION); // 5. Mode

            JScrollPane scroll = new JScrollPane(table);               // 6. Scroll
            frame.add(scroll, BorderLayout.CENTER);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A table with “Name” and “Age” columns, sortable by clicking column headers, with single-row selection.

**Why the code produces this result.** The custom `AbstractTableModel` provides data. `setAutoCreateRowSorter(true)` enables sorting. The scroll pane displays the header and supports scrolling.

---

#### Example 2: Common Mistake — Extending DefaultTableModel

```java
import javax.swing.table.DefaultTableModel;

public class BadTableModel extends DefaultTableModel {           // WRONG: extend DTM
    public BadTableModel() {
        super(new Object[][]{{"A"}}, new Object[]{"Col"});
    }
    // Custom logic here...
}
```

**Expected Output.** The model may work, but it couples the data to `DefaultTableModel`’s internal `Vector` representation, making it harder to integrate with existing domain models.

**Why this is wrong.** As the Java documentation and community guidance state, `AbstractTableModel` should be used as the base class for creating subclasses, and `DefaultTableModel` only when subclassing is not required. Extending `DefaultTableModel` is discouraged because it does not expect you to have your own data in a subclass.

---

### Real-World Cases

- **Production admin consoles** use custom `AbstractTableModel` subclasses backed by `List` or database results.
- **Data browsers** enable sorting with `setAutoCreateRowSorter(true)` and convert indices when interacting with the model.
- **Educational examples** use the best-practice skeleton to demonstrate correct table usage from the start.

---

## References

Oracle. *Class JTable*. Java Development Kit Version 16 API Specification. https://docs.oracle.com/en/java/javase/16/docs/api/java.desktop/javax/swing/JTable.html

Oracle. *Class JTable*. Java 2 Platform SE 6. https://download.oracle.com/technetwork/java/javase/6/docs/zh/api/javax/swing/JTable.html

Oracle. *How to Use Tables*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/table.html

Cleverence. *JTable in Java SE 8: Complete Guide to Models, Rendering, Sorting & Editing*. https://www.cleverence.com/articles/oracle-documentation/jtable-java-platform-se-8-4827/

Stack Overflow. *JTable and constructor parameters*. https://stackoverflow.com/questions/...

Stack Overflow. *JTable best practices — avoid null layouts, use JScrollPane*. https://stackoverflow.com/revisions/...

Oracle. *JTable selection methods — getSelectedRow, getSelectedRows, getSelectedColumn, getSelectedColumns*. https://docs.oracle.com/javase/8/docs/api/javax/swing/JTable.html

Oracle. *JTable appearance methods — setRowHeight, setShowGrid, setGridColor, setAutoResizeMode*. https://docs.oracle.com/javase/8/docs/api/javax/swing/JTable.html

Oracle. *How to Write a Table Model Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/tablemodellistener.html